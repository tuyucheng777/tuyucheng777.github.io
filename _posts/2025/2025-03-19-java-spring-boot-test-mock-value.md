---
layout: post
title:  在Spring Boot测试中Mock @Value
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

在Spring Boot中编写单元测试时，经常会遇到必须Mock使用[@Value](https://www.baeldung.com/spring-value-annotation)[注解](https://www.baeldung.com/spring-boot-annotations)加载的外部配置或属性的情况，**这些属性通常从application.properties或application.yml文件加载并注入到我们的Spring组件中**。但是，我们通常不想使用外部文件加载完整的[Spring上下文](https://www.baeldung.com/spring-application-context)。相反，我们希望Mock这些值以保持测试快速且独立。

在本教程中，我们将了解为什么以及如何在Spring Boot测试中Mock @Value，以确保在不加载整个应用程序上下文的情况下顺利有效地进行测试。

## 2. 如何在Spring Boot测试中Mock @Value

假设我们有一个服务类ValueAnnotationMock，它使用@Value从application.properties文件中获取外部API URL的值：

```java
@Service
public class ValueAnnotationMock {
    @Value("${external.api.url}")
    private String apiUrl;

    public String getApiUrl() {
        return apiUrl;
    }

    public String callExternalApi() {
        return String.format("Calling API at %s", apiUrl);
    }
}
```

另外，这是我们的application.properties文件：

```java
external.api.url=http://dynamic-url.com
```

那么，我们如何在测试类中Mock这个属性？

有多种方法可以Mock @Value注解，让我们逐一看看。

### 2.1 使用@TestPropertySource注解

在Spring Boot测试中Mock @Value属性的最简单方法是使用[@TestPropertySource](https://www.baeldung.com/spring-test-property-source)注解，**这允许我们直接在测试类中定义属性**。

让我们通过一个例子来更好地理解：

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = ValueAnnotationMock.class)
@SpringBootTest
@TestPropertySource(properties = {
        "external.api.url=http://mocked-url.com"
})
public class ValueAnnotationMockTestPropertySourceUnitTest {
    @Autowired
    private ValueAnnotationMock valueAnnotationMock;

    @Test
    public void givenValue_whenUsingTestPropertySource_thenMockValueAnnotation() {
        String apiUrl = valueAnnotationMock.getApiUrl();
        assertEquals("http://mocked-url.com", apiUrl);
    }
}
```

在这个例子中，@TestPropertySource为external.api.url属性提供了一个Mock值，即http://mocked-url.com，然后将其注入到ValueAnnotationMock Bean中。

让我们探讨一下使用@TestPropertySource Mock @Value的一些主要优点：

- 这种方法使我们能够使用Spring的属性注入机制Mock应用程序的真实行为，这意味着被测试的代码接近生产中运行的代码。
- 我们可以在测试类中直接指定特定于测试的属性值，从而简化复杂属性的Mock。

现在，让我们转而研究一下这种方法的一些缺点：

- 使用@TestPropertySource的测试会加载Spring上下文，这可能比不需要上下文加载的单元测试慢。
- 这种方法会给简单的单元测试增加不必要的复杂性，因为它引入了许多Spring的机制，使得测试更不独立、更慢。

### 2.2 使用ReflectionTestUtils

对于我们想要直接将Mock值注入私有字段(例如用@Value标注的字段)的情况，我们可以使用Spring的[ReflectionTestUtils](https://www.baeldung.com/spring-reflection-test-utils)类来手动设置该值。

让我们看看实现：

```java
public class ValueAnnotationMockReflectionUtilsUnitTest {
    @Autowired
    private ValueAnnotationMock valueAnnotationMock;

    @Test
    public void givenValue_whenUsingReflectionUtils_thenMockValueAnnotation() {
        valueAnnotationMock = new ValueAnnotationMock();
        ReflectionTestUtils.setField(valueAnnotationMock, "apiUrl", "http://mocked-url.com");
        String apiUrl = valueAnnotationMock.getApiUrl();
        assertEquals("http://mocked-url.com", apiUrl);
    }
}
```

**这种方法绕过了Spring的整个上下文，使其成为纯单元测试的理想选择，因为我们根本不想涉及Spring Boot的依赖注入机制**。

让我们看看这种方法的一些好处：

- 我们可以直接操作私有字段，即使是那些用@Value标注的字段，而无需修改原始类，这对于无法轻松重构的遗留代码很有帮助。
- 它避免加载Spring应用程序上下文，从而使测试更快、更独立。
- 我们可以通过在测试期间动态改变对象的内部状态来测试确切的行为。

使用ReflectionUtils有一些缺点：

- 通过反射直接访问或修改私有字段会破坏封装，这违背了面向对象设计的最佳实践。
- 虽然它不会加载Spring的上下文，但由于运行时检查和修改类结构的开销，反射比标准访问慢。

### 2.3 使用构造函数注入

这种方法使用[构造函数注入](https://www.baeldung.com/constructor-injection-in-spring)来处理@Value属性，为Spring上下文注入和单元测试提供了一个干净的解决方案，而不需要反射或Spring的完整环境。

我们有一个主类，其中使用[构造函数](https://www.baeldung.com/java-constructors)中的@Value注解注入了apiUrl和apiPassword等属性：

```java
public class ValueAnnotationConstructorMock {
    private final String apiUrl;
    private final String apiPassword;

    public ValueAnnotationConstructorMock(@Value("#{myProps['api.url']}") String apiUrl,
                                          @Value("#{myProps['api.password']}") String apiPassword) {
        this.apiUrl = apiUrl;
        this.apiPassword = apiPassword;
    }

    public String getApiUrl() {
        return apiUrl;
    }

    public String getApiPassword() {
        return apiPassword;
    }
}
```

**这些值不是通过字段注入(可能需要在测试中进行反射或其他设置)来注入的，而是直接通过构造函数传递**。这样，该类就可以轻松地使用特定值进行实例化，以进行测试。

在测试类中，我们不需要Spring的上下文来注入这些值。相反，我们可以简单地用任意值实例化该类：

```java
public class ValueAnnotationMockConstructorUnitTest {
    private ValueAnnotationConstructorMock valueAnnotationConstructorMock;

    @BeforeEach
    public void setUp() {
        valueAnnotationConstructorMock = new ValueAnnotationConstructorMock("testUrl", "testPassword");
    }

    @Test
    public void testDefaultUrl() {
        assertEquals("testUrl", valueAnnotationConstructorMock.getApiUrl());
    }

    @Test
    public void testDefaultPassword() {
        assertEquals("testPassword", valueAnnotationConstructorMock.getApiPassword());
    }
}
```

这使得测试更加简单，因为我们不依赖于Spring的初始化过程。我们可以直接传递所需的任何值(例如anyUrl和anyPassword)来Mock @Value属性。

现在，继续讨论使用构造函数注入的一些主要优点：

- 这种方法非常适合单元测试，因为它允许我们绕过对Spring上下文的需求。我们可以在测试中直接注入Mock值，从而使测试更快、更独立。
- 它简化了类的构造并确保在构造时注入所有必需的依赖项，使得类更易于推理。
- 使用带有构造函数注入的final字段可确保不变性，从而使代码更安全且更易于调试。

接下来，我们回顾一下使用构造函数注入的一些缺点：

- 我们可能需要修改现有的代码以采用构造函数注入，但这可能并不总是可行的，特别是对于遗留应用程序而言。
- 我们需要明确管理测试中的所有依赖项，如果类具有许多依赖，这可能会导致冗长的测试设置。
- 如果我们的测试需要处理动态变化的值或大量属性，构造函数注入就会变得麻烦。

## 3. 总结

在本文中，我们看到在Spring Boot测试中Mock @Value是一种常见要求，可以保持单元测试的专注、快速且独立于外部配置文件。通过利用@TestPropertySource和ReflectionTestUtils等工具，我们可以有效地Mock配置值并维护干净、可靠的测试。

通过这些策略，我们可以自信地编写单元测试，隔离组件的逻辑而不依赖外部资源，从而确保更好的测试性能和可靠性。