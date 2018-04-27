---
title: Getting started
weight: 10
---

## Installation

### Installing Jetprobe

Jetprobe is a validation framework for building tests for big data pipelines. To install Jetprobe just download the [latest version](https://github.com/jetprobe/jetprobe/releases) .

Once downloaded, run the following commands to configure jetprobe to run from the command line.

### On Linux
```sh
$ tar -xvf /path/to/jetprobe-<version>.tgz
$ cd jetprobe-<version>
$ make install
$ export PATH=$PATH:~/local/bin
```
This would add the jetprobe located at  `/home/<user>/local/bin/jetprobe` to the `PATH` variable.

### On Windows
Assuming that java is installed and available in the `PATH` use the below commands to add the `jetprobe` executable to the current `PATH`.
```sh
C:\Users\Me>jar xf jetprobe-<version>.zip
C:\Users\Me>SET PATH=%PATH%;%cd%\jetprobe-0.2.0-SNAPSHOT\bin
```

## Verify the Installation
After installing JetProbe, verify the installation worked by opening a new terminal session and checking that `jetprobe` is available. By executing `jetprobe` you should see error output similar to this:

```sh
$ jetprobe -v
jetprobe 0.2.0-SNAPSHOT
build : 9bd6cacff213741ffdeb87813e66418859675da0
```
For getting the list of supported options just type `jetprobe --help`. Here's the expected output.
```
Usage: jetprobe [options]

  -j, --jar <file>     Testable jar is required
  -c, --config <config>    Test config file path
  --help                   prints this usage text

```

## Create and Run tests

### Add the library

If you are using SBT, then add the following as the library dependencies. Currently the libraries are available for Scala 2.11 and 2.12.

```
//Currently the project is in early development. Expect to get few nasty errors.
resolvers +=
  "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"

libraryDependencies ++= Seq(
    "com.jetprobe" %% "jetprobe-core" % "0.2.0-SNAPSHOT",
    "com.jetprobe" %% "jetprobe-rabbitmq" % "0.2.0-SNAPSHOT",
    "com.jetprobe" %% "jetprobe-mongo" % "0.2.0-SNAPSHOT",
    "com.jetprobe" %% "jetprobe-hadoop" % "0.2.0-SNAPSHOT"
  )
```

If you are using maven as the build tool, then you can add the following dependencies

```xml
<!-- Add the snapshots repository -->
<repositories>
     <repository>
       <id>snapshots-repo</id>
       <url>https://oss.sonatype.org/content/repositories/snapshots</url>
       <releases><enabled>false</enabled></releases>
       <snapshots><enabled>true</enabled></snapshots>
     </repository>
</repositories>

<properties>
  <scala.version>2.12</scala.version>
<properties>

<dependency>
    <groupId>com.jetprobe</groupId>
    <artifactId>jetprobe-core_${scala.version}</artifactId>
    <version>0.2.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>com.jetprobe</groupId>
    <artifactId>jetprobe-rabbitmq_${scala.version}</artifactId>
    <version>0.2.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>com.jetprobe</groupId>
    <artifactId>jetprobe-mongo_${scala.version}</artifactId>
    <version>0.2.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>com.jetprobe</groupId>
    <artifactId>jetprobe-hadoop_${scala.version}</artifactId>
    <version>0.2.0-SNAPSHOT</version>
</dependency>
```
Once you have added the required libraries, start by creating a Scala class to define the scenario that needs to be validated. In this case, we are trying to validate the creation of exchange, once a http request is executed.

Currently all the tests need to be written in Scala, a language offering functional programming paradigm for JVM. Every test class unlike a Junit test must extend the TestScenario trait. A trait in Scala is similar to interfaces in Java, which consists of a single method `buildScenario`. This method needs to be overridden by the Custom test class. Here is an example of Hello Test Suite in JetProbe.

```Scala
import com.jetprobe.core.TestScenario
import com.jetprobe.core.annotation.TestSuite
import com.jetprobe.core.structure.ScenarioBuilder
import com.jetprobe.mongo.storage.MongoDBConf

@TestSuite("Hello Jetprobe")
class HelloJetProbe extends TestScenario {

  val getUserInfo = Http("Create Foo ")
                    .post("http://api.server.com/v1/foo/1")
                    .body("{ 'name' : 'foo'}")

  val mongoConf = new MongoDBConf("mongodb://xxx.xxx.xx.xx")

  override def tasks: PipelineBuilder = {

    http(getUserInfo)

    //Validate the properties of Foo database
    validate("check database",mongoConf){ mongo =>

      given(mongo.getDatabaseStats("foo")){ dbStats =>
        assertEquals(2,dbStats.get("indexes").get.asInt32().getValue)
      }
    }

  }
}
```

The above test once executed will just pause the test pipeline for 5 seconds and then end the test.
Once done, create a packaged jar and use the jetprobe CLI to run the test jar as below.
`jetprobe --jar /path/to/packaged-test.jar`

Currently the framework supports testing of RabbitMQ, MongoDB, HDFS and HBase component properties. Head over to the next [section](../writing-validations) to know more about writing tests.
