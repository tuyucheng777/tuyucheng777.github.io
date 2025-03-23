---
layout: post
title:  从H2数据库中的文件运行查询
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

在本教程中，我们将探讨如何从文件运行[H2数据库](https://www.baeldung.com/spring-boot-h2-database)的脚本。H2是一个开源Java数据库，可以嵌入Java应用程序中或作为单独的服务器运行，它通常用作测试目的的内存数据库。

有时，我们可能需要在运行应用程序之前或运行期间运行脚本文件来创建表、插入数据或更新数据库中的数据。

## 2. 设置

对于我们的代码示例，我们将创建一个小型Spring Boot应用程序。我们将在应用程序中包含一个嵌入式H2数据库，然后，我们将尝试运行修改数据库的脚本文件的不同方法。

### 2.1 依赖

让我们首先将[spring-boot-starter-data-jpa](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

此依赖为具有[JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)支持的Spring Boot应用程序创建了基本设置。

为了在我们的项目中使用H2，我们需要添加[H2数据库](https://mvnrepository.com/artifact/com.h2database/h2)依赖：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 2.2 属性

接下来，我们需要在application.properties文件中配置与H2数据库的连接：

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=none
```

此配置设置了H2数据库的URL、驱动程序类、用户名和密码。它还设置了数据库平台并禁用了自动模式生成，**禁用自动模式生成很重要，因为我们想稍后运行脚本文件来创建模式**。

## 3. 在Spring Boot应用程序中运行脚本

现在我们已经设置好了项目，让我们探索在Spring Boot应用程序中运行脚本文件的不同方法。有时应用程序启动时可能需要将数据添加到数据库，例如，如果我们想使用预定义数据运行测试，我们可以使用脚本文件将数据插入数据库。

### 3.1 使用默认文件

**默认情况下，Spring Boot会在src/main/resources目录中查找名为schema.sql的文件来创建模式。它还会在同一目录中查找名为data.sql的文件，并在应用程序启动时运行其命令**。

因此，让我们创建一个schema.sql文件，该文件包含在H2数据库中创建表的SQL脚本：

```sql
CREATE TABLE IF NOT EXISTS employee (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

类似地，让我们创建一个data.sql文件，其中包含将记录插入employee表并更新一条记录的SQL脚本：

```sql
INSERT INTO employee (name) VALUES ('John');
INSERT INTO employee (name) VALUES ('Jane');
UPDATE EMPLOYEE SET NAME = 'Jane Doe' WHERE ID = 2;
```

当我们运行应用程序时，Spring Boot会自动运行这些脚本来创建表并插入/更新数据。

### 3.2 来自不同目录的文件

**我们可以通过在application.properties文件中设置spring.sql.init.schema-locations和spring.sql.init.data-locations属性来更改默认文件位置**，当我们想在不同的环境中使用不同的文件，或者想将脚本保存在不同的目录中时，这很有用。

我们将schema.sql和data.sql文件移动到不同的目录-src/main/resources/db并更新属性：

```properties
spring.sql.init.data-locations=classpath:db/data.sql
spring.sql.init.schema-locations=classpath:db/schema.sql
```

现在，当我们运行应用程序时，Spring Boot会在db目录中查找schema.sql和data.sql文件。

### 3.3 使用代码

我们还可以使用代码运行脚本文件，当我们想要有条件地运行脚本文件或想要基于某种逻辑运行脚本文件时，这很有用。对于此示例，让我们在src/main/resources目录中创建一个名为script.sql的脚本文件。

该文件包含更新和读取employee表中数据的SQL脚本：

```sql
UPDATE employee SET NAME = 'John Doe' WHERE ID = 1;
SELECT * FROM employee;
```

**H2驱动程序提供了RunScript.execute()方法来帮助我们运行脚本**，让我们使用此方法在Spring Boot应用程序中运行脚本文件。

我们将在主类中创建一个[@PostConstruct](https://www.baeldung.com/spring-postconstruct-predestroy#postConstruct)方法来运行脚本文件：

```java
@PostConstruct
public void init() throws SQLException, IOException {
    Connection connection = DriverManager.getConnection(url, user, password);
    ResultSet rs = RunScript.execute(connection, new FileReader(new ClassPathResource("db/script.sql").getFile()));
    log.info("Reading Data from the employee table");
    while (rs.next()) {
        log.info("ID: {}, Name: {}", rs.getInt("id"), rs.getString("name"));
    }
}
```

@PostConstruct注解告诉应用程序在初始化应用程序上下文后运行init()方法，在此方法中，我们创建与H2数据库的连接，并使用RunScript.execute()方法运行script.sql文件。

当我们运行应用程序时，Spring Boot会运行脚本来更新和读取employee表中的数据，我们在控制台中看到脚本的输出：

```text
Data from the employee table:
John Doe
Jane Doe
```

## 4. 通过命令行运行脚本

另一种选择是通过命令行运行脚本文件，当我们想在实时数据库上运行脚本时，这很有用。**我们可以使用H2数据库提供的RunScript工具来运行脚本文件**，此工具在H2 jar文件中可用。

要使用此工具，我们需要运行以下命令：

```shell
java -cp /path/to/h2/jar/h2-version.jar org.h2.tools.RunScript -url jdbc:h2:db/server/url -user sa -password password -script script.sql -showResults
```

这里我们在classpath(-cp)选项中提供了H2 jar文件的路径、数据库URL、用户和密码。最后，我们提供要运行的script.sql文件。如果我们想显示运行脚本的结果，则需要-showResults选项。

**需要注意的是，内存数据库不能在不同进程之间共享。如果我们在这里使用内存数据库URL，它会为RunScript工具创建一个新的数据库，而不是使用Spring Boot应用程序创建的数据库**。

## 5. 总结

在本文中，我们探讨了如何为H2数据库运行脚本文件。我们学习了如何使用默认资源文件、自定义位置的文件以及通过代码在Spring Boot应用程序中运行脚本文件。我们还学习了如何使用H2数据库提供的RunScript工具通过命令行运行脚本文件。