# kubectl-apply-job library chart

This Helm library chart generates a Job which runs `kubectl apply` before the actual installation of
a chart. It is intended for installing CRDs that a chart's own templates then depend on.

The Job and its supporting resources are rendered as Helm `pre-install,pre-upgrade` hooks, weighted
so they are created in dependency order.

## Usage

- Specify this library chart as a dependency in your `Chart.yaml`

      dependencies:
      - name: kubectl-apply-job
        version: "x.y.z" # use the latest release version here
        repository: oci://gsoci.azurecr.io/charts/giantswarm

- Run `helm dep update`
- Add at least these additional values to your `values.yaml`

      kubectlApplyJob:
        enabled: true
        files:
        - path/to/file-1.yaml
        - path/to/file-2.yaml

- Somewhere in your templates, add

      {{ include "kubectlApplyJob.job" . }}

Each entry in `files` is read from your chart with `.Files.Get`, rendered through `tpl`, and mounted
into the Job as its own ConfigMap.

## What it can apply

**The bundled ClusterRole grants permissions on CustomResourceDefinitions only.** Applying anything
else — a Deployment, a ConfigMap, a Job — fails with an RBAC error and is retried up to
`backoffLimit` times. If you need to apply other kinds, grant the extra permissions yourself: bind
your own ClusterRole to the ServiceAccount this chart creates, which is named
`<your chart name>-<jobNameSuffix>`.

The ClusterRole and ClusterRoleBinding are **cluster-scoped** and named after your chart, so two
releases of the same chart in different namespaces will collide on them.

## Values

All values live under the `kubectlApplyJob` key, except `ciliumNetworkPolicy`.

| Value | Default | Description |
|---|---|---|
| `enabled` | `true` | Render nothing when false. |
| `files` | `[]` | Paths, relative to your chart, of the manifests to apply. |
| `jobNameSuffix` | `kubectl-apply-job` | Suffix for the names of the Job, ServiceAccount, ClusterRole and ClusterRoleBinding. |
| `name` | `kubectl-apply-job` | Value of the `app.kubernetes.io/component` label. |
| `image.registry` | `gsoci.azurecr.io` | Container image registry. |
| `image.repository` | `giantswarm/docker-kubectl` | Container image repository. |
| `image.tag` | see `values.yaml` | Container image tag. |
| `image.pullPolicy` | `IfNotPresent` | Container image pull policy. |
| `backoffLimit` | `10` | Job retries. Higher means the Job is less likely to be marked failed. |
| `resources` | see `values.yaml` | Requests and limits for the kubectl container. |
| `securityContext.runAsUser` | `1000` | UID for the pod and container. |
| `securityContext.runAsGroup` | `1000` | GID for the pod and container. |
| `securityContext.seccompProfileType` | `RuntimeDefault` | Seccomp profile type. |
| `ciliumNetworkPolicy.enabled` | `false` | Also render a CiliumNetworkPolicy alongside the NetworkPolicy. |

## Development

`./test/verify.sh` renders the library through `test/consumer` and asserts the resulting document
counts. It requires `helm` and `yq`. This guards against the document-separator regressions that
have bitten this chart before: when two manifests merge into one document, a resource silently
disappears.
