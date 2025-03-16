---
layout: post
title:  Spring框架中的Fallback Bean指南
category: springboot
copyright: springboot
excerpt: Spring Boot 3
---

## 1. 概述

在本教程中，我们将讨论Spring框架中的Fallback [Bean](https://www.baeldung.com/spring-bean)概念。Fallback Bean是在Spring框架版本6.2.0-M1中引入的。当另一个相同类型的Bean不可用或无法初始化时，它们提供替代实现。

这在我们希望妥善处理故障并提供回退机制以确保应用程序继续运行的情况下非常有用。

## 2. 主Bean和后备Bean

在Spring应用程序中，我们可以定义多个相同类型的Bean。默认情况下，Spring使用Bean名称和类型来标识Bean，**当我们有多个相同名称和类型的Bean时，我们可以使用[@Primary注解](https://www.baeldung.com/spring-primary)将其中一个标记为主Bean，以优先于其他Bean**。如果在初始化应用程序上下文时创建了多个相同类型的Bean，并且我们想要指定默认情况下应使用哪个Bean，则这很有用。

类似地，我们可以定义一个后备Bean，当没有其他符合条件的Bean可用时，提供备用实现。**我们可以使用注解@Fallback将Bean标记为后备Bean。只有当没有其他同名Bean可用时，后备Bean才会被注入到应用程序上下文中**。

## 3. 代码示例

让我们看一个例子来演示在Spring应用程序中如何使用主Bean和后备Bean。我们将创建一个使用不同消息传递服务发送消息的小型应用程序，假设我们在生产和非生产环境中拥有多个消息传递服务，需要在它们之间切换以优化性能和成本。

### 3.1 消息传递接口

首先，让我们为服务定义一个接口：

```java
public interface MessagingService {
    void sendMessage(String text);
}
```

该接口有一种方法可以将提供的文本作为消息发送。

### 3.2 主Bean

接下来，让我们定义一个消息服务的实现作为主 bean：

```java
@Service
@Profile("production")
@Primary
public class ProductionMessagingService implements MessagingService {
    @Override
    public void sendMessage(String text) {
       // implementation in production environment
    }
}
```

**在此实现中，我们使用[@Profile注解](https://www.baeldung.com/spring-profiles)来指定此Bean仅在production Profile处于激活状态时可用。我们还使用@Primary注解将其标记为主Bean**。

### 3.3 非主Bean

我们将消息服务的另一种实现定义为非主Bean：

```java
@Service
@Profile("!test")
public class DevelopmentMessagingService implements MessagingService {
    @Override
    public void sendMessage(String text) {
        // implementation in development environment
    }
}

```

**在此实现中，我们使用@Profile注解来指定此Bean在test Profile未激活时可用**，这意味着它将在除test Profile之外的所有Profile中可用。

### 3.4 后备Bean

最后，让我们为消息服务定义一个后备Bean：

```java
@Service
@Fallback
public class FallbackMessagingService implements MessagingService {
    @Override
    public void sendMessage(String text) {
        // fallback implementation
    }
}
```

在这个实现中，我们使用@Fallback注解来标记这个Bean为后备Bean，**只有当没有其他同类型的Bean可用时，才会注入这个Bean**。

## 4. 测试

现在，让我们通过自动装配消息服务并检查根据激活的Profile使用了哪种实现来测试我们的应用程序。

### 4.1 不指定Profile

在第一个测试中，我们没有激活任何Profile。**由于production Profile未激活，因此ProductionMessagingService不可用，而其他两个Bean可用**。 

当我们测试消息服务时，它应该使用DevelopmentMessagingService，因为它优先于后备Bean：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {FallbackMessagingService.class, DevelopmentMessagingService.class, ProductionMessagingService.class})
public class DevelopmentMessagingServiceUnitTest {
    @Autowired
    private MessagingService messagingService;

    @Test
    public void givenNoProfile_whenSendMessage_thenDevelopmentMessagingService() {
        assertEquals(messagingService.getClass(), DevelopmentMessagingService.class);
    }
}
```

### 4.2 Production Profile

接下来，让我们激活production Profile。**现在ProductionMessagingService应该可用，其他两个Bean也可用**。

当我们测试消息服务时，它应该使用ProductionMessagingService，因为它被标记为主Bean：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {FallbackMessagingService.class, DevelopmentMessagingService.class, ProductionMessagingService.class})
@ActiveProfiles("production")
public class ProductionMessagingServiceUnitTest {
    @Autowired
    private MessagingService messagingService;

    @Test
    public void givenProductionProfile_whenSendMessage_thenProductionMessagingService() {
        assertEquals(messagingService.getClass(), ProductionMessagingService.class);
    }
}
```

### 4.3 Test Profile

最后，让我们激活test Profile，**这会从上下文中删除DevelopmentMessagingService Bean。由于我们已删除production Profile，因此ProductionMessagingService也不可用**。

在这种情况下，消息服务应该使用FallbackMessagingService，因为它是唯一可用的Bean：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {FallbackMessagingService.class, DevelopmentMessagingService.class, ProductionMessagingService.class})
@ActiveProfiles("test")
public class FallbackMessagingServiceUnitTest {
    @Autowired
    private MessagingService messagingService;

    @Test
    public void givenTestProfile_whenSendMessage_thenFallbackMessagingService() {
        assertEquals(messagingService.getClass(), FallbackMessagingService.class);
    }
}
```

## 5. 总结

在本教程中，我们讨论了Spring框架中的后备Bean概念。我们了解了如何定义主Bean和后备Bean，以及如何在Spring应用程序中使用它们。当任何其他合格Bean不可用时，后备Bean提供了替代实现。当根据激活的Profile或其他条件在不同的实现之间切换时，这很有用。