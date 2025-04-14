---
layout: post
title:  ActiveJDBC简介
category: persistence
copyright: persistence
excerpt: ActiveJDBC
---

## 1. 简介

ActiveJDBC是一个轻量级ORM，遵循Ruby on Rails的主要ORM ActiveRecord的核心思想。

**它专注于通过删除典型持久化管理器的额外层来简化与数据库的交互，并专注于SQL的使用而不是创建新的查询语言**。

此外，它还通过DBSpec类提供了自己的编写数据库交互单元测试的方法。

让我们看看这个库与其他流行的Java ORM有何不同以及如何使用它。

## 2. ActiveJDBC与其他ORM对比

ActiveJDBC与大多数其他Java ORM相比有着显著的不同，**它从数据库中推断出数据库模式参数，从而无需将实体映射到底层表**。

**无需会话，无需持久化管理器**，无需学习新的查询语言，无需Getter/Setter方法，该库本身在大小和依赖数量方面都很轻量。

这种实现鼓励使用测试数据库，测试数据库在执行测试后由框架清理，从而降低维护测试数据库的成本。

然而，每当我们创建或更新模型时，都需要一些额外的[检测步骤](http://javalite.io/instrumentation)，我们将在接下来的章节中讨论这一点。

## 3. 设计原则

- 从数据库推断元数据
- 基于约定的配置
- 没有会话，没有“连接、重新连接”
- 轻量级模型，简单的POJO
- 无代理
- 避免贫血领域模型
- 不需要DAO和DTO

## 4. 设置库

使用MySQL数据库的典型Maven设置包括：

```xml
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activejdbc</artifactId>
    <version>3.4-j11</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.32</version>
</dependency>
```

可以在Maven Central仓库中找到[activejdbc](https://mvnrepository.com/artifact/org.javalite/activejdbc)和[mysql连接器](https://mvnrepository.com/artifact/mysql/mysql-connector-java)工件的最新版本。

**[检测](http://javalite.io/instrumentation)是简化的代价，在使用ActiveJDBC项目时是必需的**。

项目中有一个需要配置的检测插件：

```xml
<plugin>
    <groupId>org.javalite</groupId>
    <artifactId>activejdbc-instrumentation</artifactId>
    <version>3.4-j11</version>
    <executions>
        <execution>
            <phase>process-classes</phase>
            <goals>
                <goal>instrument</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

最新的[activejdbc-instrumentation](https://mvnrepository.com/artifact/org.javalite/activejdbc-instrumentation)插件也可以在Maven Central中找到。

现在，我们可以通过执行以下两个命令之一来处理检测：

```shell
mvn process-classes
mvn activejdbc-instrumentation:instrument
```

## 5. 使用ActiveJDBC

### 5.1 模型

我们只需一行代码就可以创建一个简单的模型-它涉及扩展Model类。

该库利用英语的词[形变化](http://javalite.io/english_inflections)来实现名词复数和单数形式的转换，可以使用@Table注解覆盖此功能。

让我们看看一个简单的模型是什么样的：

```java
import org.javalite.activejdbc.Model;

public class Employee extends Model {}
```

### 5.2 连接数据库

提供了两个类Base和DB用于连接数据库。

连接数据库的最简单方法是：

```java
Base.open("com.mysql.jdbc.Driver", "jdbc:mysql://host/organization", "user", "xxxxx");
```

模型运行时，会使用当前线程中的连接，该连接由Base类或DB类在任何DB操作之前放置在本地线程上。

上述方法允许使用更简洁的API，从而无需像其他Java ORM那样使用DB Session或持久化管理器。

让我们看看如何使用DB类连接数据库：

```java
new DB("default").open(
    "com.mysql.jdbc.Driver", 
    "jdbc:mysql://localhost/dbname", 
    "root", 
    "XXXXXX");
```

如果我们观察Base和DB用于连接数据库的不同方式，我们就能得出结论：如果操作单个数据库，应该使用Base；如果操作多个数据库，应该使用DB。

### 5.3 插入记录

向数据库添加记录非常简单。如前所述，不需要Setter和Getter：

```java
Employee e = new Employee();
e.set("first_name", "Hugo");
e.set("last_name", "Choi");
e.saveIt();
```

或者，我们可以用这种方式添加相同的记录：

```java
Employee employee = new Employee("Hugo","Choi");
employee.saveIt();
```

甚至可以流式地写：

```java
new Employee()
    .set("first_name", "Hugo", "last_name", "Choi")
    .saveIt();
```

### 5.4 更新记录

下面的代码片段展示了如何更新记录：

```java
Employee employee = Employee.findFirst("first_name = ?", "Hugo");
employee
    .set("last_name","Choi")
    .saveIt();
```

### 5.5 删除记录

```java
Employee e = Employee.findFirst("first_name = ?", "Hugo");
e.delete();
```

如果需要删除所有记录：

```java
Employee.deleteAll();
```

如果想从主表中删除级联到子表的记录，请使用deleteCascade：

```java
Employee employee = Employee.findFirst("first_name = ?","Hugo");
employee.deleteCascade();
```

### 5.6 获取记录

让我们从数据库中获取一条记录：

```java
Employee e = Employee.findFirst("first_name = ?", "Hugo");
```

如果我们想要获取多条记录，可以使用where方法：

```java
List<Employee> employees = Employee.where("first_name = ?", "Hugo");
```

## 6. 事务支持

在Java ORM中，存在显式的连接或管理器对象(例如JPA中的EntityManager、Hibernate中的SessionManager等等)，但在ActiveJDBC中没有这些。

调用Base.open()打开一个连接，并将其连接到当前线程，这样所有模型的后续方法都会重用此连接。调用Base.close()关闭连接并将其从当前线程中移除。

为了管理事务，有几个方便的调用：

开始事务：

```java
Base.openTransaction();
```

提交事务：

```java
Base.commitTransaction();
```

回滚事务：

```java
Base.rollbackTransaction();
```

## 7. 支持的数据库

最新版本支持SQLServer、MySQL、Oracle、PostgreSQL、H2、SQLite3、DB2等数据库。

## 8. 总结

在本快速教程中，我们重点关注并探索了ActiveJDBC的基础知识。