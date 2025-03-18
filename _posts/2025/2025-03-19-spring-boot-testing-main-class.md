---
layout: post
title:  测试Spring Boot应用程序的主类
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

**测试Spring Boot应用程序的主类对于确保应用程序正确启动至关重要**。单元测试通常侧重于单个组件，而验证应用程序上下文是否加载无问题可以防止生产中出现运行时错误。

在本教程中，我们将探索不同的策略来有效地测试Spring Boot应用程序的主类。

## 2. 设置

首先，我们建立一个简单的Spring Boot应用程序，我们可以使用[Spring Initializr](https://start.spring.io/)来生成基本的项目结构。

### 2.1 Maven依赖

要设置我们的项目，我们需要以下依赖项：

- [Spring Boot Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter)：构建Spring Boot应用程序的核心依赖项。
- [Spring Boot Starter Test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)：为Spring Boot应用程序提供JUnit和Assert等测试库。
- [Mockito Core](https://mvnrepository.com/artifact/org.mockito/mockito-core)：用于在单元测试中创建和验证Mock的Mock框架。

我们在pom.xml文件中添加以下依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>4.0.0</version>
    <scope>test</scope>
</dependency>
```

### 2.2 主应用程序类

主应用程序类是任何Spring Boot应用程序的核心，它不仅充当应用程序的入口点，还充当主配置类，管理组件并设置环境。让我们分解一下它的结构，并了解每个部分的重要性。

典型的Spring Boot主类如下所示：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

关键要素包括：

1. **[@SpringBootApplication](https://www.baeldung.com/spring-boot-annotations)注解**：此注解是组合三个基本注解的简写：

   - @Configuration：将此类标记为Bean定义的来源。
   - @EnableAutoConfiguration：告诉Spring Boot根据类路径设置、其他Bean和属性设置开始添加Bean。
   - @ComponentScan：扫描此类所在的包并注册所有Spring组件(Bean、服务等)。

   这种组合确保应用程序正确配置，而无需大量的XML配置或手动设置。

2. **main()方法**：与任何Java程序一样，main()方法用作入口点。在这里，它调用SpringApplication.run()：

   - 引导应用程序并加载Spring上下文，根据@SpringBootApplication注解配置一切。
   - 如果它是一个Web应用程序，则启动嵌入式Web服务器(如Tomcat或Jetty)，这意味着该应用程序可以独立运行而不需要外部服务器。
   - 接收命令行参数，可用于在运行时配置Profile(如–spring.profiles.active=dev)或其他设置。

3. **SpringApplication.run()**：此方法执行启动应用程序的繁重工作：

   - 它创建一个保存Bean和配置的ApplicationContext，允许Spring管理所有依赖项和组件。
   - 应用程序属性或命令行参数的任何运行时配置都在这里应用，并且启动依赖于这些设置的任何组件。

### 2.3 自定义并测试主应用程序类

对于大多数应用程序来说，application.properties或application.yml是设置配置的首选位置，因为它可以保持主类的整洁，并将设置组织在一个中心文件中。但是，我们可以直接在主类中自定义某些设置：

```java
@SpringBootApplication
public class Application {
   public static void main(String[] args) {
      SpringApplication app = new SpringApplication(Application.class);
      app.setBannerMode(Banner.Mode.OFF);
      app.setLogStartupInfo(false);
      app.setDefaultProperties(Collections.singletonMap("server.port", "8083"));
      app.run(args);
   }
}
```

在此示例中，我们实施以下调整：

- **禁用Banner**：Spring Boot默认在启动时打印Banner，如果我们想要更清晰的控制台输出，可以禁用它。
- **抑制启动日志**：Spring Boot默认会记录大量初始化信息，如果不需要，我们可以关闭其中的一些。
- **设置默认属性**：我们可以添加默认属性，例如指定自定义服务器端口。

当我们想要控制应用程序的详细程度和行为时，这些小的调整在测试环境或调试期间特别有用。

**测试主应用程序类有时看起来是多余的，但它很重要，因为它可以验证**：

- **上下文加载**：确保所有必需的Bean和配置都已到位。
- 环境配置：验证运行时Profile和环境属性是否正确应用。
- 启动逻辑：确认在主类中添加的任何自定义逻辑(如事件监听器或Banner)不会导致启动问题。

通过彻底理解并潜在地定制我们的主要应用程序类，我们可以确保我们的Spring Boot应用程序灵活且适用于不同的环境，无论是开发、测试还是生产。

## 3. 测试策略

我们将探讨测试主类的几种策略，从基本上下文加载测试到Mock和命令行参数。

### 3.1 基本上下文加载测试

测试应用程序上下文是否加载的最简单方法是使用@SpringBootTest，不带任何其他参数：

```java
@SpringBootTest
public class ApplicationContextTest {
    @Test
    void contextLoads() {
    }
}
```

在这里，@SpringBootTest会加载完整的应用程序上下文。如果任何Bean配置错误，测试就会失败，从而帮助我们尽早发现问题。在较大的应用程序中，我们可能会考虑将此测试配置为仅加载特定Bean以加快执行速度。

### 3.2 直接测试main()方法

为了让SonarQube等工具覆盖main()方法，我们可以直接对其进行测试：

```java
public class ApplicationMainTest {
    @Test
    public void testMain() {
        Application.main(new String[]{});
    }
}
```

这个简单的测试验证了main()方法的执行是否没有抛出异常。它不会加载整个上下文，但可以确保该方法不包含运行时问题。

### 3.3 Mock SpringApplication.run()

启动整个应用程序上下文非常耗时，因此为了优化这一点，我们可以使用Mockito Mock SpringApplication.run()。

如果我们在main()方法中围绕SpringApplication.run添加了自定义逻辑(例如，记录日志、处理参数或设置自定义属性)，则在不加载整个应用程序上下文的情况下测试该逻辑可能很有意义。在这种情况下，我们可以Mock SpringApplication.run()来验证围绕调用的其他行为。

从3.4.0版本开始，Mockito支持[静态方法Mock](https://www.baeldung.com/mockito-mock-static-methods)，这使我们能够Mock SpringApplication.run()。如果我们的main()方法包含我们想要在不加载完整应用程序上下文的情况下验证的其他逻辑，则Mock SpringApplication.run()会特别有用。

为了便于测试隔离，我们可以重构main()方法，以便用单独的可测试方法处理实际的启动逻辑。这种分离使我们能够专注于测试初始化逻辑，而无需启动整个应用程序：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        initializeApplication(args);
    }

    static ConfigurableApplicationContext initializeApplication(String[] args) {
        return SpringApplication.run(Application.class, args);
    }
}
```

现在，main()方法委托给initialiseApplication()，我们可以单独Mock它。

使用重构的initializeApplication()方法，**我们可以继续Mock SpringApplication.run()并验证行为，而无需完全启动应用程序上下文**：

```java
public class ApplicationMockTest {
    @Test
    public void testMainWithMock() {
        try (MockedStatic<SpringApplication> springApplicationMock = mockStatic(SpringApplication.class)) {
            ConfigurableApplicationContext mockContext = mock(ConfigurableApplicationContext.class);
            springApplicationMock.when(() -> SpringApplication.run(Application.class, new String[] {}))
              .thenReturn(mockContext);

            Application.main(new String[] {});

            springApplicationMock.verify(() -> SpringApplication.run(Application.class, new String[] {}));
        }
    }
}
```

在此测试中，我们Mock SpringApplication.run()以防止实际应用程序启动，从而节省时间并隔离测试。通过返回Mock的ConfigurableApplicationContext，我们可以安全地处理initializeApplication()中的任何交互，从而避免实际上下文初始化。此外，我们使用verify()来确认SpringApplication.run()是否使用正确的参数调用，这使我们能够验证启动顺序而无需完整的应用程序上下文。

当main()方法包含自定义启动逻辑时，这种方法特别有用，因为它允许我们独立测试和验证该逻辑，从而保持测试执行快速且隔离。

### 3.4 使用@SpringBootTest和useMainMethod

从Spring Boot 2.2开始，我们可以指示[@SpringBootTest](https://www.baeldung.com/spring-boot-testing)在启动应用程序上下文时使用main()方法：

```java
@SpringBootTest(useMainMethod = SpringBootTest.UseMainMethod.ALWAYS)
public class ApplicationUseMainTest {
    @Test
    public void contextLoads() {
    }
}
```

**将useMainMethod设置为ALWAYS可确保main()方法在测试期间运行**，此方法在以下情况下非常有用：

- **main()方法包含额外的设置逻辑**：如果我们的main()方法包含任何对于应用程序正确启动很重要的设置或配置(例如设置自定义属性或额外的日志记录)，则此测试会将该逻辑作为上下文初始化的一部分进行验证。
- **增加代码覆盖率**：此策略允许我们将main()方法作为测试的一部分进行覆盖，确保在单个测试中验证完整的启动序列(包括main()方法)。当希望进行完整的启动验证而无需编写单独的测试来直接调用main()时，这尤其有用。

### 3.5 从覆盖范围中排除主类

如果主类不包含关键逻辑，我们可能会选择将其从代码覆盖率报告中排除，以关注更有意义的领域。

要将main()方法排除在代码覆盖范围之外，我们可以使用@Generated对其进行标注，该注解可从javax.annotation(或jakarta.annotation(如果使用Jakarta EE))包中获得。此方法向代码覆盖率工具(例如[JaCoCo](https://www.baeldung.com/jacoco)或[SonarQube](https://www.baeldung.com/sonar-qube))发出信号，表示应在覆盖率指标中忽略该方法：

```java
@SpringBootApplication
public class Application {
    @Generated(value = "Spring Boot")
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

@Generated中的value属性是必需的，它通常指示生成代码的来源。这里指定“Spring Boot”是为了明确此代码是Spring Boot启动序列的一部分。

如果我们更喜欢一种更简单的方法并且我们的覆盖工具支持它，我们可以在main()方法上使用@SuppressWarnings(“unused”)将其排除在覆盖范围之外：

```java
@SpringBootApplication
public class Application {
    @SuppressWarnings("unused")
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

一般来说，使用@Generated是一种更可靠的代码覆盖率排除方法，因为大多数工具将此注解识别为忽略覆盖率指标中注解代码的指令。

**从覆盖范围中排除主类的另一种选择是直接配置我们的代码覆盖范围工具**。

对于pom.xml中的JaCoCo：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.7</version>
            <configuration>
                <excludes>
                    <exclude>cn/tuyucheng/taketoday/mainclasstest/Application</exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

对于SonarQube(在sonar-project.properties中)：

```properties
sonar.exclusions=src/main/java/cn/tuyucheng/taketoday/mainclasstest/Application.java
```

从覆盖范围中排除琐碎代码比仅仅为了满足覆盖范围指标而编写测试更为实用。

### 3.6 处理应用程序参数

如果我们的应用程序使用命令行参数，我们可能希望使用特定输入来测试main()方法：

```java
public class ApplicationArgumentsTest {
    @Test
    public void testMainWithArguments() {
        String[] args = { "--spring.profiles.active=test" };
        Application.main(args);
    }
}
```

此测试检查应用程序是否使用特定参数正确启动，当需要验证某些Profile或配置时，此测试非常有用。

## 4. 总结

测试Spring Boot应用程序的主类可确保应用程序正确启动并增加代码覆盖率。我们探索了各种策略，从基本上下文加载测试到Mock SpringApplication.run()。根据项目的需求，我们可以选择最能平衡测试执行时间和覆盖率要求的方法。当方法包含琐碎代码时，将主类排除在覆盖范围之外也是一个可行的选择。

通过实施这些测试，我们增强了应用程序启动过程的可靠性，并在开发周期早期发现潜在问题。