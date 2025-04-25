---
layout: post
title:  Avaje Inject简介
category: libraries
copyright: libraries
excerpt: Avaje Inject
---

## 1. 简介

[Avaje Inject](https://avaje.io/inject/)是由Ebean的创建者开发的，它是一个基于JVM的高级编译时依赖注入(DI)框架，它采用生成可读源代码的方式，以支持各种DI操作。Avaje读取JSR-330注解的Bean，并生成类来从我们的应用程序中收集Bean实例，并在适当的时候使用它们。

## 2. 依赖注入

提醒一下，[依赖注入](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)是控制反转(其中程序控制自己的流程)原则的具体应用。

不同的框架实现依赖注入的方式不同，其中最显著的区别之一就是注入发生在运行时还是编译时。

运行时DI通常基于反射，它使用简单，但运行时速度较慢。Spring是[运行时DI框架](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)的典范。

另一方面，编译时DI通常基于代码生成，这意味着所有重量级操作都在编译阶段执行，它通常执行速度更快，因为它**无需在运行时进行密集的反射或类路径扫描**。

Avaje就是完全依赖注解处理来生成编译时源。

## 3. Maven/Gradle配置

要在项目中使用Avaje Inject，**我们需要将[avaje-inject依赖](https://mvnrepository.com/artifact/io.avaje/avaje-inject)添加到pom.xml中**：

```xml
<dependency>
    <groupId>io.avaje</groupId>
    <artifactId>avaje-inject</artifactId>
    <version>9.5</version>
</dependency>
<dependency>
    <groupId>io.avaje</groupId>
    <artifactId>avaje-inject-test</artifactId>
    <version>9.5</version>
    <scope>test</scope>
</dependency>
```

此外，我们还需要包含[注入代码生成器](https://mvnrepository.com/artifact/io.avaje/avaje-inject-generator)，以便读取带注解的类并生成用于注入的代码，之所以添加[optional作用域](https://www.baeldung.com/maven-optional-dependency#:~:text=How%20to%20Use,make%20any%20Maven%20dependency%20optional.&text=In%20this%20example%2C%20although%20optional,optional%3E%20tag%20was%20never%20there.)，是因为该生成器仅在编译时才需要：

```xml
<dependency> 
    <groupId>io.avaje</groupId>
    <artifactId>avaje-inject-generator</artifactId>
    <version>9.5</version>
    <scope>provided</scope>
    <optional>true</optional>
</dependency>
```

如果使用Gradle，我们将把这些依赖包含为：

```groovy
implementation('io.avaje:avaje-inject:9.5')
annotationProcessor('io.avaje:avaje-inject-generator:9.5')

testImplementation('io.avaje:avaje-inject-test:9.5')
testAnnotationProcessor('io.avaje:avaje-inject-generator:9.5')
```

现在我们的项目中有了avaje-inject，让我们创建一个示例应用程序来查看它是如何工作的。

## 4. 实现

在我们的示例中，我们将尝试通过将骑士的武器作为依赖注入来构建一个骑士，avaje-inject生成器将在编译时读取所有各种注解，并在target/generated-sources/annotations中生成DI代码。

### 4.1 @Singleton和@Inject

与Spring的@Component类似，**Avaje使用标准JSR-330注解@Singleton将类标记为Bean**。为了注入依赖，我们将@Inject注解添加到字段或构造函数中，生成的DI代码位于同一包中，因此要使用字段/方法注入，这些元素必须是公共的或包私有的。

让我们看一下Knight类，它有两个武器依赖：

```java
@Singleton
public class Knight {

    private Sword sword;

    private Shield shield;

    @Inject
    public Knight(Sword sword, Shield shield) {
        this.sword = sword;
        this.shield = shield;
    }
    //standard getters and setters
}
```

**代码生成器将读取注解并生成一个类来收集依赖并注册Bean**：

```java
@Generated("io.avaje.inject.generator")
public final class Knight$DI  {

    public static void build(Builder builder) {
        if (builder.isAddBeanFor(Knight.class)) {
            var bean = new Knight(builder.get(Sword.class,"!sword"), builder.get(Shield.class,"!shield"));
            builder.register(bean);
        }
    }
}
```

此类检查现有的Knight类，然后从当前范围检索依赖。

### 4.2 @Factory和@Bean

与Spring的@Configuration类一样，**我们可以用@Factory标注一个类，用@Bean标注其方法，以将该类标记为创建Bean的工厂**。

在ArmsFactory类中，我们使用它来向应用程序范围提供骑士的武器：

```java
@Factory
public class ArmsFactory {

    @Bean
    public Sword provideSword() {
        return new Sword();
    }

    @Bean
    public Brand provideShield() {
        return new Shield(25);
    }
}
```

代码生成器将读取注解并生成一个类来调用构造函数和工厂方法：

```java
@Generated("io.avaje.inject.generator")
public final class ArmsFactory$DI  {

    public static void build(Builder builder) {
        if (builder.isAddBeanFor(ArmsFactory.class)) {
            var bean = new ArmsFactory();
            builder.register(bean);
        }
    }

    public static void build_provideEngine(Builder builder) {
        if (builder.isAddBeanFor(Sword.class)) {
            var factory = builder.get(ArmsFactory.class);
            var bean = factory.provideEngine();
            builder.register(bean);
        }
    }

    public static void build_provideBrand(Builder builder) {
        if (builder.isAddBeanFor(Shield.class)) {
            var factory = builder.get(ArmsFactory.class);
            var bean = factory.provideBrand();
            builder.register(bean);
        }
    }
}
```

### 4.3 @PostConstruct和@PreDestroy

Avaje可以使用生命周期方法将自定义操作附加到Bean的创建和销毁，**@PostConstruct方法在BeanScope完成所有Bean的注入后执行，而@Predestroy方法在BeanScope关闭时运行**。

鉴于@PostConstruct方法将在所有Bean注入完成后执行，我们可以添加一个BeanScope参数，以便使用已完成的BeanScope进行进一步配置，下面的Ninja类使用@PostConstruct将其成员设置为来自应用程序范围的Bean：

```java
@Singleton
public class Ninja {

    private Sword sword;

    @PostConstruct
    void equip(BeanScope scope) {
        sword = scope.get(Sword.class);
    }

    @PreDestroy
    void dequip() {
        sword = null;
    }

    //getters/setters
}
```

代码生成器将读取注解并生成一个类来调用构造函数并注册生命周期方法：

```java
@Generated("io.avaje.inject.generator")
public final class Ninja$DI  {

    public static void build(Builder builder) {
        if (builder.isAddBeanFor(Ninja.class)) {
            var bean = new Ninja();
            var $bean = builder.register(bean);
            builder.addPostConstruct($bean::equip);
            builder.addPreDestroy($bean::dequip);
        }
    }
}
```

## 5. 生成的模块

在编译时，avaje-inject-generator会读取所有Bean定义并确定所有Bean的注入方式，然后，它会生成一个Module类，用于表示我们的应用程序及其依赖。

对于上述所有类，都会生成下面的IntroModule来执行所有要添加到应用程序范围的注入，**我们可以看到应用程序中所有Bean的定义和装配顺序**：

```java
@Generated("io.avaje.inject.generator")
@InjectModule()
public final class IntroModule implements Module {

    private Builder builder;

    @Override
    public Class<?>[] classes() {
        return new Class<?>[]{
                cn.tuyucheng.taketoday.avaje.intro.ArmsFactory.class,
                cn.tuyucheng.taketoday.avaje.intro.Knight.class,
                cn.tuyucheng.taketoday.avaje.intro.Ninja.class,
                cn.tuyucheng.taketoday.avaje.intro.Shield.class,
                cn.tuyucheng.taketoday.avaje.intro.Sword.class,
        };
    }

    /**
     * Creates all the beans in order based on constructor dependencies.
     * The beans are registered into the builder along with callbacks for
     * field/method injection, and lifecycle support.
     */
    @Override
    public void build(Builder builder) {
        this.builder = builder;
        build_intro_ArmsFactory();
        build_intro_Ninja();
        build_intro_Sword();
        build_intro_Shield();
        build_intro_Knight();
    }

    @DependencyMeta(type = "cn.tuyucheng.taketoday.avaje.intro.ArmsFactory")
    private void build_intro_ArmsFactory() {
        ArmsFactory$DI.build(builder);
    }

    @DependencyMeta(type = "cn.tuyucheng.taketoday.avaje.intro.Ninja")
    private void build_intro_Ninja() {
        Ninja$DI.build(builder);
    }

    @DependencyMeta(
            type = "cn.tuyucheng.taketoday.avaje.intro.Sword",
            method = "cn.tuyucheng.taketoday.avaje.intro.ArmsFactory$DI.build_provideSword",
            dependsOn = {"cn.tuyucheng.taketoday.avaje.intro.ArmsFactory"})
    private void build_intro_Sword() {
        ArmsFactory$DI.build_provideSword(builder);
    }

    @DependencyMeta(
            type = "cn.tuyucheng.taketoday.avaje.intro.Shield",
            method = "cn.tuyucheng.taketoday.avaje.intro.ArmsFactory$DI.build_provideShield",
            dependsOn = {"cn.tuyucheng.taketoday.avaje.intro.ArmsFactory"})
    private void build_intro_Shield() {
        ArmsFactory$DI.build_provideShield(builder);
    }

    @DependencyMeta(
            type = "cn.tuyucheng.taketoday.avaje.intro.Knight",
            dependsOn = {
                    "cn.tuyucheng.taketoday.avaje.intro.Sword",
                    "cn.tuyucheng.taketoday.avaje.intro.Shield"
            })
    private void build_intro_Knight() {
        Knight$DI.build(builder);
    }
}
```

## 6. 使用BeanScope检索Bean

为了管理依赖关系，**Avaje的BeanScope会加载并执行应用程序及其依赖中生成的所有模块类**，并存储创建的Bean以供稍后检索。

让我们构建BeanScope并接收我们的Knight类，配备Sword和Shield：

```java
final var scope = BeanScope.builder().build();
final var knight = scope.get(Knight.class);

assertNotNull(knight);
assertNotNull(knight.sword());
assertNotNull(knight.shield());
assertEquals(25, knight.shield().defense());
```

## 7. 使用@InjectTest进行组件测试 

**当我们需要为测试引导一个Bean作用域时，@InjectTest注解非常有用**，该注解通过创建一个将被使用的测试BeanScope来实现。

我们可以使用Mockito的@Mock注解将Mock对象添加到测试的BeanScope中，当我们在字段上使用@Mock注解时，该Mock对象将被注入到该字段中，并在测试作用域中注册，该Mock对象将替换作用域中任何现有的同类型的Bean。

如果未定义相同类型的Bean，则会添加一个新Bean，这对于需要Mock特定Bean(例如外部服务)的组件测试非常有用。

**在这里，我们使用注入的Shield Mock来存根防御方法**。然后，我们使用@Inject从测试范围中获取骑士Bean，以验证它是否包含Mock的盾牌：

```java
@InjectTest
class ExampleInjectTest {

    @Mock Shield shield;

    @Inject Knight knight;

    @Test
    void givenMockedShield_whenGetShield_thenShieldShouldHaveMockedValue() {

        Mockito.when(shield.defense()).thenReturn(0);
        assertNotNull(knight);
        assertNotNull(knight.sword());
        assertEquals(knight.shield(), shield);
        assertEquals(0, knight.shield().defense());
    }
}
```

## 8. 总结

在本文中，我们通过一个基本示例介绍了如何设置和使用Avaje Inject，我们了解了它如何使用代码生成来执行各种DI操作，以及如何创建测试和使用模拟。