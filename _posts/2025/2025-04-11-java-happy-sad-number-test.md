---
layout: post
title:  在Java中检查数字是否为快乐数字
category: algorithms
copyright: algorithms
excerpt: 快乐数字
---

## 1. 概述

我们在编程中经常解决数学问题，判断一个数字是否是快乐数字是一项有趣的任务。

在本教程中，我们将了解如何定义快乐数字，并探索如何实现Java程序来检查给定的数字是否是快乐数字。

## 2. 理解快乐数字

**快乐数通过用其各位数字的平方和反复替换，最终达到1**。相反，非快乐(悲伤)数字会陷入无限循环，永远无法达到1。

与往常一样，几个例子可以帮助我们快速理解快乐数字的定义：
```text
Given number: 19

 19 -> 1^2 + 9^2 = 82
 82 -> 8^2 + 2^2 = 68
 68 -> 6^2 + 8^2 = 100
100 -> 1^2 + 0^2 + 0^2 = 1
  1
```

如上例所示，对于输入数字19，我们最终得到了1，因此，19是一个快乐数字。同样，更多快乐数字示例是7、10、13、23等。

然而，如果输入是15，我们永远不会达到1：
```text
Given number: 15                  
                                  
       15 -> 1^2 + 5^2 = 26       
       26 -> 2^2 + 6^2 = 40       
       40 -> 4^2 + 0^2 = 16       
+--->  16 -> 1^2 + 6^2 = 37       
|      37 -> 3^2 + 7^2 = 58       
|      58 -> 5^2 + 8^2 = 89       
|      89 -> 8^2 + 9^2 = 145      
|     145 -> 1^2 + 4^2 + 5^2 = 42 
|      42 -> 4^2 + 2^2 = 20       
|      20 -> 2^2 + 0^2 = 4        
|       4 -> 4^2 = 16             
+------16                            
```

我们可以看到，这个过程在16和4之间无限循环，永远不会达到1，所以，15是一个悲伤数字。

按照这个规则，我们还可以找到更多的悲伤数字，例如4、6、11、20等等。

## 3. 实现Check方法

现在我们了解了如何定义快乐数字，让我们实现Java方法来检查给定的数字是否是快乐数字。

如果每个数字的平方和构成的数列包含循环，则该数就是悲伤数。换句话说，**给定一个数列，如果其中一步的计算结果已经存在于该数列中，则该数就是悲伤数**。

我们可以利用[HashSet](https://www.baeldung.com/java-hashset)数据结构在Java中实现此逻辑，这使我们能够存储每个步骤的结果并有效地检查集合中是否已存在新计算的结果：
```java
class HappyNumberDecider {
    public static boolean isHappyNumber(int n) {
        Set<Integer> checkedNumbers = new HashSet<>();
        while (true) {
            n = sumDigitsSquare(n);
            if (n == 1) {
                return true;
            }
            if (checkedNumbers.contains(n)) {
                return false;
            }
            checkedNumbers.add(n);
        }
    }

    // ...
}
```

如上面的代码所示，sumDigitsSquare()方法执行实际计算并返回结果：
```java
private static int sumDigitsSquare(int n) {
    int squareSum = 0;
    while (n != 0) {
        squareSum += (n % 10) * (n % 10);
        n /= 10;
    }
    return squareSum;
}
```

接下来，让我们创建一个测试来验证我们的isHappyNumber()方法是否针对不同的输入报告预期的结果：
```java
assertTrue(HappyNumberDecider.isHappyNumber(7));
assertTrue(HappyNumberDecider.isHappyNumber(10));
assertTrue(HappyNumberDecider.isHappyNumber(13));
assertTrue(HappyNumberDecider.isHappyNumber(19));
assertTrue(HappyNumberDecider.isHappyNumber(23));
 
assertFalse(HappyNumberDecider.isHappyNumber(4));
assertFalse(HappyNumberDecider.isHappyNumber(6));
assertFalse(HappyNumberDecider.isHappyNumber(11));
assertFalse(HappyNumberDecider.isHappyNumber(15));
assertFalse(HappyNumberDecider.isHappyNumber(20));
```

## 4. 性能分析

首先，我们来分析一下该解决方案的时间复杂度。

假设我们有一个k位数字的数n，因此，我们需要k次迭代来计算各位数字的平方和。此外，由于k = log n(以10为底的对数)，**因此计算第一个结果的时间复杂度为O(log n)，我们将其称为result1；由于最高位数字为9，因此result1的最大值是9^2 * log n**。

如果result1不为1，则必须重复调用sumDigitsSquare()，此时，**时间复杂度为O(log n) + O(log(9^2 * log n)) = O(log n) + O(log 81 + log log n)**，删除常数部分后，**我们得到O(log n) + O(log log n)**。

因此，我们的总时间复杂度变为O(log n) + O(log log n) + O(log log log n) + O(log log log log n) + ...直到结果达到1或检测到循环，**由于log n是此表达式中的主导部分，因此解决方案的时间复杂度为O(log n)**。

在进行空间复杂度分析之前，我们先列出最大的k位数字n和相应的结果1：
```text
k     Largest n        result1
1     9                81
2     99               162
3     999              243
4     9999             324
5     99999            405
6     999999           486
7     9999999          567
8     99999999         648
9     999999999        729
10    9999999999       810
11    99999999999      891
12    999999999999     972
13    9999999999999    1053
       ...
1m    9..a million..9  81000000
```

我们可以看到，给定一个3位数以上的数字n，**重复应用sumDigitsSquare()运算可以在几个步骤内将n迅速减少为3位数**。

例如，当k = 1m时，n远大于Java的Long.MAX_VALUE，只需两步即可得出小于3位数的数字：9..(1m)..9 -> 81000000(9^2 * 1m = 81000000) -> 65(8^2 + 1^2 = 65)

因此，在Java的int或long范围内，可以合理地假设n需要常熟C步才能达到3位或更少的数字。因此，我们的HashSet最多可容纳C + 243个结果，**空间复杂度为O(1)**。

虽然该方法的空间复杂度为O(1)，但它仍然需要空间来存储检测循环的结果。

现在我们的目标是改进解决方案，以便在不使用额外空间的情况下识别快乐数字。

## 5. 改进isHappyNumber()方法

[Floyd的循环检测算法](https://en.wikipedia.org/wiki/Cycle_detection#:~:text=Floyd'scycle-findingalgorithmis,isnamedafterRobertW.Floyd)可以检测序列中的循环，例如，我们可以[应用此算法](https://www.baeldung.com/cs/check-if-linked-list-is-circular#two-pointer)来检查[LinkedList](https://www.baeldung.com/cs/check-if-linked-list-is-circular)是否包含循环。

其思路是维护两个指针，一个慢指针，一个快指针；**慢指针每次更新一步，慢指针每次更新两步**，如果它们在1处相遇，则给定的数字是快乐数字。否则，**如果它们相遇但值不是1，则检测到一个循环**，因此，给定的数字是悲伤数字。

接下来我们用Java来实现慢快逻辑：
```java
public static boolean isHappyNumberFloyd(int n) {
    int slow = n;
    int fast = n;
    do {
        slow = sumDigitsSquare(slow);
        fast = sumDigitsSquare(sumDigitsSquare(fast));
    } while (slow != fast);

    return slow == 1;
}
```

让我们用相同的输入来测试isHappyNumberFloyd()方法：
```java
assertTrue(HappyNumberDecider.isHappyNumberFloyd(7));
assertTrue(HappyNumberDecider.isHappyNumberFloyd(10));
assertTrue(HappyNumberDecider.isHappyNumberFloyd(13));
assertTrue(HappyNumberDecider.isHappyNumberFloyd(19));
assertTrue(HappyNumberDecider.isHappyNumberFloyd(23));
                                                   
assertFalse(HappyNumberDecider.isHappyNumberFloyd(4));
assertFalse(HappyNumberDecider.isHappyNumberFloyd(6));
assertFalse(HappyNumberDecider.isHappyNumberFloyd(11));
assertFalse(HappyNumberDecider.isHappyNumberFloyd(15));
assertFalse(HappyNumberDecider.isHappyNumberFloyd(20));
```

最后，该解决方案的时间复杂度仍为O(log n)，但由于不需要额外的空间，其空间复杂度为O(1)。

## 6. 总结

在本文中，我们学习了快乐数字的概念，并实现了Java方法来确定给定数字是否快乐。