---
layout: post
title:  Apache Curator简介
category: apache
copyright: apache
excerpt: Apache Curator
---

## 1. 简介

[Apache Curator](https://curator.apache.org/)是[Apache Zookeeper](https://zookeeper.apache.org/)的Java客户端，Apache Zookeeper是流行的分布式应用程序协调服务。

在本教程中，我们将介绍Curator提供的一些最相关的功能：

- 连接管理：管理连接和重试策略
- 异步：通过添加异步功能和使用Java 8 Lambda表达式来增强现有客户端
- 配置管理：对系统进行集中配置
- 强类型模型：使用类型模型
- 方案：实现领导者选举、分布式锁或计数器

## 2. 先决条件

首先，建议快速了解一下[Apache Zookeeper](https://zookeeper.apache.org/)及其功能。

对于本教程，我们假设127.0.0.1:2181上已经有一个独立的Zookeeper实例在运行；如果你刚刚开始，[这里](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)有关于如何安装和运行它的说明。

首先，我们需要将[curator-x-async](https://mvnrepository.com/artifact/org.apache.curator/curator-x-async)依赖添加的pom.xml中：

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-x-async</artifactId>
    <version>4.0.1</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**Apache Curator 4.XX的最新版本与Zookeeper 3.5.X有硬依赖关系**，后者目前仍处于测试阶段。

因此，在本文中，我们将使用当前最新稳定的[Zookeeper 3.4.11](https://zookeeper.apache.org/doc/r3.4.11/index.html)。

因此我们需要排除Zookeeper依赖，并将[我们的Zookeeper版本的依赖](https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
</dependency>
```

有关兼容性的更多信息，请参阅[此链接](https://curator.apache.org/docs/zk-compatibility-34)。

## 3. 连接管理

**Apache Curator的基本用例是连接到正在运行的Apache Zookeeper实例**。

该工具提供了一个工厂，使用重试策略来建立与Zookeeper的连接：

```java
int sleepMsBetweenRetries = 100;
int maxRetries = 3;
RetryPolicy retryPolicy = new RetryNTimes(maxRetries, sleepMsBetweenRetries);

CuratorFramework client = CuratorFrameworkFactory
    .newClient("127.0.0.1:2181", retryPolicy);
client.start();
 
assertThat(client.checkExists().forPath("/")).isNotNull();
```

在这个简单的例子中，我们将重试3次，并且在出现连接问题时每次重试之间等待100毫秒。

一旦使用CuratorFramework客户端连接到Zookeeper，我们现在就可以浏览路径、获取/设置数据并与服务器进行交互。

## 4. 异步

**Curator Async模块包装上述CuratorFramework客户端，以使用[CompletionStage Java 8 API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletionStage.html)提供非阻塞功能**。

让我们看看前面的示例使用Async包装器是什么样子的：

```java
int sleepMsBetweenRetries = 100;
int maxRetries = 3;
RetryPolicy retryPolicy = new RetryNTimes(maxRetries, sleepMsBetweenRetries);

CuratorFramework client = CuratorFrameworkFactory
    .newClient("127.0.0.1:2181", retryPolicy);

client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);

AtomicBoolean exists = new AtomicBoolean(false);

async.checkExists()
    .forPath("/")
    .thenAcceptAsync(s -> exists.set(s != null));

await().until(() -> assertThat(exists.get()).isTrue());
```

现在，checkExists()操作以异步模式运行，不会阻塞主线程。我们也可以使用thenAcceptAsync()方法(该方法使用了[CompletionStage API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletionStage.html))来串联执行这些操作。

## 5. 配置管理

在分布式环境中，最常见的挑战之一是管理多个应用程序之间的共享配置，**我们可以使用Zookeeper作为数据存储来保存配置**。

让我们看一个使用Apache Curator获取和设置数据的示例：

```java
CuratorFramework client = newClient();
client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);
String key = getKey();
String expected = "my_value";

client.create().forPath(key);

async.setData()
    .forPath(key, expected.getBytes());

AtomicBoolean isEquals = new AtomicBoolean();
async.getData()
    .forPath(key)
    .thenAccept(data -> isEquals.set(new String(data).equals(expected)));

await().until(() -> assertThat(isEquals.get()).isTrue());
```

在此示例中，我们创建节点路径，在Zookeeper中设置数据，然后恢复该路径并检查值是否相同；key字段可以是类似/config/dev/my_key的节点路径。

### 5.1 观察者

Zookeeper的另一个有趣功能是能够监视键或节点，**它允许我们监听配置的变化并更新应用程序，而无需重新部署**。

让我们看看上面的例子在使用观察者时是什么样子的：

```java
CuratorFramework client = newClient()
client.start();
AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);
String key = getKey();
String expected = "my_value";

async.create().forPath(key);

List<String> changes = new ArrayList<>();

async.watched()
    .getData()
    .forPath(key)
    .event()
    .thenAccept(watchedEvent -> {
        try {
            changes.add(new String(client.getData()
                .forPath(watchedEvent.getPath())));
        } catch (Exception e) {
            // fail ...
        }});

// Set data value for our key
async.setData()
    .forPath(key, expected.getBytes());

await()
    .until(() -> assertThat(changes.size()).isEqualTo(1));
```

我们配置观察器，设置数据，然后确认观察事件已触发；我们可以一次观察一个节点或一组节点。

## 6. 强类型模型

Zookeeper主要处理字节数组，因此我们需要对数据进行序列化和反序列化，这让我们能够灵活地处理任何可序列化实例，但维护起来可能比较困难。

为了解决这个问题，Curator添加了[类型模型](https://curator.apache.org/docs/modeled)的概念，**它可以委托序列化/反序列化，并允许我们直接使用我们的类型**。让我们看看它是如何工作的。

首先，我们需要一个序列化器框架，Curator建议使用Jackson实现，因此我们将[Jackson](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

现在，让我们尝试持久化我们的自定义类HostConfig：

```java
public class HostConfig {
    private String hostname;
    private int port;

    // getters and setters
}
```

我们需要提供从HostConfig类到路径的模型规范映射，并使用Apache Curator提供的模型框架包装器：

```java
ModelSpec<HostConfig> mySpec = ModelSpec.builder(
    ZPath.parseWithIds("/config/dev"), 
    JacksonModelSerializer.build(HostConfig.class))
    .build();

CuratorFramework client = newClient();
client.start();

AsyncCuratorFramework async = AsyncCuratorFramework.wrap(client);
ModeledFramework<HostConfig> modeledClient = ModeledFramework.wrap(async, mySpec);

modeledClient.set(new HostConfig("host-name", 8080));

modeledClient.read()
    .whenComplete((value, e) -> {
       if (e != null) {
            fail("Cannot read host config", e);
       } else {
            assertThat(value).isNotNull();
            assertThat(value.getHostname()).isEqualTo("host-name");
            assertThat(value.getPort()).isEqualTo(8080);
       }
     });
```

whenComplete()方法在读取路径/config/dev的时候会返回Zookeeper中的HostConfig实例。

## 7. Cookbook

**Zookeeper提供了[此指南](https://zookeeper.apache.org/doc/current/recipes.html)来实现高级解决方案或方案，例如领导者选举、分布式锁或共享计数器**。

Apache Curator为大多数此类配方提供了实现，要查看完整列表，请访问[Curator文档](https://curator.apache.org/docs/recipes)。

所有这些功能都可以在单独的模块中找到：

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.0.1</version>
</dependency>
```

让我们直接开始通过一些简单的例子来理解这些。

### 7.1 领导者选举

在分布式环境中，我们可能需要一个主节点或领导节点来协调复杂的工作。

Curator中[Leader Election](https://curator.apache.org/docs/recipes-leader-election)的使用方法如下：

```java
CuratorFramework client = newClient();
client.start();
LeaderSelector leaderSelector = new LeaderSelector(client, 
    "/mutex/select/leader/for/job/A", 
    new LeaderSelectorListener() {
        @Override
        public void stateChanged(
          CuratorFramework client, 
          ConnectionState newState) {
        }
    
        @Override
        public void takeLeadership(
          CuratorFramework client) throws Exception {
        }
    });

// join the members group
leaderSelector.start();

// wait until the job A is done among all members
leaderSelector.close();
```

当我们启动领导者选择器时，我们的节点会加入路径/mutex/select/leader/for/job/A内的成员组；一旦我们的节点成为领导者，就会调用takeLeadership方法，然后我们作为领导者就可以恢复工作。

### 7.2 共享锁

[共享锁](https://curator.apache.org/docs/recipes-shared-lock)的配方是关于拥有一个完全分布式的锁：

```java
CuratorFramework client = newClient();
client.start();
InterProcessSemaphoreMutex sharedLock = new InterProcessSemaphoreMutex(client, "/mutex/process/A");

sharedLock.acquire();

// do process A

sharedLock.release();
```

当我们获取锁时，Zookeeper会确保没有其他应用程序同时获取相同的锁。

### 7.3 计数器

[计数器](https://curator.apache.org/docs/recipes-shared-counter)协调所有客户端之间的共享整数：

```java
CuratorFramework client = newClient();
client.start();

SharedCount counter = new SharedCount(client, "/counters/A", 0);
counter.start();

counter.setCount(counter.getCount() + 1);

assertThat(counter.getCount()).isEqualTo(1);
```

在这个例子中，Zookeeper将整数值存储在路径/counters/A中，如果该路径尚未创建，则将该值初始化为0。

## 8. 总结

在本文中，我们了解了如何使用Apache Curator连接到Apache Zookeeper并利用其主要功能。