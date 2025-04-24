---
layout: post
title:  Java表情符号支持改进
category: java-new
copyright: java-new
excerpt: Java 21
---

## 1. 概述

[Java 21](https://www.baeldung.com/java-lts-21-new-features)在java.lang.Character类中引入了一组新方法，**以便为表情符号提供更好的支持，这些方法使我们能够轻松检查某个字符是否是表情符号，并检查表情符号字符的属性和特征**。

在本教程中，我们将探索新添加的方法并介绍与Java 21中表情符号处理相关的关键概念。

## 2. Character API更新

Java 21在java.lang.Character类中引入了6个与表情符号处理相关的新方法，**所有新方法都是静态的，以表示字符的Unicode[代码点](https://www.baeldung.com/java-char-encoding#3-code-point)的int作为参数，并返回boolean值**。

[Unicode代码点](https://www.baeldung.com/cs/unicode-encodings)是Unicode标准中为每个字符分配的唯一数值，它代表不同平台和语言的特定字符。例如，代码点U+0041代表字母“A”，十六进制形式为0x0041。

Unicode联盟是一家非营利性公司，负责维护和开发Unicode标准，并提供表情符号及其对应Unicode代码点的[完整列表](https://unicode.org/emoji/charts/full-emoji-list.html)。

现在，让我们仔细看看这些与表情符号相关的新方法。

### 2.1 isEmoji()

isEmoji(int codePoint)方法是新表情符号方法中最基本的一个，它接收一个表示字符Unicode代码点的int值，并**返回一个布尔值，指示该字符是否为表情符号**。

我们来看看它的用法：

```java
String messageWithEmoji = "Hello Java 21! 😄";
String messageWithoutEmoji = "Hello Java!";

assertTrue(messageWithEmoji.codePoints().anyMatch(Character::isEmoji));
assertFalse(messageWithoutEmoji.codePoints().anyMatch(Character::isEmoji));
```

### 2.2 isEmojiPresentation()

**isEmojiPresentation(int codePoint)方法确定字符是否应呈现为表情符号**，某些字符(如数字(0-9)和货币符号($或€)可以根据上下文呈现为表情符号或文本字符。

以下代码片段演示了此方法的用法：
```java
String emojiPresentationMessage = "Hello Java 21! 🔥😄";
String nonEmojiPresentationMessage = "Hello Java 21!";

assertTrue(emojiPresentationMessage.codePoints().anyMatch(Character::isEmojiPresentation));
assertFalse(nonEmojiPresentationMessage.codePoints().anyMatch(Character::isEmojiPresentation));
```

### 2.3 isEmojiModifier()

**isEmojiModifier(int codePoint)方法检查某个字符是否为表情符号修饰符，表情符号修饰符是可以修改现有表情符号外观的字符，例如应用[肤色变化](https://www.w3schools.com/charsets/ref_emoji_skin_tones.asp)**。

让我们看看如何使用这种方法来检测表情符号修饰符：
```java
assertTrue(Character.isEmojiModifier(0x1F3FB)); // light skin tone
assertTrue(Character.isEmojiModifier(0x1F3FD)); // medium skin tone
assertTrue(Character.isEmojiModifier(0x1F3FF)); // dark skin tone
```

在这个测试中，我们使用Unicode代码点的十六进制形式(例如0x1F3FB)，而不是实际的表情符号字符，因为表情符号修饰符通常不会呈现为独立的表情符号并且缺乏视觉区别。

### 2.4 isEmojiModifierBase()

**isEmojiModifierBase(int codePoint)方法确定表情符号修饰符是否可以修饰给定字符**，此方法有助于识别支持修饰的表情符号，因为并非所有表情符号都具有此功能。

让我们看一些例子来更好地理解这一点：
```java
assertTrue(Character.isEmojiModifierBase(Character.codePointAt("👍", 0)));
assertTrue(Character.isEmojiModifierBase(Character.codePointAt("👶", 0)));
    
assertFalse(Character.isEmojiModifierBase(Character.codePointAt("🍕", 0)));
```

我们看到竖起大拇指的表情符号“👍”和婴儿表情符号“👶”是有效的表情修饰基础，并且可以通过应用肤色变化来改变其外观，从而表达多样性。

另一方面，披萨表情符号“🍕”不符合有效表情符号修饰基础的条件，因为它是一个独立的表情符号，代表一个物体，而不是可以修改外观的字符或符号。

### 2.5 isEmojiComponent()

**isEmojiComponent(int codePoint)方法检查某个字符是否可以用作创建新表情符号的组件**，这些字符通常与其他字符组合形成新的表情符号，而不是作为独立的表情符号出现。

例如，**[零宽度连接符](https://en.wikipedia.org/wiki/Zero-width_joiner)(ZWJ)是一种非打印字符，它向渲染系统指示相邻字符应显示为单个表情符号**。通过使用零宽度连接符(0x200D)将男人表情符号“👨”(0x1F468)和火箭表情符号“🚀”(0x1F680)组合起来，我们可以创建一个新的宇航员表情符号“👨‍🚀”。我们可以使用[Unicode代码转换器](https://r12a.github.io/app-conversion/)网站通过输入进行测试：0x1F4680x200D0x1F680。

**肤色字符也是表情符号的组成部分**，我们可以将深色肤色字符(0x1F3FF)与挥手表情符号“👋”(0x1F44B)结合起来，创建一个深色肤色的挥手“👋🏿”(0x1F44B0x1F3FF)。由于我们修改的是现有表情符号的外观，而不是创建新的表情符号，因此我们不需要使用ZWJ字符来改变肤色。

让我们看看它的用法并在我们的代码中检测表情符号组件：
```java
assertTrue(Character.isEmojiComponent(0x200D)); // Zero width joiner
assertTrue(Character.isEmojiComponent(0x1F3FD)); // medium skin tone
```

### 2.6 isExtendedPictographic()

**isExtendedPictographic(int codePoint)方法检查某个字符是否属于象形符号的更广泛类别**，该类别不仅包括传统的表情符号，还包括通常由文本处理系统以不同方式呈现的其他符号。

**物体、动物和其他图形符号具有扩展的象形文字属性**。虽然它们并不总是被视为典型的表情符号，但它们需要作为表情符号集的一部分进行识别和处理，以确保正确显示。

以下示例演示了此方法的用法：
```java
assertTrue(Character.isExtendedPictographic(Character.codePointAt("☀️", 0)));  // Sun with rays
assertTrue(Character.isExtendedPictographic(Character.codePointAt("✔️", 0)));  // Checkmark
```

如果传递给isEmojiPresentation()方法，上述两个代码点均返回false，因为它们属于更广泛的扩展象形文字类别，但没有表情符号表示属性。

## 3. 正则表达式中的表情符号支持

除了新的表情符号方法之外，Java 21还在[正则表达式](https://www.baeldung.com/regular-expressions-java)中引入了表情符号支持。我们现在可以使用\\p{IsXXXX}构造根据表情符号属性匹配字符。

让我们举一个例子，使用正则表达式在字符串中搜索任何表情符号：
```java
String messageWithEmoji = "Hello Java 21! 😄";
Matcher isEmojiMatcher = Pattern.compile("\\p{IsEmoji}").matcher(messageWithEmoji);
    
assertTrue(isEmojiMatcher.find());

String messageWithoutEmoji = "Hello Java!";
isEmojiMatcher = Pattern.compile("\\p{IsEmoji}").matcher(messageWithoutEmoji);
    
assertFalse(isEmojiMatcher.find());
```

类似地，我们可以在正则表达式中使用其他表情符号构造：

- IsEmoji_Presentation
- IsEmoji_Modifier
- IsEmoji_Modifier_Base
- IsEmoji_Component
- IsExtended_Pictographic

**需要注意的是，正则表达式属性构造使用蛇形命名格式来引用方法，这与Character类中的静态方法使用的驼峰命名格式不同**。

这些正则表达式构造提供了一种干净、简单的方法来搜索和操作字符串中的表情符号。

## 4. 总结

在本文中，我们探讨了Java 21的Character类中引入的新的表情符号相关方法，我们通过查看各种示例了解了这些静态方法的行为。

我们还讨论了正则表达式中新添加的表情符号支持，以便使用Pattern类搜索表情符号字符及其属性。

当我们构建聊天应用程序、社交媒体平台或任何其他使用表情符号的应用程序时，这些新功能可以为我们提供帮助。