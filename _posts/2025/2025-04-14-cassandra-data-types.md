---
layout: post
title:  CQL数据类型
category: persistence
copyright: persistence
excerpt: Apache Cassandra
---

## 1. 概述

在本教程中，我们将展示Apache Cassandra数据库的一些不同数据类型。**[Apache Cassandra](https://www.baeldung.com/cassandra-with-java)支持丰富的数据类型，包括集合类型、原生类型、元组类型和用户定义类型**。

Cassandra查询语言(CQL)是结构化查询语言(SQL)的简单替代方案，它是一种声明式语言，旨在提供与数据库的通信。与SQL类似，CQL也将数据存储在表中，并将数据组织成行和列。

## 2. Cassandra数据库配置

让我们使用[Docker镜像](https://hub.docker.com/r/bitnami/cassandra/)创建一个数据库，并使用cqlsh将其连接到数据库。接下来，我们应该创建一个keyspace：

```shell
CREATE KEYSPACE tuyucheng WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 1};
```

出于本教程的目的，我们创建了一个仅包含一份数据的keyspace。现在，让我们将客户端会话连接到keyspace：

```shell
USE tuyucheng;
```

## 3. 内置数据类型

CQL支持丰富的原生数据类型。这些数据类型是预定义的，我们可以直接引用其中任何一种。

### 3.1 数字类型

数字类型类似于Java和其他语言中的标准类型，例如整数或浮点数，但范围不同：

![](/assets/images/2025/persistence/cassandradatatypes01.png)

让我们创建一个包含所有这些数据类型的表：

```sql
CREATE TABLE numeric_types
(
    type1 int PRIMARY KEY,
    type2 bigint,
    type3 smallint,
    type4 tinyint,
    type5 varint,
    type6 float,
    type7 double,
    type8 decimal
);
```

### 3.2 文本类型

CQL提供了两种用于表示文本的数据类型，**我们可以使用text或varchar来创建UTF-8字符串**，UTF-8是较新且广泛使用的文本标准，并且支持国际化。

**还有用于创建ASCII字符串的ascii类型**，如果我们要处理ASCII格式的旧数据，那么ascii类型非常有用。文本值的大小受列的最大大小限制，单列值大小为2GB，但建议仅为1MB。

让我们创建一个包含所有这些数据类型的表：

```sql
CREATE TABLE text_types
(
    primaryKey int PRIMARY KEY,
    type2      text,
    type3      varchar,
    type4      ascii
);
```

### 3.3 日期类型

现在，我们来谈谈日期类型。Cassandra提供了几种类型，它们在定义唯一分区键或定义普通列时非常有用：

![](/assets/images/2025/persistence/cassandradatatypes02.png)

time did用[UUID版本1](https://en.wikipedia.org/wiki/Universally_unique_identifier)表示，我们可以将整数或字符串输入到CQL时间戳、时间和日期中。**持续时间类型的值被编码为3个有符号整数**。

第一个整数表示月份数，第二个整数表示天数，第三个整数表示纳秒数。

我们来看一个create table命令的例子：

```sql
CREATE TABLE date_types
(
    primaryKey int PRIMARY KEY,
    type1      timestamp,
    type2      time,
    type3      date,
    type4      timeuuid,
    type5      duration
);
```

### 3.4 计数器类型

**计数器类型用于定义计数器列**，计数器列的值为64位有符号整数，我们只能对计数器列执行两种操作：递增和递减。

因此，我们不能将值赋给计数器。我们可以使用计数器来跟踪统计数据，例如页面浏览量、推文数量、日志消息数量等等。**我们不能将计数器类型与其他类型混合使用**。

让我们看一个例子：

```sql
CREATE TABLE counter_type
(
    primaryKey uuid PRIMARY KEY,
    type1      counter
);
```

### 3.5 其他数据类型

- boolean是一个简单的true/false值
- uuid是Type 4的UUID，完全基于随机数，我们可以使用以短划线分隔的十六进制数字序列来输入UUID
- 二进制大对象(blob)是一个通俗的计算术语，指任意字节数组。CQL blob类型用于存储媒体或其他二进制文件类型，blob的最大大小为2GB，但建议小于1MB
- inet是表示IPv4或IPv6 Internet地址的类型

再次，让我们创建一个包含这些类型的表：

```sql
CREATE TABLE other_types
(
    primaryKey int PRIMARY KEY,
    type1      boolean,
    type2      uuid,
    type3      blob,
    type4      inet
);
```

## 4. 集合数据类型

有时我们希望存储相同类型的数据，而不生成新的列，集合可以存储多个值。CQL提供了三种集合类型来帮助我们：列表、集合和映射。

例如，我们可以创建一个表，其中包含文本元素列表、整数列表或一些其他元素类型的列表。

### 4.1 集合

**我们可以使用集合数据类型存储多个唯一值**。同样，在Java中，元素不是按顺序存储的。

让我们创建一个集合：

```sql
CREATE TABLE collection_types
(
    primaryKey int PRIMARY KEY,
    email      set<text>
);
```

### 4.2 列表

**在此数据类型中，值以列表的形式存储，我们无法更改元素的顺序**。将值存储在列表中后，元素会获得特定的索引，我们可以使用这些索引来检索数据。

与集合不同，列表可以存储重复的值。让我们在表中添加一个列表：

```sql
ALTER TABLE collection_types
    ADD scores list<text>;
```

### 4.3 映射

使用Cassandra时，**我们可以使用map数据类型将数据存储在键值对集合中**。键是唯一的，因此，我们可以按键对map进行排序。

让我们在表中添加另一列：

```sql
ALTER TABLE collection_types
    ADD address map<uuid, text>;
```

## 5. 元组

**元组是一组不同类型的元素**。这些集合具有固定的长度：

```sql
CREATE TABLE tuple_type
(
    primaryKey int PRIMARY KEY,
    type1 tuple<int, text, float>
);
```

## 6. 用户定义数据类型

**Cassandra提供了创建我们自己的数据类型的可能性**，我们可以创建、修改和删除这些数据类型。首先，让我们创建自己的类型：

```sql
CREATE TYPE user_defined_type (
    type1 timestamp,
    type2 text,
    type3 text,
    type4 text);
```

因此，现在可以用我们的类型创建一个表：

```sql
CREATE TABLE user_type
(
    primaryKey int PRIMARY KEY,
    our_type   user_defined_type
);
```

## 7. 总结

在本快速教程中，我们探索了基本的CQL数据类型。此外，我们还创建了使用这些数据类型的表。之后，我们讨论了它们可以存储哪些类型的数据。