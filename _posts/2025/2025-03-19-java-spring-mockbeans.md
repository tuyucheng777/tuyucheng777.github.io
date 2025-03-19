---
layout: post
title:  Spring @MockBeans指南
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

在本快速教程中，我们将探讨Spring Boot [@MockBeans](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/mock/mockito/MockBeans.html)注解的用法。

## 2. 示例设置

在深入研究之前，让我们创建一个将在本教程中用到的简单票务验证器示例：

```java
public class TicketValidator {
    private CustomerRepository customerRepository;

    private TicketRepository ticketRepository;

    public boolean validate(Long customerId, String code) {
        customerRepository.findById(customerId)
                .orElseThrow(() -> new RuntimeException("Customer not found"));

        ticketRepository.findByCode(code)
                .orElseThrow(() -> new RuntimeException("Ticket with given code not found"));
        return true;
    }
}
```

在这里，我们定义了validate()方法来检查数据库中是否存在给定的数据。它使用CustomerRepository和TicketRepository作为依赖。

现在，让我们研究如何使用Spring的@MockBean和@MockBeans注解创建测试和Mock依赖项。

## 3. @MockBean注解

**[Spring框架](https://www.baeldung.com/spring-tutorial)提供了[@MockBean](https://www.baeldung.com/java-spring-mockito-mock-mockbean#spring-boots-mockbean-annotation)注解来Mock依赖项以进行测试，此注解允许我们定义特定Bean的Mock版本**。新创建的Mock将添加到Spring [ApplicationContext](https://www.baeldung.com/spring-application-context)中。因此，如果已存在相同类型的Bean，它将被Mock版本替换。

此外，我们可以在我们想要Mock的字段或测试类级别上使用此注解。

使用@MockBean注解，我们可以通过Mock其依赖对象的行为来隔离我们想要测试的代码的特定部分。

现在，让我们看看@MockBean的实际作用，让我们用Mock实现替换现有的CustomerRepository Bean：

```java
class MockBeanTicketValidatorUnitTest {
    @MockBean
    private CustomerRepository customerRepository;

    @Autowired
    private TicketRepository ticketRepository;

    @Autowired
    private TicketValidator ticketValidator;

    @Test
    void givenUnknownCustomer_whenValidate_thenThrowException() {
        String code = UUID.randomUUID().toString();
        when(customerRepository.findById(any())).thenReturn(Optional.empty());

        assertThrows(RuntimeException.class, () -> ticketValidator.validate(1L, code));
    }
}
```

这里，我们用@MockBean注解标注了CustomerRepository字段，Spring将Mock注入到该字段并将其添加到应用程序上下文中。

**需要记住的一点是，我们不能使用@MockBean注解来Mock应用程序上下文刷新期间Bean的行为**。

此外，此注解被定义为[@Repeatable](https://www.baeldung.com/java-default-annotations#5-repeatable)，这允许我们在类级别多次定义相同的注解：

```java
@MockBean(CustomerRepository.class)
@MockBean(TicketRepository.class)
@SpringBootTest(classes = Application.class)
class MockBeanTicketValidatorUnitTest {
    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private TicketRepository ticketRepository;

    @Autowired
    private TicketValidator ticketValidator;

    // ...
}
```

## 4. @MockBeans注解

讨论完了@MockBean注解，下面我们来讨论一下[@MockBeans](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/mock/mockito/MockBeans.html)注解，简单来说，**这个注解代表了多个@MockBean注解的集合，作为这些注解的容器**。

此外，它还帮助我们组织测试用例。我们可以在同一个地方定义多个Mock，使测试类更简洁、更有条理。此外，在多个测试类中重复使用Mock Bean时，它非常有用。

我们可以使用@MockBeans来替代我们之前看到的可重复的@MockBean解决方案：

```java
@MockBeans({@MockBean(CustomerRepository.class), @MockBean(TicketRepository.class)})
@SpringBootTest(classes = Application.class)
class MockBeansTicketValidatorUnitTest {
    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private TicketRepository ticketRepository;

    @Autowired
    private TicketValidator ticketValidator;

    // ...
}
```

**值得注意的是，我们对想要Mock的 Bean使用了[@Autowired](https://www.baeldung.com/spring-autowire)注解**。

此外，这种方法与在每个字段上定义@MockBean在功能上没有区别。但是，如果我们使用Java或更高版本，@MockBeans注解可能看起来是多余的，因为Java支持可重复的注解。

**@MockBeans注解背后的主要思想是允许开发人员在一个地方指定Mock Bean**。

## 5. 总结

在这篇短文中，我们学习了如何在定义用于测试的Mock时使用@MockBeans注解。

总而言之，我们可以使用@MockBeans注解来分组多个@MockBean注解并在一个地方定义所有Mock。