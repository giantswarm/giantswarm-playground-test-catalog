[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/node-problem-detector-app/tree/master.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/node-problem-detector-app/tree/master)

# node-problem-detector-app

This Giant Swarm managed app allows deploying [node-problem-detector](https://github.com/kubernetes/node-problem-detector)
with a custom configuration that adds monitoring of disk space.

Starting with legacy release 19.3.0, this allows to automatically terminate nodes when one of the key disks is full.
