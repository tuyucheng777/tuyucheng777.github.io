---
layout: post
title:  如何预热JVM
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

JVM是有史以来最古老但功能最强大的虚拟机之一。

在本文中，我们将快速介绍预热JVM的含义以及如何执行预热。

## 2. JVM架构基础

每当一个新的JVM进程启动时，所有需要的类都会由[ClassLoader](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ClassLoader.html)的实例加载到内存中，此过程分为三个步骤：

1. **引导类加载器**：“引导类加载器”将Java代码和必要的Java类(例如java.lang.Object)加载到内存中，这些加载的类位于JRE\\lib\\rt.jar中。
2. **扩展类加载**：ExtClassLoader负责加载位于java.ext.dirs路径下的所有JAR文件，在非Maven或非Gradle的应用程序中，如果开发人员手动添加JAR文件，则所有这些类都会在此阶段加载。
3. **应用程序类加载**：AppClassLoader加载位于应用程序类路径中的所有类。

此初始化过程基于延迟加载方案。

## 3. 什么是JVM预热

类加载完成后，所有重要的类(在进程启动时使用)都会被推送到[JVM缓存(原生代码)](https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/rdyn_tunediskcache.html)中，这使得它们在运行时可以更快地被访问，其他类则根据每个请求进行加载。

对Java Web应用程序的首次请求通常比其整个生命周期的平均响应时间慢得多，这段预热时间通常可以归因于延迟类加载和即时编译。

记住这一点，对于低延迟应用程序，我们需要预先缓存所有类-以便在运行时访问时立即可用。

**这个调整JVM的过程称为预热**。

## 4. 分层编译

由于JVM的完善架构，在应用程序生命周期内，常用的方法会被加载到本机缓存中。

我们可以使用此属性在应用程序启动时强制将关键方法加载到缓存中，为此，我们需要设置一个名为“**Tiered Compilation**”的VM参数：

```shell
-XX:CompileThreshold -XX:TieredCompilation
```

通常，虚拟机使用解释器来收集输入到编译器的方法的分析信息。在分层方案中，除了解释器之外，客户端编译器还用于生成方法的编译版本，以收集有关自身的分析信息。

由于编译后的代码比解释的代码快得多，因此程序在性能分析阶段的执行性能更好。

在启用此VM参数的JBoss和JDK 7上运行的应用程序，由于记录的[错误](https://issues.jboss.org/browse/JBEAP-26)，一段时间后往往会崩溃，此问题已在JDK 8中修复。

这里需要注意的另一点是，为了强制加载，我们必须确保所有(或大多数)即将执行的类都需要被访问。这类似于在单元测试中确定代码覆盖率，覆盖的代码越多，性能就越好。

下一节将演示如何实现这一点。

## 5. 手动实现

我们可以实现另一种技术来预热JVM，在这种情况下，简单的手动预热可能包括在应用程序启动后重复创建数千次不同的类。

首先，我们需要创建一个具有正常方法的虚拟类：

```java
public class Dummy {
    public void m() {
    }
}
```

接下来，我们需要创建一个具有静态方法的类，该方法将在应用程序启动时执行至少100000次，并且每次执行时，它都会创建我们之前创建的上述虚拟类的新实例：

```java
public class ManualClassLoader {
    protected static void load() {
        for (int i = 0; i < 100000; i++) {
            Dummy dummy = new Dummy();
            dummy.m();
        }
    }
}
```

现在，为了衡量性能提升，我们需要创建一个主类，该类包含一个静态块，其中包含对ManualClassLoader的load()方法的直接调用。

在主函数中，我们再次调用ManualClassLoader的load()方法，并捕获函数调用前后的系统时间(以纳秒为单位)。最后，我们将这些时间相减，得到实际的执行时间。

我们必须运行该应用程序两次；一次在静态块内使用load()方法调用，一次不使用此方法调用：

```java
public class MainApplication {
    static {
        long start = System.nanoTime();
        ManualClassLoader.load();
        long end = System.nanoTime();
        System.out.println("Warm Up time : " + (end - start));
    }
    public static void main(String[] args) {
        long start = System.nanoTime();
        ManualClassLoader.load();
        long end = System.nanoTime();
        System.out.println("Total time taken : " + (end - start));
    }
}
```

以下结果以纳秒为单位重现：

|   预热    |   不预热    | 差距(％) |
|:-------:|:--------:|:-----:|
| 1220056 | 8903640  |  730  |
| 1083797 | 13609530 | 1256  |
| 1026025 | 9283837  |  905  |
| 1024047 | 7234871  |  706  |
| 868782  | 9146180  | 1053  |

正如预期的那样，采用预热方法比采用正常方法表现出更好的性能。

当然，这只是一个非常简单的基准测试，只能从表面层面展现该技术的影响。此外，需要注意的是，在实际应用中，我们需要先熟悉系统中的典型代码路径。

## 6. 工具

我们还可以使用一些工具来预热JVM，最著名的工具之一是Java微基准测试工具([JMH)](http://openjdk.java.net/projects/code-tools/jmh/)，它通常用于微基准测试。加载后，**它会反复运行代码片段并监控预热迭代周期**。

要使用它，我们需要向pom.xml添加另一个依赖：

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
</dependency>
```

可以在[Central Maven Repository](https://mvnrepository.com/search?q=org.openjdk.jmh)中检查JMH的最新版本。

或者，我们可以使用JMH的Maven插件来生成示例项目：

```shell
mvn archetype:generate \
    -DinteractiveMode=false \
    -DarchetypeGroupId=org.openjdk.jmh \
    -DarchetypeArtifactId=jmh-java-benchmark-archetype \
    -DgroupId=cn.tuyucheng.taketoday \
    -DartifactId=test \
    -Dversion=1.0
```

接下来，让我们创建一个main方法：

```java
public static void main(String[] args) throws RunnerException, IOException {
    Main.main(args);
}
```

现在，我们需要创建一个方法并使用JMH的@Benchmark注解对其进行标注：

```java
@Benchmark
public void init() {
    //code snippet	
}
```

在这个init方法内部，我们需要编写需要重复执行的代码以进行预热。

## 7. 性能基准

在过去的20年里，Java的大部分贡献都与GC(垃圾收集器)和JIT(即时编译器)有关，网上几乎所有的性能基准测试都是在已经运行了一段时间的JVM上进行的。

不过，[北京航空航天大学](http://www.eecg.toronto.edu/~yuan/papers/osdi16-hottub.pdf)发布了一份考虑了JVM预热时间的基准测试报告，他们使用基于Hadoop和Spark的系统来处理海量数据：

![](/assets/images/2025/javajvm/javajvmwarmup01.png)

这里HotTub指的是JVM预热的环境。

正如你所见，加速效果非常显著，特别是对于相对较小的读取操作而言-这就是为什么这些数据值得考虑。

## 8. 总结

在这篇简短的文章中，我们展示了JVM在应用程序启动时如何加载类以及如何预热JVM以获得性能提升。