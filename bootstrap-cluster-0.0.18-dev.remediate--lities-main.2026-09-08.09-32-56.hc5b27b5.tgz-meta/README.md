# mctl

`mctl` automates the creation and initial configuration of GiantSwarm management clusters.

It replaces [mc-bootstrap](https://github.com/giantswarm/mc-bootstrap) with a gitops-first approach: instead of imperatively provisioning everything in one shot, `mctl` produces pull requests that are reviewed before anything is applied to the cluster.

## How it works

Setting up a new management cluster has three stages:

**1. `mctl init`** — creates the cluster App manifest and opens a PR in the CMC repository. No secrets are generated. This is the point to review and adjust cluster values (network config, feature flags, etc.) before committing.

**2. `mctl config`** — reads the manifest from the init PR branch, generates all secrets and credentials (AGE key, SSH deploy keys, Teleport token, provider IAM/SP credentials, Dex config, Grafana tokens, etc.), renders them into files using templates, and opens PRs in:
- The CMC repository (secrets, kustomizations, configmaps)
- The CCR repository (`installations/<name>/`)
- The installations repository (`cluster.yaml`)

Progress within `mctl config` is checkpointed — re-running after a failure resumes from the last completed step. See [the runbook](docs/runbook.md) for details.

**3. Bootstrap** — once the PRs are merged and Flux is running on the management cluster, `mctl create mc` can be used to spin up the cluster using a temporary kind bootstrap cluster. This phase-based command handles CAPI installation, cluster creation, Flux setup, pivot, and cleanup. Individual phases can be skipped or re-run with `--start-from`.

## Quick start

```sh
# 1. Create the manifest PR
mctl init \
  --installation <name> \
  --provider capa \
  --base-domain <name>.eu-west-1.aws.gigantic.io \
  --region eu-west-1 \
  --account-id 123456789012

# Review and merge (or amend) the PR, then:

# 2. Generate configuration PRs
op run --env-file .env.tpl -- mctl config \
  --installation <name> \
  --cmc-repo giantswarm-management-clusters

# 3. Bootstrap the cluster (after PRs are merged)
mctl create mc --values-file <name>.yaml
```

Pass `--dry-run` to either command to print generated output without creating any GitHub PRs or external resources.

### Config file input

For installations with complex values, you can supply a YAML file to `mctl init` instead of individual flags:

```sh
mctl init --installation <name> --config-file config.yaml
```

The file uses the same structure as the cluster App values ConfigMap. See [configuration reference](docs/configuration.md) for the field mapping.

## Create an MC from CI

The whole flow can run in the CMC repository, driven by PR comments. You never run `mctl` locally, except for the two interactive steps listed below.

### Steps

1. **Create a branch** named `<installation>-init-<provider>`, for example `antiaws-init-capa`. Valid providers: `capa`, `capz`, `capv`, `capvcd`.

   The `Init` workflow runs `mctl init` with placeholder values, opens a PR, and comments the exact `/init` command to use.

2. **Fill in the values.** Comment on the PR with the provider flags:

   ```
   /init --region eu-west-1 --account-id 123456789012 --base-domain test.eu-west-1.aws.gigantic.io
   ```

   The manifest is regenerated on the same branch. Repeat as needed until the `validate-manifest` check is green.

3. **Generate the configuration.** Comment `/config`.

   This runs `mctl config --ci`. The bot replies with the CCR PR and installations PR links, plus any step that CI cannot do (see [Manual steps](#manual-steps)). The `validate-config` check then verifies the output.

4. **Bootstrap the cluster.** Comment `/create-mc` (org members and collaborators only).

   The CCR PR is merged first, so Flux can read the config, then the Tekton `mctl-create-mc` pipeline starts. Progress and the result are posted back to the PR.

5. **Merge the init PR** once the `mctl/ready-to-merge` status is green. This merges the installations PR and arms auto-merge on the teleport-fleet PR.

   Closing the PR without merging closes all three PRs instead.

6. **Register the Teleport bot.** The merge comment contains the `tctl bots add` command. This step is not automated.

### Manual steps

`mctl config --ci` skips the steps that need a browser or an interactive token. The `/config` comment lists them with a ready-to-run command:

| Step | Reason | Command |
|---|---|---|
| Dex | Azure AD admin consent needs a browser redirect | `op run --env-file .env.tpl -- mctl config --installation <name> --dex` |
| Slack | The Slack app config token is only available from the Slack UI and expires in ~12h | `op run --env-file .env.tpl -- mctl config --installation <name> --slack` |

Run them from a local checkout, then continue with `/create-mc`.

### Init flags

The `/init` comment accepts any `mctl init` flag. Required per provider:

| Provider | Flags |
|---|---|
| `capa` | `--base-domain` `--region` `--account-id` |
| `capz` | `--base-domain` `--subscription-id` `--tenant-id` `--location` |
| `capv` | `--base-domain` `--server` |
| `capvcd` | `--base-domain` `--site` `--org` `--ovdc` `--ovdc-network` `--vip-subnet` |

Useful optional flags:

| Flag | Default | Description |
|---|---|---|
| `--pipeline` | `testing` | `testing` \| `stable` \| `ephemeral` |
| `--release-version` | latest | GiantSwarm release to pin, for example `v34.1.0` |
| `--dns-delegation-iam-role` | | CAPA only; needed when the parent DNS zone is in another AWS account |

Installation name, provider, customer, CMC repo and CCR repo are derived from the branch and repository names, so do not pass them.

### Repository setup

One-time, per CMC repository:

1. Copy the workflows from [`.github/workflows-cmc-template/`](.github/workflows-cmc-template) into `.github/workflows/`.
2. Copy [`op.env`](.github/workflows-cmc-template/op.env) to `.github/workflows/op.env` and set the `op://` paths of the `mctl` 1Password vault.
3. Set the repository variable `MCTL_IMAGE` to the mctl image to run.
4. Set the repository secrets:

   | Secret | Used for |
   |---|---|
   | `TAYLORBOT_GITHUB_ACTION` | Cross-repo write access (CMC, CCR, installations, teleport-fleet) |
   | `OP_SERVICE_ACCOUNT_TOKEN` | 1Password service account, scoped to the vault in `op.env` |

5. Register `mctl/ready-to-merge` as a required status check on `main`:

   ```sh
   gh api -X PATCH \
     repos/giantswarm/<customer>-management-clusters/branches/main/protection/required_status_checks \
     -f 'contexts[]=mctl/ready-to-merge'
   ```

6. Register the Tekton EventListener as a webhook on the repository and apply the pipeline resources — see [`tekton-cmc-template/`](tekton-cmc-template) and [the runbook](docs/runbook.md).

## Clean up

`mctl clean` removes the resources a test run creates. The default set deletes the local bootstrap cluster, closes the GitHub PRs, removes the deploy keys, and resets local state. Add `--provider` to also delete the cloud infrastructure.

```sh
# Azure teardown (subscription from AZURE_SUBSCRIPTION_ID)
mctl clean --installation <name> --provider azure --force

# AWS teardown
mctl clean --installation <name> --provider aws --force
```

`GITHUB_TOKEN` is required for the GitHub steps. Cloud credentials come from `az login` (Azure) or the active AWS profile.

| Flag | Effect |
|---|---|
| `--provider aws\|azure` | Also delete that cloud's infrastructure. Omit to skip cloud cleanup. |
| `--dry-run` | Show the resources to delete. Delete nothing. |
| `--force` | Skip the per-step confirmation prompts. |
| `--list-steps` | Print all steps and mark the ones that run. |
| `--enable` / `--disable` | Run only, or skip, the given step slugs (comma-separated). |
| `--customer` | Customer whose repos hold the cluster's config (`<customer>-management-clusters` / `-configs`). Used to close the init/CCR PRs and delete deploy keys. Default: `giantswarm`. |

Azure options:

| Flag | Effect |
|---|---|
| `--subscription` | Subscription ID. Default: `AZURE_SUBSCRIPTION_ID`. |
| `--resource-group` | Resource group. Default: the installation name. |
| `--shared-resource-group` | Delete the resource group only when empty. Default deletes the whole group. |

After a run, `mctl clean` prints a **Manual follow-up** summary: removal PRs it opened for the cluster's already-merged config/token PRs (review and merge them), and the parent-zone DNS delegation it cannot remove.

See [docs/clean.md](docs/clean.md) for the full step list and the AWS options.

## Relation to mc-bootstrap

`mctl` is the successor to `mc-bootstrap`. The key differences:

- **Gitops by default**: `mctl init` and `mctl config` produce PRs for review rather than applying changes directly.
- **Config lives in the manifest**: topology settings and feature flags that were env vars in mc-bootstrap are now chart values or annotations in the cluster App manifest committed to the CMC repo.
- **Credentials still use env vars**: external service credentials (tokens, API keys) are still passed at runtime via `op run --env-file`.
- **Resumable**: `mctl config` checkpoints progress to `.mctl-state.json` and can be safely re-run.

See [docs/configuration.md](docs/configuration.md) for the full mc-bootstrap → mctl migration reference.

## Documentation

- [**Runbook**](docs/runbook.md) — step-by-step guide for bringing up a new MC
- [**Configuration reference**](docs/configuration.md) — all flags, env vars, annotations, and the mc-bootstrap mapping
- [**Cleaning up a test run**](docs/clean.md) — `mctl clean` for AWS and Azure teardown
