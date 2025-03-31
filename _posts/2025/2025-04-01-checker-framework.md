---
layout: post
title:  Checker框架-Java的可插拔类型系统
category: libraries
copyright: libraries
excerpt: Checker Framework
---

## 1. 概述

从Java 8版本开始，可以使用所谓的[可插拔类型系统](https://docs.oracle.com/javase/tutorial/java/annotations/type_annotations.html)来编译程序-它可以应用比编译器所应用的检查更严格的检查。

我们只需要使用可用的几个可插拔类型系统提供的注解。

在这篇简短的文章中，我们将探索华盛顿大学提供的[Checker Framework](https://checkerframework.org/)。

## 2. Maven

要开始使用Checker框架，我们首先需要将其添加到pom.xml中：

```xml
<dependency>
    <groupId>org.checkerframework</groupId>
    <artifactId>checker-qual</artifactId>
    <version>3.42.0</version>
</dependency>
<dependency>
    <groupId>org.checkerframework</groupId>
    <artifactId>checker</artifactId>
    <version>3.42.0</version>
</dependency>
```

你可以在[Maven Central](https://mvnrepository.com/artifact/org.checkerframework/checker)上检查库的最新版本。

前两个依赖包含Checker框架的代码，其中所有类型均已由Checker框架的开发人员进行了适当的标注。

然后，我们必须适当调整maven-compiler-plugin以使用Checker Framework作为可插拔类型系统：

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>${maven-compiler-plugin.version}</version>
    <configuration>
        <fork>true</fork>
        <compilerArgument>-Xlint:all</compilerArgument>
        <showWarnings>true</showWarnings>
        <annotationProcessorPaths>
            <path>
                <groupId>org.checkerframework</groupId>
                <artifactId>checker</artifactId>
                <version>${checker.version}</version>
            </path>
        </annotationProcessorPaths>
        <annotationProcessors>
            <annotationProcessor>
                org.checkerframework.checker.nullness.NullnessChecker
            </annotationProcessor>
            <annotationProcessor>
                org.checkerframework.checker.interning.InterningChecker
            </annotationProcessor>
            <annotationProcessor>
                org.checkerframework.checker.fenum.FenumChecker
            </annotationProcessor>
            <annotationProcessor>
                org.checkerframework.checker.formatter.FormatterChecker
            </annotationProcessor>
            <annotationProcessor>
                org.checkerframework.checker.regex.RegexChecker
            </annotationProcessor>
        </annotationProcessors>
        <compilerArgs combine.children="append">
            <arg>-Awarns</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.main=ALL-UNNAMED</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.model=ALL-UNNAMED</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED</arg>
            <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED</arg>
            <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.comp=ALL-UNNAMED</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

这里的重点是<annotationProcessors\>标签的内容，这里我们列出了所有想要针对我们的源运行的检查器。

## 3. 避免NullPointerException

Checker Framework可以帮助我们的第一个场景是识别可能产生NullPointerException的代码片段：

```java
private static int countArgs(@NonNull String[] args) {
    return args.length;
}

public static void main(@Nullable String[] args) {
    System.out.println(countArgs(args));
}
```

在上面的例子中，我们用@NonNull注解声明countArgs()的args参数不能为空。

无论这个限制如何，在main()中，我们调用该方法时都会传递一个确实可以为空的参数，因为它已用@Nullable标注。

当我们编译代码时，Checker Framework会适时地警告我们代码中可能存在错误：

```text
[WARNING] /checker-plugin/.../NonNullExample.java:[12,38] [argument.type.incompatible]
 incompatible types in argument.
  found   : null
  required: @Initialized @NonNull String @Initialized @NonNull []
```

## 4. 正确使用常量作为枚举

有时我们使用一系列常量作为枚举项。

假设我们需要一系列国家和行星。然后，我们可以用@Fenum注解标注这些元素，以将属于同一“假”枚举的所有常量分组：

```java
static final @Fenum("country") String ITALY = "IT";
static final @Fenum("country") String US = "US";
static final @Fenum("country") String UNITED_KINGDOM = "UK";

static final @Fenum("planet") String MARS = "Mars";
static final @Fenum("planet") String EARTH = "Earth";
static final @Fenum("planet") String VENUS = "Venus";
```

之后，当我们编写一个应该接收“planet”字符串的方法时，我们可以正确地标注该参数：

```java
void greetPlanet(@Fenum("planet") String planet){
    System.out.println("Hello " + planet);
}
```

由于错误，我们可以使用尚未定义为行星的可能值的字符串来调用greetPlanet()，例如：

```java
public static void main(String[] args) {
    obj.greetPlanets(US);
}
```

Checker Framework可以发现错误：

```text
[WARNING] /checker-plugin/.../FakeNumExample.java:[29,26] [argument.type.incompatible]
 incompatible types in argument.
  found   : @Fenum("country") String
  required: @Fenum("planet") String
```

## 5. 正则表达式

假设我们知道一个字符串变量必须存储至少一个匹配组的正则表达式。

我们可以利用Checker框架并像这样声明这样的变量：

```java
@Regex(1) private static String FIND_NUMBERS = "\\d*";
```

这显然是一个潜在的错误，因为我们分配给FIND_NUMBERS的正则表达式没有任何匹配的组。

事实上，Checker Framework会在编译时认真地告知我们错误：

```text
[WARNING] /checker-plugin/.../RegexExample.java:[7,51] [assignment.type.incompatible]
incompatible types in assignment.
  found   : @Regex String
  required: @Regex(1) String
```

## 6. 总结

对于想要超越标准编译器并提高代码正确性的开发人员来说，Checker Framework是一个有用的工具。

它能够在编译时检测到一些通常只能在运行时检测到的典型错误，甚至可以通过引发编译错误来停止编译。

标准检查比我们在本文中介绍的要多得多；请查看[此处](https://checkerframework.org/manual/)的Checker Framework官方手册中提供的检查，甚至可以编写自己的检查。