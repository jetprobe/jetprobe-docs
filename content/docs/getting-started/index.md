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
C:\Users\Me>SET PATH=%PATH%;%cd%\jetprobe-0.1.0-SNAPSHOT\bin
```

## Verify the Installation
After installing JetProbe, verify the installation worked by opening a new terminal session and checking that `jetprobe` is available. By executing `jetprobe` you should see error output similar to this:

```sh
$ jetprobe -v
jetprobe 0.1.0-SNAPSHOT
build : 86f0f6fcf17f832214a1ff38017f6ed5c9db67e0
```
For getting the list of supported options just type `jetprobe --help`. Here's the expected output.
```
Usage: jetprobe [options]

  -t, --testjar <file>     Testable jar is required
  -c, --config <config>    Test config file path
  -r, --reportPath <path>  Report output file path
  --help                   prints this usage text

```

## Create and Run tests

Currently all the tests need to be written in Scala, a language offering functional programming paradigm for JVM. Every test class unlike a Junit test must extend the TestScenario trait. A trait in Scala is similar to interfaces in Java, which consists of a single method `buildScenario`. This method needs to be overridden by the Custom test class. Here is an example of the most basic test Scenario.

```Scala
import com.jetprobe.core.TestScenario
import com.jetprobe.core.structure.ExecutableScenario
import scala.concurrent.duration._

class MyFirstTest extends TestScenario {

  override def buildScenario: ExecutableScenario = {

    scenario("FirstTest").pause(5.seconds)

  }
}
```

The above test once executed will just pause the test pipeline for 5 seconds and then end the test.
Once done, create a packaged jar and use the jetprobe CLI to run the test jar as below.
`jetprobe -testjar /path/to/packaged-test.jar`

Currently the framework supports testing of RabbitMQ and MongoDB component properties. Head over to the next [section](../writing-validations) to know more about writing tests.
