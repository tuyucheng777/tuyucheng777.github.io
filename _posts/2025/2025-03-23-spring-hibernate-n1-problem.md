---
layout: post
title:  Hibernate和Spring Data JPA中的N + 1问题
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

[Spring JPA和Hibernate](https://www.baeldung.com/learn-jpa-hibernate)提供了强大的工具来实现无缝数据库通信，但是，由于客户端将更多的控制权委托给框架，因此生成的查询可能远非最佳。

在本教程中，我们将回顾使用Spring JPA和Hibernate时常见的N + 1问题，我们将检查可能导致问题的不同情况。

## 2. 社交媒体平台

为了更好地形象化这个问题，我们需要概述实体之间的关系。让我们以一个简单的社交网络平台为例，只有User和Post：

![](/assets/images/2025/springdata/springhibernaten1problem01.png)

我们在图中用的是[Iterable](https://www.baeldung.com/java-iterable-to-collection)，并且我们将为每个示例提供具体的实现：[List](https://www.baeldung.com/java-arraylist)或[Set](https://www.baeldung.com/java-hashset)。

为了测试请求数量，我们将使用[专用库](https://www.baeldung.com/spring-jpa-onetomany-list-vs-set#testing)而不是检查日志。不过，我们将参考日志以更好地了解请求的结构。

如果每个示例中未明确提及，则关系的[获取类型](https://www.baeldung.com/hibernate-lazy-eager-loading)将被视为默认类型。所有一对一关系都是即时获取，而对多关系则是延迟获取。此外，代码示例使用[Lombok](https://www.baeldung.com/intro-to-project-lombok)来减少代码中的噪音。

## 3. N + 1问题

[N +1问题](https://www.baeldung.com/cs/orm-n-plus-one-select-problem)是指，对于单个请求(例如获取用户)，我们会对每个用户发出额外请求以获取其信息。**虽然此问题通常与延迟加载有关，但情况并非总是如此**。

任何类型的关系都可能出现此问题，但是它通常出现在多对多或一对多关系中。

### 3.1 延迟获取

首先，让我们看看延迟加载如何导致N + 1问题，我们将考虑以下示例：

```java
@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String email;
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "author")
    protected List<Post> posts;
    // constructors, getters, setters, etc.
}
```

用户与帖子具有一对多关系，这意味着每个用户都有多个帖子。我们没有明确标识字段的获取策略，该策略是从注解中推断出来的。**如前所述，@OneToMany默认具有延迟获取**：

```java
@Target({METHOD, FIELD}) 
@Retention(RUNTIME)
public @interface OneToMany {
    Class targetEntity() default void.class;
    CascadeType[] cascade() default {};
    FetchType fetch() default FetchType.LAZY;
    String mappedBy() default "";
    boolean orphanRemoval() default false;
}
```

如果我们尝试获取所有用户，则惰性获取不会提取比我们访问的更多信息：

```java
@Test
void givenLazyListBasedUser_WhenFetchingAllUsers_ThenIssueOneRequests() {
    getUserService().findAll();
    assertSelectCount(1);
}
```

因此，要获取所有用户，我们将发出单个请求。让我们尝试访问帖子，Hibernate将发出额外的请求，因为之前没有获取信息。对于单个用户，这意味着总共需要两个请求：

```java
@ParameterizedTest
@ValueSource(longs = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10})
void givenLazyListBasedUser_WhenFetchingOneUser_ThenIssueTwoRequest(Long id) {
    getUserService().getUserByIdWithPredicate(id, user -> !user.getPosts().isEmpty());
    assertSelectCount(2);
}
```

getUserByIdWithPredicate(Long, [Predicate](https://www.baeldung.com/java-8-functional-interfaces#Predicates))方法会过滤用户，但其在测试中的主要目标是触发加载。我们将有1 + 1个请求，但如果我们对其进行扩展，我们将遇到N + 1问题：

```java
@Test
void givenLazyListBasedUser_WhenFetchingAllUsersCheckingPosts_ThenIssueNPlusOneRequests() {
    int numberOfRequests = getUserService().countNumberOfRequestsWithFunction(users -> {
        List<List<Post>> usersWithPosts = users.stream()
                .map(User::getPosts)
                .filter(List::isEmpty)
                .toList();
        return users.size();
    });
    assertSelectCount(numberOfRequests + 1);
}
```

我们应该谨慎对待延迟加载，在某些情况下，延迟加载有助于减少从数据库获取的数据。**但是，如果我们在大多数情况下访问延迟加载的信息，则可能会增加请求量**。为了做出最佳判断，我们必须调查访问模式。

### 3.2 即时获取

在大多数情况下，预加载可以帮助我们解决N + 1问题。但是，结果取决于我们实体之间的关系。让我们考虑一个类似的User类，但明确设置了预加载：

```java
@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String email;
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "author", fetch = FetchType.EAGER)
    private List<Post> posts;
    // constructors, getters, setters, etc.
}
```

如果我们获取单个用户，则获取类型将强制Hibernate在单个请求中加载所有数据：

```java
@ParameterizedTest
@ValueSource(longs = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10})
void givenEagerListBasedUser_WhenFetchingOneUser_ThenIssueOneRequest(Long id) {
    getUserService().getUserById(id);
    assertSelectCount(1);
}
```

与此同时，获取所有用户的情况也发生了变化。**无论我们是否要使用Post，我们都会立即获得N + 1**：

```java
@Test
void givenEagerListBasedUser_WhenFetchingAllUsers_ThenIssueNPlusOneRequests() {
    List<User> users = getUserService().findAll();
    assertSelectCount(users.size() + 1);
}
```

尽管即时获取改变了Hibernate提取数据的方式，但很难称其为成功的优化。

## 4. 多个集合

让我们在初始域中引入群组：

![](/assets/images/2025/springdata/springhibernaten1problem02.png)

Group包含User列表：

```java
@Entity
public class Group {
    @Id
    private Long id;
    private String name;
    @ManyToMany
    private List<User> members;
    // constructors, getters, setters, etc.
}

```

### 4.1 延迟获取

这种关系通常与前面的惰性提取示例类似，每次访问惰性提取的信息时，我们都会收到一个新请求。

因此，除非我们直接访问用户，否则我们只会有一个请求：

```java
@Test
void givenLazyListBasedGroup_whenFetchingAllGroups_thenIssueOneRequest() {
    groupService.findAll();
    assertSelectCount( 1);
}

@ParameterizedTest
@ValueSource(longs = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10})
void givenLazyListBasedGroup_whenFetchingAllGroups_thenIssueOneRequest(Long groupId) {
    Optional<Group> group = groupService.findById(groupId);
    assertThat(group).isPresent();
    assertSelectCount(1);
}
```

但是，如果我们尝试访问组中的每个用户，就会产生N + 1问题：

```java
@Test
void givenLazyListBasedGroup_whenFilteringGroups_thenIssueNPlusOneRequests() {
    int numberOfRequests = groupService.countNumberOfRequestsWithFunction(groups -> {
        groups.stream()
                .map(Group::getMembers)
                .flatMap(Collection::stream)
                .collect(Collectors.toSet());
        return groups.size();
    });
    assertSelectCount(numberOfRequests + 1);
}
```

countNumberOfRequestsWithFunction(ToIntFunction)方法计算请求数并触发延迟加载。

### 4.2 即时获取

让我们使用即时获取检查一下行为，在请求单个组时，我们将获得以下结果：

```java
@ParameterizedTest
@ValueSource(longs = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10})
void givenEagerListBasedGroup_whenFetchingAllGroups_thenIssueNPlusOneRequests(Long groupId) {
    Optional<Group> group = groupService.findById(groupId);
    assertThat(group).isPresent();
    assertSelectCount(1 + group.get().getMembers().size());
}
```

这是合理的，因为我们需要急切地获取每个用户的信息。同时，当我们获取所有组时，请求数量会大幅增加：

```java
@Test
void givenEagerListBasedGroup_whenFetchingAllGroups_thenIssueNPlusMPlusOneRequests() {
    List<Group> groups = groupService.findAll();
    Set<User> users = groups.stream().map(Group::getMembers).flatMap(List::stream).collect(Collectors.toSet());
    assertSelectCount(groups.size() + users.size() + 1);
}
```

我们需要获取有关用户的信息，然后针对每个用户获取他们的帖子。从技术上讲，我们遇到了N + M + 1的情况。**因此，无论是惰性获取还是急切获取都无法完全解决问题**。

### 4.3 使用Set

让我们换一种方式来处理这种情况，我们用Set替换List。我们将使用即时获取，因为惰性Set和List的行为类似：

```java
@Entity
public class Group {
    @Id
    private Long id;
    private String name;
    @ManyToMany(fetch = FetchType.EAGER)
    private Set<User> members;
    // constructors, getters, setters, etc.
}

@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String email;
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "author", fetch = FetchType.EAGER)
    protected Set<Post> posts;
    // constructors, getters, setters, etc.
}

@Entity
public class Post {
    @Id
    private Long id;
    @Lob
    private String content;
    @ManyToOne
    private User author;
    // constructors, getters, setters, etc.
}
```

让我们运行类似的测试来看看是否有任何区别：

```java
@ParameterizedTest
@ValueSource(longs = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10})
void givenEagerSetBasedGroup_whenFetchingAllGroups_thenCreateCartesianProductInOneQuery(Long groupId) {
    groupService.findById(groupId);
    assertSelectCount(1);
}
```

我们在获取单个Group时解决了N + 1问题，Hibernate在一次请求中获取了用户及其帖子。此外，获取所有Group也减少了请求数，但仍然是N + 1：

```java
@Test
void givenEagerSetBasedGroup_whenFetchingAllGroups_thenIssueNPlusOneRequests() {
    List<Group> groups = groupService.findAll();
    assertSelectCount(groups.size() + 1);
}
```

虽然我们部分解决了这个问题，但我们又产生了另一个问题，Hibernate使用多个JOIN，造成了笛卡尔积：

```sql
SELECT g.id, g.name, gm.interest_group_id,
       u.id, u.username, u.email,
       p.id, p.author_id, p.content
FROM group g
         LEFT JOIN (group_members gm JOIN user u ON u.id = gm.members_id)
                   ON g.id = gm.interest_group_id
         LEFT JOIN post p ON u.id = p.author_id
WHERE g.id = ?
```

查询可能会变得过于复杂，并且由于对象之间存在许多依赖关系，会占用大量数据库。

**由于Set的性质，Hibernate可以确保结果集中的所有重复项都来自笛卡尔积**，List无法做到这一点，因此在使用List时应在单独的请求中获取数据以保持其完整性。

大多数关系都符合Set不变量，允许用户拥有多个相同的帖子是没有意义的。同时，我们可以明确提供[获取模式](https://www.baeldung.com/hibernate-fetchmode)，而不是依赖默认行为。

## 5. 权衡

在简单情况下，选择提取类型可能有助于减少请求数量。但是，使用简单的注解，我们对查询生成的控制有限。此外，它是透明的，域模型中的微小更改可能会产生巨大影响。

解决这个问题的最佳方法是观察系统的行为并确定访问模式，**创建单独的方法、SQL和JPQL查询可以帮助针对每种情况进行定制**。此外，我们可以使用提取模式来提示Hibernate如何加载相关实体。

添加简单的测试有助于解决模型中的意外变化，这样，我们可以确保新的关系不会产生笛卡尔积或N + 1问题。

## 6. 总结

虽然即时获取类型可以通过附加查询缓解一些简单问题，但它可能会导致其他问题，有必要测试应用程序以确保其性能。

不同的获取类型和关系组合经常会产生意想不到的结果，这就是为什么最好用测试覆盖关键部分。