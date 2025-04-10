---
layout: post
title:  Java中的维吉尼亚密码
category: algorithms
copyright: algorithms
excerpt: 维吉尼亚密码
---

## 1. 简介

在本文中，我们将研究维吉尼亚密码。我们将了解该密码的工作原理，然后学习如何在Java中实现和逆向该密码。

## 2. 什么是维吉尼亚密码？

**维吉尼亚密码是经典凯撒密码的一个变体，只是每个字母的移动量不同**。

在凯撒密码中，我们将明文中的每个字母移动相同的量。例如，如果我们将每个字母移动3位，那么字符串“BAELDUNG”将变成“EDHOGXQJ”：

![](/assets/images/2025/algorithms/javavigenerecipher01.png)

逆转这个密码只不过是将字母向相反方向移动相同的量。

**维吉尼亚密码与此完全相同，只是每个字母的移动量不同**，这意味着我们需要一种方法来表示键-每个字母移动的量；这通常由另一串字母表示，每个字母根据其在字母表中的位置对应相应的移动量。

例如，键“HELLO”表示将第一个字母移动8个字符，将第二个字母移动5个字符，依此类推。因此，使用此键，字符串“BAELDUNG”现在将变为“JFQXSCSS”：

![](/assets/images/2025/algorithms/javavigenerecipher02.png)

## 3. 实现维吉尼亚密码

**现在我们知道了维吉尼亚密码的工作原理，让我们看看如何用Java实现它**。

我们将从密码方法开始：
```java
public String encode(String input, String key) {
    String result = "";

    for (char c : input.toCharArray()) {
        result += c;
    }

    return result;
}
```

如上所述，此方法用处不大，它只是构建一个与输入字符串相同的输出字符串。

接下来我们要做的是从键中获取每个输入字符对应的字符，我们将为接下来要使用的键位置设置一个计数器，然后在每次传递时递增该计数器：
```java
int keyPosition = 0;
for (char c : input.toCharArray()) {
    char k = key.charAt(keyPosition % key.length());
    keyPosition++;
    // .....
}
```

这看起来有点吓人，让我们来分析一下。首先，我们将键的位置与键的总长度[取模](https://www.baeldung.com/modulo-java)，这意味着当它到达字符串末尾时，它会绕回。然后我们取这个位置的字符，作为输入字符串中这个位置的键字符。

最后，我们需要根据键字符调整我们的输入字符：
```java
String characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
.....
int charIndex = characters.indexOf(c);
int keyIndex = characters.indexOf(k);

int newCharIndex = (charIndex + keyIndex + 1) % characters.length();
c = characters.charAt(newCharIndex);
```

这里，我们有一个包含所有支持字符的字符串。然后我们在这个字符串中找到输入字符和键字符的索引。之后，我们只需将它们相加即可得到新的字符位置。

请注意，我们还要在此基础上加1，因为我们希望键字母的索引为1，而不是0。也就是说，键中的“A”应该相当于移动1个字母，而不是移动0个字母。我们还对可能的字符数进行另一次取模，以便所有字符再次环绕。

但是，这并不完全有效。具体来说，只有当输入字符串和键中的每个字符都在我们支持的列表中时，这才会正常工作。为了解决这个问题，我们只需要在输入字符受支持的情况下增加键的位置，并且仅在输入字符和键字符都受支持时才生成新字符：
```java
if (charIndex >= 0) {
    if (keyIndex >= 0) {
        int newCharIndex = (charIndex + keyIndex + 1) % characters.length();
        c = characters.charAt(newCharIndex);
    }

    keyPosition++;
}
```

至此，我们的密码已完成并且可以运行。

### 3.1 实际演示

**现在我们已经有了密码的有效实现，让我们看看它的实际效果**。我们将使用键“BAELDUNG”对字符串“VIGENERE CIPHER IN JAVA”进行编码。

我们从keyPosition 0开始，第一次迭代得到：

- c = “V”
- k = “B”
- charIndex = 21
- keyIndex = 1
- newCharIndex = 23

这给了我们结果字符“X”。

我们的第二次迭代将是：

- keyPosition = 1
- c = “I”
- k = “A”
- charIndex = 8
- keyIndex = 0
- newCharIndex = 9

这给了我们结果字符“J”。

让我们继续往前看，来看第9个字符：

- keyPosition = 9
- c = ” “
- k = “B”.
- charIndex = -1
- keyIndex = 1

这里有两点比较有意思。首先，我们注意到键位置已经超出了键的长度，所以我们又绕回了开头。其次，我们的输入字符不受支持，所以我们将跳过对这个字符的编码。

如果我们坚持到最后，我们将得到“XJLQRZFL EJUTIM WU LBAM”。

## 4. 解密维吉尼亚密码

**既然我们可以使用维吉尼亚密码对事物进行编码，那么我们也需要能够对它们进行解码**。

或许这并不奇怪，解码算法的绝大部分与编码算法相同。毕竟，我们只是根据正确的键位置将字符移动一定量。

**不同的部分是我们需要移动的方向，我们的newCharIndex需要使用减法而不是加法来计算**。但是，我们还需要手动进行模计算，因为在这个方向上它无法正常工作：
```java
int newCharIndex = charIndex - keyIndex - 1;
if (newCharIndex < 0) {
    newCharIndex = characters.length() + newCharIndex;
}
```

使用此算法版本，我们现在可以成功逆转密码。例如，让我们尝试使用键“BAELDUNG”解码之前的字符串“XJLQRZFL EJUTIM WU LBAM”。

和以前一样，我们的keyPosition从0开始，我们的第一个输入字符是“X”-字符索引为23，我们的第一个键字符是“B”-字符索引为1。

我们的新字符索引是23 - 1 - 1，即21，转换回来后就是“V” 。

如果我们继续对整个字符串进行操作，我们将得到“VIGENERE CIPHER IN JAVA”的结果。

## 5. 维吉尼亚密码调整

**现在我们已经了解了如何在Java中实现维吉尼亚密码，让我们看看可以进行哪些调整**，我们不会在这里实际实现它们-这留给读者练习。

**对密码的任何更改都必须在编码和解码端进行同等程度的修改，否则，编码的消息将无法读取**。这也意味着，任何不知道我们所做更改的攻击者都将更难以攻击任何编码消息。

第一个也是最明显的改变是使用的字符集，我们可以添加更多受支持的字符-包括重音符、空格和其他字符。这样一来，输入字符串中就可以使用更广泛的字符集，并且所有字符在输出字符串中的表示方式也会随之调整。例如，如果我们只是添加一个空格作为允许的字符，那么使用键“BAELDUNG”编码“VIGENERE CIPHER IN JAVA”将得到“XJLQRZELBDNALZEGKOEVEPO”。

我们还可以更改映射字符串中字符的顺序，如果它们不是严格按照字母顺序排列的，一切仍然有效，但结果会有所不同。例如，如果我们将字符串更改为“JQFVHPWORZSLNMKYCGBUXIEDTA”，那么使用键“HELLO”对字符串“BAELDUNG”进行编码现在会生成“DERDPTZV”而不是“JFQXSCSS”。

我们可以做很多其他改动，但这些修改往往会使我们离真正的维吉尼亚密码越来越远。例如，我们可以交换编码和解码算法-编码时将字母表向后移动，解码时向前移动；这通常被称为[变体博福特算法](https://en.wikipedia.org/wiki/Vigenère_cipher#Variant_Beaufort)。

## 6. 总结

到这里，我们了解了维吉尼亚密码及其工作原理，我们还学习了如何用Java自己实现该密码来对消息进行编码和解码。最后，我们还学习了一些可以对该算法进行调整以提高其安全性的方法。