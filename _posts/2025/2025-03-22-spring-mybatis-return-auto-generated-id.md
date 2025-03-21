---
layout: post
title:  使用MyBatis和Spring从插入中返回自动生成的ID
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

[MyBatis](https://www.baeldung.com/mybatis)是一个开源Java持久层框架，可以作为[JDBC](https://www.baeldung.com/java-jdbc)和[Hibernate](https://www.baeldung.com/learn-jpa-hibernate)的替代品。它可以帮助我们减少代码并简化结果的检索，让我们专注于编写自定义SQL查询或存储过程。

在本教程中，我们将学习如何使用MyBatis和Spring Boot插入数据时返回自动生成的ID。

## 2. 依赖设置

在开始之前，让我们在pom.xml中添加[mybatis-spring-boot-starter](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter)依赖项：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

## 3. 示例设置

让我们首先创建一个将在整篇文章中使用的简单示例。

### 3.1 定义实体

首先，让我们创建一个代表汽车的简单实体类：

```java
public class Car {
    private Long id;

    private String model;

    // getters and setters
}
```

其次，我们定义一个创建表的SQL语句，并将其放在car-schema.sql文件中：

```sql
CREATE TABLE IF NOT EXISTS CAR
(
    ID    INTEGER PRIMARY KEY AUTO_INCREMENT,
    MODEL VARCHAR(100) NOT NULL
);
```

### 3.2 定义数据源

接下来，让我们指定数据源，我们将使用H2嵌入式数据库：

```java
@Bean
public DataSource dataSource() {
    EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
    return builder
            .setType(EmbeddedDatabaseType.H2)
            .setName("testdb")
            .addScript("car-schema.sql")
            .build();
}

@Bean
public SqlSessionFactory sqlSessionFactory() throws Exception {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource());
    return factoryBean.getObject();
}
```

现在我们已经完成所有设置，让我们看看如何使用基于注解和基于XML的方法来检索自动生成的ID。

## 4. 使用注解

让我们定义Mapper，它代表MyBatis用于将方法绑定到相应SQL语句的接口：

```java
@Mapper
public interface CarMapper {
    // ...
}
```

接下来我们添加一个插入语句：

```java
@Insert("INSERT INTO CAR(MODEL) values (#{model})")
void save(Car car);
```

本能地，我们可能倾向于返回Long并期望MyBatis返回所创建实体的ID。然而，这并不准确。如果我们这样做，它会返回1，表示插入语句成功。

**要检索生成的ID，我们可以使用[@Options](https://mybatis.org/mybatis-3/es/apidocs/org/apache/ibatis/annotations/Options.html)或[@SelectKey](https://mybatis.org/mybatis-3/apidocs/org/apache/ibatis/annotations/SelectKey.html)注解**。

### 4.1 @Options注解

我们可以扩展insert语句的一种方法是使用@Options注解：

```java
@Insert("INSERT INTO CAR(MODEL) values (#{model})")
@Options(useGeneratedKeys = true, keyColumn = "ID", keyProperty = "id")
void saveUsingOptions(Car car);
```

这里我们设置了三个属性：

- useGeneratedKeys：表示我们是否要使用生成的ID功能
- keyColumn：设置包含键的列的名称
- keyProperty：表示保存键值的字段名称

此外，我们可以用逗号分隔来指定多个关键属性。

在后台，**MyBatis使用[反射](https://www.baeldung.com/java-reflection)将ID列的值映射到Car对象的id字段中**。

接下来，让我们创建一个测试来确认一切是否按预期进行：

```java
@Test
void givenCar_whenSaveUsingOptions_thenReturnId() {
    Car car = new Car();
    car.setModel("BMW");

    carMapper.saveUsingOptions(car);

    assertNotNull(car.getId());
}
```

### 4.2 @SelectKey注解

返回ID的另一种方法是使用@SelectKey注解，**当我们想要使用序列或标识函数来检索标识符时，此注解非常有用**。

此外，如果我们使用@SelectKey注解修饰我们的方法，MyBatis会忽略诸如@Options之类的注解。

让我们在CarMapper内部创建一个新方法来检索插入后的标识值：

```java
@Insert("INSERT INTO CAR(MODEL) values (#{model})")
@SelectKey(statement = "CALL IDENTITY()", before = false, keyColumn = "ID", keyProperty = "id", resultType = Long.class)
void saveUsingSelectKey(Car car);
```

让我们检查一下我们使用的属性：

- statement：保存将在insert语句之后执行的语句
- before：表示语句应该在插入之前还是之后执行
- keyColumn：保存代表键的列的名称
- keyProperty：指定将保存语句返回的值的字段的名称
- resultType：表示keyProperty的类型

此外，我们应该注意到**IDENTITY()函数已从H2数据库中删除**。更多详细信息请参见[此处](https://github.com/h2database/h2database/issues/3333)。

为了能够在H2数据库上执行CALL IDENTITY()，我们需要将模式设置为LEGACY：

```properties
"testdb;MODE=LEGACY"
```

让我们测试一下我们的方法来确认它能正常工作：

```java
@Test
void givenCar_whenSaveUsingSelectKey_thenReturnId() {
    Car car = new Car();
    car.setModel("BMW");

    carMapper.saveUsingSelectKey(car);

    assertNotNull(car.getId());
}
```

## 5. 使用XML

让我们看看如何实现相同的功能，但这次，我们将使用基于XML的方法。

首先，让我们定义CarXmlMapper接口：

```java
@Mapper
public interface CarXmlMapper {
     // ...
}
```

与基于注解的方法不同，我们不会在Mapper接口中直接编写SQL语句。相反，我们将定义XML映射器文件并将所有查询放入其中：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="cn.tuyucheng.taketoday.mybatis.generatedid.CarXmlMapper">

</mapper>
```

此外，**在namespace属性中，我们指定了CarXmlMapper接口的完全限定名称**。

### 5.1 UseGeneratedKeys属性

接下来，让我们在CarXmlMapper接口中定义一个方法：

```java
void saveUsingOptions(Car car);
```

此外，让我们使用XML映射器来定义insert语句并将其映射到我们放置在CarXmlMapper接口内的saveUsingOptions()方法：

```xml
<insert id="saveUsingOptions" parameterType="cn.tuyucheng.taketoday.mybatis.generatedid.Car"
        useGeneratedKeys="true" keyColumn="ID" keyProperty="id">
    INSERT INTO CAR(MODEL)
    VALUES (#{model});
</insert>
```

让我们探索一下我们使用的属性：

- id：将查询绑定到CarXmlMapper类中的特定方法
- parameterType：saveUsingOptions()方法的参数类型 
- useGeneratedKeys：表示我们要使用生成的ID功能
- keyColumn：指定代表键的列的名称
- keyProperty：指定Car对象中保存密钥的字段名称

此外，让我们测试一下我们的解决方案：

```java
@Test
void givenCar_whenSaveUsingOptions_thenReturnId() {
    Car car = new Car();
    car.setModel("BMW");

    carXmlMapper.saveUsingOptions(car);

    assertNotNull(car.getId());
}
```

### 5.2 SelectKey元素

接下来，让我们在CarXmlMapper接口中添加一个新方法，看看如何使用selectKey元素检索身份：

```java
void saveUsingSelectKey(Car car);
```

此外，让我们在XML映射器文件中指定语句并将其绑定到方法：

```xml
<insert id="saveUsingSelectKey" parameterType="cn.tuyucheng.taketoday.mybatis.generatedid.Car">
    INSERT INTO CAR(MODEL)
    VALUES (#{model});

    <selectKey resultType="Long" order="AFTER" keyColumn="ID" keyProperty="id">
        CALL IDENTITY()
    </selectKey>
</insert>
```

在这里，我们使用以下属性定义selectKey元素：

- resultType：指定语句返回的类型
- order：指示语句CALL IDENTITY()应在insert语句之前还是之后调用
- keyColumn：保存代表标识符的列的名称
- keyProperty：保存应将键映射到的字段的名称

最后，让我们创建一个测试：

```java
@Test
void givenCar_whenSaveUsingSelectKey_thenReturnId() {
    Car car = new Car();
    car.setModel("BMW");

    carXmlMapper.saveUsingSelectKey(car);

    assertNotNull(car.getId());
}
```

## 6. 总结

在本文中，我们学习了如何使用MyBatis和Spring从insert语句中检索自动生成的ID。

总而言之，我们探索了如何使用基于注解的方法以及@Options和@SelectKey注解来检索ID。此外，我们还研究了如何使用基于XML的方法返回ID。