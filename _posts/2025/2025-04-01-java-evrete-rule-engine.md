---
layout: post
title:  Evrete规则引擎简介
category: libraries
copyright: libraries
excerpt: Evrete
---

## 1. 简介

本文首次对Evrete(一种新的开源Java规则引擎)进行了实际概述。

从历史上看，[Evrete](https://www.evrete.org/)是作为[Drools规则引擎](https://www.baeldung.com/java-rule-engines#drools)的轻量级替代品而开发的，它完全符合[Java规则引擎规范](https://jcp.org/en/jsr/detail?id=94)，并使用经典的前向链接RETE算法，并进行了一些调整和功能来处理大量数据。

它需要Java 8及更高版本，零依赖，无缝操作JSON和XML对象，并**允许函数接口作为规则的条件和操作**。

它的大多数组件都可以通过服务提供者接口进行扩展，其中一个SPI实现将带注解的Java类转换为可执行规则集。

## 2. Maven依赖

在跳转到Java代码之前，我们需要在项目的pom.xml中声明[evrete-core](https://mvnrepository.com/artifact/org.evrete/evrete-core) Maven依赖：

```xml
<dependency>
    <groupId>org.evrete</groupId>
    <artifactId>evrete-core</artifactId>
    <version>3.0.01</version>
</dependency>
```

## 3. 用例场景

为了使介绍不那么抽象，我们假设我们经营一家小企业，今天是财政年度的结束，我们想要计算每个客户的总销售额。

我们的域数据模型将包括两个简单的类-Customer和Invoice：

```java
public class Customer {
    private double total = 0.0;
    private final String name;

    public Customer(String name) {
        this.name = name;
    }

    public void addToTotal(double amount) {
        this.total += amount;
    }
    // getters and setters
}
```

```java
public class Invoice {
    private final Customer customer;
    private final double amount;

    public Invoice(Customer customer, double amount) {
        this.customer = customer;
        this.amount = amount;
    }
    // getters and setters
}
```

附带说明一下，该引擎开箱即用地支持[Java记录](https://www.baeldung.com/java-16-new-features#records-jep-395)，并**允许开发人员将任意类属性声明为函数接口**。

在本介绍的后面部分，我们将获得一系列发票和客户，逻辑表明我们需要两个规则来处理数据：

- 第一条规则清除每个客户的总销售值
- 第二条规则匹配发票和客户并更新每个客户的总额

再次，我们将使用流式的规则构建器接口和带注解的Java类来实现这些规则。让我们从规则构建器API开始。

## 4. 规则构建器API

规则构建器是开发规则领域特定语言(DSL)的核心构建块，开发人员在解析Excel源、纯文本或需要转换为规则的任何其他DSL格式时会使用它们。

不过，在我们的案例中，我们主要感兴趣的是他们将规则直接嵌入到开发人员的代码中的能力。

### 4.1 规则集声明

使用规则构建器，我们可以使用流式的接口声明我们的两个规则：

```java
KnowledgeService service = new KnowledgeService();
Knowledge knowledge = service
        .newKnowledge()
        .newRule("Clear total sales")
        .forEach("$c", Customer.class)
        .execute(ctx -> {
            Customer c = ctx.get("$c");
            c.setTotal(0.0);
        })
        .newRule("Compute totals")
        .forEach(
                "$c", Customer.class,
                "$i", Invoice.class
        )
        .where("$i.customer == $c")
        .execute(ctx -> {
            Customer c = ctx.get("$c");
            Invoice i = ctx.get("$i");
            c.addToTotal(i.getAmount());
        });
```

首先，我们创建了一个KnowledgeService实例，它本质上是一个共享的执行器服务。通常，**每个应用程序都应该有一个KnowledgeService实例**。

生成的Knowledge实例是我们两个规则的预编译版本，我们这样做的原因与我们通常编译源代码的原因相同-确保正确性并更快地启动代码。

熟悉Drools规则引擎的人会发现我们的规则声明在语义上等同于以下相同逻辑的DRL版本：

```text
rule "Clear total sales"
  when
    $c: Customer
  then
    $c.setTotal(0.0);
end

rule "Compute totals"
  when
    $c: Customer
    $i: Invoice(customer == $c)
  then
    $c.addToTotal($i.getAmount());
end
```

### 4.2 模拟测试数据

我们将针对3名客户和10万张发票测试我们的规则集，这些发票的金额是随机的，并且在客户之间随机分布：

```java
List<Customer> customers = Arrays.asList(
    new Customer("Customer A"),
    new Customer("Customer B"),
    new Customer("Customer C")
);

Random random = new Random();
Collection<Object> sessionData = new LinkedList<>(customers);
for (int i = 0; i < 100_000; i++) {
    Customer randomCustomer = customers.get(random.nextInt(customers.size()));
    Invoice invoice = new Invoice(randomCustomer, 100 * random.nextDouble());
    sessionData.add(invoice);
}
```

现在，sessionData变量包含我们将插入到规则会话中的Customer和Invoice实例。

### 4.3 规则执行

现在我们需要做的就是将所有100003个对象(10万张发票加上3个客户)提供给一个新的会话实例并调用其fire()方法：

```java
knowledge
    .newStatelessSession()
    .insert(sessionData)
    .fire();

for(Customer c : customers) {
    System.out.printf("%s:\t$%,.2f%n", c.getName(), c.getTotal());
}
```

最后几行将打印每个客户的销售量：

```text
Customer A:	$1,664,730.73
Customer B:	$1,666,508.11
Customer C:	$1,672,685.10
```

## 5. 基于注解的Java规则

尽管我们之前的示例按预期工作，但它并没有使库符合规范，该规范要求规则引擎能够：

- 通过外部化业务或应用程序逻辑来促进声明式编程。
- 包括记录的文件格式或编写规则的工具，以及应用程序外部的规则执行集。

简而言之，这意味着兼容的规则引擎必须能够执行在其运行时之外编写的规则。

[Evrete](https://www.evrete.org/)的基于注解的Java规则扩展模块满足了这一要求，**该模块实际上是一个“展示”DSL，它完全依赖于库的核心API**。

让我们看看它是如何工作的。

### 5.1 安装

基于注解的Java规则是[Evrete](https://www.evrete.org/)服务提供者接口(SPI)之一的实现，并且需要额外的[evrete-dsl-java](https://mvnrepository.com/artifact/org.evrete/evrete-dsl-java) Maven依赖：

```xml
<dependency>
    <groupId>org.evrete</groupId>
    <artifactId>evrete-dsl-java</artifactId>
    <version>3.0.01</version>
</dependency>
```

### 5.2 规则集声明

让我们使用注解创建相同的规则集，我们将选择纯Java源代码而不是类和捆绑的jar：

```java
public class SalesRuleset {

    @Rule
    public void rule1(Customer $c) {
        $c.setTotal(0.0);
    }

    @Rule
    @Where("$i.customer == $c")
    public void rule2(Customer $c, Invoice $i) {
        $c.addToTotal($i.getAmount());
    }
}
```

此源文件可以有任意名称，无需遵循Java命名约定。引擎将即时编译源代码，我们需要确保：

- 我们的源文件包含所有必要的导入
- 第三方依赖和域类位于引擎的类路径上

然后我们告诉引擎从外部位置读取我们的规则集定义：

```java
KnowledgeService service = new KnowledgeService();
URL rulesetUrl = new URL("ruleset.java"); // or file.toURI().toURL(), etc
Knowledge knowledge = service.newKnowledge(
    "JAVA-SOURCE",
    rulesetUrl
);
```

就是这样，假设我们的其余代码保持不变，我们将打印相同的3个客户及其随机销售量。

关于这个特定示例的几点说明：

- 我们选择从纯Java(“JAVA-SOURCE”参数)构建规则，从而允许引擎从方法参数中推断事实名称。
- 如果我们选择.class或.jar源，方法参数将需要@Fact注解。
- 引擎已自动按方法名称对规则进行排序，如果我们交换名称，重置规则将清除之前计算的销量。因此，我们将看到0销量。

### 5.3 工作原理

每当创建新会话时，引擎都会将其与注解规则类的新实例耦合。本质上，我们可以将这些类的实例视为会话本身。

因此，如果定义了类变量，则规则方法可以访问它。

如果我们定义条件方法或将新字段声明为方法，那么这些方法也可以访问类变量。

与常规Java类一样，此类规则集可以扩展、重用并打包到库中。

### 5.4 附加功能

简单示例非常适合介绍，但遗漏了许多重要主题。对于基于注解的Java规则，这些包括：

- 条件作为类方法
- 任意属性声明为类方法
- 阶段监听器、继承模型以及对运行时环境的访问
- 最重要的是，全面使用类字段-从条件到动作和字段定义

## 6. 总结

在本文中，我们简要测试了一个新的Java规则引擎，关键要点包括：

1. 其他引擎可能更擅长提供即用型DSL解决方案和规则仓库
2. [Evrete](https://www.evrete.org/)是为开发人员构建任意DSL而设计的
3. 那些习惯用Java编写规则的人可能会发现“基于注解的Java规则”包是一个更好的选择

值得一提的是本文未涉及但在库的API中提到的其他功能：

- 声明任意事实属性
- 条件作为Java谓词
- 动态更改规则条件和操作
- 冲突解决技巧
- 将新规则附加到实时会话
- 库扩展接口的自定义实现

官方文档位于https://www.evrete.org/docs/。