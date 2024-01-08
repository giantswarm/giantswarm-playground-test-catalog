[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/pyroscope-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/pyroscope-app/tree/main)

# Pyroscope chart

Giant Swarm offers a pyroscope App which can be installed in management clusters.
Here we define the pyroscope chart with its templates and default configuration.

**What is this app?**

Pyroscope is an open-source continuous profiling platform integrated to `grafana`, allowing its users to have a deeper understanding of their workloads's resources usage. It thus allows users to optimize one's resources consumption and can be useful in a cost-saving approach.

When used at its fullest, Pyroscope can be integrated with other observability signals such as metrics, logs and traces to have a complete view of a cluster's behavior.

**Why did we add it?**

This app is currently not a default one, meaning that it won't be deployed automatically on our management clusters. However, we wanted to provide our users with a continuous profiling tool that could help them manage their resources usage.

## Installing

In order to install pyroscope onto a management cluster, you will have to create an [App resource](https://docs.giantswarm.io/use-the-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/getting-started/app-platform/).

You will also have to create an object storage, either by enabling `MinIO` in the values file (only working on vintage clusters) or by providing an access to an existing object storage solution you own.
This can be done by creating a `Bucket` CR when the cluster is runing the [object-storage-operator](https://github.com/giantswarm/object-storage-operator).

## Configuring

### values.yaml

This is an example of a values file for a CAPZ cluster (namely : `glippy`) you could upload using our web interface:

```yaml
psp:
  enabled: true
ingress:
  enabled: true
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-giantswarm
    giantswarm.io/external-dns: managed
  className: nginx
  hosts:
    - pyroscope.glippy.azuretest.gigantic.io
  tls:
    - hosts:
        - pyroscope.glippy.azuretest.gigantic.io
      secretName: pyroscope-ingress-cert
pyroscope:
  pyroscope:
    config: |
      storage:
        backend: azure
        azure:
          account_name: "<azure_storage_account_name>"
          account_key: "<azure_storage_account_key>"
          container_name: "giantswarm-glippy-pyroscope"
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

Note that if you want to deploy the app on a management cluster you will need to add the `app-operator.giantswarm.io/version: 0.0.0` label to the App CR's metadata so that the `app-operator` picks up and deploys the app.

The `pyroscope-user-values` configmap is the one that you can create from the values file's example as shown in the previous section.

See our [full reference on how to configure apps](https://docs.giantswarm.io/getting-started/app-platform/app-configuration/) for more details.

## Shipping profiles

This app deploys `pyroscope` on a cluster but pyroscope itself won't scrape profiles on its own. To achieve this, you will need to deploy an instance of the [grafana-agent-app](https://github.com/giantswarm/grafana-agent-app) (you can name the app `pyroscope-agent` not to mistake it with another `grafana-agent` that might be running on the cluster) with the correct config.

## Viewing profiles

You can visualize profiles as a flame graph from the Grafana's explore panel with `Pyroscope` as a data source.

## Limitations

Due to upstream limitations, `Pyroscope` cannot be run with `MinIO` as an object storage solution on CAPI clusters.

## Credit

- https://github.com/grafana/pyroscope
- https://grafana.com/docs/pyroscope/latest/
