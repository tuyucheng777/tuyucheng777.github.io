---
layout: post
title:  Flogger流式日志记录
category: log
copyright: log
excerpt: Flogger
---

##  1. 概述

在本教程中，我们将讨论[Flogger](https://google.github.io/flogger/)框架，这是Google设计的流式的Java日志记录API。

## 2. 为什么要使用Flogger？

目前市场上已经有了Log4j和Logback等多种日志框架，为什么我们还需要另一个日志框架呢？

事实证明，Flogger比其他框架有几个优势-让我们来看看。

### 2.1 可读性

Flogger API的流式性在很大程度上提高了它的可读性。

让我们看一个例子，我们想每10次迭代记录一条消息。

使用传统的日志框架，我们会看到类似这样的内容：

```java
int i = 0;

// ...

if (i % 10 == 0) {
    logger.info("This log shows every 10 iterations");
    i++;
}
```

但是现在，有了Flogger，上述内容可以简化为：

```java
logger.atInfo().every(10).log("This log shows every 10 iterations");
```

尽管有人会认为Flogger版本的记录器语句看起来比传统版本更冗长一些，但**它确实允许更强大的功能，并最终给出更易读和更具表现力的日志语句**。

### 2.2 性能

只要我们避免在记录的对象上调用toString，记录对象就会得到优化：

```java
User user = new User();
logger.atInfo().log("The user is: %s", user);
```

如果我们记录日志，如上所示，后端就有机会优化日志。另一方面，如果我们直接调用toString，或者拼接字符串，那么这个机会就失去了：

```java
logger.atInfo().log("Ths user is: %s", user.toString());
logger.atInfo().log("Ths user is: %s" + user);
```

### 2.3 可扩展性

Flogger框架已经涵盖了我们期望日志框架具备的大部分基本功能。

但是，有些情况下我们需要添加功能，在这些情况下，可以扩展API。

**目前，这需要一个单独的支持类**。例如，我们可以通过编写UserLogger类来扩展Flogger API：

```java
logger.at(INFO).forUserId(id).withUsername(username).log("Message: %s", param);
```

当我们想要一致地格式化消息时，这可能很有用。然后，UserLogger将提供自定义方法forUserId(String id)和withUsername(String username)的实现。

为此，**UserLogger类必须扩展AbstractLogger类并提供API的实现**。如果我们查看FluentLogger，它只是一个没有其他方法的记录器，因此，**我们可以先按原样此类，然后在此基础上通过向其添加方法进行构建**。

### 2.4 效率

传统框架广泛使用可变参数，这些方法需要分配并填充一个新的Object[]，然后才能调用该方法。此外，传入的任何基本类型都必须自动装箱。

**所有这些都会在调用点产生额外的字节码和延迟**，如果日志语句实际上没有启用，那就特别不幸了。**这种成本在经常出现在循环中的debug级别日志中变得更加明显**；Flogger通过完全避免使用可变参数来消除这些成本。

**Flogger通过使用流式的调用链(可以从中构建日志语句)解决了此问题**，这样，框架只需对日志方法进行少量覆盖，从而能够避免诸如可变参数和自动装箱之类的问题。**这意味着API可以容纳各种新功能，而不会出现组合爆炸**。

典型的日志框架具有以下方法：

```java
level(String, Object)
level(String, Object...)
```

其中level可以是大约7个日志级别名称之一(例如，severe)，并且具有接收附加日志级别的规范日志方法：

```java
log(Level, Object...)
```

除此之外，通常还有一些方法的变体，它们接收与日志语句相关的原因(Throwable实例)：

```java
level(Throwable, String, Object)
level(Throwable, String, Object...)
```

很明显，该API将3个关注点耦合到一个方法调用中：

1. 它试图指定日志级别(方法选择)
2. 尝试将元数据附加到日志语句(Throwable原因)
3. 并且指定日志消息和参数

这种方法迅速增加了满足这些独立问题所需的不同日志记录方法的数量。

现在我们可以明白为什么在链中拥有两种方法很重要：

```java
logger.atInfo().withCause(e).log("Message: %s", arg);
```

现在让我们看看如何在我们的代码库中使用它。

## 3. 依赖

设置Flogger非常简单，我们只需要将[flogger](https://mvnrepository.com/artifact/com.google.flogger)和[flogger-system-backend](https://mvnrepository.com/artifact/com.google.flogger)添加到pom中：

```xml
<dependencies>
    <dependency>
        <groupId>com.google.flogger</groupId>
        <artifactId>flogger</artifactId>
        <version>0.4</version>
    </dependency>
    <dependency>
        <groupId>com.google.flogger</groupId>
        <artifactId>flogger-system-backend</artifactId>
        <version>0.4</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

设置这些依赖后，我们现在可以继续探索我们可以使用API。

## 4. 探索Fluent API

首先，让我们为记录器声明一个静态实例：

```java
private static final FluentLogger logger = FluentLogger.forEnclosingClass();
```

现在我们可以开始记录了，从一些简单的事情开始：

```java
int result = 45 / 3;
logger.atInfo().log("The result is %d", result);
```

日志消息可以使用任何Java的printf格式说明符，例如%s、%d或%016x。

### 4.1 避免在日志点使用

Flogger的创建者建议我们避免在日志点工作。

假设我们有以下长期运行的方法来总结组件的当前状态：

```java
public static String collectSummaries() {
    longRunningProcess();
    int items = 110;
    int s = 30;
    return String.format("%d seconds elapsed so far. %d items pending processing", s, items);
}
```

在我们的日志语句中直接调用collectSummaries是很诱人的：

```java
logger.atFine().log("stats=%s", collectSummaries());
```

**但是，无论配置的日志级别或速率限制如何，每次都会调用collectSummaries方法**。

让禁用日志记录语句的成本几乎为零是日志记录框架的核心，这反过来意味着，代码中可以保留更多日志记录语句而不会造成损害，像我们刚才那样编写日志记录语句会消除这一优势。

**相反，我们应该使用LazyArgs.lazy方法**：

```java
logger.atFine().log("stats=%s", LazyArgs.lazy(() -> collectSummaries()));
```

现在，日志点几乎无需执行任何工作-只需为Lambda表达式创建实例；**Flogger仅在打算实际记录消息时才会评估此Lambda**。

尽管允许使用isEnabled来保护日志语句：

```java
if (logger.atFine().isEnabled()) {
    logger.atFine().log("summaries=%s", collectSummaries());
}
```

这不是必需的，我们应该避免这样做，因为Flogger会替我们进行这些检查。这种方法也只能按级别保护日志语句，而对速率受限的日志语句没有帮助。

### 4.2 处理异常

那么如果出现异常，我们该如何处理呢？

嗯，Flogger带有withStackTrace方法，我们可以使用它来记录Throwable实例：

```java
try {
    int result = 45 / 0;
} catch (RuntimeException re) {
    logger.atInfo().withStackTrace(StackSize.FULL).withCause(re).log("Message");
}
```

withStackTrace将StackSize枚举作为参数，其常量值为SMALL、MEDIUM、LARGE或FULL。withStackTrace()生成的堆栈跟踪将在默认java.util.logging后端中显示为LogSiteStackTrace异常。不过，其他后端可能会选择以不同的方式处理此问题。

### 4.3 日志配置和级别

到目前为止，我们在大多数示例中都使用了logger.atInfo，但Flogger确实支持许多其他级别。我们将介绍这些级别，但首先，让我们介绍如何配置日志记录选项。

为了配置日志记录，我们使用LoggerConfig类。

例如，当我们想要将日志记录级别设置为FINE时：

```java
LoggerConfig.of(logger).setLevel(Level.FINE);
```

Flogger支持多种日志级别：

```java
logger.atInfo().log("Info Message");
logger.atWarning().log("Warning Message");
logger.atSevere().log("Severe Message");
logger.atFine().log("Fine Message");
logger.atFiner().log("Finer Message");
logger.atFinest().log("Finest Message");
logger.atConfig().log("Config Message");
```

### 4.4 速率限制

那么速率限制问题怎么办？如果我们不想记录每次迭代，我们该如何处理这种情况？

**Flogger使用every(int n)方法来帮助我们**：

```java
IntStream.range(0, 100).forEach(value -> {
    logger.atInfo().every(40).log("This log shows every 40 iterations => %d", value);
});
```

运行上述代码时我们得到以下输出：

```text
Sep 18, 2019 5:04:02 PM cn.tuyucheng.taketoday.flogger.FloggerUnitTest lambda$givenAnInterval_shouldLogAfterEveryTInterval$0
INFO: This log shows every 40 iterations => 0 [CONTEXT ratelimit_count=40 ]
Sep 18, 2019 5:04:02 PM cn.tuyucheng.taketoday.flogger.FloggerUnitTest lambda$givenAnInterval_shouldLogAfterEveryTInterval$0
INFO: This log shows every 40 iterations => 40 [CONTEXT ratelimit_count=40 ]
Sep 18, 2019 5:04:02 PM cn.tuyucheng.taketoday.flogger.FloggerUnitTest lambda$givenAnInterval_shouldLogAfterEveryTInterval$0
INFO: This log shows every 40 iterations => 80 [CONTEXT ratelimit_count=40 ]
```

如果我们想每10秒记录一次怎么办？那么我们**可以使用atMostEvery(int n, TimeUnit unit)**：

```java
IntStream.range(0, 1_000_0000).forEach(value -> {
    logger.atInfo().atMostEvery(10, TimeUnit.SECONDS).log("This log shows [every 10 seconds] => %d", value);
});
```

这样，结果就变成了：

```text
Sep 18, 2019 5:08:06 PM cn.tuyucheng.taketoday.flogger.FloggerUnitTest lambda$givenATimeInterval_shouldLogAfterEveryTimeInterval$1
INFO: This log shows [every 10 seconds] => 0 [CONTEXT ratelimit_period="10 SECONDS" ]
Sep 18, 2019 5:08:16 PM cn.tuyucheng.taketoday.flogger.FloggerUnitTest lambda$givenATimeInterval_shouldLogAfterEveryTimeInterval$1
INFO: This log shows [every 10 seconds] => 3545373 [CONTEXT ratelimit_period="10 SECONDS [skipped: 3545372]" ]
Sep 18, 2019 5:08:26 PM cn.tuyucheng.taketoday.flogger.FloggerUnitTest lambda$givenATimeInterval_shouldLogAfterEveryTimeInterval$1
INFO: This log shows [every 10 seconds] => 7236301 [CONTEXT ratelimit_period="10 SECONDS [skipped: 3690927]" ]
```

## 5. 使用Flogger与其他后端

那么，如果我们想**将Flogger添加到已经使用[Slf4j](https://www.baeldung.com/slf4j-with-log4j2-logback)或[Log4j](https://www.baeldung.com/java-logging-intro)的现有应用程序中**，该怎么办？这在我们想利用现有配置的情况下可能很有用。我们将看到，Flogger支持多个后端。

### 5.1 使用Slf4j的Flogger

配置Slf4j后端很简单，首先，我们需要将[flogger-slf4j-backend](https://mvnrepository.com/artifact/com.google.flogger)依赖添加到pom中：

```xml
<dependency>
    <groupId>com.google.flogger</groupId>
    <artifactId>flogger-slf4j-backend</artifactId>
    <version>0.4</version>
</dependency>
```

接下来，我们需要告诉Flogger我们想要使用与默认后端不同的后端，我们通过系统属性注册Flogger工厂来实现这一点：

```java
System.setProperty("flogger.backend_factory", "com.google.common.flogger.backend.slf4j.Slf4jBackendFactory#getInstance");
```

现在我们的应用程序将使用现有的配置。

### 5.2 使用Log4j的Flogger

我们按照类似的步骤配置Log4j后端，让我们将[flogger-log4j-backend](https://mvnrepository.com/artifact/com.google.flogger/flogger-log4j-backend)依赖添加到pom中：

```xml
<dependency>
    <groupId>com.google.flogger</groupId>
    <artifactId>flogger-log4j-backend</artifactId>
    <version>0.4</version>
    <exclusions>
        <exclusion>
            <groupId>com.sun.jmx</groupId>
            <artifactId>jmxri</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jdmk</groupId>
            <artifactId>jmxtools</artifactId>
        </exclusion>
        <exclusion>
            <groupId>javax.jms</groupId>
            <artifactId>jms</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>apache-log4j-extras</artifactId>
    <version>1.2.17</version>
</dependency>
```

我们还需要为Log4j注册一个Flogger后端工厂：

```java
System.setProperty("flogger.backend_factory", "com.google.common.flogger.backend.log4j.Log4jBackendFactory#getInstance");
```

就这样，我们的应用程序现在设置为使用现有的Log4j配置。

## 6. 总结

在本教程中，我们了解了如何使用Flogger框架作为传统日志框架的替代方案，我们还了解了使用该框架时可以受益的一些强大功能。

我们还了解了如何通过注册不同的后端(如Slf4j和Log4j)来利用现有的配置。