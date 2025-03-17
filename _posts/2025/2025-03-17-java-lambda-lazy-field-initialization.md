---
layout: post
title:  使用Lambda进行惰性字段初始化
category: java
copyright: java
excerpt: Java Function
---

## 1. 简介

通常，当我们处理需要执行昂贵或缓慢的方法(例如数据库查询或REST调用)的资源时，我们倾向于使用本地缓存或私有字段。通常，[Lambda函数](https://www.baeldung.com/java-functional-programming)允许我们使用方法作为参数并推迟方法的执行或完全忽略它。

在本教程中，我们将展示使用Lambda函数延迟初始化字段的不同方法。

## 2. Lambda替换

让我们实现我们自己的解决方案的第一个版本。作为第一次迭代，我们将提供LambdaSupplier类：

```java
public class LambdaSupplier<T> {

    protected final Supplier<T> expensiveData;

    public LambdaSupplier(Supplier<T> expensiveData) {
        this.expensiveData = expensiveData;
    }

    public T getData() {
        return expensiveData.get();
    }
}
```

LambdaSupplier通过延迟执行Supplier.get()来实现字段的惰性初始化，**如果多次调用getData()方法，则多次调用Supplier.get()方法**。因此，此类的行为与Supplier接口完全相同，每次调用getData()方法时都会执行底层方法。

为了展示这种行为，让我们编写一个单元测试：

```java
@Test
public void whenCalledMultipleTimes_thenShouldBeCalledMultipleTimes() {
    @SuppressWarnings("unchecked") Supplier<String> mockedExpensiveFunction = Mockito.mock(Supplier.class);
    Mockito.when(mockedExpensiveFunction.get())
        .thenReturn("expensive call");
    LambdaSupplier<String> testee = new LambdaSupplier<>(mockedExpensiveFunction);
    Mockito.verify(mockedExpensiveFunction, Mockito.never())
        .get();
    testee.getData();
    testee.getData();
    Mockito.verify(mockedExpensiveFunction, Mockito.times(2))
        .get();
}
```

正如预期的那样，我们的测试用例验证了Supplier.get()函数被调用了两次。

## 3. 惰性Supplier

由于LambdaSupplier无法缓解多次调用问题，我们实现的下一步发展旨在保证昂贵方法的单次执行。LazyLambdaSupplier通过将返回值缓存到私有字段来扩展LambdaSupplier的实现：

```java
public class LazyLambdaSupplier<T> extends LambdaSupplier<T> {

    private T data;

    public LazyLambdaSupplier(Supplier<T> expensiveData) {
        super(expensiveData);
    }

    @Override
    public T getData() {
        if (data != null) {
            return data;
        }
        return data = expensiveData.get();
    }
}
```

此实现将返回的值存储到私有字段data中，以便可以在连续调用中重复使用该值。

以下测试用例验证新实现在顺序调用时不会进行多次调用：

```java
@Test
public void whenCalledMultipleTimes_thenShouldBeCalledOnlyOnce() {
    @SuppressWarnings("unchecked") Supplier<String> mockedExpensiveFunction = Mockito.mock(Supplier.class);
    Mockito.when(mockedExpensiveFunction.get())
        .thenReturn("expensive call");
    LazyLambdaSupplier<String> testee = new LazyLambdaSupplier<>(mockedExpensiveFunction);
    Mockito.verify(mockedExpensiveFunction, Mockito.never())
        .get();
    testee.getData();
    testee.getData();
    Mockito.verify(mockedExpensiveFunction, Mockito.times(1))
        .get();
}
```

本质上，此测试用例的模板与我们之前的测试用例相同。重要的区别在于，在第二个用例中，我们验证Mock函数仅被调用一次。

**为了证明这个解决方案不是线程安全的，让我们编写一个并发执行的测试用例**：

```java
@Test
public void whenCalledMultipleTimesConcurrently_thenShouldBeCalledMultipleTimes() throws InterruptedException {
    @SuppressWarnings("unchecked") Supplier mockedExpensiveFunction = Mockito.mock(Supplier.class);
    Mockito.when(mockedExpensiveFunction.get())
        .thenAnswer((Answer) invocation -> {
            Thread.sleep(1000L);
            return "Late response!";
        });
    LazyLambdaSupplier testee = new LazyLambdaSupplier<>(mockedExpensiveFunction);
    Mockito.verify(mockedExpensiveFunction, Mockito.never())
        .get();

    ExecutorService executorService = Executors.newFixedThreadPool(4);
    executorService.invokeAll(List.of(testee::getData, testee::getData));
    executorService.shutdown();
    if (!executorService.awaitTermination(5, TimeUnit.SECONDS)) {
        executorService.shutdownNow();
    }

    Mockito.verify(mockedExpensiveFunction, Mockito.times(2))
        .get();
}
```

在上述测试中，Supplier.get()函数被调用了两次。为了实现这一点，ExecutorService同时调用了两个线程来调用LazyLambdaSupplier.getData()函数。此外，我们添加到mockedExpensiveFunction的Thread.sleep()调用保证了当两个线程都调用getData()函数时，字段data仍为空。

## 4. 线程安全解决方案

最后，让我们解决上面演示的线程安全限制。为了实现这一点，我们需要使用同步数据访问和线程安全的值包装器，即AtomicReference。

让我们结合目前所学的知识来编写LazyLambdaThreadSafeSupplier：

```java
public class LazyLambdaThreadSafeSupplier<T> extends LambdaSupplier<T> {

    private final AtomicReference<T> data;

    public LazyLambdaThreadSafeSupplier(Supplier<T> expensiveData) {
        super(expensiveData);
        data = new AtomicReference<>();
    }

    public T getData() {
        if (data.get() == null) {
            synchronized (data) {
                if (data.get() == null) {
                    data.set(expensiveData.get());
                }
            }
        }
        return data.get();
    }
}
```

为了解释为什么这种方法是线程安全的，我们需要想象多个线程同时调用getData()方法。线程确实会阻塞，并且执行将是连续的，直到data.get()调用不为空。一旦data字段初始化完成，多个线程就可以同时访问它。

乍一看，有人可能会认为getData()方法中的双重空值检查是多余的，但事实并非如此。**事实上，外层空值检查可确保当data.get()不为空时，线程不会阻塞在同步块上**。

为了验证我们的实现是否是线程安全的，让我们以与以前的解决方案相同的方式提供单元测试：

```java
@Test
public void whenCalledMultipleTimesConcurrently_thenShouldBeCalledOnlyOnce() throws InterruptedException {
    @SuppressWarnings("unchecked") Supplier mockedExpensiveFunction = Mockito.mock(Supplier.class);
    Mockito.when(mockedExpensiveFunction.get())
        .thenAnswer((Answer) invocation -> {
            Thread.sleep(1000L);
            return "Late response!";
        });
    LazyLambdaThreadSafeSupplier testee = new LazyLambdaThreadSafeSupplier<>(mockedExpensiveFunction);
    Mockito.verify(mockedExpensiveFunction, Mockito.never())
        .get();

    ExecutorService executorService = Executors.newFixedThreadPool(4);
    executorService.invokeAll(List.of(testee::getData, testee::getData));
    executorService.shutdown();
    if (!executorService.awaitTermination(5, TimeUnit.SECONDS)) {
        executorService.shutdownNow();
    }

    Mockito.verify(mockedExpensiveFunction, Mockito.times(1))
        .get();
}
```

## 5. 总结

在本文中，我们展示了使用Lambda函数延迟初始化字段的不同方法。通过这种方法，我们可以避免多次执行昂贵的调用，还可以推迟它们。我们的示例可以用作本地缓存或[Project Lombok](https://www.baeldung.com/intro-to-project-lombok)的Lazy Getter的替代方案。