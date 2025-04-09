---
layout: post
title:  创建自定义Logback Appender
category: log
copyright: log
excerpt: Logback
---

## 1. 简介

在本文中，我们将探讨如何创建自定义Logback附加程序，如果你正在寻找Java日志记录的介绍，请查看[本文](https://www.baeldung.com/java-logging-intro)。

Logback附带许多内置附加程序，可写入标准输出、文件系统或数据库。此框架架构的优点在于其模块化，这意味着我们可以轻松对其进行自定义。

在本教程中，我们将重点介绍logback-classic，它需要以下Maven依赖：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.3.5</version>
</dependency>
```

此依赖的最新版本可在[Maven Central](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic)上找到。

## 2. 基本Logback Appender

Logback提供了我们可以扩展的基类来创建自定义附加器。

Appender是所有附加器都必须实现的泛型接口，泛型类型是ILoggingEvent或AccessEvent，具体取决于我们使用的是logback-classic还是logback-access。

**我们的自定义附加器应该扩展AppenderBase或UnsynchronizedAppenderBase**，它们都实现Appender并处理过滤器和状态消息等功能。

AppenderBase是线程安全的；UnsynchronizedAppenderBase子类负责管理其线程安全。

就像ConsoleAppender和FileAppender都扩展了OutputStreamAppender并调用父方法setOutputStream()一样，**如果自定义附加器要写入OutputStream，它应该成为OutputStreamAppender的子类**。

## 3. 自定义Appender

对于我们的自定义示例，我们将创建一个名为MapAppender的示例附加程序，此附加程序会将所有日志事件插入ConcurrentHashMap中，并使用时间戳作为键。首先，我们将子类化AppenderBase并使用ILoggingEvent作为泛型类型：

```java
public class MapAppender extends AppenderBase<ILoggingEvent> {

    private ConcurrentMap<String, ILoggingEvent> eventMap = new ConcurrentHashMap<>();

    @Override
    protected void append(ILoggingEvent event) {
        eventMap.put(String.valueOf(System.currentTimeMillis()), event);
    }

    public Map<String, ILoggingEvent> getEventMap() {
        return eventMap;
    }
}
```

接下来，为了使MapAppender开始接收日志记录事件，让我们将其作为附加器添加到配置文件logback.xml中：

```xml
<configuration>
    <appender name="map" class="cn.tuyucheng.taketoday.logback.MapAppender"/>
    <root level="info">
        <appender-ref ref="map"/>
    </root>
</configuration>
```

## 4. 设置属性

Logback使用JavaBeans自省来分析附加器上设置的属性，我们的自定义附加器将需要Getter和Setter方法，以便自省器能够查找和设置这些属性。

让我们向MapAppender添加一个属性，为eventMap的键提供前缀：

```java
public class MapAppender extends AppenderBase<ILoggingEvent> {

    //...

    private String prefix;

    @Override
    protected void append(ILoggingEvent event) {
        eventMap.put(prefix + System.currentTimeMillis(), event);
    }

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    //...
}
```

接下来，在我们的配置中添加一个属性来设置这个前缀：

```xml
<configuration debug="true">

    <appender name="map" class="cn.tuyucheng.taketoday.logback.MapAppender">
        <prefix>test</prefix>
    </appender>

    //...
</configuration>
```

## 5. 错误处理

为了处理在创建和配置自定义附加器期间发生的错误，我们可以使用从AppenderBase继承的方法。

例如，当prefix属性为空或空字符串时，MapAppender可以调用addError()并提前返回：

```java
public class MapAppender extends AppenderBase<ILoggingEvent> {

    //...

    @Override
    protected void append(final ILoggingEvent event) {
        if (prefix == null || "".equals(prefix)) {
            addError("Prefix is not set for MapAppender.");
            return;
        }

        eventMap.put(prefix + System.currentTimeMillis(), event);
    }

    //...
}
```

当我们的配置中打开debug标志时，我们会在控制台中看到一个错误，警告我们prefix属性尚未设置：

```xml
<configuration debug="true">

    //...

</configuration>
```

## 6. 总结

在此快速教程中，我们重点介绍了如何为Logback实现自定义附加器。