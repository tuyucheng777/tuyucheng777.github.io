---
layout: post
title:  Apache Geode快速指南
category: apache
copyright: apache
excerpt: Apache Geode
---

## 1. 概述

[Apache Geode](http://geode.apache.org/)是一个支持缓存和数据计算的分布式内存数据网格。

在本教程中，我们将介绍Geode的关键概念并使用其Java客户端运行一些代码示例。

## 2. 设置

首先，我们需要下载并安装Apache Geode，并设置gfsh环境，具体操作可以参考[Geode官方指南](http://geode.apache.org/docs/guide/16/getting_started/15_minute_quickstart_gfsh.html)。

其次，本教程将创建一些文件系统构件。因此，我们可以通过创建一个临时目录并从那里启动文件来隔离它们。

### 2.1 安装和配置

从我们的临时目录中，我们需要启动一个Locator实例：

```shell
gfsh> start locator --name=locator --bind-address=localhost
```

**Locator负责Geode集群不同成员之间的协调，我们可以通过JMX进一步管理**。

接下来我们启动一个Server实例来托管一个或多个数据Region：

```shell
gfsh> start server --name=server1 --server-port=0
```

我们将–server-port选项设置为0，这样Geode就会选择任何可用的端口，如果不设置，服务器将使用默认端口40404。**服务器是Cluster中可配置的成员，作为长生命周期进程运行，负责管理数据Region**。

最后，我们需要一个Region：

```text
gfsh> create region --name=tuyucheng --type=REPLICATE
```

**该Region最终是我们存储数据的地方**。

### 2.2 验证

在继续下一步之前，让我们先确保一切正常。

首先，让我们检查一下我们是否有Server和Locator：

```shell
gfsh> list members
 Name   | Id
------- | ----------------------------------------------------------
server1 | 192.168.0.105(server1:6119)<v1>:1024
locator | 127.0.0.1(locator:5996:locator)<ec><v0>:1024 [Coordinator]
```

接下来，检查Region是否存在：

```shell
gfsh> describe region --name=tuyucheng
..........................................................
Name            : tuyucheng
Data Policy     : replicate
Hosting Members : server1

Non-Default Attributes Shared By Hosting Members  

 Type  |    Name     | Value
------ | ----------- | ---------------
Region | data-policy | REPLICATE
       | size        | 0
       | scope       | distributed-ack
```

此外，我们的临时目录下的文件系统上应该有一些名为“locator”和“server1”的目录。

## 3. Maven依赖

现在我们已经有一个正在运行的Geode，让我们开始查看客户端代码。

为了在我们的Java代码中使用Geode，我们需要将[Apache Geode Java客户端库](https://mvnrepository.com/artifact/org.apache.geode/geode-core)添加到pom中：

```xml
<dependency>
     <groupId>org.apache.geode</groupId>
     <artifactId>geode-core</artifactId>
     <version>1.15.1</version>
</dependency>
```

让我们首先在几个区域中简单地存储和检索一些数据。

## 4. 简单的存储和检索

让我们演示如何存储单个值、批量值以及自定义对象。

要开始在我们的“tuyucheng”区域存储数据，让我们使用定位器连接到它：

```java
@Before
public void connect() {
    this.cache = new ClientCacheFactory()
            .addPoolLocator("localhost", 10334)
            .create();
    this.region = cache.<String, String>
                    createClientRegionFactory(ClientRegionShortcut.CACHING_PROXY)
            .create("tuyucheng");
}
```

### 4.1 保存单个值

现在，我们可以简单地在区域中存储和检索数据：

```java
@Test
public void whenSendMessageToRegion_thenMessageSavedSuccessfully() {
    this.region.put("A", "Hello");
    this.region.put("B", "tuyucheng");

    assertEquals("Hello", region.get("A"));
    assertEquals("tuyucheng", region.get("B"));
}
```

### 4.2 一次保存多个值

我们还可以一次保存多个值，例如在尝试减少网络延迟时：

```java
@Test
public void whenPutMultipleValuesAtOnce_thenValuesSavedSuccessfully() {
    Supplier<Stream<String>> keys = () -> Stream.of("A", "B", "C", "D", "E");
    Map<String, String> values = keys.get()
        .collect(Collectors.toMap(Function.identity(), String::toLowerCase));

    this.region.putAll(values);

    keys.get()
        .forEach(k -> assertEquals(k.toLowerCase(), this.region.get(k)));
}
```

### 4.3 保存自定义对象

字符串很有用，但是我们很快就需要存储自定义对象。

假设我们有一条客户记录，想要使用以下键类型进行存储：

```java
public class CustomerKey implements Serializable {
    private long id;
    private String country;
    
    // getters and setters
    // equals and hashcode
}
```

以及以下值类型：

```java
public class Customer implements Serializable {
    private CustomerKey key;
    private String firstName;
    private String lastName;
    private Integer age;
    
    // getters and setters 
}
```

要存储这些内容，还需要几个额外的步骤：

首先，**它们应该实现Serializable接口**，虽然这不是一个严格的要求，但是通过实现Serializable接口，[Geode可以更稳健地存储它们](https://geode.apache.org/docs/guide/16/developing/data_serialization/data_serialization_options.html)。

其次，**它们需要位于我们应用程序的类路径以及Geode Server的类路径上**。

为了将它们放入服务器的类路径，让我们将它们打包，例如使用mvn clean package。

然后我们可以在新的start server命令中引用生成的jar文件：

```text
gfsh> stop server --name=server1
gfsh> start server --name=server1 --classpath=../lib/apache-geode-1.0.0.jar --server-port=0
```

**再次，我们必须从临时目录运行这些命令**。

最后，让我们使用创建“tuyucheng”区域时使用的相同命令在服务器上创建一个名为“tuyucheng-customers”的新区域：

```shell
gfsh> create region --name=tuyucheng-customers --type=REPLICATE
```

在代码中，我们将像以前一样连接定位器，指定自定义类型：

```java
@Before
public void connect() {
    // ... connect through the locator
    this.customerRegion = this.cache.<CustomerKey, Customer>
                    createClientRegionFactory(ClientRegionShortcut.CACHING_PROXY)
            .create("tuyucheng-customers");
}
```

然后，我们可以像以前一样存储Customer：

```java
@Test
public void whenPutCustomKey_thenValuesSavedSuccessfully() {
    CustomerKey key = new CustomerKey(123);
    Customer customer = new Customer(key, "William", "Russell", 35);

    this.customerRegion.put(key, customer);

    Customer storedCustomer = this.customerRegion.get(key);
    assertEquals("William", storedCustomer.getFirstName());
    assertEquals("Russell", storedCustomer.getLastName());
}
```

## 5. 区域类型

对于大多数环境，我们将拥有区域的多个副本或多个分区，具体取决于我们的读写吞吐量要求。

到目前为止，我们已经使用了内存复制区域，让我们仔细看看。

### 5.1 复制区域

顾名思义，**复制区域(Replicated Region)在多个服务器上维护其数据的副本**，我们来测试一下。

从工作目录中的gfsh控制台，让我们向集群中添加一个名为server2的服务器：

```shell
gfsh> start server --name=server2 --classpath=../lib/apache-geode-1.0.0.jar --server-port=0
```

记得我们在创建“tuyucheng”时使用了–type=REPLICATE，这样，**Geode会自动将我们的数据复制到新服务器**。

让我们通过停止server1来验证这一点：

```shell
gfsh> stop server --name=server1
```

然后，让我们对“tuyucheng”区域执行快速查询。

如果数据复制成功，我们将得到结果：

```shell
gfsh> query --query='select e.key from /tuyucheng.entries e'
Result : true
Limit  : 100
Rows   : 5

Result
------
C
B
A 
E
D
```

因此，看起来复制成功了。

在我们的区域中添加副本可以提高数据可用性，而且，由于多个服务器可以响应查询，我们也能获得更高的读取吞吐量。

但是，**如果它们都崩溃了怎么办？由于这些是内存区域，数据将会丢失**。为此，我们可以改用–type=REPLICATE_PERSISTENT，它在复制的同时也会将数据存储在磁盘上。

### 5.2 分区区域

对于更大的数据集，我们可以通过配置Geode将区域划分为单独的分区或存储桶来更好地扩展系统。

让我们创建一个名为“tuyucheng-partitioned”的分区区域：

```shell
gfsh> create region --name=tuyucheng-partitioned --type=PARTITION
```

添加一些数据：

```shell
gfsh> put --region=tuyucheng-partitioned --key="1" --value="one"
gfsh> put --region=tuyucheng-partitioned --key="2" --value="two"
gfsh> put --region=tuyucheng-partitioned --key="3" --value="three"
```

并快速验证：

```shell
gfsh> query --query='select e.key, e.value from /tuyucheng-partitioned.entries e'
Result : true
Limit  : 100
Rows   : 3

key | value
--- | -----
2   | two
1   | one
3   | three
```

然后，为了验证数据是否已分区，让我们再次停止server1并重新查询：

```shell
gfsh> stop server --name=server1
gfsh> query --query='select e.key, e.value from /tuyucheng-partitioned.entries e'
Result : true
Limit  : 100
Rows   : 1

key | value
--- | -----
2   | two
```

这次我们只恢复了部分数据条目，因为该服务器只有一个数据分区，因此当服务器1掉线时，其数据就丢失了。

**但是如果我们同时需要分区和冗余怎么办**？Geode还支持[许多其他类型](https://geode.apache.org/docs/guide/11/reference/topics/region_shortcuts_reference.html)。以下三种比较方便：

- PARTITION_REDUNDANT在集群的不同成员之间对数据进行分区和复制
- PARTITION_PERSISTENT像PARTITION一样对数据进行分区，但分区到磁盘
- PARTITION_REDUNDANT_PERSISTENT为我们提供了所有三种行为

## 6. 对象查询语言

Geode还支持对象查询语言(OQL)，它比简单的键查找功能更强大，有点像SQL。

对于此示例，让我们使用之前构建的“tuyucheng-customer”区域。

如果我们再添加几个Customer：

```java
Map<CustomerKey, Customer> data = new HashMap<>();
data.put(new CustomerKey(1), new Customer("Gheorge", "Manuc", 36));
data.put(new CustomerKey(2), new Customer("Allan", "McDowell", 43));
this.customerRegion.putAll(data);
```

然后我们可以使用QueryService来查找名字为“Allan”的Customer：

```java
QueryService queryService = this.cache.getQueryService();
String query = "select * from /tuyucheng-customers c where c.firstName = 'Allan'";
SelectResults<Customer> results = (SelectResults<Customer>) queryService.newQuery(query).execute();
assertEquals(1, results.size());
```

## 7. 函数

内存数据网格的一个更强大的概念是“将计算带入数据”的想法。

简而言之，由于Geode是纯Java，**因此我们不但可以轻松发送数据，还可以发送对数据的逻辑**。

这可能让我们想起PL-SQL或Transact-SQL等SQL扩展的想法。

### 7.1 定义函数

为了定义Geode要执行的工作单元，我们实现了Geode的Function接口。

例如，假设我们需要将所有Customer的姓名改为大写。

我们不需要查询数据并让我们的应用程序完成工作，只需实现Function即可：

```java
public class UpperCaseNames implements Function<Boolean> {
    @Override
    public void execute(FunctionContext<Boolean> context) {
        RegionFunctionContext regionContext = (RegionFunctionContext) context;
        Region<CustomerKey, Customer> region = regionContext.getDataSet();

        for ( Map.Entry<CustomerKey, Customer> entry : region.entrySet() ) {
            Customer customer = entry.getValue();
            customer.setFirstName(customer.getFirstName().toUpperCase());
        }
        context.getResultSender().lastResult(true);
    }

    @Override
    public String getId() {
        return getClass().getName();
    }
}
```

**请注意，getId必须返回一个唯一的值，因此类名通常是一个不错的选择**。

FunctionContext包含我们所有的区域数据，因此我们可以对其进行更复杂的查询，或者像我们在这里所做的那样对其进行变异。

而且Function的功能远不止于此，因此请查看[官方手册](https://geode.apache.org/docs/guide/114/developing/function_exec/chapter_overview.html)，尤其是[getResultSender](https://geode.apache.org/releases/latest/javadoc/org/apache/geode/cache/execute/FunctionContext.html#getResultSender--)方法。

### 7.2 部署函数

我们需要让Geode感知到我们的函数才能运行它，就像处理自定义数据类型一样，我们将打包这个jar文件。

但这次，我们只需使用deploy命令即可：

```shell
gfsh> deploy --jar=./lib/apache-geode-1.0.0.jar
```

### 7.3 执行函数

现在，我们可以使用FunctionService从应用程序执行该函数：

```java
@Test
public void whenExecuteUppercaseNames_thenCustomerNamesAreUppercased() {
    Execution execution = FunctionService.onRegion(this.customerRegion);
    execution.execute(UpperCaseNames.class.getName());
    Customer customer = this.customerRegion.get(new CustomerKey(1));
    assertEquals("GHEORGE", customer.getFirstName());
}
```

## 8. 总结

在本文中，我们学习了Apache Geode生态系统的基本概念，我们研究了标准和自定义类型的简单get和put操作、复制和分区区域以及oql和函数支持。