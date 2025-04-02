---
layout: post
title:  Java SE/EE/ME之间的区别
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

**在这个简短的教程中，我们将比较三个不同的Java版本**，我们将了解它们提供的功能及其典型用例。

## 2. Java标准版

让我们从Java Standard Edition(简称Java SE)开始，**此版本提供了Java语言的核心功能**。

Java SE为Java应用程序提供了基本组件：[Java虚拟机、Java运行时环境和Java开发工具包](https://www.baeldung.com/jvm-vs-jre-vs-jdk)。在撰写本文时，最新版本是Java 18。

让我们描述一个Java SE应用程序的简单用例。我们可以使用[OOP概念](https://www.baeldung.com/java-oop)实现业务逻辑，使用[java.net包](https://www.baeldung.com/java-9-http-client)发出HTTP请求，并使用[JDBC](https://www.baeldung.com/java-jdbc)连接到数据库。我们甚至可以使用Swing或AWT显示用户界面。

## 3. Java企业版

**Java EE基于标准版并提供了更多的API**，缩写代表Java Enterprise Edition，但也可以称为Jakarta EE。[它们指的是同一件事](https://www.baeldung.com/java-enterprise-evolution)。

**新的Java EE API允许我们创建更大、可扩展的应用程序**。

通常，Java EE应用程序部署到应用程序服务器，提供了许多与Web相关的API来促进这一点：[WebSocket](https://www.baeldung.com/java-websockets)、[JavaServer Pages](https://www.baeldung.com/jsp)、[JAX-RS](https://www.baeldung.com/jax-rs-spec-and-implementations#inclusion-in-java-ee)等。企业功能还包括与JSON处理、安全性、Java消息服务、[Java Mail](https://www.baeldung.com/java-email)等相关的API。

在Java EE应用程序中，我们可以使用标准API中的所有内容。最重要的是，我们可以使用更先进的技术。

现在让我们看一个Java EE的用例。例如，我们可以创建[Servlet](https://www.baeldung.com/intro-to-servlets)来处理来自用户的HTTP请求，并使用[JavaServer Pages](https://www.baeldung.com/jsp)创建动态UI。我们可以使用JMS生成和消费消息、[发送电子邮件](https://www.baeldung.com/java-email)和[验证用户](https://www.baeldung.com/java-ee-8-security)以确保我们的应用程序安全。此外，我们可以使用[依赖注入机制](https://www.baeldung.com/java-ee-cdi)来使我们的代码更易于维护。

## 4. Java微型版

**Java Micro Edition或Java ME为面向嵌入式和移动设备的应用程序提供API**，这些可以是手机、机顶盒、传感器、打印机等。

**Java ME包括一些Java SE功能，同时提供特定于这些设备的新API**。例如，蓝牙、位置、传感器API等。

**大多数时候，这些小型设备在CPU或内存方面存在资源限制**，我们在使用Java ME时必须考虑这些约束。

有时，我们甚至可能无法使用目标设备来测试我们的代码。SDK可以帮助解决这个问题，因为它提供了模拟器、应用程序分析和监控。

例如，一个简单的Java ME应用程序可以读取温度传感器的值并在HTTP请求中将其连同其位置一起发送。

## 5. 总结

在这篇简短的文章中，我们了解了3个Java版本，并比较了它们各自提供的功能。

Java SE可用于简单的应用程序，这是学习Java的最佳起点。我们可以使用Java EE来创建更复杂、更健壮的应用程序。最后，如果我们想要针对嵌入式和移动设备，我们可以使用Java ME。