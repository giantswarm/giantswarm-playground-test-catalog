[![CircleCI](https://circleci.com/gh/giantswarm/gatling-app.svg?style=shield)](https://circleci.com/gh/giantswarm/gatling-app)

# gatling-app chart

Giant Swarm offers a Gatling Managed App which can be installed in workload clusters.
Here we define the Gatling chart with its templates and default configuration.

## Required values

In order to run Gatling, it must be provided with a simulation file which defines the
test(s) to run. This configuration can be provided either as a `ConfigMap`, or as a URL
to a simulation hosted somewhere accessible by the cluster. If the URL method is used
then an init container will download the simulation file before Gatling runs. These
values must be provided via the `values.yaml` file.

Notes:
 - If you are using the `ConfigMap` method, then it should be created _before_ the app
is deployed.
 - If you provide a value for `simulation.url` then any `ConfigMap` is ignored.

### `ConfigMap` method

Below is a sample `ConfigMap` which is used to test the performance of an ingress controller.

Notes:
 - The `ConfigMap` name must be provided via the `simulation.configmap` key in
`values.yaml` (`simulation-configmap` in the example below).
 - The `simulation.file` key (`IngressSimulation.scala` in the example below) must be a valid
file name for a simulation file.
 - The `simulation.class` key (`ingress.IngressSimulation` in the example below) must be a valid
class name for a simulation class.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: simulation-configmap
  labels:
    app: gatling
data:
  IngressSimulation.scala: |
    package ingress

    import io.gatling.core.Predef._
    import io.gatling.http.Predef._
    import scala.concurrent.duration._

    class IngressSimulation extends Simulation {

      val httpProtocol = http
        .baseUrl("http://ingress-nginx-controller.kube-system.svc.cluster.local")
        .header("Host", "loadtest.local")
        .acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
        .doNotTrackHeader("1")
        .acceptLanguageHeader("en-US,en;q=0.5")
        .acceptEncodingHeader("gzip, deflate")
        .userAgentHeader("Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:16.0) Gecko/20100101 Firefox/16.0")

      val scn = scenario("basic ingress test")
        .repeat(100, "n") {
          exec(
            http("request_1").get("/")
          )
        }

      setUp(
        scn.inject(
          atOnceUsers(1),
          rampUsers(3) during (2 seconds)
        ).protocols(httpProtocol)
      ).maxDuration(5 minutes)
    }
```
