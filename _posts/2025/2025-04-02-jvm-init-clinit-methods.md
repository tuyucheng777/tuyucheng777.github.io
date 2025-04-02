---
layout: post
title:  JVM中的<init>和<clinit>方法
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

JVM使用两种不同的方法来初始化对象实例和类。

在这篇简短的文章中，我们将了解编译器和运行时如何使用<init\>和<clinit\>方法进行初始化。

## 2. 实例初始化方法

让我们从一个简单的对象分配和赋值开始：

```java
Object obj = new Object();
```

如果我们编译此代码片段并通过javap -c查看它的字节码，我们将看到类似以下内容：

```text
0: new           #2      // class java/lang/Object
3: dup
4: invokespecial #1      // Method java/lang/Object."<init>":()V
7: astore_1
```

为了初始化对象，JVM调用一个名为<init\>的特殊方法。**用JVM术语来说，这个方法就是一个[实例初始化方法](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.9.1)**。当且仅当满足以下条件时，方法才是实例初始化：

-   它是在一个类中定义
-   它的名字是<init\>
-   它返回void

**每个类都可以有0个或多个实例初始化方法**，这些方法通常对应于基于JVM的编程语言(如Java或Kotlin)中的构造函数。

### 2.1 构造函数和实例初始化块

为了更好地理解Java编译器如何将构造函数转换为<init\>，让我们考虑另一个示例：

```java
public class Person {

    private String firstName = "Foo"; // <init>
    private String lastName = "Bar"; // <init>

    // <init>
    {
        System.out.println("Initializing...");
    }

    // <init>
    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // <init>
    public Person() {
    }
}
```

这是这个类的字节码：

```text
public Person(java.lang.String, java.lang.String);
  Code:
     0: aload_0
     1: invokespecial #1       // Method java/lang/Object."<init>":()V
     4: aload_0
     5: ldc           #7       // String Foo
     7: putfield      #9       // Field firstName:Ljava/lang/String;
    10: aload_0
    11: ldc           #15      // String Bar
    13: putfield      #17      // Field lastName:Ljava/lang/String;
    16: getstatic     #20      // Field java/lang/System.out:Ljava/io/PrintStream;
    19: ldc           #26      // String Initializing...
    21: invokevirtual #28      // Method java/io/PrintStream.println:(Ljava/lang/String;)V
    24: aload_0
    25: aload_1
    26: putfield      #9       // Field firstName:Ljava/lang/String;
    29: aload_0
    30: aload_2
    31: putfield      #17      // Field lastName:Ljava/lang/String;
    34: return
```

**尽管构造函数和初始化块在Java中是分开的，但它们在字节码级别处于相同的实例初始化方法中**。事实上，这个<init\>方法：

-   首先，初始化firstName和lastName字段(索引0到13)
-   然后，它将一些内容作为实例初始化程序块的一部分打印到控制台(索引16到21)
-   最后，它使用构造函数参数更新实例变量

如果我们按如下方式创建一个Person：

```java
Person person = new Person("Brian", "Goetz");
```

然后这将转换为以下字节码：

```text
0: new           #7        // class Person
3: dup
4: ldc           #9        // String Brian
6: ldc           #11       // String Goetz
8: invokespecial #13       // Method Person."<init>":(Ljava/lang/String;Ljava/lang/String;)V
11: astore_1
```

这次JVM调用另一个具有与Java构造函数相对应的签名的<init\>方法。

**这里的关键是构造函数和其他实例初始化器等同于JVM世界中的<init\>方法**。

## 3. 类初始化方法

在Java中，当我们要在类级别初始化某些东西时，[静态初始化块](https://www.baeldung.com/java-static#a-static-block)很有用：

```java
public class Person {

    private static final Logger LOGGER = LoggerFactory.getLogger(Person.class); // <clinit>

    // <clinit>
    static {
        System.out.println("Static Initializing...");
    }

    // omitted
}
```

当我们编译前面的代码时，编译器将静态块转换为字节码级别的[类初始化方法](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.9.2)。

简而言之，一个方法若且仅若满足以下条件，则其为类初始化方法：

-   它的名字是<clinit\>
-   它返回 无效

**因此，在Java中生成<clinit\>方法的唯一方法是使用静态字段和静态块初始化器**。

JVM在我们第一次使用相应的类时调用<clinit\>。因此，<clinit\>调用发生在运行时，我们在字节码级别看不到该调用。

## 4. 总结

在这篇简短的文章中，我们了解了JVM中<init\>和<clinit\>方法之间的区别。<init\>方法用于初始化对象实例；此外，JVM会在必要时调用<clinit\>方法来初始化类。

为了更好地理解初始化在JVM中的工作方式，强烈建议阅读[JVM规范](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-5.html#jvms-5.5)。