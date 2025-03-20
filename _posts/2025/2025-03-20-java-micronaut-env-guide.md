---
layout: post
title:  Micronaut Environment指南
category: micronaut
copyright: micronaut
excerpt: Micronaut
---

## 1. 概述

**在[Micronaut](https://www.baeldung.com/micronaut)中，与其他Java框架类似，Environment接口是与配置文件相关的抽象。配置文件是一个我们可以视为容器的概念，它保存特定于配置文件的属性和Bea**n。

通常，配置文件与执行环境相关，例如local-profile、docker-profile、k8s-profile等。我们可以使用Micronaut环境在.properties或.yaml文件中创建不同的属性集，具体取决于我们是在本地、在云端等执行应用程序。

在本教程中，我们将介绍Micronaut中的Environment抽象，并了解正确设置它的不同方法。最后，我们将学习如何使用特定于环境的属性和 bean，以及如何使用环境来应用不同的实现。

## 2. Micronaut Environment与Spring Profile

**如果我们熟悉Spring Profile，那么理解Micronaut环境就很容易了。它们有很多相似之处，但也有一些关键的区别**。

**使用Micronaut环境，我们可以以与[Spring](https://www.baeldung.com/properties-with-spring)类似的方式设置属性**，这意味着我们可以：

- 使用@ConfigurationProperties标注的属性文件
- 使用@Value注解向类注入特定属性
- 通过注入整个Environment实例向类注入特定属性，然后使用getProperty()方法

**Spring和Micronaut之间一个令人困惑的区别是，尽管两者都允许多个Environment/Profile，但在Micronaut中，通常会看到许多活动Environment，而在[Spring Profile](https://www.baeldung.com/spring-profiles)中，我们很少看到多个活动Profile**，这会导致对在许多活动环境中指定的属性或Bean产生一些混淆。为了解决这个问题，我们可以设置环境优先级，稍后会详细介绍。

**另一个值得注意的区别是Micronaut提供了完全禁用环境的选项**，这与Spring Profile无关，因为当不设置活动Profile时，通常会使用默认值。相比之下，Micronaut可能从所使用的不同框架或工具设置了不同的活动环境，例如：

- JUnit在活动环境中添加了“test”环境
- Cucumber添加“cucumber”环境
- OCI可能会添加“cloud”和/或“k8s”等。

为了禁用环境，我们可以使用`java -Dmicronaut.env.deduction=false -jar myapp.jar`。

## 3. 设置Micronaut环境

有多种方法来设置Micronaut环境。最常见的是：

- 使用micronaut.environments参数：java -Dmicronaut.environments=cloud,production -jar myapp.jar。
- 在main()中使用defaultEnvironment()方法：Micronaut.build(args).defaultEnvironments('local').mainClass(MicronautServiceApi.class).start();。
- 将MICRONAUT_ENV中的值设置为环境变量。
- 正如我们前面提到的，环境有时会被扣除，这意味着从后台的框架中设置，比如JUnit和Cucumber。

在设置环境的方式上，没有最佳实践，我们可以选择最适合我们需求的方式。

## 4. Micronaut环境优先级和解决方案

**由于允许多个活动的Micronaut环境，因此在某些情况下，属性或Bean可能在多个或没有一个中明确定义。这会导致冲突，有时还会导致运行时异常**。属性和Bean的优先级和解析处理方式不同。

### 4.1 属性

**当某个属性存在于多个活动属性源中时，环境顺序决定其获取哪个值**，从最低到最高的层次结构为：

- 从其他工具/框架导出的环境
- micronaut.environments参数中设置的环境
- 在MICRONAUT_ENV环境变量中设置的环境
- Micronaut构建器中加载的环境

假设我们有一个属性service.test.property，并且我们希望在不同的环境中为其设置不同的值。我们在application-dev.yml和application-test.yml文件中设置不同的值：

```java
@Test
public void whenEnvironmentIsNotSet_thenTestPropertyGetsValueFromDeductedEnvironment() {
    ApplicationContext applicationContext = Micronaut.run(ServerApplication.class);
    applicationContext.start();

    assertThat(applicationContext.getEnvironment()
            .getActiveNames()).containsExactly("test");
    assertThat(applicationContext.getProperty("service.test.property", String.class)).isNotEmpty();
    assertThat(applicationContext.getProperty("service.test.property", String.class)
            .get()).isEqualTo("something-in-test");
}

@Test
public void whenEnvironmentIsSetToBothProductionAndDev_thenTestPropertyGetsValueBasedOnPriority() {
    ApplicationContext applicationContext = ApplicationContext.builder("dev", "production").build();
    applicationContext.start();

    assertThat(applicationContext.getEnvironment()
            .getActiveNames()).containsExactly("test", "dev", "production");
    assertThat(applicationContext.getProperty("service.test.property", String.class)).isNotEmpty();
    assertThat(applicationContext.getProperty("service.test.property", String.class)
            .get()).isEqualTo("something-in-dev");
}
```

在第一个测试中，我们没有设置任何活动环境，但有从JUnit中推断出的测试。在这种情况下，该属性从application-test.yml获取其值。但在第二个示例中，我们在具有更高阶的ApplicationContext中也设置了dev环境，在这种情况下，该属性从application-dev.yml获取其值。

**但是如果我们尝试注入任何活动环境中都不存在的属性，我们会得到一个运行时错误DependencyInjectionException，因为缺少属性**：

```java
@Test
public void whenEnvironmentIsSetToBothProductionAndDev_thenMissingPropertyIsEmpty() {
    ApplicationContext applicationContext = ApplicationContext.builder("dev", "production")
            .build();
    applicationContext.start();

    assertThat(applicationContext.getEnvironment()
            .getActiveNames()).containsExactly("test", "dev", "production");
    assertThat(applicationContext.getProperty("service.dummy.property", String.class)).isEmpty();
}
```

在此示例中，我们尝试直接从ApplicationContext检索缺失的属性system.dummy.property，这将返回一个空的Optional。如果该属性被注入到某个Bean中，则会导致运行时异常。

### 4.2 Bean

对于特定于环境的Bean，事情会稍微复杂一些。假设我们有一个EventSourcingService接口，它有一个方法sendEvent()(它应该是void，但为了演示目的我们返回了String)：

```java
public interface EventSourcingService {
    String sendEvent(String event);
}
```

该接口只有两个实现，一个用于环境开发，一个用于生产：

```java
@Singleton
@Requires(env = Environment.DEVELOPMENT)
public class VoidEventSourcingService implements EventSourcingService {
    @Override
    public String sendEvent(String event) {
        return "void service. [" + event + "] was not sent";
    }
}
```

```java
@Singleton
@Requires(env = "production")
public class KafkaEventSourcingService implements EventSourcingService {
    @Override
    public String sendEvent(String event) {
        return "using kafka to send message: [" + event + "]";
    }
}
```

@Requires注解告知框架，此实现仅在指定的一个或多个环境处于活动状态时才有效。否则，永远不会创建此Bean。

我们可以假设VoidEventSourcingService什么都不做，只返回一个字符串，因为也许我们不想在开发环境中发送事件。而KafkaEventSourcingService实际上在Kafka上发送事件，然后返回一个字符串。

**现在，如果我们忘记在活动环境中设置其中一个，会发生什么？在这种情况下，我们会得到一个NoSuchBeanException异常**：

```java
public class InvalidEnvironmentEventSourcingUnitTest {
    @Test
    public void whenEnvironmentIsNotSet_thenEventSourcingServiceBeanIsNotCreated() {
        ApplicationContext applicationContext = Micronaut.run(ServerApplication.class);
        applicationContext.start();

        assertThat(applicationContext.getEnvironment().getActiveNames()).containsExactly("test");
        assertThatThrownBy(() -> applicationContext.getBean(EventSourcingService.class))
                .isInstanceOf(NoSuchBeanException.class)
                .hasMessageContaining("None of the required environments [production] are active: [test]");
    }
}
```

在此测试中，我们没有设置任何活动环境。首先，我们断言唯一活动的环境是test，这是通过使用JUnit框架得出的。然后我们断言，如果我们尝试获取EventSourcingService实现的Bean ，我们实际上会得到一个异常，错误表明所有必需的环境都不是活跃的。

**相反，如果我们设置两个环境，我们会再次收到错误，因为接口的两个实现不能同时存在**：

```java
public class MultipleEnvironmentsEventSourcingUnitTest {
    @Test
    public void whenEnvironmentIsSetToBothProductionAndDev_thenEventSourcingServiceBeanHasConflict() {
        ApplicationContext applicationContext = ApplicationContext.builder("dev", "production").build();
        applicationContext.start();

        assertThat(applicationContext.getEnvironment()
                .getActiveNames()).containsExactly("test", "dev", "production");
        assertThatThrownBy(() -> applicationContext.getBean(EventSourcingService.class))
                .isInstanceOf(NonUniqueBeanException.class)
                .hasMessageContaining("Multiple possible bean candidates found: [VoidEventSourcingService, KafkaEventSourcingService]");
    }
}
```

这不是错误或糟糕的编码，这可能是一个现实生活中的场景，在这种情况下，当我们忘记设置正确的环境时，我们可能希望出现故障。但是，如果我们想确保在这种情况下永远不会出现运行时错误，我们可以通过不添加@Requires注解来设置默认实现。对于我们想要覆盖默认的环境，我们应该添加@Requires和@Replaces注解：

```java
public interface LoggingService {
    // methods omitted
}

@Singleton
@Requires(env = { "production", "canary-production" })
@Replaces(LoggingService.class)
public class FileLoggingServiceImpl implements LoggingService {
    // implementation of the methods omitted
}

@Singleton
public class ConsoleLoggingServiceImpl implements LoggingService {
    // implementation of methods omitted
}
```

LoggingService接口定义了一些方法，默认实现是ConsoleLoggingServiceImpl，适用于所有环境。FileLoggingServiceImpl类在production和canary-production环境中覆盖了默认实现。

## 5. 在实践中使用Micronaut环境

除了特定于环境的属性和Bean之外，我们还可以在更多情况下使用环境。通过注入环境变量并使用getActiveNames()方法，我们可以在代码中检查活动环境是什么并更改一些实现细节：

```java
if (environment.getActiveNames().contains("production") || environment.getActiveNames().contains("canary-production")) {
    sendAuditEvent();
}
```

这段代码检查环境是production环境还是canary-production环境，并仅在这两种环境中调用sendAuditEvent()方法。**这当然是一种不好的做法。相反，我们应该使用[策略设计模式](https://www.baeldung.com/java-strategy-pattern)或特定的Bean**，如前所述。

但我们还有选择，更常见的情况是在测试中使用此代码，因为我们的测试代码有时更简单而不是更干净：

```java
if (environment.getActiveNames().contains("cloud")) {
    assertEquals(400, response.getStatusCode());
} else {
    assertEquals(500, response.getStatusCode());
}
```

这是一个测试片段，由于服务未处理某些错误请求，因此可能会在本地环境中收到500状态响应。另一方面，在部署环境中会给出400状态响应，因为API网关在请求到达服务之前会做出响应。

## 6. 总结

在本文中，我们了解了Micronaut环境。我们介绍了主要概念和与Spring Profile的相似之处，并列出了设置活动环境的不同方法。然后，我们了解了在设置了多个环境或未设置多个环境的情况下如何解析特定于环境的Bean和属性。最后，我们讨论了如何在代码中直接使用环境，这通常不是一种好的做法。