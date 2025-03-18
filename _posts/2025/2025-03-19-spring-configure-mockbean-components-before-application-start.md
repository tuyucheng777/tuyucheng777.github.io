---
layout: post
title:  在应用程序启动之前配置@MockBean组件
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 简介

[@MockBean](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/mock/mockito/MockBean.html)是Spring框架提供的注解，**它有助于创建[Spring Bean](https://www.baeldung.com/spring-bean)的[Mock对象](https://www.baeldung.com/java-spring-mockito-mock-mockbean?)，使我们能够在测试期间用Mock对象替换实际Bean**。这在[集成测试](https://www.baeldung.com/spring-boot-testing)期间特别有用，因为我们希望隔离某些组件而不依赖于它们的实际实现。

在本教程中，我们将研究配置@MockBean组件来测试Spring Boot应用程序的各种方法。

## 2. 早期配置的必要性

当我们需要在测试期间控制应用程序中某些Bean的行为时，在应用程序启动之前配置@MockBean组件至关重要，尤其是当这些Bean与数据库或Web服务等外部系统交互时。

早期配置的好处：

- **隔离测试**：通过Mock被测单元的依赖关系，帮助隔离被测单元的行为
- **避免外部调用**：它可以防止对外部系统(如数据库或外部API)的调用，否则原始Bean会完成这些调用
- **控制Bean行为**：我们可以预定义Mock Bean的响应和行为，从而确保测试是可预测的，并且不依赖于外部因素

## 3. 早期配置技术

我们将首先研究如何配置@MockBean组件，然后探索在应用程序启动之前配置这些组件的各种方法。

### 3.1 在测试类中直接声明

这是Mock Bean最简单的方法，我们可以直接在测试类中的字段上使用@MockBean注解，Spring Boot会自动将上下文中的实际Bean替换为Mock Bean：

```java
@SpringBootTest(classes = ConfigureMockBeanApplication.class)
public class DirectMockBeanConfigUnitTest {
    @MockBean
    private UserService mockUserService;

    @Autowired
    private UserController userController;

    @Test
    void whenDirectMockBean_thenReturnUserName(){
        when(mockUserService.getUserName(1L)).thenReturn("John Doe");
        assertEquals("John Doe", userController.getUserName(1L));
        verify(mockUserService).getUserName(1L);
    }
}
```

**在这个方法中，Mock无缝地替换了[Spring上下文](https://www.baeldung.com/spring-application-context)中的真实Bean，允许依赖于它的其他Bean使用Mock**。我们可以独立定义和控制Mock，而不会影响其他测试。

### 3.2 使用@BeforeEach配置@MockBean

我们可以在@BeforeEach方法中使用Mockito配置Mock Bean，确保它们在测试开始之前可用。

当我们想要在更高级别Mock某些Bean(例如Repository、Service或Controller)时，它很有用-这些Bean不能直接测试或具有复杂的依赖关系。

在集成测试中，组件通常相互依赖，带有@BeforeEach的@MockBean可通过在每次测试开始时隔离某些组件并模拟它们的依赖关系来帮助创建受控环境，使我们能够专注于正在测试的功能：

```java
@SpringBootTest(classes = ConfigureMockBeanApplication.class)
public class ConfigureBeforeEachTestUnitTest {

    @MockBean
    private UserService mockUserService;

    @Autowired
    private UserController userController;

    @BeforeEach
    void setUp() {
        when(mockUserService.getUserName(1L)).thenReturn("John Doe");
    }

    @Test
    void whenParentContextConfigMockBean_thenReturnUserName(){
        assertEquals("John Doe", userController.getUserName(1L));
        verify(mockUserService).getUserName(1L);
    }
}
```

这种方法可确保我们的Mock配置在每次测试之前重置，从而有助于隔离测试。

### 3.3 在嵌套测试配置类中使用@Mockbean

**为了使测试类更简洁，并重用Mock配置，我们可以将Mock设置移至单独的嵌套[配置类](https://docs.spring.io/spring-framework/reference/core/beans/java/configuration-annotation.html)**。我们需要使用@TestConfiguration标注配置类，在这个类中，我们实例化并配置Mock：

```java
@SpringBootTest(classes = ConfigureMockBeanApplication.class)
@Import(InternalConfigMockBeanUnitTest.TestConfig.class)
public class InternalConfigMockBeanUnitTest {
    @TestConfiguration
    static class TestConfig {
        @MockBean
        UserService userService;

        @PostConstruct
        public void initMock(){
            when(userService.getUserName(3L)).thenReturn("Bob Johnson");
        }
    }
    
    @Autowired
    private UserService userService;

    @Autowired
    private UserController userController;

    @Test
    void whenConfiguredUserService_thenReturnUserName(){
        assertEquals("Bob Johnson", userController.getUserName(3L));
        verify(userService).getUserName(3L);
    }
}
```

这种方法有助于测试类专注于测试，并将配置分开，从而更容易管理复杂的配置。

### 3.4 在外部测试配置类中使用@Mockbean

**当我们需要在多个测试类中重用测试配置时，我们可以将测试配置外部化为单独的类**。与以前的方法类似，我们需要使用@TestConfiguration标注配置类。它允许我们创建一个单独的特定于测试的配置类，可用于在Spring上下文中Mock或替换Bean：

```java
@TestConfiguration
class TestConfig {

    @MockBean
    UserService userService;

    @PostConstruct
    public void initMock(){
        when(userService.getUserName(2L)).thenReturn("Jane Smith");
    }
}
```

我们可以使用@Import(TestConfig.class)在我们的测试类中导入这个测试配置：

```java
@SpringBootTest(classes = ConfigureMockBeanApplication.class)
@Import(TestConfig.class)
class ConfigureMockBeanApplicationUnitTest {

    @Autowired
    private UserService mockUserService;

    @Autowired
    private UserController userController;

    @Test
    void whenConfiguredUserService_thenReturnUserName(){
        assertEquals("Jane Smith", userController.getUserName(2L));
        verify(mockUserService).getUserName(2L);
    }
}
```

**当我们需要配置多个测试组件或希望拥有可以在不同测试用例之间共享的可重复使用的Mock设置时，这种方法很有用**。

### 3.5 特定Profile的配置

当我们需要[针对不同的Profile(例如不同的环境(开发或测试))进行测试](https://www.baeldung.com/spring-boot-junit-5-testing-active-profile)时，我们可以创建特定于Profile的配置。通过将@ActiveProfiles应用于我们的测试类，我们可以加载不同的应用程序配置以满足我们的测试需求。

让我们为我们的Dev Profile创建一个测试配置：

```java
@Configuration
@Profile("Dev")
class DevProfileTestConfig {

    @MockBean
    UserService userService;

    @PostConstruct
    public void initMock(){
        when(userService.getUserName(4L)).thenReturn("Alice Brown");
    }
}
```

然后，我们可以在测试类中将激活的Profile设置为“Dev”：

```java
@SpringBootTest(classes = ConfigureMockBeanApplication.class)
@ActiveProfiles("Dev")
public class ProfileBasedMockBeanConfigUnitTest {

    @Autowired
    private UserService userService;

    @Autowired
    private UserController userController;

    @Test
    void whenDevProfileActive_thenReturnUserName(){
        assertEquals("Alice Brown", userController.getUserName(4L));
        verify(userService).getUserName(4L);
    }
}
```

**这种方法在开发、测试或生产等不同环境进行测试时很有用，可确保我们Mock特定Profile所需的精确条件**。

### 3.6 使用Mockito的Answer进行动态Mock

**当我们想要更好地控制Mock行为时，例如在运行时根据输入或其他条件动态更改响应，我们可以利用[Mockito的Answer](https://www.baeldung.com/mockito-doanswer-thenreturn)接口**，这使我们能够在测试开始之前配置动态Mock行为：

```java
@SpringBootTest(classes = ConfigureMockBeanApplication.class)
public class MockBeanAnswersUnitTest {
    @MockBean
    private UserService mockUserService;

    @Autowired
    private UserController userController;

    @BeforeEach
    void setUp() {
        when(mockUserService.getUserName(anyLong())).thenAnswer(invocation ->{
            Long input = invocation.getArgument(0);
            if(input == 1L)
                return "John Doe";
            else if(input == 2L)
                return "Jane Smith";
            else
                return "Bob Johnson";
        });
    }

    @Test
    void whenDirectMockBean_thenReturnUserName(){
        assertEquals("John Doe", mockUserService.getUserName(1L));
        assertEquals("Jane Smith", mockUserService.getUserName(2L));
        assertEquals("Bob Johnson", mockUserService.getUserName(3L));

        verify(mockUserService).getUserName(1L);
        verify(mockUserService).getUserName(2L);
        verify(mockUserService).getUserName(3L);
    }
}
```

在这个例子中，我们根据方法调用配置了动态响应，使我们在复杂的测试场景中具有更大的灵活性。

## 4. 测试策略和注意事项

- **避免过多Mock**：过多使用@MockBean会降低测试的有效性，理想情况下，我们应该将Mock的使用限制在那些真正外部或难以控制的依赖项(例如外部API和数据库)。
- **使用@TestConfiguration进行复杂的设置**：当我们需要配置更复杂的行为时，我们应该使用@TestConfiguration，因为这使我们能够更优雅地设置Mock并为高级配置提供更好的支持。
- **验证交互**：除了设置返回值之外，验证与Mock的交互也很重要。这可确保按预期调用正确的方法。

## 5. 总结

在应用程序启动之前配置@MockBean组件是测试Spring Boot应用程序的一个有价值的策略，因为这使我们能够微调对Mock依赖项的控制。

在本文中，我们了解了配置Mock Bean组件的各种方法。通过遵循测试所需的方法并应用最佳实践和测试策略，我们可以有效地隔离组件并在受控环境中验证行为。