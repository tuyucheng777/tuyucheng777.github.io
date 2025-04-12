---
layout: post
title:  Apache Beam简介
category: apache
copyright: apache
excerpt: Apache Beam
---

## 1. 概述

在本教程中，我们将介绍Apache Beam并探讨其基本概念。

我们将首先演示Apache Beam的用例和优势，然后介绍基础概念和术语。之后，我们将通过一个简单的示例来阐述Apache Beam的所有重要方面。

## 2. 什么是Apache Beam？

**Apache Beam(Batch + strEAM)是用于批量和流式数据处理作业的统一编程模型**，它提供了一个软件开发工具包，用于定义和构建数据处理管道以及执行这些管道的运行器。

**Apache Beam旨在提供可移植的编程层**，实际上，Beam管道运行器将数据处理管道转换为与用户所选后端兼容的API。目前，支持以下分布式处理后端：

- Apache Apex
- Apache Flink
- Apache Gearpump(孵化中)
- Apache Samza
- Apache Spark
- Google Cloud Dataflow
- Hazelcast Jet

## 3. 为什么选择Apache Beam？

**Apache Beam融合了批处理和流数据处理，而其他平台通常通过单独的API来实现**。因此，当需求发生变化时，可以非常轻松地将流处理更改为批处理，反之亦然。

**Apache Beam提升了可移植性和灵活性**，我们专注于逻辑，而非底层细节。此外，我们可以随时更改数据处理后端。

Apache Beam提供Java、Python、Go和Scala SDK，团队中的每个人都可以使用自己选择的语言。

## 4. 基本概念

使用Apache Beam，我们可以构建工作流图(管道)并执行它们。其编程模型中的关键概念如下：

- PCollection：表示一个数据集，可以是固定批次或数据流
- PTransform：一种数据处理操作，它接收一个或多个PCollection并输出0个或多个PCollection
- Pipeline：表示PCollection和PTransform的有向无环图，因此封装了整个数据处理作业
- PipelineRunner：在指定的分布式处理后端执行管道

简单来说，一个PipelineRunner执行一个Pipeline，一个Pipeline由PCollection和PTransform组成。

## 5. 字数统计示例

现在我们已经了解了Apache Beam的基本概念，让我们设计和测试一个字数统计任务。

### 5.1 构建Beam管道

设计工作流图是每个Apache Beam作业的第一步，让我们定义一个字数统计任务的步骤：

1. 从来源读取文本
2. 将文本拆分成单词列表
3. 将所有单词小写
4. 修剪标点符号
5. 过滤停用词
6. 计算每个唯一单词的数量

为了实现这一点，我们需要使用PCollection和PTransform抽象将上述步骤转换为单个管道。

### 5.2 依赖

在实现工作流图之前，我们应该将[Apache Beam](https://mvnrepository.com/artifact/org.apache.beam/beam-sdks-java-core)的核心依赖添加到我们的项目中：

```xml
<dependency>
    <groupId>org.apache.beam</groupId>
    <artifactId>beam-sdks-java-core</artifactId>
    <version>2.45.0</version>
</dependency>
```

Beam Pipeline Runners依赖分布式处理后端来执行任务，让我们添加[DirectRunner](https://mvnrepository.com/artifact/org.apache.beam/beam-runners-direct-java)作为运行时依赖：

```xml
<dependency>
    <groupId>org.apache.beam</groupId>
    <artifactId>beam-runners-direct-java</artifactId>
    <version>2.45.0</version>
    <scope>runtime</scope>
</dependency>
```

与其他Pipeline Runner不同，DirectRunner不需要任何额外的设置，这使其成为初学者的理想选择。

### 5.3 实现

Apache Beam采用Map-Reduce编程范式(与[Java Stream](https://www.baeldung.com/java-8-streams-introduction)相同)，实际上，在继续之前，最好先了解[reduce()](https://www.baeldung.com/java-stream-reduce)、[filter()](https://www.baeldung.com/java-stream-filter-lambda)、[count()](https://www.baeldung.com/java-stream-filter-count)、[map()和flatMap()](https://www.baeldung.com/java-difference-map-and-flatmap)的基本概念。

创建管道是我们要做的第一件事：

```java
PipelineOptions options = PipelineOptionsFactory.create();
Pipeline p = Pipeline.create(options);
```

现在我们应用字数统计任务：

```java
PCollection<KV<String, Long>> wordCount = p
        .apply("(1) Read all lines",
                TextIO.read().from(inputFilePath))
        .apply("(2) Flatmap to a list of words",
                FlatMapElements.into(TypeDescriptors.strings())
                        .via(line -> Arrays.asList(line.split("\\s"))))
        .apply("(3) Lowercase all",
                MapElements.into(TypeDescriptors.strings())
                        .via(word -> word.toLowerCase()))
        .apply("(4) Trim punctuations",
                MapElements.into(TypeDescriptors.strings())
                        .via(word -> trim(word)))
        .apply("(5) Filter stopwords",
                Filter.by(word -> !isStopWord(word)))
        .apply("(6) Count words",
                Count.perElement());
```

apply()的第一个(可选)参数是一个字符串，这只是为了提高代码的可读性，以下是上述代码中每个apply()函数的作用：

1. 首先，我们使用TextIO逐行读取输入文本文件
2. 我们将每一行用空格分开，然后将其平面映射到一个单词列表
3. 字数不区分大小写，因此我们将所有单词都小写
4. 之前，我们用空格分割行，最后得到像“word!”和“word?”这样的单词，所以我们删除了标点符号
5. 诸如“is”和“by”之类的停用词在几乎每个英文文本中都很常见，因此我们将其删除
6. 最后，我们使用内置函数Count.perElement()来计算唯一单词的数量

如前所述，管道是在分布式后端上处理的，由于PCollection分布在多个后端上，因此无法在内存中对其进行迭代。因此，我们将结果写入外部数据库或文件。

首先，我们将PCollection转换为String。然后，我们使用TextIO写入输出：

```java
wordCount.apply(MapElements.into(TypeDescriptors.strings())
    .via(count -> count.getKey() + " --> " + count.getValue()))
    .apply(TextIO.write().to(outputFilePath));
```

现在我们的管道定义已经完成，我们可以运行并测试它。

### 5.4 运行和测试

到目前为止，我们已经为字数统计任务定义了一个Pipeline。现在，让我们运行这个Pipeline：

```java
p.run().waitUntilFinish();
```

在这行代码中，Apache Beam会将我们的任务发送到多个DirectRunner实例，因此，最终会生成多个输出文件，它们包含以下内容：

```text
...
apache --> 3
beam --> 5
rocks --> 2
...
```

在Apache Beam中定义和运行分布式作业就是这样简单且富有表现力，相比之下，[Apache Spark](https://www.baeldung.com/apache-spark)、[Apache Flink](https://www.baeldung.com/apache-flink)和[Hazelcast Jet](https://www.baeldung.com/hazelcast-jet)上也提供了字数统计实现。

## 6. 接下来是什么

我们成功地统计了输入文件中每个单词的出现频率，但还没有得到最常见单词的报告。当然，下一步，对PCollection进行排序是一个值得解决的问题。

然后，我们可以了解更多关于窗口、触发器、指标以及更复杂的转换的知识。[Apache Beam文档](https://beam.apache.org/documentation/)提供了深入的信息和参考资料。

## 7. 总结

在本教程中，我们了解了Apache Beam是什么，以及为什么它比其他替代方案更受欢迎；我们还通过一个字数统计示例演示了Apache Beam的基本概念。