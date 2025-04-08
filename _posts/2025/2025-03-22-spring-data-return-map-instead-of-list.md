---
layout: post
title:  在Spring Data JPA中返回Map而不是List
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

[Spring JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)提供了非常灵活且方便的API来与数据库交互。但是，有时，我们需要对其进行自定义或向返回的集合添加更多功能。

使用[Map](https://www.baeldung.com/java-hashmap)作为JPA Repository方法的返回类型可能有助于在服务和数据库之间创建更直接的交互。**不幸的是，Spring不允许这种转换自动发生**。在本教程中，我们将检查如何克服这个问题并学习一些有趣的技术来实现我们的Repository功能更加强大。

## 2. 手动实现

当框架无法提供某些功能时，解决问题的最明显方法就是我们自己实现它。在这种情况下，JPA允许我们从头开始实现Repository，跳过整个生成过程，或使用默认方法来获得两全其美的效果。

### 2.1 使用List

我们可以实现一种方法将结果List映射到Map中，[Stream API](https://www.baeldung.com/java-8-streams)对这项任务有很大帮助，几乎允许单行实现：

```java
default Map<Long, User> findAllAsMapUsingCollection() {
    return findAll().stream()
        .collect(Collectors.toMap(User::getId, Function.identity()));
}
```

### 2.2 使用Stream

我们可以做类似的事情，但直接使用Stream。为此，我们可以确定一个将返回用户流的自定义方法。幸运的是，Spring JPA支持此类返回类型，我们可以从自动生成中受益：

```java
@Query("select u from User u")
Stream<User> findAllAsStream();
```

之后，我们可以实现一个自定义方法，将结果映射到我们需要的数据结构中：

```java
@Transactional
default Map<Long, User> findAllAsMapUsingStream() {
    return findAllAsStream()
        .collect(Collectors.toMap(User::getId, Function.identity()));
}
```

返回Stream的Repository方法应在事务内调用，本例中，我们直接在默认方法中添加了[@Transactional](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)注解。

### 2.3 使用Streamable

这与之前讨论的方法类似，唯一的变化是我们将使用@Streamable，我们需要创建一个自定义方法来首先返回它：

```java
@Query("select u from User u")
Streamable<User> findAllAsStreamable();
```

然后，我们可以适当地映射结果：

```java
default Map<Long, User> findAllAsMapUsingStreamable() {
    return findAllAsStreamable().stream()
        .collect(Collectors.toMap(User::getId, Function.identity()));
}
```

## 3. 自定义Streamable包装器

前面的示例向我们展示了该问题的非常简单的解决方案。然而，假设我们有几种不同的操作或数据结构，我们想要将结果映射到这些操作或数据结构。**在这种情况下，我们最终可能会得到分散在代码中的笨拙映射器或执行类似操作的多个Repository方法**。

更好的方法可能是创建一个表示实体集合的专用类，并将与该集合上的操作相关的所有方法放置在其中。为此，我们将使用Streamable。

如前所述，Spring JPA理解Streamable并可以将结果映射到它。有趣的是，我们可以扩展Streamable并为其提供方便的方法。让我们创建一个Users类，它代表User对象的集合：

```java
public class Users implements Streamable<User> {

    private final Streamable<User> userStreamable;

    public Users(Streamable<User> userStreamable) {
        this.userStreamable = userStreamable;
    }

    @Override
    public Iterator<User> iterator() {
        return userStreamable.iterator();
    }

    // custom methods
}
```

为了使其与JPA配合使用，我们应该遵循一个简单的惯例。**首先，我们应该实现Streamable，其次，提供Spring能够初始化它的方法**。初始化部分可以通过接收Streamable的公共构造函数或名称为(Streamable<T\>)或valueOf(Streamable<T\>)的静态工厂来处理。

之后，我们可以使用Users作为JPA Repository方法的返回类型：

```java
@Query("select u from User u")
Users findAllUsers();
```

现在，我们可以将Repository中保留的方法直接放置在Users类中：

```java
public Map<Long, User> getUserIdToUserMap() {
    return stream().collect(Collectors.toMap(User::getId, Function.identity()));
}
```

最好的部分是我们可以使用与处理或映射User实体相关的所有方法，假设我们想按某些标准过滤掉用户：

```java
@Test
void fetchUsersInMapUsingStreamableWrapperWithFilterThenAllOfThemPresent() {
    Users users = repository.findAllUsers();
    int maxNameLength = 4;
    List<User> actual = users.getAllUsersWithShortNames(maxNameLength);
    User[] expected = {
            new User(9L, "Moe", "Oddy"),
            new User(25L, "Lane", "Endricci"),
            new User(26L, "Doro", "Kinforth"),
            new User(34L, "Otho", "Rowan"),
            new User(39L, "Mel", "Moffet")
    };
    assertThat(actual).containsExactly(expected);
}
```

此外，我们可以通过某种方式对它们进行分组：

```java
@Test
void fetchUsersInMapUsingStreamableWrapperAndGroupingThenAllOfThemPresent() {
    Users users = repository.findAllUsers();
    Map<Character, List<User>> alphabeticalGrouping = users.groupUsersAlphabetically();
    List<User> actual = alphabeticalGrouping.get('A');
    User[] expected = {
            new User(2L, "Auroora", "Oats"),
            new User(4L, "Alika", "Capin"),
            new User(20L, "Artus", "Rickards"),
            new User(27L, "Antonina", "Vivian")};
    assertThat(actual).containsExactly(expected);
}
```

通过这种方式，我们可以隐藏此类方法的实现，消除服务中的混乱，并卸载Repository。

## 4. 总结

Spring JPA允许自定义，但有时实现这一点非常简单，围绕框架限制的类型构建应用程序可能会影响代码的质量甚至应用程序的设计。

使用自定义集合作为返回类型可能会使设计更加简单，并且不会因映射和过滤逻辑而变得混乱，**对实体集合使用专用包装器可以进一步改进代码**。