---
layout: post
title:  查找不包含重复字符的最长子字符串
category: algorithms
copyright: algorithms
excerpt: 字符串
---

## 1. 概述

在本教程中，我们将比较使用Java查找最长唯一字母子字符串的方法。例如，“CODINGISAWESOME”中最长唯一字母子字符串是“NGISAWE”。

## 2. 暴力破解法

让我们从一个简单的方法开始，首先，**我们可以检查每个子字符串是否包含唯一字符**：
```java
String getUniqueCharacterSubstringBruteForce(String input) {
    String output = "";
    for (int start = 0; start < input.length(); start++) {
        Set<Character> visited = new HashSet<>();
        int end = start;
        for (; end < input.length(); end++) {
            char currChar = input.charAt(end);
            if (visited.contains(currChar)) {
                break;
            } else {
                visited.add(currChar);
            }
        }
        if (output.length() < end - start + 1) {
            output = input.substring(start, end);
        }
    }
    return output;
}
```

由于有n * (n + 1) / 2个可能的子字符串，因此**该方法的时间复杂度为O(n^2)**。

## 3. 优化方法

现在，让我们看一个优化的方法，我们从左到右开始遍历字符串，并跟踪以下内容：

1. 借助start和end索引，找到不重复字符的当前子字符串
2. 最长不重复子串输出
3. **已访问字符的查找表**
```java
String getUniqueCharacterSubstring(String input) {
    Map<Character, Integer> visited = new HashMap<>();
    String output = "";
    for (int start = 0, end = 0; end < input.length(); end++) {
        char currChar = input.charAt(end);
        if (visited.containsKey(currChar)) {
            start = Math.max(visited.get(currChar)+1, start);
        }
        if (output.length() < end - start + 1) {
            output = input.substring(start, end + 1);
        }
        visited.put(currChar, end);
    }
    return output;
}
```

对于每个新字符，我们都会在已访问的字符中查找，如果该字符已被访问过并且是当前子字符串中不包含重复字符的一部分，则更新起始索引；否则，我们将继续遍历字符串。

由于我们只遍历字符串一次，因此**时间复杂度将是线性的，即O(n)**。

**这种方法也称为[滑动窗口模式](https://www.baeldung.com/cs/sliding-window-algorithm)**。

## 4. 测试

最后，让我们测试我们的实现以确保它有效：
```java
@Test
void givenString_whenGetUniqueCharacterSubstringCalled_thenResultFoundAsExpected() {
    assertEquals("", getUniqueCharacterSubstring(""));
    assertEquals("A", getUniqueCharacterSubstring("A"));
    assertEquals("ABCDEF", getUniqueCharacterSubstring("AABCDEF"));
    assertEquals("ABCDEF", getUniqueCharacterSubstring("ABCDEFF"));
    assertEquals("NGISAWE", getUniqueCharacterSubstring("CODINGISAWESOME"));
    assertEquals("be coding", getUniqueCharacterSubstring("always be coding"));
}
```

在这里，我们测试了边界条件以及更典型的用例。

## 5. 总结

在本教程中，我们学习了如何使用滑动窗口技术来查找具有非重复字符的最长子字符串。