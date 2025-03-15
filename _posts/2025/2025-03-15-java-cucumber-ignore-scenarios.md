---
layout: post
title:  如何忽略Cucumber中的场景
category: bdd
copyright: bdd
excerpt: Cucumber
---

## 1. 简介

在本教程中，我们将学习如何在Cucumber中有选择地运行或跳过场景。

[Cucumber](https://www.baeldung.com/cucumber-rest-api-testing)是一款支持[行为驱动开发(BDD)](https://cucumber.io/docs/bdd/)的工具，它以纯文本形式读取可执行规范，并验证软件是否按照规范执行。

## 2. 创建场景

首先，让我们创建几个场景。

让我们以生成问候消息的API端点/greetings为例，我们可以为其编写以下场景：

```gherkin
Feature: Time based Greeter
    # Morning
    Scenario: Should greet Good Morning in the morning
        Given the current time is "0700" hours
        When I ask the greeter to greet
        Then I should receive "Good Morning!"
    # Evening
    Scenario: Should greet Good Evening in the evening
        Given the current time is "1900" hours
        When I ask the greeter to greet
        Then I should receive "Good Evening!"
    # Night
    Scenario: Should greet Good Night in the night
        Given the current time is "2300" hours
        When I ask the greeter to greet
        Then I should receive "Good Night!"
    # Midnight
    Scenario: Should greet Good Night at midnight
        Given the current time is "0000" hours
        When I ask the greeter to greet
        Then I should receive "Good Night!"
```

现在让我们实现一个根据上述规范生成响应的控制器方法：

```java
@GetMapping("/greetings")
@ResponseBody
public String greet(@RequestParam("hours") String hours) {
    String greeting;
    int currentHour = Integer.parseInt(hours.substring(0, 2));
    if (currentHour >= 6 && currentHour < 12) {
        greeting = "Good Morning!";
    } else if (currentHour >= 12 && currentHour < 16) {
        greeting = "Good Afternoon!";
    } else if (currentHour >= 16 && currentHour <= 19) {
        greeting = "Good Evening!";
    } else {
        greeting = "Good Night!";
    }
    return greeting;
}
```

## 3. 方法

现在，我们介绍如何忽略刚刚定义的一个或多个场景。

### 3.1 使用自定义标签进行标记

在这种方法中，我们使用自定义注解来标注我们的场景，它可以是我们选择的任何字符串，例如@ignore、@skip或@disable。让我们使用完全随机的东西，比如@custom-ignore来确认我们选择什么都没关系。但是，有意义的东西是更好的选择。

现在，这是一个两步过程。首先，**我们将使用注解标记场景**，让我们使用此注解标记第三个场景：

```gherkin
# Night
@custom-ignore
Scenario: Should greet Good Night in the night
    Given the current time is "2300" hours
    When I ask the greeter to greet
    Then I should receive "Good Night!"
```

**然后，我们需要在测试resources文件夹下的junit-platform.properties中添加属性cucumber.filter.tags**：

```properties
cucumber.filter.tags=not @custom-ignore
```

就是这样。在后续的测试运行中，此场景将被忽略。

**值得注意的是，如果我们使用JUnit 4，则需要使用Cucumber TestRunner类上的[@CucumberOptions](https://cucumber.io/docs/cucumber/api/#ignoring-a-subset-of-scenarios)注解指定相同的配置**：

```java
@CucumberOptions(tags = "not @custom-ignore")
```

此外，我们还可以[仅运行选择性标签](https://www.baeldung.com/java-overriding-cucumber-option-values#using-the-cucumberoptions-annotation-a-subset-of-scenarios)。

### 3.2 注释掉场景

值得注意的是，还有其他选择，但它们可能不像第一种方法那样惯用。

其中一种方法是注释掉该场景：

```gherkin
# Night
#  Scenario: Should greet Good Night in the night
#      Given the current time is "2300" hours
#      When I ask the greeter to greet
#      Then I should receive "Good Night!"
```

然而，这很容易出错，因为我们可能会忽略注释和取消注释多行。

## 4. 总结

在本文中，我们研究了如何使用自定义标签/注释可靠地忽略Cucumber中的一个或多个场景。

值得注意的是，Cucumber允许使用灵活的标签来注释和跳过场景。虽然我们也探索了其他选项，但我们了解到使用基于标签的方法是实现此目标的惯用方法。