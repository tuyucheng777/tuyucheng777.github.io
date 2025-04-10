---
layout: post
title:  在Java中检查列表是否已排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 概述

在本教程中，我们将了解在Java中检查列表是否已排序的不同方法。

## 2. 迭代方法

迭代法是一种简单且直观的检查列表是否排序的方法，在这种方法中，我们将迭代列表并比较相邻元素，如果任何两个相邻元素未排序，我们就可以说该列表未排序。

列表可以按自然顺序或自定义顺序排序，我们将使用Comparable和Comparator接口介绍这两种情况。

### 2.1 使用Comparable

首先，让我们看一个元素类型为Comparable的列表的示例。这里，我们考虑一个包含String类型对象的列表：
```java
public static boolean isSorted(List<String> listOfStrings) {
    if (isEmpty(listOfStrings) || listOfStrings.size() == 1) {
        return true;
    }

    Iterator<String> iter = listOfStrings.iterator();
    String current, previous = iter.next();
    while (iter.hasNext()) {
        current = iter.next();
        if (previous.compareTo(current) > 0) {
            return false;
        }
        previous = current;
    }
    return true;
}
```

### 2.2 使用Comparable

现在，让我们考虑一个没有实现Comparable的Employee类。因此，在这种情况下，我们需要使用Comparator来比较列表中的相邻元素：
```java
public static boolean isSorted(List<Employee> employees, Comparator<Employee> employeeComparator) {
    if (isEmpty(employees) || employees.size() == 1) {
        return true;
    }

    Iterator<Employee> iter = employees.iterator();
    Employee current, previous = iter.next();
    while (iter.hasNext()) {
        current = iter.next();
        if (employeeComparator.compare(previous, current) > 0) {
            return false;
        }
        previous = current;
    }
    return true;
}
```

上述两个例子很相似，唯一的区别在于我们如何比较列表中的前一个元素和当前元素。

此外，**我们还可以使用Comparator对排序检查进行精确控制**。有关这两者的更多信息，请参阅我们的[Java中的Comparator和Comparable](https://www.baeldung.com/java-comparator-comparable)教程。

## 3. 递归方法

现在，我们将看到如何使用递归检查排序列表：
```java
public static boolean isSorted(List<String> listOfStrings) {
    return isSorted(listOfStrings, listOfStrings.size());
}

public static boolean isSorted(List<String> listOfStrings, int index) {
    if (index < 2) {
        return true;
    } else if (listOfStrings.get(index - 2).compareTo(listOfStrings.get(index - 1)) > 0) {
        return false;
    } else {
        return isSorted(listOfStrings, index - 1);
    }
}
```

## 4. 使用Guava

使用第三方库通常比编写自己的逻辑更好，Guava库有一些实用程序类，可以用来检查列表是否已排序。

### 4.1 Guava Ordering类

在本节中，我们将了解如何使用Guava中的Ordering类来检查排序列表。

首先，我们来看一个包含Comparable类型元素的列表的示例：
```java
public static boolean isSorted(List<String> listOfStrings) {
    return Ordering.<String> natural().isOrdered(listOfStrings);
}
```

接下来，我们将看到如何使用Comparator检查Employee对象列表是否已排序：
```java
public static boolean isSorted(List<Employee> employees, Comparator<Employee> employeeComparator) {
    return Ordering.from(employeeComparator).isOrdered(employees);
}
```

另外，我们可以使用natural().reverseOrder()来检查列表是否按相反顺序排序。此外，我们可以使用natural().nullFirst()和natural().nullLast()来检查null是否出现在排序列表的第一个或最后一个。

要了解有关Guava Ordering类的更多信息，我们可以参考[Guava Ordering指南](https://www.baeldung.com/guava-ordering)文章。

### 4.2 Guava Comparators类

如果我们使用的是Java 8或更高版本，Guava在Comparators类方面提供了更好的替代方案，下面是一个使用此类的isInOrder方法的示例：
```java
public static boolean isSorted(List<String> listOfStrings) {
    return Comparators.isInOrder(listOfStrings, Comparator.<String> naturalOrder());
}
```

如我们所见，在上面的例子中，我们使用了自然排序来检查列表是否已排序，也可以使用Comparator来自定义排序检查。

## 5. 总结

在本文中，我们了解了如何使用简单的迭代方法、递归方法和Guava检查列表是否已排序，我们还简要介绍了Comparator和Comparable在确定排序检查逻辑中的用法。