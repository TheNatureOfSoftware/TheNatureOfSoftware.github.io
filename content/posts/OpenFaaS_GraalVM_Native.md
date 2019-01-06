---
title: "OpenFaaS native Kotlin functions"
date: 2019-01-06T22:22:52+01:00
draft: false
description: "Writing GraalVM Native functions in Kotlin for OpenFaaS"
tags: ["OpenFaaS", "Serverless", "CloudNative", "Kubernetes", "Docker", "Kotlin", "GraalVM"]
categories: ["English", "Serverless", "CloudNative", "Kotlin"]
featured_image: "/images/openfaas_kotlin_graalvm.png"
lang: "en"
---

I recently wrote a short blog post: [OpenFaaS templates for the JVM]({{< ref "OpenFaaS_templates_for_the_JVM.md" >}}) and
created a set of [OpenFaaS Kotlin templates](https://github.com/TheNatureOfSoftware/openfaas-templates). I like the idea of
using Docker images as packaging format for Serverless functions, but the image size is quite large. The `openjdk:8-jre-alpine`
base image size is `83 MB` and the `Kotlin Micronaut` template image is `118 MB`. But what if we could shrink the image size to `16 MB`
with improved startup time and reduced memory footprint?

## GraalVM

[GraalVM](https://www.graalvm.org) has a tool for creating [Native Images](https://www.graalvm.org/docs/reference-manual/aot-compilation/) that
can even be statically linked. I've created a template named `kotlin-graalvm-native` that uses GraalVM for transforming a Kotlin JVM function
to a statically linked function. The template uses the classic `watchdog`.

## What's the catch?

I'm no GraalVM expert so I point you to this blog post: [Instant Netty Startup using GraalVM Native Image Generation](https://medium.com/graalvm/instant-netty-startup-using-graalvm-native-image-generation-ed6f14ff7692) to give you an idea of what to look out for.
But if you are careful with your dependencies and stay away from reflection then I think this could be really use full.

## Do you want to give it a try?

Here's how to deploy your first Kotlin GraalVM Native function. First make sure you have the following prerequisite:

* OpenFaaS up and running
* `faas-cli` installed and working

First pull the template:

```bash
$ faas-cli template pull https://github.com/TheNatureOfSoftware/openfaas-templates
```

This will give you a template directory with four templates:

```bash
$ ls template/
kotlin-graalvm-native	kotlin-gradle-mn	kotlin-maven-mn		kotlin-maven-spring
```

Create a new function using the `kotlin-graalvm-native` template:

```bash
$ faas-cli new fn-kotlin-native --lang kotlin-graalvm-native --prefix <your-image-prefix>
```

... and build and deploy it:

```bash
$ faas-cli up -f fn-kotlin-native.yml
```

And finally invoke your Kotlin GraalVM Native function:

```bash
$ echo -n "This is awsome" | faas-cli invoke fn-kotlin-native
```

Hope you enjoyed this!
