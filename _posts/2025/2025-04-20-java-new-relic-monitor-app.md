---
layout: post
title:  使用New Relic监控Java应用程序
category: libraries
copyright: libraries
excerpt: New Relic
---

## 1. 概述

在本教程中，我们将探索如何使用[New Relic](https://newrelic.com/)监控Java应用程序。New Relic是一个强大的可观察性平台，可以实时洞察应用程序的性能，它可以监控响应时间、吞吐量、错误率等。

为了探索New Relic的功能，我们将构建和监控一个简单的Spring Boot应用程序。

## 2. 项目设置

让我们首先设置一个[Spring Boot应用程序](https://www.baeldung.com/spring-boot-start)并添加一些REST端点。

### 2.1 REST控制器

我们将创建一个[REST控制器](https://www.baeldung.com/spring-controller-vs-restcontroller#spring-mvc-rest-controller)并公开两个端点：

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, New Relic!";
    }

    @GetMapping("/error")
    public String error() {
        throw new RuntimeException("An error occurred");
    }
}
```

此控制器公开了一个返回简单消息的端点/hello和一个抛出异常的端点/error，我们将使用这些端点来演示New Relic如何监控应用程序的性能和错误。

## 3. 安装New Relic

要使用New Relic监控我们的Java应用程序，我们必须[注册](https://newrelic.com/signup)一个帐户并配置New Relic Java代理。注册是免费的，之后我们可以根据需求选择一个方案。

注册后，我们就可以开始[在本地设置我们的代理](https://www.baeldung.com/java-new-relic-introduction#setup)。

### 3.1 Maven依赖

首先，我们应该将[newrelic-java](https://mvnrepository.com/artifact/com.newrelic.agent.java/newrelic-java) Maven依赖作为zip文件包含进去：

```xml
<dependency>
    <groupId>com.newrelic.agent.java</groupId>
    <artifactId>newrelic-java</artifactId>
    <version>8.18.0</version>
    <scope>provided</scope>
    <type>zip</type>
</dependency>
```

该zip文件包含代理JAR文件及其运行所需的文件，provided的作用域确保New Relic代理在运行时可用，但不会包含在应用程序的JAR文件中。

### 3.2 将unpack-dependencies目标添加到Maven Dependency插件

**我们还必须配置Maven依赖插件，以便在应用程序打包时解压ZIP文件**。

为此，我们将unpack-dependencies目标添加到Maven Dependency Plugin中，以解压缩New Relic Java代理：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
        <execution>
            <id>unpack-newrelic</id>
            <phase>package</phase>
            <goals>
                <goal>unpack-dependencies</goal>
            </goals>
            <configuration>
                <includeGroupIds>com.newrelic.agent.java</includeGroupIds>
                <includeArtifactIds>newrelic-java</includeArtifactIds>
                <!-- you can optionally exclude files -->
                <excludes>**/newrelic.yml</excludes>
                <overWriteReleases>false</overWriteReleases>
                <overWriteSnapshots>false</overWriteSnapshots>
                <overWriteIfNewer>true</overWriteIfNewer>
                <outputDirectory>${project.build.directory}</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

当我们构建项目时，Maven会下载New Relic Java代理，当我们打包应用程序时，代理会被解压到/target目录中。

这将使代理在运行时在我们的类路径上可用。

### 3.3 配置代理

要自定义代理以使用我们的帐户和配置，我们可以下载newrelic.yml文件的模板并进行编辑，其中最重要的两个条目是app_name和license_key，**app_name是将根据此应用程序的指标显示在New Relic仪表板上的名称，license_key可以通过[New Relic API Keys](https://one.newrelic.com/launcher/api-keys-ui.api-keys-launcher)创建/获取**。

让我们看一下所需的最小newrelic.yml配置文件：

```yaml
common: &default_settings
    license_key: 'YOUR_LICENSE_KEY'
    app_name: 'NewRelicApplication'
```

New Relic还为上述步骤提供了[引导安装向导](https://docs.newrelic.com/install/java/)。

## 4. 运行应用程序

现在依赖已可用，我们将代理添加到类路径，以便使用New Relic监控应用程序，我们还将向API发出一些请求并检查仪表板。

### 4.1 启动代理

我们可以在运行应用程序时使用-javaagent JVM选项。

**此外，如果JAR和配置文件不在同一个目录中，我们必须使用-Dnewrelic.config.file选项提供配置文件的路径**。

让我们启动我们的应用程序并使用New Relic对其进行检测：

```shell
java -javaagent:path/to/newrelic.jar -Dnewrelic.config.file=path/to/newrelic.yml -jar target/my-app.jar
```

### 4.2 发出请求

具体来说，我们将使用任何浏览器或客户端调用/hello和/error端点来生成监控数据。

例如，我们可以使用curl命令发送请求：


```shell
curl http://localhost:8080/hello
curl http://localhost:8080/error
```

我们可以多次访问这些端点来产生一些流量，代理会捕获这些请求，包括服务器如何执行这些请求以及何时接收这些请求。

## 5. 监控仪表板

一旦应用程序运行并收到流量，我们就可以导航到[New Relic仪表板](https://one.newrelic.com/)来查看捕获的指标和洞察。**仪表板位于我们的个人资料下，位于与我们的许可证密钥关联的应用程序列表中**。

接下来，我们来看一些New Relic提供的指标示例。

### 5.1 应用概述

**我们可以选择“APM & Services > Summary”选项卡来概览应用程序的性能**，这包括响应时间、吞吐量、错误率等指标：

![](/assets/images/2025/libraries/javanewrelicmonitorapp01.png)

在这里，我们可以看到“Web transaction time”、“Throughput”和“Error Rate”。

### 5.2 深入研究API性能

我们还可以深入研究特定请求以查看每个端点的详细指标。

如果我们向下滚动，我们可以看到列出的事务：

![](/assets/images/2025/libraries/javanewrelicmonitorapp02.png)

点击其中任何一个还将提供更详细的指标：

![](/assets/images/2025/libraries/javanewrelicmonitorapp03.png)

最后，这有助于识别应用程序中的瓶颈并根据每个路由优化性能。

### 5.3 JVM指标

另一个重要方面是监视JVM指标，**此外，我们可以在“JVM”选项卡下查看JVM指标，例如CPU利用率、内存使用情况、堆内存、非堆内存和垃圾收集时间**：

![](/assets/images/2025/libraries/javanewrelicmonitorapp04.png)

我们还可以查看JVM线程的使用情况：

![](/assets/images/2025/libraries/javanewrelicmonitorapp05.png)

这些指标特别有助于我们了解应用程序如何利用系统资源并识别任何潜在问题。

## 6. 总结

在本教程中，我们探索了如何使用New Relic监控Java应用程序。我们搭建了一个简单的Spring Boot应用程序，配置了New Relic Java代理，并在New Relic仪表板上查看了各项指标。

New Relic提供了一个强大的平台，用于监控和优化Java应用程序的性能。最终，通过利用这些洞察，我们可以识别瓶颈、优化性能并确保应用程序的可靠性。