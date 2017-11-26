---
date: 2016-03-09T20:08:11+01:00
title: Writing validations
weight: 30
---

## Scenario Definition

Validations are special actions, designated to validate the state of an infrastructure component.

## RabbitMQ Validations

Let's look at a scenario where we would want to validate if the particular component of RabbitMQ infrastructure has been initialized before we start the stream
processing.

```Scala
//create the RabbitMQ instance that needs to be validated. The host here ${rabbit.host} would be resolved at the runtime.
val rabbit = RabbitMQSink("${rabbit.host}")

//Define the scenario which has the validation as part of the entire data validation pipeline.
scenario("RabbitMQ validation")
.validate[RabbitMQSink](rabbitMQ) {

    // the second parameter takes a collection(Seq in scala) of validation rules for the RabbitMQ Sink
      given(exchange("amq.topic", "/"))(
        //a type parameterized method, where type signifies the result type of the validation.
        checkExchange[Boolean](true, exchangeProps => exchangeProps.durable),
        checkExchange[Boolean](false, _.isAutoDelete)
      )
    }
 .build
```

The test utility method `checkExchange` takes the expected output value of the function that would be executed on the actual Exchange properties that would be fetched
by the JetProbe framework during the runtime. This way the framework frees the developers to write the boilerplate code pertaining to connecting to RabbitMQ and fetching the
required details, instead the developer would just need to extract the required value for validating the state of the component.

## MongoDB Validations

For validating the mongodb components like database, collection and the records, there are some utility methods allowing the developers to quickly write the validation rules
given the state of the component as a POJO/case class(approx).

```scala

// Declare the instance of MongoDB Sink that needs to be validated
 val mongo = MongoSink("mongodb://${mongo.host}/")

//Define the scenario which has the validation as part of the entire data validation pipeline.
scenario("MongoDB validation")
.validate[MongoSink](mongo){

//General server stats validation rules
  val serverValidations = Seq(
      checkStats[String]("3.4.0", _.version),
      checkStats[Boolean](true, _.version.startsWith("3.4")),
      checkStats[Boolean](true, _.connections.current < 50),
      checkStats[Long](80L, _.opcounters.insert)
    )

  // Validations for the given database
  val databaseStats = given(mongo("${customer.transaction.db}"))(
      checkDBStats[Boolean](true, _.collections == 10)
    )    

  // Add both the validations
  serverValidations ++ databaseStats
}
```
Just like the RabbitMQ validations, the JetProbe framework provides the developer with helper methods to express different validation rules. Other helper methods are :

* **checkDatabaseList** : Fetches the list of databases
* **checkCollectionStats** : Fetches the details of the given collection in a database.
* **checkDocuments** : Given a Mongo query, it fetches the list of documents, against which validation rules can be written.

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
.validate[HttpRequestBuilder](getPosts) {

  //validate the response status code
   val responseValiation = Seq(
     checkHttpResponse(202, _.status)
   )
   //validate the comments count
    val commentsCount = given(jsonQuery = "$.commentsCount")(
      checkExtractedValue(true, x => x == 400)
    )

    val contentValidation = given(jsonQuery = "$.comments[0].author")(
      checkExtractedValue(true, _ == "Dan")
    )
      responseValiation ++ commentsCount ++ contentValidation
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
