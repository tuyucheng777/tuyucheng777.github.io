---
layout: post
title:  Apache Ignite指南
category: libraries
copyright: libraries
excerpt: Apache Ignite
---

## 1. 简介

Apache Ignite是一个开源的以内存为中心的分布式平台，我们可以将它用作数据库、缓存系统或用于内存数据处理。

该平台使用内存作为存储层，因此具有令人印象深刻的性能。简而言之，**这是目前生产中使用的最快的原子数据处理平台之一**。

## 2. 安装和设置

首先，请查看[入门页面](https://apacheignite.readme.io/docs/getting-started)，了解初始设置和安装说明。

我们将要构建的应用程序的Maven依赖：

```xml
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-core</artifactId>
    <version>${ignite.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-indexing</artifactId>
    <version>${ignite.version}</version>
</dependency>
```

**[ignite-core](https://mvnrepository.com/artifact/org.apache.ignite/ignite-core)是项目唯一的强制依赖**，由于我们还想与SQL进行交互，所以这里也包含了ignite-indexing，[${ignite.version}](https://mvnrepository.com/artifact/org.apache.ignite/ignite-core)是Apache Ignite的最新版本。

作为最后一步，我们启动Ignite节点：

```text
Ignite node started OK (id=53c77dea)
Topology snapshot [ver=1, servers=1, clients=0, CPUs=4, offheap=1.2GB, heap=1.0GB]
Data Regions Configured:
^-- default [initSize=256.0 MiB, maxSize=1.2 GiB, persistenceEnabled=false]
```

上面的控制台输出表明我们已经准备好了。

## 3. 内存架构

**该平台基于持久内存架构**，这使得数据可以在磁盘和内存中存储和处理，它通过有效利用集群的RAM资源来提高性能。

内存和磁盘上的数据具有相同的二进制表示形式，这意味着在从一个层移动到另一个层时无需对数据进行额外的转换。

持久内存架构分为固定大小的块，称为页面。页面存储在Java堆之外，并组织在RAM中。它有一个唯一标识符：FullPageId。

页面使用PageMemory抽象与内存进行交互。

它有助于读取、写入页面，还可以分配页面ID。**在内存中，Ignite将页面与内存缓冲区关联起来**。

## 4. 内存页面

页面可以具有以下状态：

- Unloaded：内存中未加载页面缓冲区
- Clear：页面缓冲区已加载并与磁盘上的数据同步
- Durty：页面缓冲区保存的数据与磁盘中的数据不同
- Dirty in checkpoint：在第一个修改保存到磁盘之前，另一个修改已开始。此处检查点开始，PageMemory为每个页面保留两个内存缓冲区

**持久内存在本地分配一个称为数据区域的内存段**，默认情况下，它的容量为集群内存的20%。多个区域配置允许将可用的数据保存在内存中。

该区域的最大容量是一个Memory Segment，它是一段物理内存或者一个连续的字节数组。

**为了避免内存碎片，单个页面可容纳多个键值条目，每个新条目都将添加到最优页面**。如果键值对大小超出页面的最大容量，Ignite会将数据存储在多个页面中，更新数据时也适用相同的逻辑。

SQL和缓存索引存储在称为B+树的结构中，缓存键按其键值排序。

## 5. 生命周期

**每个Ignite节点都在单个JVM实例上运行**，但是，可以配置为在单个JVM进程中运行多个Ignite节点。

让我们来看看生命周期事件类型：

- BEFORE_NODE_START：在Ignite节点启动之前
- AFTER_NODE_START：在Ignite节点启动后立即触发
- BEFORE_NODE_STOP：在启动节点停止之前
- AFTER_NODE_STOP：Ignite节点停止后

要启动默认Ignite节点：

```java
Ignite ignite = Ignition.start();
```

或者从配置文件中：

```java
Ignite ignite = Ignition.start("config/example-cache.xml");
```

如果我们需要对初始化过程进行更多的控制，还有另一种方法，借助LifecycleBean接口：

```java
public class CustomLifecycleBean implements LifecycleBean {

    @Override
    public void onLifecycleEvent(LifecycleEventType lifecycleEventType) throws IgniteException {
        if(lifecycleEventType == LifecycleEventType.AFTER_NODE_START) {
            // ...
        }
    }
}
```

在这里，我们可以使用生命周期事件类型在节点启动/停止之前或之后执行操作。

为此，我们将配置实例与CustomLifecycleBean传递给启动方法：

```java
IgniteConfiguration configuration = new IgniteConfiguration();
configuration.setLifecycleBeans(new CustomLifecycleBean());
Ignite ignite = Ignition.start(configuration);
```

## 6. 内存数据网格

**Ignite数据网格是一个分布式键值存储**，与分区HashMap非常相似。它是水平扩展的，这意味着我们添加的集群节点越多，缓存或存储在内存中的数据就越多。

它可以作为缓存的附加层，为第三方软件(如NoSql、RDMS数据库)提供显著的性能改进。

### 6.1 缓存支持

**数据访问API基于JCache JSR 107规范**。

作为示例，让我们使用模板配置创建缓存：

```java
IgniteCache<Employee, Integer> cache = ignite.getOrCreateCache("tuyuchengCache");
```

让我们看看这里发生了什么以了解更多细节。首先，Ignite找到缓存存储的内存区域。

然后根据key的哈希码定位到B+树索引Page，如果索引存在，则定位到对应key的数据Page。

**当索引为NULL时，平台使用给定的键创建新的数据条目**。

接下来，让我们添加一些Employee对象：

```java
cache.put(1, new Employee(1, "John", true));
cache.put(2, new Employee(2, "Anna", false));
cache.put(3, new Employee(3, "George", true));
```

再次，持久内存将查找缓存所属的内存区域。根据缓存键，索引页将位于B+树结构中。

**当索引页不存在时，将请求一个新的索引页并将其添加到树中**。

接下来，将数据页分配给索引页。

要从缓存中读取员工，我们只需使用键值：

```java
Employee employee = cache.get(1);
```

### 6.2 流支持

内存数据流为基于磁盘和文件系统的数据处理应用程序提供了一种替代方法，**Streaming API将高负载数据流分成多个阶段并路由进行处理**。

我们可以修改示例并从文件中流式传输数据。首先，我们定义一个数据流器：

```java
IgniteDataStreamer<Integer, Employee> streamer = ignite
    .dataStreamer(cache.getName());
```

接下来，我们可以注册一个流转换器来将收到的员工标记为已就业：

```java
streamer.receiver(StreamTransformer.from((e, arg) -> {
    Employee employee = e.getValue();
    employee.setEmployed(true);
    e.setValue(employee);
    return employee;
}));
```

最后一步，我们遍历employees.txt文件行并将其转换为Java对象：

```java
Path path = Paths.get(IgniteStream.class.getResource("employees.txt")
    .toURI());
Gson gson = new Gson();
Files.lines(path)
    .forEach(l -> streamer.addData(
        employee.getId(), 
        gson.fromJson(l, Employee.class)));
```

**使用streamer.addData()将员工对象放入流中**。

## 7. SQL支持

**该平台提供以内存为中心、具有容错功能的SQL数据库**。

我们可以使用纯SQL API或JDBC进行连接。此处的SQL语法是ANSI-99，因此支持查询、DML、DDL语言操作中的所有标准聚合函数。

### 7.1 JDBC

为了更加实用，让我们创建一个员工表并向其中添加一些数据。

为此，我们注册一个JDBC驱动程序并打开一个连接作为下一步：

```java
Class.forName("org.apache.ignite.IgniteJdbcThinDriver");
Connection conn = DriverManager.getConnection("jdbc:ignite:thin://127.0.0.1/");
```

在标准DDL命令的帮助下，我们填充Employee表：

```java
sql.executeUpdate("CREATE TABLE Employee (" +
    " id LONG PRIMARY KEY, name VARCHAR, isEmployed tinyint(1)) " +
    " WITH \"template=replicated\"");
```

在WITH关键字之后，我们可以设置缓存配置模板。这里我们使用REPLICATED。**默认情况下，模板模式为PARTITIONED。为了指定数据的副本数，我们还可以在此处指定BACKUPS参数，默认情况下为0**。

然后，让我们使用INSERT DML语句添加一些数据：

```java
PreparedStatement sql = conn.prepareStatement("INSERT INTO Employee (id, name, isEmployed) VALUES (?, ?, ?)");

sql.setLong(1, 1);
sql.setString(2, "James");
sql.setBoolean(3, true);
sql.executeUpdate();

// add the rest
```

然后，我们选择记录：

```java
ResultSet rs = sql.executeQuery("SELECT e.name, e.isEmployed " 
    + " FROM Employee e " 
    + " WHERE e.isEmployed = TRUE ")
```

### 7.2 查询对象

**还可以对缓存中存储的Java对象执行查询**，Ignite将Java对象视为单独的SQL记录：

```java
IgniteCache<Integer, Employee> cache = ignite.cache("tuyuchengCache");

SqlFieldsQuery sql = new SqlFieldsQuery("select name from Employee where isEmployed = 'true'");

QueryCursor<List<?>> cursor = cache.query(sql);

for (List<?> row : cursor) {
    // do something with the row
}
```

## 8. 总结

在本教程中，我们快速了解了Apache Ignite项目，本指南重点介绍了该平台相对于其他类似产品的优势，例如性能提升、耐用性、轻量级API。

因此，我们学习了如何使用SQL语言和Java API来存储、检索和流式传输持久化****或内存网格内的数据。