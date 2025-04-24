---
layout: post
title:  Java 21中的新特性
category: java-new
copyright: java-new
excerpt: Java 21
---

## 1. 概述

在本文中，我们将讨论Java 21中添加的新功能和增强功能。

Java 21于2023年9月19日发布，是继上一个[Java 17](https://www.baeldung.com/java-17-new-features)之后的最新LTS版本。

## 2. JEP列表

接下来，我们将讨论最显著的增强功能，也称为Java增强提案(JEP)，即该语言引入的新版本。

### 2.1 记录模式([JEP 440](https://openjdk.org/jeps/440))

[记录模式](https://www.baeldung.com/java-19-record-patterns)作为预览功能包含在[Java 19](https://www.baeldung.com/java-19-record-patterns)和[Java 20](https://www.baeldung.com/java-20-new-features#record-patterns-jep-432)中，现在，随着Java 21的推出，它们不再是预览版本，并且包含一些改进。

**此JEP扩展了现有的模式匹配功能，以解构记录类实例，从而可以编写复杂的数据查询**。此外，此JEP增加了对嵌套模式的支持，以实现更多可组合的数据查询。

例如，我们可以更简洁地编写代码：
```java
record Point(int x, int y) {}
    
public static int beforeRecordPattern(Object obj) {
    int sum = 0;
    if(obj instanceof Point p) {
        int x = p.x();
        int y = p.y();
        sum = x+y;
    }
    return sum;
}
    
public static int afterRecordPattern(Object obj) {
    if(obj instanceof Point(int x, int y)) {
        return x+y;
    }
    return 0;
}
```

因此，我们还可以在记录中嵌套记录并对其进行解构：
```java
enum Color {RED, GREEN, BLUE}
record ColoredPoint(Point point, Color color) {}
```

```java
record RandomPoint(ColoredPoint cp) {}
public static Color getRamdomPointColor(RandomPoint r) {
    if(r instanceof RandomPoint(ColoredPoint cp)) {
        return cp.color();
    }
    return null;
}
```

上面，我们直接从ColoredPoint访问.color()方法。

### 2.2 Switch模式匹配([JEP 441](https://openjdk.org/jeps/441))

[Switch模式匹配](https://www.baeldung.com/java-switch-pattern-matching)最初在JDK 17中引入，在JDK 18、19和20中得到完善，并在JDK 21中得到进一步改进。

**此功能的主要目标是允许在Switch case标签中使用模式，并提高Switch语句和表达式的表达能力**。此外，还通过允许使用null case标签来增强处理NullPointerException的能力。

让我们通过一个例子来探讨一下，假设我们有一个Account类：
```java
static class Account{
    double getBalance(){
       return 0;
    }
}
```

我们将其扩展到各种类型的账户，每种账户都有其计算余额的方法：
```java
static class SavingsAccount extends Account {
    double getSavings() {
        return 100;
    }
}
static class TermAccount extends Account {
    double getTermAccount() {
        return 1000;
    } 
}
static class CurrentAccount extends Account {
    double getCurrentAccount() {
        return 10000;
    } 
}
```

在Java 21之前，我们可以使用以下代码来获取余额：
```java
static double getBalanceWithOutSwitchPattern(Account account) {
    double balance = 0;
    if(account instanceof SavingsAccount sa) {
        balance = sa.getSavings();
    }
    else if(account instanceof TermAccount ta) {
        balance = ta.getTermAccount();
    }
    else if(account instanceof CurrentAccount ca) {
        balance = ca.getCurrentAccount();
    }
    return balance;
}
```

上面的代码表达能力不太强，因为if–else语句中有很多噪音。在Java 21中，我们可以利用case标签中的模式更简洁地编写相同的逻辑：
```java
static double getBalanceWithSwitchPattern(Account account) {
    double result = 0;
    switch (account) {
        case null -> throw new RuntimeException("Oops, account is null");
        case SavingsAccount sa -> result = sa.getSavings();
        case TermAccount ta -> result = ta.getTermAccount();
        case CurrentAccount ca -> result = ca.getCurrentAccount();
        default -> result = account.getBalance();
    };
    return result;
}
```

为了确认我们所写的是正确的，让我们通过测试来证明这一点：
```java
SwitchPattern.SavingsAccount sa = new SwitchPattern.SavingsAccount();
SwitchPattern.TermAccount ta = new SwitchPattern.TermAccount();
SwitchPattern.CurrentAccount ca = new SwitchPattern.CurrentAccount();

assertEquals(SwitchPattern.getBalanceWithOutSwitchPattern(sa), SwitchPattern.getBalanceWithSwitchPattern(sa));
assertEquals(SwitchPattern.getBalanceWithOutSwitchPattern(ta), SwitchPattern.getBalanceWithSwitchPattern(ta));
assertEquals(SwitchPattern.getBalanceWithOutSwitchPattern(ca), SwitchPattern.getBalanceWithSwitchPattern(ca));
```

因此，我们只是更简洁地重写了一个Switch。

模式case标签还支持与相同标签的表达式匹配，例如，假设我们需要处理包含简单“Yes”或“No”的输入字符串：
```java
static String processInputOld(String input) {
    String output = null;
    switch(input) {
        case null -> output = "Oops, null";
        case String s -> {
            if("Yes".equalsIgnoreCase(s)) {
                output = "It's Yes";
            }
            else if("No".equalsIgnoreCase(s)) {
                output = "It's No";
            }
            else {
                output = "Try Again";
            }
        }
    }
    return output;
}
```

再次，我们可以看到编写if–else逻辑可能会变得很难看。相反，在Java 21中，我们可以使用when子句以及case标签来将标签的值与表达式进行匹配：
```java
static String processInputNew(String input) {
    String output = null;
    switch(input) {
        case null -> output = "Oops, null";
        case String s when "Yes".equalsIgnoreCase(s) -> output = "It's Yes";
        case String s when "No".equalsIgnoreCase(s) -> output = "It's No";
        case String s -> output = "Try Again";
    }
    return output;
}
```

### 2.3 字符串文字([JEP 430](https://openjdk.org/jeps/430))

Java提供了多种使用字符串文字和表达式来组成字符串的机制，其中包括字符串拼接、StringBuilder类、String类format()方法和MessageFormat类。

**Java 21引入了字符串模板**，这些模板通过将文字文本与模板表达式和模板处理器相结合来产生所需的结果，从而补充了Java现有的字符串文字和文本块。

让我们看一个例子：
```java
String name = "Tuyucheng";
String welcomeText = STR."Welcome to \{name}";
System.out.println(welcomeText);
```

上面的代码片段打印文本“Welcome to Tuyucheng”。

![](/assets/images/2025/javanew/javalts21newfeatures01.png)

在上图中，我们有一个模板处理器(STR)、一个点字符和一个包含嵌入表达式({name})的模板。在运行时，当模板处理器评估模板表达式时，它会将模板中的文字文本与嵌入表达式的值结合起来以产生结果。

STR是Java的模板处理器之一，它会自动导入到所有Java源文件中，Java还提供FMT和RAW模板处理器。

### 2.4 虚拟线程([JEP 444](https://openjdk.org/jeps/444))

虚拟线程最初在Java 19中作为预览功能引入到Java语言中，并在Java 20中得到进一步完善，Java 21引入了一些新的变化。

**虚拟线程是轻量级线程，旨在减少开发高并发应用程序的工作量**。传统线程(也称为平台线程)是操作系统线程的薄包装器，平台线程的主要问题之一是它们在操作系统线程上运行代码并在操作系统线程的整个生命周期内捕获操作系统线程，操作系统线程的数量是有限的，这造成了可伸缩性瓶颈。

**与平台线程一样，虚拟线程也是java.lang.Thread类的一个实例，但它并不与特定的操作系统线程绑定**。它在特定的操作系统线程上运行代码，但不会在整个生命周期内捕获线程。因此，多个虚拟线程可以共享操作系统线程来运行其代码。

让我们通过一个例子来看一下虚拟线程的使用：
```java
try(var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.rangeClosed(1, 10_000).forEach(i -> {
        executor.submit(() -> {
            System.out.println(i);
            try {
                Thread.sleep(Duration.ofSeconds(1));
            } 
            catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    });
}
```

在上面的代码片段中，我们使用了静态的newVirtualThreadPerTaskExecutor()方法，此执行器为每个任务创建一个新的虚拟线程，因此在上面的示例中，我们创建了10000个 虚拟线程。

Java 21对虚拟线程进行了两项显著的改变：

- 虚拟线程现在始终支持线程局部变量。
- 通过Thread.Builder API创建的虚拟线程也会在其整个生命周期内受到监控，并且可以在新的线程转储中观察到。

### 2.5 序列集合([JEP 431](https://openjdk.org/jeps/431))

在Java集合框架中，没有任何集合类型能够表示具有明确遇到顺序的元素序列。例如，List和Deque接口定义了出现顺序，但它们的通用超类型Collection却没有定义。同样，Set也没有定义出现顺序，但LinkedHashSet或SortedSet等子类型却有定义。

**Java 21引入了3个新接口来表示序列列表、序列集合和序列映射**。

[序列集合](https://www.baeldung.com/java-21-sequenced-collections)是元素具有定义遇到顺序的集合，它有第一个元素和最后一个元素，并且位于第一个元素和最后一个元素之间的元素具有后继元素和前继元素。序列集合是没有重复元素的有序集合，序列映射是条目具有定义遇到顺序的映射。

下图展示了集合框架层次结构中新引入的接口的改造：

![](/assets/images/2025/javanew/javalts21newfeatures02.png)

### 2.6 密钥封装机制API([JEP 452](https://openjdk.org/jeps/452))

密钥封装是一种使用非对称密钥或公钥加密来保护对称密钥的技术。

传统方法是使用公钥来保护随机生成的对称密钥，然而，这种方法需要填充，这很难证明其安全性。

密钥封装机制(KEM)使用公钥来派生不需要任何填充的对称密钥。

**Java 21引入了新的KEM API，使应用程序能够使用KEM算法**。

## 3. 总结

在本文中，我们讨论了Java 21中的一些显著变化。

我们讨论了记录模式、Switch模式匹配、字符串模板、序列集合、虚拟线程、字符串模板和新的KEM API。

其他增强和改进遍布JDK 21软件包和类，不过，本文应该是探索Java 21新功能的一个很好的起点。