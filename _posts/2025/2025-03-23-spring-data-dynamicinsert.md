---
layout: post
title:  Spring Data JPA中的@DynamicInsert指南
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

**[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)中的@DynamicInsert注解通过在SQL语句中仅包含非空字段来优化插入操作**，此过程加快了生成的查询速度，减少了不必要的数据库交互。虽然它可以提高具有许多可空字段的实体的效率，但它会带来一些运行时开销。因此，在排除空列的好处大于性能成本的情况下，我们应该有选择地使用它。

## 2. JPA中的默认插入行为

使用[EntityManager](https://www.baeldung.com/hibernate-entitymanager)或Spring Data JPA的save()方法持久化JPA实体时，Hibernate会生成一条SQL插入语句。此语句包括每个实体列，即使某些列包含空值。因此，在处理包含许多可选字段的大型实体时，插入操作效率可能很低。

现在让我们看看这个保存操作，我们首先考虑一个简单的Account实体：

```java
@Entity
public class Account {
    @Id
    private int id;

    @Column
    private String name;

    @Column
    private String type;

    @Column
    private boolean active;

    @Column
    private String description;

    // getters, setters, constructors
}
```

我们还将为Account实体创建一个JPA Repository：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {}
```

在这种情况下，当我们保存一个新的Account对象时：

```java
Account account = new Account();
account.setId(ACCOUNT_ID);
account.setName("account1");
account.setActive(true);
accountRepository.save(account);
```

Hibernate将生成包含所有列的SQL插入语句，即使只有少数列具有非空值：

```sql
insert into Account (active,description,name,type,id) values (?,?,?,?,?)
```

这种行为并不总是最佳的，特别是对于某些字段可能为空或仅稍后初始化的大型实体。

## 3. 使用@DynamicInsert

我们可以在实体级别使用[@DynamicInsert](https://docs.jboss.org/hibernate/orm/6.5/javadocs/org/hibernate/annotations/DynamicInsert.html)注解来优化此插入行为。**应用后，Hibernate将生成仅包含非空值列的SQL插入语句，从而避免SQL查询中出现不必要的列**。

让我们将@DynamicInsert注解添加到Account实体：

```java
@Entity
@DynamicInsert
public class Account {
    @Id
    private int id;

    @Column
    private String name;

    @Column
    private String type;

    @Column
    private boolean active;

    @Column
    private String description;

    // getters, setters, constructors
}
```

现在，当我们保存一个新的Account实体并且其中某些列设置为null时，Hibernate将生成一个优化的SQL语句：

```java
Account account = new Account();
account.setId(ACCOUNT_ID);
account.setName("account1");
account.setActive(true);
accountRepository.save(account);
```

生成的SQL将仅包含非空列：

```sql
insert into Account (active,name,id) values (?,?,?)
```

## 4. @DynamicInsert的工作原理

在Hibernate级别，@DynamicInsert影响框架生成和执行SQL插入语句的方式。**默认情况下，Hibernate会预先生成并缓存包含每个映射列的静态SQL插入语句，即使在持久化实体时某些列包含空值**。这是Hibernate性能优化的一部分，因为它可以重用预编译的SQL语句，而无需每次都重新生成它们。

但是，当我们将@DynamicInsert注解应用于实体时，Hibernate会改变此行为并在运行时动态生成插入SQL语句。

## 5. 何时使用@DynamicInsert

**@DynamicInsert注解是Hibernate中的一项强大功能，但应根据特定场景选择性地使用它**。其中一种场景涉及具有许多可空字段的实体，当实体具有可能不总是填充的字段时，@DynamicInsert会通过从SQL查询中排除未设置的列来优化插入操作，这减少了生成的查询的大小并提高了性能。

当某些数据库列具有默认值时，此注解也很有用，防止Hibernate插入空值允许数据库使用其默认设置处理这些字段。例如，created_at列可能会在插入记录时自动设置当前时间戳。通过在插入语句中排除此字段，Hibernate可以保留数据库的逻辑并防止覆盖默认行为。

**此外，@DynamicInsert在插入性能至关重要的情况下也很有用**，此注解可确保仅将具有许多字段(可能并非全部填充)的相关数据发送到数据库。这在高性能系统中特别有用，因为最小化SQL语句的大小会显著影响效率。

## 6. 何时不使用@DynamicInsert

尽管如我们所见，此注解有很多好处，但在某些情况下它并不是最佳选择。**我们不应该使用@DynamicInsert的最具体情况是我们的实体大多具有非空值的情况**，在这种情况下，动态SQL生成可能会增加不必要的复杂性，而不会提供显著的效果。在这种情况下，由于大多数字段是在插入操作期间填充的，因此@DynamicInsert提供的优化变得多余。

此外，不仅具有许多非空字段的表不适合使用@DynamicInsert，而且具有少量属性的表也不适合使用@DynamicInsert。**对于简单实体或只有少量字段的小表，使用@DynamicInsert的优势很小，排除空值带来的性能提升不太可能很明显**。

在涉及批量插入的场景中，@DynamicInsert的动态特性可能会导致效率低下。**由于Hibernate为每个实体重新生成SQL，而不是重复使用预编译查询，因此批量插入的执行效率可能不如静态SQL插入**。

在某些情况下，@DynamicInsert可能无法很好地适应复杂的数据库配置或模式，尤其是在涉及复杂的约束或触发器时。例如，如果模式具有要求某些列具有特定值的约束，则@DynamicInsert可能会忽略这些列(如果它们为null)，从而导致约束违规或错误。

假设我们的Account实体带有一个数据库触发器，如果文章类型为空，则插入“UNKNOWN”：

```sql
CREATE TRIGGER `account_type` BEFORE INSERT ON `account` FOR EACH ROW BEGIN
    IF NEW.type IS NULL THEN
        SET NEW.type = 'UNKNOWN';
    END IF;
END
```

如果没有提供类型，Hibernate将完全排除该列。因此，触发器可能无法激活。

## 7. 总结

在本文中，我们探讨了@DynamicInsert注解，并通过代码示例了解了它的实际应用。我们研究了Hibernate如何动态生成仅包含非空列的SQL插入语句，通过避免SQL查询中不必要的数据来优化性能。

我们还讨论了其优点，例如提高了具有许多可空字段的实体的效率，并更好地尊重数据库默认值。但是，我们强调了它的局限性，包括插入故意的空值时的潜在开销和复杂性。通过了解这些方面，我们可以做出明智的决定，决定何时以及如何在我们的应用程序中使用@DynamicInsert。