---
layout: post
title:  如何在Selenium中处理警报和弹出窗口
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 概述

在本教程中，我们将探讨如何处理[Selenium](https://www.baeldung.com/java-selenium-with-junit-and-testng)中的警报和弹出窗口。警报和弹出窗口是可能中断自动化脚本流程的常见元素，因此有效地管理它们对于确保顺利执行测试至关重要。

首先，我们需要了解警报和弹出窗口有多种形式，需要不同的处理技术。

简单警报是需要确认的基本通知，通常通过“OK”按钮([HTML浏览器标准](https://developer.mozilla.org/en-US/docs/Web/API/Window/alert)的一部分)进行确认。确认警报提示用户接受或拒绝操作，而提示警报则请求用户输入。此外，弹出窗口可以显示为单独的浏览器窗口或模态对话框。

在本教程中，我们将研究在Web测试期间可能遇到的警报和弹出窗口类型。我们将演示如何使用Selenium与每个元素进行交互，以确保我们的测试不间断地进行。

## 2. 设置和配置

为了处理警报和弹出窗口，我们首先需要使用两个必需的依赖项来设置我们的环境：[Selenium Java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java/latest)库，它提供自动化浏览器的核心功能，以及[WebDriverManager](https://mvnrepository.com/artifact/io.github.bonigarcia/webdrivermanager/latest)，它对于通过自动下载和配置正确的版本来管理浏览器驱动程序至关重要。

首先，让我们在项目的pom.xml文件中包含所需的依赖项：

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.23.1</version>
</dependency>
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.8.0</version>
</dependency>
```

一旦设置了依赖，我们就会初始化一个新的ChromeDriver实例来自动化Google Chrome，也可以轻松修改此配置以适应其他浏览器：

```java
private WebDriver driver;

@BeforeEach
public void setup() {
    driver = new ChromeDriver();
}

@AfterEach
public void tearDown() {
    driver.quit();
}
```

## 3. 处理简单警报

在本节中，我们将重点介绍使用Selenium处理简单警报所需的实际步骤。**简单警报是带有文本和“OK”按钮的基本警报窗口**：

![](/assets/images/2025/automatedtest/javaseleniumhandlealertspopups01.png)

我们将导航到演示简单警报的示例网页，并编写JUnit测试来触发和管理警报。[JavaScript警报测试页](https://testpages.herokuapp.com/styled/alerts/alert-test.html)提供了各种类型警报的示例，包括我们旨在处理的简单警报：

```java
@Test
public void whenSimpleAlertIsTriggered_thenHandleIt() {
    driver.get("https://testpages.herokuapp.com/styled/alerts/alert-test.html");
    driver.findElement(By.id("alertexamples")).click();

    Alert alert = driver.switchTo().alert();
    String alertText = alert.getText();
    alert.accept();
    assertEquals("I am an alert box!", alertText);
}
```

在这个测试中，我们初始化WebDriver并导航到测试页面，单击“Show alert box”按钮即可触发简单警报。一旦触发警报，我们就使用Selenium的switchTo().alert()方法将控件从主浏览器窗口切换到警报窗口。

一旦进入警告窗口，我们现在就可以使用Alert接口提供的方法与其进行交互。对于简单的警告框，我们通过使用方法alert.accept()单击警告框上的“OK”按钮来处理它。除了接受或关闭警告外，我们还可以使用其他有用的方法(例如alert.getText())从警告窗口提取文本：

```java
String alertText = alert.getText();
```

**以这种方式处理警报至关重要，因为如果不这样做，自动脚本将遇到异常**。让我们通过触发警报、故意不处理它并尝试单击另一个元素来测试此行为：

```java
@Test
public void whenAlertIsNotHandled_thenThrowException() {
    driver.get("https://testpages.herokuapp.com/styled/alerts/alert-test.html");
    driver.findElement(By.id("alertexamples")).click();

    assertThrows(UnhandledAlertException.class, () -> {
        driver.findElement(By.xpath("/html/body/div/div[1]/div[2]/a[2]")).click();
    });
}
```

测试用例的结果证实，当在尝试与页面上的其他元素交互之前未处理警报时，将抛出UnhandledAlertException。

## 4. 处理确认警报

确认警报与简单警报略有不同，它们通常在用户操作需要确认时出现，例如删除记录或提交敏感信息。与仅显示“OK”按钮的简单警报不同，确认警报提供两个选项：“OK”以确认操作或“Cancel”以关闭操作。

![](/assets/images/2025/automatedtest/javaseleniumhandlealertspopups02.png)

为了演示如何处理确认警报，我们将继续使用[JavaScript警报测试页](https://testpages.herokuapp.com/styled/alerts/alert-test.html)。**我们的目标是触发确认警报并通过接受和关闭它来与其进行交互，然后验证结果**。

让我们看看如何在Selenium中处理确认警报：

```java
@Test
public void whenConfirmationAlertIsTriggered_thenHandleIt() {
    driver.get("https://testpages.herokuapp.com/styled/alerts/alert-test.html");
    driver.findElement(By.id("confirmexample")).click();

    Alert alert = driver.switchTo().alert();
    String alertText = alert.getText();
    alert.accept();
    assertEquals("true", driver.findElement(By.id("confirmreturn")).getText());

    driver.findElement(By.id("confirmexample")).click();
    alert = driver.switchTo().alert();
    alert.dismiss();
    assertEquals("false", driver.findElement(By.id("confirmreturn")).getText());
}
```

在此测试中，我们首先浏览页面并通过单击“Show confirm box”按钮触发确认警报。使用switchTo().alert()，我们将控件切换为关注警报并捕获文本以进行验证。然后使用accept()方法接受警报，并检查页面上显示的结果以确认操作已成功完成。

为了进一步演示确认警报的处理，测试再次触发警报，但这次，我们使用了dismiss()方法来取消操作。在取消操作后，我们验证相应的操作是否已正确中止。

## 5. 处理提示警报

与简单警报和确认警报相比，提示警报是一种更具交互性的浏览器警报形式。与仅向用户显示消息的简单警报和确认警报不同，**提示警报会显示消息和文本输入字段，用户可以在其中输入响应**：

![](/assets/images/2025/automatedtest/javaseleniumhandlealertspopups03.png)

当网页上的操作需要用户输入时，通常会出现提示警报。在Selenium中处理这些警报涉及将所需的输入文本发送到警报，然后通过接受输入或关闭警报来管理响应。

为了演示如何处理提示警报，我们将使用相同的测试页面来触发提示警报。我们的目标是触发警报，通过提交输入与其进行交互，并验证是否处理了正确的输入。

让我们看一个示例，了解如何在Selenium中处理提示警报：

```java
@Test
public void whenPromptAlertIsTriggered_thenHandleIt() {
    driver.get("https://testpages.herokuapp.com/styled/alerts/alert-test.html");
    driver.findElement(By.id("promptexample")).click();

    Alert alert = driver.switchTo().alert();
    String inputText = "Selenium Test";
    alert.sendKeys(inputText);
    alert.accept();
    assertEquals(inputText, driver.findElement(By.id("promptreturn")).getText());
}
```

此测试导航到测试页面并触发提示警报，**处理提示警报时的关键步骤是将输入文本发送到警报**，我们使用sendKeys()将文本输入到提示窗口的输入字段中。在本例中，我们发送字符串“Selenium Test”作为输入。发送输入后，我们使用accept()方法提交输入并关闭警报。

最后，我们通过检查处理警报后页面上显示的文本来验证是否提交了正确的输入，此步骤可帮助我们确保应用程序正确处理测试脚本提供给提示警报的输入。

## 6. Selenium中处理警报的其他概念

除了我们之前介绍过的方法之外，Selenium中用于管理警报的另外两个重要概念是带有WebDriverWait的alertIsPresent()和处理NoAlertPresentException。 

### 6.1 使用alertIsPresent()和WebDriverWait

alertIsPresent()条件是Selenium的ExpectedConditions类的一部分，它与WebDriver的wait功能结合使用，暂停脚本的执行，直到页面上出现警报后再与其交互。**它在警报出现不是立即或不可预测的情况下很有用**。

让我们看看如何将alertIsPresent()与WebDriverWait一起使用：

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
Alert alert = wait.until(ExpectedConditions.alertIsPresent());
alert.accept();
```

在此实现中，我们不会立即切换到警报，而是使用WebDriverWait和ExpectedConditions.alertIsPresent()来暂停脚本，直到检测到警报。这种方法可确保我们的脚本仅在警报可供交互时才继续运行。

### 6.2 处理NoAlertPresentException

有时，警报可能不会按预期出现，**如果我们尝试与不存在的警报进行交互，我们的测试将失败并出现NoAlertPresentException**。为了处理这种情况，我们可以使用try-catch块来捕获异常，如果警报不存在，则继续执行替代逻辑：

```java
@Test
public void whenNoAlertIsPresent_thenHandleGracefully() {
    driver.get("https://testpages.herokuapp.com/styled/alerts/alert-test.html");

    boolean noAlertExceptionThrown = false;
    try {
        Alert alert = driver.switchTo().alert();
        alert.accept();
    } catch (NoAlertPresentException e) {
        noAlertExceptionThrown = true;
    }

    assertTrue(noAlertExceptionThrown, "NoAlertPresentException should be thrown");
    assertTrue(driver.getTitle().contains("Alerts"), "Page title should contain 'Alerts'");
}
```

## 7. 处理弹出窗口

在本节中，我们将探讨如何在Selenium中处理弹出窗口，并讨论处理弹出窗口时遇到的一些挑战。弹出窗口是许多网站的常见功能，**通常可分为两大类：浏览器级弹出窗口和网站应用程序弹出窗口**。每个类别都需要在Selenium中采用不同的方法，处理它们的策略根据其行为和实现而有所不同。

### 7.1 浏览器级弹出窗口

浏览器级弹出窗口由浏览器本身生成，与网页的HTML内容无关。这些弹出窗口通常包括系统对话框，例如基本身份验证窗口。**浏览器级弹出窗口不是DOM的一部分**，因此无法使用Selenium的标准findElement()方法直接与之交互。

浏览器级弹出窗口的常见示例包括：

- **基本身份验证弹出窗口**：要求用户在访问页面之前输入用户名和密码
- **文件上传/下载对话框**：当用户需要上传或下载文件时出现
- **打印对话框**：打印网页或元素时由浏览器触发

在本教程中，我们将重点介绍如何处理基本身份验证弹出窗口。基本身份验证弹出窗口需要凭据(用户名和密码)才能授予对网页的访问权限，我们将使用演示页面[Basic Auth](https://the-internet.herokuapp.com/basic_auth)来触发和处理弹出窗口：

![](/assets/images/2025/automatedtest/javaseleniumhandlealertspopups04.png)

此类浏览器级弹出窗口无法通过标准Web元素检查技术访问，因此，我们无法使用sendKeys()方法输入凭据。相反，我们需要在浏览器级别处理这些弹出窗口。我们的方法是通过将必要的凭据直接嵌入URL中来完全绕过弹出窗口。

让我们看看如何在Selenium中处理基本的身份验证弹出窗口：

```java
@Test
public void whenBasicAuthPopupAppears_thenBypassWithCredentials() {
    String username = "admin";
    String password = "admin";
    String url = "https://" + username + ":" + password + "@the-internet.herokuapp.com/basic_auth";

    driver.get(url);

    String bodyText = driver.findElement(By.tagName("body")).getText();
    assertTrue(bodyText.contains("Congratulations! You must have the proper credentials."));
}
```

在此示例中，我们通过将用户名和密码嵌入URL来绕过基本身份验证弹出窗口，此技术适用于基本HTTP身份验证弹出窗口。然后，我们导航到指定的URL，浏览器将请求发送到服务器，并在URL中嵌入凭据。服务器识别这些凭据并验证请求，而不会触发浏览器级弹出窗口。

### 7.2 Web应用程序弹出窗口

Web应用程序弹出窗口是直接嵌入网页HTML中的元素，是应用程序前端的一部分。这些弹出窗口使用JavaScript或CSS创建，可以包含模态对话框、横幅或自定义警报等元素。**与浏览器级弹出窗口不同，Web应用程序弹出窗口可以使用标准Selenium命令进行交互，因为它们存在于DOM中**。

Web应用程序弹出窗口的一些常见示例包括：

- **模态对话框**：覆盖并阻止用户与页面其余部分的交互，直到关闭
- **JavaScript弹出窗口**：由用户操作触发，例如确认删除或提交表单
- **自定义警报和提示**：用于通知用户某项操作的通知或消息

在本教程中，我们将重点介绍如何处理许多Web应用程序的模态对话框。

模态对话框显示重要信息或提示用户输入，而无需离开页面。在本节中，我们将重点介绍如何使用Selenium与模态对话框进行交互-特别是如何关闭它们：

![](/assets/images/2025/automatedtest/javaseleniumhandlealertspopups05.png)

通常，我们会检查模态框的HTML结构以找到关闭按钮或其他交互元素，一旦找到，我们就可以使用Selenium的click()方法关闭模态框。

下面是如何使用标准Selenium click()方法处理模态对话框的示例：

```java
@Test
public void whenModalDialogAppears_thenCloseIt() {
    driver.get("https://the-internet.herokuapp.com/entry_ad");

    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10), Duration.ofMillis(500));
    WebElement modal = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("modal")));
    WebElement closeElement = driver.findElement(By.xpath("//div[@class='modal-footer']/p"));

    closeElement.click();

    WebDriverWait modalWait = new WebDriverWait(driver, Duration.ofSeconds(10));
    boolean modalIsClosed = modalWait.until(ExpectedConditions.invisibilityOf(modal));

    assertTrue(modalIsClosed, "The modal should be closed after clicking the close button");
}
```

在此测试中，WebDriverWait确保在与模态框交互之前，该模态框完全可见。模态框出现后，我们找到关闭按钮(在本例中，它是模态框页脚内的<p\>元素)，并调用click()方法将其关闭。

点击后，我们使用ExpectedConditions.invisibilityOf验证模态框是否不再可见。我们的测试通过了，表明模态框已被找到并成功关闭。

## 8. 总结

在本文中，我们学习了如何使用switchTo().alert()来发出JavaScript警报、使用accept()、dismiss()和sendKeys()方法、如何利用WebDriverWait和alertIsPresent()来实现更好的同步以及如何绕过浏览器级别的身份验证弹出窗口。

关键点是，我们需要记住，具体方法可能因应用程序的实现而异，因此我们需要相应地调整策略。