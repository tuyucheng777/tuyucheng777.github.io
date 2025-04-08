---
layout: post
title:  将Jolokia与Spring Boot集成
category: springboot
copyright: springboot
excerpt: Jolokia
---

## 1. 概述

监控Java应用程序的性能和稳定性是一种常见的做法。[Spring Boot](https://www.baeldung.com/spring-boot)通过[Actuators](https://www.baeldung.com/spring-boot-actuators)提供这种监控功能，但是，我们可能需要更细粒度的控制或轻量级的方法来实现相同的目的。

**[Jolokia](https://docs.spring.io/spring-boot/docs/1.2.1.RELEASE/reference/html/production-ready-jmx.html)帮助我们通过API端点监控应用程序，同时提供批量操作和安全机制等功能**。

在本教程中，我们将研究如何将Jolokia集成到Spring Boot应用程序中。

## 2. 设置

现在来看看在我们的应用程序中启用Jolokia所需的步骤。

### 2.1 依赖

首先，我们**需要添加[Jolokia](https://mvnrepository.com/artifact/org.jolokia/jolokia-support-spring/)依赖才能使用Jolokia**，它会添加传递依赖和服务库：

```xml
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-support-spring</artifactId>
    <version>2.2.1</version>
</dependency>
```

### 2.2 启用Jolokia

**默认情况下，Spring BootActuator仅公开有限的端点。要启用Jolokia，我们需要在应用程序的属性文件中进行配置**：

```properties
management.endpoints.web.exposure.include=jolokia
```

现在，添加了依赖并配置了应用程序，当我们启动Spring Boot应用程序时就可以访问Jolokia。

## 3. 访问Jolokia

**Jolokia提供了几个命令来完成各种任务**：

- read：读取MBean属性
- write：设置MBean属性
- exec：在MBean上执行操作
- list：列出可用的MBean及其属性和操作
- search：搜索MBean
- version：版本和服务器信息
- notification：订阅并接收通知

接下来，让我们看看其中一些命令。

### 3.1 version命令

让我们导航到以下端点：

```text
http://localhost:8080/actuator/jolokia/version
```

我们应该得到以下响应：

![](/assets/images/2025/springboot/jolokiaspringbootintegration01.png)

我们可以看到，我们得到了一些有关版本的信息，例如它是否安全，是否有代理等等。

### 3.2 list命令

现在，**让我们通过执行list命令来获取所有可用MBean的列表**：

```text
http://localhost:8080/actuator/jolokia/list
```

我们可以看到可用Bean的列表，以及每个Bean上允许的操作：

![](/assets/images/2025/springboot/jolokiaspringbootintegration02.png)

### 3.3 read命令

**我们将尝试使用read命令获取有关堆内存使用情况的信息**：

```text
http://localhost:8080/actuator/jolokia/read/java.lang:type=Memory/HeapMemoryUsage
```

正如我们在响应中看到的，HeapMemoryUsage属性的不同值是可见的：

![](/assets/images/2025/springboot/jolokiaspringbootintegration03.png)

### 3.4 exec命令

**让我们通过执行exec命令来启动垃圾收集**：

```text
http://localhost:8080/actuator/jolokia/exec/java.lang:type=Memory/gc
```

我们收到了200状态码，这意味着垃圾收集命令已被接受：

![](/assets/images/2025/springboot/jolokiaspringbootintegration04.png)

**获取信息或对任何公开的MBean发起操作都很容易，但是，我们需要确保只有少数具有访问权限的人才能执行预期的操作**。

## 4. 保护Jolokia

正如我们所观察到的，Jolokia默认公开所有JMX资源。这肯定会带来问题，因为所有用户都被允许执行可能危害应用程序的所有操作。为了避免这种情况，我们需要保护端点。

### 4.1 禁用Jolokia端点

在生产服务器上，我们可以通过在应用程序属性中设置以下内容来完全禁用Jolokia：

```properties
management.endpoint.jolokia.enabled=false
```

### 4.2 使用安全规则限制访问

**我们可以通过创建jolokia-access.xml文件并将其放在类路径中来配置安全规则**。

让我们创建一个，以便Jolokia访问由用户的IP地址保护：

```xml
<restrict>
    <remote>
        <host>127.0.0.1</host>
    </remote>
</restrict>
```

当我们尝试访问Jolokia端点时，我们得到了预期的错误：

![](/assets/images/2025/springboot/jolokiaspringbootintegration05.png)

接下来，我们将尝试限制某些命令或操作。例如，我们仅允许read和list命令：

```xml
<commands>
    <command>read</command>
    <command>list</command>
</commands>
```

现在，让我们尝试调用之前尝试过的垃圾收集的exec命令：

![](/assets/images/2025/springboot/jolokiaspringbootintegration06.png)

可以看到，由于我们只允许read和list操作，因此exec命令是被禁止的。

最后，我们还可以允许或拒绝某些MBean操作，这些操作甚至可以覆盖commands部分中的条目。

```xml
<allow>
    <mbean>
        <name>java.lang:type=Memory</name>
        <operation>gc</operation>
    </mbean>
</allow>

<deny>
<mbean>
    <name>jdk.management.jfr:type=FlightRecorder</name>
    <attribute>*</attribute>
    <operation>*</operation>
</mbean>
</deny>
```

我们已允许对Memory MBean执行gc操作。此外，在deny部分，我们已拒绝FlightRecorder MBean上的所有属性和操作。

## 5. 总结

在本文中，我们学习了如何将Jolokia与Spring Boot集成。首先，我们了解了初始设置和一些基本命令。

接下来，我们探讨了如何保护各种MBean的访问和操作，即通过限制IP地址、将命令列入白名单以及允许或拒绝某些MBean及其属性/操作。