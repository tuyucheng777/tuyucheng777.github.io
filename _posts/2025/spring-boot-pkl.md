---
layout: post
title:  将Pkl与Spring Boot集成
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将学习如何使用[Pkl](https://pkl-lang.org/main/current/language-reference/index.html)(一种配置即代码语言)在Spring Boot应用程序中定义配置。

传统上，我们可能在YAML、JSON或基于键值的Properties中定义应用程序配置。但是，这些都是静态格式，校验属性具有挑战性。此外，随着配置属性数量的增加，定义更复杂的分层配置变得越来越困难。

因此，通过Pkl、[HCL(Hashicorp配置语言)](https://developer.hashicorp.com/terraform/language)和[Dhal配置语言](https://dhall-lang.org/)等特殊语言使用配置即代码(CaC)可以帮助我们克服静态属性格式文件的挑战。

## 2. Pkl简介

配置即代码是一个流行的概念，它提倡“不重复自己”(DRY)原则并有助于简化配置管理。它采用声明式编码风格，提供了一种结构化且高效的方法来定义配置模板。它还提高了配置的可读性。

**Pkl有助于定义各种元素，例如对象类型、集合、Map和各种原始数据类型**。这种灵活性使该语言具有可扩展性，并有助于清晰简洁地轻松建模复杂配置。此外，其类型和验证机制有助于在应用程序部署之前捕获配置错误。

此外，Pkl还提供了出色的工具支持，便于轻松采用。它具有用于生成不同语言(如Java、Kotlin、Go和Swift)代码的工具，这对于在这些编程语言构建的应用程序中嵌入和读取Pickle配置至关重要。

此外，**[IntelliJ](https://pkl-lang.org/intellij/current/features/index.html)和[VS Code](https://pkl-lang.org/vscode/current/features.html)等IDE具有插件，可帮助使用Pkl开发配置**。Pkl还附带一个名为[pkl](https://pkl-lang.org/main/current/pkl-cli/index.html#installation)的CLI，可帮助评估pickle模块。它的一个功能是将.pkl配置转换为JSON、YAML和XML格式。

## 3. Spring配置Java Bean绑定

定义Spring配置的最常见方法是创建属性文件并使用[@Value](https://www.baeldung.com/spring-value-annotation)注解注入它们。然而，这通常会导致过多的样板代码。

值得庆幸的是，**Spring框架借助[@ConfigurationProperties](https://www.baeldung.com/configuration-properties-in-spring-boot)注解简化了这个过程，实现了配置与Java Bean的无缝绑定**。

但是，使用这种方法，我们仍然必须手动创建Java Bean，并对成员字段进行必要的验证。因此，配置漂移很难检测，并且经常会导致应用程序中出现严重错误。因此，像Pkl这样具有自动Java代码生成支持的配置定义语言是集成我们的应用程序配置的更可靠的方法。

## 4. Spring Boot集成

Pkl提供库来从Pickle文件生成POJO，稍后在运行时，pkl-spring库可以将Pickle配置填充到POJO中。让我们了解如何将其与Spring Boot应用程序集成。

### 4.1 先决条件

首先，我们将包含pkl-spring库的[Maven依赖](https://mvnrepository.com/artifact/org.pkl-lang/pkl-spring)：

```xml
<dependency>
    <groupId>org.pkl-lang</groupId>
    <artifactId>pkl-spring</artifactId>
    <version>0.16.0</version>
</dependency>
```

此外，我们将使用exec-maven-plugin从Pickle配置文件生成POJO：

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>gen-pkl-java-bind</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>java</goal>
            </goals>
            <configuration>
                <mainClass>org.pkl.codegen.java.Main</mainClass>
                <includeProjectDependencies>false</includeProjectDependencies>
                <additionalClasspathElements>
                    <classpathElement>${pkl-tools-filepath}/pkl-tools-0.27.2.jar</classpathElement>
                </additionalClasspathElements>
                <arguments>
                    <argument>${project.basedir}/src/main/resources/ToolIntegrationTemplate.pkl</argument>
                    <argument>-o</argument>
                    <argument>${project.build.directory}/generated-sources</argument>
                    <argument>--generate-spring-boot</argument>
                    <argument>--generate-getters</argument>
                </arguments>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Maven插件执行Java代码生成器工具[pkl-tools](https://mvnrepository.com/artifact/org.pkl-lang/pkl-tools/)：

```shell
java -cp pkl-tools-0.27.2.jar org.pkl.codegen.java.Main \
src/main/resources/ToolIntegration.pkl \ 
-o ${project.basedir}/src/main --generate-spring-boot
```

假设有一个Pickle文件ToolIntegrationTemplate.pkl，该工具会在target/generated-sources文件夹中生成与Pickle文件对应的源代码。

**–-generate-spring-boot参数将必要的Spring Boot类级别@ConfigurationProperties注解包含在Java Bean中。此外，–-generate-getters参数将属性键声明为私有**。

对于Gradle用户来说幸运的是，[pkl-gladle插件](https://pkl-lang.org/main/current/pkl-gradle/index.html)提供了对生成POJO的开箱即用支持。

### 4.2 在Pkl中创建配置

让我们考虑一个数据集成应用程序，它从Git和Jira等源系统获取数据并将其发送到其他目标系统。

让我们首先定义Pickle模板ToolIntegrationTemplate.pkl来保存连接属性：

```json
module cn.tuyucheng.taketoday.spring.pkl.ToolIntegrationProperties

class Connection {
    url:String
    name:String
    credential:Credential
}

class Credential {
    user:String
    password:String
}

gitConnection: Connection
jiraConnection: Connection
```

总的来说，我们定义了两个Pkl类Connection和Credential，并声明了两个Connection类型的对象gitConnection和jiraConnection。

接下来我们在application.pkl文件中定义gitConnection和jiraConnection：

```json
amends "ToolIntegrationTemplate.pkl"
gitConnection {
    url = "https://api.github.com"
    name = "GitHub"
    credential {
        user = "gituser"
        password = read("env:GITHUB_PASSWORD")
    }
}
jiraConnection {
    url = "https://jira.atlassian.com"
    name = "Jira"
    credential {
        user = "jirauser"
        password = read("env:JIRA_PASSWORD")
    }
}
```

我们已经用数据源的连接属性填充了模板，值得注意的是，我们没有在配置文件中硬编码密码。相反，我们使用Pkl [read()](https://pkl-lang.org/main/current/language-reference/index.html#resources)函数从环境变量中检索密码，这是一种安全的方法。

### 4.3 将Pkl模板转换为POJO

在Pickle文件中定义配置后，可以通过执行pom.xml文件中定义的Maven目标exec:java来生成POJO：

```shell
mvn exec:java@gen-pkl-java-bind
```

最终，pkl-tools库生成一个POJO和一个属性文件：

![](/assets/images/2025/springboot/springbootpkl01.png)

ToolIntegrationProperties类由两个内部类Connection和Credential组成，它还带有@ConfigurationProperties注解，可帮助将application.pkl中的属性与ToolIntegrationProperties对象绑定。

### 4.4 将Pickle配置绑定到POJO

我们先定义与源系统集成的服务类JiraService和GitHubService：

```java
public class JiraService {
    private final ToolIntegrationProperties.Connection jiraConnection;

    public JiraService(ToolIntegrationProperties.Connection connection) {
        this.jiraConnection = connection;
    }
    // ...methods getting data from Jira
}
```

```java
public class GitHubService {
    private final ToolIntegrationProperties.Connection gitConnection;

    public GitHubService(ToolIntegrationProperties.Connection connection) {
        this.gitConnection = connection;
    }
    // ...methods getting data from GitHub
}
```

这两个服务都有以ToolIntegrationProperties作为参数的构造函数。稍后，参数中的连接详细信息将分配给ToolIntegrationProperties.Connection类型的类级属性。

接下来，我们将定义一个配置类，实例化服务并将其声明为Spring框架Bean：

```java
@Configuration
public class ToolConfiguration {
    @Bean
    public GitHubService getGitHubService(ToolIntegrationProperties toolIntegration) {
        return new GitHubService(toolIntegration.getGitConnection());
    }

    @Bean
    public JiraService getJiraService(ToolIntegrationProperties toolIntegration) {
        return new JiraService(toolIntegration.getJiraConnection());
    }
}
```

在应用程序启动时，Spring框架将方法参数与application.pkl文件中的配置绑定在一起。最终，通过调用服务构造函数来实例化Bean。

### 4.5 服务注入和执行

最后，**我们可以在其他Spring组件中使用@Autowired注解注入服务Bean**。

使用@SpringBootTest，我们将验证应用程序是否可以从Pickle文件加载Jira连接配置：

```java
@SpringBootTest
public class SpringPklUnitTest {
    @Autowired
    private JiraService jiraService;

    @Test
    void whenJiraConfigsDefined_thenLoadFromApplicationPklFile() {
        ToolIntegrationProperties.Connection jiraConnection = jiraService.getJiraConnection();
        ToolIntegrationProperties.Credential jiraCredential = jiraConnection.getCredential();

        assertAll(
                () -> assertEquals("Jira", jiraConnection.getName()),
                () -> assertEquals("https://jira.atlassian.com", jiraConnection.getUrl()),
                () -> assertEquals("jirauser", jiraCredential.getUser()),
                () -> assertEquals("jirapassword", jiraCredential.getPassword()),
                () -> assertEquals("Reading issues from Jira URL https://jira.atlassian.com",
                        jiraService.readIssues())
        );
    }
}
```

为了演示，我们通过pom.xml文件在环境变量中设置了Git和JIRA密码。运行测试后，我们确认JIRA连接和凭据详细信息与application.pkl文件中定义的值匹配。最后，我们还假设如果正确实现，JiraService#readIssues()将成功执行。目前，该方法仅返回一个虚拟字符串。

类似地，让我们检查应用程序启动后是否从Pickle文件加载GitHub连接配置：

```java
@SpringBootTest
public class SpringPklUnitTest {
    @Autowired
    private GitHubService gitHubService;

    @Test
    void whenGitHubConfigsDefined_thenLoadFromApplicationPklFile() {
        ToolIntegrationProperties.Connection gitHubConnection = gitHubService.getGitConnection();
        ToolIntegrationProperties.Credential gitHubCredential = gitHubConnection.getCredential();

        assertAll(
          () -> assertEquals("GitHub", gitHubConnection.getName()),
          () -> assertEquals("https://api.github.com", gitHubConnection.getUrl()),
          () -> assertEquals("gituser", gitHubCredential.getUser()),
          () -> assertEquals("gitpassword", gitHubCredential.getPassword()),

          () -> assertEquals("Reading issues from GitHub URL https://api.github.com", 
            gitHubService.readIssues())
        );
    }
}
```

正如预期的那样，这次GitHub连接和凭据详细信息也与application.pkl文件中定义的值相匹配。因此，有了正确的连接详细信息，我们可以推测GitHubServiceService#readIssues()如果正确实现，将连接到GitHub并获取问题。与JiraService#readIssues()一样，该方法目前仅返回一个虚拟字符串。

## 5. 总结

在本文中，我们了解了使用Pickle文件定义Spring Boot应用程序配置的好处和方法。但是，掌握Pkl语言概念对于设计可扩展且可维护的复杂配置同样重要。此外，了解Spring框架将外部属性绑定到POJO的功能也是必不可少的。