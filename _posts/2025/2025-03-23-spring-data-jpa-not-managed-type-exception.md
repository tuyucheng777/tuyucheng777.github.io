---
layout: post
title:  Spring Data JPA 中的“Not a Managed Type”异常
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

使用[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)时，我们可能会在引导期间遇到问题。某些Bean可能无法创建，导致应用程序无法启动。虽然实际的堆栈跟踪可能会有所不同，但通常看起来像这样：

```text
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'requestMappingHandlerAdapter'
...
Caused by: java.lang.IllegalArgumentException: Not a managed type: ...OurEntity
	at org.hibernate.metamodel.internal.MetamodelImpl.managedType(MetamodelImpl.java:583)
	at org.hibernate.metamodel.internal.MetamodelImpl.managedType(MetamodelImpl.java:85)
...
```

根本原因是“Not a Managed Type”异常。**在本文中，我们将深入研究此异常的可能原因并探索其解决方案**。

## 2. 缺少@Entity注解

**出现此异常的一个可能原因是我们可能忘记使用[@Entity注解](https://www.baeldung.com/jpa-entities#entity)标记我们的实体**。

### 2.1 重现问题

假设我们有以下实体类：

```java
public class EntityWithoutAnnotation {
    @Id
    private Long id;
}
```

我们还有它的Spring Data JPA Repository：

```java
public interface EntityWithoutAnnotationRepository extends JpaRepository<EntityWithoutAnnotation, Long> {
}
```

最后，我们有扫描上面提到的所有类的应用程序类：

```java
@SpringBootApplication
public class EntityWithoutAnnotationApplication {
}
```

让我们尝试使用此应用程序来引导Spring上下文：

```java
@Test
void givenEntityWithoutAnnotationApplication_whenBootstrap_thenExpectedExceptionThrown() {
    Exception exception = assertThrows(Exception.class,
            () -> SpringApplication.run(EntityWithoutAnnotationApplication.class));

    assertThat(exception)
            .getRootCause()
            .hasMessageContaining("Not a managed type");
}
```

正如预期的那样，我们遇到了与我们的实体相关的“Not a Managed Type”异常。

### 2.2 修复问题

让我们将@Entity注解添加到实体的修复版本中：

```java
@Entity
public class EntityWithoutAnnotationFixed {
    @Id
    private Long id;
}
```

应用程序和Repository类保持不变，让我们尝试再次启动应用程序：

```java
@Test
void givenEntityWithoutAnnotationApplicationFixed_whenBootstrap_thenRepositoryBeanShouldBePresentInContext() {
    ConfigurableApplicationContext context = run(EntityWithoutAnnotationFixedApplication.class);
    EntityWithoutAnnotationFixedRepository repository = context
            .getBean(EntityWithoutAnnotationFixedRepository.class);

    assertThat(repository).isNotNull();
}
```

我们成功检索了ConfigurableApplicationContext实例并从那里获取了Repository实例。

## 3. 从javax.persistence迁移到jakarta.persistence

**将我们的应用程序迁移到[Jakarta Persistence API](https://mvnrepository.com/artifact/jakarta.persistence/jakarta.persistence-api)时，我们可能会遇到另一种这种异常的情况**。

### 3.1 重现问题

假设我们有下一个实体：

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class EntityWithJakartaAnnotation {
    @Id
    private Long id;
}
```

这里我们开始使用jakarta.persistence包，但我们仍然使用Spring Boot 2，我们将以类似于上一节中创建的方式创建Repository和应用程序类。

现在，让我们尝试启动应用程序：

```java
@Test
void givenEntityWithJakartaAnnotationApplication_whenBootstrap_thenExpectedExceptionThrown() {
    Exception exception = assertThrows(Exception.class,
            () -> run(EntityWithJakartaAnnotationApplication.class));

    assertThat(exception)
            .getRootCause()
            .hasMessageContaining("Not a managed type");
}
```

我们再次遇到“Not a Managed Type”的问题，**我们的JPA实体扫描器要求我们使用javax.persistence.Entity注解，而不是jakarta.persistence.Entity**。

### 3.2 修复问题

在这种情况下，我们有两个潜在的解决方案。我们可以[迁移到Spring Boot 3](https://www.baeldung.com/spring-boot-3-migration)，并且Spring Data JPA开始使用jakarta.persistence。或者，如果我们尚未准备好迁移，我们可以继续使用javax.persistence.Entity。

## 4. 遗漏或错误配置@EntityScan

我们可能会遇到“Not a Managed Type”异常的另一种常见情况是**当我们的JPA实体扫描器无法在预期路径中找到实体时**。

### 4.1 重现问题

首先，我们创建另一个实体：

```java
package cn.tuyucheng.taketoday.spring.entity;
@Entity
public class CorrectEntity {
    @Id
    private Long id;
}
```

它有@Entity注解，并放置在entity包中。现在，让我们创建一个Repository：

```java
package cn.tuyucheng.taketoday.spring.repository;

public interface CorrectEntityRepository extends JpaRepository<CorrectEntity, Long> {
}
```

我们的Repository放在repository包中。最后，我们创建一个应用程序类：

```java
package cn.tuyucheng.taketoday.spring.app;

@SpringBootApplication
@EnableJpaRepositories(basePackages = "cn.tuyucheng.taketoday.spring.repository")
public class WrongEntityScanApplication {
}
```

它位于app包中，默认情况下，Spring Data会扫描主类包及其子包下的Repository。因此，我们需要使用[@EnableJpaRepositories](https://www.baeldung.com/spring-data-annotations#5-enablejparepositories)指定基本Repository包。现在，让我们尝试引导应用程序：

```java
@Test
void givenWrongEntityScanApplication_whenBootstrap_thenExpectedExceptionThrown() {
    Exception exception = assertThrows(Exception.class,
            () -> run(WrongEntityScanApplication.class));

    assertThat(exception)
            .getRootCause()
            .hasMessageContaining("Not a managed type");
}
```

我们再次遇到“Not a Managed Type”异常，原因是实体扫描的逻辑与Repository扫描的逻辑相同。**我们扫描了app包下的包，但没有找到任何实体，并且在CorrectEntityRepository构造期间收到异常**。


### 4.2 修复问题

为了解决这个问题，我们可以使用[@EntityScan](https://www.baeldung.com/spring-entityscan-vs-componentscan#the-entityscan-annotation)注解。

让我们创建另一个应用程序类：

```java
@SpringBootApplication
@EnableJpaRepositories(basePackages = "cn.tuyucheng.taketoday.spring.repository")
@EntityScan("cn.tuyucheng.taketoday.spring.entity")
public class WrongEntityScanFixedApplication {
}
```

现在我们使用@EnableJpaRepositories注解指定Repository包，使用@EntityScan注解指定实体包。让我们看看它是如何工作的：

```java
@Test
void givenWrongEntityScanApplicationFixed_whenBootstrap_thenRepositoryBeanShouldBePresentInContext() {
    SpringApplication app = new SpringApplication(WrongEntityScanFixedApplication.class);
    app.setAdditionalProfiles("test");
    ConfigurableApplicationContext context = app.run();
    CorrectEntityRepository repository = context
            .getBean(CorrectEntityRepository.class);
    assertThat(repository).isNotNull();
}
```

我们成功启动了应用程序，我们从上下文中检索了CorrectEntityRepository，这意味着它已构建，并且CorrectEntity已成功识别为JPA实体。

## 5. 总结

在本教程中，我们探讨了使用Spring Data JPA时为什么会遇到“Not a Managed Type”异常。我们还学习了避免该异常的解决方案。解决方案的选择取决于具体情况，但了解可能的原因有助于我们识别所面临的确切变体。