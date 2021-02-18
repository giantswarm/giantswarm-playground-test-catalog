[![CircleCI](https://circleci.com/gh/giantswarm/keda-app.svg?style=shield)](https://circleci.com/gh/giantswarm/keda-app)

# KEDA (Kubernetes Event-driven Autoscaling) chart

Giant Swarm offers a KEDA App which can be installed in tenant clusters.
Here we define the KEDA chart with its templates and default configuration.

## Installing

There are 3 ways to install this app onto a tenant cluster.

1. [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app)
2. [Using our API](https://docs.giantswarm.io/api/#operation/createClusterAppV5)
3. Directly creating the [App custom resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) on the management cluster.

## Configuring

Configuration values are documented in [`helm/keda/README.md`](https://github.com/giantswarm/keda-app/blob/master/helm/keda/README.md).

See our [full reference page on how to configure applications](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Compatibility

Keda requires Kubernetes >=1.16.0

## Credit

* [https://github.com/kedacore/charts](https://github.com/kedacore/charts)
