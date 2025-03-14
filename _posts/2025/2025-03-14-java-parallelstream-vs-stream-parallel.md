---
layout: post
title:  Java中parallelStream()和stream().parallel()之间的区别
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在本教程中，我们将探索Java [Streams](https://www.baeldung.com/java-streams) API的Collections.parallelStream()和stream().parallel()区别。Java在Java 8中将parallelStream()方法引入Collection接口，并将parallel()方法引入BaseStream接口。 

## 2. Java中的并行流和并行性

并行流允许我们使用多核处理，通过在多个CPU核心上[并行执行流操作](https://www.baeldung.com/cs/concurrency-vs-parallelism)，数据被拆分为多个子流，并并行执行预期操作，最后将结果聚合回来形成最终输出。 

除非另有说明，否则Java中创建的流在本质上始终是串行的。我们可以通过两种方式将流转换为并行流： 

- 调用Collections.parallelStream()
- 调用BaseStream.parallel()

如果流操作未指定，则Java编译器和运行时在执行并行流操作时决定处理顺序，以获得最佳的并行计算效益。

例如，我们有一个很长的Book对象列表，我们必须确定在特定年份出版的书籍数量：

```java
public class Book {
    private String name;
    private String author;
    private int yearPublished;
    
    // getters and setters
}
```

我们可以在这里利用并行流，比串行执行更有效地查找计数。我们示例中的执行顺序不会以任何方式影响最终结果，使其成为并行流操作的完美[候选者](https://www.baeldung.com/java-when-to-use-parallel-stream)。 

## 3. Collections.parallelStream()的用法

我们在应用程序中使用并行流的方法之一是在数据源上调用parallelStream()，**此操作返回一个可能并行的Stream，其源是提供的集合**。我们可以将其应用到我们的示例中，并找出特定年份出版的书籍数量：

```java
long usingCollectionsParallel(Collection<Book> listOfbooks, int year) {
    AtomicLong countOfBooks = new AtomicLong();
    listOfbooks.parallelStream()
        .forEach(book -> {
            if (book.getYearPublished() == year) {
                countOfBooks.getAndIncrement();
            }
        });
    return countOfBooks.get();
}
```

parallelStream()方法的默认实现从Collection的[Spliterator](https://www.baeldung.com/java-spliterator)<T\>接口创建一个并行Stream，Spliterator是一个用于遍历和分割其源元素的对象。Spliterator可以使用其trySplit()方法分割其源的一些元素，使其符合可能的并行操作条件。 

Spliterator API与Iterator类似，允许遍历其源的元素，旨在支持高效的并行遍历。Collection的默认Spliterator将用于parallelStream()调用。 

##  4. 在Stream上使用parallel()

我们可以通过首先将集合转换为Stream来实现相同的结果，通过对其调用parallel()将结果生成的顺序流转换为并行Stream。一旦我们有了并行Stream，我们就可以按照上面相同的方式找到结果：

```java
long usingStreamParallel(Collection<Book> listOfBooks, int year) {
    AtomicLong countOfBooks = new AtomicLong();
    listOfBooks.stream().parallel()
        .forEach(book -> {
            if (book.getYearPublished() == year) {
                countOfBooks.getAndIncrement();
            }
        });
    return countOfBooks.get();
}
```

Streams API的BaseStream接口将根据源集合的默认Spliterator允许的程度对底层数据进行拆分，然后使用[Fork-Join](https://www.baeldung.com/java-fork-join)框架将其转换为并行Stream。 

两种方法的结果都是相同的。 

## 5. parallelStream()和stream().parallel()的区别

Collections.parallelStream()使用源集合的默认Spliterator来分割数据源以实现并行执行。均匀分割数据源对于实现正确的并行执行非常重要，不均匀分割的数据源对并行执行的危害比其顺序对应方更大。

在所有这些示例中，我们一直使用List<Book\>来保存书籍列表。现在让我们尝试通过重写Collection<T\>接口为我们的书籍创建自定义集合。 

**我们应该记住，覆盖接口意味着我们必须提供基接口中定义的抽象方法的实现**。但是，我们看到spliterator()、stream()和 parallelStream()等方法作为接口中的默认方法存在，这些方法默认提供实现。不过，仍然可以用我们的版本覆盖这些实现。

我们将把我们自定义的书籍集合称为MyBookContainer，并且还将定义我们自己的Spliterator： 

```java
public class BookSpliterator<T> implements Spliterator<T> {
    private final Object[] books;
    private int startIndex;
    public BookSpliterator(Object[] books, int startIndex) {
        this.books = books;
        this.startIndex = startIndex;
    }
    
    @Override
    public Spliterator<T> trySplit() {
        // Always Assuming that the source is too small to split, returning null
        return null;
    }

    // Other overridden methods such as tryAdvance(), estimateSize() etc
}
```

在上面的代码片段中，我们看到我们的Spliterator版本接收一个Object数组(在我们的例子中是Book)进行拆分，并且在trySplit()方法中，它始终返回null。 

我们应该注意，Spliterator<T\>接口的这种实现很容易出错，它不会将数据分成相等的部分；而是返回null，导致数据不平衡。这只是为了表示目的。

我们将在自定义Collection类MyBookContainer中使用这个Spliterator： 

```java
public class MyBookContainer<T> implements Collection<T> {
    private static final long serialVersionUID = 1L;
    private T[] elements;

    public MyBookContainer(T[] elements) {
        this.elements = elements;
    }

    @Override
    public Spliterator<T> spliterator() {
        return new BookSpliterator(elements, 0);
    }

    @Override
    public Stream<T> parallelStream() {
        return StreamSupport.stream(spliterator(), false);
    }
    
    // standard overridden methods of Collection Interface
}
```

我们将尝试将数据存储在我们的自定义容器类中并对其执行并行流操作：

```java
long usingWithCustomSpliterator(MyBookContainer<Book> listOfBooks, int year) {
    AtomicLong countOfBooks = new AtomicLong();
    listOfBooks.parallelStream()
        .forEach(book -> {
            if (book.getYearPublished() == year) {
                countOfBooks.getAndIncrement();
            }
        });
    return countOfBooks.get();
}
```

本例中的数据源是MyBookContainer类型的实例，此代码内部使用我们自定义的Spliterator来拆分数据源，生成的并行Stream最多是一个顺序流。 

我们刚刚利用了parallelStream()方法返回一个顺序流，尽管其名称暗示了parallelStream。这就是该方法与stream().parallel()不同的地方，后者始终尝试返回提供给它的流的并行版本。Java在其文档中记录了此行为，如下所示：

```java
@implSpec
/* The default implementation creates a parallel {@code Stream} from the
 * collection's {@code Spliterator}.
 *
 * @return a possibly parallel {@code Stream} over the elements in this
 * collection
 * @since 1.8
 */
default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}
```

## 6. 总结

在本文中，我们介绍了从Collection数据源创建并行Streams的方法。我们还尝试通过实现Collection和Spliterator的自定义版本来找出parallelStream()和stream().parallel()之间的区别。