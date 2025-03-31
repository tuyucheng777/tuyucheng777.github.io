---
layout: post
title:  Hazelcast简介
category: libraries
copyright: libraries
excerpt: Hazelcast
---

## 1. 概述

这是一篇关于Hazelcast的介绍性文章，我们将了解如何创建集群成员、分布式Map以在集群节点之间共享数据，以及如何创建Java客户端来连接和查询集群中的数据。

## 2. 什么是Hazelcast？

Hazelcast是一个适用于Java的分布式内存数据网格平台，该架构支持集群环境中的高可扩展性和数据分布，它支持节点的自动发现和智能同步。

Hazelcast有[不同的版本](http://docs.hazelcast.org/docs/latest/manual/html-single/index.html#hazelcast-editions.)。在本教程中，我们将使用开源版本。

同样，Hazelcast提供各种功能，例如分布式数据结构、分布式计算、分布式查询等。出于本文的目的，我们将重点介绍分布式Map。

## 3. Maven依赖

Hazelcast提供了许多不同的库来满足各种需求，我们可以在Maven Central的[com.hazelcast](https://mvnrepository.com/artifact/com.hazelcast)组下找到它们。

但是，在本文中，我们将仅使用创建独立Hazelcast集群成员和Hazelcast Java客户端所需的核心依赖：

```xml
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast</artifactId>
    <version>4.0.2</version>
</dependency>

```

当前版本可在[Maven中央仓库](https://mvnrepository.com/artifact/com.hazelcast/hazelcast)中找到。

## 4. 第一个Hazelcast应用程序

### 4.1 创建Hazelcast成员

成员(也称为节点)自动连接在一起形成集群，这种自动连接通过成员用来相互查找的各种发现机制进行。

让我们创建一个在Hazelcast分布式Map中存储数据的成员：

```java
public class ServerNode {
    
    HazelcastInstance hzInstance = Hazelcast.newHazelcastInstance();
    // ...
}
```

当我们启动ServerNode应用程序时，我们可以在控制台中看到流动的文本，这意味着我们在JVM中创建了一个新的Hazelcast节点，该节点必须加入集群。

```text
Members [1] {
    Member [192.168.1.105]:5701 - 899898be-b8aa-49aa-8d28-40917ccba56c this
}
```

要创建多个节点，我们可以启动ServerNode应用程序的多个实例。这样，Hazelcast将自动创建并向集群添加新成员。

例如，如果我们再次运行ServerNode应用程序，我们将在控制台中看到以下日志，表明集群中有两个成员。

```text
Members [2] {
    Member [192.168.1.105]:5701 - 899898be-b8aa-49aa-8d28-40917ccba56c
    Member [192.168.1.105]:5702 - d6b81800-2c78-4055-8a5f-7f5b65d49f30 this
}
```

### 4.2 创建分布式Map

接下来，让我们创建一个分布式Map。我们需要之前创建的HazelcastInstance实例来构造一个扩展java.util.concurrent.ConcurrentMap接口的分布式Map。

```java
Map<Long, String> map = hazelcastInstance.getMap("data");
// ...
```

最后，让我们向Map添加一些条目：

```java
FlakeIdGenerator idGenerator = hazelcastInstance.getFlakeIdGenerator("newid");
for (int i = 0; i < 10; i++) {
    map.put(idGenerator.newId(), "message" + i);
}
```

如上所示，我们已向Map添加了10个条目。我们使用FlakeIdGenerator来确保获取Map的唯一键，有关FlakeIdGenerator的更多详细信息，我们可以查看以下[链接](https://javadoc.io/doc/com.hazelcast/hazelcast/4.0.2/com/hazelcast/flakeidgen/FlakeIdGenerator.html)。

虽然这可能不是真实世界的示例，但我们仅用它来演示可应用于分布式Map的众多操作之一。稍后，我们将了解如何从Hazelcast Java客户端检索集群成员添加的条目。

在内部，Hazelcast对Map条目进行分区，并在集群成员之间分发和条目。有关Hazelcast Map的更多详细信息，我们可以查看以下[链接](https://docs.hazelcast.org/docs/4.0.2/manual/html-single/index.html#map)。

### 4.3 创建Hazelcast Java客户端

Hazelcast客户端允许我们在不成为集群成员的情况下执行所有Hazelcast操作，它连接到集群成员之一并将所有集群范围的操作委托给它。

让我们创建一个原生客户端：

```java
ClientConfig config = new ClientConfig();
config.setClusterName("dev");
HazelcastInstance hazelcastInstanceClient = HazelcastClient.newHazelcastClient(config);
```

就这么简单。

### 4.4 从Java客户端访问分布式Map

接下来，我们将使用之前创建的HazelcastInstance实例来访问分布式Map：

```java
Map<Long, String> map = hazelcastInstanceClient.getMap("data");
// ...
```

现在我们可以在不成为集群成员的情况下对Map执行操作。例如，让我们尝试迭代条目：

```java
for (Entry<Long, String> entry : map.entrySet()) {
    // ...
}
```

## 5. 配置Hazelcast

在本节中，我们将重点介绍如何使用声明式(XML)和编程式(API)配置Hazelcast网络，并使用Hazelcast管理中心来监控和管理正在运行的节点。

Hazelcast启动时会查找hazelcast.config系统属性，如果已设置，则将其值用作路径。否则，Hazelcast会在工作目录或类路径中搜索hazelcast.xml文件。

如果以上方法均不起作用，Hazelcast将加载默认配置，即hazelcast.jar附带的hazelcast-default.xml。

### 5.1 网络配置

默认情况下，Hazelcast使用多播来发现可以组成集群的其他成员。如果多播不是我们环境中的首选发现方式，那么我们可以为完整的TCP/IP集群配置Hazelcast。

让我们使用声明式配置来配置TCP/IP集群：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hazelcast xmlns="http://www.hazelcast.com/schema/config"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.hazelcast.com/schema/config
                               http://www.hazelcast.com/schema/config/hazelcast-config-4.0.xsd";
    <network>
        <port auto-increment="true" port-count="20">5701</port>
        <join>
            <multicast enabled="false"/>
            <tcp-ip enabled="true">
                <member>machine1</member>
                <member>localhost</member>
            </tcp-ip>
        </join>
    </network>
</hazelcast>
```

或者，我们可以使用Java配置方法：

```java
Config config = new Config();
NetworkConfig network = config.getNetworkConfig();
network.setPort(5701).setPortCount(20);
network.setPortAutoIncrement(true);
JoinConfig join = network.getJoin();
join.getMulticastConfig().setEnabled(false);
join.getTcpIpConfig()
    .addMember("machine1")
    .addMember("localhost").setEnabled(true);
```

默认情况下，Hazelcast将尝试绑定100个端口。在上面的示例中，如果我们将端口值设置为5701，并将端口数限制为20，则当成员加入集群时，Hazelcast会尝试查找5701和5721之间的端口。

如果我们只想选择使用一个端口，我们可以通过将自动增量设置为false来禁用自动增量功能。

### 5.2 管理中心配置

管理中心可以让我们监控集群的整体状态，还可以详细分析和浏览数据结构，更新Map配置，并从节点获取线程转储。

要使用Hazelcast管理中心，我们可以将mancenter-version.war应用程序部署到我们的Java应用程序服务器/容器中，也可以从命令行启动Hazelcast管理中心。我们可以从[hazelcast.org](https://hazelcast.org/imdg/download/)下载最新的Hazelcast ZIP，ZIP包含mancenter-version.war文件。

我们可以通过将Web应用程序的URL添加到hazelcast.xml来配置我们的Hazelcast节点，然后让Hazelcast成员与管理中心进行通信。

现在让我们使用声明式配置来配置管理中心：

```xml
<management-center enabled="true">
    http://localhost:8080/mancenter
</management-center>
```

同样，这是编程式配置：

```java
ManagementCenterConfig manCenterCfg = new ManagementCenterConfig();
manCenterCfg.setEnabled(true).setUrl("http://localhost:8080/mancenter");
```

## 6. 总结

在本文中，我们介绍了有关Hazelcast的入门概念。有关更多详细信息，我们可以查看[参考手册](http://docs.hazelcast.org/docs/3.7/manual/html-single/index.html)。