[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/pyroscope-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/pyroscope-app/tree/main)

# Pyroscope chart

Giant Swarm offers a pyroscope App which can be installed in management clusters.
Here we define the pyroscope chart with its templates and default configuration.

**What is this app?**

Pyroscope is an open-source continuous profiling platform integrated to `grafana`, allowing its users to have a deeper understanding of their workloads's resources usage. It thus allow one to optimize one's resources consumption and can be useful in a cost-saving approach.

When used at its fullest, Pyroscope can be integrated with other observability signals such as metrics, logs and traces to have a complete view of a cluster's behavior.

**Why did we add it?**

This app is currently not a default one, meaning that it won't be deployed automatically on our management clusters. However, we wanted to provide our users with a continuous profiling tool that could help them manage their resources usage.

## Installing

In order to install pyroscope onto a management cluster, you will have to create an [App resource](https://docs.giantswarm.io/use-the-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/getting-started/app-platform/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the app in a vintage management cluster

```yaml
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: pyroscope
  namespace: giantswarm
spec:
  catalog: giantswarm-playground-test
  kubeConfig:
    inCluster: true
  userConfig:
    configMap:
      name: pyroscope-user-values
      namespace: giantswarm
  name: pyroscope
  namespace: pyroscope
  version: 0.1.0
```

The `pyroscope-user-values` configmap is the one that you can create from the values file's example as shown in the previous section.

See our [full reference on how to configure apps](https://docs.giantswarm.io/getting-started/app-platform/app-configuration/) for more details.

## Limitations

This app is still a work in progress.

## Credit

- https://github.com/grafana/pyroscope
- https://grafana.com/docs/pyroscope/latest/
