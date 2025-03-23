---
layout: post
title:  Spring Data Redis的基于属性的配置
category: spring-data
copyright: spring-data
excerpt: Spring Data Redis
---

## 1. 概述

Spring Boot的主要吸引力之一是它通常将第三方配置减少到仅几个属性。

在本教程中，我们将**了解Spring Boot如何简化使用Redis的操作**。

## 2. 为什么选择Redis？

[Redis](https://redis.io/)是最流行的内存数据结构存储之一，因此，它可以用作数据库、缓存和消息代理。

在性能方面，它以[快速的响应时间](https://redis.io/topics/benchmarks)而闻名。因此，它每秒可以处理数十万次操作，并且易于扩展。

而且它与Spring Boot应用程序配合得很好，例如，我们的微服务架构可以将其用作缓存，也可以将其用作NoSQL数据库。

## 3. 运行Redis

首先，让我们使用其[官方Docker镜像](https://hub.docker.com/_/redis/)创建一个Redis实例。

```shell
$ docker run -p 16379:6379 -d redis:6.0 redis-server --requirepass "mypass"
```

上面我们刚刚在端口16379上启动了一个Redis实例，密码为mypass。

## 4. Starter

Spring大力支持使用[Spring Data Redis](https://www.baeldung.com/spring-data-redis-tutorial)连接我们的Spring Boot应用程序和Redis。

因此，接下来，让我们确保pom.xml中具有[spring-boot-starter-data-redis](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.7.11</version>    
</dependency>
```

## 5. Lettuce

接下来我们来配置客户端。

**我们将使用的Java Redis客户端是[Lettuce](https://www.baeldung.com/java-redis-lettuce)，因为Spring Boot默认使用它**。但是，我们也可以使用[Jedis](https://www.baeldung.com/jedis-java-redis-client-library)。

无论哪种方式，结果都是RedisTemplate的一个实例：

```java
@Bean
public RedisTemplate<Long, Book> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<Long, Book> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    // Add some specific configuration here. Key serializers, etc.
    return template;
}
```

## 6. 属性

当我们使用Lettuce时，我们不需要配置RedisConnectionFactory，Spring Boot会为我们完成这项工作。

那么，我们所剩下的就是在我们的application.properties文件中指定一些属性(对于Spring Boot 2.x)：

```properties
spring.redis.database=0
spring.redis.host=localhost
spring.redis.port=16379
spring.redis.password=mypass
spring.redis.timeout=60000
```

对于Spring Boot 3.x，我们需要设置以下属性：

```properties
spring.data.redis.database=0
spring.data.redis.host=localhost
spring.data.redis.port=16379
spring.data.redis.password=mypass
spring.data.redis.timeout=60000
```

解释：

- database设置连接工厂使用的数据库索引
- host是服务器主机所在的位置
- port表示服务器监听的端口
- password是服务器的登录密码
- timeout建立连接超时

当然，我们还可以配置许多其他属性，[完整的配置属性列表](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#data-properties)可在Spring Boot文档中找到。

## 7. 演示

最后，让我们尝试在应用程序中使用它。如果我们想象一个Book类和一个BookRepository，我们可以创建和检索Book，使用我们的[RedisTemplate](https://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/core/RedisTemplate.html)与Redis作为我们的后端进行交互：

```java
@Autowired
private RedisTemplate<Long, Book> redisTemplate;

public void save(Book book) {
    redisTemplate.opsForValue().set(book.getId(), book);
}

public Book findById(Long id) {
    return redisTemplate.opsForValue().get(id);
}
```

Lettuce默认会为我们管理序列化和反序列化，所以现在无需再做任何事情。不过，值得庆幸的是，这也可以进行配置。

另一个重要特性是**RedisTemplate是线程安全的**，因此它在[多线程环境](https://www.baeldung.com/java-thread-safety)中也能正常工作。

## 8. 总结

在本文中，我们配置了Spring Boot以通过Lettuce与Redis通信。我们通过一个Starter、一个@Bean配置和一些属性实现了它。

总而言之，我们使用RedisTemplate让Redis充当简单的后端。