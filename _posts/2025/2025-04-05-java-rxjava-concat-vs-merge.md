---
layout: post
title:  RxJava Observables中的concat()与merge()运算符
category: rxjava
copyright: rxjava
excerpt: RxJava
---

## 1. 概述

concat()和[merge()](https://www.baeldung.com/rxjava-combine-observables#merge)是两个强大的运算符，用于在[RxJava](https://www.baeldung.com/rx-java)中组合多个Observable实例。

concat()按顺序从每个Observable发出元素，等待每个元素完成后再移动到下一个，而merge()在生成所有Observable实例时同时发出它们。

在本教程中，**我们将探讨concat()和merge()显示相似和不同行为的场景**。

## 2. 同步源

**当两个源都是同步的时，concat()和merge()运算符的行为完全相同**，让我们模拟这种情况以更好地理解它。

### 2.1 场景设置

让我们**首先使用Observable.just()工厂方法创建三个同步源**：

```java
Observable<Integer> observable1 = Observable.just(1, 2, 3);
Observable<Integer> observable2 = Observable.just(4, 5, 6);
Observable<Integer> observable3 = Observable.just(7, 8, 9);
```

此外，让我们创建两个订阅者，即testSubscriberForConcat和testSubscriberForMerge：

```java
TestSubscriber<Integer> testSubscriberForConcat = new TestSubscriber<>();
TestSubscriber<Integer> testSubscriberForMerge = new TestSubscriber<>();
```

### 2.2 concat()和merge()

首先，让我们应用concat()运算符并使用testSubscriberForConcat订阅生成的Observable：

```java
Observable.concat(observable1, observable2, observable3)
    .subscribe(testSubscriberForConcat);
```

进一步，让我们**验证一下发射是否按顺序进行，其中observable1中的元素出现在observable2之前，而observable2出现在observable3之前**：

```java
testSubscriberForConcat.assertValues(1, 2, 3, 4, 5, 6, 7, 8, 9);
```

![](/assets/images/2025/rxjava/javarxjavaconcatvsmerge01.png)

类似地，我们可以应用merge()运算符并使用testSubscriberForMerge订阅结果：

```java
Observable.merge(observable1, observable2, observable3).subscribe(testSubscriberForMerge);
```

接下来，让我们验证通过merge发出的元素是否遵循与concat相同的顺序：

```java
testSubscriberForMerge.assertValues(1, 2, 3, 4, 5, 6, 7, 8, 9);
```

![](/assets/images/2025/rxjava/javarxjavaconcatvsmerge02.png)

最后，我们必须注意，**同步Observable实例会立即发出所有元素**，然后发出完成信号。此外，**每个Observable都会在下一个Observable开始之前完成其发射**。因此，两个运算符都会按顺序处理每个Observable，从而产生相同的输出。

因此，无论源是同步还是异步的，**一般规则是，如果我们需要保持源的发射顺序，则应使用concat()**。另一方面，如果我们想要合并从多个源发射的元素，则应使用merge()。

## 3. 可预测的异步源

在本节中，让我们模拟一个具有异步源的场景，其中发射顺序是可预测的。

### 3.1 场景设置

让我们创建两个异步源，即observable1和observable2：

```java
Observable<Integer> observable1 = Observable.interval(100, TimeUnit.MILLISECONDS)
    .map(i -> i.intValue() + 1)
    .take(3);

Observable<Integer> observable2 = Observable.interval(30, TimeUnit.MILLISECONDS)
    .map(i -> i.intValue() + 4)
    .take(7);
```

我们必须注意到observable1的发射分别在100ms、200ms和300ms后到达。另一方面，observable2的发射以30ms的间隔到达。

现在，让我们创建testSubscriberForConcat和testSubscriberForMerge：

```java
TestSubscriber<Integer> testSubscriberForConcat = new TestSubscriber<>();
TestSubscriber<Integer> testSubscriberForMerge = new TestSubscriber<>();
```

### 3.2 concat()与merge()

首先，让我们应用concat()运算符并使用testSubscribeForConcat调用subscribe()：

```java
Observable.concat(observable1, observable2)
    .subscribe(testSubscriberForConcat);
```

接下来，我们**必须调用awaitTerminalEvent()方法来确保收到所有发射**：

```java
testSubscriberForConcat.awaitTerminalEvent();
```

现在，我们可以验证结果包含来自observable1的所有元素以及来自observable2的所有元素：

```java
testSubscriber.assertValues(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
```

![](/assets/images/2025/rxjava/javarxjavaconcatvsmerge03.png)

此外，让我们应用merge()运算符并使用testSubscriberForMerge调用subscribe()：

```java
Observable.merge(observable1, observable2)
  .subscribe(testSubscriberForMerge);
```

最后，让我们等待发射并检查发射的值：

```java
testSubscriberForMerge.awaitTerminalEvent();
testSubscriberForMerge.assertValues(4, 5, 6, 1, 7, 8, 9, 2, 10, 3);
```

![](/assets/images/2025/rxjava/javarxjavaconcatvsmerge04.png)

结果包含**按照observer1和observer2的实际发射顺序交错在一起的所有元素**。

## 4. 具有竞争条件的异步源

在本节中，我们将模拟具有两个异步源的场景，其中组合发射的顺序相当难以预测。

### 4.1 场景设置

首先，让我们**创建两个具有完全相同延迟的异步源**：

```java
Observable<Integer> observable1 = Observable.interval(100, TimeUnit.MILLISECONDS)
    .map(i -> i.intValue() + 1)
    .take(3);

Observable<Integer> observable2 = Observable.interval(100, TimeUnit.MILLISECONDS)
    .map(i -> i.intValue() + 4)
    .take(3);
```

我们知道每个源的一次发射分别在100ms、200ms和300ms后到达。但是，**由于竞争条件，我们无法预测确切的顺序**。

接下来，让我们创建两个测试订阅者：

```java
TestSubscriber<Integer> testSubscriberForConcat = new TestSubscriber<>();
TestSubscriber<Integer> testSubscriberForMerge = new TestSubscriber<>();
```

### 4.2 concat()与merge()

首先，让我们应用concat()运算符，然后订阅testSubscribeForConcat：

```java
Observable.concat(observable1, observable2)
    .subscribe(testSubscriberForConcat);
testSubscriberForConcat.awaitTerminalEvent();
```

现在，让我们**验证concat()运算符的结果是否保持不变**：

```java
testSubscriberForConcat.assertValues(1, 2, 3, 4, 5, 6);
```

![](/assets/images/2025/rxjava/javarxjavaconcatvsmerge05.png)

此外，让我们使用testSubscriberForMerge进行merge()和订阅：

```java
Observable.merge(observable1, observable2)
    .subscribe(testSubscriberForMerge);
testSubscriberForMerge.awaitTerminalEvent();
```

接下来，让我们**将所有发射累积到一个列表中，并验证它是否包含所有值**：

```java
List<Integer> actual = testSubscriberForMerge.getOnNextEvents();
List<Integer> expected = Arrays.asList(1, 2, 3, 4, 5, 6);
assertTrue(actual.containsAll(expected) && expected.containsAll(actual));
```

最后，让我们记录发射以观察其实际作用：

```text
21:05:43.252 [main] INFO actual emissions: [4, 1, 2, 5, 3, 6]
```

![](/assets/images/2025/rxjava/javarxjavaconcatvsmerge06.png)

我们可能针对不同的运行得到不同的顺序。

## 5. 总结

在本文中，我们了解了RxJava中的concat()和merge()运算符如何处理同步和异步数据源。此外，我们比较了涉及可预测和不可预测发射模式的场景，强调了这两个运算符之间的差异。