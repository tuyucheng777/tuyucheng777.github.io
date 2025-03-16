---
layout: post
title:  如何使用Java检测用户名
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 概述

有时，在使用Java应用程序时，我们需要访问系统属性和环境变量的值。

在本教程中，我们将学习如何从正在运行的Java应用程序中检索用户名。

## 2. System.getProperty

获取用户信息(更准确地说是其名称)的一种方法是使用System.getProperty(String)，此方法接收键。**通常，它们是[统一且预定义](https://docs.oracle.com/javase/tutorial/essential/environment/sysprop.html)的，例如java.version、[os.name](https://www.baeldung.com/java-detect-os)、user.home等**。在我们的例子中，我们对user.name感兴趣：

```java
String username = System.getProperty("user.name");
System.out.println("User: " + username);
```

此方法的重载版本.getProperty(String，String)可以保护我们免受不存在的属性的影响，它采用默认值。

```java
String customProperty = System.getProperty("non-existent-property", "default value");
System.out.println("Custom property: " + customProperty);
```

除了使用预定义的系统属性之外，此方法还允许我们获取使用[-D前缀](https://www.baeldung.com/java-lang-system#5-accessing-runtime-properties)传递的自定义属性的值。如果我们使用以下命令运行应用程序：

```shell
java -jar app.jar -Dcustom.prop=`Hello World!`
```

在应用程序内部，我们可以使用此值

```java
String customProperty = System.getProperty("custom.prop");
System.out.println("Custom property: " + customProperty);
```

这可以帮助我们通过在启动时传递值来创建灵活且可扩展的代码库。

此外，也可以使用System.setProperty，但更改关键属性可能会对系统产生不可预测的影响。

## 3. System.getenv

**另外，我们可以使用[环境变量](https://www.baeldung.com/java-lang-system#env-props)来获取用户名**，通常可以通过USERNAME或USER找到它：

```java
String username = System.getenv("USERNAME");
System.out.println("User: " + username);
```

环境变量是只读的，但也提供了一种获取有关应用程序运行的系统信息的极好的机制。

## 4. 总结

在本文中，我们讨论了几种获取所需环境信息的方法，环境变量和属性是获取有关系统的更多信息的便捷方法。此外，它们还允许自定义变量使应用程序更加灵活。