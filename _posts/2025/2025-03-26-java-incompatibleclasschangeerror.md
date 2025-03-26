---
layout: post
title:  Java中的IncompatibleClassChangeError
category: libraries
copyright: libraries
excerpt: Java
---

## 1. 概述

在本文中，我们将探讨Java中的IncompatibleClassChangeError，这是当JVM检测到与先前加载的类不兼容的类更改时发生的运行时错误。

我们将通过示例和有效的解决方案深入探讨其原因。

## 2. Java中的IncompatibleClassChangeError类

IncompatibleClassChangeError是Java中的一种链接[错误](https://www.baeldung.com/java-errors-vs-exceptions)，**链接错误通常表示一个或多个依赖类存在问题**。

IncompatibleClassChangeError是LinkageError的一个子类，当一个或多个依赖类的类定义发生不兼容的更改时会引发该错误。

需要注意的是，这是Error的一个子类，因此我们不应该尝试捕获这些错误，因为它表示应用程序或运行时出现异常。

让我们尝试在程序中模拟IncompatibleClassChangeError，以便更好地理解它。

## 3. 重现错误

让我们尝试模拟导致IncompatibleClassChangeError的场景。

### 3.1 准备库

我们首先创建一个简单的库，它有一个父类Dinosaur和一个扩展了Dinosaur的子类Coelophysis：

```java
public class Dinosaur {
    public void species(String sp) {
        if(sp == null) {
            System.out.println("I am a generic Dinosaur");
        } else {
            System.out.println(sp);
        }
    }
}

public class Coelophysis extends Dinosaur {
    public void mySpecies() {
        species("My species is Coelophysis of the Triassic Period");
    }

    public static void main(String[] args) {
        Coelophysis coelophysis = new Coelophysis();
        coelophysis.mySpecies();
    }
}
```

我们应该注意到父类中的species()方法是非静态的。

### 3.2 从库生成JAR

**完成后，我们运行mvn package并从该项目生成一个jar文件**。

如果我们创建Coelophysis类的一个实例并调用species()方法，它将正常工作并生成所需的输出：

```text
➜ javac Coelophysis.java
➜ java Coelophysis
My species is Coelophysis of the Triassic Period
```

### 3.3 创建库的第二个版本

**接下来，我们创建另一个类似的库，但其父类Dinosaur的版本略有不同，包括一个静态species()方法**：

```java
public class Dinosaur {
    public Dinosaur() {
    }

    public static void species(String sp) {
        if (sp == null) {
            System.out.println("I am a generic Dinosaur");
        } else {
            System.out.println(sp);
        }
    }
}
```

我们也为该项目创建一个jar，并使用Maven系统导入命令将它们都导入到我们的客户端库中：

```xml
<dependency>
    <groupId>org.dinosaur</groupId>
    <artifactId>dinosaur</artifactId>
    <version>2</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/src/main/java/cn/tuyucheng/taketoday/incompatibleclasschange/dinosaur-1.jar</systemPath>
</dependency>
```

### 3.4 生成错误

现在，当我们通过将修改后的版本作为类路径依赖传递来调用Coelophysis类时，我们收到错误：

```text
➜  java -cp dinosaur-2:dinosaur-1 Coelophysis
Exception in thread "main" java.lang.IncompatibleClassChangeError: Expecting non-static method 'void Dinosaur.species(java.lang.String)'
	at Coelophysis.mySpecies(Coelophysis.java:3)
	at Coelophysis.main(Coelophysis.java:8)
```

## 4. IncompatibleClassChangeError的常见原因

Java中的IncompatibleClassChangeError是指类之间存在二进制不兼容的情况，这通常是由依赖类的定义发生变化引起的。让我们来看看可能导致该错误的一些常见场景。

### 4.1 对依赖类或二进制文件的类定义的更改

让我们考虑一个子类-父类场景，并且依赖子类的某些字段发生了变化。更改可能是将非静态非私有字段或方法更改为[静态](https://www.baeldung.com/java-static)字段，**在这种情况下，父类在运行时会生成IncompatibleClassChangeError异常**。

发生这种情况的原因是，运行时JVM所期望的一致性被破坏了。

我们可以通过依赖文件中的以下变化观察到类似的行为：

- 非final字段变为静态
- 类变成接口，反之亦然
- 非常量字段变为非静态
- 依赖类中方法的签名发生了一些变化

### 4.2 继承方式的变化

**当子类的[继承](https://www.baeldung.com/java-inheritance)模式发生被禁止的更改时，JVM也可能会抛出异常**。这包括实现接口而不添加所需抽象方法的重写实现，或者错误地实现类等情况。

### 4.3 Classpath中同一依赖的不同版本

假设我们使用[Maven](https://www.baeldung.com/maven)进行项目依赖管理，并通过在pom.xml中定义两个库A和B，将它们包含在类路径中。但是，这两个库可能依赖于同一个第三个库C的不同版本。

**因此，这两个库都尝试将库C的不同版本拉入类路径**，这些版本的结构略有不同。

## 5. 修复IncompatibleClassChangeError异常

现在我们已经了解了导致错误的原因，让我们看看如何修复和避免它。

每当依赖库或二进制文件发生变化时，我们都应该重新编译客户端代码以了解兼容性，我们必须确保编译时类定义与运行时类定义相匹配。因此，**保持向后二进制兼容性对于确保依赖客户端应用程序不会中断非常重要**。

像IntelliJ这样的现代IDE已经检查类路径中依赖的变化并对不兼容的变化发出警告。

**Maven等工具也会生成所有依赖的完整依赖关系图**，并突出显示pom.xml中不兼容或重大更改。此外，执行干净构建会自动重新生成所有依赖的源，这有助于避免出现此异常。

我们还可以使用Maven等构建工具来确保类路径中不存在相同依赖的重复或冲突版本，不断从target文件夹中删除过时的类文件以确保最新的类文件始终存在以供执行也是一种很好的做法。

## 6. 总结

在本教程中，我们讨论了IncompatibleClassChangeError，并强调了在编译时和运行时之间保持一致的类结构的重要性。

我们还讨论了在应用程序中可能产生此错误的方式以及如何有效地防止此错误。