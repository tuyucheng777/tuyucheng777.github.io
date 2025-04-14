---
layout: post
title:  Java Cassandra指南
category: persistence
copyright: persistence
excerpt: Cassandra
---

## 1. 概述

本教程是使用Java的[Apache Cassandra](http://cassandra.apache.org/)数据库的入门指南。

你将找到关键概念的解释，以及涵盖从Java连接和开始使用此NoSQL数据库的基本步骤的工作示例。

## 2. Cassandra

Cassandra是一个可扩展的NoSQL数据库，它提供持续可用性，没有单点故障，并能够以卓越的性能处理大量数据。

该数据库采用环形设计，而非主从架构，环形设计中没有主节点，所有参与节点都是相同的，并以对等体的身份相互通信。

这使得Cassandra成为一个水平可扩展的系统，允许增量添加节点而无需重新配置。

### 2.1 关键概念

让我们首先简要介绍一下Cassandra的一些关键概念：

- **集群**：以环形架构排列的节点或数据中心的集合，每个集群必须分配一个名称，该名称随后将由参与的节点使用。
- **键空间**：如果你使用的是关系型数据库，那么模式就是Cassandra中相应的键空间。键空间是Cassandra中数据的最外层容器，每个键空间需要设置的主要属性包括复制因子、副本放置策略和列族。
- **列族**：Cassandra中的列族类似于关系数据库中的表，每个列族包含一组行，这些行由Map<RowKey, SortedMap<ColumnKey,ColumnValue\>\>表示，键值用于同时访问相关数据。
- **列**：Cassandra中的列是一种数据结构，包含列名、值和时间戳。与数据结构良好的关系数据库相比，列和每行的列数可能会有所不同。

## 3. 使用Java客户端

### 3.1 Maven依赖

我们需要在pom.xml中定义以下Cassandra依赖，其最新版本可以在[这里](https://mvnrepository.com/artifact/com.datastax.cassandra)找到：

```xml
<dependency>
    <groupId>com.datastax.cassandra</groupId>
    <artifactId>cassandra-driver-core</artifactId>
    <version>3.1.0</version>
</dependency>
```

为了测试代码，我们还应该添加测试容器依赖，其最新版本可以在[这里](https://mvnrepository.com/artifact/org.testcontainers/testcontainers/1.19.0)找到：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>cassandra</artifactId>
    <version>1.15.3</version>
</dependency>
```

### 3.2 连接到Cassandra

为了从Java连接到Cassandra，我们需要构建一个Cluster对象。

需要提供一个节点地址作为联系点，如果我们不提供端口号，则将使用默认端口(9042)。

这些设置允许驱动程序发现集群的当前拓扑。

```java
public class CassandraConnector {

    private Cluster cluster;

    private Session session;

    public void connect(String node, Integer port) {
        Builder b = Cluster.builder().addContactPoint(node);
        if (port != null) {
            b.withPort(port);
        }
        cluster = b.build();

        session = cluster.connect();
    }

    public Session getSession() {
        return this.session;
    }

    public void close() {
        session.close();
        cluster.close();
    }
}
```

### 3.3 创建键空间

让我们创建“library”键空间：

```java
public void createKeyspace(String keyspaceName, String replicationStrategy, int replicationFactor) {
    StringBuilder sb =
            new StringBuilder("CREATE KEYSPACE IF NOT EXISTS ")
                    .append(keyspaceName).append(" WITH replication = {")
                    .append("'class':'").append(replicationStrategy)
                    .append("','replication_factor':").append(replicationFactor)
                    .append("};");

    String query = sb.toString();
    session.execute(query);
}
```

除了keyspaceName之外，我们还需要定义两个参数，即replicationFactor和replicationStrategy，这两个参数分别决定了副本的数量以及副本在环上的分布方式。

通过复制，Cassandra通过在多个节点存储数据副本来确保可靠性和容错能力。

此时我们可以测试我们的键空间是否已成功创建：

```java
private KeyspaceRepository schemaRepository;
private Session session;

@Before
public void connect() {
    CassandraConnector client = new CassandraConnector();
    client.connect("127.0.0.1", 9142);
    this.session = client.getSession();
    schemaRepository = new KeyspaceRepository(session);
}

@Test
public void whenCreatingAKeyspace_thenCreated() {
    String keyspaceName = "library";
    schemaRepository.createKeyspace(keyspaceName, "SimpleStrategy", 1);

    ResultSet result =
            session.execute("SELECT * FROM system_schema.keyspaces;");

    List<String> matchedKeyspaces = result.all()
            .stream()
            .filter(r -> r.getString(0).equals(keyspaceName.toLowerCase()))
            .map(r -> r.getString(0))
            .collect(Collectors.toList());

    assertEquals(matchedKeyspaces.size(), 1);
    assertTrue(matchedKeyspaces.get(0).equals(keyspaceName.toLowerCase()));
}
```

### 3.4 创建列族

现在，我们可以将第一个列族“books”添加到现有的键空间中：

```java
private static final String TABLE_NAME = "books";
private Session session;

public void createTable() {
    StringBuilder sb = new StringBuilder("CREATE TABLE IF NOT EXISTS ")
            .append(TABLE_NAME).append("(")
            .append("id uuid PRIMARY KEY, ")
            .append("title text,")
            .append("subject text);");

    String query = sb.toString();
    session.execute(query);
}
```

用于测试列族是否已创建的代码如下：

```java
private BookRepository bookRepository;
private Session session;

@Before
public void connect() {
    CassandraConnector client = new CassandraConnector();
    client.connect("127.0.0.1", 9142);
    this.session = client.getSession();
    bookRepository = new BookRepository(session);
}

@Test
public void whenCreatingATable_thenCreatedCorrectly() {
    bookRepository.deleteTable(BOOKS);
    bookRepository.createTable();
    ResultSet result = session.execute("SELECT * FROM " + KEYSPACE_NAME + "." + BOOKS + ";");

    // Collect all the column names in one list.
    List columnNames = result.getColumnDefinitions().asList().stream().map(cl -> cl.getName()).collect(Collectors.toList());
    assertEquals(columnNames.size(), 4);
    assertTrue(columnNames.contains("id"));
    assertTrue(columnNames.contains("title"));
    assertTrue(columnNames.contains("author"));
    assertTrue(columnNames.contains("subject"));
}
```

### 3.5 修改列族

书也有出版商，但在创建的表中找不到此列，我们可以使用以下代码修改表并添加一个新列：

```java
public void alterTablebooks(String columnName, String columnType) {
    StringBuilder sb = new StringBuilder("ALTER TABLE ")
            .append(TABLE_NAME).append(" ADD ")
            .append(columnName).append(" ")
            .append(columnType).append(";");

    String query = sb.toString();
    session.execute(query);
}
```

让我们确保已添加新的列publisher：

```java
@Test
public void whenAlteringTable_thenAddedColumnExists() {
    bookRepository.deleteTable(BOOKS);
    bookRepository.createTable();

    bookRepository.alterTablebooks("publisher", "text");

    ResultSet result = session.execute("SELECT * FROM " + KEYSPACE_NAME + "." + BOOKS + ";");

    boolean columnExists = result.getColumnDefinitions().asList().stream().anyMatch(cl -> cl.getName().equals("publisher"));
    assertTrue(columnExists);
}
```

### 3.6 在列族中插入数据

现在已经创建了books表，我们可以开始向表中添加数据了：

```java
public void insertbookByTitle(Book book) {
    StringBuilder sb = new StringBuilder("INSERT INTO ")
            .append(TABLE_NAME_BY_TITLE).append("(id, title) ")
            .append("VALUES (").append(book.getId())
            .append(", '").append(book.getTitle()).append("');");

    String query = sb.toString();
    session.execute(query);
}
```

“books”表中添加了一个新行，因此我们可以测试该行是否存在：

```java
@Test
public void whenAddingANewBook_thenBookExists() {
    bookRepository.deleteTable(BOOKS_BY_TITLE);
    bookRepository.createTableBooksByTitle();

    String title = "Effective Java";
    String author = "Joshua Bloch";
    Book book = new Book(UUIDs.timeBased(), title, author, "Programming");
    bookRepository.insertbookByTitle(book);

    Book savedBook = bookRepository.selectByTitle(title);
    assertEquals(book.getTitle(), savedBook.getTitle());
}
```

在上面的测试代码中，我们使用了不同的方法来创建一个名为booksByTitle的表：

```java
public void createTableBooksByTitle() {
    StringBuilder sb = new StringBuilder("CREATE TABLE IF NOT EXISTS ")
            .append("booksByTitle").append("(")
            .append("id uuid, ")
            .append("title text,")
            .append("PRIMARY KEY (title, id));");

    String query = sb.toString();
    session.execute(query);
}
```

在Cassandra中，最佳实践之一是使用每个查询一个表的模式。这意味着，对于不同的查询，需要不同的表。

在我们的示例中，我们按title选择一本书，为了满足selectByTitle查询，我们创建了一个包含复合主键的表，该主键由title和id列组成。其中，title列是分区键，而id列是聚类键。

这样，数据模型中的许多表都会包含重复数据。这并非该数据库的缺点，相反，这种做法可以优化读取性能。

让我们看看当前保存在表中的数据：

```java
public List<Book> selectAll() {
    StringBuilder sb = new StringBuilder("SELECT * FROM ").append(TABLE_NAME);

    String query = sb.toString();
    ResultSet rs = session.execute(query);

    List<Book> books = new ArrayList<Book>();

    rs.forEach(r -> books.add(new Book(
            r.getUUID("id"),
            r.getString("title"),
            r.getString("subject"))));
    return books;
}
```

对查询返回预期结果的测试：

```java
@Test
public void whenSelectingAll_thenReturnAllRecords() {
    bookRepository.deleteTable(BOOKS);
    bookRepository.createTable();

    Book book = new Book(UUIDs.timeBased(), "Effective Java", "Joshua Bloch", "Programming");
    bookRepository.insertbook(book);

    book = new Book(UUIDs.timeBased(), "Clean Code", "Robert C. Martin", "Programming");
    bookRepository.insertbook(book);

    List<Books> books = bookRepository.selectAll();

    assertEquals(2, books.size());
    assertTrue(books.stream().anyMatch(b -> b.getTitle().equals("Effective Java")));
    assertTrue(books.stream().anyMatch(b -> b.getTitle().equals("Clean Code")));
}
```

到目前为止一切都很好，但有一件事必须注意。我们开始使用表books，但与此同时，为了满足按title列的选择查询，我们必须创建另一个名为booksByTitle的表。

这两个表完全相同，但包含重复的列，但我们只在booksByTitle表中插入了数据。因此，两个表中的数据目前不一致。

我们可以使用批量查询来解决这个问题，批量查询包含两条插入语句，每张表一条，批量查询将多条DML语句作为单个操作执行。

提供了此类查询的一个示例：

```java
public void insertBookBatch(Book book) {
    StringBuilder sb = new StringBuilder("BEGIN BATCH ")
            .append("INSERT INTO ").append(TABLE_NAME)
            .append("(id, title, subject) ")
            .append("VALUES (").append(book.getId()).append(", '")
            .append(book.getTitle()).append("', '")
            .append(book.getSubject()).append("');")
            .append("INSERT INTO ")
            .append(TABLE_NAME_BY_TITLE).append("(id, title) ")
            .append("VALUES (").append(book.getId()).append(", '")
            .append(book.getTitle()).append("');")
            .append("APPLY BATCH;");

    String query = sb.toString();
    session.execute(query);
}
```

我们再次测试批量查询结果如下：

```java
@Test
public void whenAddingANewBookBatch_ThenBookAddedInAllTables() {
    // Create table books
    bookRepository.deleteTable(BOOKS);
    bookRepository.createTable();

    // Create table booksByTitle
    bookRepository.deleteTable(BOOKS_BY_TITLE);
    bookRepository.createTableBooksByTitle();

    String title = "Effective Java";
    String author = "Joshua Bloch";
    Book book = new Book(UUIDs.timeBased(), title, author, "Programming");
    bookRepository.insertBookBatch(book);

    List<Book> books = bookRepository.selectAll();

    assertEquals(1, books.size());
    assertTrue(books.stream().anyMatch(b -> b.getTitle().equals("Effective Java")));

    List<Book> booksByTitle = bookRepository.selectAllBookByTitle();

    assertEquals(1, booksByTitle.size());
    assertTrue(booksByTitle.stream().anyMatch(b -> b.getTitle().equals("Effective Java")));
}
```

注意：从3.0版本开始，新增了一项名为“物化视图”的功能，我们可以用它来代替批量查询。“物化视图”的详细示例可在[此处](http://www.datastax.com/dev/blog/new-in-cassandra-3-0-materialized-views)获取。

### 3.7 删除列族

下面的代码显示如何删除表：

```java
public void deleteTable() {
    StringBuilder sb = new StringBuilder("DROP TABLE IF EXISTS ").append(TABLE_NAME);

    String query = sb.toString();
    session.execute(query);
}
```

选择键空间中不存在的表会导致InvalidQueryException：未配置的表books：

```java
@Test(expected = InvalidQueryException.class)
public void whenDeletingATable_thenUnconfiguredTable() {
    bookRepository.createTable();
    bookRepository.deleteTable("books");
       
    session.execute("SELECT * FROM " + KEYSPACE_NAME + ".books;");
}
```

### 3.8 删除键空间

最后，让我们删除键空间：

```java
public void deleteKeyspace(String keyspaceName) {
    StringBuilder sb = new StringBuilder("DROP KEYSPACE ").append(keyspaceName);

    String query = sb.toString();
    session.execute(query);
}
```

并测试键空间是否已被删除：

```java
@Test
public void whenDeletingAKeyspace_thenDoesNotExist() {
    String keyspaceName = "testTuyuchengKeyspace";

    schemaRepository.createKeyspace(keyspaceName, "SimpleStrategy", 1);
    schemaRepository.deleteKeyspace(keyspaceName);

    ResultSet result = session.execute("SELECT * FROM system_schema.keyspaces;");
    boolean isKeyspaceCreated = result.all().stream().anyMatch(r -> r.getString(0).equals(keyspaceName.toLowerCase()));
    assertFalse(isKeyspaceCreated);
}
```

## 4. 总结

本教程涵盖了使用Java连接和使用Cassandra数据库的基本步骤，此外，我们还讨论了该数据库的一些关键概念，以帮助你快速入门。