---
layout: post
title:  在Spring Boot中禁用@Cacheable
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 简介

缓存是一种有效的策略，当执行结果在已知时间内没有改变(实际上相同)时，它可以避免重复执行逻辑，从而提高性能。

**Spring Boot提供了[@Cacheable](https://www.baeldung.com/spring-cache-tutorial)注解，我们在方法上定义它，它会缓存方法的结果。在某些情况下，比如在较低的环境中测试时，我们可能需要禁用缓存以观察某些修改后的行为**。

在本文中，我们将在Spring Boot中配置缓存，并学习如何在需要时禁用缓存。

## 2. 缓存设置

让我们设置一个通过ISBN查询书评的简单用例，并使用@Cacheable在某些逻辑中缓存该方法。

我们的实体类将是BookReview类，它包含ratings、isbn等等：

```java
@Entity
@Table(name="BOOK_REVIEWS")
public class BookReview {
    @Id
    @GeneratedValue(strategy= GenerationType.SEQUENCE, generator = "book_reviews_reviews_id_seq")
    @SequenceGenerator(name = "book_reviews_reviews_id_seq", sequenceName = "book_reviews_reviews_id_seq", allocationSize = 1)
    private Long reviewsId;
    private String userId;
    private String isbn;
    private String bookRating;
   
    // getters & setters
}
```

我们在BookRepository中添加一个简单的findByIsbn()方法来通过isbn查询书评：

```java
public interface BookRepository extends JpaRepository<BookReview, Long> {
    List<BookReview> findByIsbn(String isbn);
}
```

BookReviewsLogic类包含一个在BookRepository中调用findByIsbn()的方法，我们添加了@Cacheable注解，它将给定isbn的结果缓存在book_reviews缓存中：

```java
@Service
public class BookReviewsLogic {
    @Autowired
    private BookRepository bookRepository;

    @Cacheable(value = "book_reviews", key = "#isbn")
    public List<BookReview> getBooksByIsbn(String isbn){
        return bookRepository.findByIsbn(isbn);
    }
}
```

**由于我们在逻辑类中使用了@Cacheable，因此我们需要配置缓存。我们可以通过带有@Configuration和@EnableCaching的注解配置类来设置缓存配置**。在这里，我们返回HashMap作为缓存存储：

```java
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
}
```

我们已经准备好缓存设置。如果我们在BookReviewsLogic中执行getBooksByIsbn()，我们的结果将在第一次执行时被缓存，并且从那时起立即返回，而无需重新计算(即查询数据库)，从而提高性能。

我们来写一个简单的测试来验证一下：

```java
@Test
public void givenCacheEnabled_whenLogicExecuted2ndTime_thenItDoesntQueriesDB(CapturedOutput output){
    BookReview bookReview = insertBookReview();

    String target = "Hibernate: select bookreview0_.reviews_id as reviews_1_0_, "
            + "bookreview0_.book_rating as book_rat2_0_, "
            + "bookreview0_.isbn as isbn3_0_, "
            + "bookreview0_.user_id as user_id4_0_ "
            + "from book_reviews bookreview0_ "
            + "where bookreview0_.isbn=?";

    // 1st execution
    bookReviewsLogic.getBooksByIsbn(bookReview.getIsbn());
    String[] logs = output.toString()
            .split("\\r?\\n");
    assertThat(logs).anyMatch(e -> e.contains(target));

    // 2nd execution
    bookReviewsLogic.getBooksByIsbn(bookReview.getIsbn());
    logs = output.toString()
            .split("\\r?\\n");

    long count = Arrays.stream(logs)
            .filter(e -> e.equals(target))
            .count();

    // count 1 means the select query log from 1st execution.
    assertEquals(1,count);
}
```

在上面的测试中，我们执行了两次getBooksByIsbn()，捕获日志，并确认select查询只执行了一次，因为getBooksByIsbn()方法在第二次执行时返回了缓存结果。

要为针对数据库执行的查询生成SQL日志，我们可以在application.properties文件中设置以下属性：

```properties
spring.jpa.show-sql=true
```

## 3. 禁用缓存

**为了禁用缓存，我们将在application.properties文件中使用额外自定义属性(即appconfig.cache.enabled)**：

```properties
appconfig.cache.enabled=true
```

之后，我们可以在缓存配置文件中读取此配置并进行条件检查：

```java
@Bean
public CacheManager cacheManager(@Value("${appconfig.cache.enabled}") String isCacheEnabled) {
    if (isCacheEnabled.equalsIgnoreCase("false")) {
        return new NoOpCacheManager();
    }

    return new ConcurrentMapCacheManager();
}
```

如上所示，我们的逻辑检查属性是否设置为禁用缓存。如果是，我们**可以返回NoOpCacheManager的实例，这是一个不执行缓存的缓存管理器**。否则，我们可以返回基于哈希的缓存管理器。

通过上述简单的设置，我们可以在Spring Boot应用中禁用缓存。让我们通过一个简单的测试来验证上述设置。

首先，我们需要修改在application.properties中定义的缓存属性。对于我们的测试设置，我们可以使用[@TestPropertySource](https://www.baeldung.com/spring-test-property-source)覆盖该属性：

```java
@SpringBootTest(classes = BookReviewApplication.class)
@ExtendWith(OutputCaptureExtension.class)
@TestPropertySource(properties = {
        "appconfig.cache.enabled=false"
})
public class BookReviewsLogicCacheDisabledUnitTest {
    // ...
}
```

现在，我们的测试与之前的类似，我们执行了两次逻辑。我们检查SQL查询日志，看它是否在当前测试中记录了两次，因为执行不会被缓存：

```java
long count = Arrays.stream(logs)
   .filter(e -> e.contains(target))
   .count();

// count 2 means the select query log from 1st and 2nd execution.
assertEquals(2, count);
```

## 4. 总结

在本教程中，我们简要介绍了Spring Boot中的缓存，然后在应用程序中设置了缓存。我们还学习了当需要测试代码的某些部分时如何禁用缓存。此外，我们编写了必要的测试来验证启用和禁用缓存的工作原理。