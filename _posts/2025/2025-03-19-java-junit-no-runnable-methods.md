---
layout: post
title:  避免JUnit中的“no runnable methods”错误
category: unittest
copyright: unittest
excerpt: JUnit 5
---

## 1. 概述

[JUnit](https://www.baeldung.com/junit-5)是Java[单元测试](https://www.baeldung.com/java-unit-testing-best-practices#whatIsUnitTesting)的首选，在测试执行过程中，开发人员经常会遇到一个奇怪的错误，即使我们导入了正确的类，也提示没有可运行的方法。

在本教程中，我们将看到导致此错误的一些具体情况以及如何修复它们。

## 2. 缺少@Test注解

首先，测试引擎必须识别测试类才能执行测试。如果没有有效的测试可运行，我们将收到异常：

```text
java.lang.Exception: No runnable methods
```

**为了避免这种情况，我们需要确保测试类始终使用JUnit库中的@Test注解进行标注**。

对于JUnit 4.x，我们应该使用：

```java
import org.junit.Test;
```

另一方面，如果我们的测试库是JUnit 5.x，我们应该从[JUnit Jupiter](https://www.baeldung.com/junit-5#2-junit-jupiter)导入包：

```java
import org.junit.jupiter.api.Test;
```

另外，我们要特别注意[TestNG](https://www.baeldung.com/testng)框架的@Test注解：

```java
import org.testng.annotations.Test;
```

当我们导入该类来代替JUnit的@Test注解时，它可能会导致“no runnable methods”错误。

## 3. 混合使用JUnit 4和JUnit 5

一些遗留项目可能在类路径中同时包含JUnit 4和JUnit 5库，虽然当我们混合使用这两个库时编译器不会报告任何错误，但在运行JUnit时我们可能会遇到“no runnable methods”错误。让我们看一些案例。

### 3.1 错误的JUnit 4导入

这种情况主要是由于IDE的自动导入功能会导入第一个匹配的类，让我们看看正确的JUnit 4导入：

```java
import org.junit.After;
import org.junit.AfterClass;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.Test;
import static org.junit.Assert.*;
```

**如上所示，我们可以注意到org.junit包包含JUnit 4的核心类**。 

### 3.2 错误的JUnit 5导入

类似地，我们可能会错误地导入JUnit 4类而不是JUnit 5，导致测试无法运行。因此，我们需要确保为JUnit 5导入了正确的类：

```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
```

**在这里，我们可以看到JUnit 5的核心类属于org.junit.jupiter.api包**。

### 3.3 @RunWith和@ExtendWith注解

对于使用Spring框架的项目，我们需要导入一个特殊的注解来进行集成测试。对于JUnit 4，**我们使用[@RunWith](https://www.baeldung.com/junit-5-migration#5-new-annotations-for-running-tests)注解来加载Spring TestContext框架**。 

但是，**对于在JUnit 5上编写的测试，我们应该使用[@ExtendWith](https://www.baeldung.com/integration-testing-in-spring#1-enable-spring-in-tests-with-junit-5)注解来获得相同的行为**。当我们将这两个注解与不同的JUnit版本互换时，测试引擎可能找不到这些测试。

此外，为了对基于Spring Boot的应用程序执行JUnit 5测试，我们可以使用[@SpringBootTest](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/context/SpringBootTest.html)注解，它在@ExtendWith注解之上提供附加功能。

## 4. 测试实用程序类

当我们想在不同的类中重用相同的代码时，工具类很有用。因此，测试工具类或父类共享一些通用的设置或初始化方法。由于类命名，测试引擎将这些类识别为真正的测试类并尝试找到可测试的方法。

我们来看一个工具类：

```java
public class NameUtilTest {
    public String formatName(String name) {
        return (name == null) ? name : name.replace("$", "_");
    }
}
```

在这种情况下，**我们可以观察到NameUtilTest类符合真实测试类的[命名约定](https://www.baeldung.com/maven-cant-find-junit-tests#naming-conventions)**。但是，没有使用@Test标注的方法，这会导致“no runnable methods”错误。为了避免这种情况，我们可以重新考虑这些实用程序类的命名。

因此，以“Test”结尾的实用程序类可以重命名为“TestHelper”或类似的：

```java
public class NameUtilTestHelper {
    public String formatName(String name) {
        return (name == null) ? name : name.replace("$", "_");
    }
}
```

或者，我们可以为以“Test”模式结尾的父类(例如BaseTest)指定abstract修饰符，以阻止该类执行测试。

## 5. 明确忽略的测试

虽然这不是常见的情况，但有时所有测试方法或整个测试类都可能被错误地标记为可跳过。

**@Ignore(JUnit 4)和@Disabled(JUnit 5)注解可用于暂时阻止某些测试运行**。当测试修复很复杂或我们需要紧急部署时，这可能是一种快速修复方法，可让构建重回正轨：

```java
public class JUnit4IgnoreUnitTest {
    @Ignore
    @Test
    public void whenMethodIsIgnored_thenTestsDoNotRun() {
        Assert.assertTrue(true);
    }
}
```

在上述情况下，封闭的JUnit4IgnoreUnitTest类只有一个方法，并且被标记为@Ignore。当我们使用IDE或Maven构建运行测试时，这可能会导致“no runnable methods”错误，因为Test类没有可测试的方法。

**为了避免此错误，最好删除@Ignore注解或至少执行一种有效的测试方法**。

## 6. 总结

在本文中，我们看到了一些在JUnit中运行测试时出现“no runnable methods”错误的情况，以及如何解决每种情况。

首先，我们发现缺少正确的@Test注解可能会导致此错误。其次，我们了解到混合使用JUnit 4和JUnit 5中的类也会导致同样的情况。我们还观察到了命名测试实用程序类的最佳方法。最后，我们讨论了明确忽略的测试以及它们如何成为问题。