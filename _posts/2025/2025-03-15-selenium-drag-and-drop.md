---
layout: post
title:  如何在Selenium中拖放
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 概述

在本教程中，我们将探索如何使用[Selenium和WebDriver](https://www.baeldung.com/selenium-webdriver-page-object)执行拖放操作。拖放功能通常用于Web应用程序中，从重新排列页面上的元素到处理文件上传，使用Selenium自动执行此任务有助于我们涵盖用户交互。

我们将讨论Selenium提供的三种核心方法：dragAndDrop()、clickAndHold()和dragAndDropBy()。这些方法允许我们模拟具有不同控制级别的拖放操作，从元素之间的基本移动到使用偏移的更复杂、更精确的拖动。在整个教程中，我们将在不同的测试页面上演示它们的用法。

## 2. 设置和配置

在开始拖放之前，我们需要设置环境。我们将使用[Selenium Java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java/)库和[WebDriverManager](https://mvnrepository.com/artifact/io.github.bonigarcia/webdrivermanager/)来管理浏览器驱动程序。我们的示例使用Chrome，但任何其他受支持的浏览器都可以类似地使用。

看看我们需要在pom.xml中包含的依赖项：

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.27.0</version>
</dependency>
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.9.2</version>
</dependency>
```

接下来，初始化我们的WebDriver：

```java
private WebDriver driver;

@BeforeEach
public void setup() {
    driver = new ChromeDriver();
    driver.manage().window().maximize();
}

@AfterEach
public void tearDown() {
    driver.quit();
}
```

我们将使用[演示页面](http://the-internet.herokuapp.com/drag_and_drop)来展示我们的拖放示例。

## 3. 使用dragAndDrop()方法

在Web开发中，元素通常设置为可拖动或可放置。**可拖动元素可以单击并在屏幕上移动；另一方面，可放置元素可用作放置可拖动元素的目标**。例如，在任务管理应用程序中，我们可以拖动任务(可拖动)并将其放置到另一个部分(可放置)，以更有效地组织任务。

Selenium的dragAndDrop()方法是一种直接模拟将元素从一个位置拖到另一个位置的方法，特别是从可拖动元素拖到可放置元素。此方法需要指定源(draggable)元素和目标(droppable)元素。

让我们来看看如何实现它：

```java
@Test
public void givenTwoElements_whenDragAndDropPerformed_thenElementsSwitched() {
    String url = "http://the-internet.herokuapp.com/drag_and_drop";
    driver.get(url);

    WebElement sourceElement = driver.findElement(By.id("column-a"));
    WebElement targetElement = driver.findElement(By.id("column-b"));

    Actions actions = new Actions(driver);
    actions.dragAndDrop(sourceElement, targetElement)
        .build()
        .perform();
}
```

在此测试中，我们首先导航到测试页面，通过ID找到源元素和目标元素，然后使用dragAndDrop()执行拖放操作。此方法会自动处理移动，因此适合于我们想要模拟基本拖放行为的简单用例。

## 4. 使用clickAndHold()方法

有时，我们需要对拖放操作进行更多控制。例如，如果我们想模拟悬停在中间元素上或在页面上缓慢拖动，则**clickAndHold()方法可让我们对拖动序列的每个部分进行更精细的控制**。

以下是我们如何实现它：

```java
@Test
public void givenTwoElements_whenClickAndHoldUsed_thenElementsSwitchedWithControl() {
    String url = "http://the-internet.herokuapp.com/drag_and_drop";
    driver.get(url);

    WebElement sourceElement = driver.findElement(By.id("column-a"));
    WebElement targetElement = driver.findElement(By.id("column-b"));

    Actions actions = new Actions(driver);
    actions.clickAndHold(sourceElement)
        .moveToElement(targetElement)
        .release()
        .build()
        .perform();
}
```

在本例中，我们使用clickAndHold()单击源元素，使用moveToElement()将其移动到目标，使用release()将其放下。这种方法的灵活性使我们能够模拟诸如在屏幕上拖动或与中间元素交互之类的场景。

在测试需要复杂动作或多个步骤的功能时，额外的控制很有用，例如将项目拖到页面的特定部分或调整滑块。

## 5. 对可拖动元素使用dragAndDropBy()

在某些情况下，我们可能需要将元素拖到准确位置，而不是将其放到目标上。**dragAndDropBy()方法允许我们沿X和Y轴将元素拖动特定数量的像素**，这对于测试滑块等UI元素或处理可调整大小或可移动元素时特别有用。

让我们使用[jqueryui draggable演示页面](https://jqueryui.com/draggable/)来演示这种方法：

```java
@Test
public void givenDraggableElement_whenDragAndDropByUsed_thenElementMovedByOffset() {
    String url = "https://jqueryui.com/draggable/";
    driver.get(url);
    driver.switchTo().frame(0);

    WebElement draggable = driver.findElement(By.id("draggable"));

    Actions actions = new Actions(driver);
    actions.dragAndDropBy(draggable, 100, 100)
        .build()
        .perform();
}
```

在这个测试中，我们首先导航到jqueryui draggable演示页面，切换到draggable元素所在的iframe，然后使用dragAndDropBy()方法将元素沿X和Y轴分别移动100像素的偏移量。

**dragAndDropBy()方法让我们可以灵活地将元素移动到页面上的任何位置，而无需特定的可拖放目标**。这在测试滑块、面板或图像库等元素时特别有用，因为精确的移动至关重要。

## 6. 总结

在本文中，我们讨论了在Selenium中自动执行拖放功能的各种方法。我们从dragAndDrop()方法开始，该方法在指定的源位置和目标位置之间移动元素，此方法非常适合涉及可拖动和可放置元素的场景。

接下来，我们研究了clickAndHold()方法，该方法为复杂的动作(如缓慢拖动或与中间元素交互)提供了更多控制。最后，我们讨论了dragAndDropBy()方法，该方法允许使用偏移量精确定位元素。