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

**3. Bootstrap** — once the PRs are merged and Flux is running on the management cluster, `mctl create mc` can be used to spin up the cluster using a temporary bootstrap cluster. This phase-based command handles CAPI installation, cluster creation, Flux setup, pivot, verification, and tearing the bootstrap cluster down again. Individual phases can be skipped or re-run with `--start-from`.

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

1. **Create a branch** named `<installation>-init-<provider>`, for example `test-mc-init-capa`. Valid providers: `capa`, `capz`, `capv`, `capvcd`.

   The `Init` workflow runs `mctl init` with placeholder values, opens a PR, and comments the exact `/init` command to use.

2. **Fill in the values.** Comment on the PR with the provider flags:

   ```
   /init --region eu-west-1 --account-id 123456789012 --base-domain test.eu-west-1.aws.gigantic.io
   ```

   The manifest is regenerated on the same branch. Repeat as needed until the `validate-manifest` check is green.

   **CAPA only:** this also opens a PR in `giantswarm/aws-account-setup` (see [AWS account setup](#aws-account-setup)). Approve it before `/config`.

3. **Generate the configuration.** Comment `/config`.

   This runs `mctl config --ci`. The bot replies with the CCR PR and installations PR links, plus any step that CI cannot do (see [Manual steps](#manual-steps)). The `validate-config` check then verifies the output.

4. **Bootstrap the cluster.** Comment `/create-mc` (org members and collaborators only).

   The CCR PR is merged first, so Flux can read the config, then the Tekton `mctl-create-mc` pipeline starts. Progress and the result are posted back to the PR.

5. **Merge the init PR** once the `mctl/ready-to-merge` status is green. This merges the installations PR and arms auto-merge on the teleport-fleet PR.

   Closing the PR without merging closes all three PRs instead.

6. **Register the Teleport bots.** The merge comment contains both `tctl bots add` commands. This step is not automated.

### AWS account setup

CAPA installations need two IAM users in the Giant Swarm root account, `<mc>-capa-controller` and `<mc>-crossplane`. Terraform in `aws-account-setup` creates them, so `mctl config` fails until that config is applied.

`mctl init` opens the PR for this automatically and arms auto-merge. What it contains depends on the customer:

| Customer in `scripts/customers.sh` | PR contains |
|---|---|
| Listed | one `management_clusters` entry |
| Not listed | the entry, plus the customer's whole OpenTofu configuration |

The second case is a new customer's first CAPA installation. Check the customer name before approving — it comes from the CMC repository name and creates a permanent directory and OpenTofu state.

Approval is always required: that repo's `main` needs one review and enforces it for admins. Merging applies the Terraform.

Test installations are exempt. `giantswarm` and `antonia` pool under `customers/giantswarm`, which already exists.

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

One-time, per customer:

```sh
GITHUB_TOKEN=$(gh auth token) op run --env-file .env.tpl -- mctl create cmc-repo --customer <customer>
```

This creates `<customer>-management-clusters` from
[template-management-clusters](https://github.com/giantswarm/template-management-clusters)
(which carries `bases/`, `extras/` and `CODEOWNERS`) and sets the
`giantswarm-mctlbot` App secrets and the Tekton webhook. With `--mctl-image` it
also sets the `MCTL_IMAGE` variable, an override (e.g. a canary build) for the
mctl version pinned in the workflows.

The GitHub Actions workflows are not installed by mctl. They are generated by
devctl's `fleet` flavour (`.github/workflows/zz_generated.mctl_*.yaml`, templates
in devctl `pkg/gen/input/workflows/internal/file/mctl_*.yaml.template`) and synced
by align-files. After creating the repo:

1. Declare it in [giantswarm/github](https://github.com/giantswarm/github)
   `repositories/team-honeybadger.yaml` with `gen.flavours: [fleet]`.
2. Once the workflows are on `main`, add `requiredChecks: [mctl/ready-to-merge]`
   there too (see [Merge gate](docs/runbook.md#merge-gate-mctlready-to-merge)).

`op.env` (the 1Password references `/config` needs) is not installed into the
repo either: the config workflow fetches [`ci/op.env`](ci/op.env) from this repo
at runtime, at the tag matching its mctl image. Keep that path stable.

Re-running is safe; every item is compared before it is written:

```
  Repository             already exists
  App secrets            applied (3 secrets, cannot be verified)
  Tekton webhook         already correct (https://…; secret rewritten)
```

The Actions secrets are the exception: the API never returns them, so they are
rewritten every run. The webhook's shared secret is unreadable for the same
reason, so `already correct` there covers the url, events and active flag only.

`mctl init` runs *inside* the CMC repo in CI, so the repo has to exist before
anything else can happen, which is why this step is local and human-run. The
configs (CCR) repository is not created here; `mctl config` creates it on the
first run for a customer.

| Flag | Default | Description |
|---|---|---|
| `--customer` | | Customer name; the repo is `<customer>-management-clusters` (required) |
| `--org` | `giantswarm` | GitHub organisation |
| `--mctl-image` | unset | Sets the repo's `MCTL_IMAGE` variable, overriding the workflows' pinned image |
| `--webhook-url` | cicdprod endpoint | Tekton EventListener endpoint; point at cicddev to test |

Credentials come from the environment:

| Variable | Used for |
|---|---|
| `GITHUB_TOKEN` | Creating the repo and setting its secrets, variable and webhook — needs org-admin rights |
| `MCTL_APP_ID` | ID of the `giantswarm-mctlbot` GitHub App |
| `MCTL_APP_PRIVATE_KEY` | Private key of the `giantswarm-mctlbot` GitHub App — the workflows mint a short-lived installation token from it for cross-repo write access (CMC, CCR, installations, teleport-fleet) |
| `OP_SERVICE_ACCOUNT_TOKEN` | 1Password service account, scoped to the vault in [`ci/op.env`](ci/op.env) |
| `TEKTON_WEBHOOK_SECRET` | Shared secret the Tekton EventListener validates deliveries against |

The Tekton pipeline resources themselves live in
[`giantswarm/tekton-resources`](https://github.com/giantswarm/tekton-resources)
(`tekton-resources/tekton-pipelines/{pipelines,rbac,triggers}/mctl-create-mc.yaml`)
and are reconciled onto the `cicdprod` and `cicddev` clusters by Flux. See also
[the runbook](docs/runbook.md).

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
- [**Ephemeral MCs**](docs/ephemeral.md) — `create mc --ephemeral` for nightly test MCs (goten, goose) on the test Teleport
