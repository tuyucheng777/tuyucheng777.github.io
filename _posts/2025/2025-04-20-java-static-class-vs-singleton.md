---
layout: post
title:  Java中的静态类与单例模式
category: designpattern
copyright: designpattern
excerpt: 单例模式
---

## 1. 简介

在本快速教程中，我们将讨论使用[单例](https://www.baeldung.com/java-singleton)设计模式编程和使用Java中的静态类编程之间的一些显著差异，我们将回顾这两种编码方法，并从编程的不同方面进行比较。

读完本文后，我们将能够在两个选项之间做出正确的决定。

## 2. 基础知识

单例模式是一种[设计模式](https://www.baeldung.com/design-patterns-series)，它确保在应用程序的整个生命周期内，一个类只有一个实例，它还提供了对该实例的全局访问点。

[static](https://www.baeldung.com/java-static)是一个保留关键字，它修饰实例变量使其变为类变量，因此，这些变量与类(任何对象)关联。当与方法一起使用时，只需通过类名即可访问它们。最后，我们还可以创建静态[嵌套内部类](https://www.baeldung.com/java-nested-classes)。

**在此上下文中，静态类包含静态方法和静态变量**。

## 3. 单例与静态工具类

现在，让我们深入探讨一下这两个概念之间的一些显著区别。

### 3.1 运行时多态性

Java中的静态方法在编译时解析，无法在运行时被重写，因此，静态类无法真正享受运行时多态性：

```java
public class SuperUtility {

    public static String echoIt(String data) {
        return "SUPER";
    }
}

public class SubUtility extends SuperUtility {

    public static String echoIt(String data) {
        return data;
    }
}

@Test
public void whenStaticUtilClassInheritance_thenOverridingFails() {
    SuperUtility superUtility = new SubUtility();
    Assert.assertNotEquals("ECHO", superUtility.echoIt("ECHO"));
    Assert.assertEquals("SUPER", superUtility.echoIt("ECHO"));
}
```

相反，**单例可以像任何其他类一样通过从基类派生来利用运行时多态性**：

```java
public class MyLock {

    protected String takeLock(int locks) {
        return "Taken Specific Lock";
    }
}

public class SingletonLock extends MyLock {

    // private constructor and getInstance method 

    @Override
    public String takeLock(int locks) {
        return "Taken Singleton Lock";
    }
}

@Test
public void whenSingletonDerivesBaseClass_thenRuntimePolymorphism() {
    MyLock myLock = new MyLock();
    Assert.assertEquals("Taken Specific Lock", myLock.takeLock(10));
    myLock = SingletonLock.getInstance();
    Assert.assertEquals("Taken Singleton Lock", myLock.takeLock(10));
}
```

此外，**单例还可以实现接口**，这使得它们比静态类更具优势：

```java
public class FileSystemSingleton implements SingletonInterface {

    // private constructor and getInstance method

    @Override
    public String describeMe() {
        return "File System Responsibilities";
    }
}

public class CachingSingleton implements SingletonInterface {

    // private constructor and getInstance method

    @Override
    public String describeMe() {
        return "Caching Responsibilities";
    }
}

@Test
public void whenSingletonImplementsInterface_thenRuntimePolymorphism() {
    SingletonInterface singleton = FileSystemSingleton.getInstance();
    Assert.assertEquals("File System Responsibilities", singleton.describeMe());
    singleton = CachingSingleton.getInstance();
    Assert.assertEquals("Caching Responsibilities", singleton.describeMe());
}
```

实现接口的[单例范围的Spring Bean](https://www.baeldung.com/spring-bean-scopes)是这种范例的完美示例。

### 3.2 方法参数

因为它本质上是一个对象，所以**我们可以轻松地将单例作为参数传递给其他方法**：

```java
@Test
public void whenSingleton_thenPassAsArguments() {
    SingletonInterface singleton = FileSystemSingleton.getInstance();
    Assert.assertEquals("Taken Singleton Lock", singleton.passOnLocks(SingletonLock.getInstance()));
}
```

但是，创建静态实用程序类对象并在方法中传递它是毫无价值的，而且是一个坏主意。

### 3.3 对象状态、序列化和克隆性

单例可以有实例变量，并且像任何其他对象一样，它可以维护这些变量的状态：

```java
@Test
public void whenSingleton_thenAllowState() {
    SingletonInterface singleton = FileSystemSingleton.getInstance();
    IntStream.range(0, 5)
        .forEach(i -> singleton.increment());
    Assert.assertEquals(5, ((FileSystemSingleton) singleton).getFilesWritten());
}
```

此外，**单例可以被[序列化](https://www.baeldung.com/java-serialization)以保存其状态或通过介质(例如网络)进行传输**：

```java
new ObjectOutputStream(baos).writeObject(singleton);
SerializableSingleton singletonNew = (SerializableSingleton) new ObjectInputStream(new ByteArrayInputStream(baos.toByteArray())).readObject();
```

最后，实例的存在也建立了使用Object的clone方法进行[克隆](https://www.baeldung.com/java-deep-copy)的可能性：

```java
@Test
public void whenSingleton_thenAllowCloneable() {
    Assert.assertEquals(2, ((SerializableCloneableSingleton) singleton.cloneObject()).getState());
}
```

**相反，静态类只有类变量和静态方法，因此它们不具有特定于对象的状态。由于静态成员属于类，我们无法序列化它们。此外，由于缺乏可供克隆的对象，克隆对于静态类来说毫无意义**。 

### 3.4 加载机制和内存分配

单例对象和其他类的实例一样，存在于堆中。它的优点在于，当应用程序需要时，大型单例对象可以延迟加载。

另一方面，静态类在编译时包含静态方法和静态绑定变量，并在栈上分配内存。因此，静态类在JVM[类加载](https://www.baeldung.com/java-classloaders)
时总是被积极加载。

### 3.5 效率和性能

如前所述，静态类不需要对象初始化，这消除了创建对象所需的时间开销。

此外，通过编译时静态绑定，它们比单例更高效并且速度更快。

我们必须仅出于设计原因而选择单例，而不是为了提高效率或性能而选择单一实例解决方案。

### 3.6 其他细微差别

使用单例而不是静态类进行编程也可以减少所需的重构量。

毫无疑问，单例是类的一个对象。因此，我们可以轻松地从单例模式转移到类的多实例模式。

由于静态方法的调用不需要对象，而是通过类名，因此迁移到多实例环境可能需要相对较大的重构。

其次，在静态方法中，由于逻辑与类定义而不是对象耦合，因此来自被单元测试的对象的静态方法调用变得更难被Mock，甚至被虚拟或存根实现覆盖。

## 4. 做出正确的选择

如果符合以下情况，则选择单例：

- 需要为应用程序提供完整的面向对象解决方案
- 在所有给定时间只需要一个类的实例并维持状态
- 想要一个类的延迟加载解决方案，以便仅在需要时加载它

当我们执行以下操作时，请使用静态类：

- 只需要存储许多仅对输入参数进行操作且不修改任何内部状态的静态实用方法
- 不需要运行时多态性或面向对象的解决方案

## 5. 总结

在本文中，我们回顾了Java中静态类和单例模式之间的一些本质区别，我们还推断了在软件开发中何时应该使用这两种方法。