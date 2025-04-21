---
layout: post
title:  Apache Camel简介
category: messaging
copyright: messaging
excerpt: Apache Camel
---

## 1. 概述

在本文中，我们将**介绍Apache Camel并探索其核心概念之一-消息路由**。

我们将首先介绍一些基础概念和术语，然后介绍两种定义路由的选项-Java DSL和Spring DSL。

我们还将通过一个示例来演示这些内容，即定义一个路由，该路由使用一个文件夹中的文件并将它们移动到另一个文件夹中，同时在每个文件名前面添加时间戳。

## 2. 关于Apache Camel

**[Apache Camel](https://camel.apache.org/)是一个开源集成框架，旨在使系统集成变得简单和容易**。

它允许最终用户使用相同的API集成各种系统，提供对多种协议和数据类型的支持，同时具有可扩展性并允许引入自定义协议。

## 3. Maven依赖

让我们首先在pom.xml中声明[camel-spring-boot-starter](https://mvnrepository.com/artifact/org.apache.camel.springboot/camel-spring-boot-starter)和[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)依赖：

```xml
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
    <artifactId>camel-spring-boot-starter</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.11</version>
</dependency>
```

此外，为了进行测试，我们需要将[camel-test-spring-junit5](https://mvnrepository.com/artifact/org.apache.camel/camel-test-spring-junit5)和[awaitility](https://mvnrepository.com/artifact/org.awaitility/awaitility)依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring-junit5</artifactId>
    <version>4.3.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>4.2.0</version>
    <scope>test</scope>
</dependency>
```

## 4. 领域特定语言

**路由和路由引擎是Camel的核心部分**，路由包含了不同系统之间集成的流程和逻辑。

为了更简单、更清晰地定义路由，Camel为Java或Groovy等编程语言提供了几种不同的领域特定语言(DSL)。另一方面，它还提供了使用Spring DSL在XML中定义路由的功能。

使用Java DSL或Spring DSL主要是用户偏好，因为大多数功能在两者中都可用。

Java DSL提供了一些Spring DSL不支持的特性，但是，Spring DSL有时更有优势，因为无需重新编译代码即可更改XML。

## 5. 术语和架构

现在让我们讨论一下Camel的基本术语和架构。

首先，我们来看看Camel的核心概念：

-   **Message**包含正在传输到路由的数据，每条消息都有一个唯一标识符，由正文、标头和附件组成。
-   **Exchange**是消息的容器，在路由过程中，当消息被消费者接收时，Exchange就会被创建，Exchange允许系统之间进行不同类型的交互-它可以定义单向消息或请求-响应消息。
-   **Endpoint**是系统可以接收或发送消息的通道，它可以指Web服务URI、队列URI、文件、电子邮件地址等。
-   **Component**充当端点工厂，简而言之，组件使用相同的方法和语法为不同的技术提供接口。Camel在其DSL中已经支持[几乎所有](http://camel.apache.org/components.html)可能的技术，并且还提供了编写自定义组件的能力。
-   **Processor**是一个简单的Java接口，用于向路由添加自定义集成逻辑，它包含一个process方法，用于对消费者收到的消息执行自定义业务逻辑。

从高层次来看，Camel的架构很简单，**CamelContext代表Camel运行时系统，它连接了不同的概念，例如路由、组件或端点**。

在其之下，**处理器处理端点之间的路由和转换，同时端点集成不同的系统**。

## 6. 定义路由

**可以使用Java DSL或Spring DSL定义路由**。

我们将通过定义一个路由来说明这两种风格，该路由使用一个文件夹中的文件并将它们移动到另一个文件夹，同时为每个文件名添加时间戳。

### 6.1 使用Java DSL进行路由

要使用Java DSL定义路由，我们首先需要创建一个DefaultCamelContext实例。然后，我们需要扩展RouteBuilder类并实现configure方法，该方法将包含路由流程：

```java
private static final long DURATION_MILIS = 10000;
private static final String SOURCE_FOLDER = "src/test/source-folder";
private static final String DESTINATION_FOLDER = "src/test/destination-folder";

@Test
public void givenJavaDSLRoute_whenCamelStart_thenMoveFolderContent() throws Exception {
    CamelContext camelContext = new DefaultCamelContext();
    camelContext.addRoutes(new RouteBuilder() {
        @Override
        public void configure() throws Exception {
            from("file://" + SOURCE_FOLDER + "?delete=true")
                    .process(new FileProcessor())
                    .to("file://" + DESTINATION_FOLDER);
        }
    });
    camelContext.start();
    Date date = new Date();
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    File destinationFile1 = new File(DESTINATION_FOLDER + "/" + dateFormat.format(date) + "File1.txt");
    File destinationFile2 = new File(DESTINATION_FOLDER + "/" + dateFormat.format(date) + "File2.txt");

    Awaitility.await().atMost(DURATION_MILIS, TimeUnit.MILLISECONDS).untilAsserted(() -> {
        assertThat(destinationFile1.exists()).isTrue();
        assertThat(destinationFile2.exists()).isTrue();
    });
    camelContext.stop();
}
```

configure方法可以这样理解：从源文件夹中读取文件，使用FileProcessor处理，并将结果发送到目标文件夹。设置delete=true表示处理成功后，文件将从源文件夹中删除。

**为了启动Camel，我们需要调用CamelContext的start方法**。之后，我们使用await()方法(Awaitility类的静态方法之一)来为Camel预留将文件从一个文件夹移动到另一个文件夹所需的时间。atMost()方法设置了Awaitility等待条件满足的上限，在本例中，它将等待最多DURATION_MILIS毫秒。

此外，我们可以使用untilAsserted()方法，此方法允许我们提供包含一个或多个断言的Lambda表达式或方法引用。Awaitility将重复运行这些断言，直到它们全部通过或达到指定的atMost()时长。这里，使用assertThat()方法进行了两个断言，在这些断言中，我们检查目标文件夹中文件是否存在。

FileProcessor实现了Processor接口，并包含一个包含修改文件名逻辑的process方法：

```java
@Component
public class FileProcessor implements Processor {
    @Override
    public void process(Exchange exchange) throws Exception {
        String originalFileName = (String) exchange.getIn().getHeader(Exchange.FILE_NAME, String.class);

        Date date = new Date();
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        String changedFileName = dateFormat.format(date) + originalFileName;
        exchange.getIn().setHeader(Exchange.FILE_NAME, changedFileName);
    }
}
```

为了检索文件名，我们必须从交换机检索传入消息并访问其标头。同样，要修改文件名，我们必须更新消息头。

### 6.2 使用Spring DSL进行路由

**当使用Spring DSL定义路由时，我们使用XML文件来设置路由和处理器**，这使我们能够使用Spring无需任何代码即可配置路由，并最终带来完全控制反转的好处。

[现有文章](https://www.baeldung.com/spring-apache-camel-tutorial)中已经涵盖了这一点，因此我们将重点介绍如何使用Spring DSL和Java DSL，这通常是定义路由的首选方式。

在这种安排下，CamelContext在Spring XML文件中使用Camel的自定义XML语法进行定义，但没有像使用XML的“纯”Spring DSL那样的路由定义：

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <routeBuilder ref="fileRouter" />
</camelContext>
```

这样，我们告诉Camel使用FileRouter类，该类在Java DSL中包含了我们的路由定义：

```java
@Component
public class FileRouter extends RouteBuilder {

    private static final String SOURCE_FOLDER =  "src/test/source-folder";
    private static final String DESTINATION_FOLDER = "src/test/destination-folder";

    @Override
    public void configure() throws Exception {
        from("file://" + SOURCE_FOLDER + "?delete=true")
                .process(new FileProcessor())
                .to("file://" + DESTINATION_FOLDER);
    }
}
```

为了对此进行测试，我们必须创建一个ClassPathXmlApplicationContext实例，它将在Spring中加载我们的CamelContext：

```java
@Test
public void givenSpringDSLRoute_whenCamelStart_thenMoveFolderContent() throws Exception {
    ClassPathXmlApplicationContext applicationContext =
            new ClassPathXmlApplicationContext("camel-context-test.xml");

    Date date = new Date();
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    File destinationFile1 = new File(DESTINATION_FOLDER + "/" + dateFormat.format(date) + "File1.txt");
    File destinationFile2 = new File(DESTINATION_FOLDER + "/" + dateFormat.format(date) + "File2.txt");

    Awaitility.await().atMost(DURATION_MILIS, TimeUnit.MILLISECONDS).untilAsserted(() -> {
        assertThat(destinationFile1.exists()).isTrue();
        assertThat(destinationFile2.exists()).isTrue();
    });
    applicationContext.close();
}
```

通过使用这种方法，我们可以获得Spring提供的额外灵活性和好处，以及通过使用Java DSL获得Java语言的所有可能性。

## 7. 总结

在这篇简短的文章中，我们介绍了Apache Camel，并展示了使用Camel执行集成任务的好处，例如将文件从一个文件夹路由到另一个文件夹。