[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/proxmox-csi-plugin-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/proxmox-csi-plugin-app/tree/main)

# proxmox-csi-plugin-app

This repository holds the Giant Swarm App for the Proxmox CSI plugin.
The plugin is a Kubernetes CSI driver. It gives Proxmox VE block storage to workload clusters.

## Source

The chart in `helm/proxmox-csi-plugin` comes from upstream:

- Upstream project: <https://github.com/sergelogvinov/proxmox-csi-plugin>
- Upstream chart: `charts/proxmox-csi-plugin`

`vendir.yml` pins the upstream version. `vendir sync` copies the chart into `vendor/`, then into `helm/`.
The scripts in `sync/patches` apply the Giant Swarm changes on top.
Run `./sync/sync.sh` to do all these steps. See [sync/README.md](sync/README.md) for more information.

Do not edit the chart by hand. Change or add a patch in `sync/patches`, then run the sync script.

## Installation

Install this app on a workload cluster with the Giant Swarm App Platform:

- [Use GitOps to install the chart](https://docs.giantswarm.io/tutorials/continuous-deployment/helm-releases/add-helmrelease/)

## Configuration

See [helm/proxmox-csi-plugin/README.md](helm/proxmox-csi-plugin/README.md) for all the values.
