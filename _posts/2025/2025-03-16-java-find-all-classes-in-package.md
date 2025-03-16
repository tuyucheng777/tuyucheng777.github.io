---
layout: post
title:  查找Java包中的所有类
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

有时，我们想要获取有关应用程序运行时行为的信息，例如查找运行时可用的所有类。

在本教程中，我们将探讨如何在运行时查找Java包中的所有类的几个示例。

## 2. 类加载器

首先，我们先从Java[类加载器](https://www.baeldung.com/java-classloaders)开始讨论。Java类加载器是Java运行时环境(JRE)的一部分，它动态地将Java类加载到Java虚拟机(JVM)中。Java类加载器将JRE与文件和文件系统的了解分离开来，**并非所有类都由单个类加载器加载**。

让我们通过图形表示来了解Java中可用的类加载器：

![](/assets/images/2025/javareflect/javafindallclassesinpackage01.png)

Java 9对类加载器进行了一些重大更改。随着模块的引入，我们可以选择在类路径旁边提供模块路径，系统类加载器会加载模块路径上存在的类。

**类加载器是动态的**，它们不需要在运行时告诉JVM它可以提供哪些类。因此，在包中查找类本质上是一个文件系统操作，而不是使用[Java反射](https://www.baeldung.com/java-reflection)来完成的操作。

但是，我们可以编写自己的类加载器或检查类路径来查找包内的类。

## 3. 在Java包中查找类

为了说明起见，让我们创建一个包`cn.tuyucheng.taketoday.reflection.access.packages.search`。

现在，让我们定义一个示例类：

```java
public class ClassExample {
    class NestedClass {
    }
}
```

接下来我们定义一个接口：

```java
public interface InterfaceExample {
}
```

在下一节中，我们将研究如何使用系统类加载器和一些第三方库来查找类。

### 3.1 系统类加载器

首先，我们将使用内置的系统类加载器。**系统类加载器会加载在类路径中找到的所有类**，这发生在JVM的早期初始化期间：

```java
public class AccessingAllClassesInPackage {

    public Set<Class> findAllClassesUsingClassLoader(String packageName) {
        InputStream stream = ClassLoader.getSystemClassLoader()
            .getResourceAsStream(packageName.replaceAll("[.]", "/"));
        BufferedReader reader = new BufferedReader(new InputStreamReader(stream));
        return reader.lines()
            .filter(line -> line.endsWith(".class"))
            .map(line -> getClass(line, packageName))
            .collect(Collectors.toSet());
    }
 
    private Class getClass(String className, String packageName) {
        try {
            return Class.forName(packageName + "."
              + className.substring(0, className.lastIndexOf('.')));
        } catch (ClassNotFoundException e) {
            // handle the exception
        }
        return null;
    }
}
```

在上面的例子中，我们使用静态getSystemClassLoader()方法加载系统类加载器。

接下来，我们将在给定的包中找到资源，我们将使用getResourceAsStream方法将资源读取为URL流。要获取包下的资源，我们需要将包名称转换为URL字符串。因此，我们必须将所有点(.)替换为路径分隔符(“/”)。

之后，我们把流输入到BufferedReader中，并过滤所有带有.class扩展名的URL。获取所需资源后，我们将构造类并将所有结果收集到Set中。**由于Java不允许Lambda抛出异常，因此我们必须在getClass方法中处理它**。

现在让我们测试一下这个方法：

```java
@Test
public void when_findAllClassesUsingClassLoader_thenSuccess() {
    AccessingAllClassesInPackage instance = new AccessingAllClassesInPackage();
 
    Set<Class> classes = instance.findAllClassesUsingClassLoader("cn.tuyucheng.taketoday.reflection.access.packages.search");
 
    Assertions.assertEquals(3, classes.size());
}
```

包中只有两个Java文件。但是，我们声明了三个类—包括嵌套类NestedExample。因此，我们的测试产生了三个类。

请注意，搜索包与当前工作包不同。

### 3.2 反射库

[Reflections](https://www.baeldung.com/reflections-library)是一个流行的库，它可以扫描当前类路径并允许我们在运行时查询它。

首先向我们的Maven项目添加[Reflections依赖项](https://mvnrepository.com/artifact/org.reflections/reflections/0.9.12/jar)：

```xml
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId> 
    <version>0.9.12</version>
</dependency>
```

现在，让我们深入研究代码示例：

```java
public Set<Class> findAllClassesUsingReflectionsLibrary(String packageName) {
    Reflections reflections = new Reflections(packageName, new SubTypesScanner(false));
    return reflections.getSubTypesOf(Object.class)
        .stream()
        .collect(Collectors.toSet());
}
```

在此方法中，我们启动SubTypesScanner类并获取Object类的所有子类型。通过这种方法，我们在获取类时可以获得更高的粒度。

再次，让我们测试一下：

```java
@Test
public void when_findAllClassesUsingReflectionsLibrary_thenSuccess() {
    AccessingAllClassesInPackage instance = new AccessingAllClassesInPackage();
 
    Set<Class> classes = instance.findAllClassesUsingReflectionsLibrary("cn.tuyucheng.taketoday.reflection.access.packages.search");
 
    Assertions.assertEquals(3, classes.size());
}
```

与我们之前的测试类似，此测试找到给定包中声明的类。

### 3.3 Google Guava库

在本节中，我们将了解如何使用Google Guava库查找类。Google Guava提供了一个ClassPath实用程序类，用于扫描类加载器的源代码并查找所有可加载的类和资源。

首先，让我们将[Guava依赖项](https://mvnrepository.com/artifact/com.google.guava/guava/30.1.1-jre/jar)添加到我们的项目中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

让我们深入研究代码：

```java
public Set<Class> findAllClassesUsingGoogleGuice(String packageName) throws IOException {
    return ClassPath.from(ClassLoader.getSystemClassLoader())
        .getAllClasses()
        .stream()
        .filter(clazz -> clazz.getPackageName()
            .equalsIgnoreCase(packageName))
        .map(clazz -> clazz.load())
        .collect(Collectors.toSet());
}
```

在上述方法中，我们将系统类加载器作为ClassPath#from方法的输入。所有由ClassPath扫描的类都根据包名称进行过滤，然后加载(但不链接或初始化)已过滤的类并将其收集到Set中。

现在让我们测试一下这个方法：

```java
@Test
public void when_findAllClassesUsingGoogleGuice_thenSuccess() throws IOException {
    AccessingAllClassesInPackage instance = new AccessingAllClassesInPackage();
 
    Set<Class> classes = instance.findAllClassesUsingGoogleGuice("cn.tuyucheng.taketoday.reflection.access.packages.search");
 
    Assertions.assertEquals(3, classes.size());
}
```

此外，Google Guava库还提供了getTopLevelClasses()和getTopLevelClassesRecursive()方法。

值得注意的是，在上述所有示例中，**如果包下存在package-info并用一个或多个包级注解进行标注，则它包含在可用类列表中**。

下一节将讨论如何在模块化应用程序中查找类。

### 4. 在模块化应用程序中查找类

Java平台模块系统(JPMS)通过[模块](https://www.baeldung.com/java-9-modularity)为我们引入了新级别的访问控制，每个包都必须明确导出才能在模块外部访问。

在模块化应用程序中，每个模块可以是命名模块、未命名模块或自动模块之一。

对于命名模块和自动模块，内置系统类加载器将没有类路径，系统类加载器将使用应用程序模块路径搜索类和资源。

对于未命名的模块，它会将类路径设置为当前工作目录。

### 4.1 模块内

模块中的所有包都对模块中的其他包具有可见性，模块内的代码享有对所有类型及其所有成员的反射访问。

### 4.2 模块外

由于Java强制执行最严格的访问，我们必须使用export或open模块声明明确声明包，以获得对模块内类的反射访问。

对于普通模块，导出包(但不是开放包)的反射访问仅提供对声明包的公共和受保护类型及其所有成员的访问。

我们可以构造一个模块，导出需要搜索的包：

```java
module my.module {
    exports cn.tuyucheng.taketoday.reflection.access.packages.search;
}
```

对于普通模块，开放包的反射访问提供对声明包的所有类型及其成员的访问：

```java
module my.module {
    opens cn.tuyucheng.taketoday.reflection.access.packages.search;
}
```

同样，打开模块会授予对所有类型及其成员的反射访问权限，就像所有包都已打开一样。现在让我们打开整个模块进行反射访问：

```java
open module my.module{
}
```

最后，在确保为模块提供了用于访问包的正确的模块描述后，可以使用上一节中的任何方法来查找包内的所有可用类。

## 5. 总结

总之，我们了解了类加载器以及查找包中所有类的不同方法。此外，我们还讨论了在模块化应用程序中访问包。