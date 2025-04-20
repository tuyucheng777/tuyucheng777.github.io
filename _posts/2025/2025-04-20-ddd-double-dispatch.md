---
layout: post
title:  DDD中的双重调度
category: ddd
copyright: ddd
excerpt: DDD
---

## 1. 概述

双重分派是一个技术术语，**用于描述根据接收者和参数类型选择要调用的方法的过程**。

许多开发人员经常将双重分派与[策略模式](https://en.wikipedia.org/wiki/Strategy_pattern)混淆。

Java不支持双重调度，但我们可以采用一些技术来克服这一限制。

**在本教程中，我们将重点展示领域驱动设计(DDD)中的双重分派和策略模式示例**。

## 2. 双重调度

在讨论双重调度之前，让我们先回顾一些基础知识并解释一下单重调度实际上是什么。

### 2.1 单重调度

**单重分派是一种根据接收方运行时类型选择方法实现的方式**，在Java中，这基本上与多态相同。

例如，我们来看看这个简单的折扣策略接口：

```java
public interface DiscountPolicy {
    double discount(Order order);
}
```

DiscountPolicy接口有两种实现，其中最普通的实现总是返回相同的折扣：

```java
public class FlatDiscountPolicy implements DiscountPolicy {
    @Override
    public double discount(Order order) {
        return 0.01;
    }
}
```

第二个实现根据订单的总成本返回折扣：

```java
public class AmountBasedDiscountPolicy implements DiscountPolicy {
    @Override
    public double discount(Order order) {
        if (order.totalCost()
                .isGreaterThan(Money.of(CurrencyUnit.USD, 500.00))) {
            return 0.10;
        } else {
            return 0;
        }
    }
}
```

为了本例的需要，我们假设Order类有一个totalCost()方法。

**现在，Java中的单重分派只是以下测试中演示的一种非常著名的多态行为**：

```java
@DisplayName(
        "given two discount policies, " +
                "when use these policies, " +
                "then single dispatch chooses the implementation based on runtime type"
)
@Test
void test() throws Exception {
    // given
    DiscountPolicy flatPolicy = new FlatDiscountPolicy();
    DiscountPolicy amountPolicy = new AmountBasedDiscountPolicy();
    Order orderWorth501Dollars = orderWorthNDollars(501);

    // when
    double flatDiscount = flatPolicy.discount(orderWorth501Dollars);
    double amountDiscount = amountPolicy.discount(orderWorth501Dollars);

    // then
    assertThat(flatDiscount).isEqualTo(0.01);
    assertThat(amountDiscount).isEqualTo(0.1);
}
```

如果这一切看起来相当简单，请继续关注，**我们稍后会使用相同的示例**。

我们现在准备引入双重调度。

### 2.2 双重分派与方法重载

**双重分派根据接收器类型和参数类型确定运行时要调用的方法**。

Java不支持双重调度。

**请注意，双重分派经常与方法重载混淆，它们不是一回事**；方法重载仅根据编译时信息(例如变量的声明类型)来选择要调用的方法。

下面的例子详细解释了这种行为。

让我们介绍一个名为SpecialDiscountPolicy的新折扣接口：

```java
public interface SpecialDiscountPolicy extends DiscountPolicy {
    double discount(SpecialOrder order);
}
```

SpecialOrder只是扩展了Order，没有添加任何新行为。

现在，当我们创建SpecialOrder的实例但将其声明为普通Order时，则不会使用特殊折扣方法：

```java
@DisplayName(
        "given discount policy accepting special orders, " +
                "when apply the policy on special order declared as regular order, " +
                "then regular discount method is used"
)
@Test
void test() throws Exception {
    // given
    SpecialDiscountPolicy specialPolicy = new SpecialDiscountPolicy() {
        @Override
        public double discount(Order order) {
            return 0.01;
        }

        @Override
        public double discount(SpecialOrder order) {
            return 0.10;
        }
    };
    Order specialOrder = new SpecialOrder(anyOrderLines());

    // when
    double discount = specialPolicy.discount(specialOrder);

    // then
    assertThat(discount).isEqualTo(0.01);
}
```

**因此，方法重载不是双重分派**。

即使Java不支持双重分派，我们也可以使用一种模式来实现类似的行为：[访问者](https://en.wikipedia.org/wiki/Visitor_pattern)。

### 2.3 访问者模式

**访问者模式允许我们在不修改现有类的情况下为其添加新行为**，这得益于模拟双重分派的巧妙技术。

我们暂时放下折扣示例，以便介绍访问者模式。

**假设我们想为每种订单使用不同的模板生成HTML视图**，我们可以将此行为直接添加到订单类中，但这并非最佳方案，因为它违反了SRP原则。

相反，我们将使用访问者模式。

首先我们需要引入Visitable接口：

```java
public interface Visitable<V> {
    void accept(V visitor);
}
```

我们还将使用访问者接口，在我们的例子中名为OrderVisitor：

```java
public interface OrderVisitor {
    void visit(Order order);
    void visit(SpecialOrder order);
}
```

**然而，访问者模式的缺点之一是它要求可访问类能够意识到访问者的存在**。

如果类不是为支持访问者而设计的，那么应用此模式可能会很困难(如果没有源代码，甚至不可能)。

每种订单类型都需要实现Visitable接口并提供看似相同的实现，这是另一个缺点。

请注意，添加到Order和SpecialOrder的方法是相同的：

```java
public class Order implements Visitable<OrderVisitor> {
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visit(this);
    }
}

public class SpecialOrder extends Order {
    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visit(this);
    }
}
```

**在子类中不重新实现accept函数可能很诱人，然而，如果不这样做，由于多态性，OrderVisitor.visit(Order)方法总是会被使用**。

最后，我们来看看负责创建HTML视图的OrderVisitor的实现：

```java
public class HtmlOrderViewCreator implements OrderVisitor {

    private String html;

    public String getHtml() {
        return html;
    }

    @Override
    public void visit(Order order) {
        html = String.format("<p>Regular order total cost: %s</p>", order.totalCost());
    }

    @Override
    public void visit(SpecialOrder order) {
        html = String.format("<h1>Special Order</h1><p>total cost: %s</p>", order.totalCost());
    }
}
```

以下示例演示了HtmlOrderViewCreator的用法：

```java
@DisplayName(
        "given collection of regular and special orders, " +
                "when create HTML view using visitor for each order, " +
                "then the dedicated view is created for each order"
)
@Test
void test() throws Exception {
    // given
    List<OrderLine> anyOrderLines = OrderFixtureUtils.anyOrderLines();
    List<Order> orders = Arrays.asList(new Order(anyOrderLines), new SpecialOrder(anyOrderLines));
    HtmlOrderViewCreator htmlOrderViewCreator = new HtmlOrderViewCreator();

    // when
    orders.get(0)
            .accept(htmlOrderViewCreator);
    String regularOrderHtml = htmlOrderViewCreator.getHtml();
    orders.get(1)
            .accept(htmlOrderViewCreator);
    String specialOrderHtml = htmlOrderViewCreator.getHtml();

    // then
    assertThat(regularOrderHtml).containsPattern("<p>Regular order total cost: .*</p>");
    assertThat(specialOrderHtml).containsPattern("<h1>Special Order</h1><p>total cost: .*</p>");
}
```

## 3. DDD中的双重调度

在前面的章节中，我们讨论了双重分派和访问者模式。

**现在我们终于准备好展示如何在DDD中使用这些技术了**。

让我们回到订单和折扣策略的例子。

### 3.1 折扣策略作为一种策略模式

之前，我们介绍了Order类及其totalCost()方法，该方法计算所有订单项的总和：

```java
public class Order {
    public Money totalCost() {
        // ...
    }
}
```

还有一个DiscountPolicy接口用于计算订单的折扣，引入此接口是为了允许使用不同的折扣策略并在运行时进行更改。

这种设计比简单地在订单类中对所有可能的折扣策略进行硬编码要灵活得多：

```java
public interface DiscountPolicy {
    double discount(Order order);
}
```

**到目前为止，我们还没有明确提到这一点，但这个例子使用了[策略模式](https://en.wikipedia.org/wiki/Strategy_pattern)，DDD经常使用此模式来遵循[通用语言](https://martinfowler.com/bliki/UbiquitousLanguage.html)原则并实现低耦合。在DDD领域，策略模式通常被称为Policy**。

让我们看看如何结合双重调度技术和折扣策略。

### 3.2 双重分派及折扣策略

**为了正确使用策略模式，将其作为参数传递通常是一个好主意**。这种方法遵循“[告诉，不要询问](https://martinfowler.com/bliki/TellDontAsk.html)”的原则，支持更好的封装。

例如，Order类可能像这样实现totalCost：

```java
public class Order /* ... */ {
    // ...
    public Money totalCost(SpecialDiscountPolicy discountPolicy) {
        return totalCost().multipliedBy(1 - discountPolicy.discount(this), RoundingMode.HALF_UP);
    }
    // ...
}
```

现在，假设我们想以不同的方式处理每种类型的订单。

例如，在计算特殊订单的折扣时，有一些其他规则需要SpecialOrder类特有的信息。我们希望避免强制类型转换和反射，同时能够计算出每个订单在正确应用折扣后的总成本。

我们已经知道方法重载发生在编译时，**那么，一个自然而然的问题出现了：如何根据订单的运行时类型，动态地将订单折扣逻辑分派到正确的方法中**？

**答案是，我们需要稍微修改一下订单类**。

根Order类需要在运行时将折扣策略参数分派给它，最简单的实现方法是添加一个受保护的applyDiscountPolicy方法：

```java
public class Order /* ... */ {
    // ...
    public Money totalCost(SpecialDiscountPolicy discountPolicy) {
        return totalCost().multipliedBy(1 - applyDiscountPolicy(discountPolicy), RoundingMode.HALF_UP);
    }

    protected double applyDiscountPolicy(SpecialDiscountPolicy discountPolicy) {
        return discountPolicy.discount(this);
    }
    // ...
}
```

由于这种设计，我们避免在Order子类中的totalCost方法中重复业务逻辑。

我们来展示一下使用演示：

```java
@DisplayName(
        "given regular order with items worth $100 total, " +
                "when apply 10% discount policy, " +
                "then cost after discount is $90"
)
@Test
void test() throws Exception {
    // given
    Order order = new Order(OrderFixtureUtils.orderLineItemsWorthNDollars(100));
    SpecialDiscountPolicy discountPolicy = new SpecialDiscountPolicy() {

        @Override
        public double discount(Order order) {
            return 0.10;
        }

        @Override
        public double discount(SpecialOrder order) {
            return 0;
        }
    };

    // when
    Money totalCostAfterDiscount = order.totalCost(discountPolicy);

    // then
    assertThat(totalCostAfterDiscount).isEqualTo(Money.of(CurrencyUnit.USD, 90));
}
```

此示例仍然使用访问者模式，但略作修改。订单类能够感知SpecialDiscountPolicy(即访问者)的含义，并计算折扣。

**如前所述，我们希望能够根据Order的运行时类型应用不同的折扣规则。因此，我们需要在每个子类中重写受保护的applyDiscountPolicy方法**。

让我们在SpecialOrder类中重写此方法：

```java
public class SpecialOrder extends Order {
    // ...
    @Override
    protected double applyDiscountPolicy(SpecialDiscountPolicy discountPolicy) {
        return discountPolicy.discount(this);
    }
   // ...
}
```

我们现在可以在折扣策略中使用有关SpecialOrder的额外信息来计算正确的折扣：

```java
@DisplayName(
        "given special order eligible for extra discount with items worth $100 total, " +
                "when apply 20% discount policy for extra discount orders, " +
                "then cost after discount is $80"
)
@Test
void test() throws Exception {
    // given
    boolean eligibleForExtraDiscount = true;
    Order order = new SpecialOrder(OrderFixtureUtils.orderLineItemsWorthNDollars(100),
            eligibleForExtraDiscount);
    SpecialDiscountPolicy discountPolicy = new SpecialDiscountPolicy() {

        @Override
        public double discount(Order order) {
            return 0;
        }

        @Override
        public double discount(SpecialOrder order) {
            if (order.isEligibleForExtraDiscount())
                return 0.20;
            return 0.10;
        }
    };

    // when
    Money totalCostAfterDiscount = order.totalCost(discountPolicy);

    // then
    assertThat(totalCostAfterDiscount).isEqualTo(Money.of(CurrencyUnit.USD, 80.00));
}
```

此外，由于我们在订单类中使用多态行为，我们可以轻松修改总成本计算方法。

## 4. 总结

在本文中，我们学习了如何在领域驱动设计中使用双重调度技术和策略(又名Policy)模式。