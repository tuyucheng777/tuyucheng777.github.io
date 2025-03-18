---
layout: post
title:  使用Mockito Mock枚举
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

枚举是一种特殊类型的类，它扩展了[Enum](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/lang/Enum.html)类。它由一组[static](https://www.baeldung.com/java-static) [final](https://www.baeldung.com/java-final)值组成，这使得它们不可更改。此外，我们使用枚举来定义一个变量可以保存的一组预期值。这样，我们的代码更易读，更不容易出错。

但是，在测试时，我们在某些情况下想要Mock枚举值。在本教程中，我们将学习如何使用[Mockito](https://www.baeldung.com/mockito-series)库Mock枚举，并讨论这可能有用的情况。

## 2. 依赖设置

在深入研究之前，让我们将[mockito-core](https://mvnrepository.com/artifact/org.mockito/mockito-core/)依赖项添加到我们的项目中：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.15.2</version>
    <scope>test</scope>
</dependency>
```

**值得注意的是，我们需要使用依赖版本5.0.0或更高版本才能Mock静态方法**。

对于旧版本的Mockito，我们需要在pom.xml中添加额外的[mockito-inline](https://mvnrepository.com/artifact/org.mockito/mockito-inline)依赖：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-inline</artifactId>
    <version>5.2.0</version>
    <scope>test</scope>
</dependency>
```

由于mockito-core已包含它，因此 我们不需要为更高版本添加mockito-inline。

## 3. 示例设置

接下来，让我们创建一个将在本教程中使用的简单Direction枚举：

```java
public enum Direction {
    NORTH,
    EAST,
    SOUTH,
    WEST;
}
```

此外，让我们定义一个实用方法，该方法使用先前创建的枚举并根据传递的值返回描述：

```java
public static String getDescription(Direction direction) {
    return switch (direction) {
        case NORTH -> "You're headed north.";
        case EAST -> "You're headed east.";
        case SOUTH -> "You're headed south.";
        case WEST -> "You're headed west.";
        default -> throw new IllegalArgumentException();
    };
}
```

## 4. Mock枚举的解决方案

为了检查应用程序的行为，我们可以在测试用例中使用已定义的枚举值。但是，在某些情况下，我们想Mock枚举并使用不存在的值。让我们来解决一些问题。

**我们需要这种Mock的一个例子是确保我们将来可能添加到枚举中的值不会导致意外行为**；另一个例子是争取高百分比的[代码覆盖率](https://www.baeldung.com/cs/code-coverage)。

现在，让我们创建一个测试并Mock一个枚举：

```java
@Test
void givenMockedDirection_whenGetDescription_thenThrowException() {
    try (MockedStatic<Direction> directionMock = Mockito.mockStatic(Direction.class)) {
        Direction unsupported = Mockito.mock(Direction.class);
        Mockito.doReturn(4).when(unsupported).ordinal();

        directionMock.when(Direction::values)
                .thenReturn(new Direction[] { Direction.NORTH, Direction.EAST, Direction.SOUTH,
                        Direction.WEST, unsupported });

        assertThrows(IllegalArgumentException.class, () -> DirectionUtils.getDescription(unsupported));
    }
}
```

这里，我们使用[mockStatic()](https://www.baeldung.com/mockito-mock-static-methods)方法来Mock对静态方法调用的调用，此方法返回Direction类型的MockedStatic对象。接下来，我们定义了一个新的Mock枚举值，并指定了调用ordinal()方法时应该发生的情况。最后，我们将Mock值包含在values()方法返回的结果中。

因此，getDescription()方法会因不支持的值而抛出[IllegalArgumentException](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/lang/IllegalArgumentException.html)。

## 5. 无需Mock枚举的解决方案

接下来，让我们研究如何实现相同的行为，但这次不需要Mock枚举。

首先，让我们在测试目录中定义Direction枚举并添加新的UNKNOWN值：

```java
public enum Direction {
    NORTH,
    EAST,
    SOUTH,
    WEST,
    UNKNOWN;
}
```

**新创建的Direction枚举应该与我们之前在源目录中定义的枚举具有相同的路径**。

此外，由于我们在执行测试用例时使用与源目录中的相同的完全限定名称来定义枚举，因此它们将使用测试源目录中提供的枚举。

此外，**由于我们用与源目录中相同的完全限定名称定义了枚举，因此源目录下的Direction枚举将与测试目录中提供的枚举[重载](https://www.baeldung.com/java-method-overload-override)。因此，测试引擎使用测试目录中提供的枚举**。

让我们测试一下当传递UNKNOWN值时getDescription()方法的行为：

```java
@Test
void givenUnknownDirection_whenGetDescription_thenThrowException() {
    assertThrows(IllegalArgumentException.class, () -> DirectionUtils.getDescription(Direction.UNKNOWN));
}
```

正如预期的那样，对getDescription()方法的调用会引发IllegalArgumentException异常。

**使用此方法的一个可能的缺点是，现在我们拥有的枚举不是源代码的一部分，此外，它会影响我们测试套件中的所有测试**。

## 6. 总结

在本文中，我们学习了如何使用Mockito库Mock枚举，我们还研究了一些可能有用的场景。

总而言之，当我们想要测试引入新值后代码是否会按预期运行时，Mock枚举会很有帮助。我们还学习了如何通过重载现有枚举而不是Mock其值来执行相同操作。