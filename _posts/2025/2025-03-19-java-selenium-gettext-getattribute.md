---
layout: post
title:  Selenium WebDriver中getText()和getAttribute()之间的区别
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 简介

在本文中，我们将研究如何使用Java中的Selenium WebDriver获取网页上Web元素属性的值，我们还将探讨getText()和getAttribute()方法之间的区别。

为了进行测试，我们将使用[JUnit和Selenium](https://www.baeldung.com/java-selenium-with-junit-and-testng)打开[https://www.baeldung.com/contact](https://www.baeldung.com/contact)。该页面有一个名为“Your Name”的可见输入字段，我们将使用此字段来说明getText()和getAttribute()之间的区别。

## 2. 依赖

首先，我们在pom.xml中将[selenium-java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)和[Junit](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)依赖项添加到我们的项目中：

```xml
<dependency> 
    <groupId>org.seleniumhq.selenium</groupId> 
    <artifactId>selenium-java</artifactId> 
    <version>4.18.1</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.7.0</version>
</dependency>
```

## 3. 配置

接下来，我们需要配置WebDriver。在此示例中，我们将下载[最新版本](https://googlechromelabs.github.io/chrome-for-testing/)后使用其Chrome实现：

```java
@BeforeEach
public void setUp() {
    WebDriverManager.chromedriver().setup();
    driver = new ChromeDriver();
}
```

我们使用一个用@BeforeEach标注的方法在每次测试之前进行初始设置。接下来，**我们使用WebDriverManager获取Chrome驱动程序，而无需明确下载和安装它**。这使我们能够使用Selenium Web驱动程序，而无需使用驱动程序的绝对路径。不过仍然需要在运行此代码的目标机器上安装Chrome浏览器。

**测试完成后，我们应该关闭浏览器窗口**。我们可以通过将driver.close()语句放在用@AfterEach标注的方法中来实现这一点，这确保即使测试失败也会执行它：

```java
@AfterEach
public void cleanUp() {
    driver.close();
}
```

## 4. 使用getText()查找可见文本

现在脚手架已经准备好了，我们必须添加代码来识别相关的Web元素。有[几种方法可以帮助Selenium选择元素](https://www.baeldung.com/java-selenium-javascript)，例如[使用ID](https://www.baeldung.com/java-selenium-html-input-value)、[CSS选择器](https://www.baeldung.com/selenium-find-element-by-attribute)、类名、Xpath等。

联系页面有一个可见的输入字段，称为“Your Name”，我们使用浏览器开发工具(检查元素)来选择与此字段相关的HTML代码：

```html
<label> Your Name*<br>
    <span class="wpcf7-form-control-wrap" data-name="your-name">
<input size="40" maxlength="400" class="wpcf7-form-control wpcf7-text wpcf7-validates-as-required" aria-required="true" aria-invalid="false"
       value="" type="text" name="your-name">
</span>
</label>
```

### 4.1 测试可见文本

我们对标签上的可见文本感兴趣，即“Your Name”。**我们可以使用selenium Element上的getText()方法获取可见文本**。现在，从HTML摘录中可以明显看出，此标签元素除了可见文本外还包含许多其他代码。但是，getText()仅获取我们在浏览器上查看页面时可见的文本。

为了说明这一点，我们编写了一个测试，确认页面包含一个可见文本“Your Name”的字段。首先，我们定义一些常量：

```java
private static final String URL = "https://www.baeldung.com/contact";
private static final String LABEL_XPATH = "//label[contains(text(),'Your Name')]";
private static final String LABEL_TEXT = "Your Name*";
private static final String INPUT_XPATH = "//label[contains(text(),'Your Name')]//input";
```

然后我们添加一个测试方法来确认getText()返回的文本与可见文本“Your Name”完全匹配：

```java
@Test
public void givenBaeldungContactPage_whenFoundLabel_thenContainsText() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(LABEL_XPATH));
    assertEquals(LABEL_TEXT, inputElement.getText());
}
```

### 4.2 测试无可见文本

此外，**我们还注意到input元素没有可见的文本，因此，当我们在此元素上运行getText()方法时，我们期望得到一个空字符串**。我们添加另一个测试方法来确认这一点：

```java
@Test
public void givenBaeldungContactPage_whenFoundNameInput_thenContainsText() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    assertEquals("", inputElement.getText());
}
```

现在我们了解了getText()的工作原理，我们来回顾一下getAttribute()的用法。

## 5. 使用getAttribute()查找属性值

现在我们来研究一下Web元素上getAttribute()的用法，这次我们重点介绍名为“your-name”的input字段。它有许多属性，例如size、maxlength、value、type和name。**Web元素上的getAttribute()方法应返回与作为方法参数传递的属性关联的值，前提是Web元素上存在这样的属性。如果不存在这样的属性，则该方法返回空值**。

例如，如果我们查看label元素的HTML摘录，我们会注意到input元素有一个属性name，其值为“your-name”，另一个属性maxlength的值为400。

在编写涉及检查这些属性的测试时，我们使用getAttribute方法。我们首先为要检查的值定义常量：

```java
private static final String INPUT_NAME = "your-name";
private static final String INPUT_LENGTH = "400";
```

### 5.1 测试获取属性的值

让我们添加几个测试来检查名为name和maxlength的属性的属性值：

```java
@Test
public void givenBaeldungContactPage_whenFoundNameInput_thenHasAttributeName() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    assertEquals(INPUT_NAME, inputElement.getAttribute("name"));
}

@Test
public void givenBaeldungContactPage_whenFoundNameInput_thenHasAttributeMaxlength() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    assertEquals(INPUT_LENGTH, inputElement.getAttribute("maxlength"));
}
```

### 5.2 测试获取不存在的属性的值

接下来，我们编写一个测试来确认当我们使用getAttribute()查找不存在的属性X时，它返回一个空值：

```java
@Test
public void givenBaeldungContactPage_whenFoundNameInput_thenHasNoAttributeX() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    assertNull(inputElement.getAttribute("X"));
}
```

## 6. 总结

在本文中，我们学习了如何使用Java中的Selenium WebDriver获取网页上Web元素的可见文本和属性值。在浏览器上查看网页时，getText()仅显示元素上可见的纯文本，而getAttribute()允许我们获取Web元素上许多不同属性的值。

一般流程是使用Selenium选择器来识别感兴趣的Web元素，然后使用getText()或getAttribute()方法之一来获取有关Web元素的更多详细信息。

我们还添加了一些示例测试来演示这些方法在自动化测试中的用法。