---
layout: post
title:  PSQLException：服务器请求基于密码的身份验证
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

使用PostgreSQL数据库为我们的Spring Boot项目配置数据源时，一个常见的陷阱是提供错误的数据库连接密码，甚至忘记所提供用户的密码。

这就是为什么我们在启动项目并连接数据库时可能会遇到错误的原因。在本教程中，我们将学习如何避免PSQLException。

## 2. 数据源配置

让我们研究一下在Spring Boot中配置数据源和数据库连接的两种最常用技术。我们只能使用这两种方法中的一种：文件[application.properties或application.yml](https://www.baeldung.com/spring-boot-yaml-vs-properties)。

### 2.1 application.properties

**现在我们创建并配置application.properties文件，其中包含连接所需的最少字段**：

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/tutorials
spring.datasource.username=postgres
spring.datasource.password=
spring.jpa.generate-ddl=true
```

应用程序通过添加属性spring.jpa.generate-ddl在启动时创建表。

**我们不在文件中提供密码，而是尝试从命令提示符(在Windows上)启动我们的应用程序，看看会发生什么**：

```shell
mvn spring-boot:run
```

**我们注意到由于没有提供密码进行身份验证而出现错误**：

```text
org.postgresql.util.PSQLException: The server requested password-based authentication, but no password was provided.
```

如果我们使用较新版本的PostgreSQL数据库，这种消息可能会稍微延迟，并且我们可能会看到基于SCRAM的身份验证错误：

```text
org.postgresql.util.PSQLException: The server requested SCRAM-based authentication, but no password was provided.
```

### 2.2 application.yml

在继续之前，我们必须注释掉文件application.properties的内容或者从解决方案中删除该文件，这样它就不会与application.yml文件冲突。

**让我们创建并配置application.yml文件，其中包含我们之前所做的最少必需字段**：

```yaml
spring:
    datasource:
        url: jdbc:postgresql://localhost:5432/tutorials
        username: postgres
        password:
    jpa:
        generate-ddl: true
```

我们添加了属性generate-ddl来在启动时创建表。

**通过不在文件中填写密码并尝试从命令提示符启动我们的应用程序，我们注意到与之前相同的错误**：

```text
org.postgresql.util.PSQLException: The server requested password-based authentication, but no password was provided.
```

此外，如果我们使用较新的PostgreSQL数据库，则在这种情况下错误消息可能会略有延迟，而是显示基于SCRAM的身份验证错误。

### 2.3 提供密码

**在我们选择使用的任何一种配置中，如果我们在请求的参数中正确地输入密码，服务器将成功启动**。

否则，将显示特定的错误消息：

```text
org.postgresql.util.PSQLException: FATAL: password authentication failed for user "postgres"
```

现在我们使用正确的密码成功建立数据库连接并且应用程序正确启动：

```text
2024-07-19T00:03:33.429+03:00 INFO 18708 --- [ restartedMain] cn.tuyucheng.taketoday.boot.Application : Started Application in 0.484 seconds 
2024-07-19T00:03:33.179+03:00  INFO 18708 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-9 - Starting...
2024-07-19T00:03:33.246+03:00  INFO 18708 --- [  restartedMain] com.zaxxer.hikari.pool.HikariPool        : HikariPool-9 - Added connection org.postgresql.jdbc.PgConnection@76116e4a
2024-07-19T00:03:33.247+03:00  INFO 18708 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-9 - Start completed.
```

## 3. 数据库密码重置

或者，如果我们忘记或选择更改或重置数据库用户或默认用户的密码，我们可以选择更改或重置密码。

现在让我们深入了解如何重置默认用户postgres的PostgreSQL密码。

### 3.1 重置默认用户的密码

首先，我们确定安装PostgreSQL的数据目录的位置，如果在Windows上，理想情况下位于”C:\Program Files\PostgreSQL\16\data”内。

**然后，让我们通过将pg_hba.conf文件复制到其他位置或将其重命名为pg_hba.conf.backup来备份它**。在data目录中打开命令提示符并运行以下命令：

```shell
copy "pg_hba.conf" "pg_hba.conf.backup"
```

**其次，编辑pg_dba.conf文件并将所有本地连接从scram-sha-256更改为trust，以便我们可以无需密码登录PostgreSQL数据库服务器**：

```text
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust
```

**第三，使用服务功能重新启动PostgreSQL服务器(在Windows上)。或者，我们可以在命令提示符中以管理员身份运行以下命令**：

```shell
pg_ctl -D "C:\Program Files\PostgreSQL\16\data" restart
```

之后，我们使用psql或pgAdmin等工具连接到PostgreSQL数据库服务器。

**在PostgreSQL安装文件夹下的bin目录中打开命令提示符，然后输入以下psql命令**：

```shell
psql -U postgres
```

**我们已经登录数据库，因为PostgreSQL不需要密码。让我们通过执行以下命令更改用户postgres的密码**：

```shell
ALTER USER postgres WITH PASSWORD 'new_password';
```

最后，让我们恢复pg_dba.conf文件并像以前一样重新启动PostgreSQL数据库服务器。现在，我们可以使用配置文件中的新密码连接到PostgreSQL数据库。

### 3.2 为其他用户重置密码

通过选择使用psql执行此操作(在Windows上)，我们在PostgreSQL安装bin目录中打开命令提示符并运行以下命令：

```shell
psql -U postgres
```

然后我们提供postgres用户的密码并登录。

**以超级用户postgres身份登录后，让我们更改所需的用户的密码**：

```shell
ALTER USER user_name WITH PASSWORD 'new_password';
```

## 4. 总结

在本文中，我们看到了在Spring Boot应用程序中配置数据源时常见的连接问题以及解决该问题的各种选项。