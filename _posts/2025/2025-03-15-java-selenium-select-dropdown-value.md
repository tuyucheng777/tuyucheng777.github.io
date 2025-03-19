---
layout: post
title:  如何使用Selenium Webdriver从下拉列表中选择值
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 简介

在这个简短的教程中，我们将演示一个简单的示例，说明如何使用Java中的[Selenium](https://www.selenium.dev/) WebDriver从下拉元素中选择一个选项或值。

为了测试，我们将使用[JUnit和Selenium](https://www.baeldung.com/java-selenium-with-junit-and-testng)打开[https://www.baeldung.com/contact](https://www.baeldung.com/contact)并从“What is your question about?”下拉菜单中选择值“Bug Reporting”。

## 2. 依赖

首先，在pom.xml中将[selenium-java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)和[Junit](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)依赖项添加到我们的项目中：

```xml
<dependency> 
    <groupId>org.seleniumhq.selenium</groupId> 
    <artifactId>selenium-java</artifactId> 
    <version>4.18.1</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.2</version> <scope>test</scope>
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

我们使用一个带有@BeforeEach注解的方法在每次测试之前进行初始设置。接下来，**我们使用WebDriverManager获取Chrome驱动程序，而无需明确下载和安装它**。与使用驱动程序的绝对路径相比，这使我们的代码更具可移植性，不过仍然需要在运行此代码的目标机器上安装Chrome浏览器。

**测试完成后，我们应该关闭浏览器窗口**。我们可以通过将driver.close()语句放在用@AfterEach注解的方法中来实现这一点。这确保即使测试失败，它也会执行：

```java
@AfterEach
public void cleanUp() {
    driver.close();
}
```

## 4. 查找Select元素

现在基本骨架已经完成，我们必须添加代码来识别Select元素。有几种不同的方法可以帮助[Selenium选择元素](https://www.baeldung.com/java-selenium-javascript)，例如[使用ID](https://www.baeldung.com/java-selenium-html-input-value)、[CSS选择器](https://www.baeldung.com/selenium-find-element-by-attribute)、类名、Xpath等。

我们为页面URL添加变量，并为select输入和感兴趣的选项添加XPath，稍后的测试中会用到这些：

```java
private static final String URL = "https://www.baeldung.com/contact";
private static final String INPUT_XPATH = "//select[@name='question-recipient']";
private static final String OPTION_XPATH = 
  "//select[@name='question-recipient']/option[@value='Bug reporting']";
```

我们将使用XPath选择器，因为它们在选择与各种选项匹配的元素方面提供了很大的灵活性和强大功能。

在此示例中，我们对select元素下的选项“Bug Reporting”感兴趣，该元素的属性名称为“question-recipient”。**使用Xpath选择器，我们可以使用单个表达式毫无歧义地选择该精确元素**。 

然后，我们添加一个测试用例来确认我们感兴趣的选项是否存在。在本例中，该选项是带有文本“Bug Reporting”的选项：

```java
@Test
public void givenBaeldungContactPage_whenSelectQuestion_thenContainsOptionBugs() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(OPTION_XPATH));
    assertEquals("Bug reporting", inputElement.getText());
}
```

在这里，我们使用driver.get()方法来加载网页。接下来，我们找到选项元素并验证其文本是否符合我们的预期值，以确保我们处于正确的位置。接下来，我们将编写一些测试来点击该特定选项。

## 5. 在Select元素中选择一个选项

我们已经确定了我们感兴趣的选项，**Selenium在org.openqa.selenium.support.ui.Select包下提供了一个名为Select的单独类，以帮助使用HTML选择下拉菜单**。我们将编写一些测试来使用不同的方法单击我们感兴趣的选项。

### 5.1 按值选择

这里我们使用Select类中的selectByValue()方法根据其代表的值选择一个选项：

```java
@Test
public void givenBaeldungContactPage_whenSelectQuestion_thenSelectOptionBugsByValue() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    WebElement optionBug = driver.findElement(By.xpath(OPTION_XPATH));
    Select select = new Select(inputElement);
    select.selectByValue(OPTION_TEXT);
    assertTrue(optionBug.isSelected());
}
```

### 5.2 按选项文本选择

这里我们使用Select类中的selectByVisibleText()方法来选择相同的选项，这次使用对于我们作为最终用户可见的文本：

```java
@Test
public void givenBaeldungContactPage_whenSelectQuestion_thenSelectOptionBugsByText() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    WebElement optionBug = driver.findElement(By.xpath(OPTION_XPATH));
    select.selectByVisibleText(OPTION_TEXT);    
    assertTrue(optionBug.isSelected());
}
```

### 5.3 通过索引选择

考虑这一点的最后一种方法是使用要选择的选项的索引，感兴趣的选项在下拉列表中列为第五个选项。我们注意到，此库中的索引以0为第一个索引，因此我们对索引4感兴趣。我们使用selectByIndex()方法来选择相同的选项：

```java
@Test
public void givenBaeldungContactPage_whenSelectQuestion_thenSelectOptionBugsByIndex() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    WebElement optionBug = driver.findElement(By.xpath(OPTION_XPATH));
    select.selectByIndex(OPTION_INDEX);    
    assertTrue(optionBug.isSelected());
}
```

## 6. 总结

在本文中，我们学习了如何使用Selenium从选择元素中选择一个值。

我们研究了几种选择我们感兴趣的选项的不同方法，Selenium为此提供了一个名为Select的特殊支持类。包含的方法是selectByValue()、selectByVisibleText()和selectByIndex() 。

一般流程是使用Selenium选择器来识别感兴趣的下拉菜单，然后使用Select类上的方法之一单击所需的选项。