[![CircleCI](https://circleci.com/gh/giantswarm/jaeger-operator-app.svg?style=shield)](https://circleci.com/gh/giantswarm/jaeger-operator-app)

# jaeger-operator

This chart simply installs [jaeger operator](https://github.com/jaegertracing/jaeger-operator).
It's based on the official [helm-chart](https://github.com/jaegertracing/helm-charts/tree/master/charts/jaeger-operator)

Jaeger operator can be used to deploy jaegers in different configurations by creating the "Jaeger" CR.

## Configuration

This charts [values.yaml](helm/jaeger-operator-app/values.yaml) contains a default CR config which can be overwritten by providing your own values.yaml when installing the app.
Examples on how jaeger instances can be configures can be found [here](https://github.com/jaegertracing/jaeger-operator/tree/master/deploy/examples)
For further documentation visit [the official docs](https://www.jaegertracing.io/docs/1.19/operator/)

Although it's currently not recommended, `jaeger-operator` can be configured with cluster wide access by supplying the following in a custom `values.yaml`:
```yaml
  rbac:
    clusterRole: true
```

## Limitations

- can only be deployed once per cluster
- deployed in "namespace" mode by default; "cluster" mode is supported, but not recommended because of the security implications (check `Chart.yaml` for more info!)

## Installation

The application doesn't need any dependencies nor config, so it's sufficient to run:

```text
helm install -n tracing giantswarm-playground-catalog/jaeger-operator-app
```

## Compatibility

Tested on Giant Swarm release 12.1.2 on AWS with Kubernetes 1.17.9.
