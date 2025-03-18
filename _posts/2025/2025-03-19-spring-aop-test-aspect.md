---
layout: post
title:  如何测试Spring AOP切面
category: spring
copyright: spring
excerpt: Spring AOP
---

## 1. 概述

[面向切面编程(AOP)](https://www.baeldung.com/spring-aop)通过将横切关注点从主应用程序逻辑中分离为基本单元(称为切面)，改进了程序设计。Spring AOP是一个可帮助我们轻松实现切面的框架。

AOP切面与其他软件组件没有什么不同，它们需要不同的测试来验证其正确性。在本教程中，我们将学习如何对Spring AOP切面进行单元测试和集成测试。

## 2. 什么是AOP？

AOP是一种编程范式，是对[面向对象编程(OOP)](https://www.baeldung.com/java-oop)的补充，用于模块化横切关注点，这些关注点是跨越主应用程序的功能。**类是OOP中的基本单元，而切面是AOP中的基本单元**。日志记录和事务管理是横切关注点的典型示例。

一个切面由两个组件组成，一个是定义横切关注点逻辑的通知，另一个是指定在应用程序执行期间何时应用逻辑的切入点。

下表概述了常见的AOP术语：

|  术语   |             描述             |
|:-----:|:--------------------------:|
|  关注点  |         应用程序的特定功能          |
| 跨领域关注 |      跨越应用程序多个部分的特定功能       |
|  切面   | AOP基本单元，包含用于实现横切关注点的通知和切入点 |
|  通知   |     我们希望在横切关注点中调用的特定逻辑     |
|  切入点  |      选择将应用通知的连接点的表达式       |
|  连接点  |       应用程序的执行点，例如方法        |

## 3. 执行时间记录

在本节中，让我们创建一个示例切面，记录连接点周围的执行时间。

### 3.1 Maven依赖

**有不同的Java AOP框架，例如[Spring AOP和AspectJ](https://www.baeldung.com/spring-aop-vs-aspectj)**。在本教程中，我们将使用Spring AOP并在pom.xml中包含以下[依赖项](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-aop)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
    <version>3.2.5</version>
</dependency>
```

对于日志记录部分，我们选择[SLF4J](https://www.baeldung.com/slf4j-log-exceptions)作为API，并选择SLF4J简单提供程序作为日志记录实现。SLF4J是一个在不同的[日志记录实现](https://www.baeldung.com/java-logging-intro)之间提供统一API的门面。

因此，我们也在pom.xml中包含了[SLF4J API](https://mvnrepository.com/artifact/org.slf4j/slf4j-api)和[SLF4J简单提供程序](https://mvnrepository.com/artifact/org.slf4j/slf4j-simple)依赖项：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.13</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>2.0.13</version>
</dependency>
```

### 3.2 执行时间切面

我们的ExecutionTimeAspect类很简单，只包含一个通知logExecutionTime()，**我们用@Aspect和@Component标注我们的类以将其声明为一个切面并启用Spring来管理它**：

```java
@Aspect
@Component
public class ExecutionTimeAspect {
    private Logger log = LoggerFactory.getLogger(ExecutionTimeAspect.class);

    @Around("execution(* cn.tuyucheng.taketoday.unittest.ArraySorting.sort(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long t = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        log.info("Execution time=" + (System.currentTimeMillis() - t) + "ms");
        return result;
    }
}
```

**[@Around](https://www.baeldung.com/spring-aspect-oriented-programming-logging#1-using-around-advice)注解指示通知logExecutionTime()围绕[切入点表达式](https://www.baeldung.com/spring-aop-pointcut-tutorial)execution(...)定义的目标连接点运行**。在Spring AOP中，连接点始终是一种方法。

## 4. 切面的单元测试

**从单元测试的角度来看，我们仅测试切面内的逻辑，没有任何依赖项，包括Spring应用程序上下文**。在此示例中，我们使用Mockito Mock joinPoint和记录器，然后[将Mock注入](https://www.baeldung.com/junit-5-mock-captor-method-parameter-injection)到我们的测试切面。

单元测试类用@ExtendsWith(MockitoExtension.class)标注，以启用JUnit 5的Mockito功能，它会自动初始化Mock并将其注入到用@InjectMocks标注的测试单元中：

```java
@ExtendWith(MockitoExtension.class)
class ExecutionTimeAspectUnitTest {
    @Mock
    private ProceedingJoinPoint joinPoint;

    @Mock
    private Logger logger;

    @InjectMocks
    private ExecutionTimeAspect aspect;

    @Test
    void whenExecuteJoinPoint_thenLoggerInfoIsCalled() throws Throwable {
        when(joinPoint.proceed()).thenReturn(null);
        aspect.logExecutionTime(joinPoint);
        verify(joinPoint, times(1)).proceed();
        verify(logger, times(1)).info(anyString());
    }
}
```

在这个测试用例中，我们期望切面中的joinPoint.proceed()方法被调用一次。此外，记录器info()方法也应该被调用一次来记录执行时间。

为了更准确地验证日志消息，我们可以使用[ArgumentCaptor](https://www.baeldung.com/mockito-argumentcaptor)类来捕获日志消息，这使我们能够断言以“Execution time=”开头生成的消息：

```java
ArgumentCaptor<String> argumentCaptor = ArgumentCaptor.forClass(String.class);
verify(logger, times(1)).info(argumentCaptor.capture());
assertThat(argumentCaptor.getValue()).startsWith("Execution time=");
```

## 5. 基于切面的集成测试

从集成测试的角度来看，我们需要类实现ArraySorting来通过切入点表达式将我们的通知应用于目标类。sort()方法只是调用静态方法Collections.sort()对列表进行排序：

```java
@Component
public class ArraySorting {
    public <T extends Comparable<? super T>> void sort(List<T> list) {
        Collections.sort(list);
    }
}
```

你可能会想：为什么我们不将我们的通知应用于Collections.sort()静态方法呢？**Spring AOP的一个限制就是它不适用于静态方法**。Spring AOP创建动态代理来拦截方法调用，此机制需要调用目标对象上的实际方法，而静态方法可以在没有对象的情况下调用。**如果我们需要拦截静态方法，我们必须采用另一个支持编译时织入的AOP框架，例如AspectJ**。

**在集成测试中，我们需要Spring应用程序上下文来创建代理对象，以拦截目标方法并应用通知**。我们使用[@SpringBootTest](https://www.baeldung.com/springrunner-vs-springboottest#springboottest)标注集成测试类，以加载启用AOP和依赖注入功能的应用程序上下文：

```java
@SpringBootTest
class ExecutionTimeAspectIntegrationTest {
    @Autowired
    private ArraySorting arraySorting;

    private List<Integer> getRandomNumberList(int size) {
        List<Integer> numberList = new ArrayList<>();
        for (int n=0;n<size;n++) {
            numberList.add((int) Math.round(Math.random() * size));
        }
        return numberList;
    }

    @Test
    void whenSort_thenExecutionTimeIsPrinted() {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        PrintStream originalSystemOut = System.out;
        System.setOut(new PrintStream(baos));

        arraySorting.sort(getRandomNumberList(10000));

        System.setOut(originalSystemOut);
        String logOutput = baos.toString();
        assertThat(logOutput).contains("Execution time=");
    }
}
```

测试方法可以分为三个部分。首先，它将输出流重定向到专用缓冲区，以便稍后进行断言。随后，它调用sort()方法，该方法调用切面中的通知。重要的是通过@Autowired注入ArraySorting实例，而不是使用new ArraySorting()实例化实例。这可确保在目标类上激活Spring AOP。最后，它断言缓冲区中是否存在日志。

## 6. 总结

在本文中，我们讨论了AOP的基本概念，并了解了如何在目标类上使用Spring AOP切面。我们还研究了使用单元测试和集成测试来测试切面，以验证切面的正确性。