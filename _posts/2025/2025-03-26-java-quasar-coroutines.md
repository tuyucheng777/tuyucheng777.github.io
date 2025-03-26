---
layout: post
title:  Quasar协程简介
category: libraries
copyright: libraries
excerpt: Quasar
---

## 1. 简介

协程是Java线程的替代品，因为它们提供了一种在非常高的并发水平上执行可中断任务的方法，但是在Project Loom完成之前，我们必须依靠库支持来获得它。

在本教程中，我们将了解Quasar，一个提供协程支持的库。

## 2. 设置

我们将使用最新版本的Quasar，该版本**需要Java 11或更高版本**。但是，示例应用程序也可以与Java 7和8兼容的早期版本的Quasar一起使用。

**Quasar提供了三个依赖**，我们需要将其包含在构建中：

```xml
<dependency>
    <groupId>co.paralleluniverse</groupId>
    <artifactId>quasar-core</artifactId>
    <version>0.8.0</version>
</dependency>
<dependency>
    <groupId>co.paralleluniverse</groupId>
    <artifactId>quasar-actors</artifactId>
    <version>0.8.0</version>
</dependency>
<dependency>
    <groupId>co.paralleluniverse</groupId>
    <artifactId>quasar-reactive-streams</artifactId>
    <version>0.8.0</version>
</dependency>
```

**Quasar的实现依赖于字节码检测才能正常工作**，要执行字节码检测，我们有两个选择：

- 在编译时，或
- 在运行时使用Java代理

使用Java代理是首选方式，因为它没有任何特殊的构建要求并且可以适用于任何设置。

### 2.1 使用Maven指定Java代理

要使用Maven运行Java代理，我们需要包含[maven-dependency-plugin](http://maven.apache.org/plugins/maven-dependency-plugin/)以始终运行properties目标：

```xml
<plugin>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
        <execution>
            <id>getClasspathFilenames</id>
            <goals>
                <goal>properties</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**properties目标将生成一个指向类路径上的quasar-core.jar位置的属性**。

为了执行我们的应用程序，我们将使用[exec-maven-plugin](https://www.mojohaus.org/exec-maven-plugin/)：

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <workingDirectory>target/classes</workingDirectory>
        <executable>echo</executable>
        <arguments>
            <argument>-javaagent:${co.paralleluniverse:quasar-core:jar}</argument>
            <argument>-classpath</argument> <classpath/>
            <argument>cn.tuyucheng.taketoday.quasar.QuasarHelloWorldKt</argument>
        </arguments>
    </configuration>
</plugin>
```

为了使用该插件并启动我们的应用程序，我们将运行Maven：

```shell
mvn compile dependency:properties exec:exec
```

## 3. 实现协程

为了实现协程，我们将使用Quasar库中的Fibers。**Fibers提供轻量级线程**，这些线程将由JVM而不是操作系统管理。由于它们需要的RAM很少，并且对CPU的负担也小得多，因此我们可以在应用程序中拥有数百万个Fibers，而不必担心性能。

为了启动纤程，我们创建Fiber<T\>类的实例，它将包装我们要执行的代码并调用start方法：

```java
new Fiber<Void>(() -> {
    System.out.println("Inside fiber coroutine...");
}).start();
```

## 4. 总结

在本文中，我们介绍了如何使用Quasar库实现协程。我们在这里看到的只是一个最小的工作示例，Quasar库可以做更多的事情。