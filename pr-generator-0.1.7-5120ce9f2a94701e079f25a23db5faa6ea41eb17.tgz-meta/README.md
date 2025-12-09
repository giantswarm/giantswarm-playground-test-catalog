# pr-generator

[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/pr-generator/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/pr-generator/tree/main)

## What is this?

This application is a Slack bot that creates PR to Github automatically.

## Features

**PR generation**

- Add emoji :github-pr-opened and automatically the bot will create a PR in the repo you have linked for that channel

Configuration looks like

```yaml
```

## How to release

1. Create a release PR in this repo, e.g: `main#release#minor` branch
2. Merge the above release PR.
3. Create a PR in `workload-clusters-fleet` repo to bump version in [App CR](https://github.com/giantswarm/workload-clusters-fleet/blob/main/management-clusters/gazelle/organizations/giantswarm-production/workload-clusters/operations/apps/pr-generator/appcr.yaml)
4. Merge the above PR - GitOps will rollout the app upgrade.

## Behind the scene


