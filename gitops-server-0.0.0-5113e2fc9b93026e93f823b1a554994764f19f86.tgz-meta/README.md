# GitOps Server Helm Chart

This Helm Chart provides a Weave GitOps Server setup based on the [upstream Helm Chart](https://github.com/weaveworks/weave-gitops/tree/main/charts/gitops-server). The server provides a graphical interface to the GitOps environments managed by Flux. Quoting the [official documentation](https://docs.gitops.weave.works/docs/0.18.0/intro/):

> The web UI surfaces key information to help application operators easily discover and resolve issues. The intuitive interface provides a guided experience to build understanding and simplify getting started for new users; they can easily discover the relationship between Flux objects and navigate to deeper levels of information as required.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/advanced/gitops/#installing-managed-apps)
- [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/app-platform/getting-started/).

## Configuring

This section gives various example of how to configure the `values.yaml`.

### Logging to the GitOps Server

There are 2 supported methods for logging into the dashboard:

* via an OIDC provider
* via a cluster user account

Quoting the [official documentation](https://docs.gitops.weave.works/docs/0.18.0/configuration/securing-access-to-the-dashboard/), the latter one is not recommended:

> The recommended method is to integrate with an OIDC provider, as this will let you control permissions for existing users and groups that have already been configured to use OIDC. However, it is also possible to use the Emergency Cluster User Account to login, if an OIDC provider is not available to use. Both methods work with standard Kubernetes RBAC.

By default, both methods are enabled in the GitOps Server, even if only one is actually used. This can be controlled by the `--auth-methods` parameter passed to the GitOps Server. For example, in order to use only the OIDC provider method, configure the
`values.yaml` accordingly:

```yaml
additionalArgs:
  - "--auth-methods=oidc"
```

On the other hand, to use only the cluster user account, configure the `values.yaml` this way:

```yaml
additionalArgs:
  - "--auth-methods=user-account"
```

Find the sample configuration for both methods in the next paragraphs.

#### Configuring Helm Chart for OIDC provider method

The part of `values.yaml` responsible for integrating the GitOps Server with the OIDC provider is presented below:

```yaml
oidcSecret:
  create: false
  # clientID:
  # clientSecret:
  # issuerURL:
  # redirectURL:
```

Meaning of the values can be found in the [official documentation](https://docs.gitops.weave.works/docs/0.18.0/configuration/oidc-access/#configuration). Setting on these values, and switching create flag from `false` to `true`, will instruct Helm Chart to create the necessary Kubernetes Secret with OIDC provider configuration. See an example below.

```yaml
oidcSecret:
  create: true
  clientID: fakeclient          # configured with the provider
  clientSecret: fakesecret      # configured with the provider
  issuerURL: https://dex.fake.cluster.domain
  redirectURL: https://gitops.fake.cluster.domain/oauth2/callback
```

Bear in mind, as shown in the upstream documentation, the Kubernetes Secret may also be created by hand, or by some other means, in which case the create flag should remain set to `false`. No further configuration of the Helm Chart would be required in such case, the Secret should get discovered by the GitOps Server automatically.

#### Configuring Helm Chart for cluster user account method

The part of `values.yaml` responsible for creating a local cluster user for the GitOps Server is presented below:

```yaml
adminUser:
  create: false
  createClusterRole: true
  createSecret: true
  username: gitops-test-user
  passwordHash:
```

Meaning of the values can be found in the [official documentation](https://docs.gitops.weave.works/docs/0.18.0/configuration/emergency-user/).

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the app to
workload cluster `abc12`:

```yaml
# appCR.yaml

```

```yaml
# user-values-configmap.yaml

```

See our [full reference on how to configure apps](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- _add limitation_

## Credit

- {APP HELM REPOSITORY}
