# kubectl-apply-job library chart

This helm charty can be used to easily generate a Job which executes `kubectl apply` before the actual installation of a chart

## Usage

- Specify this library chart as a dependency in your `Chart.yaml`

      dependencies:
      - name: kubectl-apply-job
        version: "0.9.0" # use the latest release version here
        repository: oci://giantswarmpublic.azurecr.io/giantswarm-playground-catalog

- Run `helm dep update`
- Add at least these additional values to your `values.yaml`

      kubectlApplyJob:
        enabled: true
        files:
        - path/to/file-1.yaml
        - path/to/file-2.yaml

- Somewhere in your templates, add

      {{ include "kubectlApplyJob.job" . }}
