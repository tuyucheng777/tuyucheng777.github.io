---
layout: post
title:  如何在Selenium中从Datepicker选择日期
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 简介

在本教程中，我们将查看一个使用Java的[Selenium WebDriver](https://www.baeldung.com/selenium-automated-browser-testing)通过日期选择器控件选择日期的简单示例。

对于此测试，我们将使用[JUnit](https://www.baeldung.com/junit-5)和Selenium打开页面[https://demoqa.com/automation-practice-form](https://demoqa.com/automation-practice-form)并使用日期选择器控件在“Date of Birth”字段中选择“2 Dec 2024”。

## 2. 依赖

首先，我们需要将[selenium-java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)和[webdrivermanager](https://mvnrepository.com/artifact/io.github.bonigarcia/webdrivermanager)依赖项添加到pom.xml文件中：

```xml
<dependency> 
    <groupId>org.seleniumhq.selenium</groupId> 
    <artifactId>selenium-java</artifactId> 
    <version>4.18.1</version>
</dependency>
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.7.0</version>
</dependency>
```

这些依赖允许我们运行Java代码来调用浏览器并执行操作。此外，我们还需要[JUnit](https://www.baeldung.com/junit-5)，因为我们将创建一些测试用例：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

我们准备将这些依赖添加到我们的项目中并创建一些测试。

## 3. 配置

接下来，我们需要配置WebDriver。我们将使用Chrome，并首先下载其[最新](https://googlechromelabs.github.io/chrome-for-testing/)版本：

```java
@BeforeEach
public void setUp() {
    WebDriverManager.chromedriver().setup();
    driver = new ChromeDriver();
}
```

我们使用带有@BeforeEach注解的方法在每次测试之前进行初始设置，**接下来，我们使用WebDriverManager获取Chrome驱动程序，而无需明确下载和安装它**。

**测试完成后，我们将关闭浏览器窗口**。我们将在@AfterEach方法中调用driver.close()，以确保即使测试失败也会执行它：

```java
@AfterEach
public void cleanUp() {
    driver.close();
}
```

## 4. 查找日期选择器元素

现在基本配置已完成，我们准备开始日期选择器测试。有[几种方法](https://www.baeldung.com/java-selenium-javascript)可以帮助Selenium选择元素，例如[使用ID](https://www.baeldung.com/java-selenium-html-input-value)、[CSS选择器](https://www.baeldung.com/selenium-find-element-by-attribute)或[Xpath](https://www.baeldung.com/java-xpath)。但是，日期选择器可能与常规输入元素不同。

### 4.1 了解日期选择器

日期选择器通常比其他输入元素更复杂，**虽然有日期输入类型，但许多网站并不使用标准输入类型**。

原因是，与其他输入类型不同，日期选择器可以具有不同的美感，并且通常，实现涉及后台的专门的HTML、CSS和JavaScript代码来定制日期选择器，例如添加品牌颜色。

单击主日期控件元素后，会显示另一组控件，这些控件提供了选择特定年份、月份和日期的机会。因此，我们需要识别年份、月份和日期输入元素的XPath。

首先声明一下我们将要访问的网站：

```java
private static final String URL = "https://demoqa.com/automation-practice-form";
```

### 4.2 查找日期控件

首先，我们与日期选择器控件的交互涉及选择控件以打开日期选择器，这通常是一个按钮或一个输入元素。对于我们的示例，它是一个text类型的输入元素。

此处的Xpath表达式查找具有id属性且值为dateOfBirthInput的input元素：

```java
private static final String INPUT_XPATH = "//input[@id='dateOfBirthInput']";
private static final String INPUT_TYPE = "text";
```

在开始编写实际测试之前，让我们创建一个简单的测试来确认日期选择器控件的可用性：

```java
@Test
public void givenDemoQAPage_whenFoundDateInput_thenHasAttributeType() {
    driver.get(URL);
    WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
    assertEquals(INPUT_TYPE, inputElement.getAttribute("type"));
}
```

## 5. 选择具体日期

到目前为止，我们已经添加了一个测试来识别并确认日期选择器是否存在。接下来，我们要选择一个特定的日期。这涉及多个交互，这些交互可能因实际的日期选择器元素而异。

让我们编写一个测试，选择2 Dec 2024这个日期，然后执行断言来检查是否选择了正确的日期。此测试将涉及四个步骤：

- 单击日期选择器输入元素
- 选择年份2024
- 选择月份December
- 选择日期值2

### 5.1 单击输入元素

首先，让我们单击代表日期控件的输入元素：

```java
driver.get(URL);
WebElement inputElement = driver.findElement(By.xpath(INPUT_XPATH));
inputElement.click();
```

### 5.2 选择年份

接下来，我们将定义年份的XPath。此Xpath查找具有类属性值react-datepicker__header的div元素，此div包含日期选择器UI。在其中，我们对具有类属性react-datepicker__year-select的select元素感兴趣：

```java
private static final String INPUT_YEAR_XPATH = "//div[@class='react-datepicker__header']"
    + "//select[@class='react-datepicker__year-select']";
```

然后，我们添加一个[显示等待](https://www.baeldung.com/selenium-implicit-explicit-wait)以允许JavaScript在点击后运行并显示Datepicker：

```java
Wait<WebDriver> wait = new FluentWait(driver);
WebElement yearElement = driver.findElement(By.xpath(INPUT_YEAR_XPATH)); 
wait.until(d -> yearElement.isDisplayed());
```

等待实际的日期选择器控件显示后，我们[从下拉菜单中选择2024年](https://www.baeldung.com/java-selenium-select-dropdown-value)：

```java
// Select Year
Select selectYear = new Select(yearElement);
selectYear.selectByVisibleText("2024");
```

### 5.3 选择月份

接下来，我们需要选择月份。因此，让我们创建相应的Xpath。这次，我们的Xpath查找具有class属性的select元素，其值为react-datepicker__month-select：

```java
private static final String INPUT_MONTH_XPATH = "//div[@class='react-datepicker__header']"
    + "//select[@class='react-datepicker__month-select']";
```

现在，我们可以使用它来选择月份选择器：

```java
WebElement monthElement = driver.findElement(By.xpath(INPUT_MONTH_XPATH)); 
wait.until(d -> monthElement.isDisplayed()); 
Select selectMonth = new Select(monthElement);
```

最后，我们选择实际月份-在本例中为December：

```java
// Select Month
selectMonth.selectByVisibleText("December");

```

我们在这里使用了显式等待，以确保在执行下一次点击之前运行对日期选择器进行更改的任何JavaScript。

### 5.4 选择日期

最后，我们准备选择感兴趣的特定日期。在本例中，我们寻找12月的第二天。**这里需要注意的是，日期选择器用户界面中可能会有同一天的多个值**。

通常，日期选择器界面还会显示上个月的最后几天或下个月的第一天。因此，选择特定日期所需的代码可能会变得更加复杂。让我们编写XPath表达式：

```java
private static final String INPUT_DAY_XPATH = "//div[contains(@class,\"react-datepicker__day\") and "
    + "contains(@aria-label,\"December\") and text()=\"2\"]";
```

在这个表达式中，我们使用“react-datepicker__day”类来选择所有代表月份天数的div元素。然后，我们在选择器中添加额外的“and”子句来检查aria-label是否为December，最后检查文本值是否为2，这确保我们得到一个匹配的元素。

现在，我们可以选择日期了：

```java
// Select Day
WebElement dayElement = driver.findElement(By.xpath(INPUT_DAY_XPATH));
wait.until(d -> dayElement.isDisplayed());
dayElement.click();
```

让我们用断言来完成我们的测试用例来检查我们是否选择了正确的日期：

```java
// Check selected date value
assertEquals("02 Dec 2024", inputElement.getAttribute("value"), "Wrong Date Selected");
```

## 6. 总结

在本文中，我们学习了如何使用Selenium通过日期选择器元素选择日期值。**请注意，日期选择器元素可能不是标准化的输入元素，并且选择值的方法可能很复杂且需要定制**。

常规流程使用多个选择器来识别日期选择器输入，然后按顺序选择年、月、日。通常，我们期望在选择日期后，日期输入元素中会显示完整的选定日期值，并可用于进一步的代码逻辑或测试。