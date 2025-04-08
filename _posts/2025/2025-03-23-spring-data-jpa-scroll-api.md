---
layout: post
title:  Spring Data JPA中的Scroll API
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

[Spring Data Commons](https://docs.spring.io/spring-data/commons/docs/current/reference/html/)是[Spring Data](https://spring.io/projects/spring-data)总项目的一部分，包含用于管理持久层的接口和实现。Scroll API是Spring Data Commons提供的功能之一，用于处理从数据库读取的大量结果。

在本教程中，我们将通过示例探索Scroll API。

## 2. 依赖

Spring Boot 3.1版本添加了Scroll API支持，Spring Data Commons已包含在[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)中。因此，添加Spring Data JPA 3.1版本就足以获得Scroll API功能：

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>3.1.0</version>
</dependency>
```

最新的库版本可以在[Maven Central](https://mvnrepository.com/artifact/org.springframework.data/spring-data-jpa)中找到。

## 3. 实体类

为了举例，我们将使用BookReview实体，其中包含不同用户对各种书籍的书评评级：

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

    // getters and setters
}
```

## 4. Scroll API

**Scroll API提供分块迭代大量结果的功能，它提供稳定排序、滚动类型和结果限制**。

我们可以使用属性名称定义简单的排序表达式，并通过查询派生使用Top或First定义静态结果限制。

### 4.1 使用偏移过滤进行滚动

在下面的例子中，我们使用[查询派生](https://www.baeldung.com/spring-data-derived-queries)通过评级参数和OffsetScrollPosition找到前五本书：

```java
public interface BookRepository extends Repository<BookReview, Long> {
    Window<BookReview> findFirst5ByBookRating(String bookRating, OffsetScrollPosition position);
    Window<BookReview> findFirst10ByBookRating(String bookRating, OffsetScrollPosition position);
    Window<BookReview> findFirst3ByBookRating(String bookRating, KeysetScrollPosition position);
}
```

由于我们已经定义了Repository方法，我们可以在逻辑类中使用它们来获取前五本书并不断迭代直到获得最后的结果。

在迭代时，我们需要通过查询来检查下一个窗口是否存在：

```java
public List<BookReview> getBooksUsingOffset(String rating) {
    OffsetScrollPosition offset = ScrollPosition.offset();

    Window<BookReview> bookReviews = bookRepository.findFirst5ByBookRating(rating, offset);
    List<BookReview> bookReviewsResult = new ArrayList<>();
    do {
        bookReviews.forEach(bookReviewsResult::add);
        bookReviews = bookRepository.findFirst5ByBookRating(rating, (OffsetScrollPosition) bookReviews.positionAt(bookReviews.size() - 1));
    } while (!bookReviews.isEmpty() && bookReviews.hasNext());

    return bookReviewsResult;
}
```

**我们可以通过使用WindowIterator来简化我们的逻辑**，它提供了滚动浏览大结果的实用程序，而无需检查下一个窗口和ScrollPosition：

```java
public List<BookReview> getBooksUsingOffSetFilteringAndWindowIterator(String rating) {
    WindowIterator<BookReview> bookReviews = WindowIterator.of(position -> bookRepository
            .findFirst5ByBookRating("3.5", (OffsetScrollPosition) position)).startingAt(ScrollPosition.offset());
    List<BookReview> bookReviewsResult = new ArrayList<>();
    bookReviews.forEachRemaining(bookReviewsResult::add);

    return bookReviewsResult;
}
```

**偏移滚动的工作原理类似于分页，它通过从大量结果中跳过一定数量的记录来返回预期结果**。虽然我们只能看到请求结果的一部分，但服务器需要构建完整结果，这会导致额外的负载。

我们可以使用键集过滤来避免这种行为。

### 4.2 使用键集过滤进行滚动

键集过滤有助于使用数据库的内置功能检索结果子集，旨在**减少单个查询的计算和IO要求**。

数据库只需要从给定的键集位置构建较小的结果，而无需实现大的完整结果：

```java
public List<BookReview> getBooksUsingKeySetFiltering(String rating) {
    WindowIterator<BookReview> bookReviews = WindowIterator.of(position -> bookRepository
                    .findFirst5ByBookRating(rating, (KeysetScrollPosition) position))
            .startingAt(ScrollPosition.keyset());
    List<BookReview> bookReviewsResult = new ArrayList<>();
    bookReviews.forEachRemaining(bookReviewsResult::add);

    return bookReviewsResult;
}
```

## 5. 总结

在本文中，我们探索了Spring Data Commons库提供的Scroll API，Scroll API支持根据偏移位置和过滤条件以较小的块读取大量结果。

**Scroll API支持使用偏移量和键集进行过滤**，基于偏移量的过滤需要在数据库中实现整个结果，而键集则通过构建较小的结果来帮助减少数据库的计算和IO负载。