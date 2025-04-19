---
layout: post
title:  用Java解决杆切割问题
category: algorithms
copyright: algorithms
excerpt: 杆切割
---

## 1. 概述

杆切割问题是一个经典的优化问题，涉及寻找将杆切割成段的最佳方法，以最大化总收益。

在本教程中，我们将了解[杆切割问题](https://www.baeldung.com/cs/rod-cutting-problem)并探索使用Java解决该问题的各种方法。

## 2. 理解问题

假设我们有一根长度为n的杆，我们可以灵活地将其切割成各种长度，并出售这些切割段。此外，我们有一个不同长度切割杆的价格表，我们的目标是最大化总收益。

例如，假设一根长度为n = 4的杆，其价格为Pi = [1,5,8,9\]。Pi数组表示长度为i的杆的价格，这意味着：

P1 = 1表示长度为1的杆的价格为1个单位。

P2 = 5表示长度为2的杆的价格为5个单位。

类似地，P3 = 8表示长度为3的杆的价格为8个单位。

P4 = 9表示长度为4的杆的价格为9个单位。

我们可以对杆进行各种切割，每次切割都会根据杆各部分的价格产生一定的收益，以下是长度为4的杆可能的切割方式：

- 切成4段，每段长1米[1,1,1,1\]，收入 = 1 + 1 + 1 + 1 = 4
- 切成2段，每段长2米[2,2\]，收入 = 5 + 5 = 10
- 切成1段，长度为4米，收入 = 9
- 切成3段，长度分别为1米和2米[1,1,2\]，收入 = 1 + 1 + 5 = 7
- 切成3段，长度分别为1米和2米[1,2,1\]，收入 = 1 + 5 + 1 = 7
- 切成3段，长度分别为1米和2米[2,1,1\]，收入 = 5 + 1 + 1 = 7

这里，我们可以看到，将杆切成两段，每段长2米，总价为10。其他所有组合的价格都低于这个数字，因此我们可以看出这是最优答案。

## 3. 递归解决方案

**[递归解决方案](https://www.baeldung.com/java-recursion)涉及将问题分解为子问题并递归求解**，用于计算长度为n的杆的最大收益的递归关系为R<sub>n</sub> = max<sub>1<=i<=n</sub>(P<sub>i</sub> + R<sub>n-i</sub>)。

在上述关系中，R<sub>n</sub>表示长度为n的杆的最大收益，P<sub>i</sub>表示长度为i的杆的价格，我们来执行递归解：

```java
int usingRecursion(int[] prices, int n) {
    if (n <= 0) {
        return 0;
    }
    int maxRevenue = Integer.MIN_VALUE;
    for (int i = 1; i <= n; i++) {
        maxRevenue = Math.max(maxRevenue, prices[i - 1] + usingRecursion(prices, n - i));
    }
    return maxRevenue;
}
```

在我们的示例中，我们检查基准情况，以确定杆的长度是否小于或等于0，从而得出收益为0的总结。我们使用递归系统地探索所有可能的切割方法，以计算每种切割方法的最大收益。最终结果是所有切割方法中实现的最高收益。

现在，让我们测试一下这种方法：

```java
@Test
void givenARod_whenUsingRecursion_thenFindMaxRevenue() {
    int[] prices = {1, 5, 8, 9};
    int rodLength = 4;
    int maxRevenue = RodCuttingProblem.usingRecursion(prices, rodLength);
    assertEquals(10, maxRevenue);
}
```

递归方法有效且相对容易理解，**然而，它的时间复杂度高达O(2<sup>n</sup>)，这使得它不适用于更大的实例，因为递归调用会迅速增加。因此，如果我们有一根4米长的杆，这种方法将需要2<sup>4</sup> = 16次迭代**。

此解决方案不直接将状态存储在单独的数据结构(例如数组或Map)中，相反，它利用语言提供的调用堆栈来跟踪函数类及其各自的参数和局部变量。

这种方法缺乏记忆，导致计算冗余，因为在递归探索过程中，相同的子问题会被多次求解。这种低效率在实际场景中，尤其是杆较长的情况下，会成为一个显著的问题。如果递归深度过大，递归解决方案还可能导致堆栈溢出错误。

## 4. 记忆化递归解决方案

我们可以通过使用[记忆化](https://en.wikipedia.org/wiki/Memoization)来改进我们的递归解决方案，在这种方法中，我们将使用记忆表来存储和重用先前解决的子问题的结果：

```java
int usingMemoizedRecursion(int[] prices, int n) {
    int[] memo = new int[n + 1];
    Arrays.fill(memo, -1);

    return memoizedHelper(prices, n, memo);
}

int memoizedHelper(int[] prices, int n, int[] memo) {
    if (n <= 0) {
        return 0;
    }

    if (memo[n] != -1) {
        return memo[n];
    }
    int maxRevenue = Integer.MIN_VALUE;

    for (int i = 1; i <= n; i++) {
        maxRevenue = Math.max(maxRevenue, prices[i - 1] + memoizedHelper(prices, n - i, memo));
    }

    memo[n] = maxRevenue;
    return maxRevenue;
}
```

其理念是通过在再次求解子问题之前检查其是否已被求解来避免冗余计算，我们的记忆表采用数组的形式，其中每个索引对应杆的长度，关联值保存该特定长度的最大收益。

现在，让我们测试一下这种方法：

```java
@Test 
void givenARod_whenUsingMemoizedRecursion_thenFindMaxRevenue() {
    int[] prices = {1, 5, 8, 9};
    int rodLength = 4;
    int maxRevenue = RodCuttingProblem.usingMemoizedRecursion(prices, rodLength);
    assertEquals(10, maxRevenue);
}
```

通过记忆化避免冗余计算，与简单的递归解决方案相比，时间复杂度显著降低。**现在，时间复杂度由求解的唯一子问题的数量决定，对于杆切割问题，由于两个嵌套循环，时间复杂度通常为O(n²)**。

这意味着，对于一根长度为10米的杆，这里的时间复杂度为100，而之前的是1024，这是因为节点的修剪显著减少了工作量。**然而，这以空间复杂度为代价。该算法必须为每根杆的长度存储最多一个值，因此空间复杂度为O(n)**。

## 5. 动态规划解决方案

解决这个问题的另一种方法是动态规划，动态规划通过将问题分解成更小的子问题来解决。这与分治算法的求解技术非常相似，但是，主要的区别在于动态规划只求解一次子问题。

### 5.1 迭代(自下而上)方法

在这种方法中，避免递归并使用迭代过程消除了与递归解决方案相关的函数调用开销，从而有助于提高性能：

```java
int usingDynamicProgramming(int[] prices, int n) {
    int[] dp = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        int maxRevenue = Integer.MIN_VALUE;

        for (int j = 1; j <= i; j++) {
            maxRevenue = Math.max(maxRevenue, prices[j - 1] + dp[i - j]);
        }
        dp[i] = maxRevenue;
    }
    return dp[n];
}
```

我们也来测试一下这种方法：

```java
@Test 
void givenARod_whenUsingDynamicProgramming_thenFindMaxRevenue() {
    int[] prices = {1, 5, 8, 9};
    int rodLength = 4;
    int maxRevenue = RodCuttingProblem.usingDynamicProgramming(prices, rodLength);
    assertEquals(10, maxRevenue);
}
```

我们的动态规划表(用dp数组表示)存储了每个子问题的最优解，其中索引i对应于杆的长度，dp[i\]保存最大收益，dp数组的最后一项dp[n\]包含原始问题(杆长为n)的最大收益。

这种方法还节省内存，因为它不需要额外的记忆表。

### 5.2 无界背包解决方案

[无界背包](https://www.baeldung.com/java-knapsack)方法是一种动态规划技术，用于解决可以无限次选择同一物品的优化问题。这意味着，如果相同的切割长度有助于最大化收益，则可以多次选择并放入杆中，让我们来看看具体实现：

```java
int usingUnboundedKnapsack(int[] prices, int n) {
    int[] dp = new int[n + 1];

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < prices.length; j++) {
            if (j + 1 <= i) {
                dp[i] = Math.max(dp[i], dp[i - (j + 1)] + prices[j]);
            }
        }
    }

    return dp[n];
}
```

与迭代(自下而上)方法类似，该算法计算每种杆长对应的最大收益，让我们测试一下这个方法：

```java
@Test 
void givenARod_whenUsingGreedy_thenFindMaxRevenue() {
    int[] prices = {1, 5, 8, 9};
    int rodLength = 4;
    int maxRevenue = RodCuttingProblem.usingUnboundedKnapsack(prices, rodLength);
    assertEquals(10, maxRevenue);
}
```

这种方法的缺点是可能会增加空间复杂度。

## 6. 总结

在本教程中，我们了解了杆切割问题并讨论了解决该问题的各种方法。

**递归方法虽然简单，但时间复杂度呈指数级增长。记忆化方法通过重用解并引入轻微的空间复杂度来解决这个问题，迭代方法进一步提高了效率，消除了递归开销**。

这些方法之间的选择取决于具体问题的特征，涉及时间、空间和实现复杂度的权衡。