---
layout: post
title:  Java中的Infinispan指南
category: libraries
copyright: libraries
excerpt: Infinispan
---

## 1. 概述

在本指南中，我们将介绍[Infinispan](http://infinispan.org/)，这是一种内存键/值数据存储，与同类其他工具相比，它具有更强大的功能。

为了了解它是如何工作的，我们将构建一个简单的项目，展示最常见的功能并检查如何使用它们。

## 2. 项目设置

为了能够以这种方式使用它，我们需要在pom.xml中添加它的依赖。

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/org.infinispan/infinispan-core)中找到：

```xml
<dependency>
    <groupId>org.infinispan</groupId>
    <artifactId>infinispan-core</artifactId>
    <version>9.1.5.Final</version>
</dependency>
```

从现在开始，所有必要的底层基础设施都将以编程方式处理。

## 3. CacheManager设置

CacheManager是我们将要使用的大多数功能的基础，它充当所有已声明缓存的容器，控制其生命周期，并负责全局配置。

Infinispan提供了一种非常简单的方法来构建CacheManager：

```java
public DefaultCacheManager cacheManager() {
    return new DefaultCacheManager();
}
```

现在我们可以使用它来构建我们的缓存了。

## 4. 缓存设置

缓存由名称和配置定义，可以使用类ConfigurationBuilder构建必要的配置，该类已在我们的类路径中可用。

为了测试我们的缓存，我们将构建一个模拟一些繁重查询的简单方法：

```java
public class HelloWorldRepository {
    public String getHelloWorld() {
        try {
            System.out.println("Executing some heavy query");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // ...
            e.printStackTrace();
        }
        return "Hello World!";
    }
}
```

此外，为了能够检查缓存中的变化，Infinispan提供了一个简单的注解@Listener。

在定义我们的缓存时，我们可以传递一些对其内部发生的任何事件感兴趣的对象，Infinispan将在处理缓存时通知它：

```java
@Listener
public class CacheListener {
    @CacheEntryCreated
    public void entryCreated(CacheEntryCreatedEvent<String, String> event) {
        this.printLog("Adding key '" + event.getKey()
                + "' to cache", event);
    }

    @CacheEntryExpired
    public void entryExpired(CacheEntryExpiredEvent<String, String> event) {
        this.printLog("Expiring key '" + event.getKey()
                + "' from cache", event);
    }

    @CacheEntryVisited
    public void entryVisited(CacheEntryVisitedEvent<String, String> event) {
        this.printLog("Key '" + event.getKey() + "' was visited", event);
    }

    @CacheEntryActivated
    public void entryActivated(CacheEntryActivatedEvent<String, String> event) {
        this.printLog("Activating key '" + event.getKey()
                + "' on cache", event);
    }

    @CacheEntryPassivated
    public void entryPassivated(CacheEntryPassivatedEvent<String, String> event) {
        this.printLog("Passivating key '" + event.getKey()
                + "' from cache", event);
    }

    @CacheEntryLoaded
    public void entryLoaded(CacheEntryLoadedEvent<String, String> event) {
        this.printLog("Loading key '" + event.getKey()
                + "' to cache", event);
    }

    @CacheEntriesEvicted
    public void entriesEvicted(CacheEntriesEvictedEvent<String, String> event) {
        StringBuilder builder = new StringBuilder();
        event.getEntries().forEach(
                (key, value) -> builder.append(key).append(", "));
        System.out.println("Evicting following entries from cache: "
                + builder.toString());
    }

    private void printLog(String log, CacheEntryEvent event) {
        if (!event.isPre()) {
            System.out.println(log);
        }
    }
}
```

在打印消息之前，我们会检查所通知的事件是否已经发生，因为对于某些事件类型，Infinispan会发送两次通知：一次在处理之前，一次在处理之后。

现在让我们构建一个方法来处理缓存的创建：

```java
private <K, V> Cache<K, V> buildCache(
        String cacheName,
        DefaultCacheManager cacheManager,
        CacheListener listener,
        Configuration configuration) {

    cacheManager.defineConfiguration(cacheName, configuration);
    Cache<K, V> cache = cacheManager.getCache(cacheName);
    cache.addListener(listener);
    return cache;
}
```

请注意我们如何将配置传递给CacheManager，然后使用相同的cacheName获取与所需缓存相对应的对象。还请注意我们如何将缓存对象本身通知给监听器。

我们现在将检查5种不同的缓存配置，并了解如何设置它们并充分利用它们。

### 4.1 简单缓存

最简单的缓存类型可以在一行中定义，使用我们的方法buildCache：

```java
public Cache<String, String> simpleHelloWorldCache(
        DefaultCacheManager cacheManager,
        CacheListener listener) {
    return this.buildCache(SIMPLE_HELLO_WORLD_CACHE,
            cacheManager, listener, new ConfigurationBuilder().build());
}
```

我们现在可以构建一个服务：

```java
public String findSimpleHelloWorld() {
    String cacheKey = "simple-hello";
    return simpleHelloWorldCache
            .computeIfAbsent(cacheKey, k -> repository.getHelloWorld());
}
```

注意我们如何使用缓存，首先检查所需条目是否已缓存。如果没有，我们需要调用我们的Repository，然后将其缓存。

让我们在测试中添加一个简单的方法来计时我们的方法：

```java
protected <T> long timeThis(Supplier<T> supplier) {
    long millis = System.currentTimeMillis();
    supplier.get();
    return System.currentTimeMillis() - millis;
}
```

通过测试，我们可以检查执行两个方法调用之间的时间：

```java
@Test
public void whenGetIsCalledTwoTimes_thenTheSecondShouldHitTheCache() {
    assertThat(timeThis(() -> helloWorldService.findSimpleHelloWorld()))
            .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(() -> helloWorldService.findSimpleHelloWorld()))
            .isLessThan(100);
}
```

### 4.2 过期缓存

我们可以定义一个缓存，其中所有条目都有生命周期，换句话说，元素将在给定时间段后从缓存中删除。配置非常简单：

```java
private Configuration expiringConfiguration() {
    return new ConfigurationBuilder().expiration()
            .lifespan(1, TimeUnit.SECONDS)
            .build();
}
```

现在我们使用上述配置构建缓存：

```java
public Cache<String, String> expiringHelloWorldCache(
        DefaultCacheManager cacheManager,
        CacheListener listener) {

    return this.buildCache(EXPIRING_HELLO_WORLD_CACHE,
            cacheManager, listener, expiringConfiguration());
}
```

最后，以类似的方法使用它，就像我们上面的简单缓存一样：

```java
public String findSimpleHelloWorldInExpiringCache() {
    String cacheKey = "simple-hello";
    String helloWorld = expiringHelloWorldCache.get(cacheKey);
    if (helloWorld == null) {
        helloWorld = repository.getHelloWorld();
        expiringHelloWorldCache.put(cacheKey, helloWorld);
    }
    return helloWorld;
}
```

我们再来测试一下时间：

```java
@Test
public void whenGetIsCalledTwoTimesQuickly_thenTheSecondShouldHitTheCache() {
    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
            .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
            .isLessThan(100);
}
```

运行它，我们可以看到缓存快速连续命中。为了展示过期时间与其条目放置时间有关，让我们在条目中强制执行它：

```java
@Test
public void whenGetIsCalledTwiceSparsely_thenNeitherHitsTheCache() throws InterruptedException {
    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
            .isGreaterThanOrEqualTo(1000);

    Thread.sleep(1100);

    assertThat(timeThis(() -> helloWorldService.findExpiringHelloWorld()))
            .isGreaterThanOrEqualTo(1000);
}
```

运行测试后，请注意在给定时间之后我们的条目从缓存中过期的情况。我们可以通过查看监听器打印的日志行来确认这一点：

```text
Executing some heavy query
Adding key 'simple-hello' to cache
Expiring key 'simple-hello' from cache
Executing some heavy query
Adding key 'simple-hello' to cache
```

请注意，当我们尝试访问条目时，该条目已过期。Infinispan在两个时刻检查过期条目：当我们尝试访问它时或当收割线程扫描缓存时。

即使在主配置中没有过期的缓存中，我们也可以使用过期。put方法接受更多参数：

```java
simpleHelloWorldCache.put(cacheKey, helloWorld, 10, TimeUnit.SECONDS);
```

或者，我们可以为条目指定最大空闲时间，而不是固定的生命周期：

```java
simpleHelloWorldCache.put(cacheKey, helloWorld, -1, TimeUnit.SECONDS, 10, TimeUnit.SECONDS);
```

将idleTime属性设置为-1，缓存将不会过期，但是当我们将其与10秒的空闲时间相结合时，我们会告诉Infinispan除非在此时间范围内访问此条目，否则将使该条目过期。

### 4.3 缓存驱逐

在Infinispan中，我们可以使用逐出配置来限制给定缓存中的条目数量：

```java
private Configuration evictingConfiguration() {
    return new ConfigurationBuilder()
            .memory().evictionType(EvictionType.COUNT).size(1)
            .build();
}
```

在这个例子中，我们将此缓存中的最大条目数限制为1，这意味着，如果我们尝试输入另一个条目，它将被从缓存中逐出。

再次，该方法与这里已经介绍的方法类似：

```java
public String findEvictingHelloWorld(String key) {
    String value = evictingHelloWorldCache.get(key);
    if(value == null) {
        value = repository.getHelloWorld();
        evictingHelloWorldCache.put(key, value);
    }
    return value;
}
```

让我们编写测试：

```java
@Test
public void whenTwoAreAdded_thenFirstShouldntBeAvailable() {
    assertThat(timeThis(
            () -> helloWorldService.findEvictingHelloWorld("key 1")))
            .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
            () -> helloWorldService.findEvictingHelloWorld("key 2")))
            .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
            () -> helloWorldService.findEvictingHelloWorld("key 1")))
            .isGreaterThanOrEqualTo(1000);
}
```

运行测试，我们可以查看监听器的活动日志：

```text
Executing some heavy query
Adding key 'key 1' to cache
Executing some heavy query
Evicting following entries from cache: key 1, 
Adding key 'key 2' to cache
Executing some heavy query
Evicting following entries from cache: key 2, 
Adding key 'key 1' to cache
```

检查当我们插入第二个键时第一个键是如何自动从缓存中删除的，然后第二个键也被删除以便再次为第一个键腾出空间。

### 4.4 钝化缓存

缓存钝化是Infinispan的强大功能之一。通过结合钝化和驱逐，我们可以创建一个不占用大量内存的缓存，而不会丢失信息。

让我们看一下钝化配置：

```java
private Configuration passivatingConfiguration() {
    return new ConfigurationBuilder()
            .memory().evictionType(EvictionType.COUNT).size(1)
            .persistence()
            .passivation(true)    // activating passivation
            .addSingleFileStore() // in a single file
            .purgeOnStartup(true) // clean the file on startup
            .location(System.getProperty("java.io.tmpdir"))
            .build();
}
```

我们再次强制缓存中只有一个条目，但告诉Infinispan钝化剩余的条目，而不是仅仅删除它们。

让我们看看当我们尝试填充多个条目时会发生什么：

```java
public String findPassivatingHelloWorld(String key) {
    return passivatingHelloWorldCache.computeIfAbsent(key, k -> repository.getHelloWorld());
}
```

让我们构建测试并运行它：

```java
@Test
public void whenTwoAreAdded_thenTheFirstShouldBeAvailable() {
    assertThat(timeThis(
            () -> helloWorldService.findPassivatingHelloWorld("key 1")))
            .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
            () -> helloWorldService.findPassivatingHelloWorld("key 2")))
            .isGreaterThanOrEqualTo(1000);

    assertThat(timeThis(
            () -> helloWorldService.findPassivatingHelloWorld("key 1")))
            .isLessThan(100);
}
```

现在让我们看看日志：

```text
Executing some heavy query
Adding key 'key 1' to cache
Executing some heavy query
Passivating key 'key 1' from cache
Evicting following entries from cache: key 1, 
Adding key 'key 2' to cache
Passivating key 'key 2' from cache
Evicting following entries from cache: key 2, 
Loading key 'key 1' to cache
Activating key 'key 1' on cache
Key 'key 1' was visited
```

请注意，为了使缓存只有一个条目，我们采取了多少步骤。另外，请注意步骤的顺序-钝化、逐出，然后加载，然后激活。让我们看看这些步骤的含义：

- **钝化**：我们的条目存储在另一个地方，远离Infinispan的主存储器(在本例中为内存)
- **逐出**：删除条目，释放内存并保留缓存中配置的最大条目数
- **加载**：当尝试访问我们的钝化条目时，Infinispan会检查其存储的内容并再次将条目加载到内存中
- **激活**：该条目现在可再次在Infinispan中访问

### 4.5 事务缓存

Infinispan附带强大的事务控制，与数据库对应方一样，当多个线程尝试写入同一条目时，它有助于维护完整性。

让我们看看如何定义具有事务功能的缓存：

```java
private Configuration transactionalConfiguration() {
    return new ConfigurationBuilder()
            .transaction().transactionMode(TransactionMode.TRANSACTIONAL)
            .lockingMode(LockingMode.PESSIMISTIC)
            .build();
}
```

为了能够测试它，让我们构建两种方法-一种可以快速完成事务，另一种需要一段时间：

```java
public Integer getQuickHowManyVisits() {
    TransactionManager tm = transactionalCache
            .getAdvancedCache().getTransactionManager();
    tm.begin();
    Integer howManyVisits = transactionalCache.get(KEY);
    howManyVisits++;
    System.out.println("I'll try to set HowManyVisits to " + howManyVisits);
    StopWatch watch = new StopWatch();
    watch.start();
    transactionalCache.put(KEY, howManyVisits);
    watch.stop();
    System.out.println("I was able to set HowManyVisits to " + howManyVisits +
            " after waiting " + watch.getTotalTimeSeconds() + " seconds");

    tm.commit();
    return howManyVisits;
}
```

```java
public void startBackgroundBatch() {
    TransactionManager tm = transactionalCache
            .getAdvancedCache().getTransactionManager();
    tm.begin();
    transactionalCache.put(KEY, 1000);
    System.out.println("HowManyVisits should now be 1000, " +
            "but we are holding the transaction");
    Thread.sleep(1000L);
    tm.rollback();
    System.out.println("The slow batch suffered a rollback");
}
```

现在让我们创建一个执行这两种方法的测试并检查Infinispan将如何表现：

```java
@Test
public void whenLockingAnEntry_thenItShouldBeInaccessible() throws InterruptedException {
    Runnable backGroundJob = () -> transactionalService.startBackgroundBatch();
    Thread backgroundThread = new Thread(backGroundJob);
    transactionalService.getQuickHowManyVisits();
    backgroundThread.start();
    Thread.sleep(100); //lets wait our thread warm up

    assertThat(timeThis(() -> transactionalService.getQuickHowManyVisits()))
            .isGreaterThan(500).isLessThan(1000);
}
```

执行它后，我们将再次在控制台中看到以下信息：

```text
Adding key 'key' to cache
Key 'key' was visited
Ill try to set HowManyVisits to 1
I was able to set HowManyVisits to 1 after waiting 0.001 seconds
HowManyVisits should now be 1000, but we are holding the transaction
Key 'key' was visited
Ill try to set HowManyVisits to 2
I was able to set HowManyVisits to 2 after waiting 0.902 seconds
The slow batch suffered a rollback
```

检查主线程上的时间，等待慢速方法创建的事务结束。

## 5. 总结

在本文中，我们了解了Infinispan是什么，以及它作为应用程序内的缓存的主要特性和能力。