---
layout: post
title:  认知复杂度及其对代码的影响
category: designpattern
copyright: designpattern
excerpt: 认知复杂度
---

## 1. 概述

在本教程中，我们将学习什么是认知复杂度以及如何计算这个指标。我们将逐步讲解增加函数认知复杂度的不同模式和结构，我们将深入探讨这些元素，包括循环、条件语句、跳转标签、递归、嵌套等等。

接下来，我们将讨论认知复杂度对代码可维护性的负面影响。最后，我们将探讨一些有助于减轻这些负面影响的重构技术。

## 2. 圈复杂度与认知复杂度

曾几何时，圈复杂度是衡量代码复杂度的唯一方法。因此，出现了一种新的指标，可以让我们更准确地衡量一段代码的复杂度。虽然它提供了不错的总体评估，但它确实忽略了一些导致代码难以理解的重要方面。

### 2.1 圈复杂度

圈复杂度是最早用于测量代码复杂度的指标之一，它[由Thomas J.McCabe于1976年提出](https://en.wikipedia.org/wiki/Cyclomatic_complexity)，**他将函数的圈复杂度定义为相应代码段中所有自主路径的数量**。

例如，创建5个不同分支的switch语句将导致圈复杂度为5：

```java
public String tennisScore(int pointsWon) {
    switch (pointsWon) {
        case 0: return "Love"; // +1
        case 1: return "Fifteen"; // +1
        case 2: return "Thirty"; // +1
        case 3: return "Forty"; // +1
        default: throw new IllegalArgumentException(); // +1
    }
} // cyclomatic complexity = 5
```

虽然我们可以用这个指标来量化代码中不同路径的数量，**但我们无法精确地比较不同函数的复杂性**。它忽略了一些关键方面，例如多重嵌套、跳转标签(例如break或continue)、递归、复杂的布尔运算以及其他未能得到适当惩罚的因素。

结果，我们最终得到的函数客观上会更难理解和维护，但它们的圈复杂度并不一定更高。例如，countVowels的圈复杂度也是5：

```java
public int countVowels(String word) {
    int count = 0;
    for (String c : word.split("")) { // +1
        for(String v: vowels) { // +1
            if(c.equalsIgnoreCase(v)) { // +1
                count++;
            }
        }
    }
    if(count == 0) { // +1
        return "does not contain vowels";
    }
    return "contains %s vowels".formatted(count); // +1
}  // cyclomatic complexity = 5
```

### 2.2 认知复杂度

因此，**[Sonar](https://www.sonarsource.com/)开发了[认知复杂度](https://www.sonarsource.com/resources/cognitive-complexity/)指标，其主要目标是提供可靠的代码可理解性度量**。其根本动机是促进重构实践，以实现良好的代码质量和可读性。

尽管我们可以配置静态代码分析器(例如[SonarQube](https://www.baeldung.com/sonar-qube))来自动计算代码的认知复杂度，但让我们了解如何计算认知复杂度分数以及考虑了哪些主要原则。

首先，简化代码的结构不会带来任何损失，从而提高代码的可读性。例如，我们可以想象提取一个函数或引入提前返回来减少代码的嵌套层数。

其次，线性流程的每一次中断，认知复杂度都会增加。循环、条件语句、try-catch块以及其他类似的结构都会破坏这种线性流程，因此，它们会使复杂性级别增加一级。目标是以线性流程的方式阅读所有代码，从上到下、从左到右。

最后，嵌套会导致额外的复杂度惩罚。因此，如果我们回顾之前的代码示例，会发现使用switch语句的TennisScore函数的认知复杂度为1。另一方面，CountVowels函数会因为嵌套循环而受到严重惩罚，导致复杂度达到7级：

```java
public String countVowels(String word) {
    int count = 0;
    for (String c : word.split("")) { // +1
        for(String v: vowels) { // +2 (nesting level = 1)
            if(c.equalsIgnoreCase(v)) { // +3 (nesting level = 2)
                count++;
            }
        }
    }
    if(count == 0) { // +1
        return "does not contain vowels";
    }
    return "contains %s vowels".formatted(count);
} // cognitive complexity = 7
```

## 3. 线性流程的中断

如上一节所述，我们应该能够从头到尾流畅地、不间断地阅读认知复杂度最低的代码。然而，一些破坏代码自然流畅性的元素将受到惩罚，从而增加复杂度。以下结构就是这种情况：

- 语句：if、三元运算符、switch
- 循环：for、while、do while
- try-catch块
- 递归
- 跳转到标签：继续、中断
- 逻辑运算符序列

现在，让我们看一个简单的方法示例，并尝试找到使代码可读性较差的这些结构：

```java
public String readFile(String path) {
    // +1 for the if; +2 for the logical operator sequences ("or" and "not")
    String text = null;
    if(path == null || path.trim().isEmpty() || !path.endsWith(".txt")) {
        return DEFAULT_TEXT;
    }

    try {
        text = "";
        // +1 for the loop
        for (String line: Files.readAllLines(Path.of(path))) {
            // +1 for the if statement
            if(line.trim().isEmpty()) {
                // +1 for the jump-to label
                continue OUT;
            }
            text+= line;
        }
    } catch (IOException e) { // +1 for the catch block
        // +1 for if statement
        if(e instanceof FileNotFoundException) {
            log.error("could not read the file, returning the default content..", e);
        } else {
            throw new RuntimeException(e);
        }
    }
    // +1 for the ternary operator
    return text == null ? DEFAULT_TEXT : text;
}
```

就目前情况而言，该方法的现有结构无法实现无缝的线性流程，**我们讨论过的破坏流程的结构将使认知复杂度级别增加9级**。

## 4. 嵌套的流程中断结构

随着嵌套作用域的层数增加，代码的可读性会逐渐降低。因此，后续嵌套的if、else、catch、switch、循环和lambda表达式，每增加一层，认知复杂度都会额外增加1。回顾前面的例子，我们会发现**两个地方深度嵌套会导致复杂度得分额外下降**：

```java
public String readFile(String path) {
    String text = null;
    if(path == null || path.trim().isEmpty() || !path.endsWith(".txt")) {
        return DEFAULT_TEXT;
    }
    try {
        text = "";
        // nesting level is 1
        for (String line: Files.readAllLines(Path.of(path))) {
            // nesting level is 2 => complexity +1
            if(line.trim().isEmpty()) {
                continue OUT;
            }
            text+= line;
        }
        // nesting level is 1
    } catch (IOException e) {
        // nesting level is 2 => complexity +1
        if(e instanceof FileNotFoundException) {
            log.error("could not read the file, returning the default content..", e);
        } else {
            throw new RuntimeException(e);
        }
    }
    return text == null ? DEFAULT_TEXT : text;
}
```

因此，该方法的认知复杂度为11，准确地反映了其在可读性和理解方面的难度。然而，**通过重构，我们可以显著降低其认知复杂度，并提升其整体可读性**。我们将在下一节深入探讨重构过程的具体细节。

## 5. 重构

我们可以使用多种重构技术来降低代码的认知复杂度，让我们逐一探讨每种重构技术，并重点介绍IDE如何促进其安全高效地执行。

### 5.1 提取代码

**一种有效的方法是提取方法或类，因为它使我们能够精简代码而不会产生任何副作用**。在本例中，我们可以利用方法提取来验证filePath参数，从而增强整体的清晰度。

大多数IDE都允许你使用简单的快捷键或重构菜单自动执行此操作，例如，在IntelliJ中，我们可以通过高亮相应行并使用Ctrl + Alt + M(或Ctrl + Enter)快捷键来提取hasInvalidPath方法：

![](/assets/images/2025/designpattern/javacognitivecomplexity01.png)

```java
private boolean hasInvalidPath(String path) {
    return path == null || path.trim().isEmpty() || !path.endsWith(".txt");
}
```

### 5.2 反转条件

根据具体情况，有时反转简单的if语句可以方便地减少代码的嵌套层数。在我们的示例中，我们可以反转if语句，检查行是否为空，并避免使用continue关键字。IDE再次为这个简单的重构提供了便利：在Intellij中，我们需要高亮显示if语句，然后按下Alt + Enter键：

![](/assets/images/2025/designpattern/javacognitivecomplexity02.png)

### 5.3 语言特性

我们还应该尽可能利用语言特性来避免破坏流程的结构，例如，我们可以使用多个catch块来分别处理异常，这将有助于我们避免使用额外的if语句来增加嵌套层数。

### 5.4 提前Return

提前返回也可以使方法更简洁、更容易理解，在这种情况下，提前返回可以帮助我们处理函数末尾的三元运算符。

我们可以注意到，我们有机会为text变量引入提前返回的功能，并通过返回DEFAULT_TEXT来处理FileNotFoundException的发生。因此，我们可以通过缩小text变量的作用域来改进代码，这可以通过将其声明移到更靠近其使用位置来实现(在IntelliJ中按Alt + M)。

这种调整增强了代码的组织性，并避免了使用null：

![](/assets/images/2025/designpattern/javacognitivecomplexity03.png)

### 5.5 声明式代码

最后，声明式模式通常可以降低代码的嵌套层级和复杂性。例如，[Java Stream](https://www.baeldung.com/java-streams)可以帮助我们使代码更紧凑、更全面。让我们使用Files.lines()(它返回Stream<String\>)来代替File.readAllLines()。此外，由于它们使用相同的返回值，我们可以在初始路径验证后立即检查文件是否存在。

生成的代码仅对if语句和执行初始参数验证的逻辑运算有两个惩罚：

```java
public String readFile(String path) {
    // +1 for the if statement;  +1 for the logical operation
    if(hasInvalidPath(path) || fileDoesNotExist(path)) {
        return DEFAULT_TEXT;
    }
    try {
        return Files.lines(Path.of(path))
            .filter(not(line -> line.trim().isEmpty()))
            .collect(Collectors.joining(""));
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

## 6. 总结

Sonar开发了认知复杂度指标，因为需要一种精确的方法来评估代码的可读性和可维护性。在本文中，我们介绍了计算函数认知复杂度的过程。

之后，我们研究了那些扰乱代码线性流程的结构。最后，我们讨论了各种代码重构技巧，这些技巧使我们能够降低函数的认知复杂度。我们利用IDE的功能重构了一个函数，并将其复杂度得分从11分降低到了2分。