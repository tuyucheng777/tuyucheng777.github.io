---
layout: post
title:  Mockito中doAnswer()和thenReturn()之间的区别
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

[Mockito](https://www.baeldung.com/mockito-series)是Java应用程序广泛使用的单元测试框架，它提供了各种API来模拟对象的行为。在本教程中，我们将探讨doAnswer()和thenReturn()存根技术的用法，并进行比较。**我们可以使用这两种API来存根或Mock方法，但在某些情况下，我们只能使用其中一种**。

## 2. 依赖

我们的代码将使用Mockito与JUnit 5结合作为代码示例，并且我们需要在pom.xml文件中添加一些依赖：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
```

可以在Maven Central中找到[JUnit 5 API库](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api)、[JUnit 5 Engine库](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)和[Mockito库](https://mvnrepository.com/artifact/org.mockito/mockito-core)。

## 3. 使用thenReturn()对方法进行存根

**我们可以使用Mockito中的thenReturn()存根技术来存根返回值的方法**。为了演示，我们将使用thenReturn()和doAnswer()测试List的get()和 dd()操作：

```java
public class TaketodayList extends AbstractList<String> {
    @Override
    public String get(final int index) {
        return null;
    }

    @Override
    public void add(int index, String element) {
        // no-op
    }

    @Override
    public int size() {
        return 0;
    }
}
```

在上面的示例代码中，get()方法返回一个字符串。首先，我们将使用thenReturn()对get()方法进行存根，并通过断言其返回值与存根方法相同来验证调用：

```java
@Test
void givenThenReturn_whenGetCalled_thenValue() {
    TaketodayList myList = mock(TaketodayList.class);

    when(myList.get(anyInt()))
        .thenReturn("answer me");

    assertEquals("answer me", myList.get(1));
}
```

### 3.1 使用thenReturn()返回多个值的存根方法

除此之外，thenReturn() API允许在连续调用中返回不同的值，我们可以链接其调用以返回多个值。此外，我们可以在单个方法调用中传递多个值：

```java
@Test
void givenThenReturn_whenGetCalled_thenReturnChaining() {
    TaketodayList myList = mock(TaketodayList.class);

    when(myList.get(anyInt()))
        .thenReturn("answer one")
        .thenReturn("answer two");

    assertEquals("answer one", myList.get(1));
    assertEquals("answer two", myList.get(1));
}

@Test
void givenThenReturn_whenGetCalled_thenMultipleValues() {
    TaketodayList myList = mock(TaketodayList.class);

    when(myList.get(anyInt()))
        .thenReturn("answer one", "answer two");

    assertEquals("answer one", myList.get(1));
    assertEquals("answer two", myList.get(1));
}
```

## 4. 使用doAnswer()对void方法进行存根

add()方法是void方法，不返回任何内容。我们不能使用thenReturn()来存根add()方法，因为thenReturn()存根不能与void方法一起使用。**相反，我们将使用doAnswer()，因为它允许存根void方法**。因此，我们将使用doAnswer()来存根add()方法，并且在调用add()方法时将调用存根中提供的Answer：

```java
@Test
void givenDoAnswer_whenAddCalled_thenAnswered() {
    TaketodayList myList = mock(TaketodayList.class);

    doAnswer(invocation -> {
        Object index = invocation.getArgument(0);
        Object element = invocation.getArgument(1);

        // verify the invocation is called with the correct index and element
        assertEquals(3, index);
        assertEquals("answer", element);

        // return null as this is a void method
        return null;
    }).when(myList)
      . add(any(Integer.class), any(String.class));
    myList.add(3, "answer");
}
```

在doAnswer()中，我们验证对add()方法的调用是否被调用，并且我们断言调用它的参数是否符合预期。

### 4.1 使用doAnswer()对非void方法进行存根

由于我们可以使用返回值而不是null的Answer来存根方法，因此我们可以使用doAnswer()方法来存根非void方法。例如，我们将通过使用doAnswer()来存根get()方法并返回一个返回String的Answer来测试该方法：

```java
@Test
void givenDoAnswer_whenGetCalled_thenAnswered() {
    TaketodayList myList = mock(TaketodayList.class);

    doAnswer(invocation -> {
        Object index = invocation.getArgument(0);

        // verify the invocation is called with the index
        assertEquals(1, index);

        // return the value we want 
        return "answer me";
    }).when(myList)
        .get(any(Integer.class));

    assertEquals("answer me", myList.get(1));
}
```

### 4.2 使用doAnswer()返回多个值的存根方法

我们必须注意，在doAnswer()方法中我们只能返回一个Answer。但是，我们可以在doAnswer()方法中放置条件逻辑，根据调用收到的参数返回不同的值。因此，在下面的示例代码中，我们将根据调用get()方法的索引返回不同的值：

```java
@Test
void givenDoAnswer_whenGetCalled_thenAnsweredConditionally() {
    TaketodayList myList = mock(TaketodayList.class);

    doAnswer(invocation -> {
        Integer index = invocation.getArgument(0);
        return switch (index) {
            case 1 -> "answer one";
            case 2 -> "answer two";
            default -> "answer " + index;
        };
    }).when(myList)
        .get(anyInt());

    assertEquals("answer one", myList.get(1));
    assertEquals("answer two", myList.get(2));
    assertEquals("answer 3", myList.get(3));
}
```

## 5. 总结

Mockito框架提供了许多存根/Mock技术，例如doAnswer()、doReturn()、thenReturn()、thenAnswer()等，以方便各种类型和风格的Java代码及其测试。我们已经使用doAnswer()和thenReturn()来存根非void方法并执行类似的测试。**但是，我们只能使用doAnswer()来存根void方法，因为thenReturn()方法无法执行此功能**。