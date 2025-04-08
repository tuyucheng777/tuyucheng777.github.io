---
layout: post
title:  在Spring WebFlux中将Mono对象转换为另一个Mono对象
category: springreactive
copyright: springreactive
excerpt: Spring Reactive
---

## 1. 简介

[Spring WebFlux](https://www.baeldung.com/spring-webflux)是一个响应式编程框架，可促进异步、非阻塞通信。使用WebFlux的一个关键方面是处理[Mono](https://www.baeldung.com/java-reactor-flux-vs-mono)对象，它代表单个异步结果。在实际应用中，我们经常需要将一个Mono对象转换为另一个，无论是为了丰富数据、处理外部服务调用还是重构负载。

在本教程中，我们将探讨如何使用Project Reactor提供的各种方法将一个Mono对象转换为另一个Mono对象。

## 2. 转换Mono对象

在探索转换Mono对象的各种方法之前，让我们先设置一下编码示例。我们将在整个教程中使用借书示例来演示不同的转换方法。为了捕捉这种情况，我们将使用三个关键类。

代表图书馆用户的用户类：

```java
public class User {
    private String userId;
    private String name;
    private String email;
    private boolean active;

    // standard setters and getters
}
```

每个用户都由一个userId唯一标识，并具有姓名和电子邮件等个人信息。此外，还有一个active标志，用于指示用户当前是否有资格借阅书籍。

Book类代表图书馆的藏书：

```java
public class Book {
    private String bookId;
    private String title;
    private double price;
    private boolean available;

    //standard setters and getters
}
```

每本书都由bookId标识，并具有title和price等属性，available标志表示该书是否可以借阅。

BookBorrowResponse类来封装借阅操作的结果：

```java
public class BookBorrowResponse {
    private String userId;
    private String bookId;
    private String status;

    //standard setters and getters
}
```

此类将流程中涉及的userId和bookId联系在一起，并提供状态字段来指示借阅是否被接受或拒绝。

## 3. 使用map()进行同步转换

[map](https://www.baeldung.com/java-reactor-map-flatmap)运算符将同步函数应用于Mono内部的数据，**它适合轻量级操作，如格式化、过滤或简单计算**。例如，如果我们想从Mono用户获取电子邮件地址，我们可以使用map进行转换：

```java
@Test
void givenUserId_whenTransformWithMap_thenGetEmail() {
    String userId = "U001";
    Mono<User> userMono = Mono.just(new User(userId, "John", "john@example.com"));
    Mockito.when(userService.getUser(userId))
            .thenReturn(userMono);

    Mono<String> userEmail = userService.getUser(userId)
            .map(User::getEmail);

    StepVerifier.create(userEmail)
            .expectNext("john@example.com")
            .verifyComplete();
}
```

## 4. 使用flatMap()进行异步转换

[flatMap()](https://www.baeldung.com/java-reactor-map-flatmap)方法将每个发出的元素从Mono转换为另一个Publisher，**当转换需要新的异步过程(例如进行另一个API调用或查询数据库)时，此方法特别有用**。当转换结果为Mono时，flatMap()会将结果展平为单个序列。

让我们来看看我们的图书借阅系统。当用户请求借阅一本书时，系统会验证用户的会员状态，然后检查这本书是否可用。如果两项检查都通过，系统将处理借阅请求并返回BookBorrowResponse：

```java
public Mono<BookBorrowResponse> borrowBook(String userId, String bookId) {
    return userService.getUser(userId)
            .flatMap(user -> {
                if (!user.isActive()) {
                    return Mono.error(new RuntimeException("User is not an active member"));
                }
                return bookService.getBook(bookId);
            })
            .flatMap(book -> {
                if (!book.isAvailable()) {
                    return Mono.error(new RuntimeException("Book is not available"));
                }
                return Mono.just(new BookBorrowResponse(userId, bookId, "Accepted"));
            });
}
```

在此示例中，检索用户和图书详细信息等操作是异步的，并返回Mono对象。**使用flatMap()，我们可以以可读且合乎逻辑的方式链接这些操作，而无需嵌套多个Mono层级**。序列中的每个步骤都取决于上一步的结果。例如，仅当用户处于激活状态时才会检查图书可用性。flatMap()确保我们可以动态地做出这些决定，同时保持流程的响应性。

## 5. 使用transform()方法的可重用逻辑

transform()方法是一种多功能工具，可让我们封装可重用逻辑。**我们无需在应用程序的多个部分重复转换，只需定义一次即可在需要时应用它们**。这提高了代码的可重用性、关注点分离和可读性。

让我们看一个例子，我们需要返回一本书在应用税款和折扣后的最终价格：

```java
public Mono<Book> applyDiscount(Mono<Book> bookMono) {
    return bookMono.map(book -> {
        book.setPrice(book.getPrice() - book.getPrice() * 0.2);
        return book;
    });
}

public Mono<Book> applyTax(Mono<Book> bookMono) {
    return bookMono.map(book -> {
        book.setPrice(book.getPrice() + book.getPrice() * 0.1);
        return book;
    });
}

public Mono<Book> getFinalPricedBook(String bookId) {
    return bookService.getBook(bookId)
            .transform(this::applyTax)
            .transform(this::applyDiscount);
}
```

在此示例中，applyDiscount()方法应用20%的折扣，applyTax()方法应用10%的税。transform方法在管道中应用这两种方法，并返回Book的Mono和最终价格。

## 6. 合并来自多个来源的数据

[zip()](https://www.baeldung.com/reactor-combine-streams)方法组合多个Mono对象并生成单个结果，**它不会同时合并结果，而是等待所有Mono对象发出后再应用组合器函数**。

让我们重申一下借书的例子，我们获取用户信息和书籍信息来创建BookBorrowResponse：

```java
public Mono<BookBorrowResponse> borrowBookZip(String userId, String bookId) {
    Mono userMono = userService.getUser(userId)
            .switchIfEmpty(Mono.error(new RuntimeException("User not found")));
    Mono bookMono = bookService.getBook(bookId)
            .switchIfEmpty(Mono.error(new RuntimeException("Book not found")));
    return Mono.zip(userMono, bookMono,
            (user, book) -> new BookBorrowResponse(userId, bookId, "Accepted"));
}
```

在此实现中，zip()方法确保在创建响应之前用户和书籍信息可用。如果用户或书籍检索失败(例如，如果用户不存在或书籍不可用)，则错误将传播，并且组合的Mono将以适当的错误信号终止。

## 7. 条件变换

通过结合filter()和[switchIfEmpty()](https://www.baeldung.com/spring-reactive-switchifempty)方法，我们可以应用条件逻辑根据谓词转换Mono对象。**如果谓词为真，则返回原始Mono；如果谓词为假，则Mono切换到switchIfEmpty()提供的另一个Mono，反之亦然**。

让我们考虑这样一种场景：只有当用户活跃时我们才想要应用折扣，否则不返回折扣：

```java
public Mono<Book> conditionalDiscount(String userId, String bookId) {
    return userService.getUser(userId)
            .filter(User::isActive)
            .flatMap(user -> bookService.getBook(bookId).transform(this::applyDiscount))
            .switchIfEmpty(bookService.getBook(bookId))
            .switchIfEmpty(Mono.error(new RuntimeException("Book not found")));
}
```

在此示例中，我们使用userId获取User的Mono。filter方法检查用户是否处于活动状态，如果用户处于活动状态，我们将在应用折扣后返回Book的Mono。如果用户不活跃，Mono将变为空，switchIfEmpty()方法将启动以获取书籍而不应用折扣。最后，如果书籍本身不存在，另一个switchIfEmpty()确保传播适当的错误，使整个流程变得强大且直观。

## 8. 转换过程中的错误处理

错误处理可确保转换的弹性，从而允许优雅的回退机制或替代数据源。当转换失败时，适当的错误处理有助于优雅地恢复、记录问题或返回替代数据。

**[onErrorResume()](https://www.baeldung.com/spring-webflux-errors)方法用于通过提供替代Mono从错误中恢复。当我们想要提供默认数据或从替代源获取数据时，这特别有用**。

让我们重新回顾一下我们的借书示例；如果在获取User或Book对象时抛出任何错误，我们通过返回具有“Rejected”状态的BookBorrowResponse对象来正常处理失败：

```java
public Mono<BookBorrowResponse> handleErrorBookBorrow(String userId, String bookId) {
    return borrowBook(userId, bookId)
        .onErrorResume(ex -> Mono.just(new BookBorrowResponse(userId, bookId, "Rejected")));
}
```

**这种错误处理策略可确保即使在故障情况下，系统也能做出可预测的响应并保持无缝的用户体验**。

## 9. 转换Mono对象的最佳实践

转换Mono对象时，遵循一些最佳实践至关重要，以确保我们的响应式管道干净、高效且易于维护。当我们需要简单的同步转换(如丰富或修改数据)时，map()方法是完美的选择，而flatMap()则是涉及异步工作流的任务的理想选择，例如调用外部API或查询数据库。为了保持管道干净且可重用，我们使用transform()方法封装逻辑，促进模块化和关注点分离。为了保持可读性，我们应该更喜欢链接而不是嵌套操作。

错误处理在确保弹性方面起着关键作用，通过使用onErrorResume()等方法，我们可以提供后备响应或替代数据源，从而妥善管理错误。最后，在每个阶段验证输入和输出有助于防止问题向下游传播，从而确保管道稳健且可扩展。

## 10. 总结

在本教程中，我们学习了将一个Mono对象转换为另一个Mono对象的各种方法。了解适合该任务的正确运算符至关重要，无论是map()、flatMap()还是transform()。利用这些技术并应用最佳实践，我们可以在Spring WebFlux中构建灵活且可维护的响应式管道。