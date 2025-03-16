---
layout: post
title:  Java中的方法参数反射
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

Java 8中添加了方法参数反射支持，简单来说，它提供了在运行时获取参数名称的支持。

在本快速教程中，我们将了解如何使用反射在运行时访问构造函数和方法的参数名称。

## 2. 编译器参数 

为了能够获取方法名称信息，我们必须明确选择加入。

为此，我们**在编译期间指定parameters选项**。

对于Maven项目，我们可以在pom.xml中声明此选项：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <compilerArgument>-parameters</compilerArgument>
    </configuration>
</plugin>
```

## 3. 示例类

我们将使用一个具有一个名为fullName的属性的Person类来演示：

```java
public class Person {

    private String fullName;

    public Person(String fullName) {
        this.fullName = fullName;
    }

    public void setFullName(String fullName) {
        this.fullName = fullName;
    }

    // other methods
}
```

## 4. 使用

Parameter类是Java 8中的新类，具有多种有趣的方法，如果提供了-parameters编译器选项，isNamePresent()方法将返回true。

要访问参数的名称，我们只需调用getName()：

```java
@Test
public void whenGetConstructorParams_thenOk() throws NoSuchMethodException, SecurityException {
    List<Parameter> parameters = Arrays.asList(Person.class.getConstructor(String.class).getParameters());
    Optional<Parameter> parameter = parameters.stream().filter(Parameter::isNamePresent).findFirst();
    assertThat(parameter.get().getName()).isEqualTo("fullName");
}

@Test
public void whenGetMethodParams_thenOk() throws NoSuchMethodException, SecurityException {
    List<Parameter> parameters = Arrays.asList(Person.class.getMethod("setFullName", String.class).getParameters());
    Optional<Parameter> parameter= parameters.stream()
        .filter(Parameter::isNamePresent)
        .findFirst();
 
    assertThat(parameter.get().getName()).isEqualTo("fullName");
}
```

## 5. 总结

在这篇简短的文章中，我们研究了Java 8中对参数名称的新反射支持。

此信息最明显的用例是帮助在编辑器中实现自动完成支持。