---
layout: post
title:  解决Spring JPA属性命名问题
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 简介

[Spring](https://www.baeldung.com/spring-tutorial)为程序员提供的简化Java应用程序中数据库交互的最强大框架之一是[Spring JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)(Java Persistence API)，它提供了对JPA的可靠抽象。

**然而，尽管使用起来很方便，开发人员还是经常会遇到难以诊断和解决的错误。其中一个常见问题是“Unable to Locate Attribute with the Given Name”错误**。

**在本教程中，让我们先检查一下这个问题的根源，然后再研究如何解决这个问题**。

## 2. 定义用例

有一个实际的用例来阐述这篇文章总是有帮助的。

我们打造独特、引人注目的可穿戴设备，在最近的一项调查之后，我们的营销团队发现，在我们的平台上按传感器类型、价格和受欢迎程度对产品进行分类，可以突出显示最受欢迎的商品，从而帮助客户做出更好的购买决定。

![](/assets/images/2025/springdata/springjpatroubleshootingattributenamingissues01.png)

## 3. 添加Maven依赖

让我们使用内存H2数据库在我们的项目中创建一个可穿戴设备表，我们将在其中填充可在后续测试中使用的示例数据。

首先，让我们添加以下[Maven依赖](https://mvnrepository.com/search?q=h2+AND+spring-boot-starter-data-jpa)：

```xml
<dependency> 
    <groupId>com.h2database</groupId> 
    <artifactId>h2</artifactId> 
    <version>2.2.224</version> 
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.7.11</version>
</dependency>
```

## 4. 添加应用资源

**为了我们可以测试这个Repository，让我们创建一些应用程序属性条目，帮助我们在H2内存数据库中创建和填充一个名为WEARABLES的表**。

在我们的应用程序的main/resources文件夹中，让我们创建一个具有以下条目的application-h2.properties：

```properties
# H2 configuration
hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.hbm2ddl.auto=create-drop

# Spring Datasource URL
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
```

我们将把下面的SQL放在名为testdata.sql的main/resources文件夹中，这将帮助我们在H2数据库中创建带有一堆预定义条目的可穿戴设备表：

```sql
CREATE TABLE IF NOT EXISTS wearables (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    price DECIMAL(10, 2),
    sensor_type VARCHAR(255),
    popularity_index INT
);

DELETE FROM wearables;

INSERT INTO wearables (id, name, price, sensor_type, popularity_index)
VALUES (1, 'SensaWatch', '500.00', 'Accelerometer', 5);

INSERT INTO wearables (id, name, price, sensor_type, popularity_index)
VALUES (2, 'SensaBelt', '300.00', 'Heart Rate', 3);

INSERT INTO wearables (id, name, price, sensor_type, popularity_index)
VALUES (3, 'SensaTag', '120.00', 'Proximity', 2);

INSERT INTO wearables (id, name, price, sensor_type, popularity_index)
VALUES (4, 'SensaShirt', '150.00', 'Human Activity Recognition', 2);
```

## 5. 定义WearableEntity模型

让我们定义[实体](https://www.baeldung.com/jpa-entities)模型WearableEntity，该模型是定义可穿戴设备特征的实体：

```java
@Entity 
public class WearableEntity { 

    @Id @GeneratedValue 
    private Long Id; 
    
    @Column(name = "name") 
    private String Name; 

    @Column(name = "price") 
    private BigDecimal Price; 
    // e.g., "Heart Rate Monitor", "Neuro Feedback", etc. 

    @Column(name = "sensor_type") 
    private String SensorType; 
    
    @Column(name = "popularity_index") 
    private Integer PopularityIndex;
}
```

## 6. 定义实体过滤查询

在平台中引入上述实体后，让我们向数据库添加一个查询，使客户能够使用Spring JPA框架根据持久层中的新过滤条件过滤WearableEntity。

```java
public interface WearableRepository extends JpaRepository<WearableEntity, Long> {
    List<WearableEntity> findAllByOrderByPriceAscSensorTypeAscPopularityIndexDesc();
}
```

让我们分解上述查询以便更好地理解它。

1. findAllBy：我们使用此方法来检索所有属于或类型为WearableEntity的记录
2. OrderByPriceAsc：按价格升序对结果进行排序
3. SensorTypeAsc：按价格排序后，按传感器类型升序排序
4. PopularityIndexDesc：最后，让我们按popularityIndex降序对结果进行排序(因为流行度越高越好)

## 7. 通过集成测试测试Repository

现在让我们通过在项目中引入集成测试来检查WearableRepository的行为：

```java
public class WearableRepositoryIntegrationTest {
    @Autowired
    private WearableRepository wearableRepository;

    @Test
    public void testFindByCriteria()  {
        assertThat(wearableRepository.findAllByOrderByPriceAscSensorTypeAscPopularityIndexDesc()) .hasSize(4);
    }
}
```

## 8. 运行集成测试

但是在运行集成测试时，我们会立即注意到它无法加载应用程序上下文并失败并出现以下错误：

```text
Caused by: java.lang.IllegalArgumentException: Unable to locate Attribute  with the the given name [price] on this ManagedType [cn.tuyucheng.taketoday.spring.data.jpa.filtering.WearableEntity]
```

## 9. 理解根本原因

**Hibernate使用命名约定将字段映射到数据库列，假设实体类中的字段名称与相应的列名称或预期约定不一致。在这种情况下，Hibernate将无法映射它们，从而导致查询执行或模式验证期间出现异常**。

在此示例中：

- Hibernate期望字段名称为name、price或popularityIndex(采用camelCase)，但实体错误地使用了字段名称Id、Name、SensorType、Price和PopularityIndex(采用 PascalCase)
- 当执行findAllByOrderByPriceAsc()之类的查询时，Hibernate会尝试将SQL价格列映射到实体字段，由于该字段名为Price(大写“P”)，因此无法找到该属性，从而导致IllegalArgumentException

## 10. 通过修复实体解决错误

**现在让我们将WearableEntity类中字段的命名从PascalCase更改为camelCase**：

```java
@Entity
@Table(name = "wearables")
public class WearableValidEntity {
    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name")
    // use camelCase instead of PascalCase
    private String name;

    @Column(name = "price")
    // use camelCase instead of PascalCase
    private BigDecimal price;

    @Column(name = "sensor_type")
    // use camelCase instead of PascalCase
    private String sensorType;

    @Column(name = "popularity_index")
    // use camelCase instead of PascalCase
    private Integer popularityIndex;
}
```

完成此更改后，让我们重新运行WearableRepositoryIntegrationTest，因此它立即通过了。

## 11. 总结

在本文中，我们强调了遵循JPA命名约定的重要性，以防止运行时错误并确保顺畅的数据交互。遵守最佳实践有助于避免字段映射问题并优化应用程序性能。