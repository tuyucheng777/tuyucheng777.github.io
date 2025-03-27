---
layout: post
title:  PCollections简介
category: libraries
copyright: libraries
excerpt: PCollections
---

## 1. 概述

在本文中，我们将介绍[PCollections](https://github.com/hrldcpr/pcollections)，**这是一个提供持久、不可变集合的Java库**。

[持久数据](https://en.wikipedia.org/wiki/Persistent_data_structure)结构(集合)在更新操作期间不能直接修改，而是返回包含更新操作结果的新对象。它们不仅是不可变的，而且是持久的-**这意味着在执行修改后，集合的先前版本保持不变**。

PCollections与Java Collections框架类似且兼容。

## 2. 依赖

让我们在pom.xml中添加以下依赖，以便在项目中使用PCollections：

```xml
<dependency>
    <groupId>org.pcollections</groupId>
    <artifactId>pcollections</artifactId>
    <version>2.1.2</version>
</dependency>
```

如果我们的项目是基于Gradle的，可以将相同的工件添加到我们的build.gradle文件中：

```groovy
compile 'org.pcollections:pcollections:2.1.2'
```

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/org.pcollections/pcollections)上找到。

## 3. Map结构(HashPMap)

HashPMap是一种持久性Map数据结构，它类似于java.util.HashMap，用于存储非空键值数据。

我们可以使用HashTreePMap中方便的静态方法来实例化HashPMap，这些静态方法返回由IntTreePMap支持的HashPMap实例。

HashTreePMap类的静态empty()方法创建一个没有元素的空HashPMap-就像使用java.util.HashMap的默认构造函数一样：

```java
HashPMap<String, String> pmap = HashTreePMap.empty();
```

我们可以使用另外两种静态方法来创建HashPMap，singleton()方法创建只有一个条目的HashPMap：

```java
HashPMap<String, String> pmap1 = HashTreePMap.singleton("key1", "value1");
assertEquals(pmap1.size(), 1);
```

from()方法从现有的java.util.HashMap实例(和其他java.util.Map实现)创建一个HashPMap：

```java
Map map = new HashMap();
map.put("mkey1", "mval1");
map.put("mkey2", "mval2");

HashPMap<String, String> pmap2 = HashTreePMap.from(map);
assertEquals(pmap2.size(), 2);
```

尽管HashPMap继承了java.util.AbstractMap和java.util.Map的一些方法，但它也有一些独有的方法。

minus()方法从Map中删除单个条目，而minusAll()方法删除多个条目。还有plus()和plusAll()方法分别添加单个和多个条目：

```java
HashPMap<String, String> pmap = HashTreePMap.empty();
HashPMap<String, String> pmap0 = pmap.plus("key1", "value1");

Map map = new HashMap();
map.put("key2", "val2");
map.put("key3", "val3");
HashPMap<String, String> pmap1 = pmap0.plusAll(map);

HashPMap<String, String> pmap2 = pmap1.minus("key1");

HashPMap<String, String> pmap3 = pmap2.minusAll(map.keySet());

assertEquals(pmap0.size(), 1);
assertEquals(pmap1.size(), 3);
assertFalse(pmap2.containsKey("key1"));
assertEquals(pmap3.size(), 0);
```

需要注意的是，在pmap上调用put()将抛出UnsupportedOperationException。由于PCollections对象是持久且不可变的，因此每个修改操作都会返回一个对象的新实例(HashPMap)。

让我们继续看看其他数据结构。

## 4. 列表结构(TreePVector和ConsPStack)

TreePVector是java.util.ArrayList的持久化类似物，而ConsPStack是java.util.LinkedList的类似物。TreePVector和ConsPStack具有方便的静态方法来创建新实例-就像HashPMap一样。

empty()方法创建一个空的TreePVector，而singleton()方法创建一个只有一个元素的TreePVector。还有from()方法可用于从任何java.util.Collection创建TreePVector实例。

ConsPStack具有相同名称的静态方法，可实现相同的目标。

TreePVector具有操作它的方法，它具有minus()和minusAll()方法用于删除元素；plus()和plusAll()方法用于添加元素。

with()用于替换指定索引处的元素，subList()从集合中获取一定范围内的元素。

这些方法在ConsPStack中也可用。

让我们考虑以下示例上述方法的代码片段：

```java
TreePVector pVector = TreePVector.empty();

TreePVector pV1 = pVector.plus("e1");
TreePVector pV2 = pV1.plusAll(Arrays.asList("e2", "e3", "e4"));
assertEquals(1, pV1.size());
assertEquals(4, pV2.size());

TreePVector pV3 = pV2.minus("e1");
TreePVector pV4 = pV3.minusAll(Arrays.asList("e2", "e3", "e4"));
assertEquals(pV3.size(), 3);
assertEquals(pV4.size(), 0);

TreePVector pSub = pV2.subList(0, 2);
assertTrue(pSub.contains("e1") && pSub.contains("e2"));

TreePVector pVW = (TreePVector) pV2.with(0, "e10");
assertEquals(pVW.get(0), "e10");
```

在上面的代码片段中，pSub是另一个TreePVector对象，与pV2无关。可以看出，pV2未被subList()操作改变；而是创建了一个新的TreePVector对象，并用pV2的元素从索引0到2进行填充。

这就是不变性的含义，也是PCollections所有修改方法的现状。

## 5. 集合结构(MapPSet)

MapPSet是持久的、基于Map的、与java.util.HashSet类似的类，它可以通过HashTreePSet的静态方法(empty()、from()和singleton())方便地实例化。它们的功能与前面示例中解释的相同。

MapPSet具有plus()、plusAll()、minus()和minusAll()方法来操作集合数据。此外，它还继承了java.util.Set、java.util.AbstractCollection和java.util.AbstractSet的方法：

```java
MapPSet pSet = HashTreePSet.empty()     
    .plusAll(Arrays.asList("e1","e2","e3","e4"));
assertEquals(pSet.size(), 4);

MapPSet pSet1 = pSet.minus("e4");
assertFalse(pSet1.contains("e4"));
```

最后，还有OrderedPSet–它像java.util.LinkedHashSet一样维护元素的插入顺序。

## 6. 总结

总之，在这个快速教程中，我们探索了PCollections-类似于Java中的核心集合的持久数据结构。当然，PCollections [Javadoc](http://www.javadoc.io/doc/org.pcollections/pcollections/2.1.2)提供了有关库复杂性的更多见解。