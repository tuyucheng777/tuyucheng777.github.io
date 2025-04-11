---
layout: post
title:  在Java中检查两个字符串是否是彼此的排列
category: algorithms
copyright: algorithms
excerpt: 排列
---

## 1. 概述

排列或字谜是通过重新排列不同单词或短语的字母而形成的单词或短语，换句话说，**排列包含与另一个字符串相同的字符，但字符的排列顺序可以变化**。

在本教程中，我们将研究解决方案以检查一个字符串是否是另一个字符串的排列或字谜。

## 2. 什么是字符串排列？

让我们看一个排列示例，看看如何开始思考解决方案。

### 2.1 排列

让我们看一下单词CAT的排列：

![](/assets/images/2025/algorithms/javacheckpermutationstwostrings01.png)

值得注意的是，有6种排列(包括CAT本身)，我们可以计算n!(n的阶乘)，其中n是字符串的长度。

### 2.2 如何找到解决方案

我们看到，一个字符串有很多种可能的排列，我们可以考虑创建一个算法，循环遍历字符串的所有排列，检查其中一个是否与我们要比较的字符串相等。

这是可行的，但我们必须首先创建所有可能的排列，这可能很昂贵，尤其是对于较大的字符串。

**然而，我们注意到，无论排列如何，它都包含与原始相同的字符。例如，CAT和TAC具有相同的字符[A,C,T\]，即使顺序不同**。

因此，我们可以将字符串的字符存储在数据结构中，然后找到一种方法来比较它们。

值得注意的是，如果两个字符串的长度不同，我们可以说它们不是排列。

## 3. 排序

解决这个问题最直接的方法是对两个字符串进行[排序](https://www.baeldung.com/java-sorting-arrays)和比较。

让我们看一下算法：
```java
boolean isPermutationWithSorting(String s1, String s2) {
    if (s1.length() != s2.length()) {
        return false;
    }
    char[] s1CharArray = s1.toCharArray();
    char[] s2CharArray = s2.toCharArray();
    Arrays.sort(s1CharArray);
    Arrays.sort(s2CharArray);
    return Arrays.equals(s1CharArray, s2CharArray);
}
```

值得注意的是，字符串是原始字符的数组，因此，我们将对两个字符串的字符数组进行排序，并使用equals()方法进行比较。

我们来看看算法复杂度：

- 时间复杂度：排序为O(n log n)
- 空间复杂度：O(n)排序辅助空间

现在我们可以看一些测试，检查一个简单单词和一个句子的排列：
```java
@Test
void givenTwoStringsArePermutation_whenSortingCharsArray_thenPermutation() {
    assertTrue(isPermutationWithSorting("tuyucheng", "tyuuhceng"));
    assertTrue(isPermutationWithSorting("hello world", "world hello"));
}
```

并测试字符不匹配和字符串长度不同的负面情况：
```java
@Test
void givenTwoStringsAreNotPermutation_whenSortingCharsArray_thenNotPermutation() {
    assertFalse(isPermutationWithSorting("tuyucheng", "tyuuhcenv"));
    assertFalse(isPermutationWithSorting("tuyucheng", "tyuuceng"));
}
```

## 4. 计数频率

如果我们将单词视为有限个字符，则可以将它们的频率放在一个相同维度的数组中。我们可以更通用地考虑字母表中的26个字母或扩展的[ASCII](https://www.ascii-code.com/)编码，数组中最多有256个位置。

### 4.1 两个计数器

我们可以使用两个计数器，每个单词一个：
```java
boolean isPermutationWithTwoCounters(String s1, String s2) {
    if (s1.length() != s2.length()) {
        return false;
    }
    int[] counter1 = new int[256];
    int[] counter2 = new int[256];
    for (int i = 0; i < s1.length(); i++) {
        counter1[s1.charAt(i)]++;
    }

    for (int i = 0; i < s2.length(); i++) {
        counter2[s2.charAt(i)]++;
    }

    return Arrays.equals(counter1, counter2);
}
```

保存每个计数器中的频率后，我们可以检查计数器是否相等。

我们来看看算法复杂度：

- 时间复杂度：访问计数器的时间为O(n)
- 空间复杂度：计数器的常数大小为O(1)

接下来我们看一下正面和负面的测试用例：
```java
@Test
void givenTwoStringsArePermutation_whenTwoCountersCharsFrequencies_thenPermutation() {
    assertTrue(isPermutationWithTwoCounters("tuyucheng", "tyuuhceng"));
    assertTrue(isPermutationWithTwoCounters("hello world", "world hello"));
}

@Test
void givenTwoStringsAreNotPermutation_whenTwoCountersCharsFrequencies_thenNotPermutation() {
    assertFalse(isPermutationWithTwoCounters("tuyucheng", "tyuuhcenv"));
    assertFalse(isPermutationWithTwoCounters("tuyucheng", "tyuuceng"));
}
```

### 4.2 一个计数器

我们可以更简单，只使用一个计数器：
```java
boolean isPermutationWithOneCounter(String s1, String s2) {
    if (s1.length() != s2.length()) {
        return false;
    }
    int[] counter = new int[256];
    for (int i = 0; i < s1.length(); i++) {
        counter[s1.charAt(i)]++;
        counter[s2.charAt(i)]--;
    }
    for (int count : counter) {
        if (count != 0) {
            return false;
        }
    }
    return true;
}
```

我们仅使用一个计数器在同一个循环中添加和删除频率，因此，我们最终需要检查所有频率是否都等于0。

我们来看看算法复杂度：

- 时间复杂度：访问计数器的时间为O(n)
- 空间复杂度：O(1)计数器的常数大小

让我们再看一些测试：
```java
@Test
void givenTwoStringsArePermutation_whenOneCounterCharsFrequencies_thenPermutation() {
    assertTrue(isPermutationWithOneCounter("tuyucheng", "tyuuhceng"));
    assertTrue(isPermutationWithOneCounter("hello world", "world hello"));
}

@Test
void givenTwoStringsAreNotPermutation_whenOneCounterCharsFrequencies_thenNotPermutation() {
    assertFalse(isPermutationWithOneCounter("tuyucheng", "tyuuhcenv"));
    assertFalse(isPermutationWithOneCounter("tuyucheng", "tyuuceng"));
}
```

### 4.3 使用HashMap

我们可以使用[Map](https://www.baeldung.com/java-hashmap)而不是数组来计算频率，思路是一样的，但使用Map可以存储更多字符，**这有助于处理[Unicode](https://en.wikipedia.org/wiki/List_of_Unicode_characters)字符，例如包括东欧、非洲、亚洲或表情符号**。

让我们看一下算法：
```java
boolean isPermutationWithMap(String s1, String s2) {
    if (s1.length() != s2.length()) {
        return false;
    }
    Map<Character, Integer> charsMap = new HashMap<>();

    for (int i = 0; i < s1.length(); i++) {
        charsMap.merge(s1.charAt(i), 1, Integer::sum);
    }

    for (int i = 0; i < s2.length(); i++) {
        if (!charsMap.containsKey(s2.charAt(i)) || charsMap.get(s2.charAt(i)) == 0) {
            return false;
        }
        charsMap.merge(s2.charAt(i), -1, Integer::sum);
    }

    return true;
}
```

一旦我们知道了字符串中字符的频率，我们就可以检查另一个字符串是否包含所有所需的匹配。

值得注意的是，在使用Map时，我们不会像前面的示例那样按位置比较数组。因此，我们已经知道Map中的键是否不存在或频率值是否对应。

我们来看看算法复杂度：

- 时间复杂度：O(n)，以恒定时间访问Map
- 空间复杂度：O(m)，其中m是我们存储在Map中的字符数，具体取决于字符串的复杂度

为了测试，这次我们还可以添加非ASCII字符：
```java
@Test
void givenTwoStringsArePermutation_whenCountCharsFrequenciesWithMap_thenPermutation() {
    assertTrue(isPermutationWithMap("tuyucheng", "tyuuhceng"));
    assertTrue(isPermutationWithMap("hello world", "world hello"));
}
```

对于负面情况，为了获得100% 的测试覆盖率，我们必须考虑映射不包含键或值不匹配的情况：
```java
@Test
void givenTwoStringsAreNotPermutation_whenCountCharsFrequenciesWithMap_thenNotPermutation() {
    assertFalse(isPermutationWithMap("tuyucheng", "tyuuceng"));
    assertFalse(isPermutationWithMap("tuyucheng", "tyuuhcenv"));
    assertFalse(isPermutationWithMap("tuyucheng", "tyuucenx"));
}
```

## 5. 包含排列的字符串

如果我们想检查一个字符串是否包含另一个字符串作为排列，该怎么办？例如，我们可以看到字符串acab包含ba作为排列；ab是ba的排列，并且从第3个字符开始包含在acab中。

我们可以遍历前面讨论过的所有排列，但这样做效率不高。计算频率也行不通，例如，检查cb是否是acab的排列包含会导致假阳性。

**但是，我们仍然可以使用计数器作为起点，然后使用滑动窗口技术检查排列**。

我们对每个字符串使用一个频率计数器，首先，我们将潜在排列的计数器加起来。然后，我们循环第二个字符串的字符并遵循以下模式：

- 将新的后续字符添加到考虑的新窗口
- 超出窗口长度时删除前一个字符

窗口的长度等于潜在排列字符串的长度，我们用图来表示一下：

![](/assets/images/2025/algorithms/javacheckpermutationstwostrings02.png)

让我们看一下算法，s2是我们要检查包含性的字符串，此外，为了简单起见，我们将范围写缩小到字母表中的26个字符：
```java
boolean isPermutationInclusion(String s1, String s2) {
    int ns1 = s1.length(), ns2 = s2.length();
    if (ns1 < ns2) {
        return false;
    }

    int[] s1Count = new int[26];
    int[] s2Count = new int[26];

    for (char ch : s2.toCharArray()) {
        s2Count[ch - 'a']++;
    }

    for (int i = 0; i < ns1; ++i) {
        s1Count[s1.charAt(i) - 'a']++;
        if (i >= ns2) {
            s1Count[s1.charAt(i - ns2) - 'a']--;
        }
        if (Arrays.equals(s1Count, s2Count)) {
            return true;
        }
    }

    return false;
}
```

让我们看一下应用滑动窗口的最后一个循环，**我们增加了频率计数器，但是，当窗口超出排列长度时，我们会删除该字符的出现**。

值得注意的是，如果字符串的长度相同，我们就会陷入之前见过的字谜情况。

让我们看看算法的复杂度，设l1和l2分别为字符串s1和s2的长度：

- 时间复杂度：O(l1+ 26 * (l2 - l1))，取决于字符串和字符范围之间的差异
- 空间复杂度：O(1)常数空间用于跟踪频率

让我们看一些检查完全匹配或排列的积极测试用例：
```java
@Test
void givenTwoStrings_whenIncludePermutation_thenPermutation() {
    assertTrue(isPermutationInclusion("tuyucheng", "yu"));
    assertTrue(isPermutationInclusion("tuyucheng", "uy"));
}
```

我们来看一些负面测试案例：
```java
@Test
void givenTwoStrings_whenNotIncludePermutation_thenNotPermutation() {
    assertFalse(isPermutationInclusion("tuyucheng", "au"));
    assertFalse(isPermutationInclusion("tuyucheng", "tuyuchenga"));
}
```

## 6. 总结

在本文中，我们了解了一些用于检查一个字符串是否为另一个字符串的排列的算法。对字符串字符进行排序和比较是一种简单的解决方案；更有趣的是，我们可以通过将字符频率存储在计数器中并进行比较来实现线性复杂度，我们也可以使用Map而不是频率数组来获得相同的结果。最后，我们还看到了该问题的变体，即使用滑动窗口来检查一个字符串是否包含另一个字符串的排列。