---
layout: post
title:  Java中的平衡括号算法
category: algorithms
copyright: algorithms
excerpt: 平衡括号算法
---

## 1. 概述

平衡括号，也称为平衡圆括号，是一个常见的编程问题。

在本教程中，我们将验证给定字符串中的括号是否平衡。

这种类型的字符串是所谓的[Dyck语言](https://en.wikipedia.org/wiki/Dyck_language)的一部分。

## 2. 问题陈述

括号被视为以下任意字符-“(”， “)”，“\[”， “\]”，“{“， “}”。

如果左括号“(”、“\[”和“{”分别出现在相应的右括号“)”、“\]”和“}”的左侧，则一组括号被视为匹配对。

但是，**如果包含括号对的字符串中的括号集不匹配，则该字符串不平衡**。

类似地，**包含非括号字符(如az、AZ、0-9)或其他特殊字符(如#、$、@)的字符串也被视为不平衡**。

例如，如果输入为“{[(\])}”，则方括号“[\]”包含一个不平衡的左圆括号“(”。类似地，圆括号“()”包含一个不平衡的右方括号“\]”。因此，输入字符串“{[(\])}”是不平衡的。

因此，如果满足以下条件，则包含括号字符的字符串被认为是平衡的：

1. 匹配的左括号出现在每个对应的右括号的左侧
2. 括号内的括号也是平衡的
3. 它不包含任何非括号字符

有几个特殊情况需要记住：**null被认为是不平衡的，而空字符串被认为是平衡的**。

为了进一步说明平衡括号的定义，我们来看一些平衡括号的例子：
```text
()
[()]
{[()]}
([{{[(())]}}])
```

这是不平衡的例子：
```text
abc[](){}
{{[]()}}}}
{[(])}
```

现在我们对问题有了更好的了解，让我们看看如何解决它。

## 3. 解决方案

解决这个问题的方法有很多种，在本教程中，我们将介绍两种方法：

1. 使用String类的方法
2. 使用Deque实现

## 4. 基本设置和验证

让我们首先创建一个方法，如果输入平衡则返回true，如果输入不平衡则返回false：
```java
public boolean isBalanced(String str)
```

让我们考虑一下输入字符串的基本验证：

1. 如果传递了null输入，则它是不平衡的。
2. 字符串要达到平衡，其左右括号必须匹配。因此，可以肯定地说，长度为奇数的输入字符串将不平衡，因为它至少包含一个不匹配的括号。
3. 根据问题陈述，应该检查括号之间的平衡行为。因此，任何包含非括号字符的输入字符串都是不平衡字符串。

根据这些规则，我们可以实现验证：
```java
if (null == str || ((str.length() % 2) != 0)) {
    return false;
} else {
    char[] ch = str.toCharArray();
    for (char c : ch) {
        if (!(c == '{' || c == '[' || c == '(' || c == '}' || c == ']' || c == ')')) {
            return false;
        }
    }
}
```

现在输入字符串已经验证，我们可以继续解决这个问题。

## 5. 使用String.replaceAll方法

在这种方法中，我们将循环遍历输入字符串，使用[String.replaceAll](https://www.baeldung.com/java-remove-replace-string-part#string-api)从字符串中删除“()”、“[\]”和“{}”的出现，我们继续此过程，直到在输入字符串中找不到任何其他出现。

该过程完成后，如果字符串的长度为0，则所有匹配的括号对都已被删除，输入字符串是平衡的。但是，如果长度不为0，则字符串中仍存在一些不匹配的左括号或右括号。因此，输入字符串是不平衡的。

我们来看看完整的实现：
```java
while (str.contains("()") || str.contains("[]") || str.contains("{}")) {
    str = str.replaceAll("\\(\\)", "")
        .replaceAll("\\[\\]", "")
        .replaceAll("\\{\\}", "");
}
return (str.length() == 0);
```

## 6. 使用Deque

[Deque](https://www.baeldung.com/java-queue#3-deques)是队列的一种形式，可在队列的两端提供添加、检索和获取操作，我们将利用此数据结构的[后进先出(LIFO)](https://www.baeldung.com/java-lifo-thread-safe)顺序特性来检查输入字符串中的余额。

首先，让我们构建我们的Deque：
```java
Deque<Character> deque = new LinkedList<>();
```

请注意，我们在这里使用了[LinkedList](https://www.baeldung.com/java-linkedlist)，因为它为Deque接口提供了实现。

现在我们的双端队列已经构造好了，我们将逐个循环输入字符串的每个字符，如果字符是左括号，则将其添加为Deque的第一个元素：
```java
if (ch == '{' || ch == '[' || ch == '(') { 
    deque.addFirst(ch); 
}
```

但是，如果字符是右括号，那么我们将对LinkedList执行一些检查。

首先，我们检查LinkedList是否为空，空列表意味着右括号不匹配。因此，输入字符串不平衡，所以我们返回false。

但是，如果LinkedList不为空，则我们使用peekFirst方法查看其最后一个字符，如果它可以与右括号配对，则我们使用removeFirst方法从列表中删除这个最顶端的字符，然后继续循环的下一次迭代：
```java
if (!deque.isEmpty() 
    && ((deque.peekFirst() == '{' && ch == '}') 
    || (deque.peekFirst() == '[' && ch == ']') 
    || (deque.peekFirst() == '(' && ch == ')'))) { 
    deque.removeFirst(); 
} else { 
    return false; 
}
```

循环结束时，所有字符都经过了平衡检查，因此我们可以返回true。以下是基于Deque方法的完整实现：
```java
Deque<Character> deque = new LinkedList<>();
for (char ch: str.toCharArray()) {
    if (ch == '{' || ch == '[' || ch == '(') {
        deque.addFirst(ch);
    } else {
        if (!deque.isEmpty()
            && ((deque.peekFirst() == '{' && ch == '}')
            || (deque.peekFirst() == '[' && ch == ']')
            || (deque.peekFirst() == '(' && ch == ')'))) {
            deque.removeFirst();
        } else {
            return false;
        }
    }
}
return deque.isEmpty();
```

## 7. 总结

在本教程中，我们讨论了平衡括号的问题陈述并使用两种不同的方法解决了它。