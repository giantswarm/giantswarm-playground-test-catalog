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
