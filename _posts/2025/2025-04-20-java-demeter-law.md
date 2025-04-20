---
layout: post
title:  Java中的迪米特法则
category: designpattern
copyright: designpattern
excerpt: 迪米特法则
---

## 1. 概述

迪米特法则(LoD)，即最少知识原则，为模块化软件开发提供了[面向对象](https://www.baeldung.com/java-oop)的设计原则，它有助于构建相互依赖性较低且松散耦合的组件。

在本教程中，我们将深入研究迪米特法则及其在Java中的应用。

## 2. 理解迪米特法则

迪米特法则是面向对象编程中的几条设计准则之一，**它建议对象应避免访问其他对象的内部数据和方法**。相反，对象应该只与其直接依赖的对象进行交互。

这一概念最初是由Karl J.Lieberherr等人在一篇[论文](https://www2.ccs.neu.edu/research/demeter/papers/law-of-demeter/oopsla88-law-of-demeter.pdf)中提出的，论文指出：

> 对于所有类C以及附加到C的所有方法M，M向其发送消息的所有对象都必须是与以下类关联的类的实例：
>
> - M的参数类(包括C)
> - C类的实例变量
>
> (由M创建的对象，或由M调用的函数或方法创建的对象，以及全局变量中的对象被视为M的参数。)

简而言之，该定律规定类C的方法X只能调用以下方法：

- C类本身
- X创建的对象
- 作为参数传递给X的对象
- C的实例变量中保存的对象
- 静态字段

该定律概括为五点。

## 3. Java中的迪米特法则示例

在上一节中，我们将迪米特法则提炼为五条关键规则，让我们用一些示例代码来说明这些要点。

### 3.1 第一条规则

**第一条规则规定，类C的方法X应该只调用C的方法**：

```java
class Greetings {
    
    String generalGreeting() {
        return "Welcome" + world();
    }
    String world() {
        return "Hello World";
    }
}
```

这里，generalGreeting()方法调用了同一个类中的world()方法。这符合规律，因为它们属于同一个类。

### 3.2 第二条规则

**类C的方法X应该只调用由X创建的对象的方法**：

```java
String getHelloBrazil() {
    HelloCountries helloCountries = new HelloCountries();
    return helloCountries.helloBrazil();
}
```

在上面的代码中，我们创建了一个HelloCountries对象，并在其上调用了helloBrazil()方法，这符合getHelloBrazil()方法本身创建对象的规律。

### 3.3 第三条规则

此外，**第三条规则规定方法X应该只调用作为参数传递给X的对象**：

```java
String getHelloIndia(HelloCountries helloCountries) {
    return helloCountries.helloIndia();
}
```

这里，我们将一个HelloCountries对象作为参数传递给getHelloIndia()，将对象作为参数传递使得该方法与对象更加亲近，并且可以在不违反迪米特法则的情况下调用其方法。

### 3.4 第四条规则

**类C的方法X应该只调用C的实例变量中保存的对象的方法**：

```java
// ... 
HelloCountries helloCountries = new HelloCountries();
  
String getHelloJapan() {
    return helloCountries.helloJapan();
}
// ...
```

在上面的代码中，我们在Greetings类中创建了一个实例变量“helloCountries”。然后，我们在getHelloJapan()方法中对该实例变量调用了helloJapan()方法，这符合第四条规则。

### 3.5 第五条规则

最后，**类C的方法X可以调用在C中创建的静态字段的方法**：

```java
// ...
static HelloCountries helloCountriesStatic = new HelloCountries();
    
String getHellStaticWorld() {
    return helloCountriesStatic.helloStaticWorld();
}
// ...
```

这里，该方法在类中创建的静态对象上调用helloStaticWorld()方法。

## 4. 违反迪米特法则

让我们检查一些违反迪米特法则的示例代码并寻找可能的修复方法。

### 4.1 设置

我们首先定义一个Employee类：

```java
class Employee {
  
    private Department department = new Deparment();
  
    public Department getDepartment() {
        return department;
    }
}
```

Employee类包含一个对象引用成员变量并为其提供访问器方法。

接下来，Department类定义有以下成员变量和方法：

```java
class Department {
    private Manager manager = new Manager();
  
    public Manager getManager() {
        return manager; 
    }
}
```

此外，Manager类包含批准费用的方法：

```java
class Manager {
    public void approveExpense(Expenses expenses) {
        System.out.println("Total amounts approved" + expenses.total())
    }
}
```

最后，我们来看看Expenses类：

```java
class Expenses {
    
    private double total;
    private double tax;
    
    public Expenses(double total, double tax) {
        this.total = total;
        this.tax = tax;
    }
    
    public double total() {
        return total + tax;
    }
}
```

### 4.2 使用

这些类通过它们之间的关系表现出紧密[耦合](https://www.baeldung.com/java-coupling-classes-tight-loose)，我们将通过让Manager批准Expenses来演示迪米特法则的违反：

```java
Expenses expenses = new Expenses(100, 10); 
Employee employee = new Employee();
employee.getDepartment().getManager().approveExpense(expenses);
```

上面的代码中，**我们进行了链式调用，这违反了迪米特法则**。类之间紧密耦合，无法独立运行。

让我们通过将Manager类作为Employee中的实例变量来修复此违规，它将通过Employee构造函数传入。然后，我们将在Employee类中创建submitExpense()方法，并在其中调用Manager的approveExpense()方法：

```java
// ...
private Manager manager;
Employee(Manager manager) {
    this.manager = manager;
}
    
void submitExpense(Expenses expenses) {
    manager.approveExpense(expenses);
}
// ...
```

新的用法如下：

```java
Manager mgr = new Manager();
Employee emp = new Employee(mgr);
emp.submitExpense(expenses);
```

这种修改后的方法遵循迪米特法则，减少了类之间的耦合，并促进了更加模块化的设计。

## 5. 迪米特法则的例外

链式调用通常意味着违反迪米特法则，但也有例外。例如，**如果[构建器模式](https://www.baeldung.com/creational-design-patterns#builder)是在本地实例化的，则它并不违反迪米特法则**。其中一条规则规定：“类C的方法X应该只调用由X创建的对象的方法”。

此外，[流式API](https://www.baeldung.com/java-fluent-interface-vs-builder-pattern)中存在链式调用，**如果链式调用针对的是本地创建的对象，则流式API不会违反迪米特法则**。但是，当链式调用针对的是非本地实例化的对象或返回不同的对象时，就会违反迪米特法则。

此外，在处理数据结构时，有些情况下我们可能会违反迪米特法则。**典型的数据结构用法，例如实例化、修改和本地访问，并不违反迪米特法则**。但当我们调用从数据结构中获取的对象的方法时，就可能违反迪米特法则。

## 6. 总结

在本文中，我们学习了迪米特法则的应用以及如何在面向对象代码中遵循它。此外，我们还通过代码示例深入探讨了每条法则，迪米特法则通过将对象交互限制在直接依赖关系中来促进松散耦合。