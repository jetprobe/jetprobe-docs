---
date: 2016-03-09T20:08:11+01:00
title: Writing validations
weight: 30
---

## Scenario Definition

Validations are part of a scenario, which basically consists of set of actions that leads to state of the component, which needs to be validated. Let's look at a simple example.

### Add the library

If you are using SBT, then add the following as the library dependencies. Currently the libraries are available for Scala 2.11 and 2.12.

```
//Currently the project is in early development. Expect to get few nasty errors.
resolvers +=
  "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"

libraryDependencies ++= Seq(
      "com.jetprobe" %% "jetprobe-core" % "0.1.0-SNAPSHOT",
    "com.jetprobe" %% "jetprobe-rabbitmq" % "0.1.0-SNAPSHOT",
    "com.jetprobe" %% "jetprobe-mongo" % "0.1.0-SNAPSHOT"
  )
```
Once you have added the required libraries, start by creating a Scala class to define the scenario that needs to be validated. In this case, we are trying to validate the creation of exchange, once a http request is executed.

```scala
class Sample Scenario extends TestScenario {

// declare the RabbitMQ sink, that would be our target for validation
 val rabbit = RabbitMQSink("${rabbit.host}")

// the http request details
val createRequest = Http("Initialize RabbitMQ")
                .post("https://${http.host}/api/v1/create")
                .body(
                  fromFile("/path/to/post_req.json")
                  )

override def buildScenario: ExecutableScenario = {
                    //Give a name for the scenario
                     scenario("RabbitMQ Init Test")
                       .http(createRequest)
                       //Pause for sometime, to give enough time for creation of exchanges
                       .pause(2.seconds)
                       //Validate the properties of the exchange created post the request.
                       .validate[RabbitMQSink](rabbit) { rabbit =>

                        rabbit.forExchange(exchange = "web.events",vHost = "/")(
                          checkExchange[String]("fanout",exchangeProps => exchangeProps.exchangeType),
                          checkExchange[Int](1, _.bindings.size),
                          )

                       }
                       .build
                     }

}
```
## Execution of Scenario

Once the scenario is defined, the validation rules expressed would get executed by the Jetprobe library and at the end the results would be available both in the console and
the HTML file.

Follow the steps to run the scenario :

* Package the class file into a jar. If you are using sbt to build the project then just use `sbt package` to build the jar.
* Create a yaml configuration file say `config.yml` that would contain the properties that are being defined in the scenario definition.

```yml
//Example config
rabbit.host : 10.25.3.56
http.host : api.myserver.com
```
and save it at say `/data/path/config.yml`. Use the jar to run the validations as below.

  ```sh
  $ jetprobe --testjar /path/to/validations.jar --config /data/path/config.yml --reportPath /export/path/for/report.html
  ```

For more examples, head over to the [example project](https://github.com/jetprobe/jetprobe/tree/master/jetprobe-sample/src/main/scala/com/jetprobe/sample)
