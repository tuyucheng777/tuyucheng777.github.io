---
layout: post
title:  MapDB指南
category: libraries
copyright: libraries
excerpt: MapDB
---

## 1. 简介

在本文中，我们将介绍**[MapDB](http://www.mapdb.org/)库-一种通过类似集合的API访问的嵌入式数据库引擎**。

我们首先探索有助于配置、打开和管理数据库的核心类DB和DBMaker。然后，我们将深入研究一些用于存储和检索数据的MapDB数据结构示例。

最后，在将MapDB与传统数据库和Java集合进行比较之前，我们将了解一些内存模式。

## 2. 在MapDB中存储数据

首先，让我们介绍一下我们将在整个教程中不断使用的两个类-DB和DBMaker。**DB类代表一个开放的数据库**，它的方法调用用于创建和关闭存储集合的操作以处理数据库记录，以及处理事务事件。

**DBMaker负责数据库的配置、创建和打开**。作为配置的一部分，我们可以选择将数据库托管在内存中或文件系统上。

### 2.1 一个简单的HashMap示例

为了了解其工作原理，让我们在内存中实例化一个新的数据库。

首先，让我们使用DBMaker类创建一个新的内存数据库：

```java
DB db = DBMaker.memoryDB().make();
```

一旦我们的DB对象启动并运行，我们就可以使用它来构建一个HTreeMap来处理我们的数据库记录：

```java
String welcomeMessageKey = "Welcome Message";
String welcomeMessageString = "Hello Tuyucheng!";

HTreeMap myMap = db.hashMap("myMap").createOrOpen();
myMap.put(welcomeMessageKey, welcomeMessageString);
```

HTreeMap是MapDB的HashMap实现。现在数据库中有了数据，我们可以使用get方法检索它：

```java
String welcomeMessageFromDB = (String) myMap.get(welcomeMessageKey);
assertEquals(welcomeMessageString, welcomeMessageFromDB);
```

最后，既然我们已经完成了数据库，我们应该关闭它以避免进一步的突变：

```java
db.close();
```

要将数据存储在文件中而不是内存中，我们需要做的就是改变DB对象的实例化方式：

```java
DB db = DBMaker.fileDB("file.db").make();
```

上面的示例未使用类型参数。因此，我们只能强制转换结果以使用特定类型。在下一个示例中，我们将引入Serializers以消除强制转换的需要。

### 2.2 集合

**MapDB包括不同的集合类型**。为了演示，让我们使用NavigableSet从我们的数据库中添加和检索一些数据，其工作方式与Java Set类似：

让我们从DB对象的简单实例开始：

```java
DB db = DBMaker.memoryDB().make();
```

接下来，创建我们的NavigableSet：

```java
NavigableSet<String> set = db
    .treeSet("mySet")
    .serializer(Serializer.STRING)
    .createOrOpen();
```

在这里，序列化器确保使用String对象对来自数据库的输入数据进行序列化和反序列化。

接下来，让我们添加一些数据：

```java
set.add("Tuyucheng");
set.add("is awesome");
```

现在，让我们检查两个不同的值是否已正确添加到数据库中：

```java
assertEquals(2, set.size());
```

最后，由于这是一个集合，让我们添加一个重复的字符串并验证我们的数据库仍然只包含两个值：

```java
set.add("Tuyucheng");

assertEquals(2, set.size());
```

### 2.3 事务

与传统数据库非常相似，**DB类提供了commit和rollback我们添加到数据库中的数据的方法**。

要启用此功能，我们需要使用transactionEnable方法初始化DB：

```java
DB db = DBMaker.memoryDB().transactionEnable().make();
```

接下来，让我们创建一个简单的集合，添加一些数据，并将其提交到数据库：

```java
NavigableSet<String> set = db
    .treeSet("mySet")
    .serializer(Serializer.STRING)
    .createOrOpen();

set.add("One");
set.add("Two");

db.commit();

assertEquals(2, set.size());
```

现在，让我们向数据库中添加第三个未提交的字符串：

```java
set.add("Three");

assertEquals(3, set.size());
```

如果我们对数据不满意，我们可以使用DB的rollback方法回滚数据：

```java
db.rollback();

assertEquals(2, set.size());
```

### 2.4 序列化器

MapDB提供了种类繁多的[序列化程序](https://jankotek.gitbooks.io/mapdb/content/htreemap/#serializers)，用于处理集合中的数据。最重要的构造参数是名称，它标识DB对象中的各个集合：

```java
HTreeMap<String, Long> map = db.hashMap("indentification_name")
    .keySerializer(Serializer.STRING)
    .valueSerializer(Serializer.LONG)
    .create();
```

虽然序列化是推荐的，但它是可选的，可以跳过。然而，值得注意的是，这会导致通用序列化过程变慢。

## 3. HTreeMap

**MapDB的HTreeMap提供了HashMap和HashSet集合来处理我们的数据库**。HTreeMap是一个分段哈希树，不使用固定大小的哈希表。相反，它使用自动扩展的索引树，并且不会随着表的增长而重新哈希所有数据。最重要的是，**HTreeMap是线程安全的并且支持使用多个段的并行写入**。

首先，让我们实例化一个简单的HashMap，它使用String作为键和值：

```java
DB db = DBMaker.memoryDB().make();

HTreeMap<String, String> hTreeMap = db
    .hashMap("myTreeMap")
    .keySerializer(Serializer.STRING)
    .valueSerializer(Serializer.STRING)
    .create();
```

上面，我们为键和值定义了单独的序列化器。现在我们的HashMap已创建，让我们使用put方法添加数据：

```java
hTreeMap.put("key1", "value1");
hTreeMap.put("key2", "value2");

assertEquals(2, hTreeMap.size());
```

由于HashMap作用于Object的hashCode方法，使用相同键添加数据会导致值被覆盖：

```java
hTreeMap.put("key1", "value3");

assertEquals(2, hTreeMap.size());
assertEquals("value3", hTreeMap.get("key1"));
```

## 4. SortedTableMap

**MapDB的SortedTableMap将键存储在固定大小的表中，并使用二分搜索进行检索。值得注意的是，一旦准备好，该Map就是只读的**。

让我们来看看创建和查询SortedTableMap的过程，我们将首先创建一个内存映射卷来保存数据，以及一个接收器来添加数据。在第一次调用我们的卷时，我们将只读标志设置为false，确保我们可以写入卷：

```java
String VOLUME_LOCATION = "sortedTableMapVol.db";

Volume vol = MappedFileVol.FACTORY.makeVolume(VOLUME_LOCATION, false);

SortedTableMap.Sink<Integer, String> sink = SortedTableMap.create(
    vol,
    Serializer.INTEGER,
    Serializer.STRING)
    .createFromSink();
```

接下来，我们将添加数据并调用接收器上的create方法来创建Map：

```java
for(int i = 0; i < 100; i++){
  sink.put(i, "Value " + Integer.toString(i));
}

sink.create();
```

现在我们的Map已经存在，我们可以定义一个只读卷并使用SortedTableMap的open方法打开我们的Map：

```java
Volume openVol = MappedFileVol.FACTORY.makeVolume(VOLUME_LOCATION, true);

SortedTableMap<Integer, String> sortedTableMap = SortedTableMap.open(
    openVol,
    Serializer.INTEGER,
    Serializer.STRING);

assertEquals(100, sortedTableMap.size());
```

### 4.1 二分搜索

在继续之前，让我们更详细地了解SortedTableMap如何利用二分搜索。

SortedTableMap将存储拆分为页面，每个页面包含多个由键和值组成的节点，这些节点内是我们在Java代码中定义的键值对。

SortedTableMap执行3次二分搜索来检索正确的值：

1.  每个页面的键都存储在堆上的数组中，SortedTableMap会执行二分搜索来找到正确的页面。
2.  接下来，对节点中的每个键进行解压缩，根据键，二分查找确定正确的节点。
3.  最后，SortedTableMap搜索节点内的键以找到正确的值。

## 5. 内存模式

**MapDB提供三种类型的内存存储**，让我们快速了解每种模式，了解其工作原理并研究其优势。

### 5.1 堆

**堆模式将对象存储在一个简单的Java Collection Map中，它不使用序列化，并且对于小型数据集可以非常快**。

但是，由于数据存储在堆上，因此数据集由垃圾回收(GC)管理。GC的持续时间会随着数据集的大小而增加，从而导致性能下降。

让我们看一个指定堆模式的示例：

```java
DB db = DBMaker.heapDB().make();
```

### 5.2 Byte[]

第二种存储类型基于字节数组，在这种模式下，**数据被序列化并存储到最大1MB的数组中**。虽然从技术上讲是在堆上，但这种方法对于垃圾收集来说更有效。

这是默认推荐的，并在我们的[“Hello Tuyucheng”示例](https://www.baeldung.com/mapdb#db)中使用：

```java
DB db = DBMaker.memoryDB().make();
```

### 5.3 DirectByteBuffer

最终存储基于DirectByteBuffer，Java 1.4中引入的直接内存允许将数据直接传递到本机内存而不是Java堆。因此，数据将完全存储在堆外。

我们可以使用以下方法调用这种类型的存储：

```java
DB db = DBMaker.memoryDirectDB().make();
```

## 6. 为什么选择MapDB？

那么，为什么要使用MapDB？

### 6.1 MapDB与传统数据库

MapDB提供了大量的数据库功能，只需几行Java代码即可配置。使用MapDB时，我们可以避免通常耗时的设置各种服务和连接来使我们的程序正常工作。

除此之外，MapDB还允许我们以熟悉的Java集合方式访问数据库的复杂性。使用MapDB，我们不需要SQL，只需使用简单的get方法调用即可访问记录。

### 6.2 MapDB与简单Java集合

一旦应用程序停止执行，Java集合将不会保留应用程序的数据。MapDB提供了一种简单、灵活、可插拔的服务，使我们能够快速轻松地保留应用程序中的数据，同时保持Java集合类型的实用性。

## 7. 总结

在本文中，我们深入探讨了MapDB的嵌入式数据库引擎和集合框架。

我们首先了解了用于配置、打开和管理数据库的核心类DB和DBMaker。然后，我们介绍了MapDB提供的一些用于处理记录的数据结构示例。最后，我们了解了MapDB相对于传统数据库或Java集合的优势。