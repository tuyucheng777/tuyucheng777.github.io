---
layout: post
title:  在Java中寻找最小公倍数
category: algorithms
copyright: algorithms
excerpt: 最小公倍数
---

## 1. 概述

**两个非零整数(a,b)的[最小公倍数](https://en.wikipedia.org/wiki/Least_common_multiple)(LCM)是能够被a和b整除的最小正整数**。

在本教程中，我们将学习求两个或多个数的最小公倍数(LCM)的不同方法。需要注意的是，**负整数和0不属于LCM的候选范围**。

## 2. 使用简单算法计算两个数的最小公倍数

我们可以利用“[乘法](https://en.wikipedia.org/wiki/Multiplication)就是加法的重复”这个简单的事实来找到两个数的最小公倍数。

### 2.1 算法

寻找LCM的简单算法是一种迭代方法，它利用了两个数字的LCM的一些基本属性。

首先，我们知道任何数与零的最小公倍数都是零本身。因此，只要给定的整数中有一个为0，我们就可以提前退出该过程。

其次，我们还可以利用这样的事实：**两个非零整数的最小公倍数的下界是这两个数的绝对值中较大的一个**。

此外，如前所述，最小公倍数永远不可能是负整数。因此，我们只会**使用整数的绝对值来寻找可能的倍数**，直到找到公倍数为止。

让我们看看确定lcm(a,b)需要遵循的具体程序：

1. 如果a = 0或b = 0，则返回lcm(a,b) = 0，否则转到步骤2
2. 计算两个数字的绝对值
3. 将lcm初始化为步骤2中计算出的两个值中的较大者
4. 如果lcm可以被较小的绝对值整除，则返回
5. 将lcm增加两者中较大的绝对值，然后转到步骤4

在开始实现这个简单的方法之前，让我们先进行一次试运行来找到lcm(12,18)。

由于12和18都是正数，让我们跳到步骤3，初始化lcm = max(12,18) = 18，然后继续。

在我们的第一次迭代中，lcm = 18，它不能被12完全整除。因此，我们将其增加18并继续。

在第二次迭代中，我们可以看到lcm = 36，现在可以被12整除。因此，我们可以从算法中返回并得出总结lcm(12,18)是36。

### 2.2 实现

让我们用Java实现该算法，我们的lcm()方法需要接收两个整数参数，并将其最小公倍数作为返回值。

我们可以注意到，上述算法涉及对数字执行一些数学运算，例如查找绝对值、最小值和最大值。为此，我们可以分别使用[Math](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Math.html)类的相应静态方法，例如abs()、min()和max()。

让我们实现lcm()方法：

```java
public static int lcm(int number1, int number2) {
    if (number1 == 0 || number2 == 0) {
        return 0;
    }
    int absNumber1 = Math.abs(number1);
    int absNumber2 = Math.abs(number2);
    int absHigherNumber = Math.max(absNumber1, absNumber2);
    int absLowerNumber = Math.min(absNumber1, absNumber2);
    int lcm = absHigherNumber;
    while (lcm % absLowerNumber != 0) {
        lcm += absHigherNumber;
    }
    return lcm;
}
```

接下来我们也来验证一下这个方法：

```java
@Test
public void testLCM() {
    Assert.assertEquals(36, lcm(12, 18));
}
```

上述测试用例通过断言lcm(12,18)等于36来验证lcm()方法的正确性。

## 3. 使用质因数分解方法

**[算术基本定理](https://en.wikipedia.org/wiki/Fundamental_theorem_of_arithmetic)指出，每个大于1的整数都可以唯一地表示为素数幂的乘积**。

因此，对于任何整数N > 1，我们有N = (2<sup>k1</sup>) * (3<sup>k2</sup>) * (5<sup>k3</sup>) * ...

利用该定理的结果，我们现在将理解通过质因数分解方法来寻找两个数的最小公倍数。

### 3.1 算法

质因数分解法是[通过对两个数进行质因数分解来计算最小公倍数(LCM)](https://proofwiki.org/wiki/LCM_from_Prime_Decomposition)，我们可以使用质因数分解得到的质因数和指数来计算这两个数的最小公倍数(LCM)：

当|a| = (2<sup>p1</sup>) * (3<sup>p2</sup>) * (5<sup>p3</sup>) * ...
且|b| = (2<sup>q1</sup>) * (3<sup>q2</sup>) * (5<sup>q3</sup>) * ...
时，**lcm(a,b) = (2<sup>max(p<sub>1</sub>, q<sub>1</sub>)</sup>) * (3<sup>max(p<sub>2</sub>, q<sub>2</sub>)</sup>) * (5<sup>max(p<sub>3</sub>, q<sub>3</sub>)</sup>)**...

让我们看看如何使用这种方法计算12和18的最小公倍数：

首先，我们需要将两个数的绝对值表示为素因数的乘积：

- 12 = 2 * 2 * 3= 2² * 3¹
- 18 = 2 * 3 * 3= 2¹ * 3²

我们可以注意到，上述表示中的素因数是2和3。

接下来，我们确定最小公倍数(LCM)的每个素因数的指数。我们通过从两个表示式中取其较高次幂来实现。

使用此策略，LCM中2的幂将为max(2,1) = 2，LCM中3的幂将为max(1,2) = 2。

最后，我们可以通过将素因数与上一步获得的相应幂相乘来计算最小公倍数(LCM)。因此，我们得到lcm(12,18) =2² * 3² = 36。

### 3.2 实现

我们的Java实现使用两个数字的质因数分解表示来查找LCM。

为此，我们的getPrimeFactors()方法需要接收一个整数参数，并提供其质因数分解表示。在Java中，**我们可以使用HashMap来表示数字的质因数分解**，其中每个键表示质因数，与键关联的值表示相应因数的指数。

让我们看一下getPrimeFactors()方法的迭代实现：

```java
public static Map<Integer, Integer> getPrimeFactors(int number) {
    int absNumber = Math.abs(number);

    Map<Integer, Integer> primeFactorsMap = new HashMap<Integer, Integer>();

    for (int factor = 2; factor <= absNumber; factor++) {
        while (absNumber % factor == 0) {
            Integer power = primeFactorsMap.get(factor);
            if (power == null) {
                power = 0;
            }
            primeFactorsMap.put(factor, power + 1);
            absNumber /= factor;
        }
    }

    return primeFactorsMap;
}
```

我们知道12和18的质因数分解图分别为{2 → 2, 3 → 1}和{2 → 1, 3 → 2}，让我们用它来测试一下上述方法：

```java
@Test
public void testGetPrimeFactors() {
    Map<Integer, Integer> expectedPrimeFactorsMapForTwelve = new HashMap<>();
    expectedPrimeFactorsMapForTwelve.put(2, 2);
    expectedPrimeFactorsMapForTwelve.put(3, 1);

    Assert.assertEquals(expectedPrimeFactorsMapForTwelve, PrimeFactorizationAlgorithm.getPrimeFactors(12));

    Map<Integer, Integer> expectedPrimeFactorsMapForEighteen = new HashMap<>();
    expectedPrimeFactorsMapForEighteen.put(2, 1);
    expectedPrimeFactorsMapForEighteen.put(3, 2);

    Assert.assertEquals(expectedPrimeFactorsMapForEighteen, PrimeFactorizationAlgorithm.getPrimeFactors(18));
}
```

我们的lcm()方法首先使用getPrimeFactors()方法为每个数字查找质因数分解图，接下来，它使用两个数字的质因数分解图来找到它们的最小公倍数(LCM)，让我们看一下此方法的迭代实现：

```java
public static int lcm(int number1, int number2) {
    if(number1 == 0 || number2 == 0) {
        return 0;
    }

    Map<Integer, Integer> primeFactorsForNum1 = getPrimeFactors(number1);
    Map<Integer, Integer> primeFactorsForNum2 = getPrimeFactors(number2);

    Set<Integer> primeFactorsUnionSet = new HashSet<>(primeFactorsForNum1.keySet());
    primeFactorsUnionSet.addAll(primeFactorsForNum2.keySet());

    int lcm = 1;

    for (Integer primeFactor : primeFactorsUnionSet) {
        lcm *= Math.pow(primeFactor,
                Math.max(primeFactorsForNum1.getOrDefault(primeFactor, 0),
                        primeFactorsForNum2.getOrDefault(primeFactor, 0)));
    }

    return lcm;
}
```

作为一种良好做法，我们现在应该验证lcm()方法的逻辑正确性：

```java
@Test
public void testLCM() {
    Assert.assertEquals(36, PrimeFactorizationAlgorithm.lcm(12, 18));
}
```

## 4. 使用欧几里得算法

两个数字的[LCM](https://en.wikipedia.org/wiki/Least_common_multiple)和[GCD](https://en.wikipedia.org/wiki/Greatest_common_divisor)(最大公约数)之间存在有趣的关系，即[两个数字的乘积的绝对值等于它们的GCD和LCM的乘积](https://proofwiki.org/wiki/Product_of_GCD_and_LCM)。

如上所述，gcd(a,b) * lcm(a,b) = |a * b|。

因此，**lcm(a,b) = |a * b| / gcd(a,b)**。

使用这个公式，我们原来寻找lcm(a,b)的问题现在已经简化为寻找gcd(a,b)。

诚然，**求两个数的最大公约数(GCD)有多种策略**，但是，[欧几里得算法](https://en.wikipedia.org/wiki/Euclidean_algorithm)被认为是最有效的算法之一。

为此，我们来简单了解一下这个算法的关键，可以概括为两个关系：

- **gcd(a,b) = gcd(|a%b|, |a|); 其中|a| >= |b|**
- **gcd(p,0) = gcd(0,p) = |p|**

让我们看看如何使用上述关系找到最小公倍数(12,18)：

我们有gcd(12,18) = gcd(18%12,12) = gcd(6,12) = gcd(12%6,6) = gcd(0,6) = 6

因此，lcm(12,18) = |12 x 18| / gcd(12,18) = (12 x 18) / 6 = 36

我们现在将看到欧几里得算法的递归实现：

```java
public static int gcd(int number1, int number2) {
    if (number1 == 0 || number2 == 0) {
        return number1 + number2;
    } else {
        int absNumber1 = Math.abs(number1);
        int absNumber2 = Math.abs(number2);
        int biggerValue = Math.max(absNumber1, absNumber2);
        int smallerValue = Math.min(absNumber1, absNumber2);
        return gcd(biggerValue % smallerValue, smallerValue);
    }
}
```

上述实现使用了数字的绝对值-因为GCD是可以完美整除两个数字的最大正整数，所以我们对负除数不感兴趣。

我们现在准备验证上述实现是否按预期工作：

```java
@Test
public void testGCD() {
    Assert.assertEquals(6, EuclideanAlgorithm.gcd(12, 18));
}
```

### 4.1 两个数的最小公倍数

使用之前找到最大公约数(GCD)的方法，我们现在可以轻松计算最小公倍数(LCM)。同样，我们的lcm()方法需要接收两个整数作为输入，并返回它们的最小公倍数(LCM)，让我们看看如何在Java中实现此方法：

```java
public static int lcm(int number1, int number2) {
    if (number1 == 0 || number2 == 0)
        return 0;
    else {
        int gcd = gcd(number1, number2);
        return Math.abs(number1 * number2) / gcd;
    }
}
```

现在我们可以验证上述方法的功能：

```java
@Test
public void testLCM() {
    Assert.assertEquals(36, EuclideanAlgorithm.lcm(12, 18));
}
```

### 4.2 使用BigInteger类计算大数的最小公倍数

为了计算大数的最小公倍数，我们可以利用[BigInteger](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigInteger.html) 类。

在内部，**BigInteger类的gcd()方法使用混合算法来优化计算性能**。此外，由于BigInteger对象是不可变的，因此**该实现利用[MutableBigInteger](https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/math/MutableBigInteger.java)类的可变实例来避免频繁的内存重新分配**。

首先，它使用传统的欧几里得算法，用较小的整数重复替换较大的整数的模数。

结果，经过连续的除法运算，这两个对不仅越来越小，而且彼此越来越接近。最终，两个MutableBigInteger对象各自的int[\]值数组中保存其大小所需的int数量差达到1或0。

在此阶段，**策略切换到[二进制GCD算法](https://en.wikipedia.org/wiki/Binary_GCD_algorithm)，以获得更快的计算结果**。

同样，在本例中，我们将通过将两个数字乘积的绝对值除以其最大公约数(GCD)来计算最小公倍数(LCM)。与之前的示例类似，我们的lcm()方法接收两个BigInteger值作为输入，并以BigInteger的形式返回这两个数字的最小公倍数(LCM)，让我们看看它的实际效果：

```java
public static BigInteger lcm(BigInteger number1, BigInteger number2) {
    BigInteger gcd = number1.gcd(number2);
    BigInteger absProduct = number1.multiply(number2).abs();
    return absProduct.divide(gcd);
}
```

最后，我们可以通过测试用例来验证一下：

```java
@Test
public void testLCM() {
    BigInteger number1 = new BigInteger("12");
    BigInteger number2 = new BigInteger("18");
    BigInteger expectedLCM = new BigInteger("36");
    Assert.assertEquals(expectedLCM, BigIntegerLCM.lcm(number1, number2));
}
```

## 5. 总结

在本教程中，我们讨论了在Java中查找两个数字的最小公倍数的各种方法。

此外，我们还学习了数字与其最小公倍数(LCM)和最大公倍数(GCD)的乘积之间的关系，通过提出能够高效计算两个数字最大公倍数(GCD)的算法，我们也将LCM计算问题简化为GCD计算问题。