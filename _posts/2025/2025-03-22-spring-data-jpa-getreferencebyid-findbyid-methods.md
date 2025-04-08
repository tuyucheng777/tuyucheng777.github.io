---
layout: post
title:  何时在Spring Data JPA中使用getReferenceById()和findById()方法
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

[JpaRepository](https://www.baeldung.com/spring-data-repositories)为我们提供了CRUD操作的基本方法，然而，其中一些方法并不那么简单，有时很难确定哪种方法最适合特定情况。

**getReferenceById(ID)和findById(ID)是经常造成这种混淆的方法**，这些方法是getOne(ID)、findOne(ID)和getById(ID)的新API名称。

在本教程中，我们将了解它们之间的区别，并找出每种方法更适合的情况。

## 2. findById()

让我们从这两种方法中最简单的一种开始。这种方法正如它所说的那样，通常开发人员不会遇到任何问题。它只是在给定特定ID的Repository中查找实体：

```java
@Override
Optional<T> findById(ID id);
```

该方法返回一个[Optional](https://www.baeldung.com/java-optional)。因此，如果我们传递一个不存在的ID，则假设它将为空，这是正确的。

该方法在底层使用了急切加载，因此每当我们调用此方法时，我们都会向数据库发送请求。让我们看一个例子：

```java
public User findUser(long id) {
    log.info("Before requesting a user in a findUser method");
    Optional<User> optionalUser = repository.findById(id);
    log.info("After requesting a user in a findUser method");
    User user = optionalUser.orElse(null);
    log.info("After unwrapping an optional in a findUser method");
    return user;
}
```

该方法会生成以下日志：

```text
[2023-12-27 12:56:32,506]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.SimpleUserService - Before requesting a user in a findUser method
[2023-12-27 12:56:32,508]-[main] DEBUG org.hibernate.SQL - 
    select
        user0_."id" as id1_0_0_,
        user0_."first_name" as first_na2_0_0_,
        user0_."second_name" as second_n3_0_0_ 
    from
        "users" user0_ 
    where
        user0_."id"=?
[2023-12-27 12:56:32,508]-[main] TRACE org.hibernate.type.descriptor.sql.BasicBinder - binding parameter [1] as [BIGINT] - [1]
[2023-12-27 12:56:32,510]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.SimpleUserService - After requesting a user in a findUser method
[2023-12-27 12:56:32,510]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.SimpleUserService - After unwrapping an optional in a findUser method
```

**Spring可能会在事务中批量处理请求，但始终会执行它们**。总体而言，findById(ID)不会试图给我们带来惊喜，而是按照我们的预期行事。然而，由于它有一个类似的对应物，所以会出现混乱。

## 3. getReferenceById()

此方法具有与findById(ID)类似的签名：

```java
@Override
T getReferenceById(ID id);
```

仅根据签名判断，我们可以假设如果实体不存在，此方法将引发异常。**确实如此，但这并不是我们唯一的区别。这些方法之间的主要区别在于getReferenceById(ID)是惰性的**，Spring不会发送数据库请求直到我们明确尝试在事务中使用该实体。

### 3.1 事务

**每个事务都有一个与之配合的专用持久化上下文**。有时，我们可以将持久化上下文扩展到事务范围之外，但这并不常见，并且仅对特定场景有用。让我们检查一下持久化上下文在事务方面的行为：

![](/assets/images/2025/springboot/springdatajpagetreferencebyidfindbyidmethods01.png)

在事务内，持久化上下文内的所有实体在数据库中都有直接表示，这是一个[托管状态](https://www.baeldung.com/java-optional)。因此，对实体的所有更改都将反映在数据库中。在事务之外，实体移至[分离状态](https://www.baeldung.com/hibernate-entity-lifecycle#detached-entity)，更改将不会反映出来，直到实体移回托管状态。

延迟加载实体的行为略有不同，除非我们在持久化上下文中明确使用它们，否则Spring不会加载它们：

![](/assets/images/2025/springboot/springdatajpagetreferencebyidfindbyidmethods02.png)

Spring将分配一个空的代理占位符来从数据库中延迟获取实体。**但是，如果我们不这样做，实体将在事务之外保持为空代理，并且对它的任何调用都将导致[LazyInitializationException](https://www.baeldung.com/hibernate-initialize-proxy-exception)**。但是，如果我们以需要内部信息的方式调用或与实体交互，则将对数据库进行实际请求：

![](/assets/images/2025/springboot/springdatajpagetreferencebyidfindbyidmethods03.png)

### 3.2 非事务服务

了解了事务的行为和持久化上下文后，让我们检查以下调用Repository的非事务服务。**findUserReference没有连接到它的持久化上下文，并且getReferenceById将在单独的事务中执行**：

```java
public User findUserReference(long id) {
    log.info("Before requesting a user");
    User user = repository.getReferenceById(id);
    log.info("After requesting a user");
    return user;
}
```

此代码将生成以下日志输出：

```text
[2023-12-27 13:21:27,590]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.TransactionalUserReferenceService - Before requesting a user
[2023-12-27 13:21:27,590]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.TransactionalUserReferenceService - After requesting a user
```

我们可以看到，没有数据库请求。**理解延迟加载后，Spring假设如果我们不使用其中的实体，我们可能不需要它**。从技术上讲，我们不能使用它，因为我们的唯一的事务是getReferenceById方法内的事务。因此，我们返回的user将是一个空代理，如果我们访问其内部，将导致异常：

```java
public User findAndUseUserReference(long id) {
    User user = repository.getReferenceById(id);
    log.info("Before accessing a username");
    String firstName = user.getFirstName();
    log.info("This message shouldn't be displayed because of the thrown exception: {}", firstName);
    return user;
}
```

### 3.3 事务服务

让我们检查一下我们使用[@Transactional](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)服务时的行为：

```java
@Transactional
public User findUserReference(long id) {
    log.info("Before requesting a user");
    User user = repository.getReferenceById(id);
    log.info("After requesting a user");
    return user;
}
```

出于与上一个示例相同的原因，这将为我们提供类似的结果，因为我们不在事务中使用实体：

```text
[2023-12-27 13:32:44,486]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.TransactionalUserReferenceService - Before requesting a user
[2023-12-27 13:32:44,486]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.TransactionalUserReferenceService - After requesting a user
```

此外，任何在此事务服务方法之外与此用户交互的尝试都会导致异常：

```java
@Test
void whenFindUserReferenceUsingOutsideServiceThenThrowsException() {
    User user = transactionalService.findUserReference(EXISTING_ID);
    assertThatExceptionOfType(LazyInitializationException.class)
        .isThrownBy(user::getFirstName);
}
```

但是，现在，findUserReference方法定义了我们的事务范围。这意味着我们可以尝试在服务方法中访问用户，并且它应该会导致对数据库的调用：

```java
@Transactional
public User findAndUseUserReference(long id) {
    User user = repository.getReferenceById(id);
    log.info("Before accessing a username");
    String firstName = user.getFirstName();
    log.info("After accessing a username: {}", firstName);
    return user;
}
```

上面的代码将按以下顺序输出消息：

```text
[2023-12-27 13:32:44,331]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.TransactionalUserReferenceService - Before accessing a username
[2023-12-27 13:32:44,331]-[main] DEBUG org.hibernate.SQL - 
    select
        user0_."id" as id1_0_0_,
        user0_."first_name" as first_na2_0_0_,
        user0_."second_name" as second_n3_0_0_ 
    from
        "users" user0_ 
    where
        user0_."id"=?
[2023-12-27 13:32:44,331]-[main] TRACE org.hibernate.type.descriptor.sql.BasicBinder - binding parameter [1] as [BIGINT] - [1]
[2023-12-27 13:32:44,331]-[main] INFO  cn.tuyucheng.taketoday.spring.data.persistence.findvsget.service.TransactionalUserReferenceService - After accessing a username: Saundra
```

对数据库的请求不是在我们调用getReferenceById()时发出的，而是在我们调用user.getFirstName()时发出的。

### 3.4 具有新Repository事务的事务服务

让我们看一个更复杂的例子。想象一下，我们有一个Repository方法，每当我们调用它时，它都会创建一个单独的事务：

```java
@Override
@Transactional(propagation = Propagation.REQUIRES_NEW)
User getReferenceById(Long id);
```

**[Propagation.REQUIRES_NEW](https://www.baeldung.com/spring-transactional-propagation-isolation#6requiresnew-propagation)表示外部事务不会传播，并且Repository方法将创建其持久化上下文**。在这种情况下，即使如果我们使用事务服务，Spring将创建两个不会交互的独立持久化上下文，任何使用user的尝试都会导致异常：

```java
@Test
void whenFindUserReferenceUsingInsideServiceThenThrowsExceptionDueToSeparateTransactions() {
    assertThatExceptionOfType(LazyInitializationException.class)
        .isThrownBy(() -> transactionalServiceWithNewTransactionRepository.findAndUseUserReference(EXISTING_ID));
}
```

我们可以使用几种不同的传播配置来创建事务之间更复杂的交互，并且它们可以产生不同的结果。

### 3.5 无需获取访问实体

让我们考虑一个现实生活中的场景，假设我们有一个Group类：

```java
@Entity
@Table(name = "group")
public class Group {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    @OneToOne
    private User administrator;
    @OneToMany(mappedBy = "id")
    private Set<User> users = new HashSet<>();
    // getters, setters and other methods
}
```

我们想将用户作为管理员添加到组中，我们可以使用findById()或getReferenceById()。在此测试中，我们使用findById()获取用户并将其设置为新组的管理员：

```java
@Test
void givenEmptyGroup_whenAssigningAdministratorWithFindBy_thenAdditionalLookupHappens() {
    Optional<User> optionalUser = userRepository.findById(1L);
    assertThat(optionalUser).isPresent();
    User user = optionalUser.get();
    Group group = new Group();
    group.setAdministrator(user);
    groupRepository.save(group);
    assertSelectCount(2);
    assertInsertCount(1);
}
```

可以合理地假设我们应该有一个SELECT查询，但我们得到了两个，这是因为额外的ORM检查。让我们做一个类似的操作，但改用getReferenceById()：

```java
@Test
void givenEmptyGroup_whenAssigningAdministratorWithGetByReference_thenNoAdditionalLookupHappens() {
    User user = userRepository.getReferenceById(1L);
    Group group = new Group();
    group.setAdministrator(user);
    groupRepository.save(group);
    assertSelectCount(0);
    assertInsertCount(1);
}
```

在这种情况下，我们不需要有关用户的其他信息；我们只需要一个ID。因此，我们可以使用getReferenceById()方便地提供给我们的占位符，并且我们只需一个INSERT而无需额外的SELECT。

这样，数据库在映射时会照顾数据的正确性。例如，我们在使用不正确的ID时会收到异常：

```java
@Test
void givenEmptyGroup_whenAssigningIncorrectAdministratorWithGetByReference_thenErrorIsThrown() {
    User user = userRepository.getReferenceById(-1L);
    Group group = new Group();
    group.setAdministrator(user);
    assertThatExceptionOfType(DataIntegrityViolationException.class)
        .isThrownBy(() -> groupRepository.save(group));
    assertSelectCount(0);
    assertInsertCount(1);
}
```

同时，我们仍然有一个没有任何SELECT的INSERT。

但是，我们不能使用相同的方法将用户添加为组成员。因为我们使用Set，所以将调用equals(T)和hashCode()方法。Hibernate会抛出异常，因为getReferenceById()不会获取实际对象：

```java
@Test
void givenEmptyGroup_whenAddingUserWithGetByReference_thenTryToAccessInternalsAndThrowError() {
    User user = userRepository.getReferenceById(1L);
    Group group = new Group();
    assertThatExceptionOfType(LazyInitializationException.class)
        .isThrownBy(() -> group.addUser(user));
}
```

因此，关于方法的决定应该考虑数据类型和我们使用该实体的环境。

## 4. 总结

findById()和getReferenceById()的主要区别是将实体加载到持久化上下文中的时间，了解这一点可能有助于实现优化并避免不必要的数据库查找，这个过程与事务及其传播紧密相关，这就是为什么应该观察事务之间的关系。