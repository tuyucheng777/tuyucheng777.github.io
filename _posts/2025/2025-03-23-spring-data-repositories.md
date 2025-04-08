---
layout: post
title:  Spring Data中的CrudRepository、JpaRepository和PagingAndSortingRepository
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在这篇简短的文章中，我们将重点介绍不同类型的Spring Data Repository接口及其功能，我们将涉及：

- CrudRepository
- PagingAndSortingRepository
- JpaRepository

简而言之，[Spring Data](http://projects.spring.io/spring-data/)中的每个Repository都扩展了通用的Repository接口，但除此之外，它们各自具有不同的功能。

## 2. Spring Data Repository

让我们从[JpaRepository](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)开始-它扩展了[PagingAndSortingRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)，进而扩展了[CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)。

其中每一个都定义了其功能：

- [CrudRepository](https://docs.spring.io/spring-data/data-commons/docs/2.7.9/api/org/springframework/data/repository/CrudRepository.html)提供CRUD功能
- [PagingAndSortingRepository](https://docs.spring.io/spring-data/data-commons/docs/2.7.9/api/org/springframework/data/repository/PagingAndSortingRepository.html)提供对记录进行分页和排序的方法
- [JpaRepository](https://docs.spring.io/spring-data/jpa/docs/2.7.9/api/org/springframework/data/jpa/repository/JpaRepository.html)提供与JPA相关的方法，例如刷新持久化上下文和批量删除记录

因此，由于这种继承关系，**JpaRepository包含CrudRepository和PagingAndSortingRepository的完整API**。

当我们不需要JpaRepository和PagingAndSortingRepository提供的全部功能时，我们可以使用CrudRepository。

现在让我们看一个简单的例子来更好地理解这些API。

我们从一个简单的Product实体开始：

```java
@Entity
public class Product {

    @Id
    private long id;
    private String name;

    // getters and setters
}
```

让我们实现一个简单的操作-根据name查找Product：

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    Product findByName(String productName);
}
```

就这样，Spring Data Repository将根据我们提供的名称自动生成实现。

当然，这是一个非常简单的例子；你可以在[这里](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)深入了解Spring Data JPA。

## 3. CrudRepository

现在让我们看一下[CrudRepository](https://docs.spring.io/spring-data/data-commons/docs/2.7.9/api/org/springframework/data/repository/CrudRepository.html)接口的代码：

```java
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
    <S extends T> S save(S entity);

    T findOne(ID primaryKey);

    Iterable<T> findAll();

    Long count();

    void delete(T entity);

    boolean exists(ID primaryKey);
}
```

注意典型的CRUD功能：

- save(...)：保存实体的Iterable，在这里，我们可以传递多个对象以批量保存它们
- findOne(...)：根据传递的主键值获取单个实体
- findAll()：获取数据库中所有可用实体的Iterable
- count()：返回表中实体总数
- delete(...)：根据传递的对象删除实体
- exists(...)：根据传递的主键值验证实体是否存在

这个接口看起来非常通用和简单，但实际上，它提供了应用程序所需的所有基本查询抽象。

## 4. PagingAndSortingRepository

现在，让我们看一下另一个扩展了CrudRepository的Repository接口：

```java
public interface PagingAndSortingRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
    Iterable<T> findAll(Sort sort);

    Page<T> findAll(Pageable pageable);
}
```

该接口提供了一个方法findAll(Pageable pageable)，这个方法就是实现Pagination的关键。

当使用Pageable时，我们创建一个具有某些属性的Pageable对象，并且我们至少要指定以下内容：

1. 页面大小
2. 当前页码
3. 排序

因此，假设我们想要显示按lastName升序排序的结果集的第一页，每页不超过5条记录。我们可以使用PageRequest和Sort定义来实现这一点：

```java
Sort sort = new Sort(new Sort.Order(Direction.ASC, "lastName"));
Pageable pageable = new PageRequest(0, 5, sort);
```

将Pageable对象传递给Spring Data查询将返回有问题的结果(PageRequest的第一个参数从0开始)。

## 5. JpaRepository

最后，我们来看看[JpaRepository](https://docs.spring.io/spring-data/jpa/docs/2.7.9/api/)接口：

```java
public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> {
    List<T> findAll();

    List<T> findAll(Sort sort);

    List<T> save(Iterable<? extends T> entities);

    void flush();

    T saveAndFlush(T entity);

    void deleteInBatch(Iterable<T> entities);
}
```

再次，让我们简要地看一下这些方法：

- findAll()：获取数据库中所有可用实体的列表
- findAll(...)：获取所有可用实体的列表，并使用提供的条件对其进行排序
- save(...)：保存实体的Iterable，在这里，我们可以传递多个对象以批量保存它们
- flush()：将所有待处理的任务刷新到数据库
- saveAndFlush(...)：保存实体并立即刷新更改
- deleteInBatch(...)：删除实体的Iterable，在这里，我们可以传递多个对象以批量删除它们

显然，上述接口扩展了PagingAndSortingRepository，这意味着它还具有CrudRepository中的所有方法。

## 6. Spring Data 3中的Spring Data Repositories

在新版本的Spring Data中，一些Repository类的内部结构略有变化，增加了新的功能并提供了更简单的开发体验。

现在，我们可以使用基于List的CRUD Repository接口。此外，一些Spring Data Repository类的类层次结构基于不同的结构。

所有详细信息均可在我们的[Spring Data 3中的新CRUD Repository接口](https://www.baeldung.com/spring-data-3-crud-repository-interfaces)文章中找到。

## 7. Spring Data Repositories的缺点

除了这些Repository的所有非常有用的优点之外，直接依赖它们也有一些基本的缺点：

1. 我们将代码与库及其特定的抽象(例如“Page”或“Pageable”)耦合在一起；当然，这并不是这个库所独有的-但我们必须小心，不要暴露这些内部实现细节。
2. 通过扩展CrudRepository等，我们可以一次性公开一整套持久化方法。在大多数情况下，这可能也没什么问题，但我们可能会遇到希望对公开的方法进行更细粒度控制的情况，例如，创建一个不包含CrudRepository的save(...)和delete(...)方法的ReadOnlyRepository。

## 8. 总结

本文介绍了Spring Data JPA Repository接口的一些简短但重要的区别和功能。