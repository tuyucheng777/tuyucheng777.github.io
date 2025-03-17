---
layout: post
title:  ResourceBundle指南
category: java
copyright: java
excerpt: Java Locale
---

## 1. 概述

许多软件开发人员在其职业生涯中都遇到开发多语言系统或应用程序的机会，这些系统或应用程序通常面向来自不同地区或不同语言区域的最终用户，这允许用户以他们喜欢的语言或格式与应用程序交互。

**维护和扩展这些应用程序始终具有挑战性，修改应用程序数据应该尽可能简单，无需编译。这就是我们通常避免硬编码标签或按钮名称的原因**。

ResourceBundle类是Java中管理[本地化](https://www.baeldung.com/java-8-localization#localization)最有效的工具。在本教程中，我们将探讨ResourceBundle 的工作原理及其用法，并提供示例来展示其功能。

## 2. ResourceBundles

**ResourceBundle使我们的应用程序能够从包含特定于语言环境的数据的不同文件中加载数据**。

首先我们要知道，**一个资源包中的所有文件必须位于同一个包/目录中，并具有一个共同的基本名称**。它们可能具有特定于语言环境的后缀，用下划线符号分隔，表示语言、国家或平台。

如果已经有语言代码，则附加国家代码非常重要；如果存在语言和国家代码，则附加平台非常重要。

让我们看一下示例文件名：

- ExampleResource
- ExampleResource_en
- ExampleResource_en_US
- ExampleResource_en_US_UNIX

每个数据包的默认文件始终是没有任何后缀的文件–ExampleResource。由于ResourceBundle有两个子类：**PropertyResourceBundle和ListResourceBundle**，我们可以交替将数据保存在属性文件和Java文件中。

每个文件必须具有特定于语言环境的名称和适当的文件扩展名，例如，ExampleResource_en_US.properties或Example_en.java。

### 2.1 属性文件–PropertyResourceBundle

属性文件由PropertyResourceBundle表示，它们以区分大小写的键值对的形式存储数据。

让我们分析一个示例属性文件：

```text
# Buttons
continueButton continue
cancelButton=cancel

! Labels
helloLabel:hello
```

我们可以看到，定义键值对有三种不同的风格。

它们都是等效的，但第一个可能是Java程序中最流行的。值得一提的是，我们也可以在属性文件中添加注释，注释始终以#或!开头。

### 2.2 Java文件–ListResourceBundle

首先，为了存储特定于语言的数据，我们需要创建一个扩展ListResourceBundle并覆盖getContents()方法的类，类名约定与属性文件相同。

对于每个Locale，我们需要创建一个单独的Java类。

以下是一个示例类：

```java
public class ExampleResource_pl_PL extends ListResourceBundle {

    @Override
    protected Object[][] getContents() {
        return new Object[][] {
                {"currency", "polish zloty"},
                {"toUsdRate", new BigDecimal("3.401")},
                {"cities", new String[] { "Warsaw", "Cracow" }}
        };
    }
}
```

Java文件比属性文件有一个主要优势，那就是它可以保存我们想要的任何对象-而不仅仅是[字符串](https://www.baeldung.com/java-string)。

另一方面，每次修改或引入新的特定于语言环境的Java类都需要重新编译应用程序，而属性文件则可以在不进行任何额外努力的情况下进行扩展。

## 3. 使用资源包

我们已经知道如何定义资源包，因此我们已经准备好使用它们了。

让我们考虑一下简短的代码片段：

```java
Locale locale = new Locale("pl", "PL");
ResourceBundle exampleBundle = ResourceBundle.getBundle("package.ExampleResource", locale);

assertEquals(exampleBundle.getString("currency"), "polish zloty");
assertEquals(exampleBundle.getObject("toUsdRate"), new BigDecimal("3.401")); 
assertArrayEquals(exampleBundle.getStringArray("cities"), new String[]{"Warsaw", "Cracow"});
```

首先，我们可以定义我们的Locale，除非我们不想使用默认的。

之后，让我们调用ResourceBundle的静态工厂方法，我们需要将包名称及其包/目录和语言环境作为参数传递。

如果默认语言环境没问题，还有一个工厂方法只需要一个包名称。当我们有对象时，我们可以通过它们的键检索值。

此外，该示例显示我们可以使用getString(String key)、getObject(String key)和getStringArray(String key)来获取我们想要的值。

## 4. 选择合适的BundleResource

**如果我们想要使用BundleResource，了解Java如何选择资源文件非常重要**。

假设我们使用一个需要波兰语标签的应用程序，但你的默认[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)语言环境是Locale.US。

一开始，应用程序会在类路径中查找适合你请求的语言环境的文件。它从最具体的名称开始，其中包含平台、国家/地区和语言。

然后，它会转到更通用的语言环境。如果没有匹配，它会返回到默认语言环境，这次不会检查平台。

**如果没有匹配，它将尝试读取默认包**。当我们查看所选文件名的顺序时，一切都应该很清楚：

- Label_pl_PL_UNIX
- Label_pl_PL
- Label_pl
- Label_en_US
- Label_en
- Label

我们应该记住，每个名称都代表.java和.properties文件，但前者优先于后者。当没有合适的文件时，会抛出MissingResourceException。

## 5. 继承

资源包概念的另一个优点是属性[继承](https://www.baeldung.com/java-inheritance)，这意味着不太具体的文件中包含的键值对将被继承树中较高级别的文件继承。

假设我们有三个属性文件：

```properties
#resource.properties
cancelButton = cancel

#resource_pl.properties
continueButton = dalej

#resource_pl_PL.properties
backButton = cofnij
```

检索Locale(“pl”, “PL”)的资源包将返回结果中的所有三个键/值。值得一提的是，就属性继承而言，不会回退到默认语言环境包。

而且，**ListResourceBundles和PropertyResourceBundles不在同一层次结构中**。

### 5.1 处理跨属性文件和Java文件的键值继承

当Java应用程序查找资源包(例如属性文件)时，它只会从找到的属性文件中继承键值对。此继承不会扩展到其他类型的资源包，例如ListResourceBundles。

换句话说，**属性继承仅在属性文件之间有效，但继承不会发生在其他类型的资源(如ListResourceBundle)之间**。同样，如果Java文件作为资源包，它们将遵循相同的继承规则。

让我们考虑一下类路径中的messages.properties文件：

```properties
greeting=Hello
farewell=Goodbye
```

以及messages_en.properties：

```properties
greeting=Hi
farewell=See you later
```

现在，我们将创建一个CustomListResourceBundle并尝试从属性文件继承：

```java
public class CustomListResourceBundle extends ListResourceBundle {
    @Override
    protected Object[][] getContents() {
        return new Object[][] {
            { "customMessage", "This is a custom message." }
        };
    }
}
```

让我们看看下面的测试，它显示当我们尝试加载CustomListResourceBundle时，PropertyResourceBundle的属性不会被继承：

```java
@Test
public void givenListResourceBundle_whenUsingInheritance_thenItShouldNotInherit() {
    ResourceBundle listBundle = ResourceBundle.getBundle("cn.tuyucheng.taketoday.resourcebundle.CustomListResourceBundle", Locale.ENGLISH);

    assertEquals("This is a custom message.", listBundle.getString("customMessage"));

    assertThrows(MissingResourceException.class, () -> listBundle.getString("greeting"));
}
```

上述测试验证了ListResourceBundle不支持从其他资源包类型继承，当尝试访问其中不存在的键时，它会抛出MissingResourceException。

## 6. 自定义

上面我们了解的都是ResourceBundle的默认实现。**但是，我们可以通过一种方法来修改其行为**。

我们通过扩展ResourceBundle.Control并重写其方法来实现这一点。

例如，我们可以改变缓存中保存值的时间，或者确定何时应重新加载缓存的条件。

为了更好的理解，我们准备一个简短的方法作为示例：

```java
public class ExampleControl extends ResourceBundle.Control {

    @Override
    public List<Locale> getCandidateLocales(String s, Locale locale) {
        return Arrays.asList(new Locale("pl", "PL"));
    }
}
```

此方法的目的是改变在类路径中选择文件的方式。我们可以看到，ExampleControl将只返回Polish Locale，无论默认或定义的Locale是什么。

## 7. UTF-8

对于使用JDK8或更早版本的应用程序，值得了解的是，在Java 9之前，ListResourceBundles比PropertyResourceBundles还有一个优势。由于Java文件可以存储String对象，因此它们可以保存UTF-16编码支持的任何字符。

相反，PropertyResourceBundle默认使用[ISO 8859-1](https://www.baeldung.com/cs/unicode-encodings)编码加载文件，其字符数比UTF-8少(给我们的波兰语示例带来了问题)。

为了保存超出[UTF-8](https://www.baeldung.com/java-string-encode-utf-8)的字符，我们可以使用Native-To-ASCII转换器-native2ascii，它将所有不符合ISO 8859-1的字符编码为\uxxxx符号。

以下是示例命令：

```shell
native2ascii -encoding UTF-8 utf8.properties nonUtf8.properties
```

让我们看看编码改变前后的属性是什么样的：

```properties
#Before
polishHello=cześć

#After
polishHello=cze\u015b\u0107
```

幸运的是，Java 9中不再存在这种不便，JVM以UTF-8编码读取属性文件，使用非拉丁字符没有问题。

## 8. 总结

在本文中，我们探讨了ResourceBundle以及如何使用它来创建多语言应用程序。BundleResource包含开发多语言应用程序所需的大部分内容。我们介绍的功能简化了对不同语言环境的操作。

我们还避免对值进行硬编码，只需添加新的区域设置文件即可扩展支持的区域设置，从而使我们的应用程序能够顺利地修改和维护。