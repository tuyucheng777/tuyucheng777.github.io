---
layout: post
title:  在Quarkus应用程序上配置服务器端口
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

在本快速教程中，我们将学习如何在Quarkus应用程序上配置服务器端口。

## 2. 配置端口

默认情况下，与许多其他Java服务器应用程序类似，Quarkus监听端口8080。为了更改默认服务器端口，我们可以使用quarkus.http.port属性。

Quarkus从[各种来源](https://quarkus.io/guides/config-reference#configuration-sources)读取其配置属性，因此，我们也可以从不同的来源更改quarkus.http.port属性。例如，我们可以将其添加到application.properties中，让Quarkus监听端口9000：

```properties
quarkus.http.port=9000
```

现在，如果我们向localhost:9000发送请求，服务器将返回某种HTTP响应：

```shell
>> curl localhost:9000/hello
Hello RESTEasy
```

也可以通过Java系统属性和-D参数来配置端口：

```shell
>> mvn compile quarkus:dev -Dquarkus.http.port=9000
// omitted
Listening on: http://localhost:9000
```

如上所示，系统属性在这里覆盖了默认端口。除此之外，我们还可以使用环境变量：

```shell
>> QUARKUS_HTTP_PORT=9000 
>> mvn compile quarkus:dev
```

在这里，我们将quarkus.http.port中的所有字符转换为大写，并用下划线替换点以形成[环境变量名称](https://github.com/eclipse/microprofile-config/blob/master/spec/src/main/asciidoc/configsources.asciidoc#default-configsources)。

## 3. 总结

在这个简短的教程中，我们看到了在Quarkus中配置应用程序端口的几种方法。