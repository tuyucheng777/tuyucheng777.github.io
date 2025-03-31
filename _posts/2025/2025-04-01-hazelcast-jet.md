---
layout: post
title:  Hazelcast Jet简介
category: libraries
copyright: libraries
excerpt: Hazelcast Jet
---

## 1. 简介

在本教程中，我们将了解Hazelcast Jet。它是Hazelcast Inc提供的分布式数据处理引擎，建立在Hazelcast IMDG之上。

如果你想了解Hazelcast IMDG，[这里](https://www.baeldung.com/java-hazelcast)有一篇入门文章。

## 2. 什么是Hazelcast Jet？

Hazelcast Jet是一个分布式数据处理引擎，它将数据视为流。它可以处理存储在数据库或文件中的数据以及由Kafka服务器流式传输的数据。

此外，它还可以通过将数据流划分为子集并对每个子集应用聚合来对无限数据流执行聚合函数，此概念在Jet术语中称为窗口化。

我们可以在一组机器中部署Jet，然后将我们的数据处理作业提交给它。Jet将使集群的所有成员自动处理数据。集群的每个成员都会使用一部分数据，这使得它很容易扩展到任何级别的吞吐量。

以下是Hazelcast Jet的典型用例：

- 实时流处理
- 快速批处理
- 以分布式方式处理Java 8 Stream
- 微服务中的数据处理

## 3. 设置

要在我们的环境中设置Hazelcast Jet，我们只需要在pom.xml中添加一个Maven依赖。

以下是我们的做法：

```xml
<dependency>
    <groupId>com.hazelcast.jet</groupId>
    <artifactId>hazelcast-jet</artifactId>
    <version>4.2</version>
</dependency>
```

包含此依赖将下载一个10Mb的jar文件，它为我们提供了构建分布式数据处理管道所需的所有基础设施。

Hazelcast Jet的最新版本可以在[这里](https://mvnrepository.com/artifact/com.hazelcast.jet/hazelcast-jet)找到。

## 4. 示例应用程序

为了了解有关Hazelcast Jet的更多信息，我们将创建一个示例应用程序，该应用程序接收句子和在这些句子中查找的单词的输入，并返回这些句子中指定单词的数量。

### 4.1 管道

管道构成了Jet应用程序的基本结构，**管道内的处理遵循以下步骤**：

- 从源读取数据
- 转换数据
- 将数据写入接收器

对于我们的应用程序，管道将从分布式List中读取，应用分组和聚合的转换，最后写入分布式Map。

以下是我们编写管道的方式：

```java
private Pipeline createPipeLine() {
    Pipeline p = Pipeline.create();
    p.readFrom(Sources.<String>list(LIST_NAME))
        .flatMap(word -> traverseArray(word.toLowerCase().split("\\W+")))
        .filter(word -> !word.isEmpty())
        .groupingKey(wholeItem())
        .aggregate(counting())
        .writeTo(Sinks.map(MAP_NAME));
    return p;
}
```

从源读取后，我们将遍历数据并使用正则表达式按空格分割数据。之后，我们过滤掉空白。

最后，我们对单词进行分组、聚合并将结果写入Map。

### 4.2 作业

现在我们的管道已经定义好了，我们创建一个作业来执行管道。

以下是我们编写一个接收参数并返回计数的countWord函数的方法：

```java
public Long countWord(List<String> sentences, String word) {
    long count = 0;
    JetInstance jet = Jet.newJetInstance();
    try {
        List<String> textList = jet.getList(LIST_NAME);
        textList.addAll(sentences);
        Pipeline p = createPipeLine();
        jet.newJob(p).join();
        Map<String, Long> counts = jet.getMap(MAP_NAME);
        count = counts.get(word);
    } finally {
        Jet.shutdownAll();
    }
    return count;
}
```

我们首先创建一个Jet实例，以便创建我们的作业并使用管道。接下来，我们将输入列表复制到分布式列表，以便它可供所有实例使用。

然后，我们使用上面构建的管道提交作业。方法newJob()返回由Jet异步启动的可执行作业，join方法等待作业完成，如果作业完成时出现错误，则抛出异常。

当作业完成时，结果将在分布式Map中检索，正如我们在管道中定义的那样。因此，我们从Jet实例中获取Map，并获取针对它的单词计数。

最后，我们关闭Jet实例。执行结束后关闭它很重要，因为**Jet实例会启动自己的线程**。否则，即使我们的方法退出后，我们的Java进程仍将处于活跃状态。

下面是测试我们为Jet编写的代码的单元测试：

```java
@Test
public void whenGivenSentencesAndWord_ThenReturnCountOfWord() {
    List<String> sentences = new ArrayList<>();
    sentences.add("The first second was alright, but the second second was tough.");
    WordCounter wordCounter = new WordCounter();
    long countSecond = wordCounter.countWord(sentences, "second");
    assertEquals(3, countSecond);
}
```

## 5. 总结

在本文中，我们介绍了Hazelcast Jet。要了解有关它及其功能的更多信息，请参阅[手册](https://docs.hazelcast.com/imdg/4.2/)。