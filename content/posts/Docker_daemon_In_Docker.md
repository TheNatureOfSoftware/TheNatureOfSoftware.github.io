---
title: "Docker daemon in docker"
date: 2017-09-12T14:23:30+02:00
draft: false
description: "Running Docker daemon in docker on build agents."
tags: ["Spinnaker", "Jenkins", "Docker", "Kubernetes"]
categories: ["English", "CI/CD", "DevOps"]
featured_image: "/images/docker_daemon_in_docker.svg"
---

{{< figure src="/images/docker_daemon_in_docker.svg" >}}
So while I was working on setting up a Continuous Delivery toolchain using Kubernetes and [Spinnaker](https://spinnaker.io) I stumbled upon a little problem. Spinnaker can be used with Kubernetes for CD when you have a docker image. But you need some kind of step that transforms your application in to an image. You can trigger this step from Spinnaker by invoking the build
in your favorite Continuous Integration tool.

Packaging your application in to an image often follows align with one of these scenarios:

1. create the image as a last step of your ordinary build invoking `docker build`
2. have a separate step after your ordinary build that pulls
your application artifact from some kind of repository and builds the image
3. build your application using a build image and let the build artifact be an image

The third alternative has become much more attractive with Docker multi-stage build, which is awesome, where you can have multiple `FROM` (stage) and copy artifacts from earlier stages.
You can now have both small images and fully reproducible builds, without any complicated scripts.

There's is a great [plugin](https://wiki.jenkins.io/display/JENKINS/Kubernetes+Plugin) for Kubernetes that lets you dynamically provision Jenkins agents to run a single build. This is exactly what I want, and I want the agent to build and push an image on every build.

So far we have the Kubernetes plugin in Jenkins that:
* creates a build agent pod
* checks out the code and delivers a reproducible build

It's around here it's starts to get complicated. How do we let our
build agent, who's running in a pod as a docker container, run docker builds? If you've done this before you may be have read [~jpetazzo/Using Docker-in-Docker for your CI or testing environment? Think twice.](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/). His recommendation is to run sibling containers by mounting the
Docker socket like this:
```
docker run -v /var/run/docker.sock:/var/run/docker.sock \
           -ti docker
```

This wont work in this case because we are dependent on the hosts docker version and in my case the host is running docker `1.12.6`
with no support for multi-stage and `build --network host`.

The problem with running sibling containers is that you introduce a dependency on the host where your build container is running. Our hosts and our build containers have very different purposes. Our hosts should deliver a rock solid infrastructure for executing containers. The build environment, on the other hand, can be using the latest and greatest to
give us the best tools available.  

So instead of running sibling containers I decided to try *dind* (docker in docker). I'm now using two container in the build agent pod:

* one `jnlp` with the latest and greatest Docker CLI
* one latest and greatest `dockerd` daemon

{{< figure src="/images/spin-jenkins-slave.png" >}}

The docker daemon is using the [`vfs` storage driver](https://docs.docker.com/engine/userguide/storagedriver/vfs-driver/) that should work everywhere and accepts connections on `tcp://localhost:2375`. The Docker daemon must run in `--privileged` mode.

One of the downsides with this solution is that your build have to pull all images on every build which increases network traffic and slows down the build. One solution to this is that you can easily "pre-pull" the images used during the build, in a container, and commit that as the new
image for your `dockerd` container.

### Links

* https://docs.docker.com/engine/userguide/storagedriver/vfs-driver
* https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci
* https://wiki.jenkins.io/display/JENKINS/Kubernetes+Plugin
* https://github.com/kubernetes/charts/tree/master/stable/spinnaker
