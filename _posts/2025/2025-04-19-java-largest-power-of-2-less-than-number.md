---
layout: post
title:  使用Java求小于给定数的2的最大幂
category: algorithms
copyright: algorithms
excerpt: 最大幂
---

## 1. 概述

在本文中，我们将了解如何找到小于给定数字的最大2的幂。

为了举例，我们将取样本输入9。2<sup>0</sup>为1，我们可以找到小于给定输入的2的幂的最小有效输入是2。因此，我们只会将大于1的输入视为有效。

## 2. 简单方法

让我们从2<sup>0</sup>(即1)开始，然后我们继续将该数字乘以2，直到找到小于输入的数字：

```java
public long findLargestPowerOf2LessThanTheGivenNumber(long input) {
    Assert.isTrue(input > 1, "Invalid input");
    long firstPowerOf2 = 1;
    long nextPowerOf2 = 2;
    while (nextPowerOf2 < input) {
        firstPowerOf2 = nextPowerOf2;
        nextPowerOf2 = nextPowerOf2 * 2;
    }
    return firstPowerOf2;
}
```

让我们了解示例input = 9的代码。

firstPowerOf2的初始值是1，nextPowerOf2的初始值是2。我们可以看到，2 < 9为ture，并且我们进入while循环。

对于第一次迭代，firstPowerOf2为2，nextPowerOf2为2 * 2 = 4。同样4 < 9，因此继续while循环。

对于第二次迭代，firstPowerOf2为4，nextPowerOf2为4 * 2 = 8。现在8 < 9，继续。

对于第三次迭代，firstPowerOf2为8，nextPowerOf2为8 * 2 = 16。while条件16 < 9为true，因此跳出while循环，8是小于9的2的最大幂。

让我们运行一些测试来验证我们的代码：

```java
assertEquals(8, findPowerOf2LessThanTheGivenNumber(9));
assertEquals(16, findPowerOf2LessThanTheGivenNumber(32));
```

**我们的解决方案的时间复杂度为O(log<sub>2</sub>(N))**，在我们的例子中，我们迭代了log<sub>2</sub>(9) = 3次。

## 3. 使用Math.log

以2为底的对数表示我们可以递归地将一个数除以2的次数，换句话说，一个数的对数2等于2的幂；让我们看一些例子来理解这一点。

log<sub>2</sub>(8) = 3和log<sub>2</sub>(16) = 4，一般来说，我们可以看到y = log<sub>2</sub>x，其中x = 2<sup>y</sup>。

因此，如果我们找到一个可以被2整除的数字，我们就从中减去1，以避免出现该数字是2的完全幂的情况。

[Math.log](https://www.baeldung.com/java-lang-math)是log<sub>10</sub>。要计算log<sub>2</sub>(x)，我们可以使用公式log<sub>2</sub>(x) = log<sub>10</sub>(x) / log<sub>10</sub>(2)

让我们将其写入代码中：

```java
public long findLargestPowerOf2LessThanTheGivenNumberUsingLogBase2(long input) {
    Assert.isTrue(input > 1, "Invalid input");
    long temp = input;
    if (input % 2 == 0) {
        temp = input - 1;
    }
    long power = (long) (Math.log(temp) / Math.log(2));
    long result = (long) Math.pow(2, power);
    return result;
}
```

假设我们的样本输入为9，则temp的初始值也是9。

9 % 2等于1，所以我们的temp变量是9。这里我们使用模除法，它将得出9 / 2的余数。

为了找到log<sub>2</sub>(9)，我们计算log<sub>10</sub>(9) / log<sub>10</sub>(2) = 0.95424 / 0.30103 ~= 3。

现在，结果是2<sup>3</sup>即8。

让我们验证一下我们的代码：

```java
assertEquals(8, findLargestPowerOf2LessThanTheGivenNumberUsingLogBase2(9));
assertEquals(16, findLargestPowerOf2LessThanTheGivenNumberUsingLogBase2(32));
```

实际上，Math.pow将执行与方法1中相同的迭代。因此，我们可以说对于这个解决方案，**时间复杂度也是O(Log<sub>2</sub>(N))**。

## 4. 按位技术

对于这种方法，我们将使用按位移位技术。首先，我们来看看2的幂的二进制表示，假设我们有4位来表示这个数字

| 2<sup>0</sup> |  1| 0001|
|:-------------:| :--: | :--: |
| 2<sup>1</sup> |  2| 0010|
| 2<sup>2</sup> |  4| 0100|
| 2<sup>3</sup> |  8| 1000|

仔细观察，我们可以发现，我们可以**通过左移1的字节来计算2的幂**。即2<sup>2</sup>是将1的字节左移2位，依此类推。

让我们使用位移技术进行编码：

```java
public long findLargestPowerOf2LessThanTheGivenNumberUsingBitShiftApproach(long input) {
    Assert.isTrue(input > 1, "Invalid input");
    long result = 1;
    long powerOf2;
    for (long i = 0; i < Long.BYTES * 8; i++) {
        powerOf2 = 1 << i;
        if (powerOf2 >= input) {
            break;
        }
        result = powerOf2;
    }
    return result;
}
```

在上面的代码中，我们使用long作为数据类型，它占用8个字节，也就是64位。因此，我们将计算2的幂，直到2的64次方。我们使用位移位运算符<<来计算2的幂。对于我们的示例输入9，在第4次迭代之后，powerOf2的值=16，结果=8，此时我们跳出循环，因为16 > 9是输入。

让我们检查一下代码是否按预期工作：

```java
assertEquals(8, findLargestPowerOf2LessThanTheGivenNumberUsingBitShiftApproach(9));
assertEquals(16, findLargestPowerOf2LessThanTheGivenNumberUsingBitShiftApproach(32));
```

**这种方法的最坏时间复杂度仍然是O(log<sub>2</sub>(N))**，与我们在第一种方法中看到的情况类似。然而，这种方法更好，因为**位移位运算比乘法更高效**。

## 5. 按位与

对于我们的下一个方法，我们将使用这个公式2<sup>n</sup> AND 2<sup>n-1</sup> = 0。

让我们看一些例子来了解它是如何工作的。

4的二进制表示形式为0100，3的二进制表示形式为0011。

让我们对这两个数字进行[按位与](https://www.baeldung.com/java-bitwise-operators)运算，0100与0011的结果为0000，对于任何2的幂和小于它的数，我们都可以得出相同的总结。假设16(2<sup>4</sup>)和15，分别表示为1000和0111。同样，我们看到这两个数字的按位与结果为0。我们也可以说，除了这两个数字之外，对任何其他数字进行与运算都不会得出0。

让我们看看使用按位与解决这个问题的代码：

```java
public long findLargestPowerOf2LessThanTheGivenNumberUsingBitwiseAnd(long input) { 
    Assert.isTrue(input > 1, "Invalid input");
    long result = 1;
    for (long i = input - 1; i > 1; i--) {
        if ((i & (i - 1)) == 0) {
            result = i;
            break;
        }
    }
    return result;
}
```

在上面的代码中，我们循环遍历小于输入的数字，每当我们发现一个数字与数字-1的位与结果为0时，我们就跳出循环，因为我们知道该数字是2的幂。在本例中，对于示例输入9，当i = 8且i - 1 = 7时，我们跳出循环。

现在，让我们验证几个场景：

```java
assertEquals(8, findLargestPowerOf2LessThanTheGivenNumberUsingBitwiseAnd(9));
assertEquals(16, findLargestPowerOf2LessThanTheGivenNumberUsingBitwiseAnd(32));
```

**当输入是2的精确幂时，此方法的最坏时间复杂度为O(N/2)**。如我们所见，这不是最有效的解决方案，但了解这项技术很有用，因为它在解决类似问题时可能会派上用场。

## 6. 总结

我们介绍了寻找小于给定数的最大2的幂的不同方法，并注意到在某些情况下，按位运算可以简化计算。