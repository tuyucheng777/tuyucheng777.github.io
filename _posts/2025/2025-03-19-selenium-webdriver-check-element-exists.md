---
layout: post
title:  使用Selenium Webdriver检查元素是否存在
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 概述

在本教程中，我们将学习如何使用[Selenium WebDriver](https://www.baeldung.com/java-selenium-with-junit-and-testng)检查元素是否存在。

对于大多数检查，**最好使用[显式等待](https://www.baeldung.com/selenium-implicit-explicit-wait)来确保元素存在或可见，然后再与它们交互**。但是，有时我们只需要知道元素是否存在而不进行断言，这使我们能够根据元素的存在或不存在来实现特殊的附加逻辑。

在本教程结束时，我们将知道如何检查元素的存在。

## 2. 使用findElements()方法

**findElements()方法返回符合By条件的Web元素列表**，如果未找到匹配的元素，则返回空列表。通过检查列表是否为空，我们可以确定所需的元素是否存在：

```java
boolean isElementPresentCheckByList(By by) {
    List<WebElement> elements = driver.findElements(by);
    return !elements.isEmpty();
}
```

对于许多场景来说，这是一个简单且有效的解决方案。

## 3. 使用findElement()方法

findElement()方法是另一种常用的在网页上[定位元素](https://www.baeldung.com/selenium-find-element-by-attribute)的方法，如果存在，它将返回单个Web元素。**如果元素不存在，它将抛出NoSuchElementException**。

为了处理元素可能不存在的情况，我们应该使用try-catch块来捕获[异常](https://www.baeldung.com/java-exceptions)。如果引发异常，则意味着元素不存在，因此我们捕获它并返回false。当我们确定单个元素的存在对于测试中的下一步至关重要并且我们想要明确处理它的缺失时，此方法很有用。

通过捕获NoSuchElementException，我们可以记录适当的消息，采取纠正措施，或者正常退出测试而不会导致脚本崩溃：

```java
boolean isElementPresentCheckByHandleException(By by) {
    try {
        driver.findElement(by);
        return true;
    } catch (NoSuchElementException e) {
        return false;
    }
}
```

## 4. 总结

在本文中，我们探讨了使用Selenium WebDriver检查元素是否存在的两种基本方法：findElements()和带有异常处理的findElement()，这些方法可以帮助我们了解元素是否存在而不会导致测试失败。

但是，如果我们需要断言元素的存在，则**应该使用ExpectedConditions进行显式等待**。通过探索和理解这些不同的方法，我们可以自信地选择最合适的方法。