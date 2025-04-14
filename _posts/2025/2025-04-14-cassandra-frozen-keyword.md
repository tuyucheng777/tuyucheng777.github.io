---
layout: post
title:  Cassandra Frozen关键字
category: persistence
copyright: persistence
excerpt: Apache Cassandra
---

## 1. 概述

在本教程中，我们将讨论[Apache Cassandra数据库](https://cassandra.apache.org/_/index.html)中的Frozen关键字。首先，我们将展示如何声明冻结集合或用户定义类型(UDT)。接下来，我们将讨论使用示例以及它如何影响持久存储的基本操作。

## 2. Cassandra数据库配置

让我们使用[Docker镜像](https://hub.docker.com/r/bitnami/cassandra)创建一个数据库，并使用cqlsh将其连接到数据库。接下来，我们应该创建一个**键空间**：

```shell
CREATE KEYSPACE mykeyspace WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 1};
```

在本教程中，我们创建了一个仅包含一份数据的keyspace。现在，让我们将客户端会话连接到keyspace：

```shell
USE mykeyspace;
```

## 3. 冻结集合类型

**类型为frozen集合(set、map或list)的列只能将其值整体替换**，换句话说，我们无法像在非冻结集合类型中那样从集合中添加、更新或删除单个元素。因此，frozen关键字非常有用，例如，当我们想要保护集合免受单值更新的影响时。

此外，**由于冻结功能，我们可以使用冻结集合作为表中的主键**。我们可以使用集合类型(例如set、list或map)声明集合列，然后，我们添加集合的类型。

要声明冻结集合，我们必须在集合定义前添加关键字：

```sql
CREATE TABLE mykeyspace.users
(
    id         uuid PRIMARY KEY,
    ip_numbers frozen<set<inet>>,
    addresses  frozen<map<text, tuple<text>>>,
    emails     frozen<list<varchar>>,
);
```

让我们插入一些数据：

```sql
INSERT INTO mykeyspace.users (id, ip_numbers)
VALUES (6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47, {'10.10.11.1', '10.10.10.1', '10.10.12.1'});
```

重要的是，正如我们上面提到的，**冻结的集合只能作为一个整体进行替换**，这意味着我们无法添加或删除元素。让我们尝试向ip_numbers集合添加一个新元素：

```sql
UPDATE mykeyspace.users
SET ip_numbers = ip_numbers + {'10.10.14.1'}
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47;
```

执行更新后，我们会收到错误：

```text
InvalidRequest: Error from server: code=2200 [Invalid query] message="Invalid operation (ip_numbers = ip_numbers + {'10.10.14.1'}) for frozen collection column ip_numbers"
```

如果我们想更新集合中的数据，我们需要更新整个集合：

```sql
UPDATE mykeyspace.users
SET ip_numbers = {'11.10.11.1', '11.10.10.1', '11.10.12.1'}
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b47;
```

### 3.1 嵌套集合

有时我们必须在Cassandra数据库中使用嵌套集合，**只有将嵌套集合标记为冻结时，才可以使用嵌套集合**，这意味着该集合将是不可变的。我们可以在冻结和非冻结集合中冻结嵌套集合，让我们看一个例子：

```sql
CREATE TABLE mykeyspace.users_score
(
    id    uuid PRIMARY KEY,
    score set<frozen<set<int>>>
);
```

## 4. 冻结用户定义类型

用户定义类型(UDT)可以将多个数据字段(每个字段都具有名称和类型)附加到单个列，用于创建用户定义类型的字段可以是任何有效的数据类型，包括集合或其他UDT，让我们创建UDT：

```sql
CREATE TYPE mykeyspace.address (
    city text,
    street text,
    streetNo int,
    zipcode text
);

```

让我们看一下冻结的用户定义类型的声明：

```sql
CREATE TABLE mykeyspace.building
(
    id      uuid PRIMARY KEY,
    address frozen<address>
);
```

当我们对用户定义类型使用frozen时，Cassandra会将该值视为一个Blob，这个Blob是通过将UDT序列化为单个值获得的。因此，**我们无法更新用户定义类型值的部分内容**，我们必须覆盖整个值。

首先，让我们插入一些数据：

```sql
INSERT INTO mykeyspace.building (id, address)
VALUES (6ab09bec-e68e-48d9-a5f8-97e6fb4c9b48,
  {city: 'City', street: 'Street', streetNo: 2,zipcode: '02-212'});
```

让我们看看当我们尝试仅更新一个字段时会发生什么：

```sql
UPDATE mykeyspace.building
SET address.city = 'City2'
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b48;
```

我们将再次收到错误：

```text
InvalidRequest: Error from server: code=2200 [Invalid query] message="Invalid operation (address.city = 'City2') for frozen UDT column address"
```

因此，让我们更新整个值：

```sql
UPDATE mykeyspace.building
SET address = {city : 'City2', street : 'Street2'}
WHERE id = 6ab09bec-e68e-48d9-a5f8-97e6fb4c9b48;
```

这次，address将被更新，查询中未包含的字段将以空值填充。

## 5. 元组

与其他组合类型不同，**元组始终处于冻结状态**。因此，我们不必使用冻结关键字标记元组。因此，无法仅更新元组中的某些元素，与冻结集合或UDT的情况一样，我们必须覆盖整个值。

## 6. 总结

在本快速教程中，我们探索了Cassandra数据库中冻结组件的基本概念。接下来，我们创建了冻结集合和用户定义类型。然后，我们检查了这些数据结构的行为。之后，我们讨论了元组数据类型。