---
layout: post
title:  将JUnit 5与Gradle结合使用
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

**在本教程中，我们将使用Gradle构建工具在新的JUnit 5平台上运行测试**。 

我们将配置一个同时支持旧版本和新版本的项目。

如需了解更多关于新版本的信息，请阅读[JUnit 5指南](https://www.baeldung.com/junit-5)，如需深入了解该构建工具，请阅读[Gradle简介](https://www.baeldung.com/gradle)。

## 2. Gradle设置

**首先，我们验证是否安装了构建工具4.6或更高版本，因为这是与JUnit 5兼容的最早版本**。

最简单的方法就是运行gradle-v命令：

```shell
$> gradle -v
------------------------------------------------------------
Gradle 4.10.2
------------------------------------------------------------
```

并且，如果需要，我们可以按照[安装](https://gradle.org/install/)步骤获取正确的版本。

**一旦我们安装了所有内容，我们就需要使用build.gradle文件来配置Gradle**。

我们可以从向构建工具提供单元测试平台开始：

```groovy
test {
    useJUnitPlatform()
}
```

现在我们已经指定了平台，我们需要提供JUnit依赖，这就是JUnit 5与早期版本之间一个显著区别的地方。

看看吧，在早期版本中，我们只需要一个依赖。然而，**在JUnit 5中，API与运行时分离，这意味着需要两个依赖**。

API清单中列出了junit-jupiter-api，运行时是junit-jupiter-engine(适用于JUnit 5)和junit-vintage-engine(适用于JUnit 3或4)。

我们将分别在testImplementation和 timeRuntimeOnly中提供这两个：

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'
}
```

## 3. 创建测试

让我们写第一个测试，它看起来和之前的版本一样：

```java
@Test
public void testAdd() {
    assertEquals(42, Integer.sum(19, 23));
}
```

现在，**我们可以通过执行gradle clean test命令来运行测试**。

为了验证我们使用的是JUnit 5，我们可以查看导入，**@Test和assertEquals的导入应该包含一个以org.junit.jupiter.api开头的包**：

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
```

在上一个示例中，我们创建了一个使用多年前“旧”功能的测试。现在，我们将创建另一个示例，其中使用了JUnit 5中的一些新功能：

```java
@Test
public void testDivide() {
    assertThrows(ArithmeticException.class, () -> {
        Integer.divideUnsigned(42, 0);
    });
}
```

**assertThrows是JUnit5中的一种新断言，它取代了旧样式的@Test(expected=ArithmeticException.class)**。

## 4. 使用Gradle配置JUnit 5测试

接下来，我们将探讨Gradle和JUnit5之间更深层次的集成。

假设我们的套件中有两种类型的测试：长时间运行的和短时间运行的。我们可以使用JUnit 5的@Tag注解：

```java
public class CalculatorJUnit5Test {
    @Tag("slow")
    @Test
    public void testAddMaxInteger() {
        assertEquals(2147483646, Integer.sum(2147183646, 300000));
    }
 
    @Tag("fast")
    @Test
    public void testDivide() {
        assertThrows(ArithmeticException.class, () -> {
            Integer.divideUnsigned(42, 0);
        });
    }
}
```

**然后，我们告诉构建工具要执行哪些测试**。在本例中，我们只执行短期运行(快速)的测试：

```java
test {
    useJUnitPlatform {
    	includeTags 'fast'
        excludeTags 'slow'
    }
}
```

## 5. 启用对旧版本的支持

**现在，仍然可以使用新的Jupiter引擎创建JUnit 3和4测试**。甚至，我们可以在同一个项目中混合使用它们和新版本，例如在迁移场景中。

首先，我们向现有的构建配置添加一些依赖：

```groovy
testCompileOnly 'junit:junit:4.12' 
testRuntimeOnly 'org.junit.vintage:junit-vintage-engine:5.8.1'
```

**请注意我们的项目现在同时拥有junit-jupiter-engine和junit-vintage-engine**。

现在我们创建一个新类，并复制粘贴之前创建的testDivide方法。然后，我们添加@Test和assertEquals的导入。不过，这次我们确保使用旧版本4的包，以org.junit开头：

```java
import static org.junit.Assert.assertEquals;
import org.junit.Test;
public class CalculatorJUnit4Test {
    @Test
    public void testAdd() {
        assertEquals(42, Integer.sum(19, 23));
    }
}
```

## 6. 总结

在本教程中，我们将Gradle与JUnit 5集成。此外，我们还添加了对版本3和4的支持。

我们已经看到，构建工具对新旧版本都提供了出色的支持。因此，我们可以在现有项目中使用新功能，而无需更改所有现有测试。