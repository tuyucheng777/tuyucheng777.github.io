---
layout: post
title:  使用Selenium进行自动化可访问性测试
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 概述

**可访问性测试对于确保软件应用程序可供所有人(包括残障人士)使用至关重要，手动执行可访问性测试可能非常耗时且容易出错**。因此，使用Selenium自动执行可访问性测试可以简化流程，从而更容易及早发现和修复可访问性问题。

在本教程中，我们将探讨如何使用Selenium自动化可访问性测试。

## 2. 什么是自动化可访问性测试？

**自动化可访问性测试是使用自动化工具和脚本来评估软件应用程序满足可访问性标准(例如WCAG(Web内容可访问性指南))的程度的过程**。

它有助于识别可能阻碍残疾人有效使用软件的无障碍障碍。

通过自动化这些测试，团队可以快速检测与屏幕阅读器兼容性、键盘导航、颜色对比度和其他可访问性方面相关的问题，确保应用程序更具包容性并符合法律要求。

## 3. 为什么使用Selenium实现可访问性自动化？

[Selenium](https://www.baeldung.com/java-selenium-with-junit-and-testng) WebDriver是一个流行的开源测试自动化框架，它有助于自动化流行的浏览器(如Chrome、Firefox、Edge和Safari)，并提供更大的灵活性和与不同测试框架的兼容性。

随着Selenium 4的发布，它也完全符合W3C标准。**在许多国家/地区，法律要求和行业标准(例如Web内容可访问性指南)规定Web应用程序必须具有可访问性**。通过使用Selenium进行自动化可访问性测试，组织可以有效地评估其是否符合这些法规和标准。

## 4. 如何开始Selenium可访问性测试？

在本节中，我们将学习如何使用Selenium在LambdaTest等平台提供的云网格上执行可访问性测试。

LambdaTest是一个人工智能驱动的测试执行平台，允许开发人员和测试人员在3000多个真实环境中使用Selenium和Cypress等框架执行[可访问性自动化](https://www.lambdatest.com/accessibility-automation?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpseptember24)。

**要在LambdaTest上执行Selenium可访问性测试，我们需要启用可访问性自动化计划**。

### 4.1 测试场景

以下可访问性测试场景将在[LambdaTest电子商务](https://ecommerce-playground.lambdatest.io/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpseptember24)网站上运行：

1. 导航到LambdaTest电子商务网站
2. 将鼠标悬停在“My Account”下拉菜单上，然后单击“Login”
3. 输入有效的用户名和密码
4. 执行断言以检查登录成功后是否显示注销链接

### 4.2 设置项目

让我们在pom.xml文件中添加以下Selenium依赖项：

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.23.1</version>
</dependency>
```

对于最新版本，我们可以查看[Maven中央仓库](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)。

现在，我们需要创建一个新的ECommercePlayGroundAccessibilityTests类：

```java
public class ECommercePlayGroundAccessibilityTests {

    private RemoteWebDriver driver;
    // ...
}
```

让我们定义一个setup()方法来帮助实例化RemoteWebDriver：

```java
@BeforeTest
public void setup() {
    final String userName = System.getenv("LT_USERNAME") == null
            ? "LT_USERNAME" : System.getenv("LT_USERNAME");
    final String accessKey = System.getenv("LT_ACCESS_KEY") == null
            ? "LT_ACCESS_KEY" : System.getenv("LT_ACCESS_KEY");
    final String gridUrl = "@hub.lambdatest.com/wd/hub";
    try {
        this.driver = new RemoteWebDriver(new URL(
                "http://" + userName + ":" + accessKey + gridUrl),
                getChromeOptions());
    } catch (final MalformedURLException e) {
        System.out.println(
                "Could not start the remote session on LambdaTest cloud grid");
    }
    this.driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(30));
}
```

此方法需要LambdaTest用户名和访问密钥才能在LambdaTest云网格上运行测试，**我们可以从LambdaTest Account Settings > Password & Security中找到这些凭据**。

在实例化RemoteWebDriver时，我们需要传递BrowserVersion、PlatformName等功能来运行测试：

```java
public ChromeOptions getChromeOptions() {
    final var browserOptions = new ChromeOptions();
    browserOptions.setPlatformName("Windows 10");
    browserOptions.setBrowserVersion("127.0");
    final HashMap<String, Object> ltOptions = new HashMap<String, Object>();
    ltOptions.put("project",
            "Automated Accessibility Testing With Selenium");
    ltOptions.put("build",
            "LambdaTest Selenium Playground");
    ltOptions.put("name",
            "Accessibility test");
    ltOptions.put("w3c", true);
    ltOptions.put("plugin",
            "java-testNG");
    ltOptions.put("accessibility", true);
    ltOptions.put("accessibility.wcagVersion",
            "wcag21");
    ltOptions.put("accessibility.bestPractice", false);
    ltOptions.put("accessibility.needsReview", true);

    browserOptions.setCapability(
            "LT:Options", ltOptions);

    return browserOptions;
}
```

**对于LambdaTest上的可访问性测试，我们需要添加诸如accessibility、accessibility.wcagVersion、accessibility.bestPractice和accessibility.needsReview等功能**。

为了生成这些功能，我们可以参考LambdaTest[自动化功能生成器](https://www.lambdatest.com/capabilities-generator/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpseptember24)。

### 4.3 测试实施

我们将使用两种测试方法testNavigationToLoginPage()和testLoginFunction()来实现测试场景。

下面是testNavigationToLoginPage()方法的代码片段：

```java
@Test(priority = 1)
public void testNavigationToLoginPage() {
    driver.get("https://ecommerce-playground.lambdatest.io/");
    WebElement myAccountLink = driver.findElement(By.cssSelector(
            "#widget-navbar-217834 > ul > li:nth-child(6) > a"));
    Actions actions = new Actions(driver);
    actions.moveToElement(myAccountLink).build().perform();
    WebElement loginLink = driver.findElement(By.linkText("Login"));
    loginLink.click();

    String pageHeaderText = driver.findElement(By.cssSelector(
            "#content > div > div:nth-child(2) > div h2")).getText();
    assertEquals(pageHeaderText, "Returning Customer");
}
```

此测试实现了测试场景的前两个步骤，即导航到LambdaTest电子商务网站并将鼠标悬停在“My Account”链接上。此测试方法首先执行，因为优先级设置为“1”。

Selenium WebDriver的Actions类的moveToElement()方法悬停在“My Account”链接上。

菜单打开后，使用linkText找到Login链接，并对其执行单击操作。最后，执行断言以检查登录页面是否成功加载。

现在让我们看一下testLoginFunction()方法的代码片段：

```java
@Test(priority = 2)
public void testLoginFunction() {
    WebElement emailAddressField = driver.findElement(By.id(
            "input-email"));
    emailAddressField.sendKeys("davidJacob@demo.com");
    WebElement passwordField = driver.findElement(By.id(
            "input-password"));
    passwordField.sendKeys("Password123");
    WebElement loginBtn = driver.findElement(By.cssSelector(
            "input.btn-primary"));
    loginBtn.click();

    WebElement myAccountLink = driver.findElement(By.cssSelector(
            "#widget-navbar-217834 > ul > li:nth-child(6) > a"));
    Actions actions = new Actions(driver);
    actions.moveToElement(myAccountLink).build().perform();
    WebElement logoutLink = driver.findElement(By.linkText("Logout"));
    assertTrue(logoutLink.isDisplayed());
}
```

此测试涵盖使用有效凭据登录和验证Logout链接的最后步骤，在电子邮件地址和密码字段中输入凭据后，单击登录按钮。使用moveToElement()方法将鼠标悬停在“My Account”链接上，然后断言检查Logout链接是否可见。

### 4.4 测试执行

LambdaTest Web自动化仪表板的以下屏幕截图显示了这两项可访问性测试：

![](/assets/images/2025/automatedtest/seleniumaccessibilitytesting01.png)

## 5. 总结

可访问性测试有助于发现与网站或基于Web的应用程序可用性相关的缺陷，它有助于使网站可供所有用户(包括残障人士)使用。

必须遵循WCAG制定的准则，以使网站或Web应用程序可访问。此外，还必须对网站进行可访问性测试，以确保网站和Web应用程序可供所有人使用。