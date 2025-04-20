---
layout: post
title:  Java中的猴子补丁
category: designpattern
copyright: designpattern
excerpt: 猴子补丁
---

## 1. 简介

在软件开发中，我们经常需要调整和增强系统的现有功能。有时，修改现有代码库可能不可行，或者并非最实用的解决方案。**猴子补丁可以解决这个问题，这项技术允许我们在不更改原始源代码的情况下修改类或模块的运行时**。

在本教程中，我们将探讨如何在Java中使用猴子补丁、何时使用它以及它的缺点。

## 2. 猴子补丁

“猴子补丁”(Monkey Patching)一词源于早期术语“游击补丁”(Guerrilla Patching)，指的是在运行时偷偷地更改代码，不受任何规则的限制。由于Java、Python和Ruby等动态编程语言的灵活性，它变得流行起来。

猴子补丁使我们能够在运行时修改或扩展类或模块，这使我们能够调整或扩充现有代码，而无需直接修改源代码。当调整势在必行，**但由于各种限制而无法或不适宜直接修改时，猴子补丁尤其有用**。

在Java中，可以通过多种技术实现猴子补丁，这些方法包括代理、字节码检测、面向切面编程、反射和装饰器模式。每种方法都有其独特的方法，适用于特定的场景。

现在让我们创建一个简单的货币转换器，其中硬编码了欧元到美元的汇率，以便使用不同的方法应用猴子修补：

```java
public interface MoneyConverter {
    double convertEURtoUSD(double amount);
}
```

```java
public class MoneyConverterImpl implements MoneyConverter {
    private final double conversionRate;

    public MoneyConverterImpl() {
        this.conversionRate = 1.10;
    }

    @Override
    public double convertEURtoUSD(double amount) {
        return amount * conversionRate;
    }
}
```

## 3. 动态代理

在Java中，使用代理是实现猴子补丁的强大技术。**[代理](https://refactoring.guru/design-patterns/proxy)是一个包装器，它通过自身功能传递方法调用**，这为我们提供了修改或增强原始类行为的机会。

值得注意的是，[动态代理](https://www.baeldung.com/java-dynamic-proxies)是Java中一种基本的代理机制，并且被Spring框架等框架广泛使用。

**一个很好的例子是@Transactional注解**，当应用于方法时，关联的类会在运行时进行动态代理包装。调用该方法时，Spring会将调用重定向到代理。之后，代理会启动新的事务或加入现有事务，随后，实际方法会被调用。需要注意的是，为了能够从这种事务行为中受益，我们需要依赖Spring的依赖注入机制，因为它基于动态代理。

让我们使用动态代理来包装我们的转换方法，并为我们的货币转换器添加一些日志。首先，我们必须创建一个java.lang.reflect.InvocationHandler的子类型：

```java
public class LoggingInvocationHandler implements InvocationHandler {
    private final Object target;

    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method: " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After method: " + method.getName());
        return result;
    }
}
```

接下来，我们将创建一个测试来验证日志是否围绕转换方法：

```java
@Test
public void whenMethodCalled_thenSurroundedByLogs() {
    ByteArrayOutputStream logOutputStream = new ByteArrayOutputStream();
    System.setOut(new PrintStream(logOutputStream));
    MoneyConverter moneyConverter = new MoneyConverterImpl();
    MoneyConverter proxy = (MoneyConverter) Proxy.newProxyInstance(
            MoneyConverter.class.getClassLoader(),
            new Class[]{MoneyConverter.class},
            new LoggingInvocationHandler(moneyConverter)
    );

    double result = proxy.convertEURtoUSD(10);

    Assertions.assertEquals(11, result);
    String logOutput = logOutputStream.toString();
    assertTrue(logOutput.contains("Before method: convertEURtoUSD"));
    assertTrue(logOutput.contains("After method: convertEURtoUSD"));
}
```

## 4. 面向切面编程

**面向切面编程(AOP)是一种解决软件开发中横切关注点问题的范式**，它提供了一种模块化且具有内聚性的方法来分离原本分散在整个代码库中的关注点。AOP的实现方式是，在现有代码中添加额外的行为，而无需修改代码本身。

在Java中，我们可以通过[AspectJ](https://www.baeldung.com/aspectj)或[Spring AOP](https://www.baeldung.com/spring-aop)等框架来利用AOP，Spring AOP提供了一种轻量级且与Spring集成的方法，而AspectJ则提供了一个更强大、更独立的解决方案。

在猴子补丁中，AOP提供了一种优雅的解决方案，允许我们以集中的方式将更改应用于多个类或方法。**使用切面，我们可以解决诸如日志记录或安全策略之类的问题，这些问题需要在各个组件之间保持一致，而无需更改核心逻辑**。

让我们尝试用相同的日志包围相同的方法，为此，我们将使用[AspectJ](https://eclipse.dev/aspectj/)框架，并且需要在项目中添加spring-boot-starter-aop依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
    <version>3.2.2</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-aop)上找到该库的最新版本。

在Spring AOP中，切面通常应用于Spring管理的Bean。因此，为了简单起见，我们将货币转换器定义为Bean：

```java
@Bean
public MoneyConverter moneyConverter() {
    return new MoneyConverterImpl();
}
```

现在我们需要定义我们的切面，用日志包围我们的转换方法：

```java
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* cn.tuyucheng.taketoday.monkey.patching.converter.MoneyConverter.convertEURtoUSD(..))")
    public void beforeConvertEURtoUSD(JoinPoint joinPoint) {
        System.out.println("Before method: " + joinPoint.getSignature().getName());
    }

    @After("execution(* cn.tuyucheng.taketoday.monkey.patching.converter.MoneyConverter.convertEURtoUSD(..))")
    public void afterConvertEURtoUSD(JoinPoint joinPoint) {
        System.out.println("After method: " + joinPoint.getSignature().getName());
    }
}
```

然后我们可以创建一个测试来验证我们的切面是否正确应用：

```java
@Test
public void whenMethodCalled_thenSurroundedByLogs() {
    ByteArrayOutputStream logOutputStream = new ByteArrayOutputStream();
    System.setOut(new PrintStream(logOutputStream));

    double result = moneyConverter.convertEURtoUSD(10);

    Assertions.assertEquals(11, result);
    String logOutput = logOutputStream.toString();
    assertTrue(logOutput.contains("Before method: convertEURtoUSD"));
    assertTrue(logOutput.contains("After method: convertEURtoUSD"));
}
```

## 5. 装饰器模式

**[装饰器](https://refactoring.guru/design-patterns/decorator)是一种设计模式，它允许我们将对象放置在包装器对象中，从而为其附加行为**。因此，我们可以假设装饰器为原始对象提供了增强的接口。

在猴子补丁的语境下，它提供了一种灵活的解决方案，用于在不直接修改类代码的情况下增强或修改类的行为。我们可以创建装饰器类，使其实现与原始类相同的接口，并通过包装基类的实例来引入额外的功能。

这种模式在处理一组共享接口的相关类时尤其有用，通过使用[装饰器模式](https://www.baeldung.com/java-decorator-pattern)，可以选择性地应用修改，从而以模块化和非侵入式的方式调整或扩展单个对象的功能。

**与其他猴子补丁技术相比，装饰器模式提供了一种更结构化、更明确的方法来增强对象行为**，它的多功能性使其非常适合需要清晰的关注点分离和模块化代码修改的场景。

为了实现此模式，我们将创建一个新类来实现MoneyConverter接口，它将具有一个MoneyConverter类型的属性，该属性将处理请求，我们装饰器的目的只是添加一些日志并转发货币转换请求：

```java
public class MoneyConverterDecorator implements MoneyConverter {
    private final MoneyConverter moneyConverter;

    public MoneyConverterDecorator(MoneyConverter moneyConverter) {
        this.moneyConverter = moneyConverter;
    }

    @Override
    public double convertEURtoUSD(double amount) {
        System.out.println("Before method: convertEURtoUSD");
        double result = moneyConverter.convertEURtoUSD(amount);
        System.out.println("After method: convertEURtoUSD");
        return result;
    }
}
```

现在让我们创建一个测试来检查日志是否已添加：

```java
@Test
public void whenMethodCalled_thenSurroundedByLogs() {
    ByteArrayOutputStream logOutputStream = new ByteArrayOutputStream();
    System.setOut(new PrintStream(logOutputStream));
    MoneyConverter moneyConverter = new MoneyConverterDecorator(new MoneyConverterImpl());

    double result = moneyConverter.convertEURtoUSD(10);

    Assertions.assertEquals(11, result);
    String logOutput = logOutputStream.toString();
    assertTrue(logOutput.contains("Before method: convertEURtoUSD"));
    assertTrue(logOutput.contains("After method: convertEURtoUSD"));
}
```

## 6. 反射

**反射是程序在运行时检查和修改其行为的能力**，在Java中，我们可以借助[java.lang.reflect](https://www.baeldung.com/java-reflection)包或[Reflections库](https://www.baeldung.com/reflections-library)来使用它。虽然它提供了显著的灵活性，但我们[应该谨慎使用它](https://www.baeldung.com/java-reflection-benefits-drawbacks)，因为它可能会影响代码的可维护性和性能。

反射在猴子补丁中的常见应用包括访问类元数据、检查字段和方法，甚至在运行时调用方法。因此，此功能开启了无需直接修改源代码即可进行运行时修改的大门。

假设转换率已更新为新值，我们无法更改它，因为我们没有为转换器类创建Setter方法，而且它是硬编码的。相反，我们可以使用反射来打破封装，并将转换率更新为新值：

```java
@Test
public void givenPrivateField_whenUsingReflection_thenBehaviorCanBeChanged() throws IllegalAccessException, NoSuchFieldException {
    MoneyConverter moneyConvertor = new MoneyConverterImpl();

    Field conversionRate = MoneyConverterImpl.class.getDeclaredField("conversionRate");
    conversionRate.setAccessible(true);
    conversionRate.set(moneyConvertor, 1.2);
    double result = moneyConvertor.convertEURtoUSD(10);

    assertEquals(12, result);
}
```

## 7. 字节码检测

**通过字节码插装，我们可以动态修改已编译类的字节码**。一个流行的字节码插装框架是[Java Instrumentation API](https://www.baeldung.com/java-instrumentation)，引入此API的目的是收集数据以供各种工具使用。由于这些修改完全是附加的，因此这些工具不会改变应用程序的状态或行为。这些工具的示例包括监控代理、分析器、覆盖率分析器和事件记录器。

**然而，值得注意的是，这种方法引入了更高级别的复杂性**，并且由于它可能对我们的应用程序的运行时行为产生影响，因此必须小心处理。

## 8. 猴子补丁的用例

猴子补丁在各种情况下都非常有用，在这些情况下，对代码进行运行时修改是一种实用的解决方案。**一个常见的用例是修复第三方库或框架中的紧急错误，而无需等待官方更新**，它使我们能够通过临时修补代码来快速解决一些问题。

**另一种情况是在直接修改代码具有挑战性或不切实际的情况下，扩展或修改现有类或方法的行为**。此外，在测试环境中，猴子补丁对于引入模拟行为或临时更改功能以模拟不同场景非常有用。

此外，**当我们需要快速进行原型设计或实验时，我们可以使用“猴子补丁”技术**，这使我们能够快速迭代并探索各种实现方式，而无需进行永久性的更改。

## 9. 猴子补丁的风险

尽管猴子补丁很实用，但它也带来了一些风险，我们应该仔细考虑。潜在的副作用和冲突是一个重大风险，因为**运行时所做的修改可能会产生不可预测的交互**。此外，这种可预测性的缺乏可能会导致调试场景的挑战性，并增加维护开销。

此外，**猴子补丁可能会损害代码的可读性和可维护性**。动态注入更改可能会掩盖代码的实际行为，使我们难以理解和维护，尤其是在大型项目中。

猴子补丁也可能引发安全问题，因为**它可能引入漏洞或恶意行为**。此外，**对猴子补丁的依赖可能会阻碍采用标准编码实践和系统性问题解决方案**，从而导致代码库的健壮性和凝聚力下降。

## 10. 总结

在本文中，我们了解到猴子补丁在某些情况下可能非常有用且有效。它也可以通过各种技术来实现，每种技术都有其优缺点。然而，这种方法应该谨慎使用，因为它可能导致性能、可读性、可维护性和安全性问题。