---
layout: post
title:  Spark数据帧
category: apache
copyright: apache
excerpt: Apache Spark
---

## 1. 概述

[Apache Spark](https://www.baeldung.com/apache-spark)是一个开源的分布式分析和处理系统，支持大规模数据工程和数据科学。它通过提供统一的数据传输、大规模转换和分发API，简化了面向分析的应用程序的开发。

DataFrame是Spark API的重要组成部分。在本教程中，我们将使用一个简单的客户数据示例来研究一些Spark DataFrame API。

## 2. Spark中的DataFrame

从逻辑上讲，**DataFrame是按命名列组织的不可变记录集**，它与RDBMS中的表或Java中的ResultSet有相似之处。

DataFrame作为API，提供对多个Spark库的统一访问，包括[Spark SQL、Spark Streaming、MLib和GraphX](https://www.baeldung.com/apache-spark)。

**在Java中，我们使用Dataset<Row\>来表示DataFrame**。

本质上，Row使用名为Tungsten的高效存储，与[前代产品](https://www.baeldung.com/java-spark-dataframe-dataset-rdd)相比，它高度优化了Spark操作。

## 3. Maven依赖

让我们首先将[spark-core](https://mvnrepository.com/artifact/org.apache.spark/spark-core_2.11)和[spark-sql](https://mvnrepository.com/artifact/org.apache.spark/spark-sql_2.11)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.4.8</version>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.4.8</version>
</dependency>

```

## 4. DataFrame和Schema

本质上，DataFrame是一个带有模式的[RDD](https://www.baeldung.com/scala/apache-spark-rdd)，模式可以通过推断获得，也可以定义为StructType。

**StructType是Spark SQL中的内置数据类型，我们用它来表示StructField对象的集合**。

让我们定义一个示例Customer模式StructType：

```java
public static StructType minimumCustomerDataSchema() {
    return DataTypes.createStructType(new StructField[] {
            DataTypes.createStructField("id", DataTypes.StringType, true),
            DataTypes.createStructField("name", DataTypes.StringType, true),
            DataTypes.createStructField("gender", DataTypes.StringType, true),
            DataTypes.createStructField("transaction_amount", DataTypes.IntegerType, true) }
    );
}
```

这里，每个StructField都有一个表示DataFrame列名的名称、类型和表示是否可空的布尔值。

## 5. 构建DataFrame

每个Spark应用程序的第一个操作是通过Master获取SparkSession。

**它为我们提供了访问DataFrames的入口点**，让我们从创建SparkSession开始：

```java
public static SparkSession getSparkSession() {
    return SparkSession.builder()
            .appName("Customer Aggregation pipeline")
            .master("local")
            .getOrCreate();
}
```

请注意，我们使用本地主服务器连接到Spark，如果我们要连接到集群，则需要提供集群地址。

一旦我们有了SparkSession，我们就可以使用各种方法创建DataFrame，让我们简要地看一下其中的一些。

### 5.1 从List<POJO\>获取DataFrame

我们先构建一个List<Customer\>： 

```java
List<Customer> customers = Arrays.asList(
        aCustomerWith("01", "jo", "Female", 2000),
        aCustomerWith("02", "jack", "Male", 1200)
);
```

接下来，让我们使用createDataFrame从List<Customer\>构造DataFrame：

```java
Dataset<Row> df = SPARK_SESSION
    .createDataFrame(customerList, Customer.class);
```

### 5.2 从Dataset获取DataFrame

如果我们有一个Dataset，我们可以通过在Dataset上调用toDF轻松地将其转换为DataFrame。

让我们首先使用createDataset创建一个Dataset<Customer\>，它需要org.apache.spark.sql.Encoders：

```java
Dataset<Customer> customerPOJODataSet = SPARK_SESSION
    .createDataset(CUSTOMERS, Encoders.bean(Customer.class));
```

接下来，让我们将其转换为DataFrame：

```java
Dataset<Row> df = customerPOJODataSet.toDF();
```

### 5.3 使用RowFactory从POJO获取Row

由于DataFrame本质上是一个Dataset<Row\>，让我们看看如何从Customer POJO创建Row。

基本上，通过实现MapFunction<Customer, Row\>并重写call方法，我们可以使用RowFactory.create将每个Customer映射到Row：

```java
public class CustomerToRowMapper implements MapFunction<Customer, Row> {

    @Override
    public Row call(Customer customer) throws Exception {
        Row row = RowFactory.create(
                customer.getId(),
                customer.getName().toUpperCase(),
                StringUtils.substring(customer.getGender(),0, 1),
                customer.getTransaction_amount()
        );
        return row;
    }
}
```

我们应该注意，我们可以在将客户数据转换为Row之前在这里对其进行操作。

### 5.4 从List<Row\>获取DataFrame

我们还可以从Row对象列表创建DataFrame：

```java
List<Row> rows = customer.stream()
    .map(c -> new CustomerToRowMapper().call(c))
    .collect(Collectors.toList());
```

现在，让我们将此List<Row\>连同StructType模式一起提供给SparkSession：

```java
Dataset<Row> df = SparkDriver.getSparkSession()
    .createDataFrame(rows, SchemaFactory.minimumCustomerDataSchema());
```

请注意，**List<Row\>将根据模式定义转换为DataFrame**，模式中不存在的任何字段都不会成为DataFrame的一部分。

### 5.5 从结构化文件和数据库获取DataFrame

DataFrames可以存储列式信息(如CSV文件)以及嵌套字段和数组(如JSON文件)。

**无论我们使用的是CSV文件、JSON文件还是其他格式以及数据库，DataFrame API都保持不变**。

让我们从多行JSON数据创建DataFrame：

```java
Dataset<Row> df = SparkDriver.getSparkSession()
    .read()
    .format("org.apache.spark.sql.execution.datasources.json.JsonFileFormat")
    .option("multiline", true)
    .load("data/minCustomerData.json");
```

类似地，在从数据库读取的情况下：

```java
Dataset<Row> df = SparkDriver.getSparkSession()
    .read()
    .option("url", "jdbc:postgresql://localhost:5432/customerdb")
    .option("dbtable", "customer")
    .option("user", "user")
    .option("password", "password")
    .option("serverTimezone", "EST")
    .format("jdbc")
    .load();
```

## 6. 将DataFrame转换为Dataset

现在，让我们看看如何将DataFrame转换为Dataset，如果我们想要操作现有的POJO和仅适用于DataFrame的扩展API，这种转换非常有用。

我们将继续使用上一节中由JSON创建的DataFrame。

让我们调用一个映射函数，该函数获取Dataset<Row\>的每一行并将其转换为Customer对象：

```java
Dataset<Customer> ds = df.map(
    new CustomerMapper(),
    Encoders.bean(Customer.class)
);
```

这里，CustomerMapper实现MapFunction<Row, Customer\>：

```java
public class CustomerMapper implements MapFunction<Row, Customer> {

    @Override
    public Customer call(Row row) {
        Customer customer = new Customer();
        customer.setId(row.getAs("id"));
        customer.setName(row.getAs("name"));
        customer.setGender(row.getAs("gender"));
        customer.setTransaction_amount(Math.toIntExact(row.getAs("transaction_amount")));
        return customer;
    }
}
```

我们应该注意，**无论我们要处理的记录数有多少，MapFunction<Row, Customer\>都只实例化一次**。

## 7. DataFrame操作和转换

现在，让我们使用客户数据示例构建一个简单的管道；我们希望从两个不同的文件源中提取客户数据作为DataFrame，对其进行规范化，然后对数据执行一些转换。

最后，我们将转换后的数据写入数据库。

这些转换的目的是找出按性别和来源排序的年度支出。

### 7.1 提取数据

首先，让我们使用SparkSession的read方法从几个来源提取数据，从JSON数据开始：

```java
Dataset<Row> jsonDataToDF = SPARK_SESSION.read()
    .format("org.apache.spark.sql.execution.datasources.json.JsonFileFormat")
    .option("multiline", true)
    .load("data/customerData.json");
```

现在，让我们对CSV源执行相同的操作：

```java
Dataset<Row> csvDataToDF = SPARK_SESSION.read()
    .format("csv")
    .option("header", "true")
    .schema(SchemaFactory.customerSchema())
    .option("dateFormat", "m/d/YYYY")
    .load("data/customerData.csv"); 

csvDataToDF.show(); 
csvDataToDF.printSchema(); 
return csvData;
```

重要的是，为了读取此CSV数据，我们提供了一个确定列数据类型的StructType模式。

**一旦我们获取了数据，我们就可以使用show方法检查DataFrame的内容**。

此外，我们还可以通过在show方法中提供大小来限制行数。并且，我们可以使用printSchema来检查新创建的DataFrame的模式。

我们会注意到这两个模式有一些差异，因此，在进行任何转换之前，我们需要先对模式进行规范化。

### 7.2 规范化DataFrame

接下来，我们将规范化代表CSV和JSON数据的原始DataFrames。

这里，让我们看看执行的一些转换：

```java
private Dataset<Row> normalizeCustomerDataFromEbay(Dataset<Row> rawDataset) {
    Dataset<Row> transformedDF = rawDataset
        .withColumn("id", concat(rawDataset.col("zoneId"),lit("-"), rawDataset.col("customerId")))
        .drop(column("customerId"))
        .withColumn("source", lit("ebay"))
        .withColumn("city", rawDataset.col("contact.customer_city"))
        .drop(column("contact"))
        .drop(column("zoneId"))
        .withColumn("year", functions.year(col("transaction_date")))
        .drop("transaction_date")
        .withColumn("firstName", functions.split(column("name"), " ")
            .getItem(0))
        .withColumn("lastName", functions.split(column("name"), " ")
            .getItem(1))
        .drop(column("name"));

    return transformedDF; 
}
```

上述示例中对DataFrame的一些重要操作是：

- concat将来自多个列和文字的数据拼接起来以创建新的id列
- lit静态函数返回具有文字值的列
- functions.year从transactionDate中提取year
- function.split将name拆分为firstname和lastname列
- drop方法删除数据框中的一列
- col方法根据数据集的名称返回数据集的列
- withColumnRenamed返回具有重命名值的列

**重要的是，我们可以看到DataFrame是不可变的**。 因此，每当需要更改任何内容时，我们都必须创建一个新的DataFrame。

最终，两个数据框都被规范化为相同的模式，如下所示：

```text
root
 |-- gender: string (nullable = true)
 |-- transaction_amount: long (nullable = true)
 |-- id: string (nullable = true)
 |-- source: string (nullable = false)
 |-- city: string (nullable = true)
 |-- year: integer (nullable = true)
 |-- firstName: string (nullable = true)
 |-- lastName: string (nullable = true)
```

### 7.3 合并DataFrame

接下来让我们合并规范化的DataFrames：

```java
Dataset<Row> combineDataframes(Dataset<Row> df1, Dataset<Row> df2) {
    return df1.unionByName(df2); 
}
```

重要的是，我们应该注意：

- 如果我们在合并两个DataFrame时关心列名，则应该使用unionByName
- 如果我们在合并两个DataFrames时不关心列名，则应该使用union

### 7.4 聚合DataFrame

接下来，让我们对组合后的DataFrame进行分组，找出按年份、来源和性别划分的年度支出。

然后，我们将按年份升序和每年花费降序对汇总数据进行排序：

```java
Dataset<Row> aggDF = dataset
    .groupBy(column("year"), column("source"), column("gender"))
    .sum("transactionAmount")
    .withColumnRenamed("sum(transaction_amount)", "yearly spent")
    .orderBy(col("year").asc(), col("yearly spent").desc());
```

上述示例中对DataFrame的一些重要操作是：

- groupBy用于将DataFrame上的相同数据分组，然后执行类似于SQL“GROUP BY”子句的聚合函数
- sum在分组后对transactionAmount列应用聚合函数 
- orderBy按一个或多个列对DataFrame进行排序
- Column类中的asc和desc函数可用于指定排序顺序

最后，我们用show方法看看转换后的数据框是什么样子的：

```text
+----+------+------+---------------+
|year|source|gender|annual_spending|
+----+------+------+---------------+
|2018|amazon|  Male|          10600|
|2018|amazon|Female|           6200|
|2018|  ebay|  Male|           5500|
|2021|  ebay|Female|          16000|
|2021|  ebay|  Male|          13500|
|2021|amazon|  Male|           4000|
|2021|amazon|Female|           2000|
+----+------+------+---------------+
```

因此，最终转换后的模式应该是：

```text
root
 |-- source: string (nullable = false)
 |-- gender: string (nullable = true)
 |-- year: integer (nullable = true)
 |-- yearly spent: long (nullable = true)
```

### 7.5 从DataFrame写入关系型数据库

最后，让我们将转换后的DataFrame写为关系型数据库中的表：

```java
Properties dbProps = new Properties();

dbProps.setProperty("connectionURL", "jdbc:postgresql://localhost:5432/customerdb");
dbProps.setProperty("driver", "org.postgresql.Driver");
dbProps.setProperty("user", "postgres");
dbProps.setProperty("password", "postgres");
```

接下来我们可以使用Spark会话写入数据库：

```java
String connectionURL = dbProperties.getProperty("connectionURL");

dataset.write()
    .mode(SaveMode.Overwrite)
    .jdbc(connectionURL, "customer", dbProperties);
```

## 8. 测试

现在，我们可以使用两个摄取源以及postgres和pgAdmin Docker镜像端到端测试管道：

```java
@Test
void givenCSVAndJSON_whenRun_thenStoresAggregatedDataFrameInDB() throws Exception {
    Properties dbProps = new Properties();
    dbProps.setProperty("connectionURL", "jdbc:postgresql://localhost:5432/customerdb");
    dbProps.setProperty("driver", "org.postgresql.Driver");
    dbProps.setProperty("user", "postgres");
    dbProps.setProperty("password", "postgres");

    pipeline = new CustomerDataAggregationPipeline(dbProps);
    pipeline.run();

    String allCustomersSql = "Select count(*) from customer";

    Statement statement = conn.createStatement();
    ResultSet resultSet = statement.executeQuery(allCustomersSql);
    resultSet.next();
    int count = resultSet.getInt(1);
    assertEquals(7, count);
}
```

**运行此命令后，我们可以验证是否存在一个表，其中包含与DataFrame对应的列和行**。最后，我们还可以通过pgAdmin4客户端观察此输出：

![](/assets/images/2025/apache/sparkdataframes01.png)

我们应该在这里注意几个要点：

- 由于write操作，customer表会自动创建。
- 使用的模式是SaveMode.Overwrite，因此，这将覆盖表中所有已存在的内容；其他可用选项包括Append、Ignore和ErrorIfExists

此外，我们还可以使用write将DataFrame数据导出为CSV、JSON或parquet等格式。

## 9. 总结

在本教程中，我们研究了如何使用DataFrames在Apache Spark中执行数据操作和聚合。

首先，我们从各种输入源创建了DataFrame；然后，我们使用一些API方法对数据进行规范化、组合和聚合。

最后，我们将DataFrame导出为关系型数据库中的表。