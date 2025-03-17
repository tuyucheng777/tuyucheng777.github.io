---
layout: post
title:  在Java中使用UUID生成唯一正长整型值
category: java
copyright: java
excerpt: Java UUID
---

## 1. 概述

通用唯一标识符([UUID](https://www.baeldung.com/java-uuid))表示一个128位数字，旨在实现全球唯一性。实际上，UUID适用于需要唯一标识的情况，例如为数据库表创建主键。

Java提供了[long](https://www.baeldung.com/java-primitives#4-long)原始类型，这是一种易于人类阅读和理解的数据类型。在许多情况下，使用64位long可以提供足够的唯一性，并且冲突概率较低。此外，MySQL、PostgreSQL等数据库以及许多其他数据库都经过了优化，可以高效地处理数字数据类型。

在本文中，**我们将讨论使用UUID生成唯一的正long值，重点关注版本4 UUID**。

## 2. 生成唯一正long值

这种场景提出了一个有趣的挑战，因为UUID的范围是128位，而long值只有64位，这意味着唯一性的可能性会降低。我们将讨论如何使用随机生成的UUID获得唯一的正long值，以及这种方法的有效性。

### 2.1 使用getLeastSignificantBits()

UUID类的getLeastSignificantBits()方法返回UUID的最低64位，这意味着它仅提供128位UUID值的一半。

因此，我们在randomUUID()方法之后调用它：

```java
long randomPositiveLong = Math.abs(UUID.randomUUID().getLeastSignificantBits());
```

我们将在以下每种方法中继续使用Math.abs()来确保正值。

### 2.2 使用getMostSignificantBits()

类似地，UUID类中的getMostSignificantBits()方法也返回64位long，不同之处在于它取的是128位UUID值中位于最高位置的位。

再次，我们将其链接在randomUUID()方法之后：

```java
long randomPositiveLong = Math.abs(UUID.randomUUID().getMostSignificantBits());
```

我们断言结果值是正数(实际上是非负数，因为有可能生成0值)：

```java
assertThat(randomPositiveLong).isNotNegative();
```

## 3. 效果如何？

让我们讨论一下在这种情况下使用UUID的效果如何，为了找到答案，我们将研究几个因素。

### 3.1 安全与效率

Java中的UUID.randomUUID()使用[SecureRandom](https://www.baeldung.com/java-securerandom-generate-positive-long)类生成安全随机数，以生成版本4的UUID。如果生成的UUID将在安全或加密环境中使用，这一点尤其重要。

为了评估使用UUID生成唯一正长值的效率和相关性，让我们看一下源代码：

```java
public static UUID randomUUID() {
    SecureRandom ng = Holder.numberGenerator;

    byte[] randomBytes = new byte[16];
    ng.nextBytes(randomBytes);
    randomBytes[6]  &= 0x0f;  /* clear version        */
    randomBytes[6]  |= 0x40;  /* set to version 4     */
    randomBytes[8]  &= 0x3f;  /* clear variant        */
    randomBytes[8]  |= (byte) 0x80;  /* set to IETF variant  */
    return new UUID(randomBytes);
}
```

该方法使用SecureRandom生成16个随机字节形成UUID，然后调整这些字节中的几个位来指定UUID版本(版本4)和UUID变体(IETF)。

虽然UUID提供了强大的功能，但在这种特定情况下，更简单的方法可以实现预期结果。因此，替代方法可能会提高效率并实现更好的匹配。

此外，这种方法可能会降低生成的位的随机性。

### 3.2 唯一性和碰撞概率

虽然UUID v4的范围是128位，但其中用4位来表示版本4，用2位来表示变体。我们知道，在UUID显示格式中，每个字符代表一个四位十六进制数字：

```text
xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx
```

数字4表示四位UUID版本(在本例中为版本4)。同时，字母“y”将包含两位IETF变体。剩余的[x部分包含122个随机位](https://datatracker.ietf.org/doc/html/rfc4122#section-4.4)。因此，我们可以得到1/2^122的碰撞概率。

[RFC](https://datatracker.ietf.org/doc/html/rfc4122#section-4.4)总体解释了UUID v4是如何创建的，为了更清楚地了解位位置是如何分布的，我们可以再次查看UUID.randomUUID()的实现：

```java
randomBytes[6]  &= 0x0f;  /* clear version        */
randomBytes[6]  |= 0x40;  /* set to version 4     */
```

我们可以看到，在randomBytes[6\]中，最高有效位(MSB)位置有4位设置为版本标记。因此，此MSB中只有60个真正随机的位。因此，我们可以计算出MSB上发生碰撞的概率为1/2^59。

添加Math.abs()后，由于正数和负数重叠，碰撞的几率加倍。因此，**MSB中正长值发生碰撞的概率为2^58分之一**。

然后，在randomBytes[8\]中，在最低有效位(LSB)位置有两个位设置为IETF变体：

```java
randomBytes[8] &= 0x3f; /* clear variant */
randomBytes[8] |= (byte) 0x80; /* set to IETF variant */
```

因此，LSB中只有62个真正随机的位。因此，我们可以计算出LSB上发生碰撞的概率为1/2^61。

添加Math.abs()后，由于正数和负数重叠，碰撞的几率加倍。因此，**LSB中正long值发生碰撞的概率为2^60分之一**。

因此，**我们可以看出发生碰撞的概率很小。但是，如果我们问UUID的内容是否完全随机，那么答案是否定的**。

### 3.3 适用性

UUID的设计目标是实现全球唯一性、识别性、安全性和可移植性。**为了生成唯一的正long值，存在更有效的方法，因此UUID在此情况下是不必要的**。

虽然UUID.randomUUID()拥有128位长度，但我们已经看到只有122位是真正随机的，而Java的long数据类型只能处理64位。将128位UUID转换为64位long时，会丢失一些唯一性潜力。如果唯一性至关重要，请考虑这种权衡。

## 4. SecureRandom作为替代方案

**如果我们需要唯一的long值，使用具有适当范围的随机数生成器更有意义(例如，使用[SecureRandom](https://www.baeldung.com/java-securerandom-generate-positive-long)生成唯一的随机long值)**，这将确保我们在适当的范围内具有唯一的long值，而不会像尝试使用UUID时那样丢失大多数唯一位。

```java
SecureRandom secureRandom = new SecureRandom();
long randomPositiveLong = Math.abs(secureRandom.nextLong());
```

由于它生成完全随机的64位long值，因此发生碰撞的概率也较低。

为了确保正值，我们只需添加Math.abs()。因此，碰撞概率计算为1/2^62。以十进制形式表示，此概率约为0.000000000000000000216840434497100900。 **对于大多数实际应用，我们可以认为这个低概率微不足道**。

## 5. 总结

总之，尽管UUID提供了全局唯一标识符，但它们可能不是生成唯一正long值的最有效选择，因为会发生大量位丢失。

利用getMostSignificantBits()和getLeastSignificantBits()等方法仍然可以提供较低的碰撞概率，但使用像SecureRandom这样的随机数生成器可能更高效，并且适合直接生成唯一的正long值。