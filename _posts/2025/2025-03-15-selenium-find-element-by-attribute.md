---
layout: post
title:  在Selenium中按属性查找元素
category: selenium
copyright: selenium
excerpt: Selenium
---

## 1. 简介

[Selenium](https://www.baeldung.com/java-selenium-with-junit-and-testng)提供了很多方法来定位网页上的元素，我们经常需要根据元素的属性来查找元素。

属性是可以添加以提供更多上下文或功能的附加信息，它们大致可分为两类：

- 标准属性：这些属性是预定义的，并被浏览器识别，示例包括id、class、src、href、alt、title等。标准属性具有预定义的含义，并广泛应用于不同的HTML元素。
- 自定义属性：自定义属性是HTML规范未预定义的属性，但通常由开发人员根据其特定需求创建。这些属性通常以“data-”开头，后跟描述性名称。例如，data-id、data-toggle、data-target等。自定义属性可用于存储与元素相关的附加信息或元数据，它们通常用于Web开发，以在HTML和JavaScript之间传递数据。

在本教程中，我们将深入研究如何使用CSS来精确定位网页上的元素。我们将探索如何通过属性名称或描述来查找元素，并提供完全匹配和部分匹配选项。到最后，我们将能够轻松找到页面上的任何元素！

## 2. 根据属性名称查找元素

最简单的场景之一是根据特定属性的存在来查找元素，假设一个网页上有多个按钮，每个按钮都标有一个名为“data-action”的自定义属性。现在，假设我们想要找到页面上具有此属性的所有按钮。在这种情况下，我们可以使用[attribute\]定位器：

```java
driver.findElements(By.cssSelector("[data-action]"));
```

在上面的代码中，[data-action\]将选择页面上具有目标属性的所有元素，并且我们将得到一个WebElements列表。

## 3. 根据属性值查找元素

**当我们需要定位具有唯一属性值的具体元素时，我们可以使用CSS定位器[attribute=value\]的严格匹配变体**，此方法允许我们找到具有精确属性值匹配的元素。

让我们继续我们的网页，其中按钮有一个“data-action”属性，每个按钮都分配有一个不同的操作值，例如，data-action='delete'、data-action='edit'等等。如果我们想找到具有特定操作(例如“delete”)的按钮，我们可以使用精确匹配的属性选择器：

```java
driver.findElement(By.cssSelector("[data-action='delete']"));
```

## 4. 根据起始属性值查找元素

在确切的属性值可能有所不同但从特定子字符串开始的情况下，我们可以使用另一种方法。

让我们考虑这样一个场景：我们的应用程序有许多弹出窗口，每个弹出窗口都有一个“Accept”按钮，该按钮带有一个名为“data-action”的自定义属性。**这些按钮可能在共享前缀后附加了不同的标识符**，例如“btn_accept_user_pop_up”、“btn_accept_document_pop_up”等等。我们可以使用[attribute^=value\]定位器在基类中编写一个通用定位器：

```java
driver.findElement(By.cssSelector("[data-action^='btn_accept']"));
```

该定位器将找到“data-action”属性值以“btn_accept”开头的元素，因此我们可以为每个弹出窗口编写一个带有通用“Accept”按钮定位器的基类。

## 5. 通过结束属性值查找元素

类似地，**当我们的属性值以特定后缀结尾时**，我们使用[attribute$=value\]。想象一下，我们在页面上有一个URL列表，并希望获取所有PDF文档引用：

```java
driver.findElements(By.cssSelector("[href$='.pdf']"));
```

在此示例中，驱动程序将找到所有“href”属性值以“.pdf”结尾的元素。

## 6. 根据属性值的部分查找元素

当我们不确定我们的属性前缀或后缀时，包含方法[attribute*=value\]会很有帮助。让我们考虑一个场景，我们想要获取所有引用特定资源路径的元素：

```java
driver.findElements(By.cssSelector("[href*='archive/documents']"));
```

在此示例中，我们将从archive文件夹中接收有关文档的所有元素。

## 7. 根据特定类别查找元素

**我们可以通过使用元素的类作为属性来定位元素**，这是一种常用技术，尤其是在检查元素是否已启用、已禁用或已获得其类中反映的其他功能时。假设我们要查找已禁用的按钮：

```xml
<a href="#" class="btn btn-primary btn-lg disabled" role="button" aria-disabled="true">Accept</a>
```

这次，让我们对角色使用严格匹配，并包含对类的匹配：

```java
driver.findElement(By.cssSelector("[role='button'][class*='disabled']"));
```

在这个例子中，“class”被用作属性定位器[attribute*=value\]，并在值“btn btn-primary btn-lg disabled”中检测到“disabled”部分。

## 8. 总结

在本教程中，我们探索了根据元素的属性定位元素的不同方法。

我们将属性分为两种主要类型：标准属性，浏览器可以识别并具有预定义的含义；自定义属性，由开发人员创建，以满足特定要求。

使用CSS选择器，我们学会了如何根据属性名称、值、前缀、后缀甚至子字符串高效地查找元素。了解这些方法为我们提供了强大的工具来轻松定位元素，使我们的自动化任务更顺畅、更高效。