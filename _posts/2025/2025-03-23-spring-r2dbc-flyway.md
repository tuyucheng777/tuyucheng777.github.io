---
layout: post
title:  使用Flyway进行Spring R2DBC迁移
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 简介

在本教程中，我们将使用[Flyway](https://www.baeldung.com/flyway-migrations)探索Spring R2DBC迁移，Flyway是一种常用于[数据库迁移](https://www.baeldung.com/database-migrations-with-flyway)的开源工具。虽然在撰写本文时它还没有对R2DBC的原生支持，但我们将研究在应用程序启动期间迁移表和数据的替代方法。

## 2. 基本Spring R2DBC应用程序

在本文中，我们将使用一个简单的Spring R2DBC应用程序，并使用Flyway进行迁移来创建表并将数据插入PostgreSQL数据库。

### 2.1 R2DBC

[R2DBC代表Reactive Relational Database](https://www.baeldung.com/r2dbc)，它基于Reactive Streams规范，该规范提供了用于与SQL数据库交互的完全响应式非阻塞API。然而，尽管它有很多好处，但[Flyway等一些工具目前不支持R2DBC](https://github.com/flyway/flyway/issues/2502)，这意味着使用Flyway进行迁移需要JDBC(阻塞)驱动程序。

数据库迁移允许应用程序在部署新版本时在启动时升级数据库模式，这些迁移是使数据库结构与应用程序需求相匹配所必需的。由于这些更改是在应用程序启动之前必须进行的，因此同步阻塞迁移是可以接受的。

### 2.2 依赖

我们需要一些依赖来使R2DBC应用程序与Flyway兼容。

我们需要[spring-boot-starter-data-r2dbc](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-r2dbc)依赖，它提供核心Spring R2DBC数据抽象和[PostgreSQL R2DBC](https://mvnrepository.com/artifact/io.r2dbc/r2dbc-postgresql)驱动程序：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
    <version>1.0.5.RELEASE</version>
</dependency>
```

为了配置数据库迁移，我们需要[Flyway](https://mvnrepository.com/artifact/org.flywaydb/flyway-core)：

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
    <version>10.12.0</version>
</dependency>
```

由于Flyway尚不支持R2DBC驱动程序，因此我们还需要添加PostgreSQL的标准[JDBC驱动程序](https://mvnrepository.com/artifact/org.postgresql/postgresql)：

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.3</version>
</dependency>
```

## 3. Spring R2DBC应用程序上的Flyway迁移

让我们看看Flyway迁移和示例脚本所需的配置。

### 3.1 Spring R2DBC和Flyway的配置

由于Flyway不能与R2DBC一起使用，**我们需要使用初始化方法migration()创建Flyway Bean，这会提示Spring在创建Bean后立即运行我们的迁移**：

```java
@Configuration
@EnableConfigurationProperties({ R2dbcProperties.class, FlywayProperties.class })
class DatabaseConfig {
    @Bean(initMethod = "migrate")
    public Flyway flyway(FlywayProperties flywayProperties, R2dbcProperties r2dbcProperties) {
        return Flyway.configure()
                .dataSource(
                        flywayProperties.getUrl(),
                        r2dbcProperties.getUsername(),
                        r2dbcProperties.getPassword()
                )
                .locations(flywayProperties.getLocations()
                        .stream()
                        .toArray(String[]::new))
                .baselineOnMigrate(true)
                .load();
    }
}
```

我们可以通过将R2DBC属性与一些特定于Flyway的覆盖合并来将Flyway指向我们的数据库，**特别是URL，它必须是JDBC连接URL和Flyway将从中运行迁移的位置**。

在这个特定的例子中，Spring能够根据类路径中的依赖自动配置[Postgresql R2DBC](https://github.com/pgjdbc/r2dbc-postgresql)，但值得注意的是，R2DBC也为[其他SQL数据库驱动程序](https://docs.spring.io/spring-data/relational/reference/r2dbc/getting-started.html#r2dbc.dialects)提供支持。由于我们的应用程序中有R2DBC Starter，我们只需要为Spring R2DBC设置适当的属性，而不需要额外的配置。

让我们看一个R2DBC PostgreSQL和Flyway设置的属性文件示例，其中包含JDBC驱动程序所需的数据库URL：

```yaml
spring:
    r2dbc:
        username: local
        password: local
        url: r2dbc:postgresql://localhost:8082/flyway-test-db
    flyway:
        url: jdbc:postgresql://localhost:8082/flyway-test-db
        locations: classpath:db/postgres/migration
```

虽然目录的默认位置是db/migration，但上述配置指定我们的迁移脚本位于db/postgres/migration目录中。

[R2DBC URL](https://r2dbc.io/spec/0.8.1.RELEASE/api/io/r2dbc/spi/ConnectionFactories.html)格式包括：

- jdbc/r2dbc：连接策略
- postgresql：数据库类型
- localhost:8082：PostgreSQLDB主机
- flyway-test-db：数据库名称

### 3.2 迁移脚本

让我们创建迁移脚本，创建两个表(department和student)，并在department表中插入一些数据。

我们的第一个脚本将创建department和student表，我们将其命名为V1_1__create_tables.sql，遵循[文件命名约定](https://www.baeldung.com/database-migrations-with-flyway#3-define-first-migration)，以便Flyway可以将其识别为第一个要执行的脚本。Flyway会跟踪它运行过的所有脚本，因此它只会运行每个脚本一次：

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE IF NOT EXISTS department
(
    ID uuid PRIMARY KEY UNIQUE DEFAULT uuid_generate_v4(),
    NAME varchar(255)
);

CREATE TABLE IF NOT EXISTS student
(
    ID uuid PRIMARY KEY UNIQUE DEFAULT uuid_generate_v4(),
    FIRST_NAME varchar(255),
    LAST_NAME varchar(255),
    DATE_OF_BIRTH DATE NOT NULL,
    DEPARTMENT uuid NOT NULL CONSTRAINT student_foreign_key1 REFERENCES department (ID)
);
```

我们的下一个脚本将向表中插入一些数据，我们将其命名为V1_2__insert_department.sql，以便它第二次运行：

```sql
insert into department(NAME) values ('Computer Science');
insert into department(NAME) values ('Biomedical');
```

### 3.3 测试Spring R2DBC迁移

让我们做一个快速测试来验证设置。

首先，让我们编写一个用于启动PostgreSQL的示例docker-compose.yml：

```yaml
version: '3.9'
networks:
    obref:
services:
    postgres_db_service:
        container_name: postgres_db_service
        image: postgres:11
        ports:
            - "8082:5432"
        hostname:   postgres_db_service
        environment:
            - POSTGRES_PASSWORD=local
            - POSTGRES_USER=local
            - POSTGRES_DB=flyway-test-db
```

第一次启动应用程序时，我们应该看到确认正在应用迁移的日志：

```text
INFO 95740 --- [  restartedMain] o.f.c.internal.license.VersionPrinter    : Flyway Community Edition 9.14.1 by Redgate
INFO 95740 --- [  restartedMain] o.f.c.internal.license.VersionPrinter    : See what's new here: https://flywaydb.org/documentation/learnmore/releaseNotes#9.14.1
INFO 95740 --- [  restartedMain] o.f.c.internal.license.VersionPrinter    : 
INFO 95740 --- [  restartedMain] o.f.c.i.database.base.BaseDatabaseType   : Database: jdbc:postgresql://localhost:8082/flyway-test-db (PostgreSQL 11.16)
INFO 95740 --- [  restartedMain] o.f.core.internal.command.DbValidate     : Successfully validated 2 migrations (execution time 00:00.007s)
INFO 95740 --- [  restartedMain] o.f.c.i.s.JdbcTableSchemaHistory         : Creating Schema History table "public"."flyway_schema_history" ...
INFO 95740 --- [  restartedMain] o.f.core.internal.command.DbMigrate      : Current version of schema "public": << Empty Schema >>
INFO 95740 --- [  restartedMain] o.f.core.internal.command.DbMigrate      : Migrating schema "public" to version "1.1 - create tables"
INFO 95740 --- [  restartedMain] o.f.core.internal.command.DbMigrate      : Migrating schema "public" to version "1.2 - insert department"
INFO 95740 --- [  restartedMain] o.f.core.internal.command.DbMigrate      : Successfully applied 2 migrations to schema "public", now at version v1.2 (execution time 00:00.045s)
INFO 95740 --- [  restartedMain] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port 8080
INFO 95740 --- [  restartedMain] c.t.t.e.r.f.SpringWebfluxFlywayApplication : Started SpringWebfluxFlywayApplication in 1.895 seconds (JVM running for 2.28)
```

从以上日志我们可以看到，迁移脚本按照预期的顺序应用。

如果flyway_schema_history表不可用，则Flyway会创建该表并存储迁移状态的详细信息。脚本完成后，我们可以从该表中查看有关迁移的详细信息，包括文件的名称和执行时间。

## 4. 总结

在本文中，我们创建了一个基本的Spring R2DBC应用程序，并探索了使用Flyway为使用R2DBC的应用程序迁移数据的方法之一。我们还采用了一些策略来验证Flyway是否按预期应用了我们的迁移。