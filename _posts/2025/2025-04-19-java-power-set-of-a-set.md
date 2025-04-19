---
layout: post
title:  在Java中获取集合的幂集
category: algorithms
copyright: algorithms
excerpt: 幂集
---

## 1. 简介

在本教程中，我们将研究在Java中生成给定集合的[幂集](https://en.wikipedia.org/wiki/Power_set)的过程。

简单回顾一下，对于每个大小为n的集合，都有一个大小为2<sup>n</sup>的幂集，我们将学习如何使用各种技巧来获取它。

## 2. 幂集的定义

给定集合S的幂集是S的所有子集的集合，包括S本身和空集。

例如，对于给定的集合：

```text
{"APPLE", "ORANGE", "MANGO"}
```

幂集为：

```text
{
    {},
    {"APPLE"},
    {"ORANGE"},
    {"APPLE", "ORANGE"},
    {"MANGO"},
    {"APPLE", "MANGO"},
    {"ORANGE", "MANGO"},
    {"APPLE", "ORANGE", "MANGO"}
}
```

由于它也是一组子集，因此其内部子集的顺序并不重要，它们可以以任何顺序出现：

```text
{
    {},
    {"MANGO"},
    {"ORANGE"},
    {"ORANGE", "MANGO"},
    {"APPLE"},
    {"APPLE", "MANGO"},
    {"APPLE", "ORANGE"},
    {"APPLE", "ORANGE", "MANGO"}
}
```

## 3. Guava库

[Google Guava](https://www.baeldung.com/guava-sets)库有一些实用的Set函数，例如幂集。因此，我们也可以轻松地使用它来获取给定集合的幂集：

```java
@Test
public void givenSet_WhenGuavaLibraryGeneratePowerSet_ThenItContainsAllSubsets() {
    ImmutableSet<String> set = ImmutableSet.of("APPLE", "ORANGE", "MANGO");
    Set<Set<String>> powerSet = Sets.powerSet(set);
    Assertions.assertEquals((1 << set.size()), powerSet.size());
    MatcherAssert.assertThat(powerSet, Matchers.containsInAnyOrder(
            ImmutableSet.of(),
            ImmutableSet.of("APPLE"),
            ImmutableSet.of("ORANGE"),
            ImmutableSet.of("APPLE", "ORANGE"),
            ImmutableSet.of("MANGO"),
            ImmutableSet.of("APPLE", "MANGO"),
            ImmutableSet.of("ORANGE", "MANGO"),
            ImmutableSet.of("APPLE", "ORANGE", "MANGO")
    ));
}
```

**Guava powerSet内部通过Iterator接口进行操作，当请求下一个子集时，会计算并返回该子集。因此，空间复杂度降低到O(n)而不是O(2<sup>n</sup>)**。

但是，Guava是如何实现这一点的呢？

## 4. 幂集生成方法

### 4.1 算法

现在让我们讨论一下为该操作创建算法的可能步骤。

空集的幂集是{{}}，其中它只包含一个空集，所以这是我们最简单的情况。

对于除空集之外的每个集合S，我们首先提取一个元素并将其命名为element。然后，对于集合subsetWithoutElement的其余元素，我们递归计算它们的幂集，并将其命名为powerSetSubsetWithoutElement。然后，通过将提取的元素添加到powerSetSubsetWithoutElement中的所有集合中，我们得到powerSetSubsetWithElement。

现在，幂集S是powerSetSubsetWithoutElement和powerSetSubsetWithElement的并集：

![](/assets/images/2025/algorithms/javapowersetofaset01.png)

让我们看一个给定集合{“APPLE”, “ORANGE”, “MANGO”}的递归幂集堆栈的示例。

为了提高图像的可读性，我们使用名称的缩写形式：P表示幂集函数，“A”、“O”、“M”分别是“APPLE”、“ORANGE”和“MANGO”的缩写形式：

![](/assets/images/2025/algorithms/javapowersetofaset02.png)

### 4.2 实现

因此，首先，让我们编写用于提取一个元素并获取剩余子集的Java代码：

```java
T element = set.iterator().next();
Set<T> subsetWithoutElement = new HashSet<>();
for (T s : set) {
    if (!s.equals(element)) {
        subsetWithoutElement.add(s);
    }
}
```

然后，我们想要获取subsetWithoutElement的幂集：

```java
Set<Set<T>> powersetSubSetWithoutElement = recursivePowerSet(subsetWithoutElement);
```

接下来，我们需要将该幂集添加回原始值中：

```java
Set<Set<T>> powersetSubSetWithElement = new HashSet<>();
for (Set<T> subsetWithoutElement : powerSetSubSetWithoutElement) {
    Set<T> subsetWithElement = new HashSet<>(subsetWithoutElement);
    subsetWithElement.add(element);
    powerSetSubSetWithElement.add(subsetWithElement);
}
```

最后，powerSetSubSetWithoutElement和powerSetSubSetWithElement的并集是给定输入集的幂集：

```java
Set<Set<T>> powerSet = new HashSet<>();
powerSet.addAll(powerSetSubSetWithoutElement);
powerSet.addAll(powerSetSubSetWithElement);
```

如果我们将所有代码片段放在一起，我们就可以看到最终代码：

```java
public Set<Set<T>> recursivePowerSet(Set<T> set) {
    if (set.isEmpty()) {
        Set<Set<T>> ret = new HashSet<>();
        ret.add(set);
        return ret;
    }

    T element = set.iterator().next();
    Set<T> subSetWithoutElement = getSubSetWithoutElement(set, element);
    Set<Set<T>> powerSetSubSetWithoutElement = recursivePowerSet(subSetWithoutElement);
    Set<Set<T>> powerSetSubSetWithElement = addElementToAll(powerSetSubSetWithoutElement, element);

    Set<Set<T>> powerSet = new HashSet<>();
    powerSet.addAll(powerSetSubSetWithoutElement);
    powerSet.addAll(powerSetSubSetWithElement);
    return powerSet;
}
```

### 4.3 单元测试注意事项

现在我们来测试一下，这里有一些标准需要确认：

- 首先，我们检查幂集的大小，对于大小为n的集合，它必须是2<sup>n</sup>。
- 然后，每个元素在一个子集和2<sup>n-1</sup>个不同的子集中只出现一次。
- 最后，每个子集必须出现一次。

如果所有这些条件都满足，我们就可以确保函数有效。现在，由于我们已经使用了Set<Set\>，我们已经知道没有重复。在这种情况下，我们只需要检查幂集的大小，以及子集中每个元素出现的次数。

要检查功率集的大小，我们可以使用：

```java
MatcherAssert.assertThat(powerSet, IsCollectionWithSize.hasSize((1 << set.size())));
```

并检查每个元素出现的次数：

```java
Map<String, Integer> counter = new HashMap<>();
for (Set<String> subset : powerSet) { 
    for (String name : subset) {
        int num = counter.getOrDefault(name, 0);
        counter.put(name, num + 1);
    }
}
counter.forEach((k, v) -> Assertions.assertEquals((1 << (set.size() - 1)), v.intValue()));
```

最后，如果我们可以将所有内容放在一个单元测试中：

```java
@Test
public void givenSet_WhenPowerSetIsCalculated_ThenItContainsAllSubsets() {
    Set<String> set = RandomSetOfStringGenerator.generateRandomSet();
    Set<Set<String>> powerSet = new PowerSet<String>().recursivePowerSet(set);
    MatcherAssert.assertThat(powerSet, IsCollectionWithSize.hasSize((1 << set.size())));
   
    Map<String, Integer> counter = new HashMap<>();
    for (Set<String> subset : powerSet) {
        for (String name : subset) {
            int num = counter.getOrDefault(name, 0);
            counter.put(name, num + 1);
        }
    }
    counter.forEach((k, v) -> Assertions.assertEquals((1 << (set.size() - 1)), v.intValue()));
}
```

## 5. 优化

在本节中，我们将尝试最小化空间并减少内部操作的数量，以最佳方式计算幂集。

### 5.1 数据结构

**正如我们在给定的方法中看到的，我们需要在递归调用中进行大量的减法，这会消耗大量的时间和内存**。

**相反，我们可以将每个集合或子集映射到其他一些概念以减少操作次数**。

首先，我们需要为给定集合S中的每个对象分配一个从0 开始递增的数字，这意味着我们要处理一个有序的数字列表。

例如对于给定集合{“APPLE”, “ORANGE”, “MANGO”}我们得到：

“APPLE” -> 0

“ORANGE” -> 1

“MANGO” -> 2

因此，从现在开始，我们不再生成S的子集，而是为[0,1,2\]的有序列表生成它们，并且由于它是有序的，我们可以通过起始索引模拟减法。

例如，如果起始索引为1，则意味着我们生成[1,2\]的幂集。

为了从对象中检索映射的ID，以及反之，我们需要存储映射的两端。在我们的示例中，我们同时存储了(“MANGO” -> 2)和(2 -> “MANGO”)。由于数字的映射从0开始，因此对于反向映射，我们可以使用一个简单的数组来检索相应的对象。

该函数的可能实现之一是：

```java
private Map<T, Integer> map = new HashMap<>();
private List<T> reverseMap = new ArrayList<>();

private void initializeMap(Collection<T> collection) {
    int mapId = 0;
    for (T c : collection) {
        map.put(c, mapId++);
        reverseMap.add(c);
    }
}
```

现在，为了表示子集，有两个众所周知的想法：

1. 索引表示
2. 二进制表示

### 5.2 索引表示

每个子集由其值的索引表示，例如，给定集合{“APPLE”，“ORANGE”，“MANGO”}的索引映射为：

```text
{
    {} -> {}
    [0] -> {"APPLE"}
    [1] -> {"ORANGE"}
    [0,1] -> {"APPLE", "ORANGE"}
    [2] -> {"MANGO"}
    [0,2] -> {"APPLE", "MANGO"}
    [1,2] -> {"ORANGE", "MANGO"}
    [0,1,2] -> {"APPLE", "ORANGE", "MANGO"}
}
```

因此，我们可以从具有给定映射的索引子集中检索相应的集合：

```java
private Set<Set<T>> unMapIndex(Set<Set<Integer>> sets) {
    Set<Set<T>> ret = new HashSet<>();
    for (Set<Integer> s : sets) {
        HashSet<T> subset = new HashSet<>();
        for (Integer i : s) {
            subset.add(reverseMap.get(i));
        }
        ret.add(subset);
    }
    return ret;
}
```

### 5.3 二进制表示

或者，我们可以用二进制表示每个子集。如果实际集合中的元素存在于该子集中，则其相应值为1；否则为0。

对于我们的水果示例，幂集将是：

```text
{
    [0,0,0] -> {}
    [1,0,0] -> {"APPLE"}
    [0,1,0] -> {"ORANGE"}
    [1,1,0] -> {"APPLE", "ORANGE"}
    [0,0,1] -> {"MANGO"}
    [1,0,1] -> {"APPLE", "MANGO"}
    [0,1,1] -> {"ORANGE", "MANGO"}
    [1,1,1] -> {"APPLE", "ORANGE", "MANGO"}
}
```

因此，我们可以从具有给定映射的二进制子集中检索相应的集合：

```java
private Set<Set<T>> unMapBinary(Collection<List<Boolean>> sets) {
    Set<Set<T>> ret = new HashSet<>();
    for (List<Boolean> s : sets) {
        HashSet<T> subset = new HashSet<>();
        for (int i = 0; i < s.size(); i++) {
            if (s.get(i)) {
                subset.add(reverseMap.get(i));
            }
        }
        ret.add(subset);
    }
    return ret;
}
```

### 5.4 递归算法实现

在此步骤中，我们将尝试使用这两种数据结构来实现前面的代码。

在调用这些函数之前，我们需要调用initializeMap方法来获取有序列表。此外，在创建数据结构之后，我们需要调用相应的unMap函数来检索实际的对象：

```java
public Set<Set<T>> recursivePowerSetIndexRepresentation(Collection<T> set) {
    initializeMap(set);
    Set<Set<Integer>> powerSetIndices = recursivePowerSetIndexRepresentation(0, set.size());
    return unMapIndex(powerSetIndices);
}
```

那么，让我们尝试一下索引表示：

```java
private Set<Set<Integer>> recursivePowerSetIndexRepresentation(int idx, int n) {
    if (idx == n) {
        Set<Set<Integer>> empty = new HashSet<>();
        empty.add(new HashSet<>());
        return empty;
    }
    Set<Set<Integer>> powerSetSubset = recursivePowerSetIndexRepresentation(idx + 1, n);
    Set<Set<Integer>> powerSet = new HashSet<>(powerSetSubset);
    for (Set<Integer> s : powerSetSubset) {
        HashSet<Integer> subSetIdxInclusive = new HashSet<>(s);
        subSetIdxInclusive.add(idx);
        powerSet.add(subSetIdxInclusive);
    }
    return powerSet;
}
```

现在，让我们看看二进制方法：

```java
private Set<List<Boolean>> recursivePowerSetBinaryRepresentation(int idx, int n) {
    if (idx == n) {
        Set<List<Boolean>> powerSetOfEmptySet = new HashSet<>();
        powerSetOfEmptySet.add(Arrays.asList(new Boolean[n]));
        return powerSetOfEmptySet;
    }
    Set<List<Boolean>> powerSetSubset = recursivePowerSetBinaryRepresentation(idx + 1, n);
    Set<List<Boolean>> powerSet = new HashSet<>();
    for (List<Boolean> s : powerSetSubset) {
        List<Boolean> subSetIdxExclusive = new ArrayList<>(s);
        subSetIdxExclusive.set(idx, false);
        powerSet.add(subSetIdxExclusive);
        List<Boolean> subSetIdxInclusive = new ArrayList<>(s);
        subSetIdxInclusive.set(idx, true);
        powerSet.add(subSetIdxInclusive);
    }
    return powerSet;
}
```

### 5.5 迭代[0, 2<sup>n</sup>)

现在，我们可以对二进制表示进行一个很好的优化。如果我们看一下，我们会发现每一行都等同于[0,2<sup>n</sup>)范围内数字的二进制格式。

因此，如果我们遍历从0到2<sup>n</sup>的数字，我们可以将该索引转换为二进制，并使用它来创建每个子集的布尔表示：

```java
private List<List<Boolean>> iterativePowerSetByLoopOverNumbers(int n) {
    List<List<Boolean>> powerSet = new ArrayList<>();
    for (int i = 0; i < (1 << n); i++) {
        List<Boolean> subset = new ArrayList<>(n);
        for (int j = 0; j < n; j++)
            subset.add(((1 << j) & i) > 0);
        powerSet.add(subset);
    }
    return powerSet;
}
```

### 5.6 格雷码的最小变化子集

现在，如果我们定义任何从长度n的二进制表示到[0,2<sup>n</sup>)中的数字的[双射](https://en.wikipedia.org/wiki/Bijection)函数，我们就可以按照我们想要的任意顺序生成子集。

[格雷码](https://en.wikipedia.org/wiki/Gray_code)是一种众所周知的函数，用于生成数字的二进制表示，以便连续数字的二进制表示仅相差一位(即使最后一个数字和第一个数字的差为1)。

因此我们可以进一步优化这一点：

```java
private List<List<Boolean>> iterativePowerSetByLoopOverNumbersWithGrayCodeOrder(int n) {
    List<List<Boolean>> powerSet = new ArrayList<>();
    for (int i = 0; i < (1 << n); i++) {
        List<Boolean> subset = new ArrayList<>(n);
        for (int j = 0; j < n; j++) {
            int grayEquivalent = i ^ (i >> 1);
            subset.add(((1 << j) & grayEquivalent) > 0);
        }
        powerSet.add(subset);
    }
    return powerSet;
}
```

## 6. 延迟加载

为了最小化幂集的空间使用量(即O(2<sup>n</sup>))，我们可以利用Iterator接口来延迟获取每个子集，以及每个子集中的每个元素。

### 6.1 ListIterator

首先，为了能够从0迭代到2<sup>n</sup>，我们应该有一个特殊的Iterator来循环遍历这个范围，但不会事先消耗整个范围。

为了解决这个问题，我们将使用两个变量：一个表示大小，即2<sup>n</sup>，另一个表示当前子集索引。hasNext()函数将检查position是否小于size：

```java
abstract class ListIterator<K> implements Iterator<K> {
    protected int position = 0;
    private int size;
    public ListIterator(int size) {
        this.size = size;
    }
    @Override
    public boolean hasNext() {
        return position < size;
    }
}
```

我们的next()函数返回当前position的子集，并将position值增加1：

```java
@Override
public Set<E> next() {
    return new Subset<>(map, reverseMap, position++);
}
```

### 6.2 Subset

为了实现延迟加载Subset，我们定义了一个扩展AbstractSet的类，并重写了它的一些函数。

通过循环遍历Subset的接收mask(或position)中所有为1的位，我们可以实现Iterator和AbstractSet中的其他方法。

例如，size()是接收mark中1的数量：

```java
@Override
public int size() { 
    return Integer.bitCount(mask);
}
```

而contains()函数只是判断mask中的相应位是否为1：

```java
@Override
public boolean contains(@Nullable Object o) {
    Integer index = map.get(o);
    return index != null && (mask & (1 << index)) != 0;
}
```

我们使用另一个变量remainingSetBits来修改它，每当我们在子集中检索到其相应的元素时，我们就将该位更改为0。然后，hasNext()检查remainingSetBits是否不为0(也就是说，它至少有一个值为1的位)：

```java
@Override
public boolean hasNext() {
    return remainingSetBits != 0;
}
```

并且next()函数使用remainingSetBits中最右边的1，然后将其转换为0，并返回相应的元素：

```java
@Override
public E next() {
    int index = Integer.numberOfTrailingZeros(remainingSetBits);
    if (index == 32) {
        throw new NoSuchElementException();
    }
    remainingSetBits &= ~(1 << index);
    return reverseMap.get(index);
}
```

### 6.3 PowerSet

要拥有延迟加载的PowerSet类，我们需要一个扩展AbstractSet<Set<T\>\>的类。

size()函数只是2的集合大小的幂：

```java
@Override
public int size() {
    return (1 << this.set.size());
}
```

由于幂集将包含输入集的所有可能子集，因此contains(Object o)函数检查对象o的所有元素是否存在于reverseMap(或输入集)中：

```java
@Override
public boolean contains(@Nullable Object obj) {
    if (obj instanceof Set) {
        Set<?> set = (Set<?>) obj;
        return reverseMap.containsAll(set);
    }
    return false;
}
```

要检查给定Object与此类的相等性，我们只能检查输入集是否等于给定的Object：

```java
@Override
public boolean equals(@Nullable Object obj) {
    if (obj instanceof PowerSet) {
        PowerSet<?> that = (PowerSet<?>) obj;
        return set.equals(that.set);
    }
    return super.equals(obj);
}
```

iterator()函数返回我们已经定义的ListIterator实例：

```java
@Override
public Iterator<Set<E>> iterator() {
    return new ListIterator<Set<E>>(this.size()) {
        @Override
        public Set<E> next() {
            return new Subset<>(map, reverseMap, position++);
        }
    };
}
```

**Guava库就使用了这种延迟加载的思想，这些PowerSet和Subset是Guava库的等效实现**。

欲了解更多信息，请查看其[源代码](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Sets.java)和[文档](https://guava.dev/releases/22.0/api/docs/com/google/common/collect/Sets.html)。

此外，如果我们想对PowerSet中的子集进行并行操作，我们可以在ThreadPool中对不同的值调用Subset。

## 7. 总结

总而言之，我们首先学习了什么是幂集。然后，我们使用Guava库生成了幂集。之后，我们学习了幂集的实现方法，以及如何编写单元测试。

最后，我们利用Iterator接口来优化子集的生成空间及其内部元素。