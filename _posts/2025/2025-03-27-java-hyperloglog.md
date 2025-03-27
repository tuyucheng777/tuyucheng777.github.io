---
layout: post
title:  Java中的HyperLogLog算法指南
category: libraries
copyright: libraries
excerpt: HyperLogLog
---

## 1. 概述

HyperLogLog(HLL)数据结构是一种用于估计数据集基数的概率数据结构。

假设我们有数百万用户，并且我们想要计算访问我们网页的不同次数。一个简单的实现方法是将每个唯一的用户ID存储在一个集合中，然后集合的大小就是我们的基数。

当我们处理非常大量的数据时，通过这种方式计算基数会非常低效，因为数据集会占用大量的内存。

但是，如果我们对百分之几以内的估计感到满意，并且不需要确切的唯一访问数量，那么我们可以使用HLL，因为它正是为这样的用例设计的-**估计数百万甚至数十亿个不同值的数量**。

## 2. Maven依赖

首先我们需要添加[hll](https://mvnrepository.com/artifact/net.agkn/hll)库的Maven依赖：

```xml
<dependency>
    <groupId>net.agkn</groupId>
    <artifactId>hll</artifactId>
    <version>1.6.0</version>
</dependency>
```

## 3. 使用HLL估计基数

HLL构造函数有两个参数，我们可以根据需要进行调整：

- log2m(以2为底的对数)：这是HLL内部使用的寄存器数量(注意我们指定了m)
- regwidth：这是每个寄存器使用的位数

如果我们想要更高的准确率，我们需要将这些参数设置为更高的值。这样的配置会产生额外的开销，因为我们的HLL将占用更多内存。如果我们对较低的准确率感到满意，我们可以降低这些参数，这样我们的HLL就会占用更少的内存。

让我们创建一个HLL来计算包含1亿个条目的数据集的不同值，我们将log2m参数设置为14，将regwidth设置为5–对于这种大小的数据集来说，这些值是合理的。

**当每个新元素插入到HLL时，需要事先对其进行哈希处理**。我们将使用Guava库(包含在hll依赖中)中的Hashing.murmur3_128()，因为它既准确又快速。

```java
HashFunction hashFunction = Hashing.murmur3_128();
long numberOfElements = 100_000_000;
long toleratedDifference = 1_000_000;
HLL hll = new HLL(14, 5);
```

选择这些参数应该可以让错误率低于1%(1,000,000个元素)，我们稍后会对此进行测试。

接下来我们插入1亿个元素：

```java
LongStream.range(0, numberOfElements).forEach(element -> {
        long hashedValue = hashFunction.newHasher().putLong(element).hash().asLong();
        hll.addRaw(hashedValue);
    }
);
```

最后，**我们可以测试HLL返回的基数是否在我们期望的错误阈值内**：

```java
long cardinality = hll.cardinality();
assertThat(cardinality)
    .isCloseTo(numberOfElements, Offset.offset(toleratedDifference));
```

## 4. HLL的内存大小

我们可以使用以下公式计算上一节中的HLL将占用多少内存：numberOfBits = 2 ^ log2m * regwidth。

在我们的示例中，该数字为2 ^ 14 * 5位(大约81000位或8100字节)。因此，使用HLL估算1亿个成员集的基数仅占用8100字节内存。

让我们将其与一个简单的集合实现进行比较。在这样的实现中，我们需要一个包含1亿个Long值的集合，这将占用100,000,000 * 8字节= 800,000,000字节。

我们可以看到差异非常大。使用HLL，我们只需要8100字节，而使用简单的Set实现，我们将需要大约800兆字节。

当我们考虑更大的数据集时，HLL和纯Set实现之间的差异变得更加大。

## 5. 两个HLL的合并

在执行并集时，HLL有一个有益的特性。当我们对两个由不同数据集创建的HLL进行并集并测量其基数时，**我们将获得与使用单个HLL并从头开始计算两个数据集的所有元素的哈希值时相同的并集错误阈值**。

请注意，当我们合并两个HLL时，两者都应该具有相同的log2m和regwidth参数才能产生正确的结果。

让我们通过创建两个HLL来测试该属性-一个填充了从0到1亿的值，另一个填充了从1亿到2亿的值：

```java
HashFunction hashFunction = Hashing.murmur3_128();
long numberOfElements = 100_000_000;
long toleratedDifference = 1_000_000;
HLL firstHll = new HLL(15, 5);
HLL secondHLL = new HLL(15, 5);

LongStream.range(0, numberOfElements).forEach(element -> {
    long hashedValue = hashFunction.newHasher()
        .putLong(element)
        .hash()
        .asLong();
    firstHll.addRaw(hashedValue);
    }
);

LongStream.range(numberOfElements, numberOfElements * 2).forEach(element -> {
    long hashedValue = hashFunction.newHasher()
        .putLong(element)
        .hash()
        .asLong();
    secondHLL.addRaw(hashedValue);
    }
);
```

请注意，我们调整了HLL的配置参数，将log2m参数从14(如上一节所示)增加到此示例中的15，因为生成的HLL联合将包含两倍的元素。

接下来，让我们使用union()方法将firstHll和secondHll合并。如你所见，估计的基数在错误阈值之内，就像我们从一个包含2亿个元素的HLL中获取基数一样：

```java
firstHll.union(secondHLL);
long cardinality = firstHll.cardinality();
assertThat(cardinality)
    .isCloseTo(numberOfElements * 2, Offset.offset(toleratedDifference * 2));
```

## 6. 总结

在本教程中，我们研究了HyperLogLog算法。

我们了解了如何使用HLL来估计集合的基数。我们还看到，与简单的解决方案相比，HLL非常节省空间。我们对两个HLL执行了并集运算，并验证了并集的行为与单个HLL相同。