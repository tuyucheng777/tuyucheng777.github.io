---
layout: post
title:  在Spring Data JPA中启用事务锁
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

在本快速教程中，我们将讨论在Spring Data JPA中为[自定义查询方法](https://www.baeldung.com/spring-data-jpa-query)和预定义Repository CRUD方法启用事务锁。

我们还将了解不同的锁类型以及设置事务锁超时。

## 2. 锁类型

JPA定义了两种主要锁类型，悲观锁和乐观锁。

### 2.1 悲观锁

**当我们在事务中使用[悲观锁](https://www.baeldung.com/jpa-pessimistic-locking)并访问实体时，它将立即被锁定，事务通过提交或回滚事务来释放锁**。

### 2.2 乐观锁

**在[乐观锁](https://www.baeldung.com/jpa-optimistic-locking)中，事务不会立即锁定实体**。相反，事务通常会保存实体的状态并为其分配一个版本号。

当我们尝试在不同的事务中更新实体的状态时，事务会在更新过程中将保存的版本号与现有的版本号进行比较。

此时，如果版本号不同，则意味着我们无法修改实体。如果存在活动事务，则该事务将回滚，并且底层JPA实现将抛出[OptimisticLockException](https://docs.oracle.com/javaee/7/api/javax/persistence/OptimisticLockException.html)。

根据最适合我们当前开发环境的方法，我们还可以使用其他方法，例如时间戳、哈希值计算或序列化校验和。

## 3. 在查询方法上启用事务锁

**要获取实体上的锁，我们可以通过传递所需的锁模式类型，用[Lock](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Lock.html)注解来标注目标查询方法**。

[锁模式类型](https://docs.oracle.com/javaee/7/api/javax/persistence/LockModeType.html)是锁定实体时要指定的枚举值，指定的锁定模式随后会传播到数据库，以对实体对象应用相应的锁。

要在Spring Data JPA Repository的自定义查询方法上指定锁，我们可以使用@Lock标注该方法并指定所需的锁模式类型：

```java
@Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT)
@Query("SELECT c FROM Customer c WHERE c.orgId = ?1")
public List<Customer> fetchCustomersByOrgId(Long orgId);
```

为了对预定义的Repository方法(例如findAll或findById(id))强制锁定，我们必须在Repository中声明该方法并使用Lock注解对该方法进行标注：

```java
@Lock(LockModeType.PESSIMISTIC_READ)
public Optional<Customer> findById(Long customerId);
```

当明确启用锁并且没有活动事务时，底层JPA实现将抛出[TransactionRequiredException](https://docs.oracle.com/javaee/7/api/javax/persistence/TransactionRequiredException.html)。

如果无法授予锁，并且锁冲突不会导致事务回滚，则JPA会抛出[LockTimeoutException](https://docs.oracle.com/javaee/7/api/javax/persistence/LockTimeoutException.html)，但不会标记活动事务进行回滚。

## 4. 设置事务锁超时

使用悲观锁时，数据库将尝试立即锁定实体。当无法立即获取锁时，底层JPA实现将抛出LockTimeoutException。为了避免此类异常，我们可以指定锁定超时值。

在Spring Data JPA中，可以通过在查询方法上放置[QueryHint](https://docs.oracle.com/javaee/7/api/javax/persistence/QueryHint.html)[QueryHints](https://docs.spring.io/spring-data/jpa/docs/current/api/index.html?org/springframework/data/jpa/repository/QueryHints.html)，使用注解来指定锁超时：

```java
@Lock(LockModeType.PESSIMISTIC_READ)
@QueryHints({@QueryHint(name = "jakarta.persistence.lock.timeout", value = "3000")})
public Optional<Customer> findById(Long customerId);
```

我们可以在此[ObjectDB文章](https://www.objectdb.com/java/jpa/persistence/lock#Pessimistic_Locking_)中找到有关在不同范围内设置锁超时提示的更多详细信息。

## 5. 总结

在本文中，我们探讨了不同类型的事务锁模式。然后，我们学习了如何在Spring Data JPA中启用事务锁，我们还介绍了如何设置锁定超时。

在正确的位置应用正确的事务锁有助于维护大容量并发使用应用程序中的数据完整性。

**当事务需要严格遵守ACID规则时，我们应该使用悲观锁。当我们需要允许多个并发读取并且应用程序上下文中可以接受最终一致性时，应该应用乐观锁**。