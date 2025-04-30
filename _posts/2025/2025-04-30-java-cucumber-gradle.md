---
layout: post
title:  将Cumber与Gradle结合使用
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

[Cucumber](https://www.baeldung.com/cucumber-rest-api-testing)是一款支持行为驱动开发(BDD)的测试自动化工具，它运行以纯文本Gherkin语法编写的规范，该规范描述了系统行为。

在本教程中，我们将了解将Cucumber与Gradle集成的几种方法，以便在项目构建过程中运行BDD规范。

## 2. 设置

首先，让我们使用[Gradle Wrapper](https://www.baeldung.com/gradle-wrapper)建立一个Gradle项目。

接下来，我们将[cucumber-java](https://mvnrepository.com/artifact/io.cucumber/cucumber-java)依赖添加到build.gradle：

```groovy
testImplementation 'io.cucumber:cucumber-java:6.10.4'
```

这会将官方的Cucumber Java实现添加到我们的项目中。

## 3. 使用自定义任务运行

为了使用Gradle运行我们的规范，**我们将创建一个使用Cucumber的命令行界面运行器(CLI)的任务**。

### 3.1 配置

让我们首先将所需的配置添加到项目的build.gradle文件中：

```groovy
configurations {
    cucumberRuntime {
        extendsFrom testImplementation
    }
}
```

接下来，我们将创建自定义cucumberCli任务：

```groovy
task cucumberCli() {
    dependsOn assemble, testClasses
    doLast {
        javaexec {
            main = "io.cucumber.core.cli.Main"
            classpath = configurations.cucumberRuntime + sourceSets.main.output + sourceSets.test.output
            args = [
                '--plugin', 'pretty',
                '--plugin', 'html:target/cucumber-report.html', 
                '--glue', 'cn.tuyucheng.taketoday.cucumber', 
                'src/test/resources']
        }
    }
}
```

此任务配置为运行src/test/resources目录下的.feature文件中找到的所有测试场景。

Main类的–glue选项指定运行场景所需的步骤定义文件的位置。

–plugin选项指定测试报告的格式和位置，我们可以组合多个值来生成所需格式的报告，例如本例中的pretty和HTML格式。

还有[其他](https://github.com/cucumber/cucumber-jvm/blob/main/cucumber-core/src/main/resources/io/cucumber/core/options/USAGE.txt)几个可用选项，例如，有一些选项可以根据名称和标签过滤测试。

### 3.2 场景

现在，让我们在src/test/resources/features/account_credited.feature文件中为我们的应用程序创建一个简单的场景：

```groovy
Feature: Account is credited with amount

    Scenario: Credit amount
        Given account balance is 0.0
        When the account is credited with 10.0
        Then account should have a balance of 10.0
```

接下来，我们将实现运行场景所需的相应步骤定义：

```java
public class StepDefinitions {

    @Given("account balance is {double}")
    public void givenAccountBalance(Double initialBalance) {
        account = new Account(initialBalance);
    }

    // other step definitions
}
```

### 3.3 运行任务

最后，让我们从命令行运行cucumberCli任务：

```shell
>> ./gradlew cucumberCli

> Task :cucumberCli

Scenario: Credit amount                      # src/test/resources/features/account_credited.feature:3
  Given account balance is 0.0               # cn.tuyucheng.taketoday.cucumber.StepDefinitions.account_balance_is(java.lang.Double)
  When the account is credited with 10.0     # cn.tuyucheng.taketoday.cucumber.StepDefinitions.the_account_is_credited_with(java.lang.Double)
  Then account should have a balance of 10.0 # cn.tuyucheng.taketoday.cucumber.StepDefinitions.account_should_have_a_balance_of(java.lang.Double)

1 Scenarios (1 passed)
3 Steps (3 passed)
0m0.381s
```

可以看到，我们的规范已与Gradle集成，并成功运行，输出已显示在控制台上。此外，HTML测试报告也可在指定位置获取。

## 4. 使用JUnit运行

我们可以使用JUnit来运行Cucumber场景，而不是在Gradle中创建自定义任务。

让我们首先包含[cucumber-junit](https://mvnrepository.com/artifact/io.cucumber/cucumber-junit)依赖：

```groovy
testImplementation 'io.cucumber:cucumber-junit:6.10.4'
```

由于我们使用的是JUnit 5，因此我们还需要添加[junit-vintage-engine](https://mvnrepository.com/artifact/org.junit.vintage/junit-vintage-engine)依赖：

```groovy
testImplementation 'org.junit.vintage:junit-vintage-engine:5.7.2'
```

接下来，我们将在测试源位置创建一个空的运行器类：

```java
@RunWith(Cucumber.class)
@CucumberOptions(
        plugin = {"pretty", "html:target/cucumber-report.html"},
        features = {"src/test/resources"}
)
public class RunCucumberTest {
}
```

这里，我们在@RunWith注解中使用了JUnit Cucumber运行器，此外，所有CLI运行器选项(例如features和plugin)都可以通过[@CucumberOptions](https://github.com/cucumber/cucumber-jvm/blob/main/cucumber-junit/src/main/java/io/cucumber/junit/CucumberOptions.java)注解使用。

现在，**执行标准Gradle test任务将查找并运行所有功能测试**，以及任何其他单元测试：

```shell
>> ./gradlew test

> Task :test

RunCucumberTest > Credit amount PASSED

BUILD SUCCESSFUL in 2s
```

## 5. 使用插件运行

**最后一种方法是使用第三方插件，该插件提供从Gradle构建运行规范的能力**。

在我们的示例中，我们将使用[gradle-cucumber-runner](https://github.com/tsundberg/gradle-cucumber-runner)插件来运行Cucumber JVM，它会将所有调用转发到我们之前使用的CLI运行器，让我们将它添加到我们的项目中：

```groovy
plugins {
    id "se.thinkcode.cucumber-runner" version "0.0.8"
}
```

这会将Cucumber任务添加到我们的构建中，现在我们可以使用默认设置运行它：

```shell
>> ./gradlew cucumber
```

值得注意的是，这不是官方的Cucumber插件，还有其他插件可以提供类似的功能。

## 6. 总结

在本文中，我们演示了使用Gradle配置和运行BDD规范的几种方法。

首先，我们研究了如何使用CLI运行器创建自定义任务。然后，我们研究了如何使用Cucumber JUnit运行器通过现有的Gradle任务执行规范。最后，我们使用第三方插件来运行Cucumber，而无需创建自定义任务。