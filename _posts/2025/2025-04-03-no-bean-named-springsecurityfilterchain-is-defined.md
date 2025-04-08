---
layout: post
title:  没有定义名为‘springSecurityFilterChain’的Bean
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 问题

本文讨论了Spring Security配置问题-应用程序引导过程抛出以下异常：

```text
SEVERE: Exception starting filter springSecurityFilterChain
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No bean named 'springSecurityFilterChain' is defined
```

## 2. 原因

导致此异常的原因很简单-Spring Security查找名为springSecurityFilterChain的Bean(默认情况下)，但找不到它。主Spring Security过滤器DelegatingFilterProxy需要此Bean，该过滤器在web.xml中定义：

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

这只是一个将其所有逻辑委托给springSecurityFilterChain Bean的代理。

## 3. 解决方案

上下文中缺少此Bean的最常见原因是安全XML配置**没有定义<http\>元素**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:beans="http://www.springframework.org/schema/beans"
             xmlns:sec="http://www.springframework.org/schema/security"
             xsi:schemaLocation="
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security-3.1.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

</beans:beans>
```

如果XML配置使用安全命名空间-如上例所示，那么声明一个简单的<http\>元素将确保创建过滤器Bean并且一切都正确启动：

```xml
<http auto-config='true'>
    <intercept-url pattern="/**" access="ROLE_USER" />
</http>
```

**另一个可能的原因是安全配置根本没有导入到Web应用程序的整体上下文中**。

如果安全XML配置文件名为springSecurityConfig.xml，请确保资源已导入：

```java
@ImportResource({"classpath:springSecurityConfig.xml"})
```

或者在XML中：

```xml
<import resource="classpath:springSecurityConfig.xml" />
```

最后，可以在web.xml中更改过滤器Bean的默认名称-通常使用具有Spring Security的现有过滤器：

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
      org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>customFilter</param-value>
    </init-param>
</filter>
```

## 4. 总结

本文讨论了一个非常具体的Spring Security问题-缺少过滤链Bean，并展示了这个常见问题的解决方案。