---
layout: post
title:  确定Java中的类是否实现接口
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在本教程中，我们将介绍几种确定对象或类是否实现特定接口的方法。

## 2. 使用Java反射API

Java反射API提供了几种方法来检查对象或类是否实现了接口，使用此API可以防止将第三方库添加到我们的项目。

### 2.1 数据模型

Java运行时环境([JRE](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jre))提供了一些方法来检索类的已实现接口。

首先，让我们定义一个包含一些接口和类的模型。在本例中，我们将定义一个MasterInterface接口和两个扩展第一个接口的子接口：

![](/assets/images/2025/javareflect/javacheckclassimplementsinterface01.png)

我们将对第一个实现MasterInterface接口的类和两个分别实现ChildInterface1和ChildInterface2接口的子类执行相同的操作：

![](/assets/images/2025/javareflect/javacheckclassimplementsinterface02.png)

以下是我们的模型的代码：

```java
public interface MasterInterface {}

public interface ChildInterface1 extends MasterInterface {}

public interface ChildInterface2 extends MasterInterface {}

public class MasterClass implements MasterInterface {}

public class ChildClass1 implements ChildInterface1 {}

public class ChildClass2 implements ChildInterface2 {}
```

### 2.2 getInterfaces()方法

Java API在Class对象上提供了getInterfaces()方法，此方法检索Class已实现的接口，我们可以使用以下代码行获取类的所有已实现接口：

```java
List<Class<?>> interfaces = Arrays.asList(childClass2.getClass().getInterfaces());

assertEquals(1, interfaces.size());
assertTrue(interfaces.contains(ChildInterface2.class));
```

**我们可以看到，该方法仅检索直接实现的接口，没有检测到MasterInterface的继承**。

如果我们想要获得完整的继承树，我们必须对已实现的接口进行递归，直到找到根接口：

```java
static Set<Class<?>> getAllExtendedOrImplementedInterfacesRecursively(Class<?> clazz) {
    Set<Class<?>> res = new HashSet<Class<?>>();
    Class<?>[] interfaces = clazz.getInterfaces();

    if (interfaces.length > 0) {
        res.addAll(Arrays.asList(interfaces));

        for (Class<?> interfaze : interfaces) {
            res.addAll(getAllExtendedOrImplementedInterfacesRecursively(interfaze));
        }
    }

    return res;
}
```

然后，执行之后，我们可以检查是否在Set中检索到了MasterInterface：

```java
assertTrue(interfaces.contains(ChildInterface2.class));
assertTrue(interfaces.contains(MasterInterface.class));
```

现在，我们可以检查我们的对象是否在我们的模型中实现了特定的接口。

### 2.3 isAssignableFrom()方法

以前的方法非常繁琐和痛苦，因此Java API提供了一种更快捷的方法来检查Object是否实现了接口，**我们可以使用Class对象的isAssignableFrom()方法进行检查**。

如果对象继承了指定的接口，即使这不是直接实现，此方法也返回true：

```java
ChildClass2 childClass2 = new ChildClass2();
assertTrue(MasterInterface.class.isAssignableFrom(childClass2.getClass()));
```

### 2.4 isInstance()方法

Class对象还提供了一个isInstance()方法，**如果提供的对象实现了以下接口，则此方法返回true**：

```java
assertTrue(MasterInterface.class.isInstance(childClass2));
```

### 2.5 instanceOf运算符

**检查对象是否实现接口的最后一种方法是使用instanceOf运算符，此运算符也适用于接口**：

```java
assertTrue(childClass2 instanceof MasterInterface);
```

## 3. 使用Apache Commons库

[Apache Commons Lang](https://commons.apache.org/proper/commons-lang/)库为几乎所有东西提供了实用程序类，**ClassUtils类能够列出某个类的所有已实现接口**。

首先，我们需要将[Maven Central](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

这基本上与我们在getInterfaces()方法中看到的方式相同，但我们只需要一行代码：

```java
List<Class<?>> allSuperInterfaces = ClassUtils.getAllInterfaces(childClass2.getClass());
```

这将递归返回所有已实现的接口：

```java
assertTrue(allSuperInterfaces.contains(ChildInterface2.class)); 
assertTrue(allSuperInterfaces.contains(MasterInterface.class));
```

## 4. 使用Reflections库

[Reflections](https://www.baeldung.com/reflections-library)库是一个第三方库，它可以扫描我们的对象，然后允许我们对收集到的元数据运行查询。

我们首先需要将[Maven Central](https://mvnrepository.com/artifact/org.reflections/reflections)依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.10.2</version>
</dependency>
```

在我们运行查询之前，该库需要收集数据，因此需要对其进行初始化：

```java
Reflections reflections = new Reflections("cn.tuyucheng.taketoday.checkInterface");
```

此操作将扫描提供的整个类路径，可能需要一些时间才能完成。此操作可以在应用程序启动期间完成，以便将来的查询顺利运行。

**然后，我们可以检索类实现的接口列表**：

```java
Set<Class<?>> classes = reflections.get(ReflectionUtils.Interfaces.of(ChildClass2.class));
```

## 5. 总结

在本文中，我们了解了检查类是否实现特定接口的不同方法。根据我们应用程序的需求，我们可以使用标准Java反射API，或者如果需要更强大的功能，则依赖第三方库。