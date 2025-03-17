---
layout: post
title:  在Java中生成随机十六进制值
category: java
copyright: java
excerpt: Java Hex
---

## 1. 简介

应用程序中的随机十六进制值可用作各种用途(如数据库条目、会话令牌或游戏机制)的唯一标识符，它们还可以用于加密安全和测试过程。

在本快速教程中，我们将学习在Java中生成随机十六进制值的不同方法。

## 2. 使用java.util.Random

java.util中的[Random](https://www.baeldung.com/java-generate-random-long-float-integer-double)类提供了一种生成随机Integer和Long值的简单方法，我们可以将它们转换为十六进制值。

### 2.1 生成无界十六进制值

让我们首先生成一个无界整数，然后使用toHexString()方法将其转换为十六进制字符串：

```java
String generateUnboundedRandomHexUsingRandomNextInt() {
    Random random = new Random();
    int randomInt = random.nextInt();
    return Integer.toHexString(randomInt);
}
```

如果我们的应用程序需要更大的十六进制值，我们可以使用Random类中的nextLong()方法生成随机Long值。然后可以使用其toHexString()方法将此值转换为十六进制字符串：

```java
String generateUnboundedRandomHexUsingRandomNextLong() {
    Random random = new Random();
    long randomLong = random.nextLong();
    return Long.toHexString(randomLong);
}
```

我们还可以使用[String.format()](https://www.baeldung.com/string/format)方法将其转换为十六进制字符串，此方法允许我们使用占位符和格式说明符创建格式化的字符串：

```java
String generateRandomHexWithStringFormatter() {
    Random random = new Random();
    int randomInt = random.nextInt();
    return String.format("%02x", randomInt);
}
```

### 2.2 生成有界十六进制值

我们可以使用Random类的nextInt()方法和一个参数来生成有界的随机整数，然后将其转换为十六进制字符串：

```java
String generateRandomHexUsingRandomNextIntWithInRange(int lower, int upper) {
    Random random = new Random();
    int randomInt = random.nextInt(upper - lower) + lower;
    return Integer.toHexString(randomInt);
}
```

## 3. 使用java.security.SecureRandom

**对于需要加密安全随机数的应用程序，我们应该考虑使用[SecureRandom](https://www.baeldung.com/java-secure-random)类**。SecureRandom类继承自java.util.Random类，因此我们可以使用nextInt()方法来生成有界和无界整数。

### 3.1 生成无界的安全十六进制值

让我们使用nextInt()方法生成一个随机整数并将其转换为十六进制值：

```java
String generateRandomHexUsingSecureRandomNextInt() {
    SecureRandom secureRandom = new SecureRandom();
    int randomInt = secureRandom.nextInt();
    return Integer.toHexString(randomInt);
}
```

我们还可以使用nextLong()方法生成一个随机Long值并将其转换为十六进制值：

```java
String generateRandomHexUsingSecureRandomNextLong() {
    SecureRandom secureRandom = new SecureRandom();
    long randomLong = secureRandom.nextLong();
    return Long.toHexString(randomLong);
}
```

### 3.2 生成有界安全十六进制值

让我们生成一个范围内的随机整数并将其转换为十六进制值：

```java
String generateRandomHexUsingSecureRandomNextIntWithInRange(int lower, int upper) {
    SecureRandom secureRandom = new SecureRandom();
    int randomInt = secureRandom.nextInt(upper - lower) + lower;
    return Integer.toHexString(randomInt);
}
```

## 4. 使用Apache Commons Math3

**Apache Commons Math3提供了一个实用程序类RandomDataGenerator，它提供了更多选项来生成随机值**，此类提供了几种实用方法来生成随机数据。

要使用它，我们首先添加依赖项：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

你可以在[此处](https://mvnrepository.com/artifact/org.apache.commons/commons-math3)查看依赖的最新版本。

### 4.1 生成有界随机十六进制值

我们可以使用RandomDataGenerator类中的nextInt()方法生成随机整数，此方法类似于Random和SecureRandom类，但提供了额外的灵活性和功能：

```java
String generateRandomHexWithCommonsMathRandomDataGeneratorNextIntWithRange(int lower, int upper) {
    RandomDataGenerator randomDataGenerator = new RandomDataGenerator();
    int randomInt = randomDataGenerator.nextInt(lower, upper);
    return Integer.toHexString(randomInt);
}
```

### 4.2 生成安全有界随机十六进制值

我们还可以为加密安全应用程序生成一个安全整数，并将其转换为十六进制字符串以获取安全的随机十六进制值：

```java
String generateRandomHexWithCommonsMathRandomDataGeneratorSecureNextIntWithRange(int lower, int upper) {
    RandomDataGenerator randomDataGenerator = new RandomDataGenerator();
    int randomInt = randomDataGenerator.nextSecureInt(lower, upper);
    return Integer.toHexString(randomInt);
}
```

### 4.3 生成给定长度的随机十六进制字符串

我们可以使用RandomDataGenerator类来生成给定长度的随机十六进制字符串：

```java
String generateRandomHexWithCommonsMathRandomDataGenerator(int len) {
    RandomDataGenerator randomDataGenerator = new RandomDataGenerator();
    return randomDataGenerator.nextHexString(len);
}
```

此方法通过直接生成所需长度的十六进制字符串简化了该过程，**使用nextHex()比生成整数并转换它们更直接，提供了一种直接获取随机十六进制字符串的方法**。

### 4.4 生成给定长度的安全随机十六进制字符串

对于安全敏感的应用程序，我们还可以使用nextSecureHexString()方法生成安全的十六进制字符串：

```java
String generateSecureRandomHexWithCommonsMathRandomDataGenerator(int len) {
    RandomDataGenerator randomDataGenerator = new RandomDataGenerator();
    return randomDataGenerator.nextSecureHexString(len);
}
```

此方法使用安全随机数生成器生成十六进制字符串，非常适合安全性至关重要的应用程序。

## 5. 总结

在本文中，我们学习了几种生成随机十六进制值的方法。通过利用这些方法，我们可以确保我们的应用程序生成适合我们特定需求的强大且可靠的随机十六进制值。