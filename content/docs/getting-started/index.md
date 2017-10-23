---
title: Getting started
weight: 10
---

## Installation

### Installing Jetprobe

Jetprobe is a validation framework for building tests for big data pipelines. To install Jetprobe just download the [latest version](https://github.com/jetprobe/jetprobe/releases) and save in a directory say `/home/<user>/apps/jetprobe-<version>.tgz`

Once downloaded, run the following commands to configure jetprobe to run from the command line.
```sh
$ tar -xvf jetprobe-<version>.tgz
$ cd jetprobe-<version>
$ make install
```
This would setup the Jetprobe at the location `/home/<user>/local/bin/jetprobe`
Add the above installation path to the current `$PATH` variable to run the jetprobe command from any location in terminal.

## Verify the Installation
After installing JetProbe, verify the installation worked by opening a new terminal session and checking that `jetprobe` is available. By executing `jetprobe` you should see error output similar to this:

```sh
$ jetprobe
Error: Missing option --testjar
jetprobe 0.1
Usage: jetprobe [options]

  -t, --testjar <file>     Test jar is a required file property
  -c, --config <config>    Test config file path
2017-10-22 14:18:03 INFO  TestRunner$:30 - Unable to parse the arguments
```
## Create tests

All the tests need to be written in Scala, a language offering functional programming paradigm for JVM. Every test class unlike a Junit test must extend the TestScenario trait. A trait in Scala is similar to interfaces in Java, which consists of a single method `buildScenario`. This method needs to be overridden by the Custom test class. Here is an example of the most basic test Scenario.

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
