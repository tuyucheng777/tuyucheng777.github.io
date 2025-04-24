---
layout: post
title:  不使用Git来使用Spring Cloud Config
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Config
---

## 1. 简介

[Spring Cloud Config](https://www.baeldung.com/spring-cloud-configuration)是一个库，它使Spring应用程序的配置外部化变得容易。它允许我们将配置数据公开为服务，从而可以轻松地从具有HTTP客户端的任何其他应用程序中提取配置数据。

在本教程中，我们将研究如何在没有Git的情况下使用Spring Cloud Config。

## 2. Spring Cloud Config概述

**Spring Cloud Config库是典型的客户端-服务器模型**，一个或多个集中式服务器从某个外部数据源读取配置数据，这些服务器公开各种HTTP端点，允许任何其他应用程序查询配置数据。

![](/assets/images/2025/springcloud/springcloudconfigwithoutgit01.png)

Spring Cloud Config还可以非常轻松地自动从Spring Boot应用程序连接到配置服务器，**然后，服务器提供的配置数据可以像客户端应用程序中的任何其他属性源一样使用**。

## 3. Git提供程序

**Spring Cloud Config最常见的用例是将配置数据存储在Git仓库中**，这种设置有几个优点：

- 灵活性：Git仓库可以保存各种文件类型，包括二进制文件。
- 安全性：易于在粒度级别控制读写访问。
- 审计：强大的历史跟踪功能可以轻松审计配置更改。
- 标准化：无论提供商如何，Git操作都是标准的，这意味着我们可以自行托管或使用任意数量的第三方提供商。
- 分布式：Git从一开始就设计为分布式，因此它非常适合云原生和微服务架构。

**尽管有上述所有优点，但Git可能并不总是存储配置数据的最佳选择**。例如，我们的组织可能已经将配置数据放在另一个数据存储中，例如关系型数据库。在这种情况下，将其迁移到Git可能不值得。

在下一节中，我们将仔细研究如何在没有Git的情况下使用Spring Cloud Config。

## 4. 不使用Git使用Spring Cloud Config

当我们谈论在Spring Cloud Config中使用Git以外的其他工具时，我们实际上指的是服务器组件。**我们对数据存储的选择不会影响客户端组件**，只有服务器会受到影响。

在Spring Cloud Config Server库中，有一个名为EnvironmentRepository的接口，它定义配置源，**所有配置源(包括Git和其他配置源)都必须实现此接口**。

让我们看一些提供的实现。

### 3.1 文件系统

Spring Cloud Config支持使用文件系统作为配置源，要启用此功能，我们必须在配置服务器的application.properties文件中指定以下值：
```properties
spring.cloud.config.server.native.search-locations=resources/other.properties
```

默认情况下，搜索位置假定为类路径资源，如果我们想使用任意文件，只需包含文件资源前缀：
```properties
spring.cloud.config.server.native.search-locations=file:///external/path/other.properties
```

除了此属性之外，配置服务器还需要在启用native Profile的情况下运行：
```shell
-Dspring.profiles.active=native
```

重要的是要记住，当使用文件系统配置源时，**我们需要确保文件系统在配置服务器运行的任何地方都可用**。这可能意味着使用分布式文件系统，例如NFS。

### 3.2 JDBC

Spring Cloud Config还可以使用关系型数据库通过[JDBC](https://www.baeldung.com/java-jdbc)加载配置数据，这是通过JdbcEnvironmentRepository类实现的，要启用此类，我们必须遵循几个步骤。

首先，[spring-jdbc](https://mvnrepository.com/artifact/org.springframework/spring-jdbc)库必须存在于类路径中，如果我们已经在使用[Spring Data JDBC](https://mvnrepository.com/artifact/org.springframework.data/spring-data-jdbc)或其他依赖库，那么它已经存在。否则，我们可以手动指定它：
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
</dependency>
```

其次，我们需要指定如何连接数据库：
```properties
spring.datasource.url=jdbc:mysql://dbhost:3306/springconfig
spring.datasource.username=dbuser
spring.datasource.password=dbpassword
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

在这种情况下，我们使用MySQL，但**任何兼容JDBC的驱动程序都可以使用**。

接下来，数据库必须包含一个名为PROPERTIES的表，该表具有以下列：

- APPLICATION
- PROFILE
- LABEL
- KEY
- VALUE

最后，我们需要为配置服务器指定jdbc Profile：
```shell
-Dspring.profiles.active=jdbc
```

### 3.3 Redis

Spring Cloud Config还支持[Redis](https://www.baeldung.com/java-redis-mongodb)作为配置源，这是使用RedisEnvironmentRepository类实现的，与JDBC源类似，我们需要遵循几个步骤来启用它。

首先，我们需要添加对[Spring Data Redis](https://mvnrepository.com/artifact/org.springframework.data/spring-data-redis)的依赖：
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
</dependency>
```

其次，我们需要设置一些如何连接Redis的属性：
```properties
spring.redis.host=localhost
spring.redis.port=6379
```

接下来，我们必须确保属性正确存储在Redis中，我们可以使用HMSET命令来存储一些示例属性：
```shell
HMSET application sample.property.name1 "somevalue" sample.property.name2 "anothervalue"
```

如果我们回显这些属性，我们应该看到以下数据：
```shell
HGETALL application
{
    "sample.property.name1": "somevalue",
    "sample.property.name2": "anothervalue"
}
```

最后，我们必须为Spring Cloud Config服务器启用redis Profile：
```plaintext
-Dspring.profiles.active=redis
```

使用Redis作为配置源还支持不同的Profile，为此，我们只需将Profile名称添加到应用程序的末尾：
```shell
HMSET application-dev sample.property.name1 "somevalue" sample.property.name2 "anothervalue"
```

在此示例中，我们在名为dev的Profile下创建一组新属性。

### 3.4 Secrets

许多云提供商的热门功能是secrets，[Secrets](https://www.baeldung.com/spring-cloud-kubernetes#secrets_springcloudk8)允许我们将敏感数据作为云基础设施的一部分进行安全存储。对于像用户名、主机名和密码这样的数据来说，secrets非常实用，我们希望将它们包含在应用程序配置中。

Spring Cloud Config为许多不同的云机密提供商提供支持，下面，我们将介绍AWS，它使用AwsSecretsManagerEnvironmentRepository类将AWS机密加载到属性源中。

此类依赖AWSSecretsManager类来完成与AWS通信的繁重工作，虽然我们可以自己手动创建它，但更直接的解决方案是使用[Spring Starter](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-aws-secrets-manager-config)：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-aws-secrets-manager-config</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

此模块包含一个自动配置，它将为我们创建一个AWSSecretsManager实例，我们所要做的就是在bootstrap.yml文件中指定一组属性：
```yaml
aws:
    secretsmanager:
        default-context: application
        prefix: /config
        profile-separator: _
        fail-fast: true
        name: ConfigServerApplication
        enabled: true
```

现在，假设我们想将数据库凭据存储在一个secret中，并使其可供配置服务器使用，我们只需在路径/config/application/database_credentials处创建一个新secret，在secret中，我们将存储连接数据库所需的必要键/值对。

此构造还支持不同的Profile，例如，如果我们有一个开发数据库服务器，我们也可以为其创建一个单独的secret，我们将其命名为/config/application/database_credentials_dev。

### 3.5 S3

存储配置的另一种便捷方式是使用云文件服务，让我们看看如何使用[AWS S3](https://www.baeldung.com/spring-cloud-aws-s3)作为配置源。

首先，我们需要将[AWS SDK](https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-s3outposts)添加到我们的项目中：
```xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3outposts</artifactId>
    <version>1.12.150</version>
</dependency>
```

然后，我们需要提供一些值来配置与包含我们的属性文件的S3存储桶的连接：
```properties
amazon.s3.access-key=key
amazon.s3.secret-key=secret
```

并且，我们需要为AWS S3配置提供程序提供特定属性：
```yaml
spring:
    cloud:
        config:
            server:
                awss3:
                    region: us-east-1
                    bucket: config-bucket
```

我们还需要设置一个Profile来确保加载AWS S3配置源：
```shell
-Dspring.profiles.active=awss3
```

剩下的就是在存储桶内创建我们所需的属性文件，包括任何特定于Profile的文件。请注意，当应用程序没有Profile时，配置服务器会假定为default Profile。因此，**我们应该将带有此后缀的文件与任何其他包含特定Profile名称的文件一起添加**。

### 3.6 自定义配置源

如果任何提供的配置源不能满足我们的需求，我们总是可以选择实现自己的配置源。通常，这涉及创建一个同时实现[EnvironmentRepository](https://www.javadoc.io/doc/org.springframework.cloud/spring-cloud-config-server/latest/org/springframework/cloud/config/server/environment/EnvironmentRepository.html)和[Ordered](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/Ordered.html)的新类：
```java
public class CustomConfigurationRepository implements EnvironmentRepository, Ordered {
    @Override
    public Environment findOne(String application, String profile, String label) {
        // Return a new Environment that is populated from
        // our desired source (DB, NoSQL store, etc)
    }

    @Override
    public int getOrder() {
        // Define our order relative to other configuration repositories
        return 0;
    }
}
```

然后，我们只需将此类实例化为一个新的Spring Bean：
```java
@Bean
public CustomConfigurationRepository customConfigurationRepository() {
    return new CustomConfigurationRepository();
}
```

## 4. 多种配置源

在某些情况下，可能需要使用多个配置源运行Spring Cloud Config。在这种情况下，我们必须指定几条数据。

假设我们想同时使用JDBC和Redis作为配置源，首先需要在bootstrap.yml文件中定义每个源的顺序：
```yaml
spring:
    cloud:
        config:
            server:
                redis:
                    order: 2
                jdbc:
                    order: 1
```

这使我们能够指定优先使用哪些配置源，由于顺序遵循正常的Spring Ordered注解处理，因此将**首先检查编号较低的配置源**。

此外，我们需要为服务器定义两个Profile：
```shell
-Dspring.profiles.active=jdbc,redis
```

请注意，我们也可以在YAML中指定激活Profile，并且，**可以使用相同的模式来定义任意数量的配置源**。

## 5. 总结

在本文中，我们介绍了可以与Spring Cloud Config一起使用的各种配置源。虽然Git是许多项目的绝佳默认源，但它可能并不总是最佳选择，我们已经看到Spring Cloud Config提供了多种替代方案，以及创建自定义提供程序的能力。