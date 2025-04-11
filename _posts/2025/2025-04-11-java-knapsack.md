---
layout: post
title:  背包问题的Java实现
category: algorithms
copyright: algorithms
excerpt: Log4j
---

## 1. 简介

背包问题是一个组合优化问题，有着广泛的应用，在本教程中，我们将用Java来解决这个问题。

## 2. 背包问题

在背包问题中，我们有一组物品，每件物品都有一个重量和一个价值：

![](/assets/images/2025/algorithms/javaknapsack01.png)

我们想把这些物品放进背包，但是背包有重量限制：

![](/assets/images/2025/algorithms/javaknapsack02.png)

**因此我们需要选择总重量不超过重量限制的物品，并且它们的总价值尽可能高**。例如，上述示例的最佳解决方案是选择5公斤和6公斤的物品，这样在重量限制范围内，总价值最高可达40美元。

背包问题有几种变体，在本教程中，我们将重点介绍[0-1背包问题](https://www.baeldung.com/cs/knapsack-problem-np-completeness)。**在0-1背包问题中，每个物品要么被选中，要么被丢弃**。我们不能只取走一件物品的部分数量，此外，我们也不能多次取走一件物品。

## 3. 数学定义

现在让我们用数学符号形式化地描述0-1背包问题，给定一组n个物品和重量限制W，我们可以将优化问题定义为：

![](/assets/images/2025/algorithms/javaknapsack03.png)

该问题是NP-hard问题，目前没有多项式时间算法可以解决该问题，不过，存在一个基于动态规划的[伪多项式时间算法](https://en.wikipedia.org/wiki/Pseudo-polynomial_time)。

## 4. 递归解决方案

我们可以使用递归公式来解决这个问题：

![](/assets/images/2025/algorithms/javaknapsack04.png)

公式中，M(n,w)是重量限制为w的n个物品的最优解，它是以下两个值中的最大值：

- 从(n-1)个重量限制为w的物品中(不包括第n个物品)得出的最优解
- 第n个物品的价值加上(n-1)个物品的最优解以及w减去第n个物品的权重(包括第n个物品)

如果第n个物品的重量超过当前重量限制，则不包括该物品。因此，它属于上述两种情况的第一类。

我们可以用Java实现这个递归公式：
```java
int knapsackRec(int[] w, int[] v, int n, int W) {
    if (n <= 0) {
        return 0;
    } else if (w[n - 1] > W) {
        return knapsackRec(w, v, n - 1, W);
    } else {
        return Math.max(knapsackRec(w, v, n - 1, W), v[n - 1]
                + knapsackRec(w, v, n - 1, W - w[n - 1]));
    }
}
```

在每个递归步骤中，我们需要评估两个次优解，因此，该递归解决方案的运行时间为O(2<sup>n</sup>)。

## 5. 动态规划解决方案

动态规划是一种将原本难度呈指数级增长的规划问题线性化的策略，其核心思想是存储子问题的结果，这样我们就不必在之后重新计算它们。

我们也可以用动态规划来解决0-1背包问题；要使用动态规划，我们首先创建一个二维表，其维度从0到n和0到W。然后，我们使用自下而上的方法使用该表计算最优解：
```java
int knapsackDP(int[] w, int[] v, int n, int W) {
    if (n <= 0 || W <= 0) {
        return 0;
    }

    int[][] m = new int[n + 1][W + 1];
    for (int j = 0; j <= W; j++) {
        m[0][j] = 0;
    }

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= W; j++) {
            if (w[i - 1] > j) {
                m[i][j] = m[i - 1][j];
            } else {
                m[i][j] = Math.max(
                        m[i - 1][j],
                        m[i - 1][j - w[i - 1]] + v[i - 1]);
            }
        }
    }
    return m[n][W];
}
```

在这个解决方案中，我们对物品数量n和重量限制W进行了嵌套循环。因此，它的运行时间为O(nW)。

## 6. 总结

在本教程中，我们展示了0-1背包问题的数学定义，然后我们用Java实现提供了该问题的递归解决方案；最后，我们使用动态规划来解决这个问题。