---
layout: post
title:  在Spring Data JPA中计算结果行数
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

Spring Data JPA实现为[Jakarta Persistence API](https://www.baeldung.com/jpa-hibernate-persistence-context)提供Repository支持，用于管理持久化和对象关系映射和函数。

在本文中，我们将探索使用JPA计算表中行数的不同方法。

## 2. 实体类

对于我们的示例，我们将使用帐户实体，它与权限实体具有一对一的关系。

```java
@Entity
@Table(name="ACCOUNTS")
public class Account {
    @Id
    @GeneratedValue(strategy= GenerationType.SEQUENCE, generator = "accounts_seq")
    @SequenceGenerator(name = "accounts_seq", sequenceName = "accounts_seq", allocationSize = 1)
    @Column(name = "user_id")
    private int userId;
    private String username;
    private String password;
    private String email;
    private Timestamp createdOn;
    private Timestamp lastLogin;

    @OneToOne
    @JoinColumn(name = "permissions_id")
    private Permission permission;

    // getters , setters
}
```

权限属于账户实体。

```java
@Entity
@Table(name="PERMISSIONS")
public class Permission {
    @Id
    @GeneratedValue(strategy= GenerationType.SEQUENCE, generator = "permissions_id_sq")
    @SequenceGenerator(name = "permissions_id_sq", sequenceName = "permissions_id_sq", allocationSize = 1)
    private int id;

    private String type;

    // getters , setters
}
```

## 3. 使用JPA Repository

Spring Data JPA提供了一个可扩展的Repository接口，**它提供了开箱即用的查询方法和派生查询方法，例如findAll()、findBy()、save()、saveAndFlush()、count()、countBy()、delete()、deleteAll()**。

我们将定义扩展JpaRepository接口的AccountRepository接口，以便可以访问count方法。

如果我们需要根据一个或多个条件进行计数，例如countByFirstname()、countByPermission()或countByPermissionAndCreatedOnGreaterThan()，我们所需要的只是AccountRepository接口中方法的名称，然后查询派生将负责为其定义适当的SQL：

```java
public interface AccountRepository extends JpaRepository<Account, Integer> { 
    long countByUsername(String username);
    long countByPermission(Permission permission); 
    long countByPermissionAndCreatedOnGreaterThan(Permission permission, Timestamp ts)
}
```

在下面的示例中，我们将使用逻辑类中的AccountRepository来执行计数操作。

### 3.1 计算表中的所有行数

我们将定义一个逻辑类，在其中注入AccountRepository，对于简单的行count()操作，我们可以只使用accountRepository.count()即可得到结果。

```java
@Service
public class AccountStatsLogic {
    @Autowired
    private AccountRepository accountRepository;

    public long getAccountCount(){
        return accountRepository.count();
    }
}
```

### 3.2 根据单一条件计算结果行数

正如我们上面所定义的，AccountRepository包含countByPermission和countByUsername等方法名称，Spring Data JPA[查询派生](https://www.baeldung.com/spring-data-derived-queries)将派生这些方法的查询。

所以我们可以在逻辑类中使用这些方法进行条件计数，也会得到结果。

```java
@Service
public class AccountStatsLogic {
    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private PermissionRepository permissionRepository;

    public long getAccountCountByUsername(){
        String username = "user2";
        return accountRepository.countByUsername(username);
    }

    public long getAccountCountByPermission(){
        Permission permission = permissionRepository.findByType("reader");
        return accountRepository.countByPermission(permission);
    }
}
```

### 3.3 根据多个条件统计结果行数

我们还可以在查询派生中包含多个条件，如下所示，其中包含Permission和CreatedOnGreaterThan。

```java
@Service
public class AccountStatsLogic {
    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private PermissionRepository permissionRepository;

    public long getAccountCountByPermissionAndCreatedOn() throws ParseException {
        Permission permission = permissionRepository.findByType("reader");
        Date parsedDate = getDate();
        return accountRepository.countByPermissionAndCreatedOnGreaterThan(permission, new Timestamp(parsedDate.getTime()));
    }
}
```

## 4. 使用CriteriaQuery

在JPA中计算行数的下一个方法是使用[CriteriaQuery](https://www.baeldung.com/spring-data-criteria-queries)接口，**这个接口允许我们以面向对象的方式编写查询，这样我们就可以跳过编写原始SQL查询的知识**。

它要求我们构造一个CriteriaBuilder对象，然后它帮助我们构造CriteriaQuery。CriteriaQuery准备就绪后，我们可以使用entityManager中的createQuery方法来执行查询并获取结果。

### 4.1 计算所有行数

现在，当我们使用CriteriaQuery构造查询时，我们可以定义要计数的选择查询，如下所示。

```java
public long getAccountsUsingCQ() throws ParseException {
    // creating criteria builder and query
    CriteriaBuilder builder = entityManager.getCriteriaBuilder();
    CriteriaQuery<Long> criteriaQuery = builder.createQuery(Long.class);
    Root<Account> accountRoot = criteriaQuery.from(Account.class);

    // select query
    criteriaQuery.select(builder.count(accountRoot));

    // execute and get the result
    return entityManager.createQuery(criteriaQuery).getSingleResult();
}
```

### 4.2 根据单一条件统计结果行数

我们还可以扩展select查询以包含where条件，以根据某些条件过滤我们的查询。我们可以向构建器实例添加一个谓词并将其传递给where子句。

```java
public long getAccountsByPermissionUsingCQ() throws ParseException {
    CriteriaBuilder builder = entityManager.getCriteriaBuilder();
    CriteriaQuery<Long> criteriaQuery = builder.createQuery(Long.class);
    Root<Account> accountRoot = criteriaQuery.from(Account.class);
        
    List<Predicate> predicateList = new ArrayList<>(); // list of predicates that will go in where clause
    predicateList.add(builder.equal(accountRoot.get("permission"), permissionRepository.findByType("admin")));

    criteriaQuery
        .select(builder.count(accountRoot))
        .where(builder.and(predicateList.toArray(new Predicate[0])));

    return entityManager.createQuery(criteriaQuery).getSingleResult();
}
```

### 4.3 根据多个条件统计结果行数

在我们的谓词中，可以添加多个条件来过滤我们的查询，构建器实例提供了equal()和greaterThan()等条件方法来支持查询条件。

```java
public long getAccountsByPermissionAndCreateOnUsingCQ() throws ParseException {
    CriteriaBuilder builder = entityManager.getCriteriaBuilder();
    CriteriaQuery<Long> criteriaQuery = builder.createQuery(Long.class);
    Root<Account> accountRoot = criteriaQuery.from(Account.class);
    
    List<Predicate> predicateList = new ArrayList<>();
    predicateList.add(builder.equal(accountRoot.get("permission"), permissionRepository.findByType("reader")));
    predicateList.add(builder.greaterThan(accountRoot.get("createdOn"), new Timestamp(getDate().getTime())));

    criteriaQuery
        .select(builder.count(accountRoot))
        .where(builder.and(predicateList.toArray(new Predicate[0])));

    return entityManager.createQuery(criteriaQuery).getSingleResult();
}

```

## 5. 使用JPQL查询

执行计数的下一个方法是使用[JPQL](https://www.baeldung.com/spring-data-jpa-query)。**JPQL查询针对实体而不是直接针对数据库工作，或多或少看起来像SQL查询**。我们总是可以编写一个JPQL查询来计算JPA中的行数。

### 5.1 计算所有行数

entityManager提供了一个createQuery()方法，该方法将JPQL查询作为参数并在数据库上执行该查询。

```java
public long getAccountsByJPQL() throws ParseException {
    Query query = entityManager.createQuery("SELECT COUNT(*) FROM Account a");
    return (long) query.getSingleResult();
}
```

### 5.2 根据单一条件统计结果行数

在JPQL查询中，我们可以像在原始SQL中一样包含WHERE条件来过滤查询并计算返回的行数。

```java
public long getAccountsByPermissionUsingJPQL() throws ParseException {
    Query query = entityManager.createQuery("SELECT COUNT(*) FROM Account a WHERE a.permission = ?1");
    query.setParameter(1, permissionRepository.findByType("admin"));
    return (long) query.getSingleResult();
}
```

### 5.3 根据多个条件计算结果行数

在JPQL查询中，我们可以像在原始SQL中一样在WHERE子句中包含多个条件来过滤查询并计算返回的行数。

```java
public long getAccountsByPermissionAndCreatedOnUsingJPQL() throws ParseException {
    Query query = entityManager.createQuery("SELECT COUNT(*) FROM Account a WHERE a.permission = ?1 and a.createdOn > ?2");
    query.setParameter(1, permissionRepository.findByType("admin"));
    query.setParameter(2, new Timestamp(getDate().getTime()));
    return (long) query.getSingleResult();
}
```

## 6. 使用Hibernate Criteria计算行数

对于使用旧版Hibernate的应用程序或仍使用[Criteria API](https://www.baeldung.com/hibernate-criteria-queries)的情况，Hibernate提供了自己的Criteria接口来构建动态查询。在未使用JPA的CriteriaQuery API的情况下，这可以成为计算行数的有用替代方案。

尽管Hibernate的Criteria API已被弃用，取而代之的是CriteriaQuery API，但它在许多旧式应用程序中仍然很有用。

在此示例中，我们将使用两个实体Foo和Bar，它们通过多对一关系关联。每个Foo对象都链接到一个特定的Bar对象，我们将根据应用于Foo及其关联Bar的条件来计算Foo对象的数量。

### 6.1 统计表中的所有行

**使用Hibernate Criteria API，我们可以通过在Criteria上设置Projection来计算表中的所有行**。在此示例中，我们将配置Projections.rowCount()来检索Foo实体中的行数：

```java
public long countAllRowsUsingHibernateCriteria() {
    Session session = entityManager.unwrap(Session.class);
    Criteria criteria = session.createCriteria(Foo.class);
    criteria.setProjection(Projections.rowCount());
    Long count = (Long) criteria.uniqueResult();
    return count != null ? count : 0L;
}
```

在countAllRowsUsingHibernateCriteria()方法中，我们首先通过调用unwrap()方法从EntityManager获取Hibernate Session。然后，我们为实体创建一个Criteria实例，使用Projections.rowCount()指定我们想要计算所有行数。最后，我们调用uniqueResult()来检索查询中的单个计数结果。

### 6.2 根据单一条件计数行

**要根据单个条件计算行数，我们可以向Criteria添加限制以过滤结果**。在此示例中，我们根据特定的Bar对象名称计算Foo对象的数量：

```java
public long getFooCountByBarNameUsingHibernateCriteria(String barName) {
    Session session = entityManager.unwrap(Session.class);
    Criteria criteria = session.createCriteria(Foo.class);
    criteria.createAlias("bar", "b");
    criteria.add(Restrictions.eq("b.name", barName));
    criteria.setProjection(Projections.rowCount());
    return (Long) criteria.uniqueResult();
}
```

在此代码中，我们使用createAlias()为bar字段创建别名“b”。然后，我们使用Restrictions.eq()添加限制，将其设置为按指定的barName过滤行。最后，我们使用Projections.rowCount()表示我们想要符合此条件的行数，并调用uniqueResult()来检索结果。

### 6.3 根据多个条件计算行数

**我们可以向Criteria添加多个Restrictions，以便根据多个条件计算行数**。在此示例中，我们计算Foo行数，其中关联的Bar的名称是特定值，并且Foo的名称与指定值匹配：

```java
public long getFooCountByBarNameAndFooNameUsingHibernateCriteria(String barName, String fooName) {
    Session session = entityManager.unwrap(Session.class);
    Criteria criteria = session.createCriteria(Foo.class);
    criteria.createAlias("bar", "b");
    criteria.add(Restrictions.eq("b.name", barName));
    criteria.add(Restrictions.eq("name", fooName));
    criteria.setProjection(Projections.rowCount());
    return (Long) criteria.uniqueResult();
}
```

与上一个示例类似，我们使用createAlias()为bar字段创建别名。这次，我们添加了两个限制：一个使用Restrictions.eq()按bar.name过滤行，另一个使用Restrictions.eq()过滤Foo对象名称与指定值匹配的行。然后，我们设置Projections.rowCount()来计算符合这些条件的行数，并调用uniqueResult()来检索结果。

## 7. 总结

在本教程中，我们学习了一些不同的方法来计算JPA中的行数，CriteriaBuilder和Spring Data JPA查询派生等规范帮助我们轻松编写不同条件的计数查询。

虽然CriteriaQuery和Spring Data JPA查询派生可以帮助我们构建不需要原始SQL知识的查询，但在某些用例中，如果它不能达到目的，我们始终可以使用JPQL编写原始SQL。