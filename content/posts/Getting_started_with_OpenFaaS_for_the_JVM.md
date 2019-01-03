---
title: "Getting started with OpenFaaS for the JVM"
date: 2019-01-03T09:16:38+01:00
draft: true
description: ""
tags: ["OpenFaaS", "Serverless", "CloudNative", "Kubernetes", "Docker", "Kotlin", "Java", "Micronaut"]
categories: ["English", "Serverless", "CloudNative"]
featured_image: "/images/pexels-photo-546819.jpeg"
lang: "en"
---

I've been wanting to try out [OpenFaaS](httos://openfaas.com) for a long time and finally got some time during the holidays to give it a try. There are some excellent tutorials you can follow to get you started and I used one for [`minikube`](https://medium.com/devopslinks/getting-started-with-openfaas-on-minikube-634502c7acdf). It's easy to deploy a new function with the [`faas-cli`](https://docs.openfaas.com/cli/install/) and there are templates for all the major languages. I wanted to try out writing a function in Kotlin and see what it felt like.

There's currently no official template for Kotlin, but there is one for [Java 8](https://github.com/openfaas/templates/tree/master/template/java8).

Here's how to pull all official OpenFaaS templates:

```bash
$ faas-cli template pull
```
The templates are stored in the `templates` directory in the current dir. You don't have to pull the official templates before creating a new function. The official templates are pulled automatically if there's no template directory.

The following command creates a new function using the `java8` template:

```bash
$ faas-cli new fn-hello-java --lang java8 --prefix <your docker image prefix>
```

You can now build and deploy your new function with one command:

```bash
$ faas-cli up -f fn-hello-java.yml
```
... and invoke it:

```bash
$ echo -n "test" | faas-cli invoke fn-hello-java
```

The `faas-cli new` command creates a new directory for your function using the chosen template. I used the `java8` template and here's what it looks like:

```bash
$ tree fn-hello-java
fn-hello-java
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── settings.gradle
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── openfaas
    │               └── function
    │                   └── Handler.java
    └── test
        └── java
            └── HandlerTest.java

10 directories, 6 files
```

As you can see, the template is using `gradle` as build tool but if you try to build the function using `gradle` your build will fail:

```
FAILURE: Build failed with an exception.
```

This is because you are supposed to build your function using the `faas-cli` that uses Docker to build your function:

```bash
$ faas-cli build -f fn-hello-java.yml
```

Your `java8` function is dependent on two other artifacts: `model` and `entrypoint` that are part of the template.
You can find them if you take a look in the `template/java8` directory. This make sense from a `faas-cli` perspective but is less optimal when your a developer using your favorite IDE.

But there's no problem we can't fix and the OpenFaaS template feature is flexible enough to let you create your own templates. So that's what I've done using Kotlin and [Micronaut](https://micronaut.io).

Let's pull the new template:

```bash
$ faas-cli template pull https://github.com/TheNatureOfSoftware/openfaas-templates
```

... and create a new Kotlin function:

```bash
$ faas-cli new fn-hello-kotlin --lang kotlin-maven-mn --prefix <your docker image prefix>
```

... and build and deploy it:

```bash
$ faas-cli up -f fn-hello-kotlin.yml
```

... and finally invoke it:

```bash
$ echo -n "test" | faas-cli invoke fn-hello-kotlin
```

If you take a look at the generated directory for our new Kotlin function you can see that it's using Maven as build tool:

```bash
$ tree fn-hello-kotlin
fn-hello-kotlin
├── micronaut-cli.yml
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    └── main
        ├── kotlin
        │   └── sample
        │       ├── Fn.kt
        │       └── HelloWorldController.kt
        └── resources
            ├── application.yml
            └── logback.xml

5 directories, 8 files
```

And you can import the project in your favorite IDE or run a standalone `mvn package` build.
The function is implemented in the `HelloWorldController.kt`. If you compare the Docker images between our two `fn-hello-*`
functions you can see that our Kotlin function is slightly smaller:

```bash
$ docker images --format "{{.Repository}}\t{{.Size}}" thenatureofsoftware/fn-hello-*
thenatureofsoftware/fn-hello-java	124MB
thenatureofsoftware/fn-hello-kotlin	119MB
```

This is only to show that our Kotlin function without the `java8` templates `entrypoint` and `model` hasn't grown in any significant size or complexity.

It's very straight forward to create your own template based on your favorite framework and build tool and easy to test. Let's create a new template for Kotlin, Micronaut and Gradle.

First create a Git repository with the following directories:

```bash
$ mkdir -p openfaas-templates/kotlin-gradle-mn
```

We can now use the [Micronaut client](https://docs.micronaut.io/latest/guide/index.html#buildCLI) to bootstrap a new function (or Micronaut application):

```bash
$ cd kotlin-gradle-mn
$ mn create-app function --build gradle --lang kotlin
```

You can now test that your function template starts:

```bash
$ gradle run
```

If you try to access your function at `http://localhost:8080` you will see that Micronaut gives you a `404` reply:

```bash
$ curl http://localhost:8080
{"_links":{"self":{"href":"/","templated":false}},"message":"Page Not Found"}
```

Don't worry, that's because we haven't written any function handler.

Now let's add a handler in the `src/main/kotlin/function` directory:

{{< highlight java >}}
@Controller("/")
class Handler {
  @Consumes(MediaType.TEXT_PLAIN)
  @Post(produces = [MediaType.TEXT_PLAIN])
  fun index(@Body() msg: String): String {
    return """
      Hello World from Kotlin Micronaut!
      Message: $msg
      """.trimIndent()
  }
}
{{< /highlight >}}

And at the same time rename the main class from `Application` to `Fn`. Try run your function using `gradle run` and `curl`:
```bash
$ curl -X POST -d "test" -H "Content-Type: text/plain" http://localhost:8080
Hello World from Kotlin Micronaut!
Message: test
```

What's left is to create `Dockerfile` for building our OpenFaaS function image, you can see how I did it [here](https://github.com/TheNatureOfSoftware/openfaas-templates/blob/master/template/kotlin-gradle-mn/Dockerfile).
