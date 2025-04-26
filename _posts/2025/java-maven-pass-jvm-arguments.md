---
layout: post
title:  如何通过Maven传递JVM参数
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本教程中，我们将使用JVM参数通过[Maven](https://www.baeldung.com/maven)运行Java代码，我们将探索如何将这些参数全局应用于构建，然后重点介绍如何将它们应用于特定的Maven插件。

## 2. 设置全局JVM参数

首先，我们将看到将JVM参数应用于主Maven进程的两种不同技术。

### 2.1 使用命令行

**要使用JVM参数运行Java Maven项目，我们需要设置MAVEN_OPTS环境变量**，此变量包含JVM启动时要使用的参数，并允许我们提供其他选项：

```shell
$ export MAVEN_OPTS="-Xms256m -Xmx512m"
```

在此示例中，我们通过MAVEN_OPTS定义[最小和最大堆大小](https://www.baeldung.com/jvm-parameters#explicit-heap-memory---xms-and-xmx-options)，然后，我们可以通过以下方式运行构建：

```shell
$ mvn clean install
```

我们设置的参数适用于构建的主要过程。

### 2.2 使用jvm.config文件

自动设置全局JVM参数的另一种方法是定义一个jvm.config文件，我们必须将此文件放在项目根目录下的.mvn文件夹中，该文件的内容是要应用的JVM参数。例如，为了模拟我们之前使用的命令行，我们的配置文件应为：

```text
-Xms256m -Xmx512m
```

## 3. 为特定插件设置JVM参数

Maven插件可能会fork新的JVM进程来执行其任务，因此，设置全局参数对它们不起作用。每个插件都定义了设置参数的方式，但大多数配置看起来都差不多。具体来说，我们将展示三个广泛使用的插件的示例：spring-boot Maven插件、surefire插件和failsafe插件。

### 3.1 设置示例

**我们的想法是建立一个[基本的Spring项目](https://www.baeldung.com/spring-boot-start)，但我们希望只有在设置某些JVM参数时代码才可运行**。

让我们从编写我们的[服务](https://www.baeldung.com/spring-component-repository-service#service)类开始：

```java
@Service
class MyService {
    int getLength() throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException {
        ArrayList<String> arr = new ArrayList<>();
        Field sizeField = ArrayList.class.getDeclaredField("size");
        sizeField.setAccessible(true);
        return (int) sizeField.get(arr);
    }
}
```

Java 9中引入的模块系统给反射访问带来了更多限制，因此，这段代码在Java 8中可以编译，但在Java 9及更高版本中，需要添加JVM选项–add-open来打开java-base模块的java-util包进行反射。

现在，我们可以在其上添加一个简单的[控制器类](https://www.baeldung.com/spring-controllers)：

```java
@RestController
class MyController {
    private final MyService myService;

    public MyController(MyService myService) {
        this.myService = myService;
    }

    @GetMapping("/length")
    Integer getLength() throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException {
        return myService.getLength();
    }
}
```

为了完成我们的设置，让我们添加主应用程序类：

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 3.2 Spring Boot Maven插件

让我们添加[最新版本](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-maven-plugin)的Maven的spring-boot插件的基础配置：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>3.3.0</version>
</plugin>
```

我们现在可以通过以下命令运行该项目：

```shell
$ mvn spring-boot:run
```

可以看到，应用程序启动成功，但是，当我们尝试对http://localhost:8080/length发出GET请求时，出现了以下错误：

```text
java.lang.reflect.InaccessibleObjectException: Unable to make field private int java.util.ArrayList.size accessible: module java.base does not "opens java.util" to unnamed module
```

正如我们在设置过程中所说，我们必须添加一些与反射相关的JVM参数来避免此错误，**特别是对于spring-boot Maven插件，我们需要使用jvmArguments配置标签**：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <jvmArguments>--add-opens java.base/java.util=ALL-UNNAMED</jvmArguments>
    </configuration>
</plugin>
```

现在我们可以重新运行该项目：http://localhost:8080/length上的GET请求现在返回0，即预期的答案。

### 3.3 Surefire插件

首先，让我们为我们的服务添加一个单元测试：

```java
class MyServiceUnitTest {
    MyService myService = new MyService();
    
    @Test
    void whenGetLength_thenZeroIsReturned() throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException {
        assertEquals(0, myService.getLength());
    }
}
```

[Surefire插件](https://www.baeldung.com/maven-surefire-plugin)通常用于通过Maven运行单元测试，让我们简单添加一下[最新版本](https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-surefire-plugin)插件的基本配置：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <includes>
            <include>**/*UnitTest.java</include>
        </includes>
    </configuration>
</plugin>
```

通过Maven运行单元测试的最短方法是运行以下命令：

```shell
$ mvn test
```

我们可以看到测试失败了，错误和之前一样，不过，**使用surefire插件，可以通过设置argLine配置标签来修正**：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <includes>
            <include>**/*UnitTest.java</include>
        </includes>
        <argLine>--add-opens=java.base/java.util=ALL-UNNAMED</argLine>
    </configuration>
</plugin>
```

我们的单元测试现在可以成功运行。

### 3.4 Failsafe插件

我们现在将为我们的控制器编写集成测试：

```java
@SpringBootTest(classes = MyApplication.class)
@AutoConfigureMockMvc
class MyControllerIntegrationTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    void whenGetLength_thenZeroIsReturned() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/length"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string("0"));
    }
}
```

通常，我们使用[failsafe插件](https://www.baeldung.com/maven-failsafe-plugin)通过Maven运行集成测试，再次，让我们使用[最新版本](https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-failsafe-plugin)的基础配置进行设置：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.3.0</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <includes>
            <include>**/*IntegrationTest.java</include>
        </includes>
    </configuration>
</plugin>
```

现在让我们运行集成测试：

```shell
$ mvn verify
```

不出所料，我们的集成测试失败了，原因和之前遇到的一样。failsafe插件的修复方法与surefire插件的修复方法相同，**我们必须使用argLine配置标签来设置JVM参数**：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.3.0</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <includes>
            <include>**/*IntegrationTest.java</include>
        </includes>
        <argLine>--add-opens=java.base/java.util=ALL-UNNAMED</argLine>
    </configuration>
</plugin>
```

集成测试现在将成功运行。

## 4. 总结

在本文中，我们学习了如何在Maven中运行Java代码时使用JVM参数。我们探索了两种设置构建全局参数的技术，然后，我们了解了如何将JVM参数传递给一些最常用的插件。我们的示例并不详尽，但一般来说，其他插件的工作原理非常相似。总而言之，我们随时可以参考插件文档来了解如何配置它。