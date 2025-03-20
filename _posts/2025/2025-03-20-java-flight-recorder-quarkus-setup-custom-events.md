---
layout: post
title:  在Quarkus中使用Java Flight Recorder(JFR)
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 简介

[Java Flight Recorder(JFR)](https://www.baeldung.com/java-flight-recorder-monitoring)是一款功能强大的JVM工具，用于捕获事件以监控、分析和排除应用程序故障。将JFR与Quarkus集成后，我们可以通过添加特定于Quarkus的事件来详细了解应用程序的行为和性能。

在本文中，我们展示了如何在Quarkus项目中设置JFR、生成自定义事件以及分析数据以有效诊断潜在问题。

## 2. Maven依赖

要在Quarkus项目中启用Java Flight Recorder，我们需要添加[quarkus-jfr](https://mvnrepository.com/artifact/io.quarkus/quarkus-jfr)扩展。此扩展将JFR与Quarkus集成，从而启用自定义Quarkus事件以增强监控和诊断：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jfr</artifactId>
    <version>3.17.3</version>
</dependency>
```

## 3. 项目配置

### 3.1 创建REST端点

我们将从创建一个简单的REST端点开始，此资源将模拟轻量级应用程序交互：

```java
@Path("/hello")
public class JfrResource {
    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello";
    }
}
```

### 3.2 开始JFR

为了确保JFR自动开始录制，我们需要传递适当的JVM参数。在我们的Quarkus应用程序中，我们可以在application.properties文件中或在启动应用程序时作为命令行的一部分执行此操作。

对于开发，让我们添加以下JVM参数：

```shell
quarkus dev -Djvm.args="-XX:StartFlightRecording=name=quarkus,dumponexit=true,filename=myrecording.jfr"
```

除了在应用程序运行时自动启动JFR之外，在应用程序退出时将记录转储到myrecording.jfr。

## 4. 保存JFR事件

默认情况下，当应用程序停止时，JFR会将其记录转储到指定文件(myrecording.jfr)。我们可以通过在终端中按CTRL + C或使用[jcmd](https://docs.oracle.com/en/java/javase/23/docs/specs/man/jcmd.html)命令来触发此功能：

```shell
jcmd <PID> JFR.dump name=quarkus filename=myrecording.jfr
```

要查找应用程序的<PID\>，我们可以使用jcmd命令，这将列出所有正在运行的Java进程及其进程ID。

## 5. 打开JFR转储文件

一旦我们使用JFR记录了应用程序事件，分析转储文件对于发现有价值的见解至关重要。我们可以使用两个主要工具：用于快速命令行分析的[JFR](https://docs.oracle.com/en/java/javase/21/docs/specs/man/jfr.html) CLI工具和用于详细图形视图的[JDK Mission Control(JMC)](https://www.oracle.com/java/technologies/jdk-mission-control.html)。

### 5.1 使用JFR CLI

JFR CLI是一个轻量级工具，可直接从终端进行快速分析。要检查JFR文件的内容，我们将使用jfr print命令：

```shell
jfr print myrecording.jfr
```

运行命令后，输出列出事件、其时间戳和相关详细信息：

```text
Event: io.quarkus.HttpRequest
  Method: GET
  Path: /hello
  ResponseCode: 200
  Duration: 3ms

Event: jdk.GarbageCollection
  StartTime: 2024-11-10T14:23:45.123Z
  Duration: 5ms
  Cause: System.gc()
```

为了关注特定的事件类别，我们可以使用–categories标志：

```shell
jfr print --categories "quarkus" myrecording.jfr
```

此命令过滤输出以仅显示与Quarkus相关的事件，从而更容易分析相关数据。

### 5.2 使用JDK Mission Control(JMC)

[JMC](https://www.oracle.com/java/technologies/jdk-mission-control.html)提供了一个图形界面，其中包含强大的可视化工具，可用于深入分析。我们可以使用终端或系统的GUI启动JMC，以打开JDK Mission Control。启动JMC后，可以通过File > Open File menu直接打开录制的文件(myrecording.jfr)。加载后，JMC会将捕获的数据组织成直观的视图和类别：

![](/assets/images/2025/quarkus/javaflightrecorderquarkussetupcustomevents01.png)

JMC中的事件浏览器允许我们探索特定事件，包括HTTP请求、内存使用情况和垃圾收集，Quarkus特定事件(例如io.quarkus.HttpRequest)归入其各自的类别。时间线视图有助于关联随时间变化的事件，从而更轻松地识别模式和性能瓶颈。

## 6. 添加自定义事件

除了Quarkus和JVM提供的内置事件外，我们还可以定义自定义事件来监视应用程序的特定部分，这在跟踪特定领域的行为或性能指标时特别有用。

要定义自定义事件，请扩展jdk.jfr.Event类并用@Name标注它以指定其事件名称：

```java
@Name("cn.tuyucheng.taketoday.DatabaseQueryEvent")
public class DatabaseQueryEvent extends Event {
    private final String query;
    private final long executionTime;

    public DatabaseQueryEvent(String query, long executionTime) {
        this.query = query;
        this.executionTime = executionTime;
    }

    public String getQuery() {
        return query;
    }

    public long getExecutionTime() {
        return executionTime;
    }
}
```

我们可以在应用程序代码中实例化并提交自定义事件，例如，记录数据库查询执行时间：

```java
DatabaseQueryEvent event = new DatabaseQueryEvent("SELECT * FROM users", 15);
event.commit();
```

现在，运行我们的程序后，我们可以查看自定义创建的事件：

```shell
jfr print --events DatabaseQueryEvent myrecording.jfr
```

## 7. 总结

Java Flight Recorder与Quarkus结合使用，是用于监控和优化应用程序性能的强大工具。通过使用内置和自定义事件，我们可以深入了解应用程序行为、诊断问题并提高效率。