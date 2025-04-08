---
layout: post
title:  在Spring Data JPA中实现仅可持久实体
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

[Spring JPA](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)简化了与数据库的交互并使通信透明。然而，默认的Spring实现有时需要根据应用程序需求进行调整。

在本教程中，我们将学习如何实现默认情况下不允许更新的解决方案，我们将考虑几种方法并讨论每种方法的优缺点。

## 2. 默认行为

[JpaRepository<T, ID\>](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)中的save(T)方法默认表现为[upsert](https://www.baeldung.com/spring-data-crud-repository-save#newInstance)，这意味着如果我们已经在数据库中拥有一个实体，它将[更新](https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate)它：

```java
@Transactional
@Override
public <S extends T> S save(S entity) {
    Assert.notNull(entity, "Entity must not be null.");

    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

根据ID，如果这是第一次插入，它将[持久化](https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate#3-merge)该实体。否则，它将调用[merge(S)](https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate#3-merge)方法来更新它。

## 3. 服务检查

此问题最明显的解决方案是显式检查实体是否包含ID并选择适当的行为，**这是更具侵入性的解决方案，但与此同时，这种行为通常由领域逻辑决定**。

因此，尽管这种方法需要我们编写一条if语句和几行代码，但它是干净且明确的。此外，我们可以更自由地决定在每种情况下做什么，并且不受JPA或数据库实现的限制：

```java
@Service
public class SimpleBookService {
    private SimpleBookRepository repository;

    @Autowired
    public SimpleBookService(SimpleBookRepository repository) {
        this.repository = repository;
    }

    public SimpleBook save(SimpleBook book) {
        if (book.getId() == null) {
            return repository.save(book);
        }
        return book;
    }

    public Optional<SimpleBook> findById(Long id) {
        return repository.findById(id);
    }
}
```

## 4. Repository检查

此方法与前一种方法类似，但将检查直接移至Repository中。然而，如果我们不想从头开始提供save(T)方法的实现，我们需要实现另一个：

```java
public interface RepositoryCheckBookRepository extends JpaRepository<RepositoryCheckBook, Long> {
    default <S extends RepositoryCheckBook> S persist(S entity) {
        if (entity.getId() == null) {
            return save(entity);
        }
        return entity;
    }
}
```

**请注意，此解决方案仅在数据库生成ID时才有效**。因此，我们可以假设具有ID的实体已经持久存在，这在大多数情况下是合理的假设。这种方法的好处是我们可以更好地控制由此产生的行为，我们在这里默默地忽略更新，但如果我们想通知客户端，我们可以更改实现。

## 5. 使用EntityManager

此方法还需要自定义实现，但我们将直接使用[EntityManger](https://www.baeldung.com/hibernate-entitymanager)，它还可能为我们提供更多功能。但是，我们必须首先创建一个自定义实现，因为我们无法将Bean注入到接口中。让我们从一个接口开始：

```java
public interface PersistableEntityManagerBookRepository<S> {
    S persistBook(S entity);
}
```

之后，我们可以为其提供一个实现。我们将使用[@PersistenceContext](https://www.baeldung.com/spring-data-entitymanager#access-entitymanager-with-spring-data)，其行为类似于[@Autowired](https://www.baeldung.com/spring-autowire)，但更具体：

```java
public class PersistableEntityManagerBookRepositoryImpl<S> implements PersistableEntityManagerBookRepository<S> {
    @PersistenceContext
    private EntityManager entityManager;

    @Override
    @Transactional
    public S persist(S entity) {
        entityManager.persist(entity);
        return entity;
    }
}
```

遵循正确的命名约定很重要，**实现应该与接口具有相同的名称，但以[Impl](https://docs.spring.io/spring-data/jpa/reference/repositories/custom-implementations.html)结尾**。为了将所有的东西结合在一起，我们需要创建另一个接口来扩展我们的自定义接口和JpaRepository<T,ID\>：

```java
public interface EntityManagerBookRepository extends JpaRepository<EntityManagerBook, Long>, 
  PersistableEntityManagerBookRepository<EntityManagerBook> {
}
```

如果实体具有ID，则persist(T)方法将抛出由[PersistentObjectException](https://www.baeldung.com/hibernate-detached-entity-passed-to-persist)引起的InvalidDataAccessApiUsageException。

## 6. 使用原生查询

更改JpaRepository<T\>默认行为的另一种方法是使用[@Query](https://www.baeldung.com/spring-data-jpa-query)注解，由于我们无法使用[JPQL](https://www.baeldung.com/jpa-queries)进行插入查询，因此我们将使用[原生SQL](https://www.baeldung.com/jpa-queries#native-query)：

```java
public interface CustomQueryBookRepository extends JpaRepository<CustomQueryBook, Long> {
    @Modifying
    @Transactional
    @Query(value = "INSERT INTO custom_query_book (id, title) VALUES (:#{#book.id}, :#{#book.title})", nativeQuery = true)
    void persist(@Param("book") CustomQueryBook book);
}
```

这将强制该方法执行特定行为。然而，它有几个问题。**主要问题是我们必须提供一个ID，如果我们将其生成委托给数据库，这是不可能的**。另一件事与[修改](https://www.baeldung.com/spring-data-jpa-modifying-annotation)查询有关，它们只能返回void或int，这可能不方便。

总体而言，此方法会因ID冲突而导致[DataIntegrityViolationException](https://www.baeldung.com/spring-dataIntegrityviolationexception)，这可能会产生开销。此外，该方法的行为并不简单，因此应尽可能避免使用这种方法。

## 7. Persistable<ID\>接口

我们可以通过实现Persistable<ID\>接口来实现类似的结果：

```java
public interface Persistable<ID> {
    @Nullable
    ID getId();
    boolean isNew();
}
```

简而言之，该接口允许添加自定义逻辑，同时识别实体是新实体还是已存在实体。**这与我们在默认save(S)实现中看到的isNew()方法相同**。

我们可以实现这个接口并始终告诉JPA该实体是新的：

```java
@Entity
public class PersistableBook implements Persistable<Long> {
    // fields, getters, and setters
    @Override
    public boolean isNew() {
        return true;
    }
}
```

这将强制save(S)始终选择persist(S)，在违反ID约束的情况下抛出异常。**这个解决方案通常会起作用，但它可能会产生问题，因为我们违反了持久化契约，考虑到所有实体都是新的**。

## 8. 不可更新的字段

最好的方法是将字段定义为不可更新，**这是处理问题的最简洁的方法，并且允许我们仅识别那些我们想要更新的字段**。我们可以使用[@Column](https://www.baeldung.com/jpa-basic-annotation#basic-vs-column)注解来定义这样的字段：

```java
@Entity
public class UnapdatableBook {
    @Id
    @GeneratedValue
    private Long id;
    @Column(updatable = false)
    private String title;

    private String author;

    // constructors, getters, and setters
}
```

JPA在更新时会默默地忽略这些字段，同时，它仍然允许我们更新其他字段：

```java
@Test
void givenDatasourceWhenUpdateBookTheBookUpdatedIgnored() {
    UnapdatableBook book = new UnapdatableBook(TITLE, AUTHOR);
    UnapdatableBook persistedBook = repository.save(book);
    Long id = persistedBook.getId();
    persistedBook.setTitle(NEW_TITLE);
    persistedBook.setAuthor(NEW_AUTHOR);
    repository.save(persistedBook);
    Optional<UnapdatableBook> actualBook = repository.findById(id);
    assertTrue(actualBook.isPresent());
    assertThat(actualBook.get().getId()).isEqualTo(id);
    assertThat(actualBook.get().getTitle()).isEqualTo(TITLE);
    assertThat(actualBook.get().getAuthor()).isEqualTo(NEW_AUTHOR);
}
```

我们没有更改书名，但成功更新了书的作者。

## 9. 总结

Spring JPA不仅为我们提供了方便的与数据库交互的工具，而且还具有高度的灵活性和可配置性。我们可以使用许多不同的方法来改变默认行为并满足我们应用程序的需求。

针对特定情况选择正确的方法需要对可用功能有深入的了解。