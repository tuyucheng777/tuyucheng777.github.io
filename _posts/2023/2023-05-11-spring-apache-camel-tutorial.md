---
layout: post
title:  将Apache Camel与Spring结合使用
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

本文将演示如何通过Spring配置和使用[Apache Camel](https://camel.apache.org/)。

Apache Camel提供了很多有用的组件，支持[JPA](https://camel.apache.org/jpa.html)、[Hibernate](https://people.apache.org/~dkulp/camel/hibernate.html)、[FTP](https://camel.apache.org/ftp2.html)、[Apache-CXF](https://camel.apache.org/cxf.html)、[AWS-S3](https://camel.apache.org/components/3.12.x/aws2-s3-component.html)等库，当然还有许多其他库-所有这些都有助于在两个不同系统之间集成数据。

例如，使用Hibernate和Apache CXF组件，你可以从数据库中提取数据并通过REST API调用将其发送到另一个系统。

在本教程中，我们将介绍一个简单的Camel示例-读取文件并将其内容转换为大写，然后再转换回小写。我们将使用Camel的[文件组件](https://camel.apache.org/file2.html)和Spring 4.2。

以下是该示例的完整详细信息：

1.  从源目录读取文件
2.  使用自定义[处理器](https://camel.apache.org/processor.html)将文件内容转换为大写
3.  将转换后的输出写入目标目录
4.  使用[Camel转换器](https://camel.apache.org/message-translator.html)将文件内容转换为小写
5.  将转换后的输出写入目标目录

## 2. 添加依赖

要将Apache Camel与Spring一起使用，你需要在POM文件中添加以下依赖项：

```xml
<properties>
    <env.camel.version>2.16.1</env.camel.version>
    <env.spring.version>4.2.4.RELEASE</env.spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-core</artifactId>
        <version>${env.camel.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-spring</artifactId>
        <version>${env.camel.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-stream</artifactId>
        <version>${env.camel.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${env.spring.version}</version>
    </dependency>
</dependencies>
```

因此，我们有：

-   camel-core：Apache Camel的主要依赖
-   camel-spring：使我们能够将Camel与Spring一起使用
-   camel-stream：一个可选的依赖项，你可以使用它(例如)在路由运行时在控制台上显示一些消息
-   spring-context：标准的Spring依赖项，在我们的例子中是必需的，因为我们将在Spring上下文中运行Camel路由

## 3. Spring Camel语境

首先，我们将创建Spring配置文件，稍后我们将在其中定义我们的Camel路由。

请注意该文件如何包含所有必需的Apache Camel和Spring命名空间和模式位置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:camel="http://camel.apache.org/schema/spring"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans-4.2.xsd	
          http://camel.apache.org/schema/spring 
          http://camel.apache.org/schema/spring/camel-spring.xsd
          http://www.springframework.org/schema/util 
          http://www.springframework.org/schema/util/spring-util-4.2.xsd">

    <camelContext xmlns="http://camel.apache.org/schema/spring">
        <!-- Add routes here -->
    </camelContext>
</beans>
```

<camelContext\>元素代表Camel上下文，可以将其与Spring应用程序上下文进行比较。现在你的上下文文件已准备好开始定义Camel路由。

### 3.1 使用自定义处理器的Camel路由

接下来我们将编写第一个将文件内容转换为大写的路由。

我们需要定义路由将从中读取数据的来源。这可以是数据库、文件、控制台或任意数量的其他来源。在我们的例子中，它将是文件。

然后我们需要定义将从源读取的数据的处理器。对于这个例子，我们将编写一个自定义处理器类。此类将是一个Spring bean，它将实现标准的Camel [Processor接口](https://camel.apache.org/manual/latest/processor.html)。

处理完数据后，我们需要告诉路由将处理后的数据定向到哪里。同样，这可以是各种各样的输出之一，例如数据库、文件或控制台。在我们的例子中，我们将把它存储在一个文件中。

要设置这些步骤(包括输入、处理器和输出)，请将以下路由添加到Camel上下文文件中：

```xml
<route>
    <from uri="file://data/input" /> <!-- INPUT -->
    <process ref="myFileProcessor" /> <!-- PROCESS -->
    <to uri="file://data/outputUpperCase" /> <!-- OUTPUT -->
</route>
```

此外，我们必须定义myFileProcessor bean：

```xml
<bean id="myFileProcessor" class="org.apache.camel.processor.FileProcessor" />
```

### 3.2 自定义大写处理器

现在我们需要创建我们在bean中定义的自定义文件处理器。它必须实现Camel Processor接口，定义一个单一的process方法，该方法将Exchange对象作为其输入。此对象提供来自输入源的数据的详细信息。

我们的方法必须从Exchange读取消息，将内容大写，然后将新内容设置回Exchange对象：

```java
public class FileProcessor implements Processor {

    public void process(Exchange exchange) throws Exception {
        String originalFileContent = (String) exchange.getIn().getBody(String.class);
        String upperCaseFileContent = originalFileContent.toUpperCase();
        exchange.getIn().setBody(upperCaseFileContent);
    }
}
```

此process方法将针对从源接收的每个输入执行。

### 3.3 小写处理器

现在，我们将向Camel路由添加另一个输出。这一次，我们将把同一个输入文件的数据转换为小写。但是，这一次我们不会使用自定义处理器；我们将使用Apache Camel的[消息转换器功能](https://people.apache.org/~dkulp/camel/message-translator.html)。这是更新的Camel路由：

```xml
<route>
    <from uri="file://data/input" />
    <process ref="myFileProcessor" />
    <to uri="file://data/outputUppperCase" />
    <transform>
        <simple>${body.toLowerCase()}</simple>
    </transform>
    <to uri="file://data/outputLowerCase" />
</route>
```

## 4. 运行应用程序

为了处理我们的路由，我们只需要将Camel上下文文件加载到Spring应用程序上下文中：

```java
ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("camel-context.xml");
```

路由成功运行后，将创建两个文件：一个包含大写内容，一个包含小写内容。

## 5. 总结

如果你正在做集成工作，Apache Camel绝对可以让事情变得更容易。该库提供即插即用组件，可帮助你减少样板代码并专注于处理数据的主要逻辑。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-camel)上获得。