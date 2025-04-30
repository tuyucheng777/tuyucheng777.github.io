---
layout: post
title:  Gradle代理配置
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

**[代理服务器](https://www.baeldung.com/java-connect-via-proxy-server)充当客户端和服务器之间的中介，它帮助评估来自客户端的请求，然后根据特定标准将其转发到目标服务器**，这使得系统能够灵活地确定要连接或不连接的网络。

在本教程中，我们将学习如何配置[Gradle](https://www.baeldung.com/gradle)使其在代理服务器后工作。在我们的示例中，我们的代理在localhost上运行，代理端口为3128，用于HTTP和HTTPS连接。

## 2. 代理配置

我们可以将Gradle配置为在有或没有身份验证凭据的代理服务器后面工作。

### 2.1 基本代理配置

首先，让我们设置一个不需要身份验证凭据的基本代理配置，**让我们在Gradle项目的根目录中创建一个名为gradle.properties的文件**。

接下来，让我们在gradle.properties文件中定义代理服务器的[系统属性](https://www.baeldung.com/java-connect-via-proxy-server#1-available-system-properties)：

```properties
systemProp.http.proxyHost=localhost
systemProp.http.proxyPort=3128
systemProp.https.proxyHost=localhost
systemProp.https.proxyPort=3128
```

这里我们定义了Gradle在构建过程中将使用的系统属性，我们为HTTP和HTTPS连接定义了系统属性，在本例中，它们具有相同的主机名和代理端口。

另外，我们可以在gradle.properties文件中指定一个主机来绕过代理服务器：

```properties
systemProp.http.nonProxyHosts=*.nonproxyrepos.com
systemProp.https.nonProxyHosts=*.nonproxyrepos.com
```

在上面的配置中，子域名nonproxyrepos.com将绕过代理服务器并直接从服务器请求资源。

或者，我们可以通过终端运行带有系统属性选项的./gradlew build命令：

```shell
$ ./gradlew -Dhttp.proxyHost=localhost -Dhttp.proxyPort=3128 -Dhttps.proxyHost=localhost -Dhttps.proxyPort=3128 build
```

在这里，我们定义系统属性以通过终端连接到代理服务器。

**值得注意的是，通过终端定义系统属性会覆盖gradle.properties文件的配置**。

### 2.2 添加身份验证凭证

在代理安全的情况下，我们可以将身份验证凭据添加到gradle.properties文件：

```properties
systemProp.http.proxyUser=Tuyucheng
systemProp.http.proxyPassword=admin
systemProp.https.proxyUser=Tuyucheng
systemProp.https.proxyPassword=admin
```

在这里，我们通过定义用户名和密码的系统属性来添加身份验证凭据，此外，我们还为HTTP和HTTPS连接实现了身份验证凭据。

或者，我们可以通过终端指定用户名和密码：

```shell
$ ./gradlew -Dhttp.proxyHost=localhost -Dhttp.proxyPort=3128 -Dhttps.proxyHost=localhost -Dhttps.proxyPort=3128 -Dhttps.proxyUser=Tuyucheng -Dhttps.proxyPassword=admin build
```

在这里，我们在终端命令中包含身份验证凭据。

## 3. 可能的错误

**如果主机名和代理端口不正确，则可能会出现错误**：

```text
> Could not get resource 'https://repo.maven.apache.org/maven2/io/micrometer/micrometer-core/1.12.0/micrometer-core-1.12.0.pom'.
> Could not GET 'https://repo.maven.apache.org/maven2/io/micrometer/micrometer-core/1.12.0/micrometer-core-1.12.0.pom'.
> localhosty: Name or service not known
```

这里，构建失败是因为我们错误地将代理主机写为“localhosty”而不是“localhost”。

此外，如果我们在gradle.properties文件和命令行中定义系统属性，则命令行定义在构建期间具有最高优先级：

```shell
$ ./gradlew -Dhttp.proxyHost=localhost -Dhttp.proxyPort=3120 -Dhttps.proxyHost=localhost -Dhttps.proxyPort=3120 build
```

这里，命令行中的代理端口值为3120，这是错误的。gradle.properties文件中的代理端口值为3128，这是正确的。但是，构建失败，并显示以下错误消息：

```text
> Could not get resource 'https://repo.maven.apache.org/maven2/io/micrometer/micrometer-core/1.12.0/micrometer-core-1.12.0.pom'.
> Could not GET 'https://repo.maven.apache.org/maven2/io/micrometer/micrometer-core/1.12.0/micrometer-core-1.12.0.pom'.
> Connect to localhost:3120 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed: Connection refused
```

这里，代理服务器拒绝连接，**因为命令行参数中的代理端口定义错误，尽管gradle.properties文件中的代理端口值是正确的**，命令行参数定义优先于gradle.properties的值。

此外，如果在安全的代理服务器中身份验证凭据错误，代理服务器将拒绝连接。为了避免这些错误，需要正确检查配置。

## 4. 总结

在本文中，我们学习了如何通过在gradle.properties文件中定义所需的系统属性来配置Gradle使其在代理后工作。此外，我们还了解了如何通过终端定义系统属性。最后，我们了解了一些容易犯的错误以及如何避免它们。