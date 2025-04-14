---
layout: post
title:  Apache Cassandra的DataStax Java驱动程序简介
category: persistence
copyright: persistence
excerpt: Apache Cassandra
---

## 1. 概述

[Apache Cassandra](https://www.baeldung.com/cassandra-with-java)的DataStax发行版是一个生产就绪的分布式数据库，与开源Cassandra兼容。它增加了一些开源发行版中没有的功能，包括监控、改进的批处理和流数据处理。

DataStax还为其Apache Cassandra发行版提供了一个Java客户端，该驱动程序高度可调，可以利用DataStax发行版中的所有附加功能，同时它也与开源版本完全兼容。

在本教程中，我们将了解如何使用[Apache Cassandra的DataStax Java驱动程序](https://github.com/datastax/java-driver)连接到Cassandra数据库并执行基本的数据操作。

## 2. Maven依赖

为了使用适用于Apache Cassandra的DataStax Java驱动程序，我们需要将其包含在我们的类路径中。

使用Maven，我们只需将[java-driver-core](https://mvnrepository.com/artifact/com.datastax.oss/java-driver-core)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-core</artifactId>
    <version>4.1.0</version>
</dependency>
<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-query-builder</artifactId>
    <version>4.1.0</version>
</dependency>
```

## 3. 使用DataStax驱动程序

现在我们已经有了驱动程序，让我们看看可以用它做什么。

### 3.1 连接数据库

为了连接到数据库，我们将创建一个CqlSession：

```java
CqlSession session = CqlSession.builder().build();
```

如果我们没有明确定义任何连接点，构建器将默认为127.0.0.1:9042。

让我们创建一个连接器类，带有一些可配置的参数，以构建CqlSession：

```java
public class CassandraConnector {

    private CqlSession session;

    public void connect(String node, Integer port, String dataCenter) {
        CqlSessionBuilder builder = CqlSession.builder();
        builder.addContactPoint(new InetSocketAddress(node, port));
        builder.withLocalDatacenter(dataCenter);

        session = builder.build();
    }

    public CqlSession getSession() {
        return this.session;
    }

    public void close() {
        session.close();
    }
}
```

### 3.2 创建键空间

现在我们已经连接到数据库，我们需要创建键空间，让我们先编写一个简单的Repository类来处理键空间。

在本教程中，**我们将使用SimpleStrategy复制策略，并将副本数设置为1**：

```java
public class KeyspaceRepository {

    public void createKeyspace(String keyspaceName, int numberOfReplicas) {
        CreateKeyspace createKeyspace = SchemaBuilder.createKeyspace(keyspaceName)
                .ifNotExists()
                .withSimpleStrategy(numberOfReplicas);

        session.execute(createKeyspace.build());
    }

    // ...
}
```

另外，我们可以**开始使用当前会话中的键空间**：

```java
public class KeyspaceRepository {

    //...

    public void useKeyspace(String keyspace) {
        session.execute("USE " + CqlIdentifier.fromCql(keyspace));
    }
}
```

### 3.3 创建表

驱动程序提供了一些语句来配置和执行数据库中的查询，例如，**我们可以在每个语句中单独设置要使用的键空间**。

我们将定义Video模型并创建一个表来表示它：

```java
public class Video {
    private UUID id;
    private String title;
    private Instant creationDate;

    // standard getters and setters
}
```

让我们创建表，并定义要在其中执行查询的键空间，我们将编写一个简单的VideoRepository类来处理视频数据：

```java
public class VideoRepository {
    private static final String TABLE_NAME = "videos";

    public void createTable() {
        createTable(null);
    }

    public void createTable(String keyspace) {
        CreateTable createTable = SchemaBuilder.createTable(TABLE_NAME)
                .withPartitionKey("video_id", DataTypes.UUID)
                .withColumn("title", DataTypes.TEXT)
                .withColumn("creation_date", DataTypes.TIMESTAMP);

        executeStatement(createTable.build(), keyspace);
    }

    private ResultSet executeStatement(SimpleStatement statement, String keyspace) {
        if (keyspace != null) {
            statement.setKeyspace(CqlIdentifier.fromCql(keyspace));
        }

        return session.execute(statement);
    }

    // ...
}
```

请注意，我们这里重载方法createTable。

重载此方法背后的想法是为表创建提供两个选项：

- 在特定的键空间中创建表，提供键空间名称作为参数，与会话当前使用的键空间无关
- 在会话中使用键空间，并使用不带任何参数的方法创建表-在这种情况下，将在会话当前使用的键空间中创建表

### 3.4 插入数据

此外，该驱动程序还提供准备好的和有界的语句。

**PreparationStatement通常用于经常执行的查询，并且仅改变值**。

我们可以将所需的值填充到PreparedStatement中，之后，我们将创建一个BoundStatement并执行它。

让我们编写一个将一些数据插入数据库的方法：

```java
public class VideoRepository {

    //...

    public UUID insertVideo(Video video, String keyspace) {
        UUID videoId = UUID.randomUUID();

        video.setId(videoId);

        RegularInsert insertInto = QueryBuilder.insertInto(TABLE_NAME)
                .value("video_id", QueryBuilder.bindMarker())
                .value("title", QueryBuilder.bindMarker())
                .value("creation_date", QueryBuilder.bindMarker());

        SimpleStatement insertStatement = insertInto.build();

        if (keyspace != null) {
            insertStatement = insertStatement.setKeyspace(keyspace);
        }

        PreparedStatement preparedStatement = session.prepare(insertStatement);

        BoundStatement statement = preparedStatement.bind()
                .setUuid(0, video.getId())
                .setString(1, video.getTitle())
                .setInstant(2, video.getCreationDate());

        session.execute(statement);

        return videoId;
    }

    // ...
}
```

### 3.5 查询数据

现在，让我们添加一个方法来创建一个简单的查询来获取我们存储在数据库中的数据：

```java
public class VideoRepository {

    // ...

    public List<Video> selectAll(String keyspace) {
        Select select = QueryBuilder.selectFrom(TABLE_NAME).all();

        ResultSet resultSet = executeStatement(select.build(), keyspace);

        List<Video> result = new ArrayList<>();

        resultSet.forEach(x -> result.add(
                new Video(x.getUuid("video_id"), x.getString("title"), x.getInstant("creation_date"))
        ));

        return result;
    }

    // ...
}
```

### 3.6 整合

最后，让我们看一个使用本教程中介绍的每个部分的示例：

```java
public class Application {

    public void run() {
        CassandraConnector connector = new CassandraConnector();
        connector.connect("127.0.0.1", 9042, "datacenter1");
        CqlSession session = connector.getSession();

        KeyspaceRepository keyspaceRepository = new KeyspaceRepository(session);

        keyspaceRepository.createKeyspace("testKeyspace", 1);
        keyspaceRepository.useKeyspace("testKeyspace");

        VideoRepository videoRepository = new VideoRepository(session);

        videoRepository.createTable();

        videoRepository.insertVideo(new Video("Video Title 1", Instant.now()));
        videoRepository.insertVideo(new Video("Video Title 2",
                Instant.now().minus(1, ChronoUnit.DAYS)));

        List<Video> videos = videoRepository.selectAll();

        videos.forEach(x -> LOG.info(x.toString()));

        connector.close();
    }
}
```

执行示例后，我们可以在日志中看到数据已正确存储在数据库中：

```text
INFO cn.tuyucheng.taketoday.datastax.cassandra.Application - [id:733249eb-914c-4153-8698-4f58992c4ad4, title:Video Title 1, creationDate: 2019-07-10T19:43:35.112Z]
INFO cn.tuyucheng.taketoday.datastax.cassandra.Application - [id:a6568236-77d7-42f2-a35a-b4c79afabccf, title:Video Title 2, creationDate: 2019-07-09T19:43:35.181Z]
```

## 4. 总结

在本教程中，我们介绍了适用于Apache Cassandra的DataStax Java驱动程序的基本概念。我们连接到数据库并创建了键空间和表，此外，我们还将数据插入表中并运行查询来检索数据。