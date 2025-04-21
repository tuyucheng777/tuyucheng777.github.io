---
layout: post
title:  Java中的货币代码到货币符号的映射
category: java
copyright: java
excerpt: 货币
---

## 1. 概述

在金融应用程序中，处理[货币](https://www.baeldung.com/java-money-and-currency)代码及其对应的符号至关重要。像USD或EUR这样的货币代码在交易中很有用，但是，用户通常更喜欢看到$或€这样的符号，以便于阅读。在Java中显示正确的货币符号并不总是那么简单，尤其是在考虑本地化时。

**Java提供了多种将货币代码映射到其相应符号的方法，包括内置的Currency类、硬编码的Map以及Locale支持**。本文将深入探讨所有这些方法，并提供性能比较和JUnit测试进行验证。

## 2. 货币代码到符号的映射方法

根据本地化需求、一致性要求和维护的便捷性，应用程序需要采用不同的方法来检索货币符号，以下部分将详细探讨这些方法。

### 2.1 使用Currency类

Java提供了[Currency](https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/util/Currency.html)类，用于根据ISO 4217货币代码检索货币符号，**该类定义在util包中**：

```java
public class CurrencyUtil {
    public static String getSymbol(String currencyCode) {
        Currency currency = Currency.getInstance(currencyCode);
        return currency.getSymbol();
    }
}
```

在此实现中，我们传递一个货币代码作为参数，并使用getInstance()方法检索相应的Currency类实例。接下来，我们使用getSymbol()方法获取货币符号。

让我们验证一下Currency类是否返回给定货币代码的正确符号，如果提供了无效的代码，则应该抛出异常：

```java
class CurrencyUtilTest {
    @Test
    void givenValidCurrencyCode_whenGetSymbol_thenReturnsCorrectSymbol() {
        assertEquals("$", CurrencyUtil.getSymbol("USD"));
        assertEquals("€", CurrencyUtil.getSymbol("EUR"));
    }

    @Test
    void givenInvalidCurrencyCode_whenGetSymbol_thenThrowsException() {
        assertThrows(IllegalArgumentException.class, () -> CurrencyUtil.getSymbol("INVALID"));
    }
}
```

### 2.2 使用Currency类与Locale

应用程序可以将[Locale](https://www.baeldung.com/java-8-localization#localization)类与Currency类结合使用，以检索本地化的货币符号，这种方法对于那些需要根据用户区域设置动态调整货币符号的国际化应用程序非常有用：

```java
public class CurrencyLocaleUtil {
    public String getSymbolForLocale(Locale locale) {
        Currency currency = Currency.getInstance(locale);
        return currency.getSymbol();
    }
}
```

getSymbolForLocale()方法根据提供的语言环境检索货币符号。

让我们确保系统根据Locale正确检索货币符号：

```java
class CurrencyLocaleUtilTest {
    private final CurrencyLocaleUtil currencyLocale = new CurrencyLocaleUtil();

    @Test
    void givenLocale_whenGetSymbolForLocale_thenReturnsLocalizedSymbol() {
        assertEquals("$", currencyLocale.getSymbolForLocale(Locale.US));
        assertEquals("€", currencyLocale.getSymbolForLocale(Locale.FRANCE));
    }
}
```

### 2.3 使用硬编码Map

预定义[Map](https://www.baeldung.com/category/java/java-collections/java-map)提供了一种简单灵活的方法来显式控制货币符号映射，当应用程序需要在UI中显示一致的符号、处理不常用的货币或强制执行与语言环境设置无关的特定格式规则时，此方法尤其有用，它最适用于支持的货币集有限且不太可能频繁更改的场景：

```java
public class CurrencyMapUtil {
    private static final Map<String, String> currencymap = Map.of(
        "USD", "$",
        "EUR", "€",
        "INR", "₹"
    );

    public static String getSymbol(String currencyCode) {
        return currencymap.getOrDefault(currencyCode, "Unknown");
    }
}
```

下面的测试验证了硬编码的Map对于已知代码返回正确的符号，对于无法识别的代码默认为“Unknown”：

```java
class CurrencyMapUtilTest {
    @Test
    void givenValidCurrencyCode_whenGetSymbol_thenReturnsCorrectSymbol() {
        assertEquals("$", CurrencyMapUtil.getSymbol("USD"));
        assertEquals("€", CurrencyMapUtil.getSymbol("EUR"));
        assertEquals("₹", CurrencyMapUtil.getSymbol("INR"));
    }

    @Test
    void givenInvalidCurrencyCode_whenGetSymbol_thenReturnsUnknown() {
        assertEquals("Unknown", CurrencyMapUtil.getSymbol("XYZ"));
    }
}
```

## 3. 比较

每种方法都有不同的特点，下表比较了每种方法的用例和易维护性：

|    方法     |     维护|                       用例|
|:---------:| :----------: | :----------------------------------------------: |
| Currency类 | 无需手动更新|        最适合具有本地化支持的标准应用程序|
|  硬编码Map   | 需要手动更新| 适用于需要完全控制货币符号或处理自定义符号的情况|
|   Locale    | 无需手动更新|               最适合国际化应用程序|

如果应用程序需要一组固定的货币符号，那么硬编码Map方法是理想的。如果需要本地化支持，我们可以考虑Currency类或基于Locale的解决方案。

## 4. 总结

选择正确的方法取决于用例需求，**Currency类比较可靠，硬编码的Map对于固定符号集来说是最佳选择，而Locale方法对于需要区域适配的应用程序来说是理想的选择**。

我们应该根据应用程序需求(例如本地化需求、符号一致性和易于维护)来选择一种方法。