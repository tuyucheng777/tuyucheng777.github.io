---
layout: post
title:  在Spring Data JPA中实现更新或插入
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 简介

在应用程序开发中，执行更新或插入操作(也称为“upsert”)的需求非常普遍，此操作涉及将新记录放入数据库表中(如果不存在)或更新现有记录(如果存在)。

在本教程中，我们将学习使用[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)执行更新或插入操作的不同方法。

## 2. 设置

为了演示目的，我们将使用CreditCard实体：

```java
@Entity
@Table(name="credit_card")
public class CreditCard {
    @Id
    @GeneratedValue(strategy= GenerationType.SEQUENCE, generator = "credit_card_id_seq")
    @SequenceGenerator(name = "credit_card_id_seq", sequenceName = "credit_card_id_seq", allocationSize = 1)
    private Long id;
    private String cardNumber;
    private String expiryDate;

    private Long customerId;

    // getters and setters
}
```

## 3. 实现

我们将使用三种不同的方法实现更新或插入。

### 3.1 使用Repository方法

**在这种方法中，我们将使用从[CrudRepository](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)接口继承的[save(entity)](https://www.baeldung.com/spring-data-crud-repository-save)方法在Repository中编写一个事务默认方法。save(entity)方法将插入新记录，或者根据ID更新现有实体**：

```java
public interface CreditCardRepository extends JpaRepository<CreditCard,Long> {
    @Transactional
    default CreditCard updateOrInsert(CreditCard entity) {
        return save(entity);
    }
}
```

我们将creditCard传递给CreditCardLogic类内的updateOrInsertUsingRepository()方法，该方法根据实体id插入或更新实体：

```java
@Service
public class CreditCardLogic {
    @Autowired
    private CreditCardRepository creditCardRepository;
   
    public void updateOrInsertUsingRepository(CreditCard creditCard) {
        creditCardRepository.updateOrInsert(creditCard);
    }
}
```

这种方法的一个重要注意事项是，**实体是否要更新由id决定**。如果我们需要根据另一列(例如cardNumber而不是id)查找现有记录，则此方法将不起作用。在这种情况下，我们可以使用后面部分讨论的方法。

我们可以编写单元测试来验证我们的逻辑，首先，让我们将一些测试数据保存到credit_card表中：

```java
private CreditCard createAndReturnCreditCards() {
    CreditCard card = new CreditCard();
    card.setCardNumber("3494323432112222");
    card.setExpiryDate("2024-06-21");
    card.setCustomerId(10L);
    return creditCardRepository.save(card);
}
```

我们将使用上面保存的信用卡进行更新，让我们构建一个信用卡对象用于插入：

```java
private CreditCard buildCreditCard() {
    CreditCard card = new CreditCard();
    card.setCardNumber("9994323432112222");
    card.setExpiryDate("2024-06-21");
    card.setCustomerId(10L);

    return card;
}
```

我们已准备好编写单元测试：

```java
@Test
void givenCreditCards_whenUpdateOrInsertUsingRepositoryExecuted_thenUpserted() {
    // insert test
    CreditCard newCreditCard = buildCreditCard();
    CreditCard existingCardByCardNumber = creditCardRepository.findByCardNumber(newCreditCard.getCardNumber());
    assertNull(existingCardByCardNumber);

    creditCardLogic.updateOrInsertUsingRepository(newCreditCard);

    existingCardByCardNumber = creditCardRepository.findByCardNumber(newCreditCard.getCardNumber());
    assertNotNull(existingCardByCardNumber);

    // update test
    CreditCard cardForUpdate = existingCard;
    String beforeExpiryDate = cardForUpdate.getExpiryDate();
    cardForUpdate.setExpiryDate("2029-08-29");
    existingCardByCardNumber = creditCardRepository.findByCardNumber(cardForUpdate.getCardNumber());
    assertNotNull(existingCardByCardNumber);

    creditCardLogic.updateOrInsertUsingRepository(cardForUpdate);

    assertNotEquals("2029-08-29", beforeExpiryDate);
    CreditCard updatedCard = creditCardRepository.findById(cardForUpdate.getId()).get();
    assertEquals("2029-08-29", updatedCard.getExpiryDate());
}
```

在上面的测试中，我们断言updateOrInsertUsingRepository()方法的插入和更新操作。

### 3.2 使用自定义逻辑

**在这种方法中，我们在CreditCardLogic类中编写自定义逻辑，该类首先检查表中给定的行是否已经存在，然后根据输出决定插入或更新记录**：

```java
public void updateOrInsertUsingCustomLogic(CreditCard creditCard) {
    CreditCard existingCard = creditCardRepository.findByCardNumber(creditCard.getCardNumber());
    if (existingCard != null) {
        existingCard.setExpiryDate(creditCard.getExpiryDate());
        creditCardRepository.save(creditCard);
    } else {
        creditCardRepository.save(creditCard);
    }
}
```

根据上述逻辑，如果cardNumber已存在于数据库中，则我们根据传递的CreditCard对象更新该现有实体。否则，我们将传递的CreditCard作为新实体插入到updateOrInsertUsingCustomLogic()方法中。

我们可以编写单元测试来验证我们的自定义逻辑：

```java
@Test
void givenCreditCards_whenUpdateOrInsertUsingCustomLogicExecuted_thenUpserted() {
    // insert test
    CreditCard newCreditCard = buildCreditCard();
    CreditCard existingCardByCardNumber = creditCardRepository.findByCardNumber(newCreditCard.getCardNumber());
    assertNull(existingCardByCardNumber);

    creditCardLogic.updateOrInsertUsingCustomLogic(newCreditCard);

    existingCardByCardNumber = creditCardRepository.findByCardNumber(newCreditCard.getCardNumber());
    assertNotNull(existingCardByCardNumber);

    // update test
    CreditCard cardForUpdate = existingCard;
    String beforeExpiryDate = cardForUpdate.getExpiryDate();
    cardForUpdate.setExpiryDate("2029-08-29");

    creditCardLogic.updateOrInsertUsingCustomLogic(cardForUpdate);

    assertNotEquals("2029-08-29", beforeExpiryDate);
    CreditCard updatedCard = creditCardRepository.findById(cardForUpdate.getId()).get();
    assertEquals("2029-08-29", updatedCard.getExpiryDate());
}
```

### 3.3 使用数据库内置功能

**许多数据库都提供了内置功能来处理插入时冲突。例如，PostgreSQL提供“ON CONFLICT DO UPDATE”，MySQL提供“ON DUPLICATE KEY”。使用此功能**，我们可以在将记录插入数据库时出现重复键时编写后续更新语句。

示例查询如下：

```java
String updateOrInsert = """
        INSERT INTO credit_card (card_number, expiry_date, customer_id)
        VALUES( :card_number, :expiry_date, :customer_id )
        ON CONFLICT ( card_number )
        DO UPDATE SET
        card_number = :card_number,
        expiry_date = :expiry_date,
        customer_id = :customer_id
        """;
```

为了进行测试，我们使用H2数据库，它不提供“ON CONFLICT”功能，但我们可以使用H2数据库提供的merge查询。让我们在CreditCardLogic类中添加合并逻辑：

```java
@Transactional
public void updateOrInsertUsingBuiltInFeature(CreditCard creditCard) {
    Long id = creditCard.getId();
    if (creditCard.getId() == null) {
        BigInteger nextVal = (BigInteger) em.createNativeQuery("SELECT nextval('credit_card_id_seq')").getSingleResult();
        id = nextVal.longValue();
    }

    String upsertQuery = """
            MERGE INTO credit_card (id, card_number, expiry_date, customer_id)
            KEY(card_number)
            VALUES (?, ?, ?, ?)
            """;

    Query query = em.createNativeQuery(upsertQuery);
    query.setParameter(1, id);
    query.setParameter(2, creditCard.getCardNumber());
    query.setParameter(3, creditCard.getExpiryDate());
    query.setParameter(4, creditCard.getCustomerId());

    query.executeUpdate();
}
```

在上面的逻辑中，我们使用[EntityManager](https://www.baeldung.com/hibernate-entitymanager)提供的原生查询执行merge查询。

现在我们来编写单元测试来验证结果：

```java
@Test
void givenCreditCards_whenUpdateOrInsertUsingBuiltInFeatureExecuted_thenUpserted() {
    // insert test
    CreditCard newCreditCard = buildCreditCard();
    CreditCard existingCardByCardNumber = creditCardRepository.findByCardNumber(newCreditCard.getCardNumber());
    assertNull(existingCardByCardNumber);

    creditCardLogic.updateOrInsertUsingBuiltInFeature(newCreditCard);

    existingCardByCardNumber = creditCardRepository.findByCardNumber(newCreditCard.getCardNumber());
    assertNotNull(existingCardByCardNumber);

    // update test
    CreditCard cardForUpdate = existingCard;
    String beforeExpiryDate = cardForUpdate.getExpiryDate();
    cardForUpdate.setExpiryDate("2029-08-29");

    creditCardLogic.updateOrInsertUsingBuiltInFeature(cardForUpdate);

    assertNotEquals("2029-08-29", beforeExpiryDate);
    CreditCard updatedCard = creditCardRepository.findById(cardForUpdate.getId()).get();
    assertEquals("2029-08-29", updatedCard.getExpiryDate());
}
```

## 4. 总结

在本文中，我们讨论了在Spring Data JPA中执行更新或插入操作的不同方法，我们实现了这些方法并使用单元测试进行了验证。虽然每个数据库都提供了一些用于处理upsert的开箱即用功能，但在Spring Data JPA中基于id列在upsert之上实现自定义逻辑并不复杂。