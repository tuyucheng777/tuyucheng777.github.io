---
layout: post
title:  将YAML字符串拆分为多行
category: libraries
copyright: libraries
excerpt: SnakeYAML
---

## 1. 概述

在本文中，我们将学习如何将YAML字符串拆分为多行。

为了解析和测试我们的YAML文件，我们将使用[SnakeYAML库](https://www.baeldung.com/java-snake-yaml)。

## 2. 多行字符串

在开始之前，让我们创建一种方法，简单地将文件中的YAML键读入字符串：

```java
String parseYamlKey(String fileName, String key) {
    InputStream inputStream = this.getClass()
        .getClassLoader()
        .getResourceAsStream(fileName);
    Map<String, String> parsed = yaml.load(inputStream);
    return parsed.get(key);
}
```

**在接下来的小节中，我们将介绍一些将字符串拆分为多行的策略**。

我们还将了解YAML如何处理块开头和结尾的空行所表示的前导换行符和结束换行符。

## 3. 文字风格

文字运算符由管道符号(“|”)表示，它保留了换行符，但将字符串末尾的空行减少为单个换行符。

让我们看一下YAML文件literal.yaml：

```yaml
key: |
    Line1
    Line2
    Line3
```

我们可以看到我们的换行符被保留了：

```java
String key = parseYamlKey("literal.yaml", "key");
assertEquals("Line1\nLine2\nLine3", key);
```

接下来，让我们看一下literal2.yaml，它有一些前导和结束换行符：

```yaml
key: |


    Line1

    Line2

    Line3


...
```

我们可以看到，除了结束换行符外，每个换行符都存在，而结束换行符减少为一个：

```java
String key = parseYamlKey("literal2.yaml", "key");
assertEquals("\n\nLine1\n\nLine2\n\nLine3\n", key);
```

接下来，我们将讨论块吞噬以及它如何让我们更好地控制开始和结束换行符。

我们可以使用两种chomping方法来改变默认行为：keep和strip。

### 3.1 Keep

正如我们在literal_keep.yaml中看到的那样，保留用“+”表示：

```yaml
key: |+
    Line1
    Line2
    Line3


...
```

通过覆盖默认行为，我们可以看到每个结束的空行都被保留：

```java
String key = parseYamlKey("literal_keep.yaml", "key");
assertEquals("Line1\nLine2\nLine3\n\n", key);
```

### 3.2 条带

正如我们在literal_strip.yaml中看到的，条带用“-”表示：

```yaml
key: |-
    Line1
    Line2
    Line3

...
```

正如我们所料，这将删除所有结尾的空行：

```java
String key = parseYamlKey("literal_strip.yaml", "key");
assertEquals("Line1\nLine2\nLine3", key);复制
```

## 4. 折叠

折叠运算符用“\>”表示，正如我们在folded.yaml中看到的那样：

```yaml
key: >
    Line1
    Line2
    Line3
```

**默认情况下，连续非空行的换行符将替换为空格字符**：

```java
String key = parseYamlKey("folded.yaml", "key");
assertEquals("Line1 Line2 Line3", key);
```

让我们看一个类似的文件folded2.yaml，它有几个结尾的空行：

```yaml
key: >
    Line1
    Line2


    Line3


...
```

我们可以看到**空行被保留了下来，但是结束换行符也减少为一个**：

```java
String key = parseYamlKey("folded2.yaml", "key");
assertEquals("Line1 Line2\n\nLine3\n", key);
```

我们应该牢记，**块chomping对折叠风格的影响与对文字风格的影响是相同的**。

## 5. 引号

让我们快速了解如何使用双引号和单引号来拆分字符串。

### 5.1 双引号

使用双引号，我们可以轻松地使用“\n”创建多行字符串：

```yaml
key: "Line1\nLine2\nLine3"
```

```java
String key = parseYamlKey("plain_double_quotes.yaml", "key");
assertEquals("Line1\nLine2\nLine3", key);
```

### 5.2 单引号

另一方面，单引号将“\n”视为字符串的一部分，因此插入换行符的唯一方法是使用空行：

```yaml
key: 'Line1\nLine2

  Line3'
```

```java
String key = parseYamlKey("plain_single_quotes.yaml", "key");
assertEquals("Line1\\nLine2\nLine3", key);
```

## 6. 总结

在本快速教程中，我们通过快速实用的示例了解了将YAML字符串拆分为多行的多种方法。