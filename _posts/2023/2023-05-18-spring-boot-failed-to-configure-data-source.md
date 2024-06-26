---
layout: post
title:  解决“无法配置数据源”错误
category: springdata
copyright: springdata
excerpt: Spring Data
---

## 1. 概述

在这个简短的教程中，我们将讨论**导致Spring Boot项目中“Failed to configure a DataSource”错误的原因和解决方法**。

我们将使用两种不同的方法解决此问题：

1.  定义数据源
2.  禁用数据源的自动配置

### 延伸阅读

### [在Spring Boot中以编程方式配置数据源](https://www.baeldung.com/spring-boot-configure-data-source-programmatic)

了解如何以编程方式配置Spring Boot数据源，从而避开Spring Boot的自动数据源配置算法。

[阅读更多](https://www.baeldung.com/spring-boot-configure-data-source-programmatic)→

### [为测试配置单独的Spring DataSource](https://www.baeldung.com/spring-testing-separate-data-source)

一个快速、实用的教程，介绍如何配置单独的数据源以在Spring应用程序中进行测试。

[阅读更多](https://www.baeldung.com/spring-testing-separate-data-source)→

### [使用H2数据库的Spring Boot](https://www.baeldung.com/spring-boot-h2-database)

了解如何配置以及如何将H2数据库与Spring Boot一起使用。

[阅读更多](https://www.baeldung.com/spring-boot-h2-database)→

## 2. 问题

假设我们有一个Spring Boot项目，并且我们已经将[spring-data-starter-jpa](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa/3.0.3)依赖项和一个[MySQL JDBC驱动程序](https://central.sonatype.com/artifact/mysql/mysql-connector-java/8.0.32)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

但是当我们运行应用程序时，它会失败并出现以下错误：

```shell
Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded 
  datasource could be configured.

Reason: Failed to determine a suitable driver class
```

让我们看看为什么会发生这种情况。

## 3. 原因

**按照设计，Spring Boot自动配置会尝试根据添加到类路径的依赖项自动配置bean**。

由于我们的类路径具有JPA依赖性，因此Spring Boot会尝试自动配置JPA DataSource。**问题是我们没有给Spring执行自动配置所需的信息**。

例如，我们没有定义任何JDBC连接属性，而在使用外部数据库(如MySQL和MSSQL)时我们需要这样做。另一方面，对于H2等内存数据库，我们不会遇到这个问题，因为它们可以在没有所有这些信息的情况下创建数据源。

## 4. 解决方案

### 4.1 使用属性定义数据源

**由于问题是由于缺少数据库连接而导致的，我们可以通过提供数据源属性来简单地解决问题**。

首先，让我们在项目的[application.properties](https://www.baeldung.com/spring-testing-separate-data-source)文件中定义数据源属性：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/myDb
spring.datasource.username=user1
spring.datasource.password=pass
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

**或者我们可以在application.yml中提供数据源属性**：

```yaml
spring:
    datasource:
        driverClassName: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/myDb
        username: user1
        password: pass
```

### 4.2 以编程方式定义数据源

或者，我们可以**通过使用实用程序构建器类DataSourceBuilder[以编程方式定义数据源](https://www.baeldung.com/spring-boot-configure-data-source-programmatic)**。

我们需要提供数据库URL、用户名、密码和SQL驱动程序信息来创建我们的数据源：

```java
@Configuration
public class DatasourceConfig {
    @Bean
    public DataSource datasource() {
        return DataSourceBuilder.create()
              .driverClassName("com.mysql.cj.jdbc.Driver")
              .url("jdbc:mysql://localhost:3306/myDb")
              .username("user1")
              .password("pass")
              .build();
    }
}
```

简而言之，我们可以选择使用上述任何一个选项来根据我们的要求配置数据源。

### 4.3 排除数据源自动配置

在上一节中，我们通过将数据源属性添加到我们的项目中解决了这个问题。

但是，如果我们还没有准备好定义我们的数据源，我们如何解决这个问题呢？让我们看看**如何防止Spring Boot自动配置数据源**。

DataSourceAutoConfiguration类是使用spring.datasource.*属性配置数据源的基类。

现在，有几种方法可以将其[从自动配置中排除](https://www.baeldung.com/spring-boot-exclude-auto-configuration-test)。

首先，**我们可以使用application.properties文件中的spring.autoconfigure.exclude属性禁用自动配置**：

```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

我们可以使用我们的application.yml文件做同样的事情：

```yaml
spring:
    autoconfigure:
        exclude:
            - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

**或者我们可以在@SpringBootApplication或@EnableAutoConfiguration注解上使用exclude属性**：

```java
@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
```

在以上所有示例中，我们**禁用了DataSource的自动配置**。这不会影响任何其他bean的自动配置。

因此，综上所述，我们可以使用上述任何一种方法来禁用Spring Boot对数据源的自动配置。

理想情况下，我们应该提供数据源信息并仅使用排除选项进行测试。

## 5. 总结

在本文中，我们了解了导致“Failed to configure a DataSource”错误的原因。

首先，我们通过定义数据源解决了这个问题。

接下来，我们讨论了如何在不配置数据源的情况下解决该问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-data-modules)上获得。