---
layout: post
title: Apache Ignite与Spring Data
category: libraries
copyright: libraries
excerpt: Apache Ignite
---

## 1. 概述

在本快速指南中，我们将重点介绍如何**将Spring Data API与Apache Ignite平台集成**。

要了解Apache Ignite，请查看我们[之前](https://www.baeldung.com/apache-ignite)的指南。

## 2. Maven设置

除了现有的依赖之外，我们还必须启用Spring Data支持：

```xml
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-spring-data</artifactId>
    <version>${ignite.version}</version>
</dependency>
```

ignite-spring-data工件可从[Maven Central](https://mvnrepository.com/artifact/org.apache.ignite/ignite-spring-data)下载。

## 3. 模型和Repository

为了演示集成，我们将构建一个应用程序，使用Spring Data API将员工存储到Ignite的缓存中。

EmployeeDTO的POJO将如下所示：

```java
public class EmployeeDTO implements Serializable {
 
    @QuerySqlField(index = true)
    private Integer id;
    
    @QuerySqlField(index = true)
    private String name;
    
    @QuerySqlField(index = true)
    private boolean isEmployed;

    // getters, setters
}
```

这里，@QuerySqlField注解允许使用SQL查询字段。

接下来，我们将创建Repository来保存Employee对象：

```java
@RepositoryConfig(cacheName = "tuyuchengCache")
public interface EmployeeRepository extends IgniteRepository<EmployeeDTO, Integer> {
    EmployeeDTO getEmployeeDTOById(Integer id);
}
```

**Apache Ignite使用自己的IgniteRepository，它从Spring Data的CrudRepository扩展而来**，并且还允许从Spring Data访问SQL网格。 

它支持标准CRUD方法，但有些方法不需要id，我们将在测试部分详细了解原因。

**@RepositoryConfig注解将EmployeeRepository映射到Ignite的tuyuchengCache**。

## 4. Spring配置

现在让我们创建我们的Spring配置类。

**我们将使用@EnableIgniteRepositories注解来添加对Ignite Repository的支持**：

```java
@Configuration
@EnableIgniteRepositories
public class SpringDataConfig {

    @Bean
    public Ignite igniteInstance() {
        IgniteConfiguration config = new IgniteConfiguration();

        CacheConfiguration cache = new CacheConfiguration("tuyuchengCache");
        cache.setIndexedTypes(Integer.class, EmployeeDTO.class);

        config.setCacheConfiguration(cache);
        return Ignition.start(config);
    }
}
```

这里，igniteInstance()方法创建并将Ignite实例传递给IgniteRepositoryFactoryBean以访问Apache Ignite集群。

我们还定义并设置了tuyuchengCache配置，setIndexedTypes()方法设置缓存的SQL模式。

## 5. 测试Repository

为了测试应用程序，让我们在应用程序上下文中注册SpringDataConfiguration并从中获取EmployeeRepository：

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.register(SpringDataConfig.class);
context.refresh();

EmployeeRepository repository = context.getBean(EmployeeRepository.class);
```

然后，我们要创建EmployeeDTO实例并将其保存在缓存中：

```java
EmployeeDTO employeeDTO = new EmployeeDTO();
employeeDTO.setId(1);
employeeDTO.setName("John");
employeeDTO.setEmployed(true);

repository.save(employeeDTO.getId(), employeeDTO);
```

这里我们使用了IgniteRepository的save(key, value)方法，这样做的原因是**标准CrudRepository save(entity)、save(entities)、delete(entity)操作尚不受支持**。

这背后的问题是CrudRepository.save()方法生成的ID在集群中不是唯一的。

相反，我们必须使用save(key, value)、save(Map<ID, Entity> values)、deleteAll(Iterable<ID> ids)方法。

之后，我们可以使用Spring Data的getEmployeeDTOById()方法从缓存中获取员工对象：

```java
EmployeeDTO employee = repository.getEmployeeDTOById(employeeDTO.getId());
System.out.println(employee);
```

输出显示我们成功获取了初始对象：

```text
EmployeeDTO{id=1, name='John', isEmployed=true}
```

或者，我们可以使用IgniteCache API检索相同的对象：

```java
IgniteCache<Integer, EmployeeDTO> cache = ignite.cache("tuyuchengCache");
EmployeeDTO employeeDTO = cache.get(employeeId);
```

或者使用标准SQL：

```java
SqlFieldsQuery sql = new SqlFieldsQuery("select * from EmployeeDTO where isEmployed = 'true'");
```

## 6. 总结

本简短教程展示了如何将Spring Data框架与Apache Ignite项目集成，借助实际示例，我们学习了如何使用Spring Data API来处理Apache Ignite缓存。