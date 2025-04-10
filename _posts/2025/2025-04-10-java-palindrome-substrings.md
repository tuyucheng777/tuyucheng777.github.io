---
layout: post
title:  在Java中查找回文子字符串
category: algorithms
copyright: algorithms
excerpt: 回文字符串
---

## 1. 概述

在本快速教程中，我们将介绍不同的方法来查找给定字符串中的所有回文子字符串，我们还会记录每种方法的时间复杂度。

## 2. 暴力破解法

在这种方法中，我们只需遍历输入字符串即可找到所有子字符串。同时，我们将检查子字符串是否是回文：
```java
public Set<String> findAllPalindromesUsingBruteForceApproach(String input) {
    Set<String> palindromes = new HashSet<>();
    for (int i = 0; i < input.length(); i++) {
        for (int j = i + 1; j <= input.length(); j++) {
            if (isPalindrome(input.substring(i, j))) {
                palindromes.add(input.substring(i, j));
            }
        }
    }
    return palindromes;
}
```

在上面的例子中，我们只需将子字符串与其反向进行比较，看看它是否是回文：
```java
private boolean isPalindrome(String input) {
    StringBuilder plain = new StringBuilder(input);
    StringBuilder reverse = plain.reverse();
    return (reverse.toString()).equals(input);
}
```

当然，我们可以从[其他几种方法](https://www.baeldung.com/java-palindrome)中轻松选择。

**该方法的时间复杂度为O(n^3)**，虽然这对于较小的输入字符串来说可能是可以接受的，但如果我们要检查大量文本中的回文，就需要一种更有效的方法。

## 3. 集中化方法

**中心化方法的思想是将每个字符视为枢轴，并向两个方向扩展以找到回文**。

只有当左右两侧的字符匹配时，我们才会扩展，这样字符串才符合回文条件。否则，我们继续处理下一个字符。

让我们看一个简单的演示，其中我们将每个字符视为回文的中心：
```java
public Set<String> findAllPalindromesUsingCenter(String input) {
    Set<String> palindromes = new HashSet<>();
    for (int i = 0; i < input.length(); i++) {
        palindromes.addAll(findPalindromes(input, i, i + 1));
        palindromes.addAll(findPalindromes(input, i, i));
    }
    return palindromes;
}
```

在上面的循环中，我们在两个方向上扩展以获得以每个位置为中心的所有回文的集合。通过在循环中两次调用findPalindromes方法，我们将找到偶数长度和奇数长度的回文：
```java
private Set<String> findPalindromes(String input, int low, int high) {
    Set<String> result = new HashSet<>();
    while (low >= 0 && high < input.length() && input.charAt(low) == input.charAt(high)) {
        result.add(input.substring(low, high + 1));
        low--;
        high++;
    }
    return result;
}
```

**这种方法的时间复杂度为O(n^2)**，这比我们的暴力破解方法有所改进，但我们可以做得更好，我们将在下一节中看到。

## 4. Manacher算法

**[Manacher算法](https://www.baeldung.com/cs/manachers-algorithm)可以在线性时间内找到最长的回文子串**，我们将使用此算法查找所有回文子串。

在深入研究算法之前，我们将初始化一些变量。

首先，在将结果字符串转换为字符数组之前，我们将在输入字符串的开头和结尾使用边界字符进行保护：
```java
String formattedInput = "@" + input + "#";
char inputCharArr[] = formattedInput.toCharArray();
```

然后，我们将使用具有两行的二维数组radius-一行用于存储奇数长度回文的长度，另一行用于存储偶数长度回文的长度：
```java
int radius[][] = new int[2][input.length() + 1];
```

接下来，我们将遍历输入数组以找到以位置i为中心的回文的长度，并将该长度存储在radius[][]中：
```java
Set<String> palindromes = new HashSet<>();
int max;
for (int j = 0; j <= 1; j++) {
    radius[j][0] = max = 0;
    int i = 1;
    while (i <= input.length()) {
        palindromes.add(Character.toString(inputCharArr[i]));
        while (inputCharArr[i - max - 1] == inputCharArr[i + j + max])
            max++;
        radius[j][i] = max;
        int k = 1;
        while ((radius[j][i - k] != max - k) && (k < max)) {
            radius[j][i + k] = Math.min(radius[j][i - k], max - k);
            k++;
        }
        max = Math.max(max - k, 0);
        i += k;
    }
}
```

**该方法的时间复杂度为O(n)**，最后，我们遍历数组radius[][]来计算以每个位置为中心的回文子串：
```java
for (int i = 1; i <= input.length(); i++) {
    for (int j = 0; j <= 1; j++) {
        for (max = radius[j][i]; max > 0; max--) {
            palindromes.add(input.substring(i - max - 1, max + j + i - 1));
        }
    }
}
```

**虽然Manacher算法的时间复杂度为O(n)，但将所有回文串放入Set的逻辑复杂度为O(n²)**，这是因为[substring](https://www.baeldung.com/string/substring)方法每次都会返回一个新字符串，其性能损失与低效的[字符串拼接](https://www.baeldung.com/java-string-performance#sb-1)类似。

## 5. 总结

在这篇简短的文章中，我们讨论了查找回文子串的不同方法的时间复杂度。