---
layout: post
title:  使用Java-LSH在Java中进行局部敏感哈希处理
category: libraries
copyright: libraries
excerpt: Java-LSH
---

## 1. 概述

[Locality-Sensitive Hashing(LSH)](https://en.wikipedia.org/wiki/Locality-sensitive_hashing)算法对输入项进行哈希处理，以便相似的项很有可能被映射到相同的桶中。

在这篇快速文章中，我们将使用java-lsh库来演示该算法的一个简单用例。

## 2. Maven依赖

首先我们需要将[java-lsh](https://mvnrepository.com/artifact/info.debatty/java-lsh)库Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>info.debatty</groupId>
    <artifactId>java-lsh</artifactId>
    <version>0.10</version>
</dependency>
```

## 3. 局部敏感哈希用例

LSH有许多可能的应用，但我们将考虑一个特定的例子。

**假设我们有一个文档数据库，并想要实现一个能够识别类似文档的搜索引擎**。

我们可以使用LSH作为这个解决方案的一部分：

-   每个文档都可以转换为数字或布尔值的向量-例如，我们可以使用[word2vect](https://en.wikipedia.org/wiki/Word2vec)算法将单词和文档转换为数字向量
-   一旦我们有了代表每个文档的向量，我们就可以使用LSH算法为每个向量计算哈希值，并且由于LSH的特性，呈现为相似向量的文档将具有相似或相同的哈希值
-   因此，给定一个特定文档的向量，我们可以找到N个具有相似哈希值的向量，并将相应的文档返回给最终用户

## 4. 例子

我们将使用java-lsh库来计算输入向量的哈希值，我们不会介绍转换本身，因为这是一个超出本文范围的庞大主题。

但是，假设我们有三个输入向量，它们是由一组三个文档转换而来的，以可用作LSH算法输入的形式呈现：

```java
boolean[] vector1 = new boolean[] {true, true, true, true, true};
boolean[] vector2 = new boolean[] {false, false, false, true, false};
boolean[] vector3 = new boolean[] {false, false, true, true, false};
```

请注意，**在生产应用程序中，输入向量的数量应该多得多才能利用LSH算法**，但为了演示，我们将只使用三个向量。

重要的是要注意第一个向量与第二个和第三个向量有很大不同，而第二个和第三个向量彼此非常相似。

让我们创建一个LSHMinHash类的实例，我们需要将输入向量的大小传递给它-所有输入向量的大小应该相等。我们还需要指定我们想要多少个哈希桶以及LSH应该执行多少个计算阶段(迭代)：

```java
int sizeOfVectors = 5;
int numberOfBuckets = 10;
int stages = 4;

LSHMinHash lsh = new LSHMinHash(stages, numberOfBuckets, sizeOfVectors);
```

我们指定所有将由算法进行哈希处理的向量都应在十个存储桶中进行哈希处理，我们还希望使用四次LSH迭代来计算哈希值。

为了计算每个向量的哈希值，我们将向量传递给hash()方法：

```java
int[] firstHash = lsh.hash(vector1);
int[] secondHash = lsh.hash(vector2);
int[] thirdHash = lsh.hash(vector3);

System.out.println(Arrays.toString(firstHash));
System.out.println(Arrays.toString(secondHash));
System.out.println(Arrays.toString(thirdHash));
```

运行该代码将产生类似于以下内容的输出：

```text
[0, 0, 1, 0]
[9, 3, 9, 8]
[1, 7, 8, 8]
```

查看每个输出数组，我们可以看到在四次迭代中的每一次迭代中为相应的输入向量计算的哈希值。第一行显示第一个向量的哈希结果，第二行显示第二个向量的哈希结果，第三行显示第三个向量的哈希结果。

经过四次迭代后，LSH产生了我们预期的结果-LSH为第二个和第三个向量计算了相同的哈希值(8)，它们彼此相似，而为第一个向量计算了不同的哈希值(0)，这是不同于第二个和第三个向量。

LSH是一种基于概率的算法，因此我们不能确定两个相似的向量会落在同一个哈希桶中。但是，当我们有足够多的输入向量时，**该算法产生的结果很可能会将相似的向量分配给相同的桶**。

当我们处理海量数据集时，LSH可以成为一种方便的算法。

## 5. 总结

在这篇简短的文章中，我们了解了局部敏感哈希算法的应用，并展示了如何在java-lsh库的帮助下使用它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tu-yucheng/taketoday-tutorial4j/tree/master/opensource-libraries/libraries-1)上获得。