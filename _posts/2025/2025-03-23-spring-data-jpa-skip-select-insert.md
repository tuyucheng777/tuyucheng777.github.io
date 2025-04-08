---
layout: post
title:  Spring Data JPA中插入之前跳过选择
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在某些情况下，当我们使用[Spring Data JPA Repository](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)保存实体时，我们可能会在日志中遇到额外的SELECT，这可能会因大量额外调用而导致性能问题。

**在本教程中，我们将探讨一些跳过日志中的SELECT并提高性能的方法**。

## 2. 设置

在深入研究Spring Data JPA并进行测试之前，我们需要采取一些准备步骤。

### 2.1 依赖

为了创建我们的测试Repository，我们将使用[Spring Data JPA](https://mvnrepository.com/artifact/org.springframework.data/spring-data-jpa)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

作为测试数据库，我们将使用H2数据库，让我们添加它的[依赖](https://mvnrepository.com/artifact/com.h2database/h2)：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

在我们的集成测试中，我们将使用测试Spring上下文，让我们添加[spring-boot-starter-test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 2.2 配置

以下是我们在示例中使用的JPA配置：

```properties
spring.jpa.hibernate.dialect=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.hibernate.show_sql=true
spring.jpa.hibernate.hbm2ddl.auto=create-drop
```

根据此配置，我们将让Hibernate生成模式并将所有SQL查询记录到日志中。

## 3. SELECT查询的原因

**让我们看看为什么我们有这样的额外的SELECT查询来实现简单Repository的原因**。

首先，让我们创建一个实体：

```kotlin
@Entity
public class Task {

    @Id
    private Integer id;
    private String description;

    //getters and setters
}
```

现在，让我们为该实体创建一个Repository：

```java
@Repository
public interface TaskRepository extends JpaRepository<Task, Integer> {
}
```

现在，让我们保存一个指定ID的新任务：

```java
@Autowired
private TaskRepository taskRepository;

@Test
void givenRepository_whenSaveNewTaskWithPopulatedId_thenExtraSelectIsExpected() {
    Task task = new Task();
    task.setId(1);
    taskRepository.saveAndFlush(task);
}
```

当我们调用saveAndFlush()时-save()方法的行为将与我们的Repository的方法相同，在内部我们使用以下代码：

```typescript
public<S extends T> S save(S entity){
    if(isNew(entity)){
        entityManager.persist(entity);
        return entity;
    } else {
        return entityManager.merge(entity);
    }
}
```

因此，如果我们的实体被视为不是新的，我们将调用实体管理器的merge()方法。在merge()内部，JPA检查我们的实体是否存在于缓存和持久化上下文中。由于我们的对象是新的，因此在那里找不到它。最后，它会尝试从数据源加载实体。

这时我们在日志中遇到了SELECT查询，由于数据库中没有这样的元素，因此我们在此之后调用INSERT查询：

```text
Hibernate: select task0_.id as id1_1_0_, task0_.description as descript2_1_0_ from task task0_ where task0_.id=?
Hibernate: insert into task (id, description) values (default, ?)
```

在isNew()方法实现中我们可以找到以下代码：

```java
public boolean isNew(T entity) {
    ID id = this.getId(entity);
    return id == null;
}
```

**如果我们在应用程序端指定ID，我们的实体将被视为新的。在这种情况下，将向数据库发送额外的SELECT查询**。

## 4. 使用@GeneratedValue

**一个可能的解决方案是在应用程序端不指定ID**，我们可以使用[@GeneratedValue](https://www.baeldung.com/hibernate-identifiers#generated-identifiers)注解并指定在数据库端用于生成ID的策略。

让我们为TaskWithGeneratedIdID指定生成策略：

```java
@Entity
public class TaskWithGeneratedId {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
}
```

然后，我们保存TaskWithGeneratedId实体的实例，但现在我们不设置ID：

```java
@Autowired
private TaskWithGeneratedIdRepository taskWithGeneratedIdRepository;

@Test
void givenRepository_whenSaveNewTaskWithGeneratedId_thenNoExtraSelectIsExpected() {
    TaskWithGeneratedId task = new TaskWithGeneratedId();
    TaskWithGeneratedId saved = taskWithGeneratedIdRepository.saveAndFlush(task);
    assertNotNull(saved.getId());
}
```

正如我们在日志中看到的，日志中没有SELECT查询，并且为实体生成了一个新的ID。

## 5. 实现Persistable

**我们的另一个选择是在我们的实体中实现Persistable接口**：

```java
@Entity
public class PersistableTask implements Persistable<Integer> {
    @Id
    private int id;

    @Transient
    private boolean isNew = true;

    @Override
    public Integer getId() {
        return id;
    }

    @Override
    public boolean isNew() {
        return isNew;
    }

    //getters and setters
}
```

这里我们添加了一个新字段isNew，并将其标注为@Transient，以便不在基础字段中创建列。使用重写的isNew()方法，即使我们指定了ID，我们也可以认为我们的实体是新的。

现在，在底层，JPA使用另一种逻辑来考虑实体是否是新的：

```java
public class JpaPersistableEntityInformation {
    public boolean isNew(T entity) {
        return entity.isNew();
    }
}
```

让我们使用PersistableTaskRepository保存我们的PersistableTask：

```java
@Autowired
private PersistableTaskRepository persistableTaskRepository;

@Test
void givenRepository_whenSaveNewPersistableTask_thenNoExtraSelectIsExpected() {
    PersistableTask persistableTask = new PersistableTask();
    persistableTask.setId(2);
    persistableTask.setNew(true);
    PersistableTask saved = persistableTaskRepository.saveAndFlush(persistableTask);
    assertEquals(2, saved.getId());
}
```

我们可以看到，我们只有INSERT日志消息，并且实体包含我们指定的ID。

如果我们尝试保存几个具有相同ID的新实体，则会遇到异常：

```java
@Test
void givenRepository_whenSaveNewPersistableTasksWithSameId_thenExceptionIsExpected() {
    PersistableTask persistableTask = new PersistableTask();
    persistableTask.setId(3);
    persistableTask.setNew(true);
    persistableTaskRepository.saveAndFlush(persistableTask);

    PersistableTask duplicateTask = new PersistableTask();
    duplicateTask.setId(3);
    duplicateTask.setNew(true);

    assertThrows(DataIntegrityViolationException.class, () -> persistableTaskRepository.saveAndFlush(duplicateTask));
}
```

**因此，如果我们承担生成ID的责任，我们也应该注意它们的唯一性**。

## 6. 直接使用persist()方法

正如我们在前面的例子中看到的，我们所做的所有操作都导致我们调用[persist()](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)方法。**我们还可以为我们的Repository创建一个扩展，允许我们直接调用此方法**。

让我们创建一个具有persist()方法的接口：

```java
public interface TaskRepositoryExtension {
    Task persistAndFlush(Task task);
}
```

然后我们来创建这个接口的实现bean：

```java
@Component
public class TaskRepositoryExtensionImpl implements TaskRepositoryExtension {
    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public Task persistAndFlush(Task task) {
        entityManager.persist(task);
        entityManager.flush();
        return task;
    }
}
```

现在，我们使用一个新接口来扩展我们的TaskRepository：

```java
@Repository
public interface TaskRepository extends JpaRepository<Task, Integer>, TaskRepositoryExtension {
}
```

让我们调用自定义的persistAndFlush()方法来保存Task实例：

```java
@Test
void givenRepository_whenPersistNewTaskUsingCustomPersistMethod_thenNoExtraSelectIsExpected() {
    Task task = new Task();
    task.setId(4);
    Task saved = taskRepository.persistAndFlush(task);

    assertEquals(4, saved.getId());
}
```

我们可以看到带有INSERT调用且没有额外SELECT调用的日志消息。

## 7. 使用Hypersistence Utils中的BaseJpaRepository

**上一节中的想法已经在[Hypersistence Utils](https://github.com/vladmihalcea/hypersistence-utils)项目中实现**，该项目为我们提供了一个BaseJpaRepository，其中有persistAndFlush()方法实现及其批处理模拟。

要使用它，我们必须指定其[依赖](https://mvnrepository.com/artifact/io.hypersistence)，应该根据我们的Hibernate版本选择正确的Maven工件：

```xml
<dependency>
    <groupId>io.hypersistence</groupId>
    <artifactId>hypersistence-utils-hibernate-55</artifactId>
</dependency>
```

让我们实现另一个Repository，它扩展了Hypersistence Utils的BaseJpaRepository和Spring Data JPA的JpaRepository：

```java
@Repository
public interface TaskJpaRepository extends JpaRepository<Task, Integer>, BaseJpaRepository<Task, Integer> {
}
```

另外，我们必须使用@EnableJpaRepositories注解启用BaseJpaRepository的实现：

```java
@EnableJpaRepositories(
    repositoryBaseClass = BaseJpaRepositoryImpl.class
)
```

现在，让我们使用新的Repository保存我们的任务：

```java
@Autowired
private TaskJpaRepository taskJpaRepository;

@Test
void givenRepository_whenPersistNewTaskUsingPersist_thenNoExtraSelectIsExpected() {
    Task task = new Task();
    task.setId(5);
    Task saved = taskJpaRepository.persistAndFlush(task);

    assertEquals(5, saved.getId());
}
```

我们已经保存了任务，并且日志中没有SELECT查询。

**就像我们在应用程序端指定ID的所有示例一样，可能会存在唯一约束违规**：

```java
@Test
void givenRepository_whenPersistTaskWithTheSameId_thenExceptionIsExpected() {
    Task task = new Task();
    task.setId(5);
    taskJpaRepository.persistAndFlush(task);

    Task secondTask = new Task();
    secondTask.setId(5);

    assertThrows(DataIntegrityViolationException.class, () ->  taskJpaRepository.persistAndFlush(secondTask));
}
```

## 8. 使用@Query注解方法

**我们还可以通过直接修改[原生查询](https://www.baeldung.com/spring-data-jpa-query)来避免额外的调用**，让我们在TaskRepository中指定这样的方法：

```java
@Repository
public interface TaskRepository extends JpaRepository<Task, Integer> {

    @Modifying
    @Query(value = "insert into task(id, description) values(:#{#task.id}, :#{#task.description})", nativeQuery = true)
    void insert(@Param("task") Task task);
}
```

此方法直接调用INSERT查询，避免使用持久化上下文，ID将从方法参数中发送的Task对象中获取。

现在让我们使用此方法保存我们的任务：

```java
@Test
void givenRepository_whenPersistNewTaskUsingNativeQuery_thenNoExtraSelectIsExpected() {
    Task task = new Task();
    task.setId(6);
    taskRepository.insert(task);

    assertTrue(taskRepository.findById(6).isPresent());
}
```

已使用ID成功保存实体，在INSERT之前无需额外的SELECT查询。**我们应该考虑，通过使用此方法，我们可以避免JPA上下文和Hibernate缓存**。

## 9. 总结

当使用Spring Data JPA在应用程序端实现ID生成时，我们可能会在日志中遇到额外的SELECT查询，从而导致性能下降。在本文中，我们讨论了解决此问题的各种策略。

在某些情况下，将此逻辑移至数据库端或根据我们的需求微调持久化逻辑是有意义的。在做出决定之前，我们应该考虑每种策略的优缺点和潜在问题。