---
layout: post
title:  Apache Spark：Dataframe、Dataset和RDD之间的区别
category: apache
copyright: apache
excerpt: Apache Spark
---

## 1. 概述

**[Apache Spark](https://www.baeldung.com/apache-spark)是一个快速的分布式数据处理系统**，它进行内存数据处理，并使用内存缓存和优化的执行，从而实现快速的性能。它为Scala、Python、Java和R等流行的编程语言提供高级API。

在本快速教程中，我们将介绍Spark的三个基本概念：数据框、数据集和RDD。

## 2. 数据框

Spark SQL自Spark 1.3起引入了一种名为DataFrame的表格数据抽象，自此以后，它已成为Spark中最重要的功能之一。当我们想要处理结构化、半结构化的分布式数据时，此API非常有用。

在第3节中，我们将讨论弹性分布式数据集(RDD)。**DataFrames以比RDD更高效的方式存储数据，这是因为它们不仅利用了RDD的不可变性、内存存储、弹性、分布式和并行功能，还为数据应用了模式**，DataFrames还将SQL代码转换为优化的低级RDD操作。

我们可以通过三种方式创建DataFrame：

- 转换现有的RDD
- 运行SQL查询
- 加载外部数据

**Spark团队在2.0版本中引入了SparkSession，它统一了所有不同的上下文，确保开发人员无需担心创建不同的上下文**：

```java
SparkSession session = SparkSession.builder()
    .appName("TouristDataFrameExample")
    .master("local[*]")
    .getOrCreate();

DataFrameReader dataFrameReader = session.read();
```

我们将分析Tourist.csv文件：

```java
Dataset<Row> data = dataFrameReader.option("header", "true")
    .csv("data/Tourist.csv");
```

由于Spark 2.0 DataFrame成为了Row类型的Dataset，因此我们可以使用DataFrame作为Dataset<Row\>的别名。

我们可以选择我们感兴趣的特定列，也可以按给定列进行过滤和分组：

```java
data.select(col("country"), col("year"), col("value"))
    .show();

data.filter(col("country").equalTo("Mexico"))
    .show();

data.groupBy(col("country"))
    .count()
    .show();
```

## 3. 数据集

**数据集是一组强类型的结构化数据**，它们提供熟悉的面向对象编程风格以及类型安全的优势，因为数据集可以在编译时检查语法并捕获错误。

Dataset是DataFrame的扩展，因此我们可以将DataFrame视为数据集的非类型视图。

Spark团队在Spark 1.6中发布了Dataset API，正如他们提到的：“Spark Datasets的目标是提供一个API，让用户可以轻松地表达对象域上的转换，同时还提供Spark SQL执行引擎的性能和稳健性优势”。

首先，我们需要创建一个TouristData类型的类：

```java
public class TouristData {
    private String region;
    private String country;
    private String year;
    private String series;
    private Double value;
    private String footnotes;
    private String source;
    // ... getters and setters
}
```

要将每条记录映射到指定的类型，我们需要使用编码器，**编码器在Java对象和Spark内部二进制格式之间进行转换**：

```java
// SparkSession initialization and data load
Dataset<Row> responseWithSelectedColumns = data.select(col("region"), 
    col("country"), col("year"), col("series"), col("value").cast("double"), 
    col("footnotes"), col("source"));

Dataset<TouristData> typedDataset = responseWithSelectedColumns
    .as(Encoders.bean(TouristData.class));
```

与DataFrame一样，我们可以按特定列进行过滤和分组：

```java
typedDataset.filter((FilterFunction) record -> record.getCountry()
    .equals("Norway"))
    .show();

typedDataset.groupBy(typedDataset.col("country"))
    .count()
    .show();
```

我们还可以执行按匹配特定范围的列进行过滤或计算特定列的总和等操作，以获得其总值：

```java
typedDataset.filter((FilterFunction) record -> record.getYear() != null 
    && (Long.valueOf(record.getYear()) > 2010 
    && Long.valueOf(record.getYear()) < 2017)).show();

typedDataset.filter((FilterFunction) record -> record.getValue() != null 
    && record.getSeries()
        .contains("expenditure"))
        .groupBy("country")
        .agg(sum("value"))
        .show();
```

## 4. RDD

弹性分布式数据集(RDD)是Spark的主要编程抽象。它表示一组元素，这些元素具有以下特点：**不可变、弹性且分布式**。

**RDD封装了一个大型数据集，Spark会自动将RDD中包含的数据分布到我们的集群中，并并行化我们对其执行的操作**。

我们只能通过对稳定存储中的数据的操作或者对其他RDD的操作来创建RDD。

当我们处理大量数据且数据分布在集群机器上时，容错能力至关重要。由于Spark内置了故障恢复机制，RDD具有极强的弹性。**Spark依赖于RDD能够记忆其创建过程这一特性，以便我们能够轻松地追溯其历史来恢复分区**。

我们可以在RDD上执行两种类型的操作：**转换(Transformations)和操作(Actions)**。

### 4.1 转换

我们可以对RDD进行转换(Transformation)来操作其数据，由于RDD是不可变对象，**因此操作完成后，我们将获得一个全新的RDD**。

我们将检查如何实现两种最常见的转换Map和Filter。

首先，我们需要创建一个JavaSparkContext并从Tourist.csv文件中将数据作为RDD加载：

```java
SparkConf conf = new SparkConf().setAppName("uppercaseCountries")
    .setMaster("local[*]");
JavaSparkContext sc = new JavaSparkContext(conf);

JavaRDD<String> tourists = sc.textFile("data/Tourist.csv");
```

接下来，让我们应用map函数从每条记录中获取国家/地区名称，并将其转换为大写，我们可以将这个新生成的数据集保存为磁盘上的文本文件：

```java
JavaRDD<String> upperCaseCountries = tourists.map(line -> {
    String[] columns = line.split(COMMA_DELIMITER);
    return columns[1].toUpperCase();
}).distinct();

upperCaseCountries.saveAsTextFile("data/output/uppercase.txt");
```

如果我们只想选择一个特定的国家，我们可以在原始游客RDD上应用过滤功能：

```java
JavaRDD<String> touristsInMexico = tourists
    .filter(line -> line.split(COMMA_DELIMITER)[1].equals("Mexico"));

touristsInMexico.saveAsTextFile("data/output/touristInMexico.txt");
```

### 4.2 操作

操作在对数据进行一些计算后，将返回最终值或将结果保存到磁盘。

Spark中经常使用的两个操作是Count和Reduce。

让我们计算一下CSV文件中的国家总数：

```java
// Spark Context initialization and data load
JavaRDD<String> countries = tourists.map(line -> {
    String[] columns = line.split(COMMA_DELIMITER);
    return columns[1];
}).distinct();

Long numberOfCountries = countries.count();
```

现在，我们将按国家/地区计算总支出，我们需要筛选描述中包含支出的记录。

我们将使用JavaPairRDD而不是JavaRDD，**一对RDD是一种可以存储键值对的RDD**，接下来我们来检查一下：

```java
JavaRDD<String> touristsExpenditure = tourists
    .filter(line -> line.split(COMMA_DELIMITER)[3].contains("expenditure"));

JavaPairRDD<String, Double> expenditurePairRdd = touristsExpenditure
    .mapToPair(line -> {
        String[] columns = line.split(COMMA_DELIMITER);
        return new Tuple2<>(columns[1], Double.valueOf(columns[6]));
});

List<Tuple2<String, Double>> totalByCountry = expenditurePairRdd
    .reduceByKey((x, y) -> x + y)
    .collect();
```

## 5. 总结

总而言之，当我们需要特定领域的API、需要诸如聚合、求和或SQL查询之类的高级表达式，或者希望在编译时保证类型安全时，我们应该使用DataFrames或Datasets。

另一方面，当数据是非结构化的并且我们不需要实现特定的模式或者当我们需要低级转换和操作时，我们应该使用RDD。