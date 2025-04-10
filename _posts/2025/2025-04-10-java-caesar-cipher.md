---
layout: post
title:  Java版凯撒密码
category: algorithms
copyright: algorithms
excerpt: 凯撒密码
---

## 1. 概述

在本教程中，我们将探索凯撒密码，这是一种通过移动消息中的字母来生成另一条可读性较差的消息的加密方法。

首先，我们将介绍加密方法并了解如何在Java中实现它。

然后，我们将看到如何解密加密消息，前提是我们知道用于加密它的偏移量。

最后，我们将学习如何破解这样的密码，从而在不知道所用偏移量的情况下从加密消息中检索原始消息。

## 2. 凯撒密码

### 2.1 解释

首先，我们来定义一下什么是密码。[密码是一种加密消息的方法](https://en.wikipedia.org/wiki/Cipher)，目的是降低消息的可读性。**凯撒密码是一种替换密码，它通过将消息中的字母按给定的偏移量进行变换来转换消息**。

假设我们要将字母表移动3位，那么字母A将转换为字母D，字母B转换为E，字母C转换为F，依此类推。

以下是偏移量为3时原始字母与转换后的字母之间的完整匹配：
```text
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
```

我们可以看到，**一旦变换超出了字母Z的范围，我们就会回到字母表的开头，这样X，Y和Z分别变换为A，B和C**。

因此，如果我们选择一个大于或等于26的偏移量，我们至少会在整个字母表上循环一次。假设我们将一条消息移动28位，这实际上意味着我们将其移动了2位。事实上，移动26位之后，所有字母都与自身匹配。

实际上，我们可以通过对任何偏移量**执行模26运算**将其转换为更简单的偏移量：
```java
offset = offset % 26
```

### 2.2 Java中的算法

现在，让我们看看如何用Java实现凯撒密码。

首先，让我们创建一个CaesarCipher类，它将包含一个以消息和偏移量作为参数的cipher()方法：
```java
public class CaesarCipher {
    String cipher(String message, int offset) {}
}
```

该方法将使用凯撒密码加密消息。

这里我们假设偏移量为正数，且消息仅包含小写字母和空格。接下来，我们需要将所有字母字符按给定的偏移量进行移动：
```java
StringBuilder result = new StringBuilder();
for (char character : message.toCharArray()) {
    if (character != ' ') {
        int originalAlphabetPosition = character - 'a';
        int newAlphabetPosition = (originalAlphabetPosition + offset) % 26;
        char newCharacter = (char) ('a' + newAlphabetPosition);
        result.append(newCharacter);
    } else {
        result.append(character);
    }
}
return result;
```

我们可以看到，**我们依靠字母表字母的ASCII码来实现我们的目标**。

首先，我们计算当前字母在字母表中的位置，为此，我们取其ASCII码并从中减去字母a的ASCII码。然后我们将偏移量应用于此位置，并谨慎地使用模数以使其保持在字母表范围内。最后，我们通过将新位置添加到字母a的ASCII码来检索新字符。

现在，让我们尝试对消息“he told me i could never teach a llama to drive”执行此实现，偏移量为3：
```java
CaesarCipher cipher = new CaesarCipher();

String cipheredMessage = cipher.cipher("he told me i could never teach a llama to drive", 3);

assertThat(cipheredMessage)
    .isEqualTo("kh wrog ph l frxog qhyhu whdfk d oodpd wr gulyh");
```

我们可以看到，加密消息符合先前定义的偏移量3的匹配。

现在，这个特定示例的特殊性在于转换期间不会超过字母z，因此不必回到字母表的开头。因此，让我们再次尝试将偏移量设置为10，这样某些字母就会映射到字母表开头的字母，例如t将映射到d：
```java
String cipheredMessage = cipher.cipher("he told me i could never teach a llama to drive", 10);

assertThat(cipheredMessage)
    .isEqualTo("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo");
```

由于使用了模运算，它能够按预期工作。该运算还能处理较大的偏移量，假设我们想使用36作为偏移量，相当于10，模运算可以确保转换结果相同。

## 3. 解密

### 3.1 解释

现在，让我们看看当我们知道用于加密该消息的偏移量时如何解密这样的消息。

事实上，**解密用凯撒密码加密的消息可以看作是用负偏移量对其进行加密，也可以用互补偏移量对其进行加密**。

因此，假设我们有一条用偏移量3加密的消息。然后，我们可以用偏移量-3对其进行加密，也可以用偏移量23对其进行加密。无论哪种方式，我们都会检索原始消息。

不幸的是，我们的算法无法直接处理负偏移量。转换字母时，如果循环回到字母表末尾，就会遇到问题(例如，将字母a转换为偏移量为-1的字母z)。不过，我们可以计算出一个正的互补偏移量，然后使用我们的算法。

那么，如何获得这个互补偏移量？最简单的方法是从26中减去原始偏移量，当然，这适用于0到26之间的偏移量，但在其他情况下会得出负值。

在这里，**我们将再次使用模运算符，直接在原始偏移量上进行减法运算**。这样，我们确保始终返回正偏移量。

### 3.2 Java中的算法

现在让我们用Java来实现它，首先，我们将在类中添加一个decipher()方法：
```java
String decipher(String message, int offset) {}
```

然后，让我们使用计算出的互补偏移量来调用cipher()方法：
```java
return cipher(message, 26 - (offset % 26));
```

就这样，我们的解密算法就设置好了，让我们在偏移量为36的示例上尝试一下：
```java
String decipheredSentence = cipher.decipher("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo", 36);
assertThat(decipheredSentence)
    .isEqualTo("he told me i could never teach a llama to drive");
```

如我们所见，我们检索到了原始消息。

## 4. 破解凯撒密码

### 4.1 解释

现在我们已经介绍了如何使用凯撒密码加密和解密消息，我们可以深入研究如何破解它。也就是说，**在不知道偏移量的情况下解密加密信息**。

为此，我们将利用概率在文本中查找英文字母。我们的想法是使用偏移量0到25来解密消息，并检查哪种偏移量能呈现出与英文文本类似的字母分布。

**为了确定两个分布的相似性，我们将使用[卡方统计量](https://www.investopedia.com/terms/c/chi-square-statistic.asp)**。

卡方统计量会提供一个数字来告诉我们两个分布是否相似，数字越小，它们越相似。

因此，我们将计算每个偏移量的卡方值，然后返回卡方值最小的那个，这样就能得到用于加密消息的偏移量。

然而，我们必须牢记，这种技术并不是万无一失的，如果消息太短或使用的词语不是标准英语文本，它可能会返回错误的偏移量。

### 4.2 定义基本字母分布

现在让我们看看如何用Java实现破解算法。

首先，让我们在CaesarCipher类中创建一个breakCipher()方法，它将返回用于加密消息的偏移量：
```java
int breakCipher(String message) {}
```

然后，让我们定义一个数组，包含在英文文本中找到某个字母的概率：
```java
double[] englishLettersProbabilities = {0.073, 0.009, 0.030, 0.044, 0.130, 0.028, 0.016, 0.035, 0.074,
    0.002, 0.003, 0.035, 0.025, 0.078, 0.074, 0.027, 0.003,
    0.077, 0.063, 0.093, 0.027, 0.013, 0.016, 0.005, 0.019, 0.001};
```

从这个数组中，我们将能够通过将概率乘以消息的长度来计算给定消息中字母的预期频率：
```java
double[] expectedLettersFrequencies = Arrays.stream(englishLettersProbabilities)
    .map(probability -> probability * message.getLength())
    .toArray();
```

例如，在长度为100的消息中，我们预计字母a会出现7.3次，字母e会出现13次。

### 4.3 计算卡方

现在我们来计算解密信息字母分布和标准英文字母分布的卡方。

为了实现这一点，我们需要导入包含用于计算卡方的实用程序类的[Apache Commons Math3库](https://mvnrepository.com/artifact/org.apache.commons/commons-math3)：
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

我们现在需要做的是**创建一个数组，其中包含0到25之间每个偏移量的计算卡方**。

因此，我们将使用每个偏移量来解密加密消息，然后计算该消息中的字母数。

最后，我们将使用ChiSquareTest#chiSquare方法计算预期字母分布和观察到的字母分布之间的卡方：
```java
double[] chiSquares = new double[26];

for (int offset = 0; offset < chiSquares.length; offset++) {
    String decipheredMessage = decipher(message, offset);
    long[] lettersFrequencies = observedLettersFrequencies(decipheredMessage);
    double chiSquare = new ChiSquareTest().chiSquare(expectedLettersFrequencies, lettersFrequencies);
    chiSquares[offset] = chiSquare;
}

return chiSquares;
```

observerLettersFrequencies()方法只是实现了传递的消息中从a到z的字母计数：
```java
long[] observedLettersFrequencies(String message) {
    return IntStream.rangeClosed('a', 'z')
        .mapToLong(letter -> countLetter((char) letter, message))
        .toArray();
}

long countLetter(char letter, String message) {
    return message.chars()
        .filter(character -> character == letter)
        .count();
}
```

### 4.4 寻找最可能的偏移量

一旦计算出所有卡方，我们就可以返回与最小卡方匹配的偏移量：
```java
int probableOffset = 0;
for (int offset = 0; offset < chiSquares.length; offset++) {
    log.debug(String.format("Chi-Square for offset %d: %.2f", offset, chiSquares[offset]));
    if (chiSquares[offset] < chiSquares[probableOffset]) {
        probableOffset = offset;
    }
}

return probableOffset;
```

虽然没有必要以偏移量0进入循环，因为我们认为它在开始循环之前是最小的，但我们这样做是为了打印它的卡方值。

让我们对使用偏移量10加密的消息尝试该算法：
```java
int offset = algorithm.breakCipher("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo");
assertThat(offset).isEqualTo(10);

assertThat(algorithm.decipher("ro dyvn wo s myevn xofob dokmr k vvkwk dy nbsfo", offset))
    .isEqualTo("he told me i could never teach a llama to drive");
```

我们可以看到，该方法检索了正确的偏移量，然后可以使用它来解密消息并检索原始消息。

以下是针对此特定中断计算出的不同卡方：
```text
Chi-Square for offset 0: 210.69
Chi-Square for offset 1: 327.65
Chi-Square for offset 2: 255.22
Chi-Square for offset 3: 187.12
Chi-Square for offset 4: 734.16
Chi-Square for offset 5: 673.68
Chi-Square for offset 6: 223.35
Chi-Square for offset 7: 111.13
Chi-Square for offset 8: 270.11
Chi-Square for offset 9: 153.26
Chi-Square for offset 10: 23.74
Chi-Square for offset 11: 643.14
Chi-Square for offset 12: 328.83
Chi-Square for offset 13: 434.19
Chi-Square for offset 14: 384.80
Chi-Square for offset 15: 1206.47
Chi-Square for offset 16: 138.08
Chi-Square for offset 17: 262.66
Chi-Square for offset 18: 253.28
Chi-Square for offset 19: 280.83
Chi-Square for offset 20: 365.77
Chi-Square for offset 21: 107.08
Chi-Square for offset 22: 548.81
Chi-Square for offset 23: 255.12
Chi-Square for offset 24: 458.72
Chi-Square for offset 25: 325.45
```

我们可以看到，偏移量10的数值明显小于其他数值。

## 5. 总结

在本文中，我们介绍了凯撒密码；我们学习了如何通过将字母移位给定的偏移量来加密和解密消息，我们还学习了如何破解密码。