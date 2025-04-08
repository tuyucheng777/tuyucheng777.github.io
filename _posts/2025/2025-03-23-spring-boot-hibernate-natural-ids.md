---
layout: post
title:  Spring Boot中的Hibernate自然ID
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

一些数据库条目具有自然标识符，例如书籍的ISBN或人的SSN。除了传统的数据库ID之外，Hibernate还允许我们将某些字段声明为自然ID，并根据这些属性轻松进行查询。

在本教程中，我们将讨论@NaturalId注解，并学习在Spring Boot项目中使用和实现它。

## 2. 简单的自然ID

**我们只需使用@NaturalId注解即可将字段指定为自然标识符，这使我们能够使用Hibernate的API无缝查询关联列**。

对于本文中的代码示例，我们将使用HotelRoom和ConferenceRoom数据模型。在我们的第一个示例中，我们将实现ConferenceRoom实体，该实体可以通过其唯一的name属性来区分：

```java
@Entity
public class ConferenceRoom {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NaturalId
    private String name;

    private int capacity;

    public ConferenceRoom(String name, int capacity) {
        this.name = name;
        this.capacity = capacity;
    }

    protected ConferenceRoom() {
    }

    // getters
}
```

首先，我们需要用@NaturalId标注name字段。请注意，该字段是不可变的：它在构造函数中声明，并且不公开Setter。此外，Hibernate需要无参数构造函数，但我们可以将其设为protected并避免使用它。

**我们现在可以使用bySimpleNaturalId方法轻松地在数据库中搜索会议室，使用其名称作为自然标识符**：

```java
@Service
public class HotelRoomsService {

    private final EntityManager entityManager;

    // constructor

    public Optional<ConferenceRoom> conferenceRoom(String name) {
        Session session = entityManager.unwrap(Session.class);
        return session.bySimpleNaturalId(ConferenceRoom.class)
                .loadOptional(name);
    }
}
```

让我们运行测试并检查生成的SQL以确认预期的行为，为了查看[Hibernate/JPA SQL日志](https://www.baeldung.com/sql-logging-spring-boot)，我们将添加适当的日志配置：

```properties
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

```

现在，让我们调用conferenceRoom方法，从数据库中查询自然ID为“Colorado”的会议室：

```java
@Test
void whenWeFindBySimpleNaturalKey_thenEntityIsReturnedCorrectly() {
    conferenceRoomRepository.save(new ConferenceRoom("Colorado", 100));

    Optional<ConferenceRoom> result = service.conferenceRoom("Colorado");

    assertThat(result).isPresent()
            .hasValueSatisfying(room -> "Colorado".equals(room.getName()));
}
```

我们可以检查生成的SQL，并期望它使用其自然ID(name列)查询conference_room表：

```sql
select c1_0.id,c1_0.capacity,c1_0.name 
from conference_room c1_0 
where c1_0.name=?
```

## 3. 复合自然ID

**自然标识符也可以由多个字段组成，在这种情况下，我们可以使用@NaturalId注解标注所有相关字段**。

例如，让我们考虑GuestRoom实体，它具有由roomNumber和floor字段组成的复合自然键：

```java
@Entity
public class GuestRoom {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NaturalId
    private Integer roomNumber;

    @NaturalId
    private Integer floor;

    private String name;
    private int capacity;

    public GuestRoom(int roomNumber, int floor, String name, int capacity) {
        this.roomNumber = roomNumber;
        this.floor = floor;
        this.name = name;
        this.capacity = capacity;
    }

    protected GuestRoom() {
    }
    // getters
}
```

与第一个示例类似，我们现在将使用Hibernate的Session中的byNaturalId方法。之后，**我们将使用流式的API来指定组成复合键的字段的值**：

```java
public Optional<GuestRoom> guestRoom(int roomNumber, int floor) {
    Session session = entityManager.unwrap(Session.class);
    return session.byNaturalId(GuestRoom.class)
        .using("roomNumber", roomNumber)
        .using("floor", floor)
        .loadOptional();
}
```

现在，让我们通过尝试在数据库中查询位于三楼、编号为23的GuestRoom来测试该方法：

```java
@Test
void whenWeFindByNaturalKey_thenEntityIsReturnedCorrectly() {
    guestRoomJpaRepository.save(new GuestRoom(23, 3, "B-423", 4));

    Optional<GuestRoom> result = service.guestRoom(23, 3);

    assertThat(result).isPresent()
        .hasValueSatisfying(room -> "B-423".equals(room.getName()));
}
```

如果我们现在检查SQL，我们应该看到一个使用复合键的简单查询：

```sql
select g1_0.id,g1_0.capacity,g1_0.floor,g1_0.name,g1_0.room_number 
from guest_room g1_0 
where g1_0.floor=? 
and g1_0.room_number=?
```

## 4. 与Spring Data集成

**开箱即用，[Spring Data](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)的JpaRepository不提供通过自然标识符进行查询的支持。尽管如此，我们可以使用其他方法扩展这些接口以启用此类查询**。为此，我们必须首先声明接口：

```java
@NoRepositoryBean
public interface NaturalIdRepository<T, ID> extends JpaRepository<T, ID> {
    Optional<T> naturalId(ID naturalId);
}
```

之后，我们将创建此接口的泛型实现。此外，我们需要将泛型类型转换为域实体。为此，我们可以扩展JPA的SimpleJpaRepository，并利用其getDomainClass方法：

```java
public class NaturalIdRepositoryImpl<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements NaturalIdRepository<T, ID> {
    private final EntityManager entityManager;

    public NaturalIdRepositoryImpl(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }

    @Override
    public Optional<T> naturalId(ID naturalId) {
        return entityManager.unwrap(Session.class)
                .bySimpleNaturalId(this.getDomainClass())
                .loadOptional(naturalId);
    }
}
```

此外，我们需要添加@EnableJpaRepositories注解以允许Spring扫描整个包并注册我们的自定义Repository：

```java
@Configuration
@EnableJpaRepositories(repositoryBaseClass = NaturalIdRepositoryImpl.class)
public class NaturalIdRepositoryConfig {
}
```

这将允许我们扩展NaturalIdRepository接口来为拥有自然ID的实体创建Repository：

```java
@Repository
public interface ConferenceRoomRepository extends NaturalIdRepository<ConferenceRoom, String> {
}
```

因此，我们将能够使用丰富的Repository API并利用naturalId方法进行简单查询：

```java
@Test
void givenNaturalIdRepository_whenWeFindBySimpleNaturalKey_thenEntityIsReturnedCorrectly() {
    conferenceRoomJpaRepository.save(new ConferenceRoom("Nevada", 200));

    Optional result = conferenceRoomRepository.naturalId("Nevada");

    assertThat(result).isPresent()
            .hasValueSatisfying(room -> "Nevada".equals(room.getName()));
}
```

最后我们来检查一下生成的SQL语句：

```sql
select c1_0.id,c1_0.capacity,c1_0.name 
from conference_room c1_0 
where c1_0.name=?
```

## 5. 总结

在本文中，我们了解了具有自然标识符的实体，并发现Hibernate的API允许我们通过这些特殊标识符轻松查询。之后，我们创建了一个通用的Spring Data JPA Repository并对其进行了丰富，以利用Hibernate的这一功能。