---
layout: post
title:  如何在Spock测试中检查已记录的消息
category: unittest
copyright: unittest
excerpt: Spock
---

## 1. 简介

当我们的代码执行时，我们经常会记录感兴趣的事件。其中一些事件非常重要，因此我们希望确保我们的消息被记录下来。

在本教程中，我们将学习如何验证我们的类是否在[Spock测试](https://www.baeldung.com/groovy-spock)中记录了特定消息。我们将使用Spring默认日志框架Logback中的ListAppender来捕获我们的消息以供稍后检查。

## 2. 设置

首先，让我们创建一个ClassWithLogger Java类，用于记录我们想要验证的一些信息和警告消息。让我们使用Slf4j注解来实例化记录器，并添加一个在info级别记录消息的方法：

```java
@Slf4j
public class ClassWithLogger {
    void logInfo() {
        log.info("info message");
    }
}
```

## 3. 测试框架

现在让我们创建一个ClassWithLoggerTest规范，以ClassWithLogger实例作为主题：

```groovy
class ClassWithLoggerTest extends Specification {
    @Subject
    def subject = new ClassWithLogger()
}
```

接下来，我们必须配置一个Logger来监听ClassWithLogger消息。因此，让我们从LoggerFactory获取一个Logger，并将其转换为Logback Logger实例以使用Logback的功能。

```groovy
def logger = (Logger) LoggerFactory.getLogger(ClassWithLogger)
```

之后，**让我们创建一个ListAppender来收集消息，并将附加器添加到我们的Logger**。让我们使用setup方法，以便每次测试时重置我们收集的消息：

```groovy
def setup() {
    listAppender = new ListAppender<>()
    listAppender.start()
    logger.addAppender(listAppender)
}
```

设置好ListAppender后，我们就可以监听日志消息、收集它们并在测试中验证它们了。

## 4. 检查简单消息

让我们编写第一个测试来记录消息并检查它。因此，让我们通过调用ClassWithLogger的logInfo方法来记录一条INFO消息，并验证它是否在我们的ListAppender的消息列表中，并具有预期的文本和日志级别：

```groovy
def "when our subject logs an info message then we validate an info message was logged"() {
    when: "we invoke a method that logs an info message"
    subject.logInfo()

    then: "we get our expected message"
    with(listAppender.list[0]) {
        getMessage() == expectedText
        getLevel() == Level.INFO
    }
}
```

在这种情况下，我们知道我们的消息是第一个也是唯一一个记录的消息，因此我们可以安全地在ListAppender的捕获消息列表中的索引0处检查它。

当我们要查找的日志事件不一定是第一条消息时，我们需要一种更强大的方法。在这种情况下，让我们流式传输消息列表并过滤我们的消息：

```groovy
then: "we get our expected message"
def ourMessage = listAppender.list.stream()
        .filter(logEvent -> logEvent.getMessage().contains(expectedText))
        .findAny()
ourMessage.isPresent()

and: "the details match"
with(ourMessage.get()) {
    getMessage() == expectedText
    getLevel() == Level.INFO
}
```

## 5. 检查格式化的消息

当我们在[LoggingEvent](https://logback.qos.ch/apidocs/ch.qos.logback.classic/ch/qos/logback/classic/spi/LoggingEvent.html)上调用getMessage时，它会返回模板而不是格式化的字符串。对于简单消息，这就是我们所需要的，但是**当我们向日志消息添加参数时，我们必须使用getFormattedMessage来检查实际记录的消息**。

因此，让我们向ClassWithLogger添加一个logInfoWithParameter方法，该方法使用模板字符串和参数记录消息：

```java
void logInfoWithParameter(String extraData) { 
    log.info("info message: {}", extraData); 
}
```

现在，让我们编写一个测试来调用我们的新方法：

```groovy
def "when our subject logs an info message from a template then we validate the an info message was logged with a parameter"() {
    when: "we invoke a method that logs info messages"
    subject.logInfoWithParameter("parameter")
}
```

最后，让我们检查第一个记录的消息，以确保LoggingEvent的getArgumentArray中的第一个元素与我们的参数值匹配，并使用getFormattedMessage来验证完整的消息：

```groovy
then: 'the details match for the first message in the list'
with (listAppender.list[0]) {
    getMessage() == 'info message: {}'
    getArgumentArray()[0] == 'parameter'
    getFormattedMessage() == 'info message: parameter'
    getLevel() == Level.INFO
}
```

**请注意，getMessage返回模板字符串，而getFormattedMessage返回记录的实际消息**。

## 6. 总结

在本文中，我们学习了如何使用Logback的ListAppender收集日志消息，以及如何使用LoggingEvent检查简单消息。我们还学习了如何使用带有参数的模板字符串以及参数值本身来验证记录的消息。