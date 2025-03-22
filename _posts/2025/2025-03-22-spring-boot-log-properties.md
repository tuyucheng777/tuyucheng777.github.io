---
layout: post
title:  Spring Boot应用程序中记录属性
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

属性是Spring Boot提供的最有用的机制之一，它们可以从各种地方提供，例如专用属性文件、环境变量等。因此，有时查找和记录特定属性很有用，例如在调试时。

在这个简短的教程中，我们将看到在Spring Boot应用程序中查找和记录属性的几种不同方法。

首先，我们将创建一个简单的测试应用程序。然后，我们将尝试三种不同的方式来记录特定属性。

## 2. 创建测试应用程序

让我们创建一个具有三个自定义属性的简单应用程序。

我们可以使用[Spring Initializr](https://start.spring.io/)创建Spring Boot应用模板。我们将使用Java作为语言，可以自由选择其他选项，例如Java版本、项目元数据等。

下一步是向我们的应用添加自定义属性，我们将这些属性添加到src/main/resources中的新application.properties文件中：

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application
bael.property=stagingValue
```

## 3. 使用上下文刷新事件记录属性

在Spring Boot应用程序中记录属性的第一种方法是使用[Spring Events](https://www.baeldung.com/spring-events)，尤其是org.springframework.context.event.ContextRefreshedEvent类和相应的EventListener。我们将展示如何记录所有可用属性，以及仅从特定文件打印属性的更详细版本。

### 3.1 记录所有属性

让我们从创建一个Bean和事件监听器方法开始：

```java
@Component
public class AppContextRefreshedEventPropertiesPrinter {

    @EventListener
    public void handleContextRefreshed(ContextRefreshedEvent event) {
        // event handling logic
    }
}
```

我们用org.springframework.context.event.EventListener注解来标注事件监听器方法，当ContextRefreshedEvent发生时，Spring会调用标注的方法。

下一步是从触发的事件中获取org.springframework.core.env.ConfigurableEnvironment接口的实例，**ConfigurableEnvironment接口提供了一个有用的方法getPropertySources()，我们将使用它来获取所有属性源的列表，例如环境、JVM或属性文件变量**：

```java
ConfigurableEnvironment env = (ConfigurableEnvironment) event.getApplicationContext().getEnvironment();
```

现在让我们看看如何使用它来打印所有属性，不仅来自application.properties文件，还来自环境、JVM变量等等：

```java
env.getPropertySources()
    .stream()
    .filter(ps -> ps instanceof MapPropertySource)
    .map(ps -> ((MapPropertySource) ps).getSource().keySet())
    .flatMap(Collection::stream)
    .distinct()
    .sorted()
    .forEach(key -> LOGGER.info("{}={}", key, env.getProperty(key)));
```

首先，我们从可用的属性源创建一个[Stream](https://www.baeldung.com/java-streams)。然后，我们使用其filter()方法迭代org.springframework.core.env.MapPropertySource类的实例属性源。

顾名思义，该属性源中的属性存储在一个Map结构中。我们将在下一步中使用它，在该步骤中，我们将使用Stream的map()方法来获取属性键集。

接下来，我们使用Stream的flatMap()方法，因为我们想要迭代单个属性键，而不是一组键。我们还希望按字母顺序打印唯一、不重复的属性。

最后一步是记录属性键及其值。

当我们启动应用程序时，我们应该看到从各种来源获取的一大串属性列表：

```text
COMMAND_MODE=unix2003
CONSOLE_LOG_CHARSET=UTF-8
...
take.property=defaultValue
app.name=MyApp
app.description=MyApp is a Spring Boot application
...
java.class.version=52.0
ava.runtime.name=OpenJDK Runtime Environment
```

### 3.2 仅从application.properties文件记录属性

如果我们只想记录在application.properties文件中找到的属性，我们可以重用之前的几乎所有代码，只需要更改传递给filter()方法的Lambda函数：

```java
env.getPropertySources()
    .stream()
    .filter(ps -> ps instanceof MapPropertySource && ps.getName().contains("application.properties"))
    ...
```

现在，当我们启动应用程序时，我们应该看到以下日志：

```text
take.property=defaultValue
app.name=MyApp
app.description=MyApp is a Spring Boot application
```

## 4. 使用Environment接口记录属性

记录属性的另一种方法是使用org.springframework.core.env.Environment接口：

```java
@Component
public class EnvironmentPropertiesPrinter {
    @Autowired
    private Environment env;

    @PostConstruct
    public void logApplicationProperties() {
        LOGGER.info("{}={}", "bael.property", env.getProperty("bael.property"));
        LOGGER.info("{}={}", "app.name", env.getProperty("app.name"));
        LOGGER.info("{}={}", "app.description", env.getProperty("app.description"));
    }
}
```

与上下文刷新事件方法相比，唯一的限制是我们**需要知道属性名称才能获取其值**，Environment接口不提供列出所有属性的方法。另一方面，这绝对是一种更短更简单的技术。

当我们启动应用程序时，我们应该看到与之前相同的输出：

```text
tuyu.property=defaultValue 
app.name=MyApp 
app.description=MyApp is a Spring Boot application
```

## 5. 使用Spring Actuator记录属性

[Spring Actuator](https://www.baeldung.com/spring-boot-actuators)是一个非常有用的库，它为我们的应用程序带来了可用于生产的功能。**/env REST端点返回当前环境属性**。

首先，让我们将[Spring Actuator库](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator/2.7.3/jar)添加到我们的项目中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>3.0.0</version>
</dependency>
```

接下来，我们需要启用/env端点，因为它默认是禁用的。让我们打开application.properties并添加以下条目：

```properties
management.endpoints.web.exposure.include=env
```

现在，我们要做的就是启动应用程序并转到/env端点。在我们的例子中，地址是http://localhost:8080/actuator/env。我们应该看到一个包含所有环境变量(包括我们的属性)的大型JSON：

```json
{
    "activeProfiles": [],
    "propertySources": [
        ...
        {
            "name": "Config resource 'class path resource [application.properties]' via location 'optional:classpath:/' (document #0)",
            "properties": {
                "app.name": {
                    "value": "MyApp",
                    "origin": "class path resource [application.properties] - 10:10"
                },
                "app.description": {
                    "value": "MyApp is a Spring Boot application",
                    "origin": "class path resource [application.properties] - 11:17"
                },
                "tuyu.property": {
                    "value": "defaultValue",
                    "origin": "class path resource [application.properties] - 13:15"
                }
            }
        }
        ...
    ]
}
```

## 6. 总结

在本文中，我们学习了如何在Spring Boot应用程序中记录属性。

首先，我们创建了一个包含三个自定义属性的测试应用程序。然后，我们了解了三种不同的方法来检索和记录所需的属性。