---
layout: post
title:  JetCache简介
category: libraries
copyright: libraries
excerpt: JetCache
---

## 1. 简介

在本文中，我们将了解[JetCache](https://github.com/alibaba/jetcache)。我们将介绍它是什么、可以用它做什么以及如何使用它。

JetCache是一个缓存抽象库，我们可以在应用程序中的一系列缓存实现之上使用它。这使我们能够以与确切的缓存实现无关的方式编写代码，并允许我们随时更改实现而不会影响应用程序中的任何其他内容。

## 2. 依赖

**在使用JetCache之前，我们需要在我们的构建中包含[最新版本](https://mvnrepository.com/artifact/com.alicp.jetcache/jetcache-core)，在撰写本文时版本为[2.7.6](https://mvnrepository.com/artifact/com.alicp.jetcache/jetcache-core/2.7.6)**。

JetCache带有几个我们需要的依赖，具体取决于我们的确切需求，该功能的核心依赖位于com.alicp.jetcache:jetcache-core中。

如果我们使用Maven，我们可以将其包含在pom.xml中：

```xml
<dependency>
    <groupId>com.alicp.jetcache</groupId>
    <artifactId>jetcache-core</artifactId>
    <version>2.7.6</version>
</dependency>
```

如果核心库尚未包含任何实际的缓存实现，我们可能需要将其包含在内。无需任何额外的依赖，我们可以选择使用两种内存缓存-LinkedHashMapCache(基于标准java.util.LinkedHashMap构建)和CaffeineCache(基于[Caffeine缓存库](https://github.com/ben-manes/caffeine)构建)。

## 3. 手动使用缓存

一旦JetCache可用，我们就可以立即使用它来缓存数据。

### 3.1 创建缓存

**为此，我们首先需要创建缓存。当我们手动执行此操作时，我们需要确切知道要使用哪种类型的缓存，并使用适当的[构建器类](https://www.baeldung.com/creational-design-patterns#builder)**。例如，如果我们想使用LinkedHashMapCache，我们可以使用LinkedHashMapCacheBuilder构建一个：

```java
Cache<Integer, String> cache = LinkedHashMapCacheBuilder.createLinkedHashMapCacheBuilder()
    .limit(100)
    .expireAfterWrite(10, TimeUnit.SECONDS)
    .buildCache();
```

不同的缓存框架可能有不同的配置设置可用，但是，**一旦我们构建了缓存，JetCache就会将其公开为Cache<K, V\>对象，无论使用哪种底层框架**。这意味着我们可以更改缓存框架，唯一需要更改的是缓存的构造。例如，我们可以通过更改构建器将上面的缓存从LinkedHashMapCache替换为CaffeineCache：

```java
Cache<Integer, String> cache = CaffeineCacheBuilder.createCaffeineCacheBuilder()
    .limit(100)
    .expireAfterWrite(10, TimeUnit.SECONDS)
    .buildCache();
```

我们可以看到cache字段的类型和以前一样，与它的所有交互也和以前一样。

### 3.2 缓存和检索值

**一旦我们获得了缓存实例，我们就可以使用它来存储和检索值**。

最简单的方法是使用put()方法将值放入缓存，使用get()方法将值取回：

```java
cache.put(1, "Hello");
assertEquals("Hello", cache.get(1));
```

如果缓存中没有所需值，则使用get()方法将返回null。这意味着缓存从未缓存过提供的键，或者缓存因某种原因弹出了缓存条目-因为它已过期，或者因为缓存存储了太多其他值。

**如果需要，我们可以使用GET()方法来获取CacheGetResult<V\>对象。该对象永远不会为空，并且表示缓存值-包括缓存中没有值的原因**：

```java
// This was expired.
assertEquals(CacheResultCode.EXPIRED, cache.GET(1).getResultCode());

// This was never present.
assertEquals(CacheResultCode.NOT_EXISTS, cache.GET(2).getResultCode());
```

返回的确切结果代码将取决于所使用的底层缓存库。例如，CaffeineCache不支持指示条目已过期，因此在两种情况下它都会返回NOT_EXISTS，而其他缓存库可能会提供更好的响应。

如果需要，我们还可以使用remove()调用手动从缓存中清除一些内容：

```java
cache.remove(1);
```

我们可以利用这一点，通过删除不再需要的条目来避免弹出其他条目。

### 3.3 批量操作

**除了处理单个条目外，我们还可以批量缓存和提取条目。这完全符合我们的预期，只不过是针对适当的集合而不是单个值**。

批量缓存条目需要我们构建并传入具有相应值的Map<K, V\>，然后将其传递给putAll()方法来缓存条目：

```java
Map<Integer, String> putMap = new HashMap<>();
putMap.put(1, "One");
putMap.put(2, "Two");
putMap.put(3, "Three");
cache.putAll(putMap);
```

根据底层缓存实现，这可能与单独调用每个条目相同，但可能更高效。例如，如果我们使用Redis等远程缓存，那么这可能会减少所需的网络调用次数。

批量检索条目是通过调用getAll()方法并使用包含我们希望检索的键的Set<K\>来完成的；然后，这将返回一个Map<K, V\>，其中包含缓存中的所有条目。我们请求的任何不在缓存中的内容(例如，因为它从未被缓存或因为它已过期)都将在返回的Map中不存在：

```java
Map<Integer, String> values = cache.getAll(keys);
```

最后，我们可以使用removeAll()方法批量删除条目。与getAll()一样，我们为其提供一个要删除的键的Set<K\>，它将确保所有这些键都已被删除。

```java
cache.removeAll(keys);
```

## 4. Spring Boot集成

**手动创建和使用缓存很容易，但使用Spring Boot可以使事情变得更容易**。

### 4.1 设置

**JetCache附带一个[Spring Boot自动配置库](https://www.baeldung.com/spring-boot-custom-auto-configuration)，可以自动为我们设置一切**，我们只需将其包含在应用程序中，Spring Boot就会在启动时自动检测并加载它。为了使用它，我们需要将com.alicp.jetcache:jetcache-autoconfigure添加到我们的项目中。

如果我们使用Maven，我们可以将其包含在pom.xml中：

```xml
<dependency>
    <groupId>com.alicp.jetcache</groupId>
    <artifactId>jetcache-autoconfigure</artifactId>
    <version>2.7.6</version>
</dependency>
```

此外，JetCache附带了许多[Spring Boot Starters](https://www.baeldung.com/spring-boot-starters)，我们可以将它们包含在Spring Boot应用中，以帮助我们配置特定缓存。但是，只有当我们想要核心功能以外的功能时，这些才是必需的。

### 4.2 以编程方式使用缓存

**一旦我们添加了依赖，我们就可以在我们的应用程序中创建和使用缓存**。

将JetCache与Spring Boot结合使用会自动公开com.alicp.jetcache.CacheManager类型的Bean，这在概念上类似于Spring org.springframework.cache.CacheManager但专为JetCache使用而设计，我们可以使用它来创建缓存，而不必手动创建。这样做将有助于确保缓存正确地连接到Spring生命周期，并有助于定义一些全局属性，使我们的过程更轻松。

我们可以使用getOrCreateCache()方法创建一个新的缓存，并传递一些要使用的特定于缓存的配置：

```java
QuickConfig quickConfig = QuickConfig.newBuilder("testing")
    .cacheType(CacheType.LOCAL)
    .expire(Duration.ofSeconds(100))
    .build();
Cache<Integer, String> cache = cacheManager.getOrCreateCache(quickConfig);
```

我们可以在最合理的任何地方执行此操作-无论是在@Bean定义中，还是直接在组件中，或者在任何我们需要的地方。缓存是根据其名称注册的，因此我们可以直接从缓存管理器获取对它的引用，而无需根据需要创建Bean定义，但另一方面，创建Bean定义允许它们轻松自动装配。

**Spring Boot设置具有本地和远程缓存的概念**；本地缓存完全位于正在运行的应用程序的内存中-例如LinkedHashMapCache或CaffeineCache。远程缓存是应用程序所依赖的独立基础架构，例如Redis。

创建缓存时，我们可以指定需要本地缓存还是远程缓存。如果我们不指定，JetCache将同时创建本地缓存和远程缓存，并使用本地缓存代替远程缓存来提高性能。这种设置意味着我们可以享受共享缓存基础架构的好处，同时降低我们最近在应用程序中看到的数据的网络调用成本。

**一旦我们有了缓存实例，我们就可以像以前一样使用它。我们得到完全相同的类，并且支持所有相同的功能**。

### 4.3 缓存配置

这里需要注意的一点是，我们从未指定要创建的缓存类型。**当JetCache与Spring Boot集成时，我们可以按照标准的Spring Boot做法，使用application.properties文件指定一些常见的配置设置**。这完全是可选的，如果我们不这样做，那么大多数事情都有合理的默认值。

例如，使用LinkedHashMapCache作为本地缓存并使用[Redis和Lettuce](https://www.baeldung.com/java-redis-lettuce)作为远程缓存的配置可能如下所示：

```properties
jetcache.local.default.type=linkedhashmap
jetcache.remote.default.type=redis.lettuce
jetcache.remote.default.uri=redis://127.0.0.1:6379/
```

## 5. 方法级缓存

**除了我们已经看到的缓存标准用法之外，JetCache还支持在Spring Boot应用程序中包装整个方法并缓存结果**，我们通过标注要缓存结果的方法来实现这一点。

为了使用这个功能，我们首先需要启用它。我们在适当的配置类上使用@EnableMethodCache注解来执行此操作，包括我们要为其启用缓存的所有类所在的基本包名称：

```java
@Configuration
@EnableMethodCache(basePackages = "cn.tuyucheng.taketoday.jetcache")
public class Application {}
```

### 5.1 缓存方法结果

**此时，JetCache将自动在任何合适的注解方法上设置缓存**：

```java
@Cached
public String doSomething(int i) {
    // .....
}
```

如果我们对默认设置感到满意，则根本不需要注解参数-一个未命名的缓存，它具有本地和远程缓存，没有明确的到期配置，并使用缓存键的所有方法参数。这直接等同于：

```java
QuickConfig quickConfig = QuickConfig.newBuilder("c.t.t.j.a.AnnotationCacheUnitTest$TestService.doSomething(I)")
    .build();
cacheManager.getOrCreateCache(quickConfig);
```

请注意，即使我们没有指定缓存名称，我们也会有一个缓存名称。默认情况下，缓存名称是我们正在标注的完全限定方法签名。这有助于确保缓存不会意外发生冲突，因为每个方法签名在同一个JVM中必须是唯一的。

**此时，此方法的每次调用都将根据缓存键进行缓存，缓存键默认为整个方法参数集。如果我们随后使用相同的参数调用该方法并且我们有一个有效的缓存条目，那么它将立即返回，而无需调用该方法**。

然后，我们可以使用注解参数配置缓存，就像以编程方式配置缓存一样：

```java
@Cached(cacheType = CacheType.LOCAL, expire = 3600, timeUnit = TimeUnit.SECONDS, localLimit = 100)
```

在这种情况下，我们使用仅本地缓存，其中元素将在3600秒后过期，并且最多可存储100个元素。

此外，我们可以指定用于缓存键的方法参数。我们使用[SpEL](https://www.baeldung.com/spring-expression-language)表达式来准确描述键应该是什么：

```java
@Cached(name="userCache", key="#userId", expire = 3600)
User getUserById(long userId) {
    // .....
}
```

请注意，与SpEL一样，我们需要在编译代码时使用-parameters标志，以便按名称引用参数。如果没有，那么我们可以改用args[]数组按位置引用参数：

```java
@Cached(name="userCache", key="args[0]", expire = 3600)User getUserById(long userId) {// .....}
```

### 5.2 更新缓存条目

**除了使用@Cached注解来缓存方法的结果之外，我们还可以从其他方法更新已缓存的条目**。

最简单的情况是，由于方法调用而使缓存条目无效。我们可以使用@CacheInvalidate注解来实现这一点，这将需要使用与最初执行缓存的方法完全相同的缓存名称和缓存键进行配置，然后在调用时导致从缓存中删除相应的条目：

```java
@Cached(name="userCache", key="#userId", expire = 3600)
User getUserById(long userId) {
    // .....
}

@CacheInvalidate(name = "userCache", key = "#userId")
void deleteUserById(long userId) {
    // .....
}
```

我们还可以使用@CacheUpdate注解直接根据方法调用更新缓存条目，这是使用与以前完全相同的缓存名称和键进行配置的，但也使用表达式定义要存储在缓存中的值：

```java
@Cached(name="userCache", key="#userId", expire = 3600)
User getUserById(long userId) {
    // .....
}

@CacheUpdate(name = "userCache", key = "#user.userId", value = "#user")
void updateUser(User user) {
    // .....
}
```

这样做将始终调用标注的方法，但随后会将所需的值填充到缓存中。

## 6. 总结

在本文中，我们对JetCache进行了广泛的介绍。