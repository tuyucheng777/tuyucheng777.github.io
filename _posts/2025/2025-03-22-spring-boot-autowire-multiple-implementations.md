---
layout: post
title:  自动注入具有多个实现的接口
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

**在本文中，我们将探讨在Spring Boot中自动装配具有多个实现的接口、实现方法以及一些用例**。这是一个强大的功能，允许开发人员将接口的不同实现动态地注入到应用程序中。

## 2. 默认行为

通常，当我们有多个接口实现并尝试将该接口自动装配到组件中时，**我们会收到错误-“required a single bean, but X were found”**。原因很简单：Spring不知道我们想在该组件中看到哪个实现。幸运的是，Spring提供了多种工具来更具体。

## 3. 引入限定符

**使用@Qualifier注解，我们可以指定要在多个候选者中自动装配哪个Bean**。我们可以将其应用于组件本身，为其赋予自定义限定符名称：

```java
@Service
@Qualifier("goodServiceA-custom-name")
public class GoodServiceA implements GoodService {
    // implementation
}
```

之后，我们用@Qualifier注解参数来指定我们想要的实现：

```java
@Autowired
public SimpleQualifierController(
        @Qualifier("goodServiceA-custom-name") GoodService niceServiceA,
        @Qualifier("goodServiceB") GoodService niceServiceB,
        GoodService goodServiceC
) {
    this.goodServiceA = niceServiceA;
    this.goodServiceB = niceServiceB;
    this.goodServiceC = goodServiceC;
}
```

在上面的例子中，我们可以看到我们使用自定义限定符来自动装配GoodServiceA。同时，对于GoodServiceB，我们没有自定义限定符：

```java
@Service
public class GoodServiceB implements GoodService {
    // implementation
}
```

在本例中，我们通过类名自动装配组件。此类自动装配的限定符应采用驼峰式命名法，例如，如果类名为“MyAwesomeClass”，则“myAwesomeClass”是有效的限定符。

上述代码中的第三个参数更加有趣，我们甚至不需要用@Qualifier标注它，因为**Spring默认会尝试通过参数名称自动装配组件**，如果GoodServiceC存在，我们就会避免错误：

```java
@Service
public class GoodServiceC implements GoodService {
    // implementation 
}
```

## 4. 主要组件

此外，我们可以使用@Primary标注其中一个实现。**如果有多个候选，并且通过参数名称或限定符进行自动装配不适用，则Spring将使用此实现**：

```java
@Primary
@Service
public class GoodServiceC implements GoodService {
    // implementation
}
```

当我们频繁使用其中一种实现时它很有用，并有助于避免“required a single bean”的错误。

## 5. Profile

**可以使用Spring Profile来决定自动装配哪个组件**。例如，我们可能有一个FileStorage接口，它有两个实现-S3FileStorage和AzureFileStorage。我们可以使S3FileStorage仅在prod Profile上处于激活状态，而AzureFileStorage仅在dev Profile上处于激活状态。

```java
@Service
@Profile("dev")
public class AzureFileStorage implements FileStorage {
    // implementation
}

@Service
@Profile("prod")
public class S3FileStorage implements FileStorage {
    // implementation
}
```

## 6. 将实现自动装配到集合中

**Spring允许我们将特定类型的所有可用Bean注入到集合中**，以下是我们将GoodService的所有实现自动装配到List中的方法：

```java
@Autowired
public SimpleCollectionController(List<GoodService> goodServices) {
    this.goodServices = goodServices;
}
```

此外，我们可以将实现自动装配到集合、Map或数组中。使用Map时，格式通常为Map<String, GoodService\>，其中键是Bean的名称，值是Bean实例本身：

```java
@Autowired
public SimpleCollectionController(Map<String, GoodService> goodServiceMap) {
    this.goodServiceMap = goodServiceMap;
}

public void printAllHellos() {
    String messageA = goodServiceMap.get("goodServiceA").getHelloMessage();
    String messageB = goodServiceMap.get("goodServiceB").getHelloMessage();

    // print messages
}
```

**重要提示：Spring会将所有候选Bean自动装配到集合中，无论限定符或参数名称如何，只要它们处于激活状态**。它会忽略使用@Profile标注的Bean，这些Bean与当前Profile不匹配。同样，Spring仅在满足条件时才包含使用@Conditional标注的Bean(下一节将详细介绍)。

## 7. 高级控制

Spring允许我们对选择哪些候选对象进行自动装配进行额外的控制。

**为了更精确地说明哪些Bean可以成为自动装配的候选者，我们可以使用@Conditional对其进行标注**。它应该有一个参数，该参数带有一个实现Condition的类(它是一个函数接口)。例如，以下是检查操作系统是否为Windows的条件：

```java
public class OnWindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("os.name").toLowerCase().contains("windows");
    }
}
```

以下是我们用@Conditional标注组件的方法：

```java
@Component
@Conditional(OnWindowsCondition.class)
public class WindowsFileService implements FileService {
    @Override
    public void readFile() {
        // implementation
    }
}
```

在这个例子中，仅当OnWindowsCondition中的matches()返回true时，WindowsFileService才会成为自动装配的候选。

我们应该谨慎使用非集合自动装配的@Conditional注解，因为多个符合条件的Bean将导致错误。

此外，如果找不到候选者，我们将收到错误。因此，在将@Conditional与自动装配集成时，设置可选注入是有意义的，这确保应用程序在找不到合适的Bean时仍可以继续运行而不会抛出错误。有两种方法可以实现这一点：

```java
@Autowired(required = false)
private GoodService goodService; // not very safe, we should check this for null

@Autowired
private Optional<GoodService> goodService; // safer way
```

**当我们自动装配到集合中时，我们可以使用@Order注解指定组件的顺序**：

```java
@Order(2) 
public class GoodServiceA implements GoodService { 
    // implementation
 } 

@Order(1) 
public class GoodServiceB implements GoodService {
    // implementation 
}
```

如果我们尝试自动装配List<GoodService\>，GoodServiceB将被放置在GoodServiceA之前。重要提示：当我们自动装配到Set中时，@Order不起作用。

## 8. 总结

在本文中，我们讨论了Spring提供的用于在自动装配期间管理接口的多种实现的工具，这些工具和技术在设计Spring Boot应用程序时可以实现更动态的方法。但是，与每种工具一样，我们应该确保它们的必要性，因为使用不当可能会引入错误并使长期支持变得复杂。