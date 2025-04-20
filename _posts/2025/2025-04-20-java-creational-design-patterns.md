---
layout: post
title:  Java核心中的创建型设计模式
category: designpattern
copyright: designpattern
excerpt: 创建型设计模式
---

## 1. 简介

**[设计模式](https://www.baeldung.com/design-patterns-series)是我们在编写软件时常用的模式**，它们代表了随着时间的推移而发展起来的既定最佳实践，这些模式可以帮助我们确保代码的设计和构建都经过精心设计。

**[创建型模式](https://www.baeldung.com/creational-design-patterns)是一种专注于如何获取对象实例的设计模式**，通常，这意味着我们如何构造一个类的新实例，但在某些情况下，这意味着获取一个已经构造好的实例以供我们使用。

在本文中，我们将回顾一些常见的创建型设计模式。我们将了解它们的具体实现，以及在JVM或其他核心库中如何找到它们。

## 2. 工厂方法

工厂方法模式是一种将实例的构造与正在构造的类分离的方法，这样我们就可以抽象出确切的类型，从而允许我们的客户端代码使用接口或抽象类来工作：

```java
class SomeImplementation implements SomeInterface {
    // ...
}
```

```java
public class SomeInterfaceFactory {
    public SomeInterface newInstance() {
        return new SomeImplementation();
    }
}
```

在这里，我们的客户端代码无需了解SomeImplementation，而是通过SomeInterface来工作。更重要的是，**我们可以更改工厂返回的类型，而客户端代码无需更改**，甚至可以在运行时动态选择类型。

### 2.1 JVM中的示例

JVM中，这种模式最著名的例子可能是Collections类中的集合构建方法，例如singleton()、singletonList()和singletonMap()。这些方法都会返回相应集合的实例-Set、 List或Map，但具体类型无关紧要。此外，Stream.of()方法以及新的Set.of()、List.of()和Map.ofEntries()方法允许我们对更大的集合执行同样的操作。

还有许多其他这样的例子，包括Charset.forName()，它将根据所要求的名称返回Charset类的不同实例，以及ResourceBundle.getBundle()，它将根据提供的名称加载不同的资源包。

并非所有这些方法都需要提供不同的实例，有些只是为了隐藏内部工作原理而进行的抽象。例如，Calendar.getInstance()和NumberFormat.getInstance()总是返回同一个实例，但具体细节与客户端代码无关。

## 3. 抽象工厂

[抽象工厂](https://www.baeldung.com/java-abstract-factory-pattern)模式更进一步，所使用的工厂也具有抽象基类型。然后，我们可以基于这些抽象类型编写代码，并在运行时以某种方式选择具体的工厂实例。

首先，我们有一个接口和一些我们实际想要使用的功能的具体实现：

```java
interface FileSystem {
    // ...
}
```

```java
class LocalFileSystem implements FileSystem {
    // ...
}
```

```java
class NetworkFileSystem implements FileSystem {
    // ...
}
`````

```java`

接下来，我们有一个接口和一些具体的实现，供工厂获取上述内容：

```java
interface FileSystemFactory {
    FileSystem newInstance();
}
```

```java
class LocalFileSystemFactory implements FileSystemFactory {
    // ...
}
```

```java
class NetworkFileSystemFactory implements FileSystemFactory {
    // ...
}
```

然后我们有另一个工厂方法来获取抽象工厂，通过它我们可以获取实际的实例：

```java
class Example {
    static FileSystemFactory getFactory(String fs) {
        FileSystemFactory factory;
        if ("local".equals(fs)) {
            factory = new LocalFileSystemFactory();
        }
        else if ("network".equals(fs)) {
            factory = new NetworkFileSystemFactory();
        }
        return factory;
    }
}
```

这里，我们有一个FileSystemFactory接口，它有两个具体的实现。**我们在运行时选择具体的实现，但使用它的代码不需要关心实际使用了哪个实例**。它们各自返回一个FileSystem接口的不同具体实例，但同样，我们的代码不需要关心我们拥有的是哪个实例。

通常，我们会使用另一个工厂方法来获取工厂本身，如上所述。在本例中，getFactory()方法本身就是一个工厂方法，它返回一个抽象的FileSystemFactory，然后用它来构造一个FileSystem。

### 3.1 JVM中的示例

JVM中有很多这种设计模式的例子，最常见的是在XML包中，例如DocumentBuilderFactory、TransformerFactory和XPathFactory，**它们都有一个特殊的newInstance()工厂方法，允许我们的代码获取抽象工厂的实例**。

在内部，此方法使用了许多不同的机制-系统属性、JVM中的配置文件以及[服务提供者接口](https://www.baeldung.com/java-spi)来尝试并确定究竟要使用哪个具体实例。这样，我们就可以根据需要在应用程序中安装其他XML库，但这对实际使用它们的代码来说是透明的。

一旦我们的代码调用了newInstance()方法，它就会从相应的XML库中获取一个工厂实例。然后，该工厂会从同一个库中构建我们想要使用的实际类。

例如，如果我们使用JVM默认的Xerces实现，我们将获得com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl的实例，但如果我们想使用不同的实现，那么调用newInstance()将透明地返回该实现。

## 4. 构建器

当我们想要以更灵活的方式构造一个复杂的对象时，构建器模式非常有用。它的工作原理是，我们用一个单独的类来构建复杂的对象，并允许客户端使用更简单的接口来创建它：

```java
class CarBuilder {
    private String make = "Ford";
    private String model = "Fiesta";
    private int doors = 4;
    private String color = "White";

    public Car build() {
        return new Car(make, model, doors, color);
    }
}
```

这使我们能够单独提供make、model、doors和color的值，然后当我们构建Car时，所有构造函数参数都会解析为存储的值。

### 4.1 JVM中的示例

JVM中有一些非常关键的此模式示例，**StringBuilder和StringBuffer类是构建器，它们允许我们通过提供许多小部分来构造较长的字符串**。较新的Stream.Builder类允许我们执行完全相同的操作来构造Stream：

```java
Stream.Builder<Integer> builder = Stream.builder<Integer>();
builder.add(1);
builder.add(2);
if (condition) {
    builder.add(3);
    builder.add(4);
}
builder.add(5);
Stream<Integer> stream = builder.build();
```

## 5. 延迟初始化

我们使用延迟初始化模式来延迟某些值的计算，直到需要它为止。有时，这可能涉及单个数据，有时则可能涉及整个对象。

这在很多情况下都很有用，例如，**如果完整构造一个对象需要数据库或网络访问，而我们可能永远不需要使用它，那么执行这些调用可能会导致我们的应用程序性能不佳**。或者，如果我们计算大量可能永远不需要的值，那么这可能会导致不必要的内存占用。

通常，这是通过让一个对象成为我们需要的数据的惰性包装器，并在通过Getter方法访问时计算数据来实现的：

```java
class LazyPi {
    private Supplier<Double> calculator;
    private Double value;

    public synchronized Double getValue() {
        if (value == null) {
            value = calculator.get();
        }
        return value;
    }
}
```

计算π是一项开销很大的操作，我们可能并不需要执行，上面的代码会在我们第一次调用getValue()时执行此操作，而不是之前。

### 5.1 JVM中的示例

JVM中类似的例子相对较少，但是，Java 8中引入的[Streams API](https://www.baeldung.com/java-streams)就是一个很好的例子。**所有在流上执行的操作都是惰性的**，因此我们可以在这里执行一些昂贵的计算，并且确保它们只在需要时才会被调用。

然而，**流本身的实际生成也可以是惰性的**，Stream.generate()接收一个函数，在需要下一个值时调用，并且只在需要时调用。我们可以使用它来加载开销较大的值-例如，通过HTTP API调用，并且只在实际需要新元素时才需要成本：

```java
Stream.generate(new BaeldungArticlesLoader())
    .filter(article -> article.getTags().contains("java-streams"))
    .map(article -> article.getTitle())
    .findFirst();
```

这里，我们有一个供应商(Supplier)，它会通过HTTP调用来加载文章，并根据相关标签进行过滤，然后返回第一个匹配的标题。如果加载的第一篇文章符合此过滤器，则只需进行一次网络调用，无论实际存在多少篇文章。

## 6. 对象池

当构造一个对象的新实例时，如果创建过程可能开销很大，但重用现有实例是一个可行的替代方案，那么我们将使用对象池模式。与其每次都构造一个新实例，不如预先构造一组实例，然后在需要时使用它们。

**实际的对象池是为了管理这些共享对象而存在的**，它还会跟踪这些对象，以确保每个对象在同一时间只能在一个地方使用。在某些情况下，整个对象集合仅在开始时构建。在其他情况下，如果需要，池可能会按需创建新的实例。

### 6.1 JVM中的示例

**JVM中这种模式的主要示例是线程池的使用**，[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)会管理一组线程，并允许我们在需要执行某个任务时使用它们。使用这种方式意味着，每当我们需要生成异步任务时，无需创建新线程，也无需承担所有相关的开销：

```java
ExecutorService pool = Executors.newFixedThreadPool(10);

pool.execute(new SomeTask()); // Runs on a thread from the pool
pool.execute(new AnotherTask()); // Runs on a thread from the pool
```

这两个任务都会从线程池中分配一个线程来运行，这两个线程可能是同一个线程，也可能是完全不同的线程，至于使用哪个线程对我们的代码来说并不重要。

## 7. 原型

当我们需要创建与原始对象完全相同的新实例时，我们会使用原型模式。原始实例充当原型，并用于构造完全独立于原始实例的新实例。我们可以根据需要使用这些新实例。

**Java对此提供了一定程度的支持，它实现了Cloneable标记接口**，然后使用Object.clone()。这将生成对象的浅克隆，创建一个新实例，并直接复制字段。

这种方法开销较小，但缺点是，对象内部任何已自行构建的字段都将属于同一个实例，这意味着对这些字段的更改也会发生在所有实例上。不过，如果需要，我们随时可以自行覆盖此方法：

```java
public class Prototype implements Cloneable {
    private Map<String, String> contents = new HashMap<>();

    public void setValue(String key, String value) {
        // ...
    }
    public String getValue(String key) {
        // ...
    }

    @Override
    public Prototype clone() {
        Prototype result = new Prototype();
        this.contents.entrySet().forEach(entry -> result.setValue(entry.getKey(), entry.getValue()));
        return result;
    }
}
```

### 7.1 JVM中的示例

JVM中有一些类似的例子，我们可以通过跟踪实现Cloneable接口的类来查看。例如，PKIXCertPathBuilderResult、PKIXBuilderParameters、PKIXParameters、PKIXCertPathBuilderResult和PKIXCertPathValidatorResult都是Cloneable。

另一个例子是java.util.Date类，值得注意的是，**它重写了Object.clone()方法，以便复制额外的transient字段**。

## 8. 单例

单例模式通常用于这样的情况：一个类只能有一个实例，并且该实例应该在整个应用程序中都可以访问。通常，我们会使用一个静态实例来管理它，并通过静态方法访问该实例：

```java
public class Singleton {
    private static Singleton instance = null;

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

根据具体需求，可以有几种变化-例如，实例是在启动时还是在首次使用时创建，访问它是否需要线程安全，以及每个线程是否需要有不同的实例。

### 8.1 JVM中的示例

JVM中有一些这方面的例子，它们代表了JVM本身的核心部分-Runtime、Desktop和SecurityManager，这些类都具有访问器方法，用于返回相应类的单个实例。

**此外，Java反射API的大部分功能都适用于单例实例**。同一个实际类始终返回同一个Class实例，无论是使用Class.forName()、String.class还是通过其他反射方法访问。

类似地，我们可以将表示当前线程的Thread实例视为单例。通常会有多个这样的实例，但根据定义，每个线程只有一个实例。在同一线程中的任何位置调用Thread.currentThread()都将始终返回同一个实例。

## 9. 总结

在本文中，我们了解了用于创建和获取对象实例的各种设计模式。我们还研究了这些模式在核心JVM中的应用示例，以便了解它们如何帮助许多应用程序从中受益。