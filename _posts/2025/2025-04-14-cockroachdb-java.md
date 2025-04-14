---
layout: post
title:  Java CockroachDB指南
category: persistence
copyright: persistence
excerpt: CockroachDB
---

## 1. 简介

本教程是使用Java的CockroachDB的入门指南。

我们将解释主要功能、如何配置本地集群以及如何监控它，以及如何使用Java连接和与服务器交互的实用指南。

## 2. CockroachDB

CockroachDB是一个建立在事务性和一致性键值存储之上的分布式SQL数据库。

它使用Go语言编写，完全开源，**其主要设计目标是支持ACID事务、水平可扩展性和可存活性。基于这些设计目标**，它能够承受从单个磁盘故障到整个数据中心崩溃的各种情况，并将延迟影响降至最低，并且无需人工干预。

因此，**对于需要可靠、可用且正确数据(无论规模大小)的应用程序来说，CockroachDB是一个理想的解决方案**。然而，当极低延迟的读写操作至关重要时，它并非首选。

### 2.1 主要特点

让我们继续探索CockroachDB的一些关键方面：

- **SQL API和PostgreSQL兼容性**：用于构建、操作和查询数据
- **ACID事务**：支持分布式事务并提供强一致性
- **云就绪**：设计用于在云中或本地解决方案上运行，可在不同的云提供商之间轻松迁移，而不会中断任何服务
- **水平扩展**：增加容量就像在正在运行的集群上指向一个新节点一样简单，并且只需极少的操作员开销
- **复制**：复制数据以确保可用性并保证副本之间的一致性
- **自动修复**：只要大多数副本仍然可用于短期故障，就可以无缝继续；而对于长期故障，则使用未受影响的副本作为源，自动从丢失的节点重新平衡副本

## 3. 配置CockroachDB

[安装CockroachDB](https://www.cockroachlabs.com/docs/stable/install-cockroachdb.html)之后，我们可以启动本地集群的第一个节点：

```shell
cockroach start --insecure --host=localhost;
```

出于演示目的，我们使用insecure属性，使通信不加密，而无需指定证书位置。

此时，我们的本地集群已启动并运行。只有一个节点，我们已经可以连接并进行操作，**但为了更好地利用CockroachDB的自动复制、重新平衡和容错功能，我们将再添加两个节点**：

```shell
cockroach start --insecure --store=node2 \
  --host=localhost --port=26258 --http-port=8081 \
  --join=localhost:26257;

cockroach start --insecure --store=node3 \
  --host=localhost --port=26259 --http-port=8082 \
  --join=localhost:26257;
```

对于新增的两个节点，我们使用join标志将新节点连接到集群，并指定第一个节点的地址和端口，在本例中为localhost:26257。**本地集群上的每个节点都需要唯一的store、port和http-port值**。

在配置CockroachDB的分布式集群时，每个节点将位于不同的计算机上，因此无需指定port、store和http-port，因为默认值即可。此外，在将其他节点加入集群时，应使用第一个节点的实际IP。

### 3.1 配置数据库和用户

一旦我们的集群启动并运行，通过CockroachDB提供的SQL控制台，我们需要创建数据库和用户。

首先，让我们启动SQL控制台：

```shell
cockroach sql --insecure;
```

现在，让我们创建testdb数据库，创建一个用户并向该用户添加授权以便能够执行CRUD操作：

```sql
CREATE DATABASE testdb;
CREATE USER user17 with password 'qwerty';
GRANT ALL ON DATABASE testdb TO user17;
```

如果我们想验证数据库是否正确创建，我们可以列出当前节点中创建的所有数据库：

```sql
SHOW DATABASES;
```

最后，如果我们想验证CockroachDB的自动复制功能，我们可以在另外两个节点之一上检查数据库是否已正确创建。为此，我们必须在使用SQL控制台时显示port标志：

```shell
cockroach sql --insecure --port=26258;
```

## 4. 监控CockroachDB

现在我们已经启动了本地集群并创建了数据库，**我们可以使用CockroachDB管理UI来监控它们**：

![](/assets/images/2025/persistence/cockroachdbjava01.png)

此管理界面与CockroachDB捆绑在一起，集群启动并运行后即可通过http://localhost:8080访问，**它提供有关集群和数据库配置的详细信息，并通过监控以下指标来帮助我们优化集群性能**：

- **集群健康**：有关集群健康的基本指标
- **运行时指标**：有关节点数、CPU时间和内存使用情况的指标
- **SQL性能**：有关SQL连接、查询和事务的指标
- **复制详细信息**：有关如何在集群中复制数据的指标
- **节点详细信息**：活跃、死亡和退役节点的详细信息
- **数据库详细信息**：有关集群中的系统和用户数据库的详细信息

## 5. 项目设置

鉴于我们正在运行CockroachDB的本地集群，为了能够连接到它，我们必须在pom.xml中添加一个[额外的依赖](https://mvnrepository.com/artifact/org.postgresql/postgresql)：

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.1.4</version>
</dependency>
```

或者，对于Gradle项目：

```text
compile 'org.postgresql:postgresql:42.1.4'
```

## 6. 使用CockroachDB

由于与PostgreSQL兼容，**可以直接通过JDBC连接，也可以使用ORM(例如Hibernate)进行连接(截至本文撰写时(2018年1月)**，据开发人员称，这两个驱动程序均已经过充分测试，可以声称提供Beta级支持)。在本例中，我们将使用JDBC与数据库交互。

为了简单起见，我们将遵循基本的CRUD操作，因为它们是最好的入门操作。

让我们从连接数据库开始。

### 6.1 连接到CockroachDB

要打开与数据库的连接，我们可以使用DriverManager类的getConnection()方法，此方法需要一个连接URL字符串参数、一个用户名和一个密码：

```java
Connection con = DriverManager.getConnection(
    "jdbc:postgresql://localhost:26257/testdb", "user17", "qwerty"
);
```

### 6.2 创建表

通过创建连接，我们可以开始创建将用于所有CRUD操作的articles表：

```java
String TABLE_NAME = "articles";
StringBuilder sb = new StringBuilder("CREATE TABLE IF NOT EXISTS ")
    .append(TABLE_NAME)
    .append("(id uuid PRIMARY KEY, ")
    .append("title string,")
    .append("author string)");

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

如果我们想验证表是否正确创建，我们可以使用SHOW TABLES命令：

```java
PreparedStatement preparedStatement = con.prepareStatement("SHOW TABLES");
ResultSet resultSet = preparedStatement.executeQuery();
List tables = new ArrayList<>();
while (resultSet.next()) {
    tables.add(resultSet.getString("Table"));
}

assertTrue(tables.stream().anyMatch(t -> t.equals(TABLE_NAME)));
```

让我们看看如何修改刚刚创建的表。

### 6.3 修改表

如果我们在创建表时遗漏了某些列，或者因为我们稍后需要它们，可以轻松地添加：

```java
StringBuilder sb = new StringBuilder("ALTER TABLE ").append(TABLE_NAME)
    .append(" ADD ")
    .append(columnName)
    .append(" ")
    .append(columnType);

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

一旦我们改变了表，我们就可以使用SHOW COLUMNS FROM命令验证是否添加了新列：

```java
String query = "SHOW COLUMNS FROM " + TABLE_NAME;
PreparedStatement preparedStatement = con.prepareStatement(query);
ResultSet resultSet = preparedStatement.executeQuery();
List<String> columns = new ArrayList<>();
while (resultSet.next()) {
    columns.add(resultSet.getString("Field"));
}

assertTrue(columns.stream().anyMatch(c -> c.equals(columnName)));
```

### 6.4 删除表

使用表时，有时我们需要删除它们，只需几行代码就可以轻松实现：

```java
StringBuilder sb = new StringBuilder("DROP TABLE IF EXISTS ")
    .append(TABLE_NAME);

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

### 6.5 插入数据

一旦我们明确了可以在表上执行的操作，现在就可以开始处理数据了，我们可以开始定义Article类：

```java
public class Article {

    private UUID id;
    private String title;
    private String author;

    // standard constructor/getters/setters
}
```

可以看到如何将Article添加到我们的articles表中：

```java
StringBuilder sb = new StringBuilder("INSERT INTO ").append(TABLE_NAME)
    .append("(id, title, author) ")
    .append("VALUES (?,?,?)");

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, article.getId().toString());
preparedStatement.setString(2, article.getTitle());
preparedStatement.setString(3, article.getAuthor());
preparedStatement.execute();
```

### 6.6 读取数据

一旦数据存储在表中，就可以读取这些数据，这可以轻松实现：

```java
StringBuilder sb = new StringBuilder("SELECT * FROM ")
    .append(TABLE_NAME);

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
ResultSet rs = preparedStatement.executeQuery();
```

但是，如果我们不想读取articles表中的所有数据，而只想读取一条，我们可以简单地改变构建PreparedStatement的方式：

```java
StringBuilder sb = new StringBuilder("SELECT * FROM ").append(TABLE_NAME)
    .append(" WHERE title = ?");

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, title);
ResultSet rs = preparedStatement.executeQuery();
```

### 6.7 删除数据

最后但同样重要的一点是，如果我们想从表中删除数据，我们可以使用标准DELETE FROM命令删除一组有限的记录：

```java
StringBuilder sb = new StringBuilder("DELETE FROM ").append(TABLE_NAME)
    .append(" WHERE title = ?");

String query = sb.toString();
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, title);
preparedStatement.execute();
```

或者我们可以使用TRUNCATE函数删除表中的所有记录：

```java
StringBuilder sb = new StringBuilder("TRUNCATE TABLE ")
    .append(TABLE_NAME);

String query = sb.toString();
Statement stmt = connection.createStatement();
stmt.execute(query);
```

### 6.8 处理事务

一旦连接到数据库，默认情况下，每个单独的SQL语句都被视为一个事务，并在执行完成后立即自动提交。

但是，如果我们想将两个或多个SQL语句分组到一个事务中，我们必须以编程方式控制该事务。

首先，我们需要通过将Connection的autoCommit属性设置为false来禁用自动提交模式，然后使用commit()和rollback()方法来控制事务。

让我们看看在进行多次插入时如何实现数据一致性：

```java
try {
    con.setAutoCommit(false);

    UUID articleId = UUID.randomUUID();

    Article article = new Article(
        articleId, "Guide to CockroachDB in Java", "tuyucheng"
    );
    articleRepository.insertArticle(article);

    article = new Article(
        articleId, "A Guide to MongoDB with Java", "tuyucheng"
    );
    articleRepository.insertArticle(article); // Exception

    con.commit();
} catch (Exception e) {
    con.rollback();
} finally {
    con.setAutoCommit(true);
}
```

在这种情况下，第二次插入时由于违反主键约束而引发异常，因此articles表中没有插入任何文章。

## 7. 总结

在本文中，我们解释了什么是CockroachDB，如何设置一个简单的本地集群，以及如何从Java与其交互。