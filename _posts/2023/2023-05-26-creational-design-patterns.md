---
layout: post
title:  创建型设计模式简介
category: designpattern
copyright: designpattern
excerpt: Design Pattern
---

## 1. 简介

在软件工程中，设计模式描述了针对软件设计中最常见问题的既定解决方案。它代表了经验丰富的软件开发人员经过长期反复试验而长期发展起来的最佳实践。

在Erich Gamma、John Vlissides、Ralph Johnson和Richard Helm(也称为四人帮或GoF)于1994年出版了《[设计模式：可重用面向对象软件的元素](https://books.google.co.in/books?id=K4qv1D-LKhoC&lpg=PP1&dq=Design Patterns%3A Elements of Reusable Object-Oriented Software&pg=PP1#v=onepage&q=Design Patterns: Elements of Reusable Object-Oriented Software&f=false)》一书后，设计模式开始流行起来。

在本文中，我们将探讨创建型设计模式及其类型，并通过一些代码示例讨论这些模式适合我们设计的情况。

## 2. 创建型设计模式

**创建型设计模式与创建对象的方式有关**，它们通过以受控方式创建对象来降低复杂性和不稳定性。

new运算符通常被认为是有害的，因为它将对象分散在整个应用程序中。随着时间的推移，更改实现可能变得具有挑战性，因为类变得紧密耦合。

创建型设计模式通过将客户端与实际的初始化过程完全分离来解决这个问题。

在本文中，我们将讨论四种类型的创建型设计模式：

1.  单例：确保在整个应用程序中最多只存在对象的一个实例
2.  工厂方法：创建多个相关类的对象，而不指定要创建的确切对象
3.  抽象工厂：创建相关依赖对象的族
4.  建造者：使用循序渐进的方法构建复杂的对象

## 3. 单例设计模式

单例设计模式旨在通过**确保整个Java虚拟机中只存在对象的一个实例**来检查特定类对象的初始化。

Singleton类还提供了一个唯一的全局对象访问点，因此每次对该访问点的后续调用都只返回该特定对象。

### 3.1 单例模式示例

虽然单例模式是由GoF引入的，但原来的实现在多线程场景中是有问题的。

所以在这里，我们将遵循一种使用静态内部类的更优化的方法：

```java
public class Singleton {
    private Singleton() {
    }

    private static class SingletonHolder {
        public static final Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
}
```

在这里，我们创建了一个静态内部类来保存Singleton类的实例。它仅在有人调用getInstance()方法时创建实例，而不是在加载外部类时创建实例。

对于Singleton类来说，这是一种广泛使用的方法，因为它不需要同步、线程安全、强制延迟初始化并且样板代码相对较少。

**另外，请注意构造函数具有私有访问修饰符。这是创建单例的要求，因为公共构造函数意味着任何人都可以访问它并开始创建新实例**。

请记住，这不是最初的GoF实现。对于原始版本，请访问[这篇]()关于Java单例的文章。

### 3.2 何时使用单例设计模式

-   对于创建成本高昂的资源(如数据库连接对象)
-   将所有日志记录对象保持为单例是提高性能的好习惯
-   提供对应用程序配置设置的访问的类
-   包含以共享模式访问的资源的类

## 4. 工厂方法设计模式

工厂设计模式或工厂方法设计模式是Java中最常用的设计模式之一。

根据GoF的说法，这种模式“**定义了一个用于创建对象的接口，但让子类决定实例化哪个类**。工厂方法允许类将实例化推迟到子类”。

此模式通过创建一种虚拟构造函数，将初始化类的责任从客户端委托给特定的工厂类。

为了实现这一点，我们依靠一个为我们提供对象的工厂，隐藏实际的实现细节。创建的对象使用公共接口进行访问。

### 4.1 工厂方法设计模式示例

在这个例子中，我们将创建一个Polygon接口，该接口将由多个具体类实现。PolygonFactory用于从这个实现系列中获取对象：

![](/assets/images/2023/designpattern/creationaldesignpatterns01.png)

首先我们创建Polygon接口：

```java
public interface Polygon {
    String getType();
}
```

接下来，我们创建一些实现此接口的实现，如Square、Triangle等，并返回一个Polygon类型的对象。

现在我们可以创建一个工厂，它将numberOfSides(边数)作为参数并返回此接口的适当实现：

```java
public class PolygonFactory {
    public Polygon getPolygon(int numberOfSides) {
        if (numberOfSides == 3) {
            return new Triangle();
        }
        if (numberOfSides == 4) {
            return new Square();
        }
        if (numberOfSides == 5) {
            return new Pentagon();
        }
        if (numberOfSides == 7) {
            return new Heptagon();
        } else if (numberOfSides == 8) {
            return new Octagon();
        }
        return null;
    }
}
```

注意客户端如何依赖这个工厂给我们一个合适的Polygon，而不必直接初始化对象。

### 4.2 何时使用工厂方法设计模式

-   当接口或抽象类的实现预计频繁更改时
-   当当前实现无法轻松地适应新变化时
-   当初始化过程相对简单，构造函数只需要少数几个参数时

## 5. 抽象工厂设计模式

在上一节中，我们了解了如何使用工厂方法设计模式来创建与单个家族相关的对象。

相比之下，抽象工厂设计模式用于创建相关或依赖对象的族，它有时也被称为工厂的工厂。

有关详细说明，请查看我们的[抽象工厂]()教程。

## 6. 建造者设计模式

建造者设计模式是另一种创建型模式，旨在处理相对复杂对象的构造。

**当创建对象的复杂性增加时，Builder模式可以分离出实例化过程，使用另一个对象(builder)来构造对象**。

然后可以使用此构建器通过简单的逐步方法创建许多其他类似的表示。

### 6.1 建造者模式示例

GoF引入的原始建造者设计模式侧重于抽象，在处理复杂对象时非常好，但是设计有点复杂。

Joshua Bloch在他的Effective Java一书中介绍了建造者模式的改进版本，该版本简洁、可读性强(因为它使用了[流式设计](https://en.wikipedia.org/wiki/Fluent_interface))并且从客户的角度来看易于使用。在本例中，我们将讨论该版本。

这个例子只有一个类BankAccount，它包含一个构建器作为静态内部类(BankAccountBuilder)：

```java
public class BankAccount {

    private String name;
    private String accountNumber;
    private String email;
    private boolean newsletter;

    // constructors/getters

    public static class BankAccountBuilder {
        // builder code
    }
}
```

请注意，字段上的所有访问修饰符都声明为私有的，因为我们不希望外部对象直接访问它们。

构造函数也是私有的，因此只有分配给此类的Builder才能访问它。构造函数中设置的所有属性都是从我们作为参数提供的构建器对象中提取的。

我们在静态内部类中定义了BankAccountBuilder ：

```java
public static class BankAccountBuilder {

    private String name;
    private String accountNumber;
    private String email;
    private boolean newsletter;

    public BankAccountBuilder(String name, String accountNumber) {
        this.name = name;
        this.accountNumber = accountNumber;
    }

    public BankAccountBuilder withEmail(String email) {
        this.email = email;
        return this;
    }

    public BankAccountBuilder wantNewsletter(boolean newsletter) {
        this.newsletter = newsletter;
        return this;
    }

    public BankAccount build() {
        return new BankAccount(this);
    }
}
```

请注意，我们已经声明了外部类包含的同一组字段。任何必填字段都需要作为内部类构造函数的参数，而其余可选字段可以使用Setter方法指定。

此实现还通过让Setter方法返回构建器对象来支持流式的设计方法。

最后，build方法调用外部类的私有构造函数并将其自身作为参数传递。返回的BankAccount将使用BankAccountBuilder设置的参数进行实例化。

下面是一个建造者模式的快速示例：

```java
BankAccount newAccount = new BankAccount
        .BankAccountBuilder("Jon", "22738022275")
        .withEmail("jon@example.com")
        .wantNewsletter(true)
        .build();
```

### 6.2 何时使用建造者模式

1.  当创建对象所涉及的过程非常复杂，有很多强制和可选参数时
2.  当构造函数参数数量增加导致构造函数列表很大时
3.  当客户期望为构建的对象提供不同的表示形式时

## 7. 总结

在本文中，我们了解了Java中的创建型设计模式，并讨论了它们的四种不同类型，即单例、工厂方法、抽象工厂和建造者，它们的优点、示例以及我们应该何时使用它们。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/design-patterns-modules)上获得。