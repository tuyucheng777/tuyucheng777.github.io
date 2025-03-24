---
layout: post
title:  使用libphonenumber验证电话号码
category: libraries
copyright: libraries
excerpt: Libphonenumber
---

## 1. 概述

在此快速教程中，我们将了解如何**使用Google的开源库[libphonenumber](https://github.com/google/libphonenumber)在Java中验证电话号码**。

## 2. Maven依赖

首先，我们需要在pom.xml中添加此库的依赖：

```xml
<dependency>
    <groupId>com.googlecode.libphonenumber</groupId>
    <artifactId>libphonenumber</artifactId>
    <version>8.12.10</version>
</dependency>
```

最新版本的信息可以在[Maven Central](https://mvnrepository.com/artifact/com.googlecode.libphonenumber/libphonenumber)上找到。

## 3. PhoneNumberUtil 

该库提供了一个实用程序类[PhoneNumberUtil](https://www.javadoc.io/doc/com.googlecode.libphonenumber/libphonenumber/8.12.9/com/google/i18n/phonenumbers/PhoneNumberUtil.html)，它提供了几种处理电话号码的方法。

让我们看一些如何使用其各种API进行验证的例子。

**重要的是，在所有示例中，我们将使用此类的单例对象来进行方法调用**：

```java
PhoneNumberUtil phoneNumberUtil = PhoneNumberUtil.getInstance();
```

### 3.1 isPossibleNumber

使用PhoneNumberUtil#isPossibleNumber，我们可以检查给定的号码是否适合特定的[国家代码](https://countrycode.org/)或地区。

以美国为例，其国家代码为1，我们可以按以下方式检查给定的电话号码是否可能是美国号码：

```java
@Test
public void givenPhoneNumber_whenPossible_thenValid() {
    PhoneNumber number = new PhoneNumber();
    number.setCountryCode(1).setNationalNumber(123000L);
    assertFalse(phoneNumberUtil.isPossibleNumber(number));
    assertFalse(phoneNumberUtil.isPossibleNumber("+1 343 253 00000", "US"));
    assertFalse(phoneNumberUtil.isPossibleNumber("(343) 253-00000", "US"));
    assertFalse(phoneNumberUtil.isPossibleNumber("dial p for pizza", "US"));
    assertFalse(phoneNumberUtil.isPossibleNumber("123-000", "US"));
}
```

在这里，**我们还使用了该函数的另一种变体**，即将我们期望拨打号码的区域作为String传递。

### 3.2 isPossibleNumberForType

该库可识别不同类型的电话号码，例如固定电话、移动电话、免费电话、语音信箱、VoIP、寻呼机[等等](https://www.javadoc.io/doc/com.googlecode.libphonenumber/libphonenumber/8.12.9/com/google/i18n/phonenumbers/PhoneNumberUtil.PhoneNumberType.html)。

其实用方法isPossibleNumberForType检查给定数字是否可能存在于特定区域中的给定类型中。

举个例子，我们以阿根廷为例，因为它允许不同类型的数字有不同长度。

因此，我们可以用它来演示该API的功能：

```java
@Test
public void givenPhoneNumber_whenPossibleForType_thenValid() {
    PhoneNumber number = new PhoneNumber();
    number.setCountryCode(54);

    number.setNationalNumber(123456);
    assertTrue(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.FIXED_LINE));
    assertFalse(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.TOLL_FREE));

    number.setNationalNumber(12345678901L);
    assertFalse(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.FIXED_LINE));
    assertTrue(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.MOBILE));
    assertFalse(phoneNumberUtil.isPossibleNumberForType(number, PhoneNumberType.TOLL_FREE));
}
```

我们可以看到，上述代码验证了阿根廷允许6位数字的固定电话号码和11位数字的手机号码。

### 3.3 isAlphaNumber

此方法用于验证给定的电话号码是否是有效的字母数字，例如325-CARS：

```java
@Test
public void givenPhoneNumber_whenAlphaNumber_thenValid() {
    assertTrue(phoneNumberUtil.isAlphaNumber("325-CARS"));
    assertTrue(phoneNumberUtil.isAlphaNumber("0800 REPAIR"));
    assertTrue(phoneNumberUtil.isAlphaNumber("1-800-MY-APPLE"));
    assertTrue(phoneNumberUtil.isAlphaNumber("1-800-MY-APPLE.."));
    assertFalse(phoneNumberUtil.isAlphaNumber("+876 1234-1234"));
}
```

需要澄清的是，有效的字母数字开头至少包含3位数字，后面跟着3个或更多字母。上面的实用方法首先删除给定输入中的任何格式，然后检查此条件。

### 3.4 isValidNumber

我们讨论过的上一个API仅根据长度来快速检查电话号码。另一方面，**isValidNumber使用前缀和长度信息进行完整验证**：

```java
@Test
public void givenPhoneNumber_whenValid_thenOK() throws Exception {
    PhoneNumber phone = phoneNumberUtil.parse("+911234567890", CountryCodeSource.UNSPECIFIED.name());

    assertTrue(phoneNumberUtil.isValidNumber(phone));
    assertTrue(phoneNumberUtil.isValidNumberForRegion(phone, "IN"));
    assertFalse(phoneNumberUtil.isValidNumberForRegion(phone, "US"));
    assertTrue(phoneNumberUtil.isValidNumber(phoneNumberUtil.getExampleNumber("IN")));
}
```

这里，当我们没有指定区域时，数字都会得到验证，当我们指定区域时，数字也会得到验证。

### 3.5 isNumberGeographical

此方法检查给定的数字是否具有与之相关的地理或区域：

```java
@Test
public void givenPhoneNumber_whenNumberGeographical_thenValid() throws NumberParseException {
    PhoneNumber phone = phoneNumberUtil.parse("+911234567890", "IN");
    assertTrue(phoneNumberUtil.isNumberGeographical(phone));

    phone = new PhoneNumber().setCountryCode(1).setNationalNumber(2530000L);
    assertFalse(phoneNumberUtil.isNumberGeographical(phone));

    phone = new PhoneNumber().setCountryCode(800).setNationalNumber(12345678L);
    assertFalse(phoneNumberUtil.isNumberGeographical(phone));
}
```

这里，在上面的第一个断言中，我们给出了带有区域代码的国际格式的电话号码，该方法返回了true。第二个断言使用了美国的本地号码，第三个断言使用了免费电话号码。因此，API对这两个断言都返回了false。

## 4. 总结

在本教程中，我们了解了libphonenumber提供的一些功能，可以使用代码示例来格式化和验证电话号码。

这是一个丰富的库，提供更多实用功能，并能满足我们大多数应用程序对格式化、解析和验证电话号码的需求。