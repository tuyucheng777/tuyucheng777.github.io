---
layout: post
title:  JVM参数InitialRAMPercentage、MinRAMPercentage和MaxRAMPercentage
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本教程中，我们将讨论一些可用于设置JVM的RAM百分比的[JVM参数](https://www.baeldung.com/jvm-parameters)。

在Java 8中引入的参数InitialRAMPercentage、MinRAMPercentage和MaxRAMPercentage有助于配置Java应用程序的堆大小。

## 2. -XX:InitialRAMPercentage

InitialRAMPercentage JVM参数允许我们配置Java应用程序的初始堆大小，**它是物理服务器或容器总内存的百分比**，作为double值传递。

例如，如果我们为1GB全内存的物理服务器设置-XX:InitialRAMPercentage=50.0，则初始堆大小将约为500MB(1GB的50%)。

首先，让我们检查一下JVM中InitialRAMPercentage的默认值：

```shell
$ docker run openjdk:8 java -XX:+PrintFlagsFinal -version | grep -E "InitialRAMPercentage"
   double InitialRAMPercentage                      = 1.562500                            {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

然后，让我们将JVM的初始堆大小设置为50%：

```shell
$ docker run -m 1GB openjdk:8 java -XX:InitialRAMPercentage=50.0 -XX:+PrintFlagsFinal -version | grep -E "InitialRAMPercentage"
   double InitialRAMPercentage                     := 50.000000                           {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

重要的是要注意，当我们配置-Xms选项时，JVM会忽略InitialRAMPercentage。

## 3. -XX:MinRAMPercentage

MinRAMPercentage参数与其名称不同，**它允许为使用少量内存(小于200MB)运行的JVM设置最大堆大小**。

首先，我们将探讨MinRAMPercentage的默认值：

```shell
$ docker run openjdk:8 java -XX:+PrintFlagsFinal -version | grep -E "MinRAMPercentage"
   double MinRAMPercentage                      = 50.000000                            {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

然后，让我们使用参数来设置总内存为100MB的JVM的最大堆大小：

```shell
$ docker run -m 100MB openjdk:8 java -XX:MinRAMPercentage=80.0 -XshowSettings:VM -version

VM settings:
    Max. Heap Size (Estimated): 77.38M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

此外，JVM在为小型内存服务器/容器设置最大堆大小时会忽略MaxRAMPercentage参数：

```shell
$ docker run -m 100MB openjdk:8 java -XX:MinRAMPercentage=80.0 -XX:MaxRAMPercentage=50.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 77.38M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

## 4. -XX:MaxRAMPercentage

**MaxRAMPercentage参数允许为使用大量内存(大于200MB)运行的JVM设置最大堆大小**。

首先，让我们探讨一下MaxRAMPercentage的默认值：

```shell
$ docker run openjdk:8 java -XX:+PrintFlagsFinal -version | grep -E "MaxRAMPercentage"
   double MaxRAMPercentage                      = 25.000000                            {product}

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

然后，我们可以使用该参数将总内存为500MB的JVM的最大堆大小设置为60%：

```shell
$ docker run -m 500MB openjdk:8 java -XX:MaxRAMPercentage=60.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 290.00M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

类似地，对于大内存服务器/容器，JVM会忽略MinRAMPercentage参数：

```shell
$ docker run -m 500MB openjdk:8 java -XX:MaxRAMPercentage=60.0 -XX:MinRAMPercentage=30.0 -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 290.00M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
```

## 5. 总结

在这篇简短的文章中，我们讨论了使用JVM参数InitialRAMPercentage、MinRAMPercentage和MaxRAMPercentage来设置JVM将用于堆的RAM百分比。

首先，我们检查了JVM上设置的标志的默认值。然后，我们使用JVM参数设置初始和最大堆大小。