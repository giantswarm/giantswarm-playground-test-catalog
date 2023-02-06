[![CircleCI](https://circleci.com/gh/giantswarm/linkerd-bundle.svg?style=shield)](https://circleci.com/gh/giantswarm/linkerd-bundle)

# linkerd-bundle chart

The linkerd bundle provides the necessary components to enable service mesh capabilities in a workload cluster.

**Apps**

- [linkerd2-cni-app](https://github.com/giantswarm/linkerd2-cni-app)
- [linkerd-control-plane-app](https://github.com/giantswarm/linkerd-control-plane-app)
- [linkerd-multicluster-app (optional)](https://github.com/giantswarm/linkerd-multicluster-app)
- [linkerd-multicluster-link-app (optional)](https://github.com/giantswarm/linkerd-multicluster-link-app)
- [linkerd-viz-app](https://github.com/giantswarm/linkerd-viz-app)

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/advanced/gitops/#installing-managed-apps)
- [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/app-platform/getting-started/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the bundle to
workload cluster `abc12`:

```yaml
# user-values-configmap.yaml

apiVersion: v1
data:
  values: |
    clusterID: abc12
    organization: mati
    apps:
      linkerd2-cni:
        enabled: true
kind: ConfigMap
metadata:
  name: linkerd-bundle-user-values
  namespace: abc12
```

```yaml
# appCR.yaml

apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  labels:
    app.kubernetes.io/name: linkerd-bundle
    app-operator.giantswarm.io/version: 0.0.0
  name: linkerd-bundle-abc12
  namespace: abc12
spec:
  catalog: giantswarm-playground
  kubeConfig:
    inCluster: true
  name: linkerd-bundle
  namespace: abc12
  userConfig:
    configMap:
      name: linkerd-bundle-user-values
      namespace: abc12
  version: 0.0.0
```


See our [full reference on how to configure apps](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_

