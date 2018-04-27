---
date: 2016-03-09T19:56:50+01:00
title: Introduction
weight: 20
---

## What is Jetprobe ?

Jetprobe is an open source framework to build and run integration test for big data pipelines. It allows you to invoke http requests, ingest random test data and pause for the data pipeline to finish before running the validations.

The framework allows to validate both the attributes and the data stored in the infrastructure component. An infrastructure component can be a database, a messaging queue, a distributed processing engine(Spark, Flink ), a search engine (Solr, Elastic search) or a microservice.

## How does it help ?

*  Write integration test with expressing DSL for the big data pipeline.
*  Reduces the development time by identifying the bugs before moving to production.
*  Enables the QA developers to write comprehensive test cases.
*  Automate the execution of tests by integrating with the CI Pipeline.

## Testable components

The following components are currently supported by the Jetprobe framework.

* REST APIs with JSON response
* MongoDB
* RabbitMQ
* HBase
* HDFS

## Terminologies

Now we will look at the common terminologies used in the context of defining and executing the JetProbe test pipeline.

**Pipeline :** Every test pipeline could be part of a scenario, which consists of series of steps to be executed one after the another.

**Tasks :** Every step in the test pipeline is a task. For example connecting to a microservice through HTTP protocol is an action, named `http`. Similarly pausing for some duration is also an action.

**Task Builder :** Each action would take an ActionBuilder object as a parameter, that is used at the runtime to execute the action.

**Actors :**  JetProbe leverages actor model for concurrency and therefore each action in the pipeline runs on an actor. This allows multiple test pipeline to be executed concurrently without polluting the test results.
