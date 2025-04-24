---
layout: post
title:  Spring WebFlux Reactive Flow中的条件语句
category: springreactive
copyright: springreactive
excerpt: Spring WebFlux
---

## 1. 概述

在[Spring WebFlux](https://www.baeldung.com/spring-webflux)响应流中使用条件语句允许在处理响应流时进行动态决策，与命令式方法不同，响应式方法中的条件逻辑不仅限于if-else语句。相反，我们可以使用各种运算符(例如map()、filter()、switchIfEmpty()等)来引入条件流而不阻塞流。

在本文中，**我们将探讨在Spring WebFlux中使用条件语句的不同方法**。除非明确说明，否则每种方法都适用于Mono和Flux。

## 2. 使用map()的条件构造

我们可以使用map()运算符来转换流中的各个元素，此外，我们**可以在映射器中使用if-else语句来有条件地修改元素**。

让我们定义一个名为oddEvenFlux的Flux，并使用map()运算符将其每个元素标记为“Even”或“Odd”：
```java
Flux<String> oddEvenFlux = Flux.just(1, 2, 3, 4, 5, 6)
        .map(num -> {
            if (num % 2 == 0) {
                return "Even";
            } else {
                return "Odd";
            }
        });
```

我们应该注意到**map()是同步的，它会在发出一个元素后立即应用转换函数**。

接下来，让我们使用[StepVerifier](https://www.baeldung.com/reactive-streams-step-verifier-test-publisher#stepverifier)来测试我们的响应流的行为并确认每个元素的条件标签：
```java
StepVerifier.create(oddEvenFlux)
    .expectNext("Odd")
    .expectNext("Even")
    .expectNext("Odd")
    .expectNext("Even")
    .expectNext("Odd")
    .expectNext("Even")
    .verifyComplete();
```

正如所料，每个数字都根据其[奇偶性](https://en.wikipedia.org/wiki/Parity_(mathematics))进行标记。

## 3. 使用filter()

我们可以**使用filter()运算符通过[谓词](https://www.baeldung.com/cs/predicates)过滤掉数据**，确保下游运算符只接收相关数据。

让我们从数字流中创建一个名为evenNumbersFlux的新Flux：
```java
Flux<Integer> evenNumbersFlux = Flux.just(1, 2, 3, 4, 5, 6)
    .filter(num -> num % 2 == 0);
```

在这里，我们为filter()运算符添加了一个谓词来确定数字是否为偶数。

现在，让我们验证evenNumbersFlux是否只允许偶数向下游传递：
```java
StepVerifier.create(evenNumbersFlux)
    .expectNext(2)
    .expectNext(4)
    .expectNext(6)
    .verifyComplete();
```

## 4. 使用switchIfEmpty()和defaultIfEmpty()

在本节中，我们将学习两个有用的运算符，当底层Flux不发出任何元素时，它们可以启用条件数据流。

### 4.1 使用switchIfEmpty()

当底层Flux没有发布任何数据时，我们可能需要切换到另一个流，在这种情况下，我们**可以通过[switchIfEmpty()](https://www.baeldung.com/spring-reactive-switchifempty)运算符提供一个替代的发布者**。

假设我们有一个用filter()运算符链接的单词流，该运算符只允许长度为两个或更多字符的单词：
```java
Flux<String> flux = Flux.just("A", "B", "C", "D", "E")
    .filter(word -> word.length() >= 2);
```

当然，当没有任何单词符合过滤条件时，Flux将不会发出任何元素。

现在，让我们通过switchIfEmpty()运算符提供一个替代Flux：
```java
flux = flux.switchIfEmpty(Flux.defer(() -> Flux.just("AA", "BB", "CC")));
```

我们使用了Flux.defer()方法来确保仅当上游Flux不产生任何元素时才创建替代Flux。

最后，让我们验证一下结果Flux是否从替代源产生所有元素：
```java
StepVerifier.create(flux)
    .expectNext("AA")
    .expectNext("BB")
    .expectNext("CC")
    .verifyComplete();
```

结果看起来正确。

### 4.2 使用defaultIfEmpty()

或者，**当上游Flux不发出任何元素时，我们可以使用defaultIfEmpty()运算符来提供后备值**，而不是备用发布者：
```java
flux = flux.defaultIfEmpty("No words found!");
```

使用switchIfEmpty()和defaultIfEmpty()之间的另一个主要区别是，我们只能对后者使用单个默认值。

现在，让我们验证一下响应流的条件流：
```java
StepVerifier.create(flux)
    .expectNext("No words found!")
    .verifyComplete();
```

## 5. 使用flatMap()

我们可以**使用[flatMap()](https://www.baeldung.com/rxjava-flatmap-switchmap#flatmap)运算符在我们的响应流中创建多个条件分支**，同时保持非阻塞、异步流。

让我们看一下由单词创建并通过两个flatMap()运算符进行更改的Flux：
```java
Flux<String> flux = Flux.just("A", "B", "C")
        .flatMap(word -> {
            if (word.startsWith("A")) {
                return Flux.just(word + "1", word + "2", word + "3");
            } else {
                return Flux.just(word);
            }
        })
        .flatMap(word -> {
            if (word.startsWith("B")) {
                return Flux.just(word + "1", word + "2");
            } else {
                return Flux.just(word);
            }
        });
```

我们通过添加两个阶段的条件转换创建了动态分支，从而为响应流的每个元素提供了多条逻辑路径。

现在，验证我们响应流的条件流：

```java
StepVerifier.create(flux)
    .expectNext("A1")
    .expectNext("A2")
    .expectNext("A3")
    .expectNext("B1")
    .expectNext("B2")
    .expectNext("C")
    .verifyComplete();
```

此外，我们可以**将[flatMapMany()](https://www.baeldung.com/java-mono-list-to-flux#flatmapmany)用于Mono发布者，以实现类似的用例**。

## 6. 使用副作用运算符

在本节中，我们将探讨如何在处理响应流时执行基于条件的同步操作。

### 6.1 使用doOnNext()

我们可以**使用doOnNext()运算符对响应流的每个元素同步执行副作用操作**。

让我们首先定义evenCounter变量来跟踪我们的响应流中偶数的数量：
```java
AtomicInteger evenCounter = new AtomicInteger(0);
```

现在，让我们创建一个整数流并将其与doOnNext()运算符链接在一起以自增偶数的数量：
```java
Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6)
    .doOnNext(num -> {
    if (num % 2 == 0) {
        evenCounter.incrementAndGet();
    }
    });
```

我们在if块中添加了操作，从而实现了计数器的条件递增。

接下来，我们必须在处理响应流中的每个元素之后验证evenCounter的逻辑和状态：
```java
StepVerifier.create(flux)
    .expectNextMatches(num -> num == 1 && evenCounter.get() == 0)
    .expectNextMatches(num -> num == 2 && evenCounter.get() == 1)
    .expectNextMatches(num -> num == 3 && evenCounter.get() == 1)
    .expectNextMatches(num -> num == 4 && evenCounter.get() == 2)
    .expectNextMatches(num -> num == 5 && evenCounter.get() == 2)
    .expectNextMatches(num -> num == 6 && evenCounter.get() == 3)
    .verifyComplete();
```

### 6.2 使用doOnComplete()

类似地，我们还可以**根据从响应流接收信号的条件关联动作，比如在发布了所有元素后发送的complete信号**。

让我们首先初始化done标志：
```java
AtomicBoolean done = new AtomicBoolean(false);
```

现在，让我们定义一个整数流，并添加使用doOnComplete()运算符将done标志设置为true的操作：
```java
Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6)
    .doOnComplete(() -> done.set(true));
```

值得注意的是，**complete信号仅发送一次，因此副作用动作最多会被触发一次**。

此外，让我们通过在各个步骤中验证done标志来验证副作用的条件执行：
```java
StepVerifier.create(flux)
    .expectNextMatches(num -> num == 1 && !done.get())
    .expectNextMatches(num -> num == 2 && !done.get())
    .expectNextMatches(num -> num == 3 && !done.get())
    .expectNextMatches(num -> num == 4 && !done.get())
    .expectNextMatches(num -> num == 5 && !done.get())
    .expectNextMatches(num -> num == 6 && !done.get())
    .then(() -> Assertions.assertTrue(done.get()))
    .expectComplete()
    .verify();
```

我们可以看到，只有在所有元素成功发出后，done标志才会设置为true。但是，需要注意的是，**doOnComplete()仅适用于Flux发布者，对于Mono发布者，我们必须使用doOnSuccess()**。

## 7. 使用firstOnValue()

有时，我们可能有多个来源来收集数据，但每个来源的延迟可能不同。从性能的角度来看，最好使用延迟最小的来源的值，**对于这种有条件的数据访问，我们可以使用firstOnValue()运算符**。

首先我们定义两个源，分别是source1和source2，延迟分别为200ms和10ms：
```java
Mono<String[]> source1 = Mono.defer(() -> Mono.just(new String[] { "val", "source1" })
    .delayElement(Duration.ofMillis(200)));
Mono<String[]> source2 = Mono.defer(() -> Mono.just(new String[] { "val", "source2" })
    .delayElement(Duration.ofMillis(10)));
```

接下来，让我们将firstWithValue()运算符与两个源一起使用，并依靠框架的条件逻辑来处理数据访问：
```java
Mono<String[]> mono = Mono.firstWithValue(source1, source2);
```

最后，让我们通过将发出的元素与来自延迟较低的源的数据进行比较来验证结果：
```java
StepVerifier.create(mono)
    .expectNextMatches(item -> "val".equals(item[0]) && "source2".equals(item[1]))
    .verifyComplete();
```

此外，需要注意的是**firstWithValue()仅适用于Mono发布者**。

## 8. 使用zip()和zipWhen()

在本节中，让我们学习如何使用zip()和zipWhen()运算符来利用条件流。

### 8.1 使用zip()

我们可以使用zip()运算符来组合来自多个来源的数据，此外，我们**可以使用它的组合函数来添加数据处理的条件逻辑**，让我们看看如何使用它来确定缓存和数据库中的值是否不一致。

首先，让我们定义dataFromDB和dataFromCache发布者来模拟具有不同延迟和值的源：
```java
Mono<String> dataFromDB = Mono.defer(() -> Mono.just("db_val")
    .delayElement(Duration.ofMillis(200)));
Mono<String> dataFromCache = Mono.defer(() -> Mono.just("cache_val")
    .delayElement(Duration.ofMillis(10)));
```

现在，让我们将它们压缩起来，并使用其组合函数添加判断缓存是否一致的条件：
```java
Mono<String[]> mono = Mono.zip(dataFromDB, dataFromCache, 
    (dbValue, cacheValue) -> 
    new String[] { dbValue, dbValue.equals(cacheValue) ? "VALID_CACHE" : "INVALID_CACHE" });
```

最后，让我们验证这个模拟，并验证缓存不一致，因为db_val与cache_val不同：
```java
StepVerifier.create(mono)
    .expectNextMatches(item -> "db_val".equals(item[0]) && "INVALID_CACHE".equals(item[1]))
    .verifyComplete();
```

结果看起来正确。

### 8.2 使用zipWhen()

**当我们只想在第一个源有发射的情况下收集第二个源的发射时，zipWhen()运算符更合适**。此外，我们可以使用组合器函数添加条件逻辑来处理我们的响应流。

假设我们要计算用户的年龄类别：
```java
int userId = 1;
Mono<String> userAgeCategory = Mono.defer(() -> Mono.just(userId))
    .zipWhen(id -> Mono.defer(() -> Mono.just(20)), (id, age) -> age >= 18 ? "ADULT" : "KID");
```

我们模拟了具有有效用户ID的用户场景，因此，我们保证第二个发布者会发出消息。随后，我们将获取用户的年龄类别。

现在，让我们验证这种情况，并确认20岁的用户被归类为“ADULT”：
```java
StepVerifier.create(userDetail)
    .expectNext("ADULT")
    .verifyComplete();
```

接下来，我们使用Mono.empty()来模拟未找到有效用户的场景的分类：
```java
Mono<String> noUserDetail = Mono.empty()
    .zipWhen(id -> Mono.just(20), (id, age) -> age >= 18 ? "ADULT" : "KID");
```

最后，我们可以确认这种情况下没有排放：
```java
StepVerifier.create(noUserDetail)
  .verifyComplete();
```

此外，我们需要注意**zipWhen()仅适用于Mono发布者**。

## 9. 总结

在本文中，我们学习了如何在Spring WebFlux中包含条件语句。此外，我们探讨了如何通过不同的运算符(例如map()、flatMap()、zip()、firstOnValue()、switchIfEmpty()、defaultIfEmpty()等)实现条件流。