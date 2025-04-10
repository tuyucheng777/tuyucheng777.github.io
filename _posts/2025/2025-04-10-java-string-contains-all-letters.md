---
layout: post
title:  使用Java检查字符串是否包含字母表的所有字母
category: algorithms
copyright: algorithms
excerpt: 字符串
---

## 1. 概述

在本教程中，我们将了解如何检查字符串是否包含字母表的所有字母。

这里有一个简单的例子：“Farmer jack realized that big yellow quilts were expensive.”-它实际上包含了字母表的所有字母。

我们将讨论三种方法。

首先，我们将使用命令式方法对算法进行建模，然后使用正则表达式，最后，我们将利用Java 8的更具声明式的方法。

## 2. 命令式算法

让我们实现一个命令式算法，首先，我们将创建一个布尔数组，表示“已访问”。然后，我们将逐个字符地遍历输入字符串，并将该字符标记为已访问。

请注意，大写和小写被视为相同。因此索引0表示A和a，同样，索引25表示Z和z。

最后，我们检查访问数组中的所有字符是否都设置为true：
```java
public class EnglishAlphabetLetters {

    public static boolean checkStringForAllTheLetters(String input) {
        int index = 0;
        boolean[] visited = new boolean[26];

        for (int id = 0; id < input.length(); id++) {
            if ('a' <= input.charAt(id) && input.charAt(id) <= 'z') {
                index = input.charAt(id) - 'a';
            } else if ('A' <= input.charAt(id) && input.charAt(id) <= 'Z') {
                index = input.charAt(id) - 'A';
            }
            visited[index] = true;
        }

        for (int id = 0; id < 26; id++) {
            if (!visited[id]) {
                return false;
            }
        }
        return true;
    }
}
```

该程序的复杂度为O(n)，其中n是字符串的长度。

请注意，有很多方法可以优化算法，例如从集合中删除字母，并在集合为空时立即中断。不过，就练习而言，该算法已经足够好了。

## 3. 使用正则表达式

使用正则表达式，我们可以轻松地用几行代码获得相同的结果：
```java
public static boolean checkStringForAllLetterUsingRegex(String input) {
    return input.toLowerCase()
            .replaceAll("[^a-z]", "")
            .replaceAll("(.)(?=.*\\1)", "")
            .length() == 26;
}
```

这里，我们首先从输入中删除除字母之外的所有字符，然后我们删除重复的字符。最后，我们统计字母并确保拥有所有字母，即26个。

虽然性能较差，但这种方法的复杂度也趋向于O(n)。

## 4. Java 8 Stream

使用Java 8特性，我们可以使用Stream的filter和distinct方法以更紧凑、更具声明式的方式轻松实现相同的结果：
```java
public static boolean checkStringForAllLetterUsingStream(String input) {
    long c = input.toLowerCase().chars()
      .filter(ch -> ch >= 'a' && ch <= 'z')
      .distinct()
      .count();
    return c == 26;
}
```

这种方法的复杂度也将是O(n)。

## 5. 测试

让我们测试一下我们的算法的快乐路径：
```java
@Test
void givenString_whenContainsAllCharacter_thenTrue() {
    String sentence = "Farmer jack realized that big yellow quilts were expensive";
    assertTrue(EnglishAlphabetLetters.checkStringForAllTheLetters(sentence));
}
```

这里，sentence包含字母表的所有字母，因此，我们期望结果为true。

## 6. 总结

**在本教程中，我们介绍了如何检查字符串是否包含字母表的所有字母**。

我们看到了几种使用传统命令式编程、正则表达式和Java 8 Stream来实现这一点的方法。