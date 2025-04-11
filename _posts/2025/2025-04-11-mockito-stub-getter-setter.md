---
layout: post
title:  在Mockito中存根Getter和Setter
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 简介

虽然[Mockito](https://www.baeldung.com/mockito-series)提供了一种很好的方法来避免初始化我们不想初始化的对象，但有时它的开箱即用功能是有限的。

在本教程中，我们将探讨在[单元测试](https://www.baeldung.com/junit)上下文中对Setter和Getter进行存根的各种方法。

## 2. Mock对象与实际对象

在编写任何测试之前，让我们先了解一下[存根和Mock之间的区别](https://www.baeldung.com/cs/faking-mocking-stubbing#:~:text=A%20Stub%20is%20a%20lightweight,the%20flexibility%20of%20a%20Mock.)，**我们使用术语“存根”来描述不提供任何行为灵活性的对象。相反，Mock允许可配置行为并提供验证功能**。

为了帮助我们查看与真实场景有些相似的示例，ExampleService类调用其他对象的方法(无论是否存根)：

```java
public class ExampleService {

    public <T> T getField(Supplier<T> getter) {
        return getter.get();
    }

    public <T> void setField(Consumer<T> setter, T value) {
        setter.accept(value);
    }
}
```

ExampleService的实现考虑到了可重用性，因此getField()和setField()方法的参数选择都不同。简而言之，getField()方法会调用它接收的任何Supplier，因此在我们的例子中是Getter方法。相反，setField()方法会使用提供的值调用它接收的任何Consumer(在我们的例子中是Setter方法)。

为了清楚起见，让我们使用ExampleService编写一些Getter和Setter调用的示例：

```java
exampleService.getField(() -> fooBar.getFoo()); // invokes getFoo getter
exampleService.getField(fooBar::getBar); // invokes getFoo getter
exampleService.setField((bar) -> fooBar.setBar(bar), "newBar"); // invokes bar setter
exampleService.setField(fooBar::setBar, "newBar"); // invokes bar setter
```

开始编写测试之前的最后一步是定义我们将使用的模型，即SimpleClass：

```java
public class SimpleClass {

    private Long id;
    private String name;

    // getters, setters, constructors
}
```

首先，Mock一个相对轻量且易于初始化的对象通常不是最佳选择。事实上，我们需要编写的Mock对象代码比仅仅创建该对象的实例要长得多。以下代码片段包含一个Mock对象，其中包含存根的Setter和Getter方法以及末尾的验证：

```java
@Test
public void givenMockedSimpleClass_whenInvokingSettersGetters_thenInvokeMockedSettersGetters() {
    Long mockId = 12L;
    String mockName = "I'm 12";
    SimpleClass simpleMock = mock(SimpleClass.class);
    when(simpleMock.getId()).thenReturn(mockId);
    when(simpleMock.getName()).thenReturn(mockName);
    doNothing().when(simpleMock).setId(anyLong());
    doNothing().when(simpleMock).setName(anyString());
    ExampleService srv = new ExampleService();
    srv.setField(simpleMock::setId, 11L);
    srv.setField(simpleMock::setName, "I'm 11");
    assertEquals(srv.getField(simpleMock::getId), mockId);
    assertEquals(srv.getField(simpleMock::getName), mockName);
    verify(simpleMock).getId();
    verify(simpleMock).getName();
    verify(simpleMock).setId(eq(11L));
    verify(simpleMock).setName(eq("I'm 11"));
}
```

为了更清楚地说明问题，下面是不使用Mock的相同测试：

```java
@Test
public void givenActualSimpleClass_whenInvokingSettersGetters_thenInvokeActualSettersGetters() {
    Long id = 1L;
    String name = "I'm 1";
    SimpleClass simple = new SimpleClass(id, name);
    ExampleService srv = new ExampleService();
    srv.setField(simple::setId, 2L);
    srv.setField(simple::setName, "I'm 2");
    assertEquals(srv.getField(simple::getId), simple.getId());
    assertEquals(srv.getField(simple::getName), simple.getName());
}
```

比较这两个测试用例，可以明显看出Mock版本多出了8行，通常，Mock存根的设置和验证会导致更长的测试用例。

## 3. 简单的Mock

**使用Mockito存根的最常见情况是，当创建对象需要多行代码或其初始化速度很慢导致测试套件性能不佳时**。方便的是，Mockito的when()和thenReturn()方法提供了一种避免创建真实对象的方法，在本例中，我们需要一个比SimpleClass更复杂的对象，因此让我们引入NonSimpleClass：

```java
public class NonSimpleClass {

    private Long id;
    private String name;
    private String superComplicatedField;

    // getters, setters, constructors
}
```

顾名思义，superComplicatedField需要特殊处理。因此，我们必须在测试期间不要初始化它：

```java
@Test
public void givenNonSimpleClass_whenInvokingGetName_thenReturnMockedName() {
    NonSimpleClass nonSimple = mock(NonSimpleClass.class);
    when(nonSimple.getName()).thenReturn("Meredith");
    ExampleService srv = new ExampleService();
    assertEquals(srv.getField(nonSimple::getName), "Meredith");
    verify(nonSimple).getName();
}
```

在这种情况下，创建NonSimpleClass的实例会导致性能问题或不必要的代码。相反，Mockito存根可以达到所需的测试覆盖率，而不会引入与NonSimpleClass实例化相关的任何负面影响。

## 4. 状态Mock

Mockito不提供开箱即用的对象状态管理功能，**在我们的例子中，除非我们进行一些特殊处理，否则Setter设置的值不会在Getter的后续调用中返回**。对于这个问题，一个可行的解决方案是使用状态Mock，这样当在Setter之后调用Getter时，Getter将返回上次设置的值。Wrapper类，顾名思义，就是用来包装值的，负责状态管理：

```java
class Wrapper<T> {

    private T value;

    // getter, setter, constructors
}
```

为了利用Wrapper类，我们使用doAnswer()和thenAnswer()来代替thenReturn()和doNothing() Mockito方法，这些变体允许我们访问Mock方法的参数并应用自定义逻辑，而不是简单地返回静态值：

```java
@Test
public void givenNonSimpleClass_whenInvokingGetName_thenReturnTheLatestNameSet() {
    Wrapper<String> nameWrapper = new Wrapper<>(String.class);
    NonSimpleClass nonSimple = mock(NonSimpleClass.class);
    when(nonSimple.getName()).thenAnswer((Answer<String>) invocationOnMock -> nameWrapper.get());
    doAnswer(invocation -> {
        nameWrapper.set(invocation.getArgument(0));
        return null;
    }).when(nonSimple)
        .setName(anyString());
    ExampleService srv = new ExampleService();
    srv.setField(nonSimple::setName, "John");
    assertEquals(srv.getField(nonSimple::getName), "John");
    srv.setField(nonSimple::setName, "Nick");
    assertEquals(srv.getField(nonSimple::getName), "Nick");
}
```

我们的有状态Mock使用底层包装器实例来保留调用Setter时设置的最新值，并在调用Mock的Getter时返回最后设置的值。

## 5. 总结

在这篇简短的文章中，我们讨论了各种Mock场景，并讨论了何时使用Mock方便，何时不方便。此外，我们展示了一个有状态Mock的解决方案，这也展示了Mockito库的多功能性，使我们能够在需要时Mock相当复杂的场景。