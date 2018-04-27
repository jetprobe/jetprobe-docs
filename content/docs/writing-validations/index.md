---
date: 2016-03-09T20:08:11+01:00
title: Writing validations
weight: 30
---

## Scenario Definition

Validations are special actions, designated to validate the state of an infrastructure component.

## HDFS Validations

In the previous section we saw how to execute actions on HDFS without verbosity existing in the respective Java APIs. In this section we will how to write validations for the HDFS Storage.

```scala
validate(description = "Check file status ",hdfsConf){ hdfs =>

      //Fetch the property that needs to be validated
      val fileStatus = hdfs.usingFS(fs => fs.getFileStatus(new Path("/user/hdfs/data.in")))

      //Use the given function as the behavior driven test constructor
      given(fileStatus) { status =>
        assertEquals(10285689L,status.getBlockSize)
      }
    }
```
Every `validate` function, acts as validation task builder, which takes the Storage configuration as a parameter and expects a function that would build a validation rule with the returned Storage instance.
```scala
def validate[S <: Storage](config: Config[S])(fnRuleBuilder : S => ValidationRule[S])
```

**RabbitMQ Validations**
```Scala
//create the RabbitMQ instance that needs to be validated. The host here ${rabbit.host} would be resolved at the runtime.
val rabbit = new RabbitMQConfig("${rabbit.host}")

validate(description = "Check rabbitmq exchanges", rabbitConf) { rabbit =>

      val listofExchanges = rabbit.usingAdmin(admin => admin.getExchanges)

      given(listofExchanges) { exchanges =>
        val exchangNames = exchanges.asScala.map(_.getName)

        assertEquals(true, exchangNames.contains("amq.topic"))
        assertEquals("amq.headers", exchangNames.find(_.contains("headers")).get)

      }
    }
```

The validation rule builder `given` method takes the state of the object that needs to be validated and the state is created at the runtime by the JetProbe. This way the framework frees the developers to write the boilerplate code pertaining to connecting to RabbitMQ and fetching the
required details, instead the developer would just need to extract the required value for validating the state of the component.

## MongoDB Validations

For validating the mongodb components like database, collection and the records, there are some utility methods allowing the developers to quickly write the validation rules
given the state of the component as a POJO/case class(approx).

```scala

// Declare the instance of MongoDB Sink that needs to be validated
 val mongo = new MongoDBConf("mongodb://${mongo.host}/")

 validate(description = "check database stats",mongoConf){ mongo =>

      given(mongo.getDatabaseStats("zoo")){ dbStats =>

        assertEquals(2,dbStats.get("indexes").get.asInt32().getValue)
      }
    }
```

## HTTP validations

A common use case is to validate the HTTP requests for response code and content. Currently the framework supports json based response validation.
First let's define a http request.

```scala
val getPosts = Http("get-comments")
    .get("https://${server.hostname}/comments/post/1")
```
Let's assume that the response is as below.
```json
{
    "_reqId": "5a1b2cea4f066c709e376366",
    "page": 0,
    "commentsCount": 400,
    "comments": [
      {
        "_id" : "2541",
        "content" : "Hi ! How are you ?",
        "author" : "Dan"
      },
      {
        "_id" : "2542",
        "content" : "Rocking now with JetProbe",
        "author" : "Don"
      }
    ]
  }
```
Now for such json response, we can express the validation as below.

```scala
validateGiven("Testing a negative case",getPosts) { res =>

     given[String]("$.data.first_name") { resp =>
       assertEquals("Janet2",resp)

     }
   }
```

## Execution of Pipelines

A typical scenario would consist of collection of pipelines, which would get executed by the Jetprobe runtime library and at the end the results would be available both in the console and the HTML file.

Follow the steps to run the scenario :

* Package the class file into a jar. If you are using sbt to build the project then just use `sbt package` to build the jar.
* Create a yaml configuration file say `config.yml` that would contain the properties that are being defined in the scenario definition.

```yml
//Name of the Scenario
name : data ingestion test
project : user-analytics

tags :
  - https
  - mongo
  - test

# params for the pipelines
params :
  mongo.host.name : xx.xx.xxx.xxx
  reporter.html.outputPath : /path/to/report.html

pipelines :
  - name : Mongo-prep
    className : com.jetprobe.sample.MongoSuite
    description : Mongo db preparation
    exitOnFailure : true
```
and save it at say `/data/path/config.yml`. Use the jar to run the validations as below.

  ```sh
  $ jetprobe --jar /path/to/job.jar --config /data/path/config.yml
  ```

For more examples, head over to the [example project](https://github.com/jetprobe/jetprobe/tree/master/jetprobe-sample/src/main/scala/com/jetprobe/sample)
