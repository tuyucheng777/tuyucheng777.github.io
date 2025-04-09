---
layout: post
title:  Java中的梯度下降
category: algorithms
copyright: algorithms
excerpt: 梯度下降
---

## 1. 简介

在本教程中，我们将学习[梯度下降](https://www.baeldung.com/cs/understanding-gradient-descent)算法，我们将用Java实现该算法并逐步说明。

## 2. 什么是梯度下降？

**梯度下降是一种优化算法，用于查找给定函数的局部最小值**；它广泛用于高级机器学习算法中，以最小化损失函数。

Gradient是斜率的另一种说法，而descent的意思是下降。顾名思义，梯度下降就是沿着函数的斜率向下移动，直到到达终点。

## 3. 梯度下降的性质

**梯度下降会找到局部最小值，该最小值可能与全局最小值不同**，起始局部点作为参数传递给算法。

它是一种迭代算法，在每一步中，它都会试图沿着斜率向下移动并更接近局部最小值。

**实际上，该算法是回溯式的**，我们将在本教程中说明并实现回溯式梯度下降。

## 4. 逐步说明

梯度下降需要一个函数和一个起点作为输入，让我们定义并绘制一个函数：

![](/assets/images/2025/algorithms/javagradientdescent01.png)

我们可以从任意一个点开始，假设是x = 1：

![](/assets/images/2025/algorithms/javagradientdescent02.png)

第一步，梯度下降以预定义的步长沿斜率下降：

![](/assets/images/2025/algorithms/javagradientdescent03.png)

接下来，它以相同的步长继续前进。然而，这次它得到的y比上一步更大：

![](/assets/images/2025/algorithms/javagradientdescent04.png)

这表明算法已经过了局部最小值，因此它以减小的步长向后退：

![](/assets/images/2025/algorithms/javagradientdescent05.png)

随后，每当当前y大于前一个y时，步长就会减小并取反。迭代一直持续，直到达到所需的精度。

可以看出，梯度下降在这里找到了局部最小值，但它不是全局最小值。如果我们从x = -1而不是x = 1开始，就会找到全局最小值。

## 5. Java实现

实现梯度下降的方法有很多种，这里我们不计算函数的导数来找到斜率的方向，因此我们的实现也适用于不可微函数。

让我们定义precision和stepCoefficient并赋予它们初始值：
```java
double precision = 0.000001;
double stepCoefficient = 0.1;
```

在第一步中，我们没有先前的y可供比较，我们可以增加或减小x的值，看看y是降低还是升高，stepCoefficient为正表示我们正在增加x的值。

现在让我们执行第一步：
```java
double previousX = initialX;
double previousY = f.apply(previousX);
currentX += stepCoefficient * previousY;
```

在上面的代码中，f是一个Function<Double, Double\>，而initialX是一个double，两者都作为输入提供。

需要考虑的另一个关键点是梯度下降不能保证收敛，为了避免陷入循环，我们对迭代次数进行限制：
```java
int iter = 100;
```

稍后，我们将在每次迭代时将iter减1，因此，我们将在最多100次迭代时退出循环。

现在我们有了previousX，可以设置循环：
```java
while (previousStep > precision && iter > 0) {
    iter--;
    double currentY = f.apply(currentX);
    if (currentY > previousY) {
        stepCoefficient = -stepCoefficient/2;
    }
    previousX = currentX;
    currentX += stepCoefficient * previousY;
    previousY = currentY;
    previousStep = StrictMath.abs(currentX - previousX);
}
```

在每次迭代中，我们计算新的y并将其与前一个y进行比较，如果currentY大于previousY，我们改变方向并减小步长。

循环一直持续，直到步长小于期望的precision。最后，我们可以返回currentX作为局部最小值：
```java
return currentX;
```

## 6. 总结

在本文中，我们通过逐步说明的方式介绍了梯度下降算法。