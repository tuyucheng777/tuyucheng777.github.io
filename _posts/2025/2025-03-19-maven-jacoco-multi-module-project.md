---
layout: post
title:  使用JaCoCo实现Maven多模块项目覆盖
category: coverage
copyright: coverage
excerpt: JaCoCo
---

## 1. 概述

在本教程中，我们将构建一个Maven[多模块项目](https://www.baeldung.com/maven-multi-module)。在此项目中，服务和控制器将位于不同的模块中。然后，我们将编写一些测试并使用[Jacoco](https://www.baeldung.com/jacoco)来计算代码覆盖率。

## 2. 服务层

首先，让我们创建多模块应用程序的服务层。

### 2.1 Service类

**我们将创建Service并添加一些方法**：

```java
@Service
class MyService {

    String unitTestedOnly() {
        return "unit tested only";
    }

    String coveredByUnitAndIntegrationTests() {
        return "covered by unit and integration tests";
    }

    String coveredByIntegrationTest() {
        return "covered by integration test";
    }

    String notTested() {
        return "not tested";
    }
}
```

正如其名称所示：

- 位于同一层的[单元测试](https://www.baeldung.com/java-unit-testing-best-practices)将测试方法unitTestedOnly()
- 单元测试将测试coveredByUnitAndIntegrationTests()，控制器模块中的[集成测试](https://www.baeldung.com/integration-testing-in-spring)也将覆盖此方法的代码
- 集成测试将覆盖coveredByIntegrationTest()，但是，没有单元测试会测试此方法
- 没有测试会覆盖方法notTested()

### 2.2 单元测试

**现在我们来编写相应的单元测试**：

```java
class MyServiceUnitTest {

    MyService myService = new MyService();

    @Test
    void whenUnitTestedOnly_thenCorrectText() {
        assertEquals("unit tested only", myService.unitTestedOnly());
    }

    @Test
    void whenTestedMethod_thenCorrectText() {
        assertEquals("covered by unit and integration tests", myService.coveredByUnitAndIntegrationTests());
    }
}
```

测试只是检查方法的输出是否符合预期。

### 2.3 Surefire插件配置

**我们将使用[Maven Surefire插件](https://www.baeldung.com/maven-surefire-plugin)来运行单元测试**，让我们在服务的模块pom.xml中对其进行配置：

```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.2.5</version>
        <configuration>
            <includes>
                <include>**/*Test.java</include>
            </includes>
        </configuration>
    </plugin>
</plugins>
```

## 3. 控制器层

我们现在将在多模块应用程序中添加一个控制器层。

### 3.1 Controller类

让我们添加控制器类：

```java
@RestController
class MyController {

    private final MyService myService;

    public MyController(MyService myService) {
        this.myService = myService;
    }

    @GetMapping("/tested")
    String fullyTested() {
        return myService.coveredByUnitAndIntegrationTests();
    }

    @GetMapping("/indirecttest")
    String indirectlyTestingServiceMethod() {
        return myService.coveredByIntegrationTest();
    }

    @GetMapping("/nottested")
    String notTested() {
        return myService.notTested();
    }
}
```

fullyTested()和indirectlyTestingServiceMethod()方法将通过集成测试进行测试，因此，**这些测试将涵盖两个服务方法coveredByUnitAndIntegrationTests()和coveredByIntegrationTest()**。另一方面，我们不会为notTested()编写任何测试。

### 3.2 集成测试

**现在可以测试我们的[RestController](https://www.baeldung.com/spring-controller-vs-restcontroller#spring-mvc-rest-controller)**：

```java
@SpringBootTest(classes = MyApplication.class)
@AutoConfigureMockMvc
class MyControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void whenFullyTested_ThenCorrectText() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/tested"))
                .andExpect(MockMvcResultMatchers.status()
                        .isOk())
                .andExpect(MockMvcResultMatchers.content()
                        .string("covered by unit and integration tests"));
    }

    @Test
    void whenIndirectlyTestingServiceMethod_ThenCorrectText() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/indirecttest"))
                .andExpect(MockMvcResultMatchers.status()
                        .isOk())
                .andExpect(MockMvcResultMatchers.content()
                        .string("covered by integration test"));
    }
}
```

在这些测试中，我们启动一个应用服务器并向其发送请求。然后，我们检查输出是否正确。

### 3.3 故障安全插件配置

**我们将使用[Maven Failsafe插件](https://www.baeldung.com/maven-failsafe-plugin)来运行集成测试**，最后一步是在控制器的模块pom.xml中对其进行配置：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.1.2</version>
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

## 4. 通过Jacoco聚合覆盖范围

Jacoco是Java应用程序中用于在测试期间测量代码覆盖率的工具，现在让我们计算覆盖率报告。

### 4.1 准备Jacoco代理

**prepare-agent阶段设置了必要的钩子和配置，以便Jacoco可以在运行测试时跟踪执行的代码**。在运行任何测试之前都需要此配置，因此，我们将准备步骤直接添加到父pom.xml中：

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 4.2 收集测试结果

为了收集测试覆盖率，我们将创建一个新的模块aggregate-report，它只包含一个pom.xml并且依赖于前两个模块。

得益于准备阶段，我们可以汇总每个模块的报告。**这是report-aggregate[目标](https://www.baeldung.com/maven-goals-phases#maven-goal)的工作**：

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <phase>verify</phase>
            <goals>
                <goal>report-aggregate</goal>
            </goals>
            <configuration>
                <dataFileIncludes>
                    <dataFileInclude>**/jacoco.exec</dataFileInclude>
                </dataFileIncludes>
                <outputDirectory>${project.reporting.outputDirectory}/jacoco-aggregate</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

现在我们可以从父模块运行verify目标：

```shell
$ mvn clean verify
```

在构建结束时，我们可以看到Jacoco在aggregate-report子模块的target/site/jacoco-aggregate文件夹中生成报告。

让我们打开index.html文件来看看结果。

首先，我们可以导航到控制器类的报告：

![](/assets/images/2025/coverage/mavenjacocomultimoduleproject01.png)

正如预期的那样，构造函数和fullyTested()以及indirectlyTestingServiceMethod()方法被测试覆盖，而notTested()未被覆盖。

现在让我们看一下服务类的报告：

![](/assets/images/2025/coverage/mavenjacocomultimoduleproject02.png)

这次，我们重点关注coveredByIntegrationTest()方法。如我们所知，服务模块中没有测试覆盖此方法，通过此方法代码的唯一测试是在控制器模块内部。但是，Jacoco意识到此方法有一个测试。在这种情况下，聚合一词具有其全部含义！

## 5. 总结

在本文中，我们创建了一个多模块项目，并借助Jacoco收集了测试覆盖率。

回想一下，我们需要在测试之前运行准备阶段，而聚合则在测试之后进行。更进一步，我们可以使用[SonarQube](https://www.baeldung.com/sonarqube-jacoco-code-coverage)之类的工具来获得覆盖率结果的出色概览。