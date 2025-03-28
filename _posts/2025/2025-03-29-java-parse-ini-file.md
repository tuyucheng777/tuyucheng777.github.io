---
layout: post
title:  如何在Java中解析INI文件
category: libraries
copyright: libraries
excerpt: INI4j
---

## 1. 概述

INI文件是Windows或MS-DOS的初始化或配置文件，它们具有纯文本内容，其中包含部分中的键值对。虽然我们可能更喜欢使用Java的原生.properties文件或其他格式来配置我们的应用程序，但有时我们可能需要使用现有INI文件中的数据。

在本教程中，我们将介绍几个可以帮助我们的库。我们还将介绍一种使用INI文件中的数据填充[POJO](https://www.baeldung.com/java-pojo-class)的方法。

## 2. 创建示例INI文件

让我们从示例INI文件sample.ini开始：

```ini
; for 16-bit app support
[fonts]
letter=bold
text-size=28

[background]
color=white

[RequestResult]
RequestCode=1

[ResponseResult]
ResultCode=0
```

该文件有4个部分，使用小写、短横线命名法和大写驼峰命名法混合命名，其值可以是字符串或数字。

## 3. 使用ini4j解析INI文件

[ini4j](http://ini4j.sourceforge.net/index.html)是一个轻量级库，用于从INI文件中读取配置。自2015年以来，它一直没有更新。

### 3.1 安装ini4j

为了能够使用ini4j库，首先我们应该在pom.xml中添加它的[依赖](https://mvnrepository.com/artifact/org.ini4j/ini4j)：

```xml
<dependency>
    <groupId>org.ini4j</groupId>
    <artifactId>ini4j</artifactId>
    <version>0.5.4</version>
</dependency>
```

### 3.2 在ini4j中打开INI文件

**我们可以通过构造一个Ini对象在ini4j中打开一个INI文件**：

```java
File fileToParse = new File("sample.ini");
IniINI= new Ini(fileToParse);
```

该对象现在包含部分和键。

### 3.3 读取章节关键字

我们可以使用Ini类上的get()函数从INI文件的一个部分中读取一个键：

```java
assertThat(ini.get("fonts", "letter"))
    .isEqualTo("bold");
```

### 3.4 转换为Map

让我们看看将整个INI文件转换为Map<String, Map<String, String\>>有多么容易，这是一个表示INI文件层次结构的Java原生数据结构：

```java
public static Map<String, Map<String, String>> parseIniFile(File fileToParse) throws IOException {
    INIini = new Ini(fileToParse);
    return ini.entrySet().stream()
        .collect(toMap(Map.Entry::getKey, Map.Entry::getValue));
}
```

这里，Ini对象的entrySet本质上是String和Map<String, String\>的键值对。Ini的内部表示几乎是一个Map，因此可以使用stream()和toMap()收集器轻松将其转换为普通Map。

我们现在可以用get()从这个Map中读取各个部分：

```java
assertThat(result.get("fonts").get("letter"))
    .isEqualTo("bold");
```

Ini类开箱即用，非常容易，并且可能不需要转换为Map，但我们稍后会找到它的用途。

但是，ini4j是一个老库，而且看起来维护得不是很好，让我们考虑另一种选择。

## 4. 使用Apache Commons解析INI文件

Apache Commons有一个更复杂的工具来处理INI文件，它能够对整个文件进行建模以供读写，但我们只关注它的解析功能。

### 4.1 安装通用配置

让我们首先在pom.xml中添加所需的[依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-configuration2)：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-configuration2</artifactId>
    <version>2.8.0</version>
</dependency>
```

2.8.0版本于2022年更新，比ini4j更新。

### 4.2 打开INI文件

我们可以通过声明一个INIConfiguration对象并传递一个Reader来打开一个INI文件：

```java
INIConfiguration iniConfiguration = new INIConfiguration();
try (FileReader fileReader = new FileReader(fileToParse)) {
    iniConfiguration.read(fileReader);
}
```

这里我们使用了[try-with-resources](https://www.baeldung.com/java-try-with-resources)模式来打开一个FileReader，然后要求INIConfiguration对象使用read函数来读取它。

### 4.3 读取章节键

INIConfiguration类有一个getSection()函数来读取一个部分，还有一个getProperty()函数在返回的对象上读取一个键：

```java
String value = iniConfiguration.getSection("fonts")
    .getProperty("letter")
    .toString();
assertThat(value)
    .isEqualTo("bold");
```

我们应该注意getProperty()返回的是Object而不是String，因此需要转换为String。

### 4.4 转换为Map

我们也可以像前面一样将INIConfiguration转换为Map，这比ini4j稍微复杂一些：

```java
Map<String, Map<String, String>> iniFileContents = new HashMap<>();

for (String section : iniConfiguration.getSections()) {
    Map<String, String> subSectionMap = new HashMap<>();
    SubnodeConfiguration confSection = iniConfiguration.getSection(section);
    Iterator<String> keyIterator = confSection.getKeys();
    while (keyIterator.hasNext()) {
        String key = keyIterator.next();
        String value = confSection.getProperty(key).toString();
        subSectionMap.put(key, value);
    }
    iniFileContents.put(section, subSectionMap);
}
```

要获取所有部分，我们需要使用getSections()查找它们的名称。然后getSection()可以为我们提供每个部分。

然后，我们可以使用为该部分提供所有键的迭代器，并使用每个getProperty()来获取键值对。

虽然这里很难生成Map，但更简单的数据结构的优点是我们可以向系统的其他部分隐藏INI文件解析。或者，我们可以将配置转换为POJO。

## 5. 将INI文件转换为POJO

我们可以使用[Jackson](https://www.baeldung.com/category/json/jackson/)将Map结构转换为POJO。我们可以使用反序列化注解来修饰POJO，以帮助Jackson理解原始INI文件中的各种命名约定，POJO比我们迄今为止见过的任何数据结构都更易于使用。

### 5.1 导入Jackson

我们需要将[Jackson](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.17.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.17.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.2</version>
</dependency>
```

### 5.2 定义一些POJO

我们示例文件的字体部分使用短横线命名作为其属性，让我们定义一个类来表示该部分：

```java
@JsonNaming(PropertyNamingStrategies.KebabCaseStrategy.class)
public static class Fonts {
    private String letter;
    private int textSize;

    // getters and setters
}
```

这里我们使用了JsonNaming注解来描述属性中使用的大小写。

类似地，RequestResult部分具有使用大写驼峰命名的属性：

```java
@JsonNaming(PropertyNamingStrategies.UpperCamelCaseStrategy.class)
public static class RequestResult {
    private int requestCode;

    // getters and setters
}
```

部分名称本身有多种情况，因此我们可以在父对象中声明每个部分，并使用JsonProperty注解来显示与默认小写驼峰命名的偏差：

```java
public class MyConfiguration {
    private Fonts fonts;
    private Background background;

    @JsonProperty("RequestResult")
    private RequestResult requestResult;

    @JsonProperty("ResponseResult")
    private ResponseResult responseResult;

    // getters and setters
}
```

### 5.3 从Map转换为POJO

现在我们可以使用我们的任一库将INI文件读取为Map，并将文件内容建模为POJO，我们可以使用Jackson ObjectMapper执行转换：

```java
ObjectMapper objectMapper = new ObjectMapper();
Map<String, Map<String, String>> iniKeys = parseIniFile(TEST_FILE);
MyConfiguration config = objectMapper.convertValue(iniKeys, MyConfiguration.class);
```

让我们检查整个文件是否已正确加载：

```java
assertThat(config.getFonts().getLetter()).isEqualTo("bold");
assertThat(config.getFonts().getTextSize()).isEqualTo(28);
assertThat(config.getBackground().getColor()).isEqualTo("white");
assertThat(config.getRequestResult().getRequestCode()).isEqualTo(1);
assertThat(config.getResponseResult().getResultCode()).isZero();
```

我们应该注意到，数字属性，例如textSize和requestCode已经作为数字加载到我们的POJO中。

## 6. 库和方法的比较

ini4j库使用起来非常简单，本质上是一个简单的Map类结构。但是，这是一个较旧的库，没有定期更新。

Apache Commons解决方案功能更全面，并定期更新，但使用时需要做更多的工作。

## 7. 总结

在本文中，我们了解了如何使用几个开源库读取INI文件，我们了解了如何读取单个键以及如何遍历整个文件以生成Map。

然后我们看到了如何使用Jackson将Map转换为POJO。