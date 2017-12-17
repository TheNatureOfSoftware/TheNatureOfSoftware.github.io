+++
date = "2017-06-03T11:43:35+02:00"
title = "Vad är er containerstrategi?"
categories = ["container", "docker", "devops", "kubernetes"]
tags = ["Microsoft", "Kubernetes", "Azure", "Deis"]
author = "Lars Mogren"
banner = "img/deis-microsoft.png"
draft = false
lang = "sv"
+++

Om du tror att Docker och containers bara är för Linux så är det dags att tänka
om. [Docker Windows Containers](https://www.docker.com/microsoft) har funnits
tillgängligt sedan [Windows Server 2016](https://blog.docker.com/2016/09/build-your-first-docker-windows-server-container/).

Även om det är en stor fördel att paketera och drifta sina system som containers
så är det tillsammans med en orkestreringslösning som man når de verkliga fördelarna.
Med en orkestreringslösning för containers frikopplas applikationer och system
från infrastruktur så som nät, disk, minne och cpu. Det innebär en stor fördel om
man satsar på self-service, en av grundstenarna inom DevOps och agila organisationer.

Den främsta orkestreringslösningen för containers är [Kubernetes](https://kubernetes.io).
Kubernetes har haft (alfa) stöd för Windows Containers sedan version [1.5](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md#v150).

<img src="/img/deis-microsoft.png" style="float:left;margin-right:20px;height:80px">
I slutet på februari (2017) lanserade Microsoft Kubernetes på [Azure Container Service](https://azure.microsoft.com/en-us/blog/kubernetes-now-generally-available-on-azure-container-service/).
Azure Container Service har stöd för Windows containers i Kubernetes som en
förhandsversion ([Kom igång med Kubernetes och Windows-behållare i Container Service](https://docs.microsoft.com/sv-se/azure/container-service/container-service-kubernetes-windows-walkthrough)).

I april (2017) köpte Microsoft [Deis](https://deis.com/) ([Microsoft to acquire Deis to help companies innovate with containers](https://blogs.microsoft.com/blog/2017/04/10/microsoft-acquire-deis-help-companies-innovate-containers)),
vars lösning för att öka Dev och Ops produktivitet med containers, bygger på Kubernetes.

Kubernetes finns nu tillgängligt hos alla större molnleverantörer:

* [IBM Bluemix Container Service](https://www.ibm.com/blogs/bluemix/2017/03/kubernetes-now-available-ibm-bluemix-container-service/)
* [Google Container Engine](https://cloud.google.com/container-engine/)
* [Azure Container Service](https://azure.microsoft.com/en-us/blog/kubernetes-now-generally-available-on-azure-container-service/)
* [AWS](https://aws.amazon.com/about-aws/whats-new/2017/03/new-quick-start-deploys-heptio-kubernetes-on-the-aws-cloud/)

och on-premise (på egen hårdvara):

* [CoreOS Tectonic](https://coreos.com/tectonic/)
* [Red Hat OpenShift](https://www.openshift.com/)

Man kan nu tydligt se vart vi är på väg och utvecklingen går otroligt fort inom
området. För ett par år sedan var frågan för många organisationer om man skulle
satsa på .NET eller Java (väldigt förenklat). Många organisationer har
fortfarande valet som en strategi, men har i realiteten en väldigt blandad miljö.

Företag och organisationer är idag ofta teknikföretag, vare sig man vill det
eller inte. Containers ger oss helt andra möjligheter och tillsammans med [Microservices](https://martinfowler.com/articles/microservices.html),
[Continuous Delivery](https://continuousdelivery.com/) och
[DevOps](https://www.goodreads.com/book/show/26083308-the-devops-handbook) m.fl.
så är det dags för många att lära om.

Så vad är er containerstrategi?
