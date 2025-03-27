---
layout: post
title:  无冲突复制数据类型简介
category: libraries
copyright: libraries
excerpt: CRDT
---

## 1. 概述

在本文中，我们将介绍无冲突复制数据类型(CRDT)以及如何在Java中使用它们。在我们的示例中，我们将使用[wurmloch-crdt](https://github.com/netopyr/wurmloch-crdt)库中的实现。

当我们在分布式系统中拥有一个由N个副本节点组成的集群时，我们可能会遇到**网络分区-某些节点暂时无法相互通信**，这种情况称为裂脑。

当我们的系统出现裂脑时，**一些写入请求(即使是同一个用户的请求)可能会转到彼此不相连的不同副本**。当这种情况发生时，我们的系统仍然可用，但不一致。

我们需要决定当两个分割集群之间的网络再次开始工作时如何处理不一致的写入和数据。

## 2. 无冲突复制数据类型来解决问题

考虑两个节点A和B，它们由于脑裂而断开连接。

假设一个用户更改了他的登录名，并且请求发送到节点A。然后他/她决定再次更改它，但这次请求发送到节点B。

由于脑裂，两个节点之间没有连接。我们需要决定当网络恢复正常时，该用户的登录信息应该是什么样子。

我们可以利用几种策略：我们可以为用户提供解决冲突的机会(就像在Google Docs中所做的那样)，或者我们**可以使用CRDT为我们合并来自不同副本的数据**。

## 3. Maven依赖

首先，让我们向库添加一个依赖，以提供一组有用的CRDT：

```xml
<dependency>
    <groupId>com.netopyr.wurmloch</groupId>
    <artifactId>wurmloch-crdt</artifactId>
    <version>0.1.0</version>
</dependency>
```

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/com.netopyr.wurmloch/wurmloch-crdt)上找到。

## 4. 仅生长集

**最基本的CRDT是只增长集，元素只能添加到GSet中，而不能被移除**。当GSet发散时，可以通过计算两个集合的并集轻松合并。

首先，让我们创建两个副本来模拟分布式数据结构，并使用connect()方法连接这两个副本：

```java
LocalCrdtStore crdtStore1 = new LocalCrdtStore();
LocalCrdtStore crdtStore2 = new LocalCrdtStore();
crdtStore1.connect(crdtStore2);
```

一旦我们在集群中获得两个副本，我们就可以在第一个副本上创建一个GSet，并在第二个副本上引用它：

```java
GSet<String> replica1 = crdtStore1.createGSet("ID_1");
GSet<String> replica2 = crdtStore2.<String>findGSet("ID_1").get();
```

此时，我们的集群按预期运行，并且两个副本之间存在活动连接。我们可以从两个不同的副本向集合添加两个元素，并断言集合在两个副本上都包含相同的元素：

```java
replica1.add("apple");
replica2.add("banana");

assertThat(replica1).contains("apple", "banana");
assertThat(replica2).contains("apple", "banana");
```

假设突然出现网络分区，并且第一个和第二个副本之间没有连接。我们可以使用disconnect()方法模拟网络分区：

```java
crdtStore1.disconnect(crdtStore2);
```

接下来，当我们从两个副本向数据集添加元素时，这些更改在全局是不可见的，因为它们之间没有联系：

```java
replica1.add("strawberry");
replica2.add("pear");

assertThat(replica1).contains("apple", "banana", "strawberry");
assertThat(replica2).contains("apple", "banana", "pear");
```

**一旦两个集群成员之间的连接再次建立，GSet就会使用两个集合上的联合在内部进行合并**，并且两个副本再次保持一致：

```java
crdtStore1.connect(crdtStore2);

assertThat(replica1)
    .contains("apple", "banana", "strawberry", "pear");
assertThat(replica2)
    .contains("apple", "banana", "strawberry", "pear");
```

## 5. 仅增量计数器

仅增量计数器是一种CRDT，它在每个节点本地聚合所有增量。

**当副本同步时，在网络分区之后，结果值是通过对所有节点上的所有增量求和来计算的-这类似于java.concurrent中的LongAdder，但抽象级别更高**。

让我们使用GCounter创建一个仅递增计数器，并从两个副本中递增它。我们可以看到总和计算正确：

```java
LocalCrdtStore crdtStore1 = new LocalCrdtStore();
LocalCrdtStore crdtStore2 = new LocalCrdtStore();
crdtStore1.connect(crdtStore2);

GCounter replica1 = crdtStore1.createGCounter("ID_1");
GCounter replica2 = crdtStore2.findGCounter("ID_1").get();

replica1.increment();
replica2.increment(2L);

assertThat(replica1.get()).isEqualTo(3L);
assertThat(replica2.get()).isEqualTo(3L);
```

当我们断开两个集群成员并执行本地增量操作时，我们可以看到值不一致：

```java
crdtStore1.disconnect(crdtStore2);

replica1.increment(3L);
replica2.increment(5L);

assertThat(replica1.get()).isEqualTo(6L);
assertThat(replica2.get()).isEqualTo(8L);复制
```

但是一旦集群再次恢复健康，增量将被合并，产生适当的值：

```java
crdtStore1.connect(crdtStore2);

assertThat(replica1.get())
    .isEqualTo(11L);
assertThat(replica2.get())
    .isEqualTo(11L);
```

## 6. PN计数器

使用与仅增量计数器类似的规则，我们可以创建一个既可以增量又可以减量的计数器，PNCounter分别存储所有增量和减量。

**当副本同步时，结果值将等于所有增量的总和减去所有减量的总和**：

```java
@Test
public void givenPNCounter_whenReplicasDiverge_thenMergesWithoutConflict() {
    LocalCrdtStore crdtStore1 = new LocalCrdtStore();
    LocalCrdtStore crdtStore2 = new LocalCrdtStore();
    crdtStore1.connect(crdtStore2);

    PNCounter replica1 = crdtStore1.createPNCounter("ID_1");
    PNCounter replica2 = crdtStore2.findPNCounter("ID_1").get();

    replica1.increment();
    replica2.decrement(2L);

    assertThat(replica1.get()).isEqualTo(-1L);
    assertThat(replica2.get()).isEqualTo(-1L);

    crdtStore1.disconnect(crdtStore2);

    replica1.decrement(3L);
    replica2.increment(5L);

    assertThat(replica1.get()).isEqualTo(-4L);
    assertThat(replica2.get()).isEqualTo(4L);

    crdtStore1.connect(crdtStore2);

    assertThat(replica1.get()).isEqualTo(1L);
    assertThat(replica2.get()).isEqualTo(1L);
}
```

## 7. 最后写入者获胜注册

有时，我们的业务规则会比较复杂，而对集合或计数器的操作是不够的。我们可以使用Last-Writer-Wins注册器，**它在合并分散的数据集时仅保留最后更新的值**。Cassandra使用此策略来解决冲突。

**我们在使用此策略时需要非常谨慎，因为它会丢失其间发生的变化**。

让我们创建一个由两个副本和LWWRegister类的实例组成的集群：

```java
LocalCrdtStore crdtStore1 = new LocalCrdtStore("N_1");
LocalCrdtStore crdtStore2 = new LocalCrdtStore("N_2");
crdtStore1.connect(crdtStore2);

LWWRegister<String> replica1 = crdtStore1.createLWWRegister("ID_1");
LWWRegister<String> replica2 = crdtStore2.<String>findLWWRegister("ID_1").get();

replica1.set("apple");
replica2.set("banana");

assertThat(replica1.get()).isEqualTo("banana");
assertThat(replica2.get()).isEqualTo("banana");
```

当第一个副本将值设置为apple而第二个副本将其更改为banana时， LWWRegister仅保留最后一个值。

让我们看看如果集群断开连接会发生什么：

```java
crdtStore1.disconnect(crdtStore2);

replica1.set("strawberry");
replica2.set("pear");

assertThat(replica1.get()).isEqualTo("strawberry");
assertThat(replica2.get()).isEqualTo("pear");
```

每个副本都会保留其不一致数据的本地副本。当我们调用set()方法时，LWWRegister会使用VectorClock算法在内部为每个副本分配一个特殊的版本值，该版本值用于标识特定更新。

**当集群同步时，它会采用版本最高的值并丢弃所有先前的更新**：

```java
crdtStore1.connect(crdtStore2);

assertThat(replica1.get()).isEqualTo("pear");
assertThat(replica2.get()).isEqualTo("pear");
```

## 8. 总结

在本文中，我们展示了分布式系统在保持可用性的同时的一致性问题。

如果出现网络分区，我们需要在集群同步时合并分散的数据，我们了解了如何使用CRDT执行分散数据的合并。