---
layout: post
title:  使用SLF4J记录异常
category: log
copyright: log
excerpt: SLF4J
---

## 1. 概述

在本快速教程中，我们将展示如何使用[SLF4J](https://www.slf4j.org/)API记录Java中的异常，我们将使用slf4j-simple API作为日志记录实现。

你可以在我们之前的一篇[文章](https://www.baeldung.com/category/logging/)中探索不同的日志记录技术。

## 2. Maven依赖

首先，我们需要在pom.xml中添加以下依赖：

```xml
<dependency>                             
    <groupId>org.slf4j</groupId>         
    <artifactId>slf4j-api</artifactId>   
    <version>1.7.30</version>  
</dependency>
<dependency>                             
    <groupId>org.slf4j</groupId>         
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.30</version>  
</dependency>
```

这些库的最新版本可以在[Maven Central](https://mvnrepository.com/artifact/org.slf4j/slf4j-simple)上找到。

## 3. 示例

通常，所有异常都使用[Logger](https://static.javadoc.io/org.slf4j/slf4j-api/1.7.25/org/slf4j/Logger.html)类中的error()方法记录，此方法有很多变体，我们来看一下：

```java
void error(String msg);
void error(String format, Object... arguments);
void error(String msg, Throwable t);
```

首先我们来初始化一下要使用的Logger：

```java
Logger logger = LoggerFactory.getLogger(NameOfTheClass.class);
```

如果只需要显示错误消息，那么我们可以简单地添加：

```java
logger.error("An exception occurred!");
```

上述代码的输出将是：

```text
ERROR packageName.NameOfTheClass - An exception occurred!
```

这很简单，但是为了添加有关异常的更多相关信息(包括堆栈跟踪)，我们可以这样写：

```java
logger.error("An exception occurred!", new Exception("Custom exception"));
```

输出将是：

```text
ERROR packageName.NameOfTheClass - An exception occurred!
java.lang.Exception: Custom exception
  at packageName.NameOfTheClass.methodName(NameOfTheClass.java:lineNo)Lambda
```

在存在多个参数的情况下，如果日志记录语句中的最后一个参数是异常，那么SLF4J将假定用户希望将最后一个参数视为异常而不是简单参数：

```java
logger.error("{}, {}! An exception occurred!", 
    "Hello", 
    "World", 
    new Exception("Custom exception"));
```

在上面的代码片段中，字符串消息将根据传递的对象详细信息进行格式化，我们使用花括号作为传递给方法的字符串参数的占位符。

在这种情况下，输出将是：

```text
ERROR packageName.NameOfTheClass - Hello, World! An exception occurred!
java.lang.Exception: Custom exception 
  at packageName.NameOfTheClass.methodName(NameOfTheClass.java:lineNo)
```

## 4. 总结

在本快速教程中，我们介绍了如何使用SLF4J API记录异常。