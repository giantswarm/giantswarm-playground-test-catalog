# FAIQ

[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/faiq/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/faiq/tree/main)

## What is this?

This application is a Slack bot that uses OpenAI to generate a summary of Slack messages.

## Features
Currently, bot has two features:
1. **Thread Summarizer** - Posts a summary of all messages sent to a thread when reacted with `:support-recpie` emoji or using message shortcut `Create a thread summary`.
2. **Channel Summarizer** - Posts a summary of all messages sent to a channel for a given period (in days) when invoked: `/summarize [days]`

## How to release

1. Create a release PR in this repo, e.g: `main#release#minor` branch
2. Merge the above release PR.
3. Create a PR in `workload-clusters-fleet` repo to bump version in [App CR](https://github.com/giantswarm/workload-clusters-fleet/blob/main/management-clusters/gazelle/organizations/giantswarm-production/workload-clusters/operations/apps/faiq/appcr.yaml)
4. Merge the above PR - GitOps will rollout the app upgrade.

## Behind the scene

![](assets/diagram.jpg)
To edit the diagram, [click here](https://drive.google.com/file/d/1hNzmpAs0RbU0XnL4TOra5QbiX7QMyrtU/view?usp=sharing).

