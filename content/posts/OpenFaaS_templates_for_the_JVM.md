---
title: "OpenFaaS templates for the JVM"
date: 2019-01-03T09:16:38+01:00
draft: false
description: ""
tags: ["OpenFaaS", "Serverless", "CloudNative", "Kubernetes", "Docker", "Kotlin", "Java", "Micronaut"]
categories: ["English", "Serverless", "CloudNative"]
featured_image: "/images/pexels-photo-1181298.jpeg"
lang: "en"
---

I've been wanting to try out [OpenFaaS](httos://openfaas.com) for a long time and finally got some time during the holidays to give it a try. There are some excellent tutorials you can follow to get you started and a self-paced [workshop](https://github.com/openfaas/workshop). I used Alex Ellis [*Getting started with OpenFaaS on minikube*](https://medium.com/devopslinks/getting-started-with-openfaas-on-minikube-634502c7acdf) to get started.

### OpenFaaS templates

OpenFaaS templates makes it easy to get started writing and deploying functions using the [`faas-cli`](https://docs.openfaas.com/cli/install/).
You can read more about templates and the `store`-command in the blog post [Introducing the Template Store for OpenFaaS](https://www.openfaas.com/blog/template-store/).

The templates are stored in the `templates` directory in the current dir. You don't have to pull the official templates before creating a new function. The official templates are pulled automatically if there's no template directory.

Here's how to pull all official OpenFaaS templates:

```bash
$ faas-cli template pull
```

However you can't pull an individual template from a repository. The following command tries to pull the official `java8` template:

```bash
$ faas-cli template store pull java8
```

... but will result in all official templates beeing pulled. 

### OpenFaaS templates for the JVM

There's currently only one official template for the [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) and that's the `java8`
template.

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

The `faas-cli new` command creates a new directory for your function using the chosen template. We used the `java8` template and here's what it looks like:

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

As you can see, the template is using `gradle` as build tool, but if you try to build the function using `gradle` your build will fail:

```
FAILURE: Build failed with an exception.
```

This is because you are supposed to build your function using the `faas-cli` that uses Docker to build your function:

```bash
$ faas-cli build -f fn-hello-java.yml
```

Our `fn-hello-java` function from the `java8` template is dependent on two other artifacts: `model` and `entrypoint` that are part of the template.
You can find them if you take a look in the `template/java8` directory. This make sense from a `faas-cli` perspective but is less optimal when your
a developer using your favorite IDE.

### A Kotlin template

I've been writing Serverless applications in Kotlin for AWS Lambda using [Serverless Framework](https://serverless.com/) and wanted to do the same for
OpenFaaS to get a feel how it's like and compare the experiences.

There's currently no official template for Kotlin so I created one that uses the [Micronaut](https://micronaut.io) framework.

You can pull the template with the following command:

```bash
$ faas-cli template pull https://github.com/TheNatureOfSoftware/openfaas-templates
```

This will give you two new templates: `kotlin-maven-mn` and `kotlin-gradle-mn`. You can now create a new function
using one of them:

```bash
$ faas-cli new fn-hello-kotlin-maven --lang kotlin-maven-mn --prefix <your docker image prefix>
```

... and build and deploy it:

```bash
$ faas-cli up -f fn-hello-kotlin-maven.yml
```

... and finally invoke it:

```bash
$ echo -n "test" | faas-cli invoke fn-hello-kotlin-maven
```

If you take a look at the generated directory for our new Kotlin function you can see that it's using Maven as build tool:

```bash
$ tree fn-hello-kotlin-maven
fn-hello-kotlin-maven
├── micronaut-cli.yml
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    └── main
        ├── kotlin
        │   └── function
        │       ├── Fn.kt
        │       └── Handler.kt
        └── resources
            ├── application.yml
            └── logback.xml

5 directories, 8 files
```

The template does not follow the convention from the official `java8` template with the separation of `entrypoint`, `model` and `handler`.
This lets you build and run your function directly from your favorite IDE without running `faas-cli build`. The `entrypoint` and `model`
in the Kotlin template are replaced by the [Micronaut](https://micronaut.io) framework and uses an embedded Jetty.

You can easily create your own template and replace Micronaut with [Spring Boot](http://spring.io/projects/spring-boot).

If we compare the Docker images for our functions created with the two different templates (`java8` and `kotlin-maven-mn`) we see
that there's no significant difference is size and that the overhead of using an embedded web container like jetty is negligible.

```bash
$ docker images --format "{{.Repository}}\t{{.Size}}" thenatureofsoftware/fn-hello-*
thenatureofsoftware/fn-hello-java	124MB
thenatureofsoftware/fn-hello-kotlin-maven	119MB
```

Using a framework like Micronaut or Spring Boot let's you create [stateless microservices for OpenFaaS](https://www.openfaas.com/blog/stateless-microservices/)
using the JVM ecosystem.
