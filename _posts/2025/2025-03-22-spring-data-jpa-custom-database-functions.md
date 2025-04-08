---
layout: post
title:  使用JPA和Spring Boot调用自定义数据库函数
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

数据库函数是数据库管理系统中必不可少的组件，用于封装数据库中的逻辑和执行，它们有助于实现高效的数据处理和操作。

在本教程中，我们将探讨在JPA和Spring Boot应用程序中调用自定义数据库函数的各种方法。

## 2. 项目设置

我们将在后续章节中使用H2数据库演示这些概念。

让我们在pom.xml中包含[Spring Boot Data JPA](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)和[H2](https://mvnrepository.com/artifact/com.h2database/h2)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>3.2.2</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.224</version>
</dependency>
```

## 3. 数据库函数

数据库函数是通过在数据库内执行一组SQL语句或操作来执行特定任务的数据库对象，当逻辑是数据密集型时，这可以提高性能。尽管数据库函数和[存储过程](https://www.baeldung.com/spring-data-jpa-stored-procedures)的操作类似，但它们表现出差异。

### 3.1 函数与存储过程

虽然不同的数据库系统之间可能存在特定的差异，但它们之间的主要差异可以总结在下表中：

| 特征  |        数据库函数        |          存储过程          |
|:---:| :----------------------: | :------------------------: |
| 调用  |     可以在查询中调用     |        必须明确调用        |
| 返回值 |      始终返回单个值      | 可能返回无值、单个或多个值 |
| 参数  |      仅支持输入参数      |     支持输入和输出参数     |
| 调用  | 无法使用函数调用存储过程 |  可以使用存储过程调用函数  |
| 用法  |  通常执行计算或数据转换  |   通常用于复杂的业务逻辑   |

### 3.2 H2函数

为了说明如何从JPA调用数据库函数，我们将在[H2](https://www.baeldung.com/java-h2-automatically-create-schemas#what-is-h2)中创建一个数据库函数来说明如何从JPA调用它。H2数据库函数只是将被编译和执行的嵌入式Java源代码：

```java
CREATE ALIAS SHA256_HEX AS '
    import java.sql.*;
    @CODE
    String getSha256Hex(Connection conn, String value) throws SQLException {
        var sql = "SELECT RAWTOHEX(HASH(''SHA-256'', ?))";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, value);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                return rs.getString(1);
            }
        }
        return null;
    }
';
```

此数据库函数SHA256_HEX接收单个输入参数作为字符串，通过[SHA-256](http://baeldung.com/sha-256-hashing-java)哈希算法对其进行处理，然后返回其SHA-256哈希的十六进制表示形式。

## 4. 作为存储过程调用

第一种方法是在JPA中调用类似于存储过程的数据库函数，**我们通过用[@NamedStoredProcedureQuery](https://www.baeldung.com/spring-data-jpa-stored-procedures#entity-name)标注实体类来实现这一点，此注解允许我们直接在实体类中指定存储过程的元数据**。

以下是具有定义存储过程SHA256_HEX的Product实体类的示例：

```java
@Entity
@Table(name = "product")
@NamedStoredProcedureQuery(
        name = "Product.sha256Hex",
        procedureName = "SHA256_HEX",
        parameters = @StoredProcedureParameter(mode = ParameterMode.IN, name = "value", type = String.class)
)
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "product_id")
    private Integer id;

    private String name;

    // constructor, getters and setters
}
```

在实体类中，我们用@NamedStoredProcedureQuery标注我们的Product实体类，我们将Product.sha256Hex指定为命名存储过程的名称。

在我们的Repository定义中，我们使用@Procedure标注Repository方法并引用我们的@NamedStoredProcedureQuery的名称。此Repository方法接收一个字符串参数，然后将其提供给数据库函数，并返回数据库函数调用的结果。

```java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    @Procedure(name = "Product.sha256Hex")
    String getSha256HexByNamed(@Param("value") String value);
}
```

我们将看到Hibernate在执行时像从Hibernate日志中调用存储过程一样调用它：

```text
Hibernate: 
    {call SHA256_HEX(?)}
```

**@NamedStoredProcedureQuery主要用于调用存储过程**，数据库函数可以单独调用，与存储过程类似。但是，**对于与选择查询结合使用的数据库函数，它可能并不理想**。

## 5. 原生查询

调用数据库函数的另一种方法是通过原生查询。使用原生查询调用数据库函数有两种不同的方法。

### 5.1 原生调用

从前面的Hibernate日志中，我们可以看到Hibernate执行了一个CALL命令。同样，我们可以使用相同的命令本地调用我们的数据库函数：

```java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    @Query(value = "CALL SHA256_HEX(:value)", nativeQuery = true)
    String getSha256HexByNativeCall(@Param("value") String value);
}
```

执行结果将与我们在使用@NamedStoredProcedureQuery的示例中看到的相同。

### 5.2 原生Select

如前所述，我们不能将它与select查询结合使用，我们将它切换为select查询并将函数应用于表中的列值。在我们的示例中，我们定义了一个Repository方法，该方法使用原生select查询在Product表的name列上调用我们的数据库函数：

```java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    @Query(value = "SELECT SHA256_HEX(name) FROM product", nativeQuery = true)
    String getProductNameListInSha256HexByNativeSelect();
}
```

执行后，我们可以从Hibernate日志中获取与我们定义的相同的查询，因为我们将其定义为原生查询：

```text
Hibernate: 
    SELECT
        SHA256_HEX(name) 
    FROM
        product
```

## 6. 函数注册

函数注册是Hibernate定义和注册可在JPA或Hibernate查询中使用的自定义数据库函数的过程，这有助于Hibernate将自定义函数转换为相应的SQL语句。

### 6.1 自定义方言

**我们可以通过创建自定义方言来注册自定义函数**，以下是扩展默认H2Dialect并注册我们函数的自定义方言类：

```java
public class CustomH2Dialect extends H2Dialect {
    @Override
    public void initializeFunctionRegistry(FunctionContributions functionContributions) {
        super.initializeFunctionRegistry(functionContributions);
        SqmFunctionRegistry registry = functionContributions.getFunctionRegistry();
        TypeConfiguration types = functionContributions.getTypeConfiguration();

        new PatternFunctionDescriptorBuilder(registry, "sha256hex", FunctionKind.NORMAL, "SHA256_HEX(?1)")
                .setExactArgumentCount(1)
                .setInvariantType(types.getBasicTypeForJavaType(String.class))
                .register();
    }
}
```

当Hibernate初始化方言时，它会通过initialiseFunctionRegistry()将可用的数据库函数注册到函数注册表。**我们重写initialiseFunctionRegistry()方法来注册默认方言不包含的其他数据库函数**。

PatternFunctionDescriptorBuilder创建一个JPQL函数映射，将我们的数据库函数SHA256_HEX映射到JPQL函数sha256Hex，并将该映射注册到函数注册表。参数?1表示数据库函数的第一个输入参数。

### 6.2 Hibernate配置

我们必须指示Spring Boot采用CustomH2Dialect而不是默认的H2Dialect。这里是HibernatePropertiesCustomizer，它是Spring Boot提供的一个接口，用于自定义Hibernate使用的属性。

我们重写customize()方法来添加一个附加属性来表明我们将使用CustomH2Dialect：

```java
@Configuration
public class CustomHibernateConfig implements HibernatePropertiesCustomizer {
    @Override
    public void customize(Map<String, Object> hibernateProperties) {
        hibernateProperties.put("hibernate.dialect", "com.baeldung.customfunc.CustomH2Dialect");
    }
}
```

Spring Boot在应用程序启动期间自动检测并应用定制。

### 6.3 Repository方法

在我们的Repository中，我们现在可以使用应用新定义的函数sha256Hex的[JPQL](https://www.baeldung.com/jpql-hql-criteria-query#jpql)查询来代替原生查询：

```java
public interface ProductRepository extends JpaRepository<Product, Integer> {
    @Query(value = "SELECT sha256Hex(p.name) FROM Product p")
    List<String> getProductNameListInSha256Hex();
}
```

当我们在执行检查Hibernate日志时，我们看到Hibernate正确地将JPQL sha256Hex函数转换为我们的数据库函数SHA256_HEX：

```text
Hibernate: 
    select
        SHA256_HEX(p1_0.name) 
    from
        product p1_07'
```

## 7. 总结

在本文中，我们对数据库函数和存储过程进行了简要的比较，两者都提供了封装逻辑并在数据库中执行的强大方法。

此外，我们还探索了调用数据库函数的不同方法，包括利用@NamedStoredProcedureQuery注解、原生查询和通过自定义方言注册函数。通过将数据库函数合并到Repository方法中，我们可以轻松构建数据库驱动的应用程序。