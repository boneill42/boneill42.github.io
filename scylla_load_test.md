# Load Testing a Go + Scylla stack with Gatling in 

In this article, we'll setup Gatling for scale testing.  We'll perform a local scale test, and then we'll deploy that scale test to the cloud using [testable.io](http://testable.io).

## Gatling Setup

We use [gatling](https://gatling.io/) to load test our services.  If you've never used Gatling, you probably need to [install it](https://gatling.io/docs/current/installation/#installation).  For me, I simply downloaded the bundle, unzip'd it and made sure the bin was on my path.

You can test your installation with the out-of-the-box simulations by following the command-line prompts:

``` bash
$ gatling.sh
GATLING_HOME is set to /Users/boneill/tools/gatling
Java HotSpot(TM) 64-Bit Server VM warning: Option AggressiveOpts was deprecated in version 11.0 and will likely be removed in a future release.
Choose a simulation number:
     [0] computerdatabase.BasicSimulation
...
```

After the run completes, it will point you to the reports, which you can then open directly with:

``` bash
...
Please open the following file: /Users/boneill/tools/gatling/results/fooscaletest-20190514135249487/index.html
$ open /Users/boneill/tools/gatling/results/fooscaletest-20190514135249487/index.html
```

This will open a browser with the results of the test.

In the example run above, Gatling presented you with a list of simulations that come with the bundle.  By default, it scans the `user-files/simulations/computerdatabase` directory for simulations and presents those.  See below:

```
$GATLING_HOME/user-files/simulations/computerdatabase-> tree
.
├── BasicSimulation.scala
└── advanced
    ├── AdvancedSimulationStep01.scala
    ├── AdvancedSimulationStep02.scala
    ├── AdvancedSimulationStep03.scala
    ├── AdvancedSimulationStep04.scala
    └── AdvancedSimulationStep05.scala
```

To add your simulation, you can simply drop a new class in that directory.  For our local test, we'll point Gatling to a local folder instead using the `-sf` parameter.

## Local Scale Test

For our scale test, we fronted Scylla with a service written in Go that does a simple query/write to Scylla.   The following Scala class lets us test roundtrip performance of that service:

``` Scala
package mypackage
import io.gatling.core.Predef._
import io.gatling.core.structure.ScenarioBuilder
import io.gatling.http.Predef._
import io.gatling.http.protocol.HttpProtocolBuilder
import scala.concurrent.duration._

class ScaleTest extends Simulation {
  val httpProtocol = http.baseUrl("https://skookle.com")

  val idlinkTest: ScenarioBuilder = scenario("Massive Scale Test")
    .exec(http("Foo Service")
      .get("/foo?provider=zed&id=42")
      .check(status.is(200))
      .check(bodyBytes.exists)
      .check(jsonPath("$.data.fooId").exists)
    )

  setUp(idlinkTest.inject(constantUsersPerSec(300) during (30 minutes))).throttle(
    reachRps(300) in (10 seconds),
    holdFor(1 minute),
  ).protocols(httpProtocol)

}
```

Assuming this class is in your current directory (e.g. `ScaleTest.scala`), you can tell Gatling to scan the current directory for Simulations with the following command:

``` bash
gatling.sh -sf .
```

That should generate numbers for you.  On a toy cluster, we saw the following:

``` bash
================================================================================
---- Global Information --------------------------------------------------------
> request count                                       6538 (OK=6538   KO=0     )
> min response time                                     81 (OK=81     KO=-     )
> max response time                                   4574 (OK=4574   KO=-     )
> mean response time                                   233 (OK=233    KO=-     )
> std deviation                                        552 (OK=552    KO=-     )
> response time 50th percentile                        110 (OK=110    KO=-     )
> response time 75th percentile                        115 (OK=115    KO=-     )
> response time 95th percentile                        906 (OK=906    KO=-     )
> response time 99th percentile                       3400 (OK=3400   KO=-     )
> mean requests/sec                                   93.4 (OK=93.4   KO=-     )
---- Response Time Distribution ------------------------------------------------
> t < 800 ms                                          6200 ( 95%)
> 800 ms < t < 1200 ms                                  41 (  1%)
> t > 1200 ms                                          297 (  5%)
> failed                                                 0 (  0%)
================================================================================
```

## Scale Testing in the Cloud

Now, it's time to move this test to the cloud.  We use [testable.io](http://testable.io) for scale testing in the cloud.  They provide a number of different means of running scaled tests against services, one of which is Gatling.

If you've got a blank slate in _testable.io_, then use the New Test Wizard to define the test.  When asked for Simulation, choose "Gatling Simulation", Source: "Upload Files",  Gatling Version: "3.0.2", and Simulation Class Names: "mypackage.ScaleTest".  (NOTE: This is not the default Galing version, and we had to include the package name in the Class name.)  Then, upload the scala file.

Under Configuration, choose a single Gatling instance.  For Location, pick your favorite AWS region and select "AWS On Demand" as the source.  Then hit, "Start Test" to launch.  The test may fail because its exceeds the number of concurrent users allowed in the account.  We have it hard-coded to 100.  So, it ignored any setting you had in testable.

To fix this, let's update the test to accept parameters from testable.io.  Specifically, we'll use `Integer.getInteger("users", 100)` to get the number of concurrent users.  And more generally, we can get any parameter passed in (e.g. `durationMinutes`).

``` Scala
val numUsers = Integer.getInteger("users", 100).toInt
  val numRps = numUsers * 1
  val durationMinutes = Integer.getInteger("durationMinutes", 1).toInt

  setUp(idlinkTest.inject(constantUsersPerSec(numUsers) during (durationMinutes minutes))).throttle(
    reachRps(numRps) in (10 seconds),
    holdFor(durationMinutes minute),
  ).protocols(httpProtocol)
```

Rerunning the test should work like a charm, and now you are scale testing in the cloud!