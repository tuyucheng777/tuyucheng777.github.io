---
layout: post
title:  无法找到XML模式命名空间的Spring NamespaceHandler
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 问题

本文将讨论Spring中最常见的配置问题之一-**未找到某个Spring命名空间的命名空间处理程序**。大多数情况下，这意味着类路径中缺少一个特定的Spring jar。因此，让我们来看看这些缺失的模式可能是什么，以及每个模式缺少的依赖是什么。

## 2. http://www.springframework.org/schema/security

[security命名空间](https://docs.spring.io/spring-security/site/docs/5.2.x/reference/html/ns-config.html)不可用是迄今为止实践中遇到的最广泛的问题：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:beans="http://www.springframework.org/schema/beans"
    xsi:schemaLocation="
        http://www.springframework.org/schema/security 
        http://www.springframework.org/schema/security/spring-security-3.2.xsd
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd">

</beans:beans>
```

这会导致以下异常：

```text
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace 
[http://www.springframework.org/schema/security]
Offending resource: class path resource [securityConfig.xml]
```

解决方案很简单-项目的类路径中缺少spring-security-config依赖：

```xml
<dependency> 
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>3.2.5.RELEASE</version>
</dependency>
```

这会将正确的命名空间处理程序(在本例中为SecurityNamespaceHandler)放在类路径上，并准备解析security命名空间中的元素。

在[之前](https://www.baeldung.com/spring-security-with-maven)的教程中可以找到完整的Spring Security设置的完整Maven配置。

## 3. http://www.springframework.org/schema/aop

当使用aop命名空间而类路径上没有必要的Spring AOP库时，也会出现同样的问题：

```xml
<beans 
    xmlns="http://www.springframework.org/schema/beans" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.1.xsd">

</beans>
```

确切的异常情况：

```text
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace 
[http://www.springframework.org/schema/aop]
Offending resource: ServletContext resource [/WEB-INF/webConfig.xml]
```

解决方案类似-需要将spring-aop jar添加到项目的类路径中：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.1.0.RELEASE</version>
</dependency>
```

在这种情况下，添加新的依赖后AopNamespaceHandler将出现在类路径中。

## 4. http://www.springframework.org/schema/tx

使用事务命名空间-一个用于配置事务语义的虽小但非常有用的命名空间：

```xml
<beans 
    xmlns="http://www.springframework.org/schema/beans" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.1.xsd">

</beans>
```

如果正确的jar不在类路径上，也会导致异常：

```text
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace
[http://www.springframework.org/schema/tx]
Offending resource: class path resource [daoConfig.xml]
```

这里缺少的依赖是spring-tx：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.1.0.RELEASE</version>
</dependency>
```

现在，正确的NamespaceHandler(即TxNamespaceHandler)将出现在类路径上，允许使用XML和注解进行声明式事务管理。

## 5. http://www.springframework.org/schema/mvc

继续看mvc命名空间：

```xml
<beans 
    xmlns="http://www.springframework.org/schema/beans" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:tx="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd">

</beans>
```

缺少依赖将导致以下异常：

```text
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace
[http://www.springframework.org/schema/mvc]
Offending resource: class path resource [webConfig.xml]
```

在这种情况下，缺少的依赖是spring-mvc：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.1.0.RELEASE</version>
</dependency>
```

将其添加到pom.xml将会把MvcNamespaceHandler添加到类路径-允许项目使用命名空间配置MVC语义。

## 6. 总结

最后，如果你使用Eclipse来管理Web服务器并进行部署，请确保[项目的部署程序集部分已正确配置](http://stackoverflow.com/questions/4777026/classnotfoundexception-dispatcherservlet-when-launching-tomcat-maven-dependenci/4777496#4777496)- 即Maven依赖在部署时实际上包含在类路径中。

本教程讨论了“无法找到XML模式命名空间的Spring NamespaceHandler”问题的常见原因并针对每种情况提供了解决方案。