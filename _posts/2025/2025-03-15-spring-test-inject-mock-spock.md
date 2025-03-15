---
layout: post
title:  在Spock Spring测试中将Mock作为Spring Bean注入
category: unittest
copyright: unittest
excerpt: Spock
---

## 1. 简介

当我们使用[Spock](https://www.baeldung.com/groovy-spock)测试[Spring应用程序](https://www.baeldung.com/spring-spock-testing)时，有时我们想要更改Spring管理组件的行为。在本教程中，**我们将学习如何注入我们自己的[Stub、Mock或Spy](https://www.baeldung.com/spock-stub-mock-spy)来代替Spring自动注入的依赖项**。我们将在大多数示例中使用Spock Stub，但在使用Mock或Spy时也适用相同的技术。

## 2. 设置

### 2.1 依赖

首先，让我们为Spring Boot 3添加[Maven依赖](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>3.3.0</version>
</dependency>

```

现在，让我们为[spring-boot-starter-test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)和[spock-spring](https://mvnrepository.com/artifact/org.spockframework/spock-spring)添加Maven依赖，**由于我们使用的是Spring Boot 3/Spring 6，因此需要Spock v2.4-M1或更高版本才能获得Spock兼容的Spring注解**，因此我们使用2.4-M4-groovy-4.0：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>3.3.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-spring</artifactId>
    <version>2.4-M4-groovy-4.0</version>
    <scope>test</scope>
</dependency>
```

### 2.2 对象

有了依赖后，让我们创建一个具有Spring管理的DataProvider依赖项的AccountService类，可用于我们的测试。

首先，让我们创建AccountService：

```java
@Service
public class AccountService {
    private final DataProvider provider;

    public AccountService(DataProvider provider) {
        this.provider = provider;
    }

    public String getData(String param) {
        return "Fetched: " + provider.fetchData(param);
    }
}
```

现在，让我们创建一个DataProvider，稍后我们将替换它：

```java
@Component
public class DataProvider {
    public String fetchData(final String input) {
        return "data for " + input;
    }
}
```

## 3. 基础测试类

现在，让我们创建一个基本的测试类，使用它的常用组件来验证我们的AccountService。

我们将使用Spring的@ContextConfiguration将AccountService和DataProvider纳入范围并在我们的两个类中自动装配：

```groovy
@ContextConfiguration(classes = [AccountService, DataProvider])
class AccountServiceTest extends Specification {
    @Autowired
    DataProvider dataProvider

    @Autowired
    @Subject
    AccountService accountService

    def "given a real data provider when we use the real bean then we get our usual response"() {
        when: "we fetch our data"
        def result = accountService.getData("Something")

        then: "our real dataProvider responds"
        result == "Fetched: data for Something"
    }
}
```

## 4. 使用Spock的Spring注解

现在我们已经有了基本的测试，让我们探索一下存根对象依赖项的选项。

### 4.1 @StubBeans注解

我们经常在编写测试时希望删除依赖项，而不关心自定义其响应。**我们可以使用Spock的@StubBeans注解为类中的每个依赖项创建Stub**。

因此，让我们创建一个用Spock的@StubBeans注解标注的测试规范来存根我们的DataProvider类：

```groovy
@StubBeans(DataProvider)
@ContextConfiguration(classes = [AccountService, DataProvider])
class AccountServiceStubBeansTest extends Specification {
    @Autowired
    @Subject
    AccountService accountService
    // ... 
}
```

请注意，我们不需要为DataProvider声明单独的Stub，因为StubBeans注解已经为我们创建了一个。

当我们的AccountService的getData方法调用其fetchData方法时，我们生成的Stub将返回一个空字符串。让我们创建一个测试来断言：

```groovy
def "given a Service with a dependency when we use a @StubBeans annotation then a stub is created and injected to the service"() {
    when: "we fetch our data"
    def result = accountService.getData("Something")

    then: "our StubBeans gave us an empty string response from our DataProvider dependency"
    result == "Fetched: "
}
```

我们生成的DataProvider存根从fetchData返回了一个空字符串，导致我们的AccountService的getData返回“Fetched:”，没有任何附加内容。

当我们想要存根多个依赖项时，我们使用带有Groovy的list[]语法的StubBeans：

```groovy
@StubBeans([DataProvider, MySecondDependency, MyThirdDependency])
```

### 4.2 @SpringBean注解

当我们需要自定义响应时，我们会创建一个Stub、Mock或Spy。**因此，让我们在测试中使用Spock的SpringBean注解，而不是StubBeans注解，用Spock Stub替换我们的DataProvider**：

```groovy
@SpringBean
DataProvider mockProvider = Stub()
```

请注意，我们不能将SpringBean声明为def或Object，我们需要声明一个特定类型，如DataProvider。

现在，让我们创建一个AccountServiceSpringBeanTest规范，其中包含一个测试，该测试设置存根在调用其fetchData方法时返回“42”：

```groovy
@ContextConfiguration(classes = [AccountService, DataProvider])
class AccountServiceSpringBeanTest extends Specification {
    // ...
    def "given a Service with a dependency when we use a @SpringBean annotation then our stub is injected to the service"() {
        given: "a stubbed response"
        mockProvider.fetchData(_ as String) >> "42"

        when: "we fetch our data"
        def result = accountService.getData("Something")

        then: "our SpringBean overrode the original dependency"
        result == "Fetched: 42"
    }
}
```

@SpringBean注解确保我们的Stub被注入到AccountService中，这样我们就能获得存根“42”响应。即使上下文中存在真正的DataProvider，我们的@SpringBean注解的Stub也能胜出。

### 4.3 @SpringSpy注解

**有时，我们需要Spy来访问真实对象并修改其某些响应。因此，让我们使用Spock的SpringSpy注解将我们的DataProvider包装为Spock Spy**：

```groovy
@SpringSpy
DataProvider mockProvider
```

首先，让我们创建一个测试，验证我们监视的对象的fetchData方法是否被调用并返回了真正的“data for Something”响应：

```groovy
@ContextConfiguration(classes = [AccountService, DataProvider])
class AccountServiceSpringSpyTest extends Specification {
    @SpringSpy
    DataProvider dataProvider

    @Autowired
    @Subject
    AccountService accountService

    def "given a Service with a dependency when we use @SpringSpy and override a method then the original result is returned"() {
        when: "we fetch our data"
        def result = accountService.getData("Something")

        then: "our SpringSpy was invoked once and allowed the real method to return the result"
        1  dataProvider.fetchData(_)
        result == "Fetched: data for Something"
    }
}
```

@SpringSpy注解将Spy包裹在自动连接的DataProvider周围，并确保我们的Spy被注入到AccountService中。我们的Spy验证了我们的DataProvider的fetchData方法被调用而不改变其结果。

现在让我们添加一个测试，其中我们的Spy用“spied”覆盖结果：

```groovy
def "given a Service with a dependency when we use @SpringSpy and override a method then our spy's result is returned"() {
    when: "we fetch our data"
    def result = accountService.getData("Something")

    then: "our SpringSpy was invoked once and overrode the original method"
    1  dataProvider.fetchData(_) >> "spied"
    result == "Fetched: spied"
}
```

这一次，我们注入的Spy Bean验证了我们的DataProvider的fetchData方法被调用，并将其响应替换为“spied”。

### 4.4 @SpringBoot测试中的@SpringBean

现在我们已经在@ContextConfiguration测试中看到了SpringBean注解，让我们创建另一个测试类，但使用@SpringBootTest：

```groovy
@SpringBootTest
class AccountServiceSpringBootTest extends Specification {
    // ...
}
```

我们新类中的测试与我们在AccountServiceSpringBeanTest中创建的测试相同，因此我们不会在这里重复！

但是，如果没有SpringBootApplication，我们的@SpringBootTest测试类就无法运行，所以让我们创建一个TestApplication类：

```groovy
@SpringBootApplication
class TestApplication {
    static void main(String[] args) {
        SpringApplication.run(TestApplication, args)
    }
}
```

当我们运行测试时，Spring Boot尝试初始化DataSource。在我们的例子中，由于我们只使用spring-boot-starter，我们没有DataSource，所以我们的测试无法初始化：

```text
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
```

因此，让我们通过在SpringBootTest注解中添加spring.autoconfigure.exclude属性来排除DataSource自动配置：

```groovy
@SpringBootTest(properties = ["spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration"])
```

现在，当我们运行测试时，它成功运行！

### 4.5 上下文缓存

使用@SpringBean注解时，我们应该注意它对Spring测试框架上下文缓存的影响。通常，当我们运行使用@SpringBootTest注解的多个测试时，我们的Spring上下文会被缓存，而不是每次都创建，这使我们的测试运行得更快。

**但是，Spock的@SpringBean会将Mock附加到特定测试实例，类似于spring-boot-test模块中的@MockBean注解。这可以防止上下文缓存，当我们有大量使用它们的测试时，上下文缓存会减慢我们的整体测试执行速度**。

## 5. 总结

在本教程中，我们学习了如何使用Spock的@StubBeans注解来存根Spring管理的依赖项。接下来，我们学习了如何使用Spock的@SpringBean或@SpringSpy注解将依赖项替换为Stub、Mock或Spy。

最后，我们注意到过度使用@SpringBean注解会干扰Spring测试框架的上下文缓存，从而减慢测试的整体执行速度。因此，我们应该谨慎使用此功能！