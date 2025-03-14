---
layout: post
title: 将List作为Cucumber参数传递
category: bdd
copyright: bdd
excerpt: Cucumber
---

## 1. 简介

[Cucumber](https://www.baeldung.com/cucumber-rest-api-testing)是[行为驱动开发(BDD)](https://www.baeldung.com/cs/bdd-guide)中一种流行的工具，用于以通俗易懂的语言编写测试场景，利用Cucumber中的参数可以实现动态且可重复使用的测试。

在本文中，我们将讨论在Java测试中使用Cucumber参数的基本技术。

## 2. 了解Cucumber参数

**Cucumber参数是功能文件中的占位符，允许使用不同的输入来执行场景**。

以下是说明字符串参数的基本示例：

```gherkin
Feature: User Login
    Scenario: User logs into the system
      Given the user enters username "john_doe" and password "password123"
      When the user clicks on the login button
      Then the dashboard should be displayed
```

步骤定义使用注解捕获这些参数：

```java
public class LoginSteps {
    private static final Logger logger = LoggerFactory.getLogger(LoginSteps.class);

    @Given("the user enters username {string} and password {string}")
    public void enterUsernameAndPassword(String username, String password) {
        logger.info("Username: {}, Password: {}", username, password);
    }

    @When("the user clicks on the login button")
    public void clickLoginButton() {
        logger.info("Login button clicked");
    }

    @Then("the dashboard should be displayed")
    public void verifyDashboardDisplayed() {
        logger.info("Dashboard displayed");
    }
}
```

在这里，我们定义与功能文件中每个步骤相对应的步骤定义。**@Given、@When和@Then[注解](https://www.baeldung.com/bdd-mockito)捕获场景中指定的参数并将其传递给相应的方法**。

## 3. 使用DataTable作为列表参数

[DataTable](https://www.baeldung.com/cucumber-data-tables)允许使用多组数据进行测试。

让我们看一个如何使用DataTable的例子：

```gherkin
Scenario: Verify user login with multiple accounts
    Given the user tries to log in with the following accounts:
        | username    | password   |
        | john_doe    | password1  |
        | jane_smith  | password2  |
    When each user attempts to log in
    Then each user should access the system successfully
```

我们使用自定义对象列表来处理DataTable：

```java
public class DataTableLoginSteps {
    private static final Logger logger = LoggerFactory.getLogger(DataTableLoginSteps.class);

    @Given("the user tries to log in with the following accounts:")
    public void loginUser(List<UserCredentials> credentialsList) {
        for (UserCredentials credentials : credentialsList) {
            logger.info("Username: {}, Password: {}", credentials.getUsername(), credentials.getPassword());
        }
    }

    public static class UserCredentials {
        private String username;
        private String password;

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }
    }
}
```

在此示例中，我们使用DataTable类将功能文件中的数据表映射到UserCredentials对象列表，**这使我们能够遍历该列表并单独处理每组凭据**。

## 4. 使用正则表达式

[正则表达式](https://www.baeldung.com/regular-expressions-java)允许灵活的参数匹配，让我们考虑以下电子邮件验证示例：

```gherkin
Scenario: Verify email validation
    Given the user enters email address "john.doe@example.com"
    When the user submits the registration form
    Then the email should be successfully validated
```

我们通过步骤定义捕获电子邮件地址参数：

```java
public class RegexLoginSteps {
    private static final Logger logger = LoggerFactory.getLogger(RegexLoginSteps.class);

    @Given("the user enters email address \"([^\"]*)\"")
    public void enterEmailAddress(String emailAddress) {
        logger.info("Email: {}", emailAddress);
    }
}
```

这里我们使用正则表达式来捕获电子邮件地址参数，正则表达式\\"(\[^\\"]*)\\"可匹配任何用双引号括起来的字符串，这使我们能够处理动态电子邮件地址。

## 5. 使用自定义转换器

自定义转换器将输入字符串转换为特定的Java类型，我们来看一个例子：

```gherkin
Scenario: Verify date transformation
    Given the user enters birthdate "1990-05-15"
    When the user submits the registration form
    Then the birthdate should be stored correctly
```

接下来，让我们定义处理数据转换的步骤定义：

```java
public class LocalDateTransformer {
    private static final Logger logger = LoggerFactory.getLogger(LocalDateTransformer.class);

    @ParameterType("yyyy-MM-dd")
    public LocalDate localdate(String dateString) {
        return LocalDate.parse(dateString);
    }

    @Given("the user enters birthdate {localdate}")
    public void enterBirthdate(LocalDate birthdate) {
        logger.info("Birthdate: {}", birthdate);
    }
}
```

在此方法中，我们使用@ParameterType注解定义自定义转换器，**此转换器将“yyyy-MM-dd”格式的输入字符串转换为[LocalDate](https://www.baeldung.com/java-8-date-time-intro)对象，使我们能够在步骤定义中直接使用日期对象**。

## 6. 总结

在Cucumber中将列表作为参数传递可以增强我们有效测试各种数据集的能力，无论我们使用DataTable、正则表达式还是自定义转换器，目标始终一致：简化测试流程并确保全面覆盖。