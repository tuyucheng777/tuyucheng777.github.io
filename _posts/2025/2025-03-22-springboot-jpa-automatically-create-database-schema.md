---
layout: post
title:  使用Spring Boot自动创建数据库模式
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 简介

[Spring Boot](https://www.baeldung.com/spring-boot-start)与[JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)无缝协作，让我们能够轻松地在应用程序中实现数据访问层。它的一个强大功能是能够根据Java实体类自动创建和管理数据库模式，通过一些配置，我们可以指示Spring Boot读取我们的实体类并自动创建或更新模式。

在本文中，我们将简要讨论如何利用Spring Boot和JPA的强大功能自动创建数据库模式，而无需编写SQL语句。我们还将了解配置过程中应避免的一些常见陷阱。

## 2. 配置自动模式创建

在本节中，我们将看到配置Spring Boot以自动创建数据库的步骤。

### 2.1 添加所需的依赖

在配置我们的项目以创建模式之前，我们应该确保我们的Spring Boot项目包含正确的依赖项。

首先，我们需要[Spring Data JPA](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)来进行数据库交互。它在JPA上提供了一个抽象层：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

然后，我们需要将特定于数据库的依赖添加到我们的项目中。

对于[H2数据库](https://mvnrepository.com/artifact/com.h2database/h2)，我们需要添加以下依赖：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

对于[MySQL](https://mvnrepository.com/artifact/mysql/mysql-connector-java)来说，依赖如下：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

对于[PostgreSQL](https://mvnrepository.com/artifact/org.postgresql/postgresql)，我们应该添加以下依赖：

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>

```

### 2.2 配置application.properties

Spring Boot使用[application.properties](https://www.baeldung.com/spring-boot-yaml-vs-properties)文件来定义配置设置，我们在这里包含数据库连接详细信息和其他配置。

让我们看看H2数据库的配置：

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.h2.console.enabled=true

# JPA Configuration
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
```

对于我们的MySQL，配置如下：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydatabase
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
```

对于我们的PostgreSQL，我们来配置：

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydatabase
spring.datasource.username=postgres
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
```

### 2.3 了解spring.jpa.hibernate.ddl-auto选项

自动创建模式的关键配置选项是spring.jpa.hibernate.ddl-auto，此设置控制应用程序启动时如何管理数据库模式。接下来，让我们了解可用的选项。

none选项表示不生成或验证任何模式，当数据库模式已经完美设置，并且我们不希望Hibernate对其进行任何更改时，这是理想的选择。

validate选项检查现有数据库模式是否与实体类匹配，不会进行任何更改，但如果存在差异(例如缺少列)，则会引发错误。这对于确保对齐而无需修改非常有用。

update选项会修改数据库模式以匹配实体类，它会添加新列或新表，但不会删除或更改现有结构。当我们需要在不冒数据丢失风险的情况下改进模式时，这是一个安全的选择。

create选项会删除整个数据库模式，并在每次应用程序启动时重新创建。如果我们愿意在每次运行时丢失并重建模式，请使用此选项。

我们使用create-drop选项在应用程序启动时创建模式，并在应用程序停止时删除模式，这对于临时环境或不需要在运行之间持久化的测试非常方便。

最后，drop选项只会删除模式而不会重新创建它。当我们有意清理整个数据库时，这很有用。

在生产环境中，开发人员经常使用update选项来确保模式不断发展以匹配应用程序，而不会冒删除现有数据的风险。

### 2.4 创建实体类

配置完成后，我们可以创建[实体类](https://www.baeldung.com/jpa-entities)，这些类代表数据库中的表并直接映射到它们。

我们来看一个例子：

```java
@Entity
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String firstName;
    private String lastName;

    //Setters and getters
}
```

@Entity注解将Person类标记为JPA实体，这意味着它将被映射到相应的数据库表，除非另有说明，该表通常名为“Person”。

@Id注解指定id为主键列，而@GeneratedValue(strategy = GenerationType.IDENTITY)表示数据库将为新行自动生成主键(例如，自动递增的值)。

此外，id、firstName和lastName代表数据库表中的列。

## 3. 测试我们的更改

让我们测试使用Spring Data JPA保存和检索Person实体的功能：

```java
void whenSavingPerson_thenPersonIsPersistedCorrectly() {
    long countBefore = personRepository.count();
    assertEquals(0, countBefore, "No persons should be present before the test");

    Person person = new Person();
    person.setFirstName("John");
    person.setLastName("Doe");

    Person savedPerson = personRepository.save(person);

    assertNotNull(savedPerson, "Saved person should not be null");
    assertTrue(savedPerson.getId() > 0, "ID should be greater than 0"); // Check if ID was generated
    assertEquals("John", savedPerson.getFirstName(), "First name should be 'John'");
    assertEquals("Doe", savedPerson.getLastName(), "Last name should be 'Doe'");
}
```

此测试确保Person实体正确地保存在数据库中。它首先检查测试前是否存在Person记录，然后创建一个具有特定值的新Person对象。接下来，在使用Repository保存实体后，测试随后验证保存的实体不为空。它还测试它是否具有大于0的生成ID，并保留正确的firstName和lastName。

## 4. 常见问题故障排除

在Spring Boot中配置自动模式创建通常很简单，但有时会出现问题，并且模式可能无法按预期生成。

**一个常见的问题是，由于命名约定或数据库方言问题，系统可能无法创建表**。为了避免这种情况，我们需要确保在application.properties文件中正确设置了spring.jpa.database-platform属性。

**另一个有用的技巧是通过设置spring.jpa.show-sql=true属性来启用Hibernate日志记录**，这将打印由Hibernate生成的SQL语句，这有助于调试与模式创建有关的问题。

确保我们的实体类与使用@EnableAutoConfiguration标注的类(通常是主应用程序类)位于同一个包或子包中也很重要，如果Spring Boot无法找到我们的实体，它将不会在数据库中创建相应的表。

此外，请验证application.properties文件是否位于正确的目录中，通常在src/main/resources下，这确保Spring Boot可以正确加载我们的配置。

我们还需要谨慎配置数据库连接，如果连接设置不正确，Spring Boot可能会默认使用内存数据库，如H2或HSQLDB。检查日志中是否存在意外的数据库连接可以帮助防止出现此问题。

最后，当我们在application.properties(或application.yml)文件中使用默认的spring.datasource.*属性时，Spring Boot自动配置将自动检测这些属性并配置数据源，而无需任何额外的配置类或Bean。

但是，当我们使用自定义前缀(如h2.datasource.\*)时，Spring Boot的默认自动配置不会自动选择这些属性，因为它专门寻找spring.datasource.\*前缀。

在这种情况下，我们需要手动配置数据源Bean并告诉Spring Boot使用@ConfigurationProperties绑定自定义属性：

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "h2.datasource")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

## 5. 总结

在本教程中，我们了解了如何使用Spring Boot自动创建和更新数据库模式。这是一个强大的功能，可以简化数据库管理。只需最少的配置，我们就可以使数据库模式与Java实体保持同步，从而确保我们的应用程序保持一致且不存在与模式相关的问题。