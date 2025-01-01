[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/envoy-gateway-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/envoy-gateway-app/tree/main)

# Envoy Gateway chart

Giant Swarm offers an envoy-gateway app that can be installed in workload clusters. Here, we define the envoy-gateway chart using its templates and default configuration.

**What is this app?**

This app is a gateway that routes traffic to services in the workload cluster. It is fully compliant with the [Gateway API](https://gateway-api.sigs.k8s.io/). It deploys an Envoy proxy configured to listen to new `Gateway` and `HTTPRoute` resources and route traffic accordingly.

By default, the app has no `Gateway` or `GatewayClass` resources. We have created a [default configuration app](https://github.com/giantswarm/envoy-gateway-default-configuration) to offer you a quick start in Giant Swarm clusters.

**Why did we add it?**

Our security baseline is higher than usual so we need to add configuration, policies and network rules to make sure the app is secure and compliant with our standards. We extend it using `git` patches over the original app templates.

**Who can use it?**

It is ready to use for any platform team.

## Installing

Before installing this app, you need to configure the cert-manager application in your workload cluster to work with Gateway API. That way the cert manager will listen for Gateway API resources to create certificates when those are needed. Adapt the configmap `*****-user-values` to contain the following configuration:

```yaml
apiVersion: v1
data:
  values: |
    ...
    featureGates: "ExperimentalGatewayAPISupport=true"
kind: ConfigMap
metadata:
  name: <mylcutser>-cert-manager-user-values
  namespace: org-<your-org>
```

## Configuring

The default app configuration is ready to deploy an instance of envoy proxy on your cluster to act as gateway. It will be deployed in the `envoy-gateway-system` namespace. You may want to read the [configuration options](https://github.com/giantswarm/gateway-api-app/tree/main/helm/gateway-api) though and accommodate them to your needs.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- CAPI >= v29.0.0

## Limitations

Some of the features of the app are not yet stable and are subject to change. The app is not yet ready for production use.

## Development

Information about chart and version development can be found in [sync/README.md](https://github.com/giantswarm/envoy-gateway-app/blob/main/sync/README.md).

## Credit

- https://github.com/envoyproxy/gateway