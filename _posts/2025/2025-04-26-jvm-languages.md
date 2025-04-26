---
layout: post
title:  JVM语言概述
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 简介

除了Java，其他语言也可以在Java虚拟机上运行，如Scala、Kotlin、Groovy、Clojure。

在下面的部分中，我们将从更高层次来介绍最流行的JVM语言。

当然，我们先从JVM语言的前身Java说起。

## 2. Java

### 2.1 概述

[Java](https://java.com/en/)是一种采用面向对象范式的通用编程语言。

**该语言的核心特性是跨平台可移植性**，这意味着在一个平台上编写的程序可以在任何具有适当运行时支持的软件和硬件组合上执行，这是通过先将代码编译为字节码，而不是直接编译为特定于平台的机器码来实现的。

Java字节码指令类似于机器代码，但它们由特定于主机操作系统和硬件组合的Java虚拟机(JVM)解释。

**尽管Java最初是一种面向对象的语言，但它已经开始采用其他编程范式的概念，例如[函数式编程](https://www.baeldung.com/cs/functional-programming)**。

让我们快速了解一下Java的一些主要功能：

- 面向对象
- [强静态类型](https://www.baeldung.com/cs/statically-vs-dynamically-typed-languages)
- 独立于平台
- 垃圾回收
- 多线程

### 2.2 示例

让我们看看一个简单的“Hello,World!”示例是什么样的：

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

在这个例子中，我们创建了一个名为HelloWorld的类，并定义了在控制台上打印消息的main方法。

接下来，**我们将使用javac命令生成可以在JVM上执行的字节码**：

```shell
javac HelloWorld.java
```

最后，**java命令在JVM上执行生成的字节码**：

```shell
java HelloWorld
```

## 3. Scala

### 3.1 概述

[Scala](https://www.scala-lang.org/)代表“可扩展语言”，**Scala是一种静态类型语言，它结合了两种重要的编程范式：面向对象编程和函数式编程**。 

该语言起源于2004年，但近年来变得越来越流行。

Scala是一种纯面向对象语言，因为**它不支持原始类型**，Scala提供了定义类、对象、方法的能力，以及函数式编程特性，如特征、代数数据类型或类型类。

Scala的一些重要特性包括：

- 函数式、面向对象
- 强静态类型
- 代数数据类型
- 模式匹配
- 增强的不变性支持
- 惰性计算
- 多线程

### 3.2 示例

首先，让我们看一下与前面相同的“Hello,World!”示例，这次使用Scala：

```scala
object HelloWorld {
    def main(args: Array[String]): Unit = println("Hello, world!")
}
```

在这个例子中，我们创建了一个名为HelloWorld的单例对象和main方法。

接下来，为了编译它，我们可以使用scalac：

```shell
scalac HelloWorld.scala
```

scala命令在JVM上执行生成的字节码：

```shell
scala HelloWorld
```

## 4. Kotlin

### 4.1 概述

**[Kotlin](https://kotlinlang.org/)是由[JetBrains](https://en.wikipedia.org/wiki/JetBrains)团队开发的一种静态类型、通用开源语言**，它融合了面向对象和函数式范式。

开发Kotlin时主要关注的是Java互操作性、安全性(异常处理)、简洁性和更好的工具支持。

自Android Studio 3.0发布以来，Kotlin已成为Google在Android平台上全面支持的编程语言，它也包含在Android Studio IDE软件包中，作为标准Java编译器的替代方案。

Kotlin的一些重要特性：

- 面向对象、函数式
- 强静态类型
- 简洁
- 可与Java互操作

### 4.2 示例

让我们看一下Kotlin中的“Hello,World!”示例：

```java
fun main(args: Array<String>) { println("Hello, World!") }
```

我们可以将上面的代码写入一个名为helloWorld.kt的新文件中。

然后，**我们将使用kotlinc命令来编译它并生成可以在JVM上执行的字节码**：

```shell
kotlinc helloWorld.kt -include-runtime -d helloWorld.jar
```

-d选项用于指定类文件的输出文件或.jar文件名，**-include-runtime选项通过包含Kotlin运行时库，使生成的.jar文件自包含且可运行**。

然后，java命令在JVM上执行生成的字节码：

```shell
java -jar helloWorld.jar
```

我们再来看看使用for循环打印元素列表的另一个例子：

```java
fun main(args: Array<String>) {
    val items = listOf(1, 2, 3, 4)
    for (i in items) println(i)
}
```

## 5. Groovy

### 5.1 概述

**[Groovy](http://groovy-lang.org/)是一种面向对象、可选类型的动态领域特定语言(DSL)**，支持静态类型和静态编译功能，它旨在通过易于学习的语法提高开发人员的生产力。

**Groovy可以轻松地与任何Java程序集成**，并立即添加强大的功能，如脚本功能、运行时和编译时元编程以及函数式编程功能。

让我们重点介绍几个重要功能：

- 面向对象，具有高阶函数、柯里化、闭包等功能特性
- 类型–动态、静态、强、鸭子
- 领域特定语言
- 与Java的互操作性
- 简洁高效
- 运算符重载

### 5.2 示例

首先，让我们看看Groovy中的“Hello,World!”示例：

```groovy
println("Hello world")
```

我们将上述代码写入了一个名为HelloWorld.groovy的新文件中，现在**我们可以通过两种方式运行此代码：编译后执行，或者直接运行未编译的代码**。

我们可以使用groovyc命令编译.groovy文件，如下所示：

```shell
groovyc HelloWorld.groovy
```

然后，我们将使用java命令执行groovy代码：

```shell
java -cp <GROOVY_HOME>\embeddable\groovy-all-<VERSION>.jar;. HelloWorld
```

例如，上面的命令可能如下所示：

```shell
java -cp C:\utils\groovy-1.8.1\embeddable\groovy-all-1.8.1.jar;. HelloWorld
```

再看看如何使用groovy命令来执行.groovy文件而无需编译：

```shell
groovy HelloWorld.groovy
```

最后，这是另一个打印带有索引的元素列表的示例：

```groovy
list = [1, 2, 3, 4, 5, 6, 7, 8, 9]
list.eachWithIndex { it, i -> println "$i: $it"}
```

## 6. Clojure

### 6.1 概述

**[Clojure](https://clojure.org/)是一种通用的函数式编程语言**，该语言可以在JVM以及微软的[通用语言运行时(CLR)](https://en.wikipedia.org/wiki/Common_Language_Runtime)上运行。Clojure仍然是一种编译型语言，它保持着动态性，因为它的功能在运行时仍然受支持。

Clojure的设计者希望设计出能够在JVM上运行的现代[Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language))，因此，它也被称为Lisp编程语言的方言。**与Lisp类似，Clojure将代码视为数据，并且也拥有宏系统**。

一些重要的Clojure特性：

- 函数式
- 类型-动态，强，最近开始支持[渐进类型](https://en.wikipedia.org/wiki/Gradual_typing)
- 专为并发而设计
- 运行时多态性

### 6.2 示例

与其他JVM语言不同，在Clojure中创建简单的“Hello,World!”程序并不是那么简单。

我们将使用[Leiningen](https://leiningen.org/)工具来运行我们的示例。

首先，我们将使用以下命令使用默认模板创建一个简单的项目：

```shell
lein new hello-world
```

该项目将按照以下文件结构创建：

```text
./project.clj
./src
./src/hello-world
./src/hello-world/core.clj
```

现在我们需要使用以下内容更新./project.ctj文件来设置主源文件：

```ctj
(defproject hello-world "0.1.0-SNAPSHOT"
  :main hello-world.core
  :dependencies [[org.clojure/clojure "1.5.1"]])
```

现在我们准备更新代码，在./src/hello-world/core.clj文件中打印“Hello,World!”：

```clj
(ns hello-world.core)

(defn -main [& args]
    (println "Hello, World!"))
```

最后，我们切换到项目的根目录后，使用lein命令执行上面的代码：

```shell
cd hello-world
lein run
```

## 7. 其他JVM语言

### 7.1 Jython

**[Jython](http://www.jython.org/)是[Python](https://www.python.org/)的Java平台实现**，运行在JVM上。

该语言最初的设计目标是在不牺牲交互性的情况下编写高性能应用程序，**Jython是面向对象、多线程的，并使用Java的垃圾回收器来高效地清理内存**。

Jython包含Python语言的大部分模块，它还可以导入并使用Java库中的任何类。

让我们看一个简单的“Hello,World!”示例：

```python
print "Hello, world!"
```

### 7.2 JRuby

**[JRuby](http://www.jruby.org/)是[Ruby](http://www.ruby-lang.org/en/)编程语言在Java虚拟机上运行的实现**。

JRuby语言性能卓越，支持多线程，并拥有丰富的Java和Ruby库。此外，它还融合了两种语言的特性，例如面向对象编程和鸭子类型。

让我们在JRuby中打印“Hello,World!”：

```ruby
require "java"

stringHello= "Hello World"
puts "#{stringHello.to_s}"
```

## 8. 总结

在本文中，我们学习了许多流行的JVM语言以及一些基本的代码示例。这些语言实现了各种编程范式，例如面向对象、函数式、静态类型和动态类型。

到目前为止，即使JVM可以追溯到1995年，它仍然是现代编程语言高度相关且引人注目的平台。