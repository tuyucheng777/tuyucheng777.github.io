---
layout: post
title:  使用Selenium进行自动化浏览器测试
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 概述

如今，大多数企业都依赖网站和Web应用程序，几乎每个组织都在线运营。由于网站和Web应用程序与最终用户和客户联系在一起，因此必须在所有流行的浏览器、浏览器版本和操作系统上完美运行。

虽然手动测试可以完成工作，但自动化测试是测试速度和效率的更好选择。Selenium等自动化测试工具允许企业运行自动浏览器测试，使他们能够更快地提供高质量的网站和Web应用程序。

在本文中，我们将学习如何使用Selenium执行自动浏览器测试。

## 2. Selenium是什么

Selenium是一个开源工具，可帮助自动化Web浏览器测试。它提供跨浏览器测试、支持多种编程语言、平台独立性、庞大社区等优势。

最近推出的Selenium Manager提供了自动化的浏览器和驱动程序管理，使开发人员和测试人员可以无缝运行Selenium测试，而无需担心下载和安装驱动程序和浏览器。

## 3. 为什么使用Selenium进行自动浏览器测试？

以下是使用Selenium进行自动化浏览器测试的主要原因：

- Selenium是一个免费的开源工具，为Web自动化测试提供了经济高效的解决方案。
- 它支持多种流行的编程语言，如Java、Python、JavaScript、C#、Ruby和PHP。
- Selenium使软件团队能够在各种浏览器(例如Chrome、Firefox、Edge和Safari)上测试他们的网站和Web应用程序。
- 它拥有庞大、活跃的社区，提供广泛的支持和资源。

## 4. 如何使用Selenium执行浏览器自动化？

根据使用Selenium编写测试自动化脚本所选择的编程语言，我们需要遵循一些先决条件、设置和执行过程。

### 4.1 先决条件

以下是使用Selenium进行自动浏览器测试的一些先决条件：

- 下载并安装Java JDK
- 安装IntelliJ或Eclipse IDE
- 下载所需的Maven构建工具

我们不会下载浏览器，因为测试将在基于云的测试平台上运行，例如LambdaTest。这是一个由AI驱动的测试执行平台，可让开发人员和测试人员使用Selenium在3000多种真实浏览器、浏览器版本和操作系统上大规模执行[自动化测试](https://www.lambdatest.com/automation-testing?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpoct24)。

### 4.2 设置和配置

让我们在pom.xml文件中添加以下[Selenium WebDriver依赖](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)和[TestNG依赖](https://mvnrepository.com/artifact/org.testng/testng)：

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.25.0</version>
</dependency>
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.10.2</version>
    <scope>test</scope>
</dependency>
```

**由于我们将在LambdaTest平台上运行测试，因此我们需要为平台、浏览器和浏览器版本添加某些功能**。为此，我们使用LambdaTest[自动化功能生成器](https://www.lambdatest.com/capabilities-generator/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpoct24)。

让我们创建一个BaseTest.java类来在其中添加所有配置细节：

```java
public class BaseTest {
    private static final ThreadLocal<RemoteWebDriver> DRIVER = new ThreadLocal<>();

    //...
}
```

BaseTest类中的ThreadLocal实例用于线程安全，因为我们将并行运行测试。它还确保每个线程都有其RemoteWebDriver实例，从而促进并行测试的顺利执行。

接下来，getDriver()和setDriver()方法将使用ThreadLocal实例获取和设置RemoteWebDriver的实例。我们将使用getLtOptions()方法设置LambdaTest自动化功能：

```java
public RemoteWebDriver getDriver() {
    return DRIVER.get();
}

private void setDriver(RemoteWebDriver remoteWebDriver) {
    DRIVER.set(remoteWebDriver);
}

private HashMap<String, Object> getLtOptions() {
    HashMap<String, Object> ltOptions = new HashMap<>();
    ltOptions.put("project", "ECommerce playground website");
    ltOptions.put("build", "LambdaTest Ecommerce Website tests");
    ltOptions.put("name", "Automated Browser Testing");
    ltOptions.put("w3c", true);
    ltOptions.put("visual", true);
    ltOptions.put("plugin", "java-testNG");
    return ltOptions;
}
```

**使用getLtOptions()方法，我们将设置项目、构建和测试名称，并在LambdaTest云环境中启用java-testNG插件**。接下来，我们将添加getChromeOptions()方法，该方法将保存Chrome浏览器的功能并返回ChromeOptions。

类似地，getFirefoxOptions()方法接收browserVersion和platform作为参数，设置其各自的功能，并返回FirefoxOptions。

在BaseTest类中，我们将使用setup()方法设置SeleniumWebDriver，以便在Chrome和Firefox上运行自动测试。@BeforeTest注解确保该方法在任何测试之前运行，而@Parameters在运行时从testng.xml文件中传递浏览器、浏览器版本和平台值。这有助于在Chrome和Firefox上并行运行跨浏览器测试：

```java
if (browser.equalsIgnoreCase("chrome")) {
    try {
        setDriver(new RemoteWebDriver(new URL("http://" + userName + ":" + accessKey + gridUrl),
            getChromeOptions(browserVersion, platform)));
    } catch (final MalformedURLException e) {
        throw new Error("Could not start the Chrome browser on the LambdaTest cloud grid");
    }
} else if (browser.equalsIgnoreCase("firefox")) {
    try {
        setDriver(new RemoteWebDriver(new URL("http://" + userName + ":" + accessKey + gridUrl),
            getFirefoxOptions(browserVersion, platform)));
    } catch (final MalformedURLException e) {
        throw new Error("Could not start the Firefox browser on the LambdaTest cloud grid");
    }
}
```

**要连接到LambdaTest云网格，我们需要有LambdaTest用户名、访问密钥和网格URL**。我们可以从Account Settings > Password & Security轻松找到LambdaTest用户名和访问密钥。

之后，我们将编写处理浏览器并启动其各自会话的代码。

### 4.3 测试场景

让我们使用以下测试场景来演示在LambdaTest平台上使用Selenium进行自动浏览器测试。

1. 导航到[LambdaTest电子商务](https://ecommerce-playground.lambdatest.io/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpoct24)网站
2. 找到并点击主页上的“Shop by Category”菜单
3. 找到并点击“MP3 Players”菜单
4. 断言显示的页面显示页眉MP3播放器

### 4.4 测试实施

让我们创建一个AutomatedBrowserTest.java类来编写测试场景的测试自动化脚本，此类扩展了BaseTest类，以实现代码重用和模块化。

然后，我们将在实现测试场景的AutomatedBrowserTest类中创建whenUserNavigatesToMp3PlayersPage_thenPageHeaderShouldBeCorrectlyDisplayed()方法：

```java
@Test
public void whenUserNavigatesToMp3PlayersPage_thenPageHeaderShouldBeCorrectlyDisplayed() {
    getDriver().get("https://ecommerce-playground.lambdatest.io/");

    HomePage homePage = new HomePage(getDriver());
    homePage.openShopByCategoryMenu();
    Mp3PlayersPage mp3PlayersPage = homePage.navigateToMp3PlayersPage();
    assertEquals(mp3PlayersPage.pageHeader(), "MP3 Players");
}
```

**我们使用了[Selenium WebDriver页面对象](https://www.baeldung.com/selenium-webdriver-page-object)模式，这有助于提高代码的可读性和可维护性**。HomePage类包含LambdaTest电子商务网站主页的所有页面对象。

openShopByCategoryMenu()方法使用Selenium的linkText定位器策略定位Shop By Category菜单并执行单击操作。此MP3 Players菜单使用CSS选择器#widget-navbar-217841 \> ul \> li:nth-child(5) \> a定位，然后，navigateToMP3PlayersPage()方法定位MP3 Players页面，执行单击操作，并返回MP3PlayersPage类的新实例。

MP3PlayersPage类包含MP3 Players页面的所有对象：

```java
public class Mp3PlayersPage {
    private WebDriver driver;

    public Mp3PlayersPage(WebDriver driver) {
        this.driver = driver;
    }

    public String pageHeader() {
        return driver.findElement(By.cssSelector(".content-title h1"))
                .getText();
    }
}
```

### 4.5 测试执行

让我们创建一个新的testng.xml文件来并行运行测试，此文件配置TestNG在Windows 10上的Chrome和Firefox上运行自动浏览器测试。

现在，让我们运行测试并访问LambdaTestWeb自动化仪表板来检查测试执行结果。

以下是在Chrome浏览器上执行测试的快照：

![](/assets/images/2025/automatedtest/seleniumautomatedbrowsertesting01.png)

以下是在Firefox浏览器上执行测试的快照：

![](/assets/images/2025/automatedtest/seleniumautomatedbrowsertesting02.png)

我们可以找到测试的所有详细信息，包括浏览器名称、版本、平台和测试执行日志，以及视频记录和屏幕截图。

## 5. 总结

总而言之，自动化浏览器测试在快速测试网站和Web应用程序中起着至关重要的作用。借助Selenium，测试人员可以同时高效地运行多个测试，从而获得更快的反馈。这种方法不仅有助于尽早发现问题，而且还能确保软件保持稳定。