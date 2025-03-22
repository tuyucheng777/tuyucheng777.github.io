---
layout: post
title:  使用Spring Boot和Flyway Repair
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

Flyway迁移并不总是按计划进行，在本教程中，我们将探索从失败的迁移中恢复的选项。

## 2. 设置

让我们从一个基本的Flyway配置的Spring Boot项目开始，它具有[flyway-core](https://mvnrepository.com/artifact/org.flywaydb/flyway-core)、[spring-boot-starter-jdbc](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-jdbc)和[flyway-maven-plugin](https://mvnrepository.com/artifact/org.flywaydb/flyway-maven-plugin)依赖项。

更多配置细节请参考我们[Flyway介绍](https://www.baeldung.com/database-migrations-with-flyway)的文章。

### 2.1 配置

首先，让我们添加两个不同的profile，这将使我们能够轻松地针对不同的数据库引擎运行迁移：

```xml
<profile>
    <id>h2</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
	    <artifactId>h2</artifactId>
        </dependency>
    </dependencies>
</profile>
<profile>
    <id>postgre</id>
    <dependencies>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
    </dependencies>
</profile>
```

我们还为每个Profile添加Flyway数据库配置文件。

首先，我们创建application-h2.properties：

```properties
flyway.url=jdbc:h2:file:./testdb;DB_CLOSE_ON_EXIT=FALSE;AUTO_RECONNECT=TRUE;MODE=MySQL;DATABASE_TO_UPPER=false;
flyway.user=testuser
flyway.password=password
```

然后，让我们创建PostgreSQL的application-postgre.properties：

```properties
flyway.url=jdbc:postgresql://127.0.0.1:5431/testdb
flyway.user=testuser
flyway.password=password
```

注意：我们可以调整PostgreSQL配置以匹配已经存在的数据库，或者我们可以使用[代码示例中的docker-compose文件](https://github.com/eugenp/tutorials/tree/master/persistence-modules/flyway)。

### 2.2 迁移

让我们添加第一个迁移文件V1_0__add_table.sql：

```sql
create table table_one (
    id numeric primary key
);
```

现在让我们添加第二个包含错误的迁移文件V1_1__add_table.sql：

```sql
create table table_one (
    id numeric primary key
);
```

**我们故意犯了一个错误，使用了相同的表名，这会导致Flyway迁移错误**。

## 3. 运行迁移

现在，让我们运行应用程序并尝试应用迁移。

首先是默认的h2 Profile：

```shell
mvn spring-boot:run
```

然后对于postgre Profile：

```shell
mvn spring-boot:run -Ppostgre
```

正如预期的那样，第一次迁移成功，而第二次迁移失败：

```text
Migration V1_1__add_table.sql failed
...
Message    : Table "TABLE_ONE" already exists; SQL statement:
```

### 3.1 检查状态

在继续修复数据库之前，让我们通过运行以下命令检查Flyway迁移状态：

```shell
mvn flyway:info -Ph2
```

正如预期的那样，返回结果如下：

```text
+-----------+---------+-------------+------+---------------------+---------+
| Category  | Version | Description | Type | Installed On        | State   |
+-----------+---------+-------------+------+---------------------+---------+
| Versioned | 1.0     | add table   | SQL  | 2020-07-17 12:57:35 | Success |
| Versioned | 1.1     | add table   | SQL  | 2020-07-17 12:57:35 | Failed  |
+-----------+---------+-------------+------+---------------------+---------+
```

但是当我们使用以下命令检查PostgreSQL的状态时：

```shell
mvn flyway:info -Ppostgre
```

我们注意到第二次迁移的状态是“Pending”，而不是“Failed”：

```shell
+-----------+---------+-------------+------+---------------------+---------+
| Category  | Version | Description | Type | Installed On        | State   |
+-----------+---------+-------------+------+---------------------+---------+
| Versioned | 1.0     | add table   | SQL  | 2020-07-17 12:57:48 | Success |
| Versioned | 1.1     | add table   | SQL  |                     | Pending |
+-----------+---------+-------------+------+---------------------+---------+
```

**差异在于PostgreSQL支持DDL事务，而其他数据库(如H2或MySQL)则不支持**。因此，**PostgreSQL能够回滚失败迁移的事务**。让我们看看当我们尝试修复数据库时，这种差异会如何影响情况。

### 3.2 更正错误并重新运行迁移

让我们通过将表名从table_one更正为table_two来修复迁移文件V1_1__add_table.sql。

现在，让我们尝试再次运行该应用程序：

```shell
mvn spring-boot:run -Ph2
```

我们现在注意到H2迁移失败：

```text
Validate failed: 
Detected failed migration to version 1.1 (add table)
```

只要此版本的迁移已经失败，Flyway就不会重新运行1.1版本的迁移。

另一方面，postgre Profile成功运行。如前所述，由于回滚，状态很干净，可以应用更正后的迁移。

实际上，通过运行mvn flyway:info -Ppostgre我们可以看到两个迁移都成功应用。因此，总而言之，对于PostgreSQL，我们所要做的就是更正我们的迁移脚本并重新触发迁移。

## 4. 手动修复数据库状态

修复数据库状态的第一种方法是**手动从flyway_schema_history表中删除Flyway条目**。

让我们简单地对数据库运行此SQL语句：

```sql
delete from flyway_schema_history where version = '1.1';
```

现在，当我们再次运行mvn spring-boot:run时，我们会看到迁移已成功应用。

但是，直接操作数据库可能不是理想的选择。所以，让我们看看我们还有什么其他选择。

## 5. Flyway Repair

### 5.1 修复失败的迁移

让我们继续添加另一个损坏的迁移V1_2__add_table.sql文件，运行应用程序并回到迁移失败的状态。

修复数据库状态的另一种方法是使用[flyway:repair](https://flywaydb.org/documentation/command/repair)工具。更正SQL文件后，我们无需手动接触flyway_schema_history表，而是可以运行：

```shell
mvn flyway:repair
```

这将导致：

```text
Successfully repaired schema history table "PUBLIC"."flyway_schema_history"
```

在幕后，Flyway只是从flyway_schema_history表中删除失败的迁移条目。

现在，我们可以再次运行flyway:info并看到最后一次迁移的状态从Failed变为Pending。

让我们再次运行该应用程序，我们可以看到，更正后的迁移现已成功应用。

### 5.2 重新对齐校验和

通常建议不要更改已成功应用的迁移，但有些情况下可能别无选择。

因此，在这种情况下，让我们通过在文件开头添加注释来更改迁移V1_1__add_table.sql。

现在运行该应用程序，我们会看到**“Migration checksum mismatch”错误消息**，如下所示：

```text
Migration checksum mismatch for migration version 1.1
-> Applied to database : 314944264
-> Resolved locally    : 1304013179
```

发生这种情况是因为我们改变了已经应用的迁移并且Flyway检测到了不一致。

**为了重新调整校验和，我们可以使用相同的flyway:repair命令**。但是，这次不会执行任何迁移。只有flyway_schema_history表中版本1.1条目的校验和才会更新，以反映更新的迁移文件。

修复后，通过再次运行该应用程序，我们注意到应用程序现在可以成功启动。

请注意，在本例中，我们通过Maven使用了flyway:repair。另一种方法是安装Flyway命令行工具并运行[flyway repair](https://flywaydb.org/documentation/command/repair)。效果是一样的：**flyway repair将从flyway_schema_history表中删除失败的迁移，并重新调整已应用迁移的校验和**。

## 6. Flyway回调

如果我们不想手动干预，我们可以考虑**在迁移失败后自动从flyway_schema_history中清除失败条目的方法**。为此，我们可以使用afterMigrateError [Flyway回调](https://www.baeldung.com/flyway-callbacks)。

我们首先创建SQL回调文件db/callback/afterMigrateError__repair.sql：

```sql
DELETE FROM flyway_schema_history WHERE success=false;
```

每当发生迁移错误时，这将自动从Flyway状态历史记录中删除任何失败的条目。

让我们创建一个application-callbacks.properties Profile配置，它将在Flyway位置列表中包含db/callback文件夹：

```properties
spring.flyway.locations=classpath:db/migration,classpath:db/callback
```

现在，在添加了另一个损坏的迁移V1_3__add_table.sql之后，我们运行包括回调Profile的应用程序：

```shell
mvn spring-boot:run -Dspring-boot.run.profiles=h2,callbacks
...
Migrating schema "PUBLIC" to version 1.3 - add table
Migration of schema "PUBLIC" to version 1.3 - add table failed!
...
Executing SQL callback: afterMigrateError - repair
```

正如预期的那样，迁移失败，但是afterMigrateError回调运行并清理了flyway_schema_history。

只需更正V1_3__add_table.sql迁移文件并再次运行应用程序就足以应用于更正后的迁移。

## 7. 总结

在本文中，我们研究了从失败的Flyway迁移中恢复的不同方法。

我们看到，像PostgreSQL这样的数据库(即支持DDL事务的数据库)不需要额外的努力来修复Flyway数据库状态。

另一方面，对于像H2这样没有这种支持的数据库，我们了解了如何使用Flyway Repair来清理Flyway历史记录并最终应用更正的迁移。