---
layout: post
title:  合并java.util.Properties对象
category: java
copyright: java
excerpt: Java Properties
---

## 1. 简介

在这个简短的教程中，**我们将重点介绍如何将两个或多个Java Properties对象合并为一个**。

我们将探索三种解决方案，首先从使用迭代的示例开始。接下来，我们将研究如何使用putAll()方法，最后，我们将研究一种使用Java 8 Streams的更现代的方法。

要了解如何开始使用Java属性，请查看我们的[介绍文章](https://www.baeldung.com/java-properties)。

## 2. 快速回顾如何使用属性

首先让我们回顾一下属性的一些关键概念。

**我们通常在应用程序中使用属性来定义配置值**，在Java中，我们使用简单的键/值对来表示这些值。此外，每个对中的键和值都是字符串值。

通常我们使用java.util.Properties类来表示和管理这些值对，**需要注意的是，该类继承自Hashtable**。

要了解有关Hashtable数据结构的更多信息，请阅读 [Java.util.Hashtable简介](https://www.baeldung.com/java-hash-table)。

### 2.1 设置属性

为了简单起见，我们将以编程方式设置示例的属性：

```java
private Properties propertiesA() {
    Properties properties = new Properties();
    properties.setProperty("application.name", "my-app");
    properties.setProperty("application.version", "1.0");
    return properties;
}
```

在上面的例子中，我们创建了一个Properties对象并使用setProperty()方法设置两个属性。在内部，这会调用Hashtable类中的put()方法，但确保对象是字符串值。

**请注意，强烈建议不要直接使用put()方法，因为它允许调用者插入键或值不是String的条目**。

## 3. 使用迭代合并属性

现在让我们看看如何使用迭代合并两个或多个属性对象：

```java
private Properties mergePropertiesByIteratingKeySet(Properties... properties) {
    Properties mergedProperties = new Properties();
    for (Properties property : properties) {
        Set<String> propertyNames = property.stringPropertyNames();
        for (String name : propertyNames) {
            String propertyValue = property.getProperty(name);
            mergedProperties.setProperty(name, propertyValue);
        }
    }
    return mergedProperties;
}
```

我们将这个例子分解成几个步骤：

1. 首先，我们创建一个Properties对象来保存所有合并的属性
2. 接下来，我们循环遍历要合并的Properties对象
3. 然后我们调用stringPropertyNames()方法来获取一组属性名称
4. 然后我们循环遍历所有属性名称并获取每个名称的属性值
5. 最后，我们将属性值设置为在步骤1中创建的变量

## 4. 使用putAll()方法

现在我们来看看使用putAll()方法合并属性的另一种常见解决方案：

```java
private Properties mergePropertiesByUsingPutAll(Properties... properties) {
    Properties mergedProperties = new Properties();
    for (Properties property : properties) {
        mergedProperties.putAll(property);
    }
    return mergedProperties;
}
```

在第二个示例中，我们再次首先创建一个Properties对象来保存所有合并的属性，名为mergedProperties。同样，我们然后遍历要合并的Properties对象，但这次我们使用putAll()方法将每个Properties对象添加到mergedProperties变量中。

putAll()方法是从Hashtable继承的另一种方法，**此方法允许我们将指定Properties的所有映射复制到新的Properties对象中**。

值得一提的是，不鼓励在任何类型的Map中使用putAll()，因为我们最终可能会得到键或值不是字符串的条目。

## 5. 使用Stream API合并属性

最后，我们将研究如何使用Stream API合并多个Properties对象：

```java
private Properties mergePropertiesByUsingStreamApi(Properties... properties) {
    return Stream.of(properties)
        .collect(Properties::new, Map::putAll, Map::putAll);
}
```

在最后一个示例中，我们从属性列表中创建一个Stream，然后使用collect方法将流中的值序列归约为新的Collection。**第一个参数是Supplier函数，用于创建一个新的结果容器，在我们的例子中是一个新的Properties对象**。

## 6. 总结

在这个简短的教程中，我们介绍了合并两个或多个Properties对象的三种不同方法。