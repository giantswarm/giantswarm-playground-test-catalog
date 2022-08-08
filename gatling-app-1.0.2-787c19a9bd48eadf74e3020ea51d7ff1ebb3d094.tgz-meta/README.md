[![CircleCI](https://circleci.com/gh/giantswarm/gatling-app.svg?style=shield)](https://circleci.com/gh/giantswarm/gatling-app)

# gatling-app chart

Giant Swarm offers a Gatling Managed App which can be installed in tenant clusters.
Here we define the Gatling chart with its templates and default configuration.

## Required values

In order to run Gatling, it must be provided with a simulation file which defines the
test(s) to run. This configuration can be provided either as a `configMap`, or as a URL
to a simulation hosted somewhere accessible by the cluster. If the URL method is used
then an initContainer will download the simulation file before Gatling runs. These
values must be provided via the `values.yaml` file.

Notes:

 - If you are using the `configMap` method, then it should be created _before_ the app
is deployed.
 - If you provide a value for `simulation.url` then any configMap is ignored.

### `configMap` method

Below is a sample configMap which is used to test the performance of an ingress controller.

Notes:
 - The `configMap` name must be provided via the `simulation.configMap.name` key in
`values.yaml` (`simulation-configmap` in the example below).
 - The simulation file key (`NginxSimulation.scala` in the example below) must be a valid
file name for a simulation Class.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: simulation-configmap
  labels:
    app: gatling
data:
  NginxSimulation.scala: |
    package nginx

    import io.gatling.core.Predef._
    import io.gatling.http.Predef._
    import scala.concurrent.duration._

    class NginxSimulation extends Simulation {

      val httpProtocol = http
        .baseUrl("http://nginx-ingress-controller.kube-system.svc.cluster.local")
        .header("Host", "loadtest.local")
        .acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
        .doNotTrackHeader("1")
        .acceptLanguageHeader("en-US,en;q=0.5")
        .acceptEncodingHeader("gzip, deflate")
        .userAgentHeader("Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:16.0) Gecko/20100101 Firefox/16.0")

      val scn = scenario("Nginx basic test")
        .repeat(100, "n") {
          exec(
            http("request_1")
	      .get("/")
            )
        }

      setUp(
        scn.inject(
          atOnceUsers(1),
          rampUsers(3) during (2 seconds)
        ).protocols(httpProtocol))
        .maxDuration(5 minutes)
    }
```
