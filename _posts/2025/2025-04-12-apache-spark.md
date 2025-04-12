---
layout: post
title:  Apache Spark简介
category: apache
copyright: apache
excerpt: Apache Spark
---

## 1. 简介

**[Apache Spark](https://spark.apache.org/)是一个开源集群计算框架**，它为Scala、Java、Python和R提供了优雅的开发API，允许开发人员跨各种数据源(包括HDFS、Cassandra、HBase、S3等)执行各种数据密集型工作负载。

历史上，Hadoop的MapReduce被证明在某些迭代和交互式计算作业中效率低下，这最终导致了Spark的开发。**使用Spark，我们可以在内存中比Hadoop快两个数量级运行逻辑，在磁盘上则快一个数量级**。

## 2. Spark架构

Spark应用程序作为独立的进程集在集群上运行，如[下图](https://spark.apache.org/docs/latest/cluster-overview.html)所示：

![](/assets/images/2025/apache/apachespark01.png)

这些进程由主程序(称为驱动程序)中的SparkContext对象协调，SparkContext连接到多种类型的集群管理器(Spark自身的独立集群管理器、Mesos或YARN)，这些集群管理器负责跨应用程序分配资源。

一旦连接，Spark就会获取集群中节点上的执行器，这些执行器是运行计算并为你的应用程序存储数据的进程。

接下来，它将你的应用程序代码(由传递给SparkContext的JAR或Python文件定义)发送给执行器。最后，**SparkContext将任务发送给执行器运行**。

## 3. 核心组件

[下图](https://intellipaat.com/tutorial/spark-tutorial/apache-spark-components/)清晰地展示了Spark的不同组件：

![](/assets/images/2025/apache/apachespark02.png)

### 3.1 Spark核心

Spark Core组件负责所有基本的I/O功能、调度和监控Spark集群上的作业、任务调度、与不同存储系统的网络连接、故障恢复以及高效的内存管理。

与Hadoop不同，Spark通过使用一种称为RDD(弹性分布式数据集)的特殊数据结构来避免将共享数据存储在Amazon S3或HDFS等中间存储中。

**弹性分布式数据集是不可变的，是可以并行操作的分区记录集合，并且允许容错的“内存中”计算**。

RDD支持两种操作：

- 转换：Spark RDD转换是一个从现有RDD生成新RDD的函数，**转换函数以RDD作为输入，并生成一个或多个RDD作为输出**。转换本质上是惰性的，也就是说，当我们调用某个操作时，转换才会执行。

- 动作：转换操作会相互创建RDD，但当我们想要处理实际数据集时，动作就会被执行。因此，**动作是Spark RDD操作，它返回非RDD值**；动作的值存储在驱动程序或外部存储系统中。

动作是执行器向驱动程序发送数据的方式之一。

执行器是负责执行任务的代理。驱动程序是一个JVM进程，负责协调工作器和任务的执行，Spark的一些操作包括计数和收集。

### 3.2 Spark SQL

Spark SQL是一个用于结构化数据处理的Spark模块，它主要用于执行SQL查询。DataFrame构成了Spark SQL的主要抽象，按命名列排序的分布式数据集合在Spark中称为DataFrame。

Spark SQL支持从Hive、Avro、Parquet、ORC、JSON和JDBC等不同来源获取数据，它还可以使用Spark引擎扩展到数千个节点和数小时的查询，从而提供完整的查询中期容错能力。

### 3.3 Spark Streaming

Spark Streaming是核心Spark API的扩展，支持对实时数据流进行可扩展、高吞吐量、容错的流处理。数据可以从多个来源提取，例如Kafka、Flume、Kinesis或TCP套接字。

最后，处理后的数据可以推送到文件系统、数据库和实时仪表板。

### 3.4 Spark Mlib

MLlib是Spark的机器学习(ML)库，它的目标是使实用的机器学习变得可扩展且简单易用。概括地说，它提供以下工具：

- ML算法：常见的学习算法，如分类、回归、聚类和协同过滤
- 特征化：特征提取、变换、降维和选择
- 管道：用于构建、评估和调整ML管道的工具
- 持久化：保存和加载算法、模型和管道
- 实用程序：线性代数、统计、数据处理等

### 3.5 Spark GraphX

**GraphX是一个用于图和图并行计算的组件**，从高层次上讲，GraphX通过引入新的图抽象来扩展Spark RDD：一个有向多重图，其每个顶点和边都附加了属性。

为了支持图形计算，GraphX公开了一组基本运算符(例如，subgraph、joinVertices和aggregateMessages)。

此外，GraphX还包含越来越多的图形算法和构建器，以简化图形分析任务。

## 4. Spark中的“Hello World”

现在我们了解了核心组件，我们可以继续进行简单的基于Maven的Spark项目-用于计算字数。

我们将演示Spark在本地模式下的运行，其中所有组件都在同一台机器上本地运行，它是主节点、执行节点或Spark的独立集群管理器。

### 4.1 Maven设置

让我们在pom.xml文件中设置一个具有[Spark相关依赖](https://mvnrepository.com/artifact/org.apache.spark/spark-core)的Java Maven项目：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
	<artifactId>spark-core_2.10</artifactId>
	<version>1.6.0</version>
    </dependency>
</dependencies>
```

### 4.2 字数统计–Spark作业

现在让我们编写Spark作业来处理包含句子的文件并输出文件中的不同单词及其计数：

```java
public static void main(String[] args) throws Exception {
    if (args.length < 1) {
        System.err.println("Usage: JavaWordCount <file>");
        System.exit(1);
    }
    SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount");
    JavaSparkContext ctx = new JavaSparkContext(sparkConf);
    JavaRDD<String> lines = ctx.textFile(args[0], 1);

    JavaRDD<String> words
            = lines.flatMap(s -> Arrays.asList(SPACE.split(s)).iterator());
    JavaPairRDD<String, Integer> ones
            = words.mapToPair(word -> new Tuple2<>(word, 1));
    JavaPairRDD<String, Integer> counts
            = ones.reduceByKey((Integer i1, Integer i2) -> i1 + i2);

    List<Tuple2<String, Integer>> output = counts.collect();
    for (Tuple2<?, ?> tuple : output) {
        System.out.println(tuple._1() + ": " + tuple._2());
    }
    ctx.stop();
}
```

请注意，我们将本地文本文件的路径作为参数传递给Spark作业。

SparkContext对象是Spark的主要入口点，代表与已运行Spark集群的连接。它使用SparkConf对象来描述应用程序配置，SparkContext用于将内存中的文本文件读取为JavaRDD对象。

接下来，我们使用flatmap方法将行JavaRDD对象转换为单词JavaRDD对象，首先将每一行转换为空格分隔的单词，然后将每行处理的输出展平。

我们再次应用变换操作mapToPair，它基本上将单词的每个出现映射到单词的元组和计数1。

然后，我们应用reduceByKey操作将任何单词的多次出现(计数为1)分组到一个单词元组中，并计算计数的总和。

最后，我们执行收集RDD动作来获得最终结果。

### 4.3 执行–Spark作业

现在让我们使用Maven构建项目以在目标文件夹中生成apache-spark-1.0-SNAPSHOT.jar。

接下来，我们需要将这个WordCount作业提交给Spark：

```shell
${spark-install-dir}/bin/spark-submit --class cn.tuyucheng.taketoday.WordCount 
  --master local ${WordCount-MavenProject}/target/apache-spark-1.0-SNAPSHOT.jar
  ${WordCount-MavenProject}/src/main/resources/spark_example.txt
```

在运行上述命令之前，需要更新Spark安装目录和WordCount Maven项目目录。

提交时，后台会发生以下几个步骤：

1. 从驱动程序代码来看，SparkContext连接到集群管理器(在我们的例子中，Spark独立集群管理器在本地运行)
2. 集群管理器在其他应用程序之间分配资源
3. Spark在集群中的节点上获取执行器，在这里，我们的字数统计应用程序将获得自己的执行器进程
4. 应用程序代码(jar文件)被发送给执行器
5. 任务由SparkContext发送给执行器

最后，spark作业的结果返回给驱动程序，我们将看到文件中的单词数作为输出：

```text
Hello 1
from 2
Baledung 2
Keep 1
Learning 1
Spark 1
Bye 1
```

## 5. 总结

在本文中，我们讨论了Apache Spark的架构和各个组件；我们还演示了一个Spark作业的示例，该作业用于统计文件中的字数。