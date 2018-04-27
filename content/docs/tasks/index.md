---
title: Tasks
weight: 20
---

## Declaring Tasks

An task in the context of a data pipeline is a stage or a phase that is part of the entire execution. Every task is executed sequentially and takes an task builder
as a paramter. Let's see in detail, some of the inbuilt tasks in Jetprobe.

## Http Tasks

In case of http action, the parameter passed is the http request builder. For building a http request, a utility class known as `Http` is used. Here's a sample http request :

```scala
val getRequest = Http("get-post-detail")
    .get("https://${server.hostname}/posts/1")
```

Here's another POST request :

```scala
val postRequest = Http("create-a-user")
    .post("https://reqres.in/api/users")
    .body(
      fromFile("/path/to/post_req.json")
    )
```

Here `${server.hostname}` is a variable that can be interpolated based on the config passed during the execution of the test pipeline.

Once you have defined the request, then you can use http(...) helper method to include the above request as a part of the data pipline test.

```Scala

http(getRequest)

pause(1.seconds)

http(postRequest)
```

Now you may also want to extract a some data from a request and use it in downstream processing.
Let's define another request, where we create a test user.

```scala
val postRequest = Http("create-a-new-user")
    .post("https://${api.server.address}/api/users")
    .body(
      fromFile("/path/to/post_req.json")
    ).header("Content-type", "application/json")
    .extract(
      jsonPath("$.name", saveAs = "user.name"),
      jsonPath("$.id", saveAs = "user.id")
    )
```

Once the values are saved in the variables, the runtime engine of Jetprobe would make these variables available to all the other downstream processing actions.

### Using runtime variables
```Scala
val getUser =  Http("get-user-details")
    .get("https://${server.hostname}/user/${user.id}")
```

Here `${user.id}` would be replaced by the value extracted by the json expression.

## Pause Task

A pause action, would simply pause the data pipeline, to allow the processing of data, after which the developer can execute the validations or some other set of actions.
The pause action takes a duration as an parameter.

**A Pause Action example**
```scala
pause(4.seconds)
```

## SSH Task

A ssh action allows the developers to execute a remote command by providing the credentials for remote connections.

**SSH Action example**

```scala
//First define the ssh config
val sshConfig = SSHConfig(user = "admin", password = "secret", hostName = "xx.xx.xx.xx")

ssh(description = "some random commands ",sshConfig) { client =>
  client.run("cd /home/me/apps && mkdir temp && ls")
}

```

## Custom Tasks

A custom task for a target storage can be executed by defining the configuration required to connect to the storage system, and
then leverage the underlying APIs exposed by the Storage system.

**File Based Action**
```scala
val fs = new FilePath("/path/to/file.in")

//custom action with file
task(description = "read the file",fs) { file =>

      //read the file
      file.lines().foreach(println)

      //delete the file
      file.rm

      //copy the file
      file.copyTo("/path/to/destination")

      //Want something more ? Check this out
      file.usingFile[T](fn : File => T) : T
    }
```
In the above example, the `task` function takes the description and configuration pertaining to a storage system, and takes another callback function to execute file based commands.

Here's another example for HDFS

**HDFS Based Action**
```scala
val hdfsConf = new HDFSConfig("hdfs://<namenode-host>","loginUser")

task(description = "Copy from hdfs",hdfsConf){ hadoop =>

        //Copy to hdfs
        hadoop.copyFromLocal(localSrc= "/local/file", destination = "/user/hdfs/path")

        //Copy to local from hdfs
        hadoop.copyToLocal("/user/hdfs/srcPath","/local/save/path")

      }

```

## Validation task

Validation task take the `storage configuration`, as a parameter which is nothing but the target that needs to be validated, and also the collection of validation rules that needs
to be executed for that particular sink.

A storage could be a database, a file, a messaging infrastructure or any http service. In the next section, we would see how to write validations for a given storage.
