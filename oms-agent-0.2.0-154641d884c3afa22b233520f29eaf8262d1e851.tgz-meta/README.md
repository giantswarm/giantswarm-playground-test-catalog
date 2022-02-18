[![CircleCI](https://circleci.com/gh/giantswarm/oms-agent-app.svg?style=shield)](https://circleci.com/gh/giantswarm/oms-agent-app)

# OMS-Agent chart

Giant Swarm offers a OMS-Agent App which can be installed in Azure workload clusters.
Here we define the OMS-Agent chart with its templates and default configuration.

**What is this app?**
This app installs a daemonset of [Operatioms Management Suite Agent](https://docs.microsoft.com/en-us/azure/azure-monitor/agents/agent-linux) from Microsoft for Azure Log Analytics.
This agent collects the telemetry of the running Virtual Machines. 
**Why did we add it?**
This app has been added to accomodate our Customers running Log Anatylics to fully use the potential of monitoring services provided by Azure.
**Who can use it?**
It can be used by Giant Swarm Azure customers running Log Analytics.

## Installing

There are 3 ways to install this app onto a workload cluster.

1. [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app)
2. [Using our API](https://docs.giantswarm.io/api/#operation/createClusterAppV5)
3. Directly creating the [App custom resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) on the management cluster.

## Configuring

### values.yaml
**This is an example of a values file you could upload using our web interface.**
```
name: oms-agent
namespace: oms-agent

omsagent:
  image:
    repository: quay.io/giantswarm/ciprod
    tag: ciprod06112021
    pullPolicy: IfNotPresent

  secret:
    wsid: Azure_Log_Analytics_Workspace_ID
    key: Azure_Log_Analytics_Primary_Key
  domain: opinsights.azure.com
  env:
    clusterName: cluster_id_or_name

```

### Sample App CR and ConfigMap for the management cluster
If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the app to
workload cluster `abc12`:

```
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: oms-agent
  namespace: abc12
spec:
  catalog: giantswarm-playground
  kubeConfig:
    inCluster: false
  name: oms-agent
  namespace: oms-agent
  version: 0.0.1
  userConfig:
    configMap:
      name: oms-agent-user-values
      namespace: abc12
    secret:
      name: oms-agent-user-secrets
      namespace: abc12
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: oms-agent-user-values
  namespace: abc12
data:
  values: |
    env:
      clusterName: w5fc9 

```

```
apiVersion: v1
kind: Secret
metadata:
  name: oms-agent-user-secrets
  namespace: abc12
data:
  omsagent:
    secret:
      wsid: Azure_Log_Analytics_Workspace_ID
      key: Azure_Log_Analytics_Primary_Key
```

See our [full reference page on how to configure applications](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

* Azure v15.0.1

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

* Log Analytics Workspace ID and Key is a requirement.

## Credit

* https://github.com/microsoft/Docker-Provider/tree/ci_prod/charts/azuremonitor-container
