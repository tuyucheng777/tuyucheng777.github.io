---
layout: post
title:  使用Logback发送电子邮件
category: log
copyright: log
excerpt: Logback
---

## 1. 概述

[Logback](https://www.baeldung.com/logback)是Java应用程序最流行的日志框架之一，它内置了对高级过滤、存档和删除旧日志文件以及通过电子邮件发送日志消息的支持。

在本快速教程中，**我们将配置Logback以便在发生任何应用程序错误时发送电子邮件通知**。

## 2. 设置

Logback的电子邮件通知功能需要使用SMTPAppender，SMTPAppender使用Jakarta Mail API，而后者又依赖于Java Beans Activation Framework(JAF)。

让我们在POM中添加这些依赖：

```xml
<dependency>
    <groupId>org.eclipse.angus</groupId>
    <artifactId>angus-mail</artifactId>
    <version>2.0.1</version>
</dependency>
<dependency>
    <groupId>org.eclipse.angus</groupId>
    <artifactId>angus-activation</artifactId>
    <version>2.0.0</version>
    <scope>runtime</scope>
</dependency>
```

[Angus Mail](https://eclipse-ee4j.github.io/angus-mail/)是[Jakarta Mail API](https://github.com/jakartaee/mail-api)规范的Eclipse实现。

可以在Maven Central上找到[Java Mail API](https://mvnrepository.com/artifact/org.eclipse.angus/angus-mail)和[Java Beans Activation Framework](https://mvnrepository.com/artifact/org.eclipse.angus/angus-activation)(即[Angus Activation](https://eclipse-ee4j.github.io/angus-activation/))的最新版本。

## 3. 配置SMTPAppender

**默认情况下，Logback的SMTPAppender在记录ERROR事件时会触发电子邮件**。

它会将所有日志事件保存在一个循环缓冲区中，默认最大容量为256个事件。缓冲区满了之后，它会丢弃所有较旧的日志事件。

让我们在logback.xml中配置一个SMTPAppender：

```xml
<appender name="emailAppender" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>OUR-SMTP-HOST-ADDRESS</smtpHost>
    <!-- one or more recipients are possible -->
    <to>EMAIL-RECIPIENT-1</to>
    <to>EMAIL-RECIPIENT-2</to>
    <from>SENDER-EMAIL-ADDRESS</from>
    <subject>TUYUCHENG: %logger{20} - %msg</subject>
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
    </layout>
</appender>
```

此外，我们将把这个附加器添加到我们的Logback配置的root元素中：

```xml
<root level="INFO">
    <appender-ref ref="emailAppender"/>
</root>
```

**因此，对于记录的任何应用程序错误，它都会发送一封电子邮件**，其中包含由PatternLayout格式化的所有缓冲日志事件。

我们可以进一步用HTMLLayout替换PatternLayout，以在HTML表中格式化日志消息：

![](/assets/images/2025/log/logbacksendemail01.png)

## 4. 自定义缓冲区大小

我们现在知道，**默认情况下，发送的电子邮件将包含最后256条日志事件消息**。但是，我们可以通过包含cyclicBufferTracker配置并指定所需的bufferSize来自定义此行为。

为了触发仅包含最新5个日志事件的电子邮件通知，我们将：

```xml
<appender name="emailAppender" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>OUR-SMTP-HOST-ADDRESS</smtpHost>
    <to>EMAIL-RECIPIENT</to>
    <from>SENDER-EMAIL-ADDRESS</from>
    <subject>TUYUCHENG: %logger{20} - %msg</subject>
    <layout class="ch.qos.logback.classic.html.HTMLLayout"/>
    <cyclicBufferTracker class="ch.qos.logback.core.spi.CyclicBufferTracker">
        <bufferSize>5</bufferSize>
    </cyclicBufferTracker>
</appender>
```

## 5. Gmail的SMTPAppender

如果我们使用Gmail作为SMTP提供商，我们将必须通过SSL或STARTTLS进行身份验证。

**要通过STARTTLS建立连接，客户端首先向服务器发出STARTTLS命令，如果服务器支持此通信，则连接将切换到SSL**。

现在让我们使用STARTTLS为Gmail配置附加器：

```xml
<appender name="emailAppender" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>smtp.gmail.com</smtpHost>
    <smtpPort>587</smtpPort>
    <STARTTLS>true</STARTTLS>
    <asynchronousSending>false</asynchronousSending>
    <username>SENDER-EMAIL@gmail.com</username>
    <password>GMAIL-ACCT-PASSWORD</password>
    <to>EMAIL-RECIPIENT</to>
    <from>SENDER-EMAIL@gmail.com</from>
    <subject>TUYUCHENG: %logger{20} - %msg</subject>
    <layout class="ch.qos.logback.classic.html.HTMLLayout"/>
</appender>
```

## 6. 总结

在本文中，我们探讨了如何配置Logback的SMTPAppender以便在应用程序出现错误时发送电子邮件。