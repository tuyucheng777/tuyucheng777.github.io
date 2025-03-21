---
layout: post
title:  在Spring Boot应用程序中设置默认时区
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

有时，我们希望能够指定应用程序使用的TimeZone。对于全球运行的服务，这可能意味着所有服务器都使用相同的TimeZone发布事件，无论它们位于何处。

我们可以通过几种不同的方式实现这一点，一种方法是在执行应用程序时使用JVM参数，另一种方法是在启动生命周期的各个阶段以编程方式在代码中进行更改。

在本简短教程中，我们将介绍几种设置Spring Boot应用程序默认时区的方法。首先，我们将了解如何通过命令行实现此操作，然后，我们将深入研究在Spring Boot代码启动时以编程方式执行此操作的一些选项。最后，我们将研究这些方法之间的差异。

## 2. 主要概念

[TimeZone](https://www.baeldung.com/java-jvm-time-zone)的默认值基于运行JVM的计算机的操作系统，我们可以更改它：

- 通过传递JVM参数，使用user.timezone参数，根据我们运行任务还是JAR以不同的方式运行
- 以编程方式，使用Bean生命周期配置选项(在创建Bean期间/之前)或者甚至在类内部，在执行期间

**在Spring Boot应用程序中设置默认时区会影响不同的组件**，例如日志的时间戳、调度程序、JPA/Hibernate时间戳等，这意味着我们选择在哪里执行此操作取决于我们何时需要它生效。例如，我们是希望在某个Bean创建期间还是在WebApplicationContext初始化之后执行它？

必须准确设置此值，因为这可能会导致应用程序出现不良行为。例如，闹钟服务可能会在时区更改生效之前设置闹钟，这可能会导致闹钟在错误的时间激活。

在决定采用哪种方案之前，需要考虑的另一个因素是可测试性。使用JVM参数是更简单的选择，但测试它可能更棘手且更容易出错。无法保证单元测试将使用与生产部署相同的JVM参数运行。

## 3. 在bootRun任务上设置默认时区

如果我们使用bootRun任务来运行应用程序，我们可以使用[命令行中的JVM参数](https://www.baeldung.com/spring-boot-command-line-arguments)传递默认的TimeZone。在这种情况下，我们设置的值从执行一开始就可用：

```shell
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Duser.timezone=Europe/Athens"
```

## 4. 在JAR执行时设置默认时区

与运行bootRun任务类似，我们可以在执行JAR文件时在命令行中传递默认的TimeZone值。同样，我们设置的值从执行一开始就可用：

```shell
java -Duser.timezone=Europe/Athens -jar spring-core-4-1.0.0.jar
```

## 5. 在Spring Boot启动时设置默认时区

让我们看看如何在Spring启动过程的不同部分插入时区变化。

### 5.1 Main方法

首先，假设我们在main方法中设置该值。在这种情况下，**我们可以在执行的早期阶段使用它，甚至在检测到Spring Profile之前就可以使用它**：

```java
@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        TimeZone.setDefault(TimeZone.getTimeZone("GMT+08:00"));

        SpringApplication.run(MainApplication.class, args);
    }
}
```

虽然这是生命周期的第一步，但它并没有充分利用Spring配置带来的可能性。我们要么必须对时区进行硬编码，要么以编程方式从环境变量等中读取时区。

### 5.2 BeanFactoryPostProcessor

其次，[BeanFactoryPostProcessor](https://courses.baeldung.com/courses/1288526/lectures/29538523)是一个工厂钩子，我们可以使用它来修改应用程序上下文的Bean定义。这样，我们就**可以在任何Bean实例化发生之前设置值**：

```java
@Component
public class GlobalTimezoneBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        TimeZone.setDefault(TimeZone.getTimeZone("GMT+08:00"));
    }
}
```

### 5.3 PostConstruct

最后，我们可以在WebApplicationContext初始化完成后立即使用MainApplication类的[PostConstruct](https://www.baeldung.com/spring-postconstruct-predestroy)设置默认的TimeZone值。此时，**我们可以从配置属性中注入TimeZone值**：

```java
@SpringBootApplication
public class MainApplication {

    @Value("${application.timezone:UTC}")
    private String applicationTimeZone;

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }

    @PostConstruct
    public void executeAfterMain() {
        TimeZone.setDefault(TimeZone.getTimeZone(applicationTimeZone));
    }
}
```

## 6. 总结

在本简短教程中，我们学习了在Spring Boot应用程序中设置默认时区的几种方法。我们讨论了更改默认值可能产生的影响，并基于这些因素，我们应该能够根据我们的用例决定正确的方法。