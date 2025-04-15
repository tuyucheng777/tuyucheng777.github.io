---
layout: post
title:  在Java中对日期字符串进行排序
category: java-date
copyright: java-date
excerpt: Java Date
---

## 1. 概述

在处理包含字符串格式的日期时间的数据集时，对这些字符串进行排序是许多Java应用程序中的常见任务。

在本教程中，我们将探讨在Java中有效排序日期字符串的不同方法。

## 2. 问题介绍

我们可以直接按照特定日期格式(例如ISO日期时间格式(YYYY-MM-dd'T'HH:mm:ss))的字典顺序对字符串进行排序。然而，**这并不是对日期字符串进行排序的通用解决方案**。

我们不能对所有日期时间格式都应用字典顺序排序操作，例如，假设我们有这样一个字符串列表：

```java
List<String> dtStrings = Lists.newArrayList(
    "01/21/2013 10:41",
    "01/20/2013 10:48",
    "01/22/2013 15:13",
    "01/21/2013 16:37",
    "01/21/2013 17:16",
    "01/21/2013 17:19",
    "01/20/2013 06:16",
    "01/22/2013 06:19"
);
```

如果列表中的字符串排序正确，结果应该如下所示：

```java
List<String> EXPECTED = Lists.newArrayList(
    "01/20/2013 06:16",
    "01/20/2013 10:48",
    "01/21/2013 10:41",
    "01/21/2013 16:37",
    "01/21/2013 17:16",
    "01/21/2013 17:19",
    "01/22/2013 06:19",
    "01/22/2013 15:13"
);
```

我们将探索解决排序问题的不同方法，此外，为了简单起见，我们将使用单元测试断言来验证每个解决方案是否产生了预期的结果。

接下来，让我们看看它们的实际效果。

## 3. 使用自定义Comparator

Java标准库提供了[Collections.sort()](https://www.baeldung.com/java-sorting#sorting-a-list)方法来对集合中的元素进行排序，如果我们想按字典顺序对字符串列表进行排序，只需将列表传递给Collections.sort()方法即可。此外，该方法也接收[Comparator](https://www.baeldung.com/java-comparator-comparable)对象作为第二个参数。

接下来，让我们看看如何使用自定义比较器对日期时间字符串进行排序：

```java
DateFormat dfm = new SimpleDateFormat("MM/dd/yyyy HH:mm");
Collections.sort(dtStrings, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        try {
            return dfm.parse(o1).compareTo(dfm.parse(o2));
        } catch (ParseException e) {
            throw new IllegalArgumentException(e);
        }
    }
});
assertEquals(EXPECTED, dtStrings);
```

如上面的代码所示，首先，我们根据日期字符串格式创建了一个[SimpleDateFormat](https://www.baeldung.com/java-simple-date-format)对象。然后，在调用Collections.sort()方法时，我们将dtStrings列表与一个[匿名](https://www.baeldung.com/java-anonymous-classes)Comparator对象一起传递。

在compare()方法实现中，我们**首先[将两个日期时间字符串解析为Date对象](https://www.baeldung.com/java-string-to-date)，然后比较两个Date对象**。

如果我们的Java版本是8或更高，我们可以使用[Lambda比较](https://www.baeldung.com/java-8-sort-lambda)功能来使我们的代码紧凑且更易于阅读：

```java
final DateTimeFormatter dfm = DateTimeFormatter.ofPattern("MM/dd/yyyy HH:mm");
dtStrings.sort(Comparator.comparing(s -> LocalDateTime.parse(s, dfm)));
assertEquals(EXPECTED, dtStrings);
```

值得注意的是，**Collections.sort()和list.sort()方法都支持就地排序**，这意味着**可以直接修改原始列表**，而无需创建新的已排序副本，这在内存效率和性能方面具有显著优势。

## 4. 使用Stream API

此外，要对日期时间字符串列表进行排序，我们可以采用三步方法：

- 将字符串元素转换为LocalDateTime实例
- 对LocalDateTime对象进行排序
- 将LocalDateTime对象转换回String

[Stream API](https://www.baeldung.com/java-8-streams)使我们能够方便地处理集合，如果我们使用Stream API实现这个想法，那么[map()](https://www.baeldung.com/java-difference-map-and-flatmap)方法可以帮助我们执行以下转换：

```java
DateTimeFormatter dfm = DateTimeFormatter.ofPattern("MM/dd/yyyy HH:mm");
List<String> sortedList = dtStrings.stream()
    .map(s -> LocalDateTime.parse(s, dfm))
    .sorted()
    .map(dfm::format)
    .collect(Collectors.toList());
assertEquals(EXPECTED, sortedList);
```

与Collections.sort()和list.sort()解决方案不同，此方法不会更改原始列表。相反，**它返回一个新列表来保存已排序的字符串**。

## 5. 使用TreeMap

**Java中的[TreeMap](https://www.baeldung.com/java-treemap)类提供了基于键对条目进行自动排序的功能**，利用此功能，我们可以通过创建包含LocalDateTime和String类型的键值对的TreeMap来轻松地对日期时间字符串进行排序。

然后，如果我们从TreeMap中获取所有值，例如，使用treeMap.values()方法，我们将得到排序的结果。

接下来，我们在测试中实现这一点：

```java
DateTimeFormatter dfm = DateTimeFormatter.ofPattern("MM/dd/yyyy HH:mm");
Map<LocalDateTime, String> dateFormatMap = new TreeMap<>();
dtStrings.forEach(s -> dateFormatMap.put(LocalDateTime.parse(s, dfm), s));
List<String> result = new ArrayList<>(dateFormatMap.values());
assertEquals(EXPECTED, result);
```

这个解决方案很简单，然而，它有一个缺点，由于**标准Java Map不能包含重复的键，排序后的列表中重复的日期时间字符串将会丢失**。因此，在使用TreeMap方法对列表进行排序之前，最好确保列表不包含重复的值。

## 6. 总结

在本文中，我们探讨了对日期字符串进行排序的不同通用解决方案：

- 使用自定义Comparator进行Collections.sort()和list.sort()(就地排序)
- 将字符串转换为日期对象，对对象进行排序，然后将其转换回日期字符串
- TreeMap方法