---
layout: post
title:  RESTX简介
category: webmodules
copyright: webmodules
excerpt: RESTX
---

## 1. 概述

在本教程中，我们将介绍轻量级Java REST框架[RESTX](http://restx.io/)。

## 2. 特点

使用RESTX框架构建RESTful API非常容易，它具有REST框架的所有默认功能，例如提供和使用JSON、查询和路径参数、路由和过滤机制、使用情况统计和监控。

RESTX还带有直观的管理Web控制台和命令行安装程序，方便引导。

它使用Apache License 2的许可，并由开发人员社区维护；RESTX的最低Java要求是JDK 7。

## 3. 配置

**RESTX带有一个方便的shell/命令应用程序，可用于快速启动Java项目**。

我们需要先安装该应用程序才能继续，详细安装说明可在[此处](http://restx.io/docs/install.html)查看。

## 4. 安装核心插件

现在，是时候安装核心插件了，以便能够从shell本身创建应用程序。

在RESTX shell中，让我们运行以下命令：

```shell
shell install
```

然后它会提示我们选择要安装的插件，我们需要选择指向io.restx:restx-core-shell的数字。**安装完成后，shell将自动重启**。

## 5. Shell应用Bootstrap

使用RESTX shell可以非常方便地启动新应用程序，它提供了基于向导的指南。

我们首先在shell上执行以下命令：

```shell
app new
```

此命令将触发向导，然后我们可以采用默认选项，也可以根据我们的要求进行更改：

![](/assets/images/2025/webmodules/javarestx01.png)

由于我们选择生成pom.xml，因此可以轻松将项目导入任何标准Java IDE。

在少数情况下，我们可能需要调整[IDE设置](http://restx.io/docs/ide.html)。

我们的下一步将是构建项目：

```shell
mvn clean install -DskipTests
```

**构建成功后，我们可以从IDE将AppServer类作为Java应用程序运行**，这将使用管理控制台启动服务器，并监听端口8080。

我们可以浏览http://127.0.0.1:8080/api/@/ui并查看基本UI。

**以/@/开头的路由用于管理控制台，它是RESTX中的保留路径**。

**要登录管理控制台，我们可以使用默认用户名“admin”和创建应用程序时提供的密码**。

在我们使用控制台之前，让我们先探索一下代码并了解向导生成的内容。

## 6. RESTX资源

路由在<main_package\>.rest.HelloResource类中定义：

```java
@Component
@RestxResource
public class HelloResource {
    @GET("/message")
    @RolesAllowed(Roles.HELLO_ROLE)
    public Message sayHello() {
        return new Message().setMessage(String.format("hello %s, it's %s",
                RestxSession.current().getPrincipal().get().getName(),
                DateTime.now().toString("HH:mm:ss")));
    }
}
```

很明显，RESTX使用默认的J2EE注解来实现安全性和REST绑定；在大多数情况下，它使用自己的注解来实现依赖注入。

RESTX还支持[将方法参数映射到请求](http://restx.io/docs/ref-core.html)的许多合理的默认值。

**除了这些标准注解之外，还有@RestxResource，它将其声明为RESTX识别的资源**。

基本路径添加到src/main/webapp/WEB-INF/web.xml中，在我们的例子中，它是/api，因此我们可以向http://localhost:8080/api/message发送GET请求，假设身份验证正确。

Message类只是一个RESTX序列化为JSON的Java Bean。

我们通过使用引导程序生成的HELLO_ROLE指定RolesAllowed注解来控制用户访问。

## 7. 模块类

如前所述，RESTX使用J2EE标准的依赖注入注解，比如@Named，并在需要时发明自己的注解，很可能从Dagger框架的@Module和@Provides中得到启发。

**它使用这些来创建应用程序的主模块，其中包括定义管理员密码**：

```java
@Module
public class AppModule {

    @Provides
    public SignatureKey signatureKey() {
        return new SignatureKey("restx-demo -44749418370 restx-demo 801f-4116-48f2-906b"
                .getBytes(Charsets.UTF_8));
    }

    @Provides
    @Named("restx.admin.password")
    public String restxAdminPassword() {
        return "1234";
    }

    @Provides
    public ConfigSupplier appConfigSupplier(ConfigLoader configLoader) {
        return configLoader.fromResource("restx/demo/settings");
    }

    // other provider methods to create components 
}
```

@Module定义一个可以定义其他组件的类，类似于Dagger中的@Module或Spring中的@Configuration。

@Provides以编程方式公开组件，例如Dagger中的@Provides或Spring中的@Bean。

最后，@Named注解用于指示所生成的组件的名称。

AppModule还提供了一个SignatureKey，用于对发送给客户端的内容进行签名。例如，在为示例应用程序创建会话时，这将设置一个使用配置的密钥签名的cookie：

```text
HTTP/1.1 200 OK
...
Set-Cookie: RestxSessionSignature-restx-demo="ySfv8FejvizMMvruGlK3K2hwdb8="; RestxSession-restx-demo="..."
...
```

查看[RESTX的组件工厂/依赖注入文档](http://restx.io/docs/ref-factory.html)以了解更多信息。

## 8. 启动类

最后，AppServer类用于在嵌入式Jetty服务器中将该应用程序作为标准Java应用程序运行：

```java
public class AppServer {
    public static final String WEB_INF_LOCATION = "src/main/webapp/WEB-INF/web.xml";
    public static final String WEB_APP_LOCATION = "src/main/webapp";

    public static void main(String[] args) throws Exception {
        int port = Integer.valueOf(Optional.fromNullable(System.getenv("PORT")).or("8080"));
        WebServer server = new Jetty8WebServer(WEB_INF_LOCATION, WEB_APP_LOCATION, port, "0.0.0.0");
        System.setProperty("restx.mode", System.getProperty("restx.mode", "dev"));
        System.setProperty("restx.app.package", "restx.demo");
        server.startAndAwait();
    }
}
```

这里，dev模式用于开发阶段，以启用自动编译等功能，从而缩短开发反馈循环。

**我们可以将应用程序打包为war(Web存档)文件，以便部署在独立的J2EE Web容器中**。

在下一节中让我们了解如何测试该应用程序。

## 9. 使用规范进行集成测试

RESTX的强大功能之一是其“规范”概念，示例规范如下所示：

```yaml
title: should admin say hello
given:
    - time: 2013-08-28T01:18:00.822+02:00
wts:
    - when: |
        GET hello?who=xavier
      then: |
        {"message":"hello xavier, it's 01:18:00"}
```

该测试以[Given-When-Then](https://en.wikipedia.org/wiki/Given-When-Then)结构编写在[YAML](http://yaml.org/)文件中，该结构主要定义了在给定系统当前状态(Given)的情况下，API应如何响应(Then)特定请求(When)。

src/test/resources中的HelloResourceSpecTest类将触发上面规范中编写的测试：

```java
@RunWith(RestxSpecTestsRunner.class)
@FindSpecsIn("specs/hello")
public class HelloResourceSpecTest {}
```

RestxSpecTestsRunner类是一个[自定义的JUnit运行器](https://www.baeldung.com/junit-4-custom-runners)，它包含自定义的JUnit Rule，用于：

- 设置嵌入式服务器
- 准备系统状态(按照规范中给出的部分)
- 发出指定请求
- 验证预期响应

@FindSpecsIn注解指向应运行测试的规范文件的路径。

**规范有助于编写集成测试并在API文档中提供示例**，还有助于[Mock HTTP请求和记录请求/响应对](http://restx.io/docs/ref-specs.html)。

## 10. 手动测试

我们也可以通过HTTP手动测试，我们首先需要登录，为此，我们需要在RESTX控制台中对管理员密码进行哈希处理：

```shell
hash md5 <clear-text-password>
```

然后我们可以将其传递给/sessions端点：

```shell
curl -b u1 -c u1 -X POST -H "Content-Type: application/json" 
  -d '{"principal":{"name":"admin","passwordHash":"1d528266b85cf052803a57288"}}'
  http://localhost:8080/api/sessions
```

(请注意，Windows用户需要先[下载](https://curl.haxx.se/windows/)curl。)

现在，如果我们将会话用作/message请求的一部分：

```shell
curl -b u1 "http://localhost:8080/api/message?who=restx"
```

然后我们会得到类似这样的结果：

```json
{"message" : "hello admin, it's 09:56:51"}
```

## 11. 探索管理控制台

管理控制台提供了有用的资源来控制应用程序。

让我们通过浏览http://127.0.0.1:8080/admin/@/ui来查看主要功能。

### 11.1 API文档

API文档部分列出了所有可用路由，包括所有选项：

![](/assets/images/2025/webmodules/javarestx02.png)

我们可以点击各个路由并在控制台上尝试它们：

![](/assets/images/2025/webmodules/javarestx03.png)

### 11.2 监控

JVM Metrics部分显示带有活跃会话、内存使用情况和线程转储的应用程序指标：

![](/assets/images/2025/webmodules/javarestx04.png)

在Application Metrics下，我们默认主要监控两类元素：

- BUILD对应应用程序组件的实例化
- HTTP对应RESTX处理的HTTP请求

### 11.3 统计信息

RESTX允许用户选择在应用程序上收集和共享匿名统计数据，以便向RESTX社区提供信息，**我们可以通过排除restx-stats-admin模块轻松选择退出**。

统计信息报告诸如底层操作系统和JVM版本之类的信息：

![](/assets/images/2025/webmodules/javarestx05.png)

**由于此页面显示敏感信息，[请务必检查其配置选项](http://restx.io/stats.html)**。

除此之外，管理控制台还可以帮助我们：

- 检查服务器日志(Logs)
- 查看遇到的错误(Errors)
- 检查环境变量(Config)

## 12. 授权

**RESTX端点默认是安全的**，这意味着如果对于任何端点：

```java
@GET("/greetings/{who}")
public Message sayHello(String who) {
    return new Message(who);
}
```

未经身份验证调用时默认将返回401。

为了使端点公开，我们需要在方法或类级别使用@PermitAll注解：

```java
@PermitAll 
@GET("/greetings/{who}")
public Message sayHello(String who) {
    return new Message(who);
}
```

**请注意，在类级别，所有方法都是公共的**。

此外，该框架还允许使用@RolesAllowed注解指定用户角色：

```java
@RolesAllowed("admin")
@GET("/greetings/{who}")
public Message sayHello(String who) {
    return new Message(who);
}
```

使用此注解，RESTX将验证经过身份验证的用户是否还分配了admin角色；如果没有admin角色的经过身份验证的用户尝试访问端点，应用程序将返回403而不是401。

**默认情况下，用户角色和凭证存储在文件系统的单独文件中**。

因此，带有加密密码的用户ID存储在/data/credentials.json文件下：

```json
{
    "user1": "$2a$10$iZluUbCseDOvKnoe",
    "user2": "$2a$10$oym3Swr7pScdiCXu"
}
```

并且，用户角色在/data/users.json文件中定义：

```json
[
    {"name":"user1", "roles": ["hello"]},
    {"name":"user2", "roles": []}
]
```

在示例应用中，文件通过FileBasedUserRepository类加载到AppModule中：

```java
new FileBasedUserRepository<>(StdUser.class, mapper, 
    new StdUser("admin", ImmutableSet.<String> of("*")), 
    Paths.get("data/users.json"), Paths.get("data/credentials.json"), true)
```

StdUser类保存用户对象，它可以是自定义用户类，但需要可序列化为JSON。

当然，我们可以使用不同的[UserRepository](https://raw.githubusercontent.com/restx/restx/master/restx-security-basic/src/main/java/restx/security/UserRepository.java)实现，比如访问数据库的实现。

## 13. 总结

本教程概述了基于Java的轻量级RESTX框架。

该框架仍在开发中，使用时可能会存在一些缺陷；查看[官方文档](http://restx.io/docs/) 了解更多详细信息。