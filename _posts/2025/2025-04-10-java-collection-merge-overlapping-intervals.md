---
layout: post
title:  合并Java集合中的重叠区间
category: algorithms
copyright: algorithms
excerpt: 集合
---

## 1. 概述

在本教程中，我们将研究如何获取区间[集合](https://www.baeldung.com/java-collections)并合并任何重叠的区间。

## 2. 理解问题

首先，让我们定义一下什么是区间。**我们将其描述为一对数字，其中第二个数字大于第一个数字**。日期也可以是区间，但这里我们只使用整数。例如，区间可以是2 -> 7，或100 -> 200。

我们可以创建一个简单的Java类来表示一个区间，我们将start作为第一个数字，将end作为第二个及更大的数字：
```java
class Interval {
    int start;
    int end;

    // Usual constructor, getters, setters and equals functions
}
```

现在，我们想要获取一个Interval集合并合并任何重叠的区间。**我们可以将重叠区间定义为第一个区间结束于第二个区间开始之后的任何位置**。例如，如果我们取两个区间2 -> 10和5 -> 15，我们可以看到它们重叠，合并它们的结果将是2 -> 15。

## 3. 算法

为了执行合并，我们需要遵循以下模式的代码：
```java
List<Interval> doMerge(List<Interval> intervals) {
    intervals.sort(Comparator.comparingInt(interval -> interval.start));
    ArrayList<Interval> merged = new ArrayList<>();
    merged.add(intervals.get(0));

    // < code to merge intervals into merged comes here >

    return merged;
}
```

首先，我们定义了一个方法，它接收一个Interval列表并返回经过排序的列表。当然，我们的Interval可以存储在其他类型的Collection中，但我们这里只使用List。

处理的第一步是对我们的区间集合进行排序，**我们根据区间的开始进行排序，这样当我们循环遍历它们时，每个区间后面都会跟着最有可能与其重叠的区间**。 

假设我们的Collection是一个List，我们可以使用List.sort()来执行此操作。但是，如果我们有其他类型的Collection，我们就必须[使用其他方法](https://www.baeldung.com/java-sorting)。List.sort()接收一个Comparator并进行排序，因此我们不需要重新分配变量。**我们使用[Comparator.comparingInt()](https://www.baeldung.com/java-8-comparator-comparing#4-using-comparatorcomparingint)作为sort()方法的Comparator**。

接下来，我们需要一个地方来存放合并过程的结果。为此，我们创建了一个名为merged的ArrayList。我们还可以通过放入第一个区间来启动它，因为我们知道它是最早开始的。

现在，我们将把intervals的元素合并到merged中，这是解决方案的主要部分。根据interval的start和end，有3种可能性。让我们绘制一个ASCII图表来帮助我们更容易理解合并逻辑：
```text
-----------------------------------------------------------------------
| Legend:                                                             |
| |______|  <-- lastMerged - The last lastMerged interval             |
|                                                                     |
| |......|  <-- interval - The next interval in the sorted input list |
-----------------------------------------------------------------------


## Case 1: interval.start <= lastMerged.end && interval.end > lastMerged.end

 lastMerged.start         lastMerged.end
  |______________________________|

         |...............................|
     interval.start        interval.end


## Case 2: interval.start <= lastMerged.end && interval.end <= lastMerged.end 

 lastMerged.start         lastMerged.end
  |______________________________|

      |..................|
  interval.start     interval.end


## Case 3: interval.start > lastMerged.end 

 lastMerged.start         lastMerged.end
  |_________________________|

                               |..............|
                     interval.start        interval.end
```

根据ASCII图表，我们检测到案例1和案例2存在重叠。为了解决这个问题，我们需要合并lastMerged和interval。幸运的是，合并它们的过程很简单，**我们可以通过选择lastMerged.end和interval.end之间的较大值来更新lastMerged.end**：
```java
if (interval.start <= lastMerged.end){
    lastMerged.setEnd(max(interval.end, lastMerged.end));
}
```

对于案例3，没有重叠，因此我们**将当前interval作为新元素添加到merged结果列表中**：
```java
merged.add(interval);
```

最后我们通过在doMerge()方法中添加合并逻辑实现来获得完整的解决方案：
```java
List<Interval> doMerge(List<Interval> intervals) {
    intervals.sort(Comparator.comparingInt(interval -> interval.start));
    ArrayList<Interval> merged = new ArrayList<>();
    merged.add(intervals.get(0));

    intervals.forEach(interval -> {
        Interval lastMerged = merged.get(merged.size() - 1);
        if (interval.start <= lastMerged.end){
            lastMerged.setEnd(max(interval.end, lastMerged.end));
        } else {
            merged.add(interval);
        }
    });

    return merged;
}
```

## 4. 实例

让我们通过测试来看一下该方法的实际效果，我们可以定义一些想要合并的Interval：
```java
List<Interval> intervals = new ArrayList<>(Arrays.asList(
    new Interval(3, 5),
    new Interval(13, 20),
    new Interval(11, 15),
    new Interval(15, 16),
    new Interval(1, 3),
    new Interval(4, 5),
    new Interval(16, 17)
));
```

合并后，我们期望结果只有两个Interval：
```java
List<Interval> intervalsMerged = new ArrayList<>(Arrays.asList(
    new Interval(1, 5), 
    new Interval(11, 20)
));
```

我们现在可以使用我们之前创建的方法并确认它按预期工作：
```java
MergeOverlappingIntervals merger = new MergeOverlappingIntervals();
List<Interval> result =  merger.doMerge(intervals);
assertEquals(intervalsMerged, result);
```

## 5. 总结

在本文中，我们学习了如何在Java中合并集合中的重叠区间。首先，我们必须知道如何按起始值对它们进行正确的排序。然后，我们只需要明白，当一个区间在前一个区间结束之前开始时，我们需要合并它们。