---
layout: post
title:  利用OpenJDK CRaC帮助容器有效扩展热应用程序实例
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 简介

在本教程中，我们将学习[检查点协调恢复(CRaC)](https://wiki.openjdk.org/display/crac)，这是一个OpenJDK项目，它允许我们在更短的时间内启动Java程序并完成第一个事务。此外，我们将了解[Alpaquita容器](https://bell-sw.com/alpaquita-containers-for-spring-boot-apps/?utm_source=baeldung&utm_medium=ref_article&utm_campaign=baeldung&utm_content=openjdk_crac)如何帮助我们轻松地在Spring Boot应用程序中实现CRaC。

## 2. OpenJDK CRaC如何解决Java中的缓慢预热问题？

**Java应用程序一直以来都饱受诟病，主要原因包括启动速度慢、预热时间过长(需要较长时间才能达到稳定的峰值性能)**。此外，它们在预热期间消耗的计算资源比稳定运行时消耗的资源还要多。

这种行为很大程度上可以归因于HotSpot Java虚拟机(JVM)的根本工作原理。当应用程序启动时，JVM会在代码中查找热点并进行编译以获得更好的性能，但是，这需要时间和计算资源来实现：

![](/assets/images/2025/javajvm/openjdkcrachotapplicationinstancesscaling01.png)

此外，每个应用程序实例都必须重复此操作，**在微服务和Serverless等云原生架构中，这个问题更加严重**，我们需要尽可能缩短预热时间，同时保持资源消耗相对稳定。

如果我们能让应用程序运行到其峰值性能并检查该状态，会怎么样？然后，我们可以使用此检查点启动应用程序的多个实例，而无需花费太多时间进行预热，这基本上就是[OpenJDK CRaC](https://wiki.openjdk.org/display/crac) API向我们承诺的：

![](/assets/images/2025/javajvm/openjdkcrachotapplicationinstancesscaling02.png)

CRaC基于[用户空间检查点和恢复(CRIU)](https://criu.org/Main_Page)，**这是一个为Linux实现检查点和恢复功能的项目**。CRIU允许冻结容器或单个应用程序，并从保存的检查点文件中恢复它。

但是，CRaC采用了CRIU的通用方法，并添加了一些增强和调整，使其适用于Java应用程序。例如，CRaC对应用程序的状态施加了某些限制，以保证检查点的一致性和安全性。

## 3. 采用CRaC的挑战

CRaC为基于Java的应用程序在云环境中更高效地运行开辟了新的机会，Spring是开发基于Java的应用程序的流行框架之一，**随着Spring Boot 3.2的发布，我们现在在Spring框架中初步支持CRaC**。

但是，CRaC并不像看起来那么便携。正如我们已经讨论过的，CRaC仅适用于Linux，因为CRIU是Linux特有的功能，在其他操作系统上，CRaC有一个用于创建和加载快照的无操作实现。

此外，**CRaC要求在拍摄快照之前关闭所有文件和网络连接**。恢复检查点后，必须重新打开这些文件和网络连接，这需要Java运行时和框架的支持。

因此，我们不仅需要Spring的支持，还需要一个支持CRaC的JDK版本，例如BellSoft提供的Liberica JDK。此外，我们需要在Linux发行版上运行Spring应用程序，例如BellSoft的Alpaquita Linux。

因此，**如果我们能将应用程序与在类似Linux环境中运行的支持CRaC的JDK打包成一个可移植容器，那么该解决方案将具备极强的可移植性和即插即用性**，这正是BellSoft为现代Java应用程序提供的宝贵承诺！

## 4. CRaC与Alpaquita容器

[BellSoft](https://bell-sw.com/?utm_source=baeldung&utm_medium=ref_article&utm_campaign=baeldung&utm_content=openjdk_crac)是一家OpenJDK供应商，为云原生Java应用程序提供端到端解决方案。作为其中的一部分，它提供了一套针对运行Java应用程序高度优化的容器，他们打包了[Alpaquita Linux](https://bell-sw.com/alpaquita-linux/?utm_source=baeldung&utm_medium=ref_article&utm_campaign=baeldung&utm_content=openjdk_crac)和[Liberica JDK](https://bell-sw.com/libericajdk/?utm_source=baeldung&utm_medium=ref_article&utm_campaign=baeldung&utm_content=openjdk_crac)，这两个都是BellSoft的产品。

**Alpaquita Linux是唯一专为Java构建并针对云原生应用部署进行优化的Linux发行版**，它通过内核优化、内存管理和优化的malloc内存分配，实现了更佳性能，其基础镜像大小仅为3.28MB！

**Liberica JDK是一个用于云原生Java部署的开源Java运行时**，它支持最广泛的架构和操作系统，是一个真正统一的Java运行时，除了安全合规之外，它还有助于构建成本和时间高效的容器。

**BellSoft管理着多个公共镜像**，提供各种JDK类型(jre、jdk或jdk-all)、Java版本(包含对最新LTS版本Java 21的支持)和libc类型(glibc或musl)的组合。现在，BellSoft还提供支持CRaC和CDS(类数据共享)的镜像。

**这些即用型镜像使我们能够将CRaC无缝集成到Spring Boot应用程序中**，目前，此功能适用于x86_64架构的JDK 17和21。BellSoft声称，带有CRaC的Alpaquita Containers可将启动时间缩短164倍，并将镜像体积缩小1.1倍。

镜像大小的减小主要归功于驻留集大小(RSS)的减少，RSS指的是进程占用的内存中，保留在主内存(RAM)中的部分。其中一个关键因素是，带有CRaC的Liberica JDK在检查点之前执行了完整的垃圾收集。

## 5. 示例

BellSoft的产品非常适合基于Spring Boot的Java应用程序，**[Spring推荐使用BellSoft Liberica JDK](https://spring.io/quickstart)，它是Spring Boot中的默认Java运行时**。在本教程中，我们将使用Spring Boot应用程序，并使用Alpaquita容器执行CRaC。

### 5.1 准备应用程序

在本教程中，我们将创建一个简单的Spring Boot应用程序来探索CRaC，我们将复用[上一篇教程](https://www.baeldung.com/spring-docker-liberica#a_simple_spring_boot_application)中创建的应用程序，本教程将使用Java 21和Spring Boot 3.2.5，CRaC在这种组合下运行良好。

但是，为了能够使用CRaC，我们需要将[Maven中央仓库中提供的crac包](https://central.sonatype.com/artifact/org.crac/crac)添加为Spring Boot应用程序的依赖：

```groovy
implementation("org.crac:crac:1.4.0")
```

现在，我们必须使用Gradle构建应用程序以在目录“./build/libs”中生成可执行JAR：

```shell
$ ./gradlew clean build
```

现在我们已经创建了一个带有CRaC依赖的简单Spring Boot应用程序，我们需要使用支持CRaC的JDK来运行它，为此，我们将使用支持CRaC的Alpaquita容器，BellSoft在其[Docker Hub仓库](https://hub.docker.com/r/bellsoft/liberica-runtime-container)上管理多个镜像。

值得庆幸的是，所有支持CRaC的镜像都带有标签'crac'，在本教程中，我们将在我们的机器上拉取一个这样的镜像：

```shell
$ docker pull bellsoft/liberica-runtime-container:jdk-21-crac-slim-glibc
```

这里，“jdk-21-crac-slim-glibc”是镜像的标签。这样，我们就可以开始尝试CRaC的检查点和恢复功能了，我们将看到Alpaquita Containers如何让这一切变得轻松便捷。

### 5.2 启动应用程序

首先，在“./build/libs”目录中创建一个名为“checkpoint”的目录，用于保存应用程序转储。现在，我们将使用之前拉取的Alpaquita容器镜像来运行上一节中创建的应用程序JAR文件：

```shell
$ docker run -p 8080:8080 \
  --rm --privileged \
  -v $(pwd)/build/libs:/crac/ \
  -w /crac \
  -n fibonacci-crac \
  bellsoft/liberica-runtime-container:jdk-21-crac-slim-glibc \
  java -Xmx512m -XX:CRaCCheckpointTo=/crac/checkpoint \
  -jar spring-bellsoft-0.0.1-SNAPSHOT.jar
```

在这里，我们将容器端口8080映射到主机端口8080。**我们还使用了“privileged”模式，因为这对于底层CRIU正常工作是必要的**。

此外，我们将应用程序JAR所在的目录映射为容器内的卷，并将其用作工作目录。最后，我们提供了Java命令来运行JAR，并附带一些必要的参数。

如果一切顺利，我们应该能够检查容器日志并验证我们的应用程序确实已经启动：

```text
2024-04-22T15:27:39.730Z  INFO 129 --- [main] 
  cn.tuyucheng.taketoday.demo.Application : Started Application in 3.203 seconds (process running for 4.727)
```

现在，**我们应该向应用程序执行一些请求，以便JVM可以获取已编译的热代码，从而获得更好的性能**。不过，对于我们这个简单的应用程序来说，这些影响可以忽略不计。

### 5.3 执行检查点

现在，我们已准备好执行应用程序的检查点操作。但在此之前，我们先检查一下RSS的大小，并将其与恢复后的大小进行比较，我们需要应用程序的进程ID(PID)来执行此操作：

```shell
$ docker exec fibonacci-crac ps -a | pgrep spring-bellsoft
```

一旦我们获得了PID，我们就可以使用“pmap”命令来查找RSS的大小：

```shell
$ docker exec fibonacci-crac pmap -x <PID> | tail -1
total            4845016  134128  118736       0
```

此命令的输出显示RSS的大小(以千字节为单位)，此处的第二个值(134128)。

现在，让我们在此状态下执行应用程序的检查点，我们可以通过**使用“jcmd”命令来执行此操作，该命令向JVM发送命令以执行检查点**：

```shell
$ docker exec fibonacci-crac jcmd <PID> JDK.checkpoint
```

请注意，“fibonacci-crac”是我们启动容器时使用的名称。**执行此命令后，Java实例将被转储，容器也将停止**。应用程序转储包含多个文件，位于我们提到的位置：

```shell
$ ls
core-129.img  core-139.img  core-149.img  core-198.img   pagemap-129.img
core-130.img  core-140.img  core-150.img  core-199.img   pages-1.img
core-131.img  core-141.img  core-151.img  core-200.img   pstree.img
core-132.img  core-142.img  core-152.img  dump4.log      seccomp.img
core-133.img  core-143.img  core-154.img  fdinfo-2.img   stats-dump
core-134.img  core-144.img  core-155.img  files.img      timens-0.img
core-135.img  core-145.img  core-156.img  fs-129.img
core-136.img  core-146.img  core-158.img  ids-129.img
core-137.img  core-147.img  core-159.img  inventory.img
core-138.img  core-148.img  core-160.img  mm-129.img
```

此转储包括正在运行的Java应用程序的确切状态以及有关堆、JIT编译代码等的信息。但是，正如我们之前讨论的那样，我们在此处使用的Liberica JDK在检查点之前执行完整的垃圾收集。

### 5.4 从转储启动应用程序

现在，我们剩下要做的就是**使用之前创建的应用程序转储来恢复应用程序实例**，这就像定期启动应用程序一样简单：

```shell
docker run -p 8080:8080 \
  --rm --privileged \
  -v $(pwd)/build/libs:/crac/ \
  -w /storage \
  -n fibonacci-crac-from-checkpoint \
  bellsoft/liberica-runtime-container:jdk-21-crac-slim-glibc \
  java -XX:CRaCRestoreFrom=/crac/checkpoint
Like before, if everything goes smoothly, we should be able to verify this from the application log:
2024-04-22T16:02:21.582Z  INFO 129 --- [Attach Listener] 
  o.s.c.support.DefaultLifecycleProcessor : 
  Spring-managed lifecycle restart completed (restored JVM running for 1494 ms)
```

可以看到，应用程序已恢复到创建此检查点时的状态，我们可以注意到恢复速度明显加快，但对于这个简单的应用程序来说，恢复速度可能不太明显。

### 5.5 结果概述

正如我们在采取检查点之前所做的那样，让我们在恢复之后再次检查RSS的大小，最好是在向应用程序发出几个请求之后：

```shell
$ docker exec fibonacci-crac-from-checkpoint pmap -x 129 | tail -1
total            5044580  120261   62728       0
```

我们可以看到，该值(120261)小于我们在检查点之前注意到的值。不过，对于我们在本教程中使用的应用程序而言，这种情况并不那么明显。

我们可能还会注意到，恢复后的RSS在第一个请求后有所增加，然后达到某个稳定状态。但是，这个值仍然低于我们在进行应用程序转储之前观察到的RSS。

RSS的减少主要归因于Liberica JDK和CRaC在检查点之前执行了完整的垃圾收集，在恢复时，HotSpot虚拟机会将部分本机内存(包括在GC期间释放的页面)返回给操作系统。

## 6. CRaC与GraalVM原生镜像

我们讨论过的Java问题自其诞生之初就一直存在，但直到最近，我们才对云服务提出了尽可能经济高效的严格要求，**实现这一目标的关键因素之一是“Scale-to-Zero”(缩放至零)，这意味着在不使用时自动将所有资源缩减至零**。

当然，这要求我们的应用程序能够快速启动并响应请求。因此，在CRaC之前，也提出了一些解决方案来满足这一需求。其中，[GraalVM Native Image](https://www.graalvm.org/latest/reference-manual/native-image/)解决了更广泛的问题，包括启动时间缓慢的问题。

因此，值得将CRaC与GraalVM Native Image进行比较。**GraalVM Native Image是一个预先(AOT)编译器，可为Linux、Windows和macOS创建原生可执行文件**，BellSoft提供了一个[Liberica Native Image Kit](https://bell-sw.com/liberica-native-image-kit/?utm_source=baeldung&utm_medium=ref_article&utm_campaign=baeldung&utm_content=openjdk_crac)，用于基于GraalVM生成原生镜像：

![](/assets/images/2025/javajvm/openjdkcrachotapplicationinstancesscaling03.png)

与CRaC类似，GraalVM Native Image可以显著缩短启动时间，但**GraalVM在内存占用、安全性和应用程序文件大小方面更胜一筹**。此外，我们可以为多种操作系统生成GraalVM Native Image。

但是，使用GraalVM，我们无法使用某些Java特性，例如在运行时加载任意类。此外，**许多可观察性和测试框架也不支持GraalVM**，因为它不支持在运行时动态生成代码，也无法运行Java代理。

那么，CRaC和GraalVM原生镜像哪个更好呢？嗯，两种技术都有各自的应用领域，GraalVM原生镜像解决了与CRaC相同的问题，但限制更多，并且可能带来更昂贵的故障排除体验。

## 7. 总结

在本教程中，我们了解了什么是CRaC，以及如何在云原生环境中充分利用它。此外，我们还了解了BellSoft的支持CRaC的产品，例如Alpaquita Containers。最后，我们开发了一个Spring Boot应用程序，并体验了CRaC的实际应用。