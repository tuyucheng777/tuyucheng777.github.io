---
layout: post
title: 如何修复java.lang.UnsupportedClassVersionError
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在这个简短的教程中，我们将了解导致Java运行时错误java.lang.UnsupportedClassVersionError: Unsupported major.minor version的原因以及如何修复它。

## 2. 重现错误

让我们首先看一个错误示例：

```text
Exception in thread "main" java.lang.UnsupportedClassVersionError: cn/tuyucheng/taketoday/MajorMinorApp 
  has been compiled by a more recent version of the Java Runtime (class file version 55.0), 
  this version of the Java Runtime only recognizes class file versions up to 52.0
```

**这个错误告诉我们，我们的类是在比我们试图运行它的版本更高的Java版本上编译的**。更具体地说，在本例中，我们使用Java 11编译我们的类并尝试使用Java 8运行它。

### 2.1 Java版本号

作为参考，让我们快速浏览一下Java版本号。如果我们需要下载适当的Java版本，这会派上用场。

主版本号和次版本号存储在类字节码的第六和第七字节处。

让我们看看主版本号如何映射到Java版本：

-   45 = Java 1.1
-   46 = Java 1.2
-   47 = Java 1.3
-   48 = Java 1.4
-   49 = Java 5
-   50 = Java 6
-   51 = Java 7
-   52 = Java 8
-   53 = Java 9
-   54 = Java 10
-   55 = Java 11
-   56 = Java 12
-   57 = Java 13
-   58 = Java 14
-   59 = Java 15
-   60 = Java 16
-   61 = Java 17
-   62 = Java 18
-   63 = Java 19
-   64 = Java 20
-   65 = Java 21
-   66 = Java 22

## 3. 通过命令行修复

现在让我们讨论如何在从命令行运行Java时解决此错误。

根据我们的情况，**我们有两种方法可以解决此错误：使用早期版本的Java编译我们的代码，或者在较新的Java版本上运行我们的代码**。

最后的决定取决于我们的情况，如果我们需要使用已经在更高级别编译的第三方库，我们最好的选择可能是使用更新的Java版本运行我们的应用程序。如果我们正在打包一个应用程序以供分发，最好编译成旧版本。

### 3.1 JAVA_HOME环境变量

让我们从检查[JAVA_HOME](https://www.baeldung.com/find-java-home)变量的设置方式开始，当我们从命令行运行javac时，这将告诉我们正在使用哪个JDK：

```shell
echo %JAVA_HOME%
C:\Apps\Java\jdk8-x64
```

如果我们准备好完全迁移到更新的[JDK](https://www.baeldung.com/jvm-vs-jre-vs-jdk)，我们可以下载更新的版本并确保我们的PATH和[JAVA_HOME环境变量设置](https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux)正确。

### 3.2 运行新的JRE

回到我们的示例，让我们看看如何通过在更高版本的Java上运行来解决错误。假设我们在C:\\Apps\\jdk-11.0.2中有Java 11 JRE，我们可以使用与它打包的java命令运行我们的代码：

```shell
C:\Apps\jdk-11.0.2\bin\java cn.tuyucheng.taketoday.MajorMinorApp
Hello World!
```

### 3.3 使用较旧的JDK进行编译

**如果我们正在编写一个应用程序，希望可以在某个版本的Java上运行，我们需要编译该版本的代码**。

我们可以通过以下三种方式之一来做到这一点：使用旧的JDK来编译我们的代码；使用javac命令的-bootclasspath、-source和-target选项(JDK 8及更早版本)；或使用–release选项(JDK 9及更新版本)。

让我们从使用较旧的JDK开始，类似于我们使用较新的JRE来运行代码的方式：

```shell
C:\Apps\Java\jdk1.8.0_31\bin\javac cn/tuyucheng/taketoday/MajorMinorApp.java
```

可以只使用-source和-target，但它可能仍会创建与旧版Java不兼容的class文件。

为了确保兼容性，我们可以将-bootclasspath指向目标JRE的rt.jar：

```shell
javac -bootclasspath "C:\Apps\Java\jdk1.8.0_31\jre\lib\rt.jar" \
  -source 1.8 -target 1.8 cn/tuyucheng/taketoday/MajorMinorApp.java
```

以上主要适用于JDK 8及更低版本。**在JDK 9中，添加了–release参数以替换-source和-target，–release选项支持目标6、7、8、9、10和 11**。

让我们使用–release来定位Java 8：

```shell
javac --release 8 cn/tuyucheng/taketoday/MajorMinorApp.java
```

现在我们可以在Java 8或更高版本的JRE上运行我们的代码。

## 4. Eclipse集成开发环境

现在我们了解了错误和纠正它的一般方法，让我们利用我们所学的知识，看看在 Eclipse IDE 中工作时如何应用它。

### 4.1 更改JRE

假设我们已经为[Eclipse配置了不同版本的Java](https://www.baeldung.com/eclipse-change-java-version)，让我们更改项目的JRE。

让我们转到Project properties，然后是Java Build Path，然后是Libraries选项卡。在那里，我们将选择JRE并单击Edit：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion01.png)

现在让我们选择Alternate JRE并指向我们的Java 11安装：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion02.png)

此时，我们的应用程序将在Java 11上运行。

### 4.2 更改编译器级别

现在让我们看看如何将目标更改为较低级别的Java。

首先，让我们回到Project properties，然后是Java Compiler，然后选中Enable project specific settings：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion03.png)

在这里我们可以将项目设置为针对早期版本的Java进行编译并自定义其他合规性设置：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion04.png)

## 5. IntelliJ IDEA

我们还可以控制在IntelliJ IDEA中用于编译和运行的Java版本。

### 5.1 添加JDK

在此之前，我们需要了解如何添加其他JDK，让我们转到File -> Project Structure -> Platform Settings -> SDKs：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion05.png)

让我们单击中间列中的加号图标，从下拉列表中选择JDK，然后选择我们的JDK位置：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion06.png)

### 5.2 更改JRE

首先，我们将了解如何使用IDEA在较新的JRE上运行我们的项目。

让我们转到Run -> Edit Configurations...并将JRE更改为11：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion07.png)

现在，当运行我们的项目时，它将使用Java 11 JRE运行。

### 5.3 更改编译器级别

如果我们要分发应用程序以在较低的JRE上运行，则需要调整编译器级别以针对旧版本的Java。

让我们转到File -> Project Structure -> Project Settings -> Project并更改我们的项目SDK和项目语言级别：

![](/assets/images/2025/javajvm/javalangunsupportedclassversion08.png)

我们现在可以构建项目，生成的class文件将在Java 8及更高版本上运行。

## 6. Maven

当我们在[Maven](https://www.baeldung.com/maven)中构建和打包文件时，我们可以控制目标[Java的版本](https://www.baeldung.com/maven-java-version)。

**使用Java 8或更早版本时，我们为编译器插件设置source和target**。

让我们使用编译器插件属性设置source和target：

```xml
<properties>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.source>1.8</maven.compiler.source>
</properties>
```

或者，我们可以在编译器插件中设置source和target：

```xml
<plugins>
    <plugin>    
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```

**通过在Java 9中添加的–release选项，我们也可以使用Maven对其进行配置**。

让我们使用编译器插件属性来设置release：

```xml
<properties>
    <maven.compiler.release>8</maven.compiler.release>
</properties>
```

或者我们可以直接配置编译器插件：

```xml
<plugins>
    <plugin>    
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <release>8</release>
        </configuration>
    </plugin>
</plugins>
```

## 7. 总结

在本文中，我们介绍了导致java.lang.UnsupportedClassVersionError: Unsupported major.minor version错误消息的原因，以及如何修复它。