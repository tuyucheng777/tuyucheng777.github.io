---
layout: post
title:  CDI可移植扩展和Flyway
category: libraries
copyright: libraries
excerpt: CDI
---

## 1. 概述

在本教程中，我们将介绍CDI(上下文和依赖注入)的一个有趣功能，称为CDI可移植扩展。

首先，我们将了解它的工作原理，然后学习如何编写扩展。我们将逐步介绍如何为Flyway实现CDI集成模块，以便在CDI容器启动时运行数据库迁移。

本教程假设你对CDI有基本的了解。请参阅[这篇文章](https://www.baeldung.com/java-ee-cdi)，了解CDI的简介。

## 2. 什么是CDI可移植扩展？

CDI可移植扩展是一种机制，通过它我们可以在CDI容器之上实现额外的功能。在启动时，CDI容器会扫描类路径并创建关于已发现类的元数据。

**在此扫描过程中，CDI容器会触发许多初始化事件，这些事件只有扩展才能观察到**，这就是CDI可移植扩展发挥作用的地方。

**CDI可移植扩展观察这些事件，然后修改或向容器创建的元数据添加信息**。

## 3. Maven依赖

让我们首先在pom.xml中添加CDI API所需的依赖，这足以实现一个空的扩展。

```xml
<dependency>
    <groupId>javax.enterprise</groupId>
    <artifactId>cdi-api</artifactId>
    <version>2.0.SP1</version>
</dependency>
```

为了运行应用程序，我们可以使用任何兼容的CDI实现。在本文中，我们将使用Weld实现。

```xml
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se-core</artifactId>
    <version>3.0.5.Final</version>
    <scope>runtime</scope>
</dependency>
```

你可以在Maven Central检查是否有任何新版本的[API](https://mvnrepository.com/artifact/javax.enterprise/cdi-api)和[实现](https://mvnrepository.com/artifact/org.jboss.weld.se/weld-se-core)已发布。

## 4. 在非CDI环境中运行Flyway

在开始集成Flyway和CDI之前，我们应该首先了解如何在非CDI环境中运行它。

让我们看一下来自[Flyway官方网站](https://flywaydb.org/)的以下示例：

```java
DataSource dataSource = //...
Flyway flyway = new Flyway();
flyway.setDataSource(dataSource);
flyway.migrate();
```

我们可以看到，我们只使用了一个需要DataSource实例的Flyway实例。

我们的CDI可移植扩展稍后将生成Flyway和Datasource Bean，在本示例中，我们将使用嵌入式H2数据库，并通过DataSourceDefinition注解提供DataSource属性。

## 5. CDI容器初始化事件

在应用程序引导时，CDI容器首先加载并实例化所有CDI可移植扩展。然后，它会在每个扩展中搜索并注册初始化事件的观察者方法(如果有)，之后，它会执行以下步骤：

1. 在扫描过程开始之前触发BeforeBeanDiscovery事件
2. 执行类型发现，扫描存档Bean，并针对每个发现的类型触发ProcessAnnotatedType事件
3. 触发AfterTypeDiscovery事件
4. 执行Bean发现
5. 触发AfterBeanDiscovery事件
6. 执行Bean验证并检测定义错误
7. 触发AfterDeploymentValidation事件

CDI可移植扩展的目的是观察这些事件，检查有关发现的Bean的元数据，修改这些元数据或向其中添加内容。

**在CDI可移植扩展中，我们只能观察这些事件**。

## 6. 编写CDI可移植扩展

让我们看看如何通过构建我们自己的CDI可移植扩展来挂接其中一些事件。

### 6.1 实现SPI提供程序

CDI可移植扩展是接口javax.enterprise.inject.spi.Extension的Java SPI提供程序，请参阅[本文](https://www.baeldung.com/java-spi)，了解Java SPI的介绍。

首先，我们先提供扩展实现。之后，我们将向CDI容器引导事件添加观察者方法：

```java
public class FlywayExtension implements Extension {
}
```

然后，我们添加一个名为META-INF/services/javax.enterprise.inject.spi.Extension的文件名，内容如下：

```properties
cn.tuyucheng.taketoday.cdi.extension.FlywayExtension
```

作为SPI，此扩展在容器引导之前加载，因此，可以注册CDI引导事件的观察者方法。

### 6.2 定义初始化事件的观察者方法

在此示例中，我们在扫描过程开始之前将Flyway类告知CDI容器，这是在registerFlywayType()观察者方法中完成的：

```java
public void registerFlywayType(@Observes BeforeBeanDiscovery bbdEvent) {
    bbdEvent.addAnnotatedType(Flyway.class, Flyway.class.getName());
}
```

在这里，我们添加了有关Flyway类的元数据。**从现在开始，它的行为就像被容器扫描过一样**。为此，我们使用了addAnnotatedType()方法。

接下来，我们将观察ProcessAnnotatedType事件，以使Flyway类成为CDI托管Bean：

```java
public void processAnnotatedType(@Observes ProcessAnnotatedType<Flyway> patEvent) {
    patEvent.configureAnnotatedType()
            .add(ApplicationScoped.Literal.INSTANCE)
            .add(new AnnotationLiteral<FlywayType>() {})
            .filterMethods(annotatedMethod -> {
                return annotatedMethod.getParameters().size() == 1
                        && annotatedMethod.getParameters().get(0).getBaseType()
                        .equals(javax.sql.DataSource.class);
            }).findFirst().get().add(InjectLiteral.INSTANCE);
}
```

首先，我们用@ApplicationScoped和@FlywayType注解标注Flyway类，然后我们搜索Flyway.setDataSource(DataSource dataSource)方法并用@Inject标注它。

上述操作的最终结果与容器扫描以下Flyway Bean的效果相同：

```java
@ApplicationScoped
@FlywayType
public class Flyway {

    // ...
    @Inject
    public void setDataSource(DataSource dataSource) {
        // ...
    }
}
```

下一步是使DataSource Bean可供注入，因为我们的Flyway Bean依赖于DataSource Bean。

为此，我们将处理将DataSource Bean注册到容器中，并使用AfterBeanDiscovery事件：

```java
void afterBeanDiscovery(@Observes AfterBeanDiscovery abdEvent, BeanManager bm) {
    abdEvent.addBean()
            .types(javax.sql.DataSource.class, DataSource.class)
            .qualifiers(new AnnotationLiteral<Default>() {}, new AnnotationLiteral<Any>() {})
            .scope(ApplicationScoped.class)
            .name(DataSource.class.getName())
            .beanClass(DataSource.class)
            .createWith(creationalContext -> {
                DataSource instance = new DataSource();
                instance.setUrl(dataSourceDefinition.url());
                instance.setDriverClassName(dataSourceDefinition.className());
                return instance;
            });
}
```

正如我们所见，我们需要一个提供DataSource属性的DataSourceDefinition。

我们可以使用以下注解来注解任何托管Bean：

```java
@DataSourceDefinition(
    name = "ds", 
    className = "org.h2.Driver", 
    url = "jdbc:h2:mem:testdb")
```

为了提取这些属性，我们观察ProcessAnnotatedType事件以及@WithAnnotations注解：

```java
public void detectDataSourceDefinition(@Observes @WithAnnotations(DataSourceDefinition.class) ProcessAnnotatedType<?> patEvent) {
    AnnotatedType at = patEvent.getAnnotatedType();
    dataSourceDefinition = at.getAnnotation(DataSourceDefinition.class);
}
```

最后，我们监听AfterDeploymentValidation事件以从CDI容器中获取所需的Flyway Bean，然后调用migration()方法：

```java
void runFlywayMigration(@Observes AfterDeploymentValidation adv, BeanManager manager) {
    Flyway flyway = manager.createInstance()
        .select(Flyway.class, new AnnotationLiteral<FlywayType>() {}).get();
    flyway.migrate();
}
```

## 7. 总结

构建CDI可移植扩展一开始看起来很困难，但是一旦我们了解了容器初始化生命周期和专用于扩展的SPI，它就会成为一个非常强大的工具，我们可以用它在Jakarta EE之上构建框架。