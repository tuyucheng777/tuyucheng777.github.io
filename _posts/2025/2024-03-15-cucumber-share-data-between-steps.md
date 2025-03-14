---
layout: post
title: 如何在Cucumber中的各个步骤之间共享数据
category: bdd
copyright: bdd
excerpt: Cucumber
---

## 1. 简介

Cucumber是一种支持[行为驱动开发(BDD)](https://www.baeldung.com/cs/bdd-guide)方法的测试工具，**BDD使非技术利益相关者能够使用自然的领域特定语言描述业务特性**。

定义[Cucumber测试](https://www.baeldung.com/cucumber-rest-api-testing)的构建块是功能、场景和步骤，必须用[Gherkin](https://cucumber.io/docs/gherkin/)语言编写。在本教程中，我们将学习如何在Cucumber中的步骤之间共享数据。

## 2. 设置

我们将使用一个处理事件的简单Spring Boot应用程序来展示如何在[Cucumber](https://www.baeldung.com/cucumber-spring-integration)步骤之间共享数据，并编写BDD测试来验证事件的生命周期，从事件进入我们系统时的初始状态到处理后的最终状态。

首先，让我们定义事件类：

```java
public class Event {
    private String uuid;
    private EventStatus status;

    // standard getters and setters
}

public enum EventStatus {
    PROCESSING, ERROR, COMPLETE
}
```

**当我们处理一个事件时，它会从初始的PROCESSING状态转换为最终状态，即COMPLETE或ERROR**。

现在，让我们编写初始场景和步骤定义：

```gherkin
Scenario: new event is properly initialized
    When new event enters the system
    Then event is properly initialized
```

在下一节中，我们将看到如何在两个步骤之间共享事件数据。

## 3. 使用Spring在步骤之间共享数据

@ScenarioScope注解允许我们在场景中的不同步骤之间共享状态，**该注解指示Spring为每个场景创建一个新实例，使步骤定义能够在它们之间共享数据，并确保不同场景之间的状态不会泄露**。

在实现初始场景的步骤定义之前，我们还要实现SharedEvent测试类，它将存储步骤之间要共享的状态：

```java
@Component
@ScenarioScope
public class SharedEvent {
    private Event event;
    private Instant createdAt;
    private Instant processedAt;

    // standard getters and setters
}
```

**SharedEvent类通过引入附加字段createdAt和processedAt对Event类进行了补充，帮助我们定义更复杂的场景**。

最后，我们来定义一下场景中使用的步骤：

```java
public class EventSteps {
    static final String UUID = "1ed80153-666c-4904-8e03-08c4a41e716a";
    static final String CREATED_AT = "2024-12-03T09:00:00Z";

    @Autowired
    private SharedEvent sharedEvent;

    @When("new event enters the system")
    public void createNewEvent() {
        Event event = new Event();
        event.setStatus(EventStatus.PROCESSING);
        event.setUuid(UUID);
        sharedEvent.setEvent(event);
        sharedEvent.setCreatedAt(Instant.parse(CREATED_AT));
    }

    @Then("event is properly initialized")
    public void verifyEventIsInitialized() {
        Event event = sharedEvent.getEvent();
        assertThat(event.getStatus()).isEqualTo(EventStatus.PROCESSING);
        assertThat(event.getUuid()).isEqualTo(UUID);
        assertThat(sharedEvent.getCreatedAt().toString()).isEqualTo(CREATED_AT);
        assertThat(sharedEvent.getProcessedAt()).isNull();
    }
}
```

运行功能文件，我们看到createNewEvent()和verifyEventIsInitialized()方法通过SharedEvent类成功共享数据。

### 3.1 扩展场景

现在，让我们进一步编写另外两个场景，每个场景都有多个步骤：

```gherkin
Scenario: event is processed successfully
    Given new event enters the system
    When event processing succeeds
    Then event has COMPLETE status
    And event has processedAt

Scenario: event is is not processed due to system error
    Given new event enters the system
    When event processing fails
    Then event has ERROR status
    And event has processedAt
```

我们还将新的步骤定义添加到EventSteps类中，这将展示额外的Gherkin功能，从而提高步骤的可读性和可重用性。

首先，让我们实现When步骤：

```java
static final String PROCESSED_AT = "2024-12-03T10:00:00Z";

@When("event processing (succeeds|fails)$")
public void processEvent(String processingStatus) {
    // process event ...

    EventStatus eventStatus = "succeeds".equalsIgnoreCase(processingStatus) ? EventStatus.COMPLETE : EventStatus.ERROR;
    sharedEvent.getEvent().setStatus(eventStatus);
    sharedEvent.setProcessedAt(Instant.parse(PROCESSED_AT));
}
```

字符串“event processing (succeeds|fails)\$”是一个[正则表达式](https://www.baeldung.com/regular-expressions-java)，允许重复使用此步骤定义，匹配“event processing succeeds”和“event processing failed”步骤。**正则表达式末尾的$字符确保步骤完全匹配，而不包含任何尾随字符**。

接下来，让我们实现负责检查processedAt字段的Then步骤：

```java
@Then("event has processedAt")
public void verifyProcessedAt() {
    assertThat(sharedEvent.getProcessedAt().toString()).isEqualTo(PROCESSED_AT);
}
```

最后，让我们添加最后的Then步骤，验证事件状态：

```java
@Then("event has {status} status")
public void verifyEventStatus(EventStatus status) {
    assertThat(sharedEvent.getEvent().getStatus()).isEqualTo(status);
}

@ParameterType("PROCESSING|ERROR|COMPLETE")
public EventStatus status(String statusName) {
    return EventStatus.valueOf(statusName);
}
```

Cucumber中的ParameterType允许我们将方法参数从Cucumber表达式转换为对象，**@ParameterType status()方法将字符串转换为EventStatus枚举值**，这是使用步骤定义中的{status}占位符调用的。因此，我们的步骤定义将匹配三个步骤：

- 事件处于“PROCESSING”状态
- 事件处于“COMPLETE”状态
- 事件处于“ERROR”状态

新的场景现在已经准备就绪，在运行它们时，我们注意到多个步骤成功共享数据。

## 4. 总结

在本教程中，我们学习了如何在Cucumber中的步骤之间共享数据。**此功能允许我们在场景中的步骤之间共享数据，同时保持数据在多个场景之间隔离**。值得一提的是，在步骤之间共享数据会使它们紧密耦合，从而降低它们的可重用性。