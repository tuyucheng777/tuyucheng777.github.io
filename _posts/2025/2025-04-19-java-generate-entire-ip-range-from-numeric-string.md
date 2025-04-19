---
layout: post
title:  在Java中根据给定的数字字符串生成所有可能的IP地址
category: algorithms
copyright: algorithms
excerpt: IP地址
---

## 1. 概述

在本教程中，我们将探索在Java中根据给定的数字字符串生成所有可能的IP地址的不同方法，这个问题是一道常见的面试题。我们将首先深入探讨问题描述，然后介绍几种解决方法，包括暴力破解、回溯和动态规划。

## 2. 理解问题陈述

**给定一个仅包含数字的字符串S，目标是找出并返回所有可以通过在S中插入点号(.)来构成的有效IP地址。我们不能删除或重新排序任何数字；结果可以按任意顺序返回**。

例如，对于给定的字符串“101024”，我们将得到以下输出：

```text
["1.0.10.24","1.0.102.4","10.1.0.24","10.10.2.4","101.0.2.4"]
```

上面的输出列出了给定字符串可以构成的所有有效IP地址，**有效的IPv4地址由4个八位字节组成，每个八位字节之间用点分隔。每个八位字节都是一个介于0到255之间的数字，这意味着有效的IPv4地址如下所示**：

```text
0.0.0.0
192.168.0.8
234.223.43.42
255.255.255.0
```

## 3. 迭代方法

在这个方法中，我们将使用3个嵌套循环来遍历将字符串拆分成4个部分的可能位置。外层循环遍历第一部分，中间层循环遍历第二部分，内层循环遍历第三部分，我们将字符串的剩余部分视为第四个部分。

**一旦我们获得了段组合，我们将提取子字符串并使用isValid()方法验证每个段是否构成IP地址的有效部分**。isValid()方法负责检查给定字符串是否是IP地址的有效部分，它确保整数值介于0到255之间，并且没有前导0：

```java
public List<String> restoreIPAddresses(String s) {
    List<String> result = new ArrayList<>();

    if (s.length() < 4 || s.length() > 12) {
        return result;
    }

    for (int i = 1; i < 4; i++) {
        for (int j = i + 1; j < Math.min(i + 4, s.length()); j++) {
            for (int k = j + 1; k < Math.min(j + 4, s.length()); k++) {
                String part1 = s.substring(0, i);
                String part2 = s.substring(i, j);
                String part3 = s.substring(j, k);
                String part4 = s.substring(k);

                if (isValid(part1) && isValid(part2) && isValid(part3) && isValid(part4)) {
                    result.add(part1 + "." + part2 + "." + part3 + "." + part4);
                }
            }
        }
    }
    return result;
}

private boolean isValid(String part) {
    if (part.length() > 1 && part.startsWith("0")) {        return false;    }
    int num = Integer.parseInt(part);
    return num >= 0 && num <= 255;
}
```

restoreIPAddresses()方法生成并返回所有可能的有效IP地址，它还会检查字符串长度是否超出4到12个字符的范围，如果超出则返回一个空列表，因为在这种情况下无法生成有效的IP地址。 

**我们使用嵌套循环来迭代这3个点的所有可能位置，这会使[时间复杂度](https://www.baeldung.com/cs/big-oh-asymptotic-complexity#constant-time-algorithms)增加到O(n²)。它不是O(n³)，因为外层循环的迭代次数是固定的。此外，我们在每次迭代中都会创建临时子字符串，这会使整体空间复杂度增加到O(n)**。

## 4. 回溯法

[回溯](https://www.baeldung.com/cs/ipv4-vs-ipv6)时，我们将字符串拆分成四部分，生成所有可能的IP地址。每一步，我们都使用相同的isValid()方法执行验证，检查字符串是否非空且不包含前导0。如果是有效字符串，则进入下一部分。完成所有四部分并使用所有字符后，我们会将它们添加到结果列表中：

```java
public List<String> restoreIPAddresses(String s) {
    List<String> result = new ArrayList<>();
    backtrack(result, s, new ArrayList<>(), 0);
    return result;
}

private void backtrack(List<String> result, String s, List<String> current, int index) {
    if (current.size() == 4) {
        if (index == s.length()) {
            result.add(String.join(".", current));
        }
        return;
    }

    for (int len = 1; len <= 3; len++) {
        if (index + len > s.length()) {            break;
        }
        String part = s.substring(index, index + len);
        if (isValid(part)) {
            current.add(part);
            backtrack(result, s, current, index + len);
            current.remove(current.size() - 1);
        }
    }
}
```

**backtrack函数将字符串拆分成4个整数段，每个段最多包含3位数字。这样一来，就有3<sup>4</sup>种可能的拆分方式。评估每个段是否有效需要O(1)的时间复杂度，因为它需要检查段的长度和范围。由于可能的配置数量3<sup>4</sup>是常数，并且与输入字符串的长度无关，因此时间复杂度简化为O(1)**。

由于IP地址恰好有4个段，因此递归深度限制为4。**由于配置数量和所需空间为常数，因此空间复杂度简化为O(1)**。

## 5. 动态规划方法

[动态规划](https://www.baeldung.com/cs/greedy-approach-vs-dynamic-programming#dynamic-programming)是一种流行的算法范式，它使用递归公式来求解。它类似于分治策略，因为它将问题分解为更小的子问题，其解决方案使用数组来存储先前计算值的结果。

现在，我们将使用dp数组来存储所有有效的IP前缀。首先初始化一个二维列表dp，其中dp[i\][j\]存储所有可能的IP地址段，直到第i个字符，共j个段。之后，我们检查所有可能的段长度(1到3)，并使用isValid()方法验证每个段。如果某个段有效，则将其附加到前一个状态的前缀中，我们将有效IP地址的最终结果存储并返回到dp[n\][4\]中。

```java
public List<String> restoreIPAddresses(String s) {
    int n = s.length();
    if (n < 4 || n > 12) {        return new ArrayList<>();
    }
    List<String>[][] dp = new ArrayList[n + 1][5];
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= 4; j++) {
            dp[i][j] = new ArrayList<>();
        }
    }

    dp[0][0].add("");

    for (int i = 1; i <= n; i++) {
        for (int k = 1; k <= 4; k++) {
            for (int j = 1; j <= 3 && i - j >= 0; j++) {
                String segment = s.substring(i - j, i);
                if (isValid(segment)) {
                    for (String prefix : dp[i - j][k - 1]) {
                        dp[i][k].add(prefix.isEmpty() ? segment : prefix + "." + segment);
                    }
                }
            }
        }
    }

    return dp[n][4];
}
```

**初始化dp数组需要O(n\*4)的时间复杂度，简化为O(n)。负责填充dp表的嵌套循环需要O(n\*4\*3) 的时间复杂度，简化为O(n)。空间复杂度也是O(n\*4)，简化为O(n)**。

## 6. 总结

在本文中，我们讨论了如何根据给定的数字字符串生成所有可能的IP地址组合，并了解了各种解决方法。迭代方法简单易懂，但时间复杂度较高，为O(n²)。为了获得更好的性能，我们可以使用动态规划方法，其时间复杂度仅为O(n)。

**回溯编程方法被证明是最有效的方法，因为在我们讨论的所有其他方法中，它具有最佳的运行时间O(1)**。