---
layout: post
title:  CDI 2.0中的事件通知模型简介
category: libraries
copyright: libraries
excerpt: CDI
---

## 1. 概述

[CDI](https://www.baeldung.com/java-ee-cdi)(上下文和依赖注入)是Jakarta EE平台的标准依赖注入框架。

在本教程中，我们将介绍[CDI 2.0](http://www.cdi-spec.org/news/2017/05/15/CDI_2_is_released/)以及它如何通过添加改进的、功能齐全的事件通知模型来构建强大的CDI 1.x类型安全注入机制。

## 2. Maven依赖

首先，我们将构建一个简单的Maven项目。

**我们需要一个符合CDI 2.0的容器，而CDI的参考实现[Weld](http://weld.cdi-spec.org/)非常适合**：

```xml
<dependencies>
    <dependency>
        <groupId>javax.enterprise</groupId>
        <artifactId>cdi-api</artifactId>
        <version>2.0.SP1</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.weld.se</groupId>
        <artifactId>weld-se-core</artifactId>
        <version>3.1.6.Final</version>
    </dependency>
</dependencies>
```

像往常一样，我们可以从Maven Central中提取最新版本的[cdi-api](https://mvnrepository.com/artifact/javax.enterprise/cdi-api)和[weld-se-core](https://mvnrepository.com/artifact/org.jboss.weld.se/weld-se-core)。

## 3. 观察和处理自定义事件

简而言之，**CDI 2.0事件通知模型是[观察者模式](https://www.baeldung.com/java-observer-pattern)的经典实现，基于[@Observes](https://docs.jboss.org/cdi/api/1.0/javax/enterprise/event/Observes.html)方法参数注解**。因此，它允许我们轻松定义观察者方法，这些方法可以在响应一个或多个事件时自动调用。

例如，我们可以定义一个或多个Bean，它们将触发一个或多个特定事件，而其他Bean将收到有关事件的通知并做出相应的反应。

为了更清楚地演示其工作原理，我们将构建一个简单的示例，包括一个基本服务类、一个自定义事件类和一个对我们的自定义事件做出反应的观察者方法。

### 3.1 基本服务类

让我们首先创建一个简单的TextService类：

```java
public class TextService {

    public String parseText(String text) {
        return text.toUpperCase();
    }
}
```

### 3.2 自定义事件类

接下来，让我们定义一个示例事件类，该类在其构造函数中接收String参数：

```java
public class ExampleEvent {
    
    private final String eventMessage;

    public ExampleEvent(String eventMessage) {
        this.eventMessage = eventMessage;
    }
    
    // getter
}
```

### 3.3 使用@Observes注解定义观察者方法

现在已经定义了我们的服务和事件类，让我们使用@Observes注解为我们的ExampleEvent类创建一个观察者方法：

```java
public class ExampleEventObserver {
    
    public String onEvent(@Observes ExampleEvent event, TextService textService) {
        return textService.parseText(event.getEventMessage());
    } 
}
```

虽然乍一看，onEvent()方法的实现相当简单，但它实际上通过@Observes注解封装了很多功能。

我们可以看到，onEvent()方法是一个事件处理程序，它以ExampleEvent和TextService对象作为参数。

**请记住，@Observes注解后指定的所有参数都是标准注入点**。因此，CDI将为我们创建完全初始化的实例，并将其注入到观察者方法中。

### 3.4 初始化CDI 2.0容器

至此，我们已经创建了服务和事件类，并定义了一个简单的观察者方法来响应事件，但是，如何指示CDI在运行时注入这些实例呢？

事件通知模型的功能就在这里得到充分展示，我们只需初始化新的[SeContainer](https://docs.jboss.org/cdi/api/2.0/javax/enterprise/inject/se/SeContainer.html)实现，并通过fireEvent()方法触发一个或多个事件：

```java
SeContainerInitializer containerInitializer = SeContainerInitializer.newInstance(); 
try (SeContainer container = containerInitializer.initialize()) {
    container.getBeanManager().fireEvent(new ExampleEvent("Welcome to Tuyucheng!")); 
}
```

**请注意，我们使用SeContainerInitializer和SeContainer对象是因为我们在Java SE环境中使用CDI，而不是在Jakarta EE中**。

当通过传播事件本身触发ExampleEvent时，所有附加的观察者方法都将得到通知。

由于在@Observes注解之后作为参数传递的所有对象都将被完全初始化，因此CDI将在将其注入onEvent()方法之前负责为我们连接整个TextService对象图。

简而言之，**我们拥有类型安全的IoC容器以及功能丰富的事件通知模型的优势**。

## 4. ContainerInitialized事件

在前面的例子中，我们使用自定义事件将事件传递给观察者方法并获取完全初始化的TextService对象。

当然，当我们确实需要在应用程序的多个点之间传播一个或多个事件时，这很有用。

**有时，我们只需要获取一堆完全初始化的对象，这些对象可以在我们的应用程序类中使用**，而不必经历额外的事件的实现。

为此，**CDI 2.0提供了[ContainerInitialized](https://javadoc.io/doc/org.jboss.weld.se/weld-se/2.2.6.Final/org/jboss/weld/environment/se/events/ContainerInitialized.html)事件类，该事件在Weld容器初始化时自动触发**。

让我们看一下如何使用ContainerInitialized事件将控制权转移到ExampleEventObserver类：

```java
public class ExampleEventObserver {
    public String onEvent(@Observes ContainerInitialized event, TextService textService) {
        return textService.parseText(event.getEventMessage());
    }    
}
```

请记住，**ContainerInitialized事件类是Weld特有的**。因此，如果我们使用不同的CDI实现，则需要重构观察者方法。

## 5. 条件观察者方法

在当前的实现中，我们的ExampleEventObserver类默认定义了一个无条件的观察者方法，**这意味着无论当前上下文中是否存在该类的实例，观察者方法都会始终收到所提供事件的通知**。

同样，我们可以通过将notifyObserver = IF_EXISTS指定为@Observes注解的参数来定义条件观察者方法：

```java
public String onEvent(@Observes(notifyObserver=IF_EXISTS) ExampleEvent event, TextService textService) { 
    return textService.parseText(event.getEventMessage());
}
```

当我们使用条件观察者方法时，**仅当当前上下文中存在定义观察者方法的类的实例时，该方法才会收到匹配事件的通知**。

## 6. 事务观察者方法

**我们还可以在事务中触发事件，例如数据库更新或删除操作**，为此，我们可以通过在@Observes注解中添加during参数来定义事务观察器方法。

during参数的每个可能值对应于事务的特定阶段：

- BEFORE_COMPLETION
- AFTER_COMPLETION
- AFTER_SUCCESS
- AFTER_FAILURE

如果我们在事务中触发ExampleEvent事件，则需要相应地重构onEvent()方法以在所需阶段处理该事件：

```java
public String onEvent(@Observes(during=AFTER_COMPLETION) ExampleEvent event, TextService textService) { 
    return textService.parseText(event.getEventMessage());
}
```

**事务观察器方法仅在给定事务的匹配阶段才会收到所提供的事件的通知**。

## 7. 观察者方法顺序

CDI 2.0事件通知模型的另一个很好的改进是能够设置调用给定事件的观察者的顺序或优先级。

我们可以通过在@Observes之后指定[@Priority](https://docs.oracle.com/javaee/7/api/javax/annotation/Priority.html)注解来轻松定义调用观察者方法的顺序。

为了理解此功能的工作原理，除了ExampleEventObserver实现的方法之外，让我们定义另一个观察者方法：

```java
public class AnotherExampleEventObserver {
    
    public String onEvent(@Observes ExampleEvent event) {
        return event.getEventMessage();
    }   
}
```

**在这种情况下，两个观察者方法默认具有相同的优先级，因此，CDI调用它们的顺序根本无法预测**。

我们可以通过@Priority注解为每个方法分配一个调用优先级来轻松解决这个问题：

```java
public String onEvent(@Observes @Priority(1) ExampleEvent event, TextService textService) {
    // ... implementation
}
```

```java
public String onEvent(@Observes @Priority(2) ExampleEvent event) {
    // ... implementation
}
```

**优先级遵循自然顺序**，因此，CDI将首先调用优先级为1的观察者方法，然后调用优先级为2的方法。

同样，**如果我们在两个或多个方法中使用相同的优先级，则顺序再次未定义**。

## 8. 异步事件

到目前为止，我们学习的所有示例都是同步触发事件的。然而，CDI 2.0也允许我们轻松触发异步事件，异步观察者方法可以在不同的线程中处理这些异步事件。

我们可以使用fireAsync()方法异步触发事件：

```java
public class ExampleEventSource {
    
    @Inject
    Event<ExampleEvent> exampleEvent;
    
    public void fireEvent() {
        exampleEvent.fireAsync(new ExampleEvent("Welcome to Tuyucheng!"));
    }   
}
```

**Bean触发事件，这些事件是[Event](https://docs.jboss.org/cdi/api/2.0/javax/enterprise/event/Event.html)接口的实现。因此，我们可以像注入任何其他常规Bean一样注入它们**。

为了处理异步事件，我们需要使用[@ObservesAsync](https://docs.jboss.org/cdi/api/2.0.EDR2/javax/enterprise/event/ObservesAsync.html)注解定义一个或多个异步观察者方法：

```java
public class AsynchronousExampleEventObserver {

    public void onEvent(@ObservesAsync ExampleEvent event) {
        // ... implementation
    }
}
```

## 9. 总结

在本文中，我们学习了如何开始使用CDI 2.0捆绑的改进的事件通知模型。