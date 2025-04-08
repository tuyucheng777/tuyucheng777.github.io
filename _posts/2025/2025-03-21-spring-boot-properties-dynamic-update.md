---
layout: post
title:  在运行时更改Spring Boot属性
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

**动态管理应用程序配置是许多实际场景中的关键要求**，在微服务架构中，由于扩展操作或负载条件的变化，不同的服务可能需要动态更改配置。在其他情况下，应用程序可能需要根据用户偏好、来自外部API的数据调整其行为，或遵守动态变化的要求。

[application.properties](https://www.baeldung.com/properties-with-spring#1-applicationproperties-the-default-property-file)文件是静态的，如果不重新启动应用程序就无法生效。但是，Spring Boot提供了几种可靠的方法来在运行时调整配置而无需停机。无论是在实时应用程序中切换功能、更新数据库连接以进行负载均衡，还是在不重新部署应用程序的情况下更改第三方集成的API密钥，Spring Boot的动态配置功能都可以为这些复杂环境提供所需的灵活性。

在本教程中，**我们将探讨几种在Spring Boot应用程序中动态更新属性的策略，而无需直接修改application.properties文件**。这些方法可以满足不同的需求，从非持久性内存更新到使用外部文件的持久性更改。

我们的示例使用了Spring Boot 3.2.4和JDK 17，我们还将使用Spring Cloud 4.1.3。不同版本的Spring Boot可能需要对代码进行细微调整。

## 2. 使用原型作用域的Bean

当我们需要动态调整特定[Bean](https://www.baeldung.com/spring-bean)的属性而不影响已创建的Bean实例或改变全局应用程序状态时，直接注入@Value的简单@Service类是不够的，因为属性在[应用程序上下文](https://www.baeldung.com/spring-application-context)的生命周期内是静态的。

相反，**我们可以使用@Configuration类中的@Bean方法创建具有可修改属性的Bean**，此方法允许在应用程序执行期间动态更改属性：

```java
@Configuration
public class CustomConfig {

    @Bean
    @Scope("prototype")
    public MyService myService(@Value("${custom.property:default}") String property) {
        return new MyService(property);
    }
}
```

通过使用[@Scope("prototype")](https://www.baeldung.com/spring-bean-scopes#prototype)，我们确保每次调用myService(...)时都会创建一个新的MyService实例，从而**允许在运行时进行不同的配置**。在此示例中，MyService是一个最小POJO：

```java
public class MyService {
    private final String property;

    public MyService(String property) {
        this.property = property;
    }

    public String getProperty() {
        return property;
    }
}
```

为了验证动态行为，我们可以使用这些测试：

```java
@Autowired
private ApplicationContext context;

@Test
void whenPropertyInjected_thenServiceUsesCustomProperty() {
    MyService service = context.getBean(MyService.class);
    assertEquals("default", service.getProperty());
}

@Test
void whenPropertyChanged_thenServiceUsesUpdatedProperty() {
    System.setProperty("custom.property", "updated");
    MyService service = context.getBean(MyService.class);
    assertEquals("updated", service.getProperty());
}
```

这种方法使我们能够灵活地在运行时更改配置，而无需重新启动应用程序。更改是临时的，仅影响CustomConfig实例化的Bean。

## 3. 使用Environment，MutablePropertySources和@RefreshScope

与前一种情况不同，我们想要更新已实例化的Bean的属性。为此，**我们将使用Spring Cloud的[@RefreshScope](https://www.javadoc.io/doc/org.springframework.cloud/spring-cloud-commons-parent/1.1.9.RELEASE/org/springframework/cloud/context/scope/refresh/RefreshScope.html)注解以及[/actuator/refresh](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/application-context-services.html#endpoints)端点**。此[Actuator](https://www.baeldung.com/spring-boot-actuator-enable-endpoints)刷新所有@RefreshScope Bean，用反映最新配置的新实例替换旧实例，从而允许实时更新属性而无需重新启动应用程序。同样，这些更改不是持久的。

### 3.1 基本配置

让我们首先将这些依赖项添加到pom.xml：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
    <version>4.1.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>4.1.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>3.2.4</version>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <scope>test</scope>
    <version>4.2.0</version>
</dependency>
```

[spring-cloud-starter](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter)和[spring-cloud-starter-config](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-config)依赖是Spring Cloud框架的一部分，而[spring-boot-starter-actuator](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator)依赖是公开/actuator/refresh端点所必需的。最后，[awaitility](https://mvnrepository.com/artifact/org.awaitility/awaitility)依赖是一个用于处理异步操作的测试实用程序，正如我们将在JUnit 5测试中看到的那样。

现在让我们看一下application.properties，由于在此示例中我们不使用[Spring Cloud Config Server](https://www.baeldung.com/spring-cloud-configuration)来集中跨多个服务的配置，而只需要在单个Spring Boot应用程序中更新属性，**因此我们应该禁用尝试连接到外部配置服务器的默认行为**：

```properties
spring.cloud.config.enabled=false
```

我们仍在使用Spring Cloud功能，只是与分布式客户端-服务器架构不同。如果我们忘记了spring.cloud.config.enabled=false，应用程序将无法启动，并抛出java.lang.IllegalStateException。

然后我们需要启用Spring Boot Actuator端点来公开/actuator/refresh：

```properties
management.endpoint.refresh.enabled=true
management.endpoints.web.exposure.include=refresh
```

此外，如果我们想要在每次调用Actuator时进行记录，我们可以设置以下日志记录级别：

```properties
logging.level.org.springframework.boot.actuate=DEBUG
```

最后，**让我们为测试添加一个示例属性**：

```properties
my.custom.property=defaultValue
```

我们的基本配置已经完成。

### 3.2 示例Bean

当我们将@RefreshScope注解应用于Bean时，Spring Boot不会像平常一样直接实例化该Bean。相反，**它会创建一个代理对象，作为实际Bean的占位符或委托**。

@Value注解将application.properties文件中my.custom.property的值注入到customProperty字段中：

```java
@RefreshScope
@Component
public class ExampleBean {
    @Value("${my.custom.property}")
    private String customProperty;

    public String getCustomProperty() {
        return customProperty;
    }
}
```

代理对象会拦截对此Bean的方法调用，当/actuator/refresh端点触发刷新事件时，代理会使用更新后的配置属性重新初始化该Bean。

### 3.3 PropertyUpdaterService

为了动态更新正在运行的Spring Boot应用程序中的属性，我们可以创建PropertyUpdaterService类，以编程方式添加或更新属性。基本上，它允许我们通过在Spring环境中管理自定义属性源来在运行时注入或修改应用程序属性。

在深入研究代码之前，让我们先澄清一些关键概念：

- [Environment](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/Environment.html)：提供对属性源、Profile和系统环境变量的访问的接口
- [ConfigurableEnvironment](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/ConfigurableEnvironment.html)：Environment的子接口，**允许动态更新应用程序的属性**
- [MutablePropertySources](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/MutablePropertySources.html)：ConfigurableEnvironment持有的[PropertySource](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/PropertySources.html)对象集合，提供添加、删除或重新排序属性源的方法，例如系统属性、环境变量或自定义属性源

各个组件之间关系的UML图可以帮助我们理解动态属性更新如何在应用程序中传播：

![](/assets/images/2025/springboot/springbootpropertiesdynamicupdate01.png)

下面是我们的PropertyUpdaterService，它使用这些组件来动态更新属性：

```java
@Service
public class PropertyUpdaterService {
    private static final String DYNAMIC_PROPERTIES_SOURCE_NAME = "dynamicProperties";

    @Autowired
    private ConfigurableEnvironment environment;

    public void updateProperty(String key, String value) {
        MutablePropertySources propertySources = environment.getPropertySources();
        if (!propertySources.contains(DYNAMIC_PROPERTIES_SOURCE_NAME)) {
            Map<String, Object> dynamicProperties = new HashMap<>();
            dynamicProperties.put(key, value);
            propertySources.addFirst(new MapPropertySource(DYNAMIC_PROPERTIES_SOURCE_NAME, dynamicProperties));
        } else {
            MapPropertySource propertySource = (MapPropertySource) propertySources.get(DYNAMIC_PROPERTIES_SOURCE_NAME);
            propertySource.getSource().put(key, value);
        }
    }
}
```

让我们分解一下：

- updateProperty(...)方法检查MutablePropertySources集合中是否存在名为dynamicProperties的自定义属性源
- 如果没有，它将使用给定的属性创建一个新的MapPropertySource对象，并将其添加为第一个属性源
- **propertySources.addFirst(...)确保我们的动态属性优先于环境中的其他属性**
- 如果dynamicProperties源已存在，则该方法将使用新值更新现有属性，如果键不存在，则添加它

通过使用此服务，我们可以在运行时以编程方式更新应用程序中的任何属性。

### 3.4 使用PropertyUpdaterService的替代策略

虽然直接通过控制器公开属性更新功能对于测试目的来说很方便，但在生产环境中通常并不安全。**使用控制器进行测试时，我们应确保对其进行充分保护，以防止未经授权的访问**。

在生产环境中，有几种安全有效地使用PropertyUpdaterService的替代策略：

- 计划任务 → 属性可能会根据时间敏感条件或来自外部来源的数据而改变
- 基于条件的逻辑 → 响应特定的应用程序事件或触发器，例如负载变化、用户活动或外部API响应
- 限制访问工具 → **仅授权人员可访问的安全管理工具**
- 自定义Actuator端点 → 自定义Actuator可以更好地控制公开的功能，并可以包含额外的安全性
- 应用程序事件监听器 → 在云环境中很有用，在云环境中，实例可能需要调整设置以响应基础设施的变化或应用程序内的其他重要事件

关于内置的/actuator/refresh端点，虽然它会刷新用@RefreshScope标注的Bean，但它不会直接更新属性。我们可以使用PropertyUpdaterService以编程方式添加或修改属性，之后我们可以触发/actuator/refresh以在整个应用程序中应用这些更改。**但是，如果没有PropertyUpdaterService，仅此Actuator就无法更新或添加新属性**。

总之，我们选择的方法应该符合我们的应用程序的具体要求、配置数据的敏感性以及我们的整体安全态势。

### 3.5 使用控制器进行手动测试

在这里我们演示如何使用一个简单的控制器来测试PropertyUpdaterService的功能：

```java
@RestController
@RequestMapping("/properties")
public class PropertyController {
    @Autowired
    private PropertyUpdaterService propertyUpdaterService;

    @Autowired
    private ExampleBean exampleBean;

    @PostMapping("/update")
    public String updateProperty(@RequestParam String key, @RequestParam String value) {
        propertyUpdaterService.updateProperty(key, value);
        return "Property updated. Remember to call the actuator /actuator/refresh";
    }

    @GetMapping("/customProperty")
    public String getCustomProperty() {
        return exampleBean.getCustomProperty();
    }
}
```

使用curl执行手动测试将使我们能够验证我们的实现是否正确：

```shell
$ curl "http://localhost:8080/properties/customProperty"
defaultValue

$ curl -X POST "http://localhost:8080/properties/update?key=my.custom.property&value=tuyuchengValue"
Property updated. Remember to call the actuator /actuator/refresh

$ curl -X POST http://localhost:8080/actuator/refresh -H "Content-Type: application/json"
[]

$ curl "http://localhost:8080/properties/customProperty"
tuyuchengValue
```

它按预期工作。但是，如果第一次尝试没有成功，并且我们的应用程序非常复杂，我们应该再次尝试最后一个命令，以便Spring Cloud有时间更新Bean。

### 3.6 JUnit 5测试

自动化测试当然很有帮助，但并非易事。由于**属性更新操作是异步的，并且没有API来了解它何时完成**，因此我们需要使用超时来避免阻塞JUnit 5。它是异步的，因为对/actuator/refresh的调用会立即返回，而不会等到所有Bean都实际重新创建。

[await语句](https://www.baeldung.com/awaitility-testing)使我们无需使用复杂的逻辑来测试我们感兴趣的Bean的刷新，它使我们能够避免轮询等不太优雅的设计。

最后，要使用RestTemplate，我们需要按照@SpringBootTest(...)注解指定的方式请求启动Web环境：

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PropertyUpdaterServiceUnitTest {
    @Autowired
    private PropertyUpdaterService propertyUpdaterService;

    @Autowired
    private ExampleBean exampleBean;

    @LocalServerPort
    private int port;

    @Test
    @Timeout(5)
    public void whenUpdatingProperty_thenPropertyIsUpdatedAndRefreshed() throws InterruptedException {
        // Injects a new property into the test context
        propertyUpdaterService.updateProperty("my.custom.property", "newValue");

        // Trigger the refresh by calling the actuator endpoint
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<String> entity = new HttpEntity<>(null, headers);
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.postForEntity("http://localhost:" + port + "/actuator/refresh", entity, String.class);

        // Awaitility to wait until the property is updated
        await().atMost(5, TimeUnit.SECONDS).until(() -> "newValue".equals(exampleBean.getCustomProperty()));
    }
}
```

当然，我们需要使用我们感兴趣的所有属性和Bean来定制测试。

## 4. 使用外部配置文件

在某些情况下，**需要[在应用程序部署包之外](https://docs.spring.io/spring-boot/reference/features/external-config.html)管理配置更新，以确保属性的持久更改**。这也使我们能够将更改分发到多个应用程序。

在这种情况下，我们将使用相同的先前的Spring Cloud设置来启用@RefreshScope和/actuator/refresh支持，以及相同的示例控制器和Bean。

**我们的目标是使用外部文件external-config.properties测试ExampleBean上的动态更改**，让我们使用以下内容保存它：

```properties
my.custom.property=externalValue
```

我们可以使用–spring.config.additional-location参数告诉Spring Boot external-config.properties的位置，如Eclipse屏幕截图所示。请记住将示例/path/to/替换为实际路径：

![](/assets/images/2025/springboot/springbootpropertiesdynamicupdate02.png)

让我们验证Spring Boot是否正确加载此外部文件，以及其属性是否覆盖application.properties中的属性：

```shell
$ curl "http://localhost:8080/properties/customProperty"
externalValue
```

它按计划工作，因为external-config.properties中的externalValue替换了application.properties中的defaultValue。现在让我们尝试通过编辑external-config.properties文件来更改此属性的值：

```properties
my.custom.property=external-Tuyucheng-Value
```

像往常一样，我们需要调用Actuator：

```shell
$ curl -X POST http://localhost:8080/actuator/refresh -H "Content-Type: application/json"
["my.custom.property"]
```

最后，结果正如预期的那样，这次是持久化的：

```shell
$ curl "http://localhost:8080/properties/customProperty"
external-Tuyucheng-Value
```

这种方法的一个优点是，**每次修改external-config.properties文件时，我们都可以轻松地自动执行Actuator调用**。为此，我们可以在Linux和macOS上使用跨平台[fswatch](https://github.com/emcrisostomo/fswatch)工具，只需记住将/path/to/替换为实际路径：

```shell
$ fswatch -o /path/to/external-config.properties | while read f; do
    curl -X POST http://localhost:8080/actuator/refresh -H "Content-Type: application/json";
done
```

Windows用户可能会发现基于PowerShell的替代解决方案更加方便，但我们不会讨论这个。

## 5. 总结

在本文中，我们探讨了在Spring Boot应用程序中动态更新属性的各种方法，而无需直接修改application.properties文件。

我们首先讨论了在Bean中使用自定义配置，使用@Configuration、@Bean和@Scope("prototype")注解允许在运行时更改Bean属性而无需重新启动应用程序。此方法可确保灵活性并将更改隔离到Bean的特定实例。

然后，我们研究了Spring Cloud的@RefreshScope和/actuator/refresh端点，以便实时更新已实例化的Bean，并讨论了使用外部配置文件进行持久属性管理。这些方法为动态和集中配置管理提供了强大的选项，增强了我们的Spring Boot应用程序的可维护性和适应性。