---
layout: post
title:  使用Spring Boot和JPA存储PostgreSQL JSONB
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

本教程全面介绍了如何将JSON数据存储在PostgreSQL JSONB列中。

使用JPA，我们将快速回顾如何处理存储为可变字符(VARCHAR)数据库列的JSON值。之后，我们将比较VARCHAR类型和JSONB类型之间的差异，以了解JSONB的附加功能。最后，我们将解决JPA中的JSONB类型映射问题。

## 2. VARCHAR映射

在本节中，我们将探索如何将VARCHAR类型的JSON值转换为自定义Java POJO。为此，我们将使用[AttributeConverter](https://www.baeldung.com/jpa-attribute-converters)轻松地将Java数据类型的实体属性值转换为数据库列中的相应值。

### 2.1 Maven依赖

要创建AttributeConverter，我们必须在pom.xml中包含[最新的](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)Spring Data JPA依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>3.3.2</version>
</dependency>
```

### 2.2 表定义

让我们使用以下数据库表定义通过一个简单的示例来说明这个概念：

```sql
CREATE TABLE student (
    student_id VARCHAR(8) PRIMARY KEY,
    admit_year VARCHAR(4),
    address VARCHAR(500)
);
```

学生表有三个字段，我们期望address列存储具有以下结构的JSON值：

```json
{
    "postCode": "TW9 2SF",
    "city": "London"
}
```

### 2.3 实体类

为了解决这个问题，我们将创建一个相应的[POJO](https://www.baeldung.com/java-pojo-class)类来用Java表示地址数据：

```java
public class Address {
    private String postCode;

    private String city;

    // constructor, getters and setters
}
```

接下来，我们将创建一个实体类StudentEntity，并将其映射到我们之前创建的student表：

```java
@Entity
@Table(name = "student")
public class StudentEntity {
    @Id
    @Column(name = "student_id", length = 8)
    private String id;

    @Column(name = "admit_year", length = 4)
    private String admitYear;

    @Convert(converter = AddressAttributeConverter.class)
    @Column(name = "address", length = 500)
    private Address address;

    // constructor, getters and setters
}
```

我们将使用[@Convert](https://www.baeldung.com/jpa-attribute-converters#using-the-converter)标注address字段并应用AddressAttributeConverter将Address实例转换为其JSON表示形式。

### 2.4 AttributeConverter

之前，我们将实体类中的address字段映射为数据库中的VARCHAR类型。但是，JPA无法自动转换自定义Java类型和VARCHAR类型。因此，AttributeConverter提供了一种处理转换过程的机制来弥补这一差距。

我们使用AttributeConverter将自定义Java数据类型持久化到数据库列中，**每个AttributeConverter实现都必须定义两种转换方法**。一种方法名为convertToDatabaseColumn()，它将Java数据类型转换为其对应的数据库数据类型；另一种名为convertToEntityAttribute()，它将数据库数据类型转换为Java数据类型：

```java
@Converter
public class AddressAttributeConverter implements AttributeConverter<Address, String> {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public String convertToDatabaseColumn(Address address) {
        try {
            return objectMapper.writeValueAsString(address);
        } catch (JsonProcessingException jpe) {
            log.warn("Cannot convert Address into JSON");
            return null;
        }
    }

    @Override
    public Address convertToEntityAttribute(String value) {
        try {
            return objectMapper.readValue(value, Address.class);
        } catch (JsonProcessingException e) {
            log.warn("Cannot convert JSON into Address");
            return null;
        }
    }
}
```

### 2.5 测试用例

现在，我们可以验证Student行及其Address是否正确保存：

```java
@Test
void whenSaveAnStudentEntityAndFindById_thenTheRecordPresentsInDb() {
    String studentId = "23876213";
    String postCode = "KT5 8LJ";

    Address address = new Address(postCode, "London");
    StudentEntity studentEntity = StudentEntity.builder()
            .id(studentId)
            .admitYear("2023")
            .address(address)
            .build();

    StudentEntity savedStudentEntity = studentRepository.save(studentEntity);

    Optional<StudentEntity> studentEntityOptional = studentRepository.findById(studentId);
    assertThat(studentEntityOptional.isPresent()).isTrue();

    studentEntity = studentEntityOptional.get();
    assertThat(studentEntity.getId()).isEqualTo(studentId);
    assertThat(studentEntity.getAddress().getPostCode()).isEqualTo(postCode);
}
```

此外，我们可以检查日志来查看插入新数据时JPA正在执行什么操作：

```text
Hibernate: 
    insert 
    into
        "public"
        ."student_str" ("address", "admit_year", "student_id") 
    values
        (?, ?, ?)
binding parameter [1] as [VARCHAR] - [{"postCode":"KT6 7BB","city":"London"}]
binding parameter [2] as [VARCHAR] - [2023]
binding parameter [3] as [VARCHAR] - [23876371]
```

我们可以看到第一个参数已经通过AddressAttributeConverter从我们的Address实例成功转换并绑定为VARCHAR类型。

我们学习了如何通过将JSON数据转换为VARCHAR以及将VARCHAR转换为JSON数据来保存JSON数据。接下来，让我们看看处理JSON数据的其他解决方案。

## 3. JSONB优于VARCHAR

在PostgreSQL中，我们可以将列的类型设置为JSONB来保存JSON数据：

```sql
CREATE TABLE student (
    student_id VARCHAR(8) PRIMARY KEY,
    admit_year VARCHAR(4),
    address jsonb
);
```

在这里，我们将address列定义为JSONB。这是一种与以前使用的VARCHAR类型不同的数据类型，了解为什么我们在PostgreSQL中有这种数据类型很重要。**JSONB是PostgreSQL中用于处理JSON数据的指定数据类型**。

此外，**使用JSONB类型的列以分解的二进制格式存储数据，由于额外的转换，在存储JSON时会产生一些开销**。

此外，**与VARCHAR相比，JSONB提供了额外的功能**。因此，JSONB是PostgreSQL中存储JSON数据的更有利选择。

### 3.1 验证

**JSONB类型强制执行数据验证以确保列值是有效的JSON**。因此，尝试插入或更新具有无效JSON值的JSONB类型的列会失败。

为了证明这一点，我们可以尝试插入一个SQL查询，其中包含address列的无效JSON值，例如，city属性末尾缺少双引号：

```sql
INSERT INTO student(student_id, admit_year, address)
VALUES ('23134572', '2022', '{"postCode": "E4 8ST, "city":"London}');
```

在PostgreSQL结果中运行此查询会引发验证错误，指出我们有一个无效的JSON：

```text
SQL Error: ERROR: invalid input syntax for type json
  Detail: Token "city" is invalid.
  Position: 83
  Where: JSON data, line 1: {"postCode": "E4 8ST, "city...
```

### 3.2 查询

**PostgreSQL支持在SQL查询中使用JSON列进行查询，JPA支持使用原生查询在数据库中搜索记录**。在Spring Data中，我们可以定义一个自定义查询方法来查找Student列表：

```java
@Repository
public interface StudentRepository extends CrudRepository<StudentEntity, String> {
    @Query(value = "SELECT * FROM student WHERE address->>'postCode' = :postCode", nativeQuery = true)
    List<StudentEntity> findByAddressPostCode(@Param("postCode") String postCode);
}
```

此查询是原生SQL查询，它选择数据库中address JSON属性postCode等于提供的参数的所有Student实例。

### 3.3 索引

**JSONB支持JSON数据[索引](https://www.baeldung.com/cs/databases-indexing)，当我们必须通过JSON列中的键或属性查询数据时，这为JSONB提供了显著的优势**。

各种类型的索引都可以应用于JSON列，包括GIN、[HASH](https://www.baeldung.com/cs/hashing#hash-table)和[BTREE](https://www.baeldung.com/cs/b-tree-data-structure)。GIN适用于索引复杂的数据结构，包括数组和JSON。当我们只需要考虑相等运算符=时，HASH很重要。当我们处理范围运算符(例如<和>=)时，BTREE允许高效查询。

例如，如果我们总是需要根据address列中的postCode属性检索数据，我们可以创建以下索引：

```sql
CREATE INDEX idx_postcode ON student USING HASH((address->'postCode'));
```

## 4. JSONB映射

**当数据库列定义为JSONB时，我们不能应用相同的AttributeConverter**。否则，如果我们尝试这样做，应用程序在启动时会抛出以下错误：

```text
org.postgresql.util.PSQLException: ERROR: column "address" is of type jsonb but the expression is of type character varying
```

即使我们改变AttributeConverter类定义以使用Object而不是String作为转换的列值，我们仍然会收到错误：

```java
@Converter 
public class AddressAttributeConverter implements AttributeConverter<Address, Object> {
    // 2 conversion methods implementation
}
```

我们的应用程序抱怨不支持的类型：

```text
org.postgresql.util.PSQLException: Unsupported Types value: 1,943,105,171
```

因此，我们可以自信地说，JPA本身不支持JSONB类型。但是，我们的底层JPA实现Hibernate确实支持JSON[自定义类型](https://www.baeldung.com/hibernate-custom-types)，允许我们将复杂类型映射到Java类。

### 4.1 Maven依赖

简而言之，我们需要一个自定义类型来进行JSONB转换。幸运的是，我们可以依靠一个名为[Hypersistence Utilities](https://mvnrepository.com/artifact/io.hypersistence/hypersistence-utils-hibernate-55)的现有库。

Hypersistence Utilities是[Hibernate](https://www.baeldung.com/spring-boot-hibernate)的通用实用程序库，它的功能之一是为不同的数据库(例如PostgreSQL和Oracle)定义JSON列类型映射。因此，我们可以在pom.xml中包含这个额外依赖：

```xml
<dependency>
    <groupId>io.hypersistence</groupId>
    <artifactId>hypersistence-utils-hibernate-60</artifactId>
    <version>3.9.0</version>
</dependency>
```

### 4.2 更新的实体类

Hypersistence Utilities定义了依赖于数据库的不同自定义类型。**在PostgreSQL中，我们将使用JsonBinaryType类作为JSONB列类型**。在我们的实体类中，我们使用Hibernate的@TypeDef注解定义自定义类型，然后通过@Type将定义的类型应用于address字段：

```java
@Entity
@Table(name = "student")
public class StudentEntity {
    @Id
    @Column(name = "student_id", length = 8)
    private String id;

    @Column(name = "admit_year", length = 4)
    private String admitYear;

    @Type(JsonBinaryType.class)
    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name = "address", columnDefinition = "jsonb")
    private Address address;

    // getters and setters
}
```

**对于使用@Type的这种情况，我们不再需要将AttributeConverter应用于address字段**。Hypersistence Utilities中的自定义类型会为我们处理转换任务，使我们的代码更加简洁。

注意：请记住，@Type注解在Hibernate 6中已被弃用。

### 4.3 测试用例

完成所有这些更改后，让我们再次重新运行Student持久层测试用例：

```text
Hibernate: 
    insert 
    into
        "public"
        ."student" ("address", "admit_year", "student_id") 
    values
        (?, ?, ?)
binding parameter [1] as [OTHER] - [Address(postCode=KT6 7BB, city=London)]
binding parameter [2] as [VARCHAR] - [2023]
binding parameter [3] as [VARCHAR] - [23876371]
```

我们会看到JPA触发与之前相同的插入SQL，只是第一个参数绑定为OTHER而不是VARCHAR，这表明Hibernate这次将参数绑定为JSONB类型。

## 5. 总结

在本教程中，我们学习了如何使用Spring Boot和JPA在PostgreSQL中管理JSON数据。首先，我们使用自定义转换器解决了JSON值到VARCHAR类型和JSONB类型的映射问题。然后，我们了解了使用JSONB强制执行JSON验证和查询以及轻松索引JSON值的重要性。最后，我们使用Hypersistence库实现了JSONB列的自定义类型转换。