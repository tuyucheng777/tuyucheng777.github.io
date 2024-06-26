---
layout: post
title:  空对象模式简介
category: designpattern
copyright: designpattern
excerpt: Design Pattern
---

## 1. 概述

在这个快速教程中，我们介绍空对象模式，它是[策略模式]()的一种特例。我们将描述它的目的以及何时应该真正考虑使用它。

## 2. 空对象模式

在大多数面向对象的编程语言中，我们不允许使用空引用，这就是为什么我们经常被迫编写空检查的原因：

```java
Command cmd = getCommand();
if (cmd != null) {
    cmd.execute();
}
```

有时，如果这样的if语句的数量增加，代码可能会变得丑陋、难以阅读且容易出错，这时候空对象模式就可以派上用场了。

**空对象模式的目的是尽量减少这种空检查**，相反，我们可以识别空行为并将其封装在客户端代码期望的类型中。通常情况下，这种中立的逻辑非常简单-什么都不做，这样我们就不再需要处理空引用的特殊处理了。

我们可以像对待实际上包含一些更复杂的业务逻辑的给定类型的任何其他实例一样对待空对象。因此，客户端代码保持简洁。

由于空对象不应该有任何状态，因此不需要多次创建相同的实例。因此，我们通常会将空对象实现为[单例]()。

## 3. 空对象模式的UML图

![](/assets/images/2023/designpattern/javanullobjectpattern01.png)

如我们所见，我们可以识别以下参与者：

-   客户端需要一个AbstractObject的实例
-   AbstractObject定义了客户端期望的契约-它还可能包含实现类的共享逻辑
-   RealObject实现AbstractObject并提供真实的行为
-   NullObject实现AbstractObject并提供中性行为

## 4. 实现

现在我们对理论有了清晰的认识，让我们看一个例子。

假设我们有一个消息路由器应用程序，每条消息都应该分配一个有效的优先级，我们的系统应该将高优先级消息路由到SMS网关，而具有中等优先级的消息应该路由到JMS队列。

然而，有时，**具有“未定义”或空优先级的消息可能会进入我们的应用程序，此类消息应从进一步处理中丢弃**。

首先，我们创建路由器接口：

```java
public interface Router {
    void route(Message msg);
}
```

接下来，我们创建上述接口的两个实现，一个负责将消息路由到SMS网关，另一个将消息路由到JMS队列：

```java
public class SmsRouter implements Router {
    @Override
    public void route(Message msg) {
        // implementation details
    }
}
```

```java
public class JmsRouter implements Router {
    @Override
    public void route(Message msg) {
        // implementation details
    }
}
```

最后，**实现我们的空对象**：

```java
public class NullRouter implements Router {
    @Override
    public void route(Message msg) {
        // do nothing
    }
}
```

最后让我们看看示例客户端代码的样子：

```java
public class RoutingHandler {
    public void handle(Iterable<Message> messages) {
        for (Message msg : messages) {
            Router router = RouterFactory.getRouterForMessage(msg);
            router.route(msg);
        }
    }
}
```

如我们所见，**无论RouterFactory返回什么实现，我们都以相同的方式对待所有Router对象**，这使我们能够保持代码的干净和可读性。

## 5. 何时使用空对象模式

**当客户端检查null只是为了跳过执行或执行默认操作时，我们应该使用空对象模式**。在这种情况下，我们可以将中性逻辑封装在一个空对象中，并将其返回给客户端而不是null值。这样，客户端的代码就不再需要知道给定实例是否为空。

这种方法遵循一般的面向对象原则，例如[Tell-Don't-Ask](https://martinfowler.com/bliki/TellDontAsk.html)。

为了更好地理解什么时候应该使用空对象模式，让我们想象一下我们必须实现如下定义的CustomerDao接口：

```java
public interface CustomerDao {
    Collection<Customer> findByNameAndLastname(String name, String lastname);

    Customer getById(Long id);
}
```

**如果没有Customer匹配提供的搜索条件，大多数开发人员会从findByNameAndLastname()返回Collections.emptyList()，这是遵循空对象模式的一个很好的例子**。

相反，getById()应该返回具有给定ID的Customer，调用此方法的人希望获得特定的Customer实体，**如果不存在这样的Customer，我们应该明确返回null以表示提供的ID有问题**。 

与所有其他模式一样，**在盲目实现空对象模式之前，我们需要考虑我们的特定用例**。否则，我们可能会无意中在我们的代码中引入一些很难发现的错误。

## 6. 总结

在本文中，我们了解了空对象模式是什么以及何时可以使用它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/design-patterns-modules)上获得。