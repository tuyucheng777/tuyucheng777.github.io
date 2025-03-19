---
layout: post
title:  如何使用GitHub Actions运行Selenium回归测试
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 概述

回归测试通过提供关键反馈来确保构建已准备好发布，随着代码库随着新要求、错误修复和增强而发展，频繁运行这些测试对于保持软件质量至关重要。

**在CI/CD管道中运行回归测试非常重要，因为当我们将代码提交到远程仓库时它们会自动执行，并提供快速反馈**。

在本教程中，我们将学习如何使用GitHub Actions运行Selenium回归测试。

## 2. 什么是GitHub Actions？

**GitHub Actions自动化了构建、测试和部署工作流程，构建并测试对远程仓库的每个拉取请求，从而实现更快的反馈**。

这些反馈有助于我们在投入生产之前解决问题。GitHub Actions工作流程可以在Windows、MacOS和Linux虚拟机或数据中心或云基础架构中的自托管运行器上运行。

我们还可以利用GitHub Actions与各种工具和框架来运行回归测试。例如，GitHub Actions可以帮助[使用Selenium执行自动化浏览器测试](https://www.baeldung.com/selenium-automated-browser-testing)，从而提高回归测试的效率。

## 3. 使用GitHub Actions运行Selenium回归测试

在我们讨论在GitHub Actions上运行Selenium回归测试之前，让我们讨论一下项目设置、配置、测试场景和实施。

### 3.1 设置和配置

我们在Maven项目的pom.xml文件中添加[Selenium WebDriver依赖](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)和[TestNG依赖](https://mvnrepository.com/artifact/org.testng/testng)：

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.27.0</version>
</dependency>
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.10.2</version>
    <scope>test</scope>
</dependency>
```

我们还需要添加maven-surefire-plugin，因为当我们使用GitHub Actions工作流程运行Maven命令mvn clean install时，它可以帮助我们执行testng.xml文件：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.2.5</version>
    <executions>
        <execution>
            <goals>
                <goal>test</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <useSystemClassLoader>false</useSystemClassLoader>
        <properties>
            <property>
                <name>usedefaultlisteners</name>
                <value>false</value>
            </property>
        </properties>
        <suiteXmlFiles>
            <suiteXmlFile>testng.xml</suiteXmlFile>
        </suiteXmlFiles>
        <argLine>${argLine}</argLine>
    </configuration>
</plugin>
```

**我们将使用LambdaTest等平台在云端使用GitHub Actions运行Selenium回归测试**。

LambdaTest是一个由人工智能驱动的测试执行平台，允许我们能够在3000多个真实浏览器、浏览器版本和操作系统上执行[自动化测试](https://www.lambdatest.com/automation-testing?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpjan25)。

让我们创建一个BaseTest.java来添加所有Selenium和LambdaTest配置详细信息：

```java
public class BaseTest {
    protected RemoteWebDriver driver;

    // Other code and methods...
}
```

我们将在BaseTest类中添加setup()方法，该方法在任何测试运行之前配置Selenium和LambdaTest：

```java
@BeforeTest
public void setup() {
    String userName = System.getenv("LT_USERNAME") == null
            ? "LT_USERNAME"
            : System.getenv("LT_USERNAME");
    String accessKey = System.getenv("LT_ACCESS_KEY") == null
            ? "LT_ACCESS_KEY"
            : System.getenv("LT_ACCESS_KEY");
    String gridUrl = "@hub.lambdatest.com/wd/hub";
    try {
        this.driver = new RemoteWebDriver(
                new URL("http://" + userName + ":" + accessKey + gridUrl),
                getChromeOptions()
        );
    } catch (MalformedURLException e) {
        LOG.error("Could not start the remote session on LambdaTest cloud grid");
    }
    this.driver.manage()
            .timeouts()
            .implicitlyWait(Duration.ofSeconds(5));
}
```

与平台、浏览器版本等相关的LambdaTest功能将从同一BaseTest类中创建的getChromeOptions()方法中获取。要生成自动化功能，我们可以使用LambdaTest[自动化功能生成器](https://www.lambdatest.com/capabilities-generator/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpjan25)：

```java
public ChromeOptions getChromeOptions() {
    var browserOptions = new ChromeOptions();
    browserOptions.setPlatformName("Windows 10");
    browserOptions.setBrowserVersion("latest");

    HashMap<String, Object> ltOptions = new HashMap<>();
    ltOptions.put("project", "LambdaTest e-commerce website automation");
    ltOptions.put("build", "LambdaTest e-commerceV1.0.0");
    ltOptions.put("name", "Homepage search product test");
    ltOptions.put("w3c", true);
    ltOptions.put("plugin", "java-testNG");

    browserOptions.setCapability("LT:Options", ltOptions);

    return browserOptions;
}
```

所有测试执行完成后，tearDown()方法会更新LambdaTest云网格上的测试状态并关闭驱动程序会话：

```java
@AfterTest
public void tearDown() {
    this.driver.executeScript("lambda-status=" + this.status);
    this.driver.quit();
}
```

我们将在最新Chrome浏览器上的LambdaTest云网格上使用Selenium和GitHub Actions自动执行以下测试场景。

### 3.2 测试场景

让我们使用以下测试场景来演示使用GitHub Actions进行Selenium回归测试：

1. 导航到[LambdaTest电子商务](https://ecommerce-playground.lambdatest.io/?utm_source=baeldung&utm_medium=bdblog&utm_campaign=gpjan25)网站
2. 在主页上搜索产品
3. 检查产品搜索页面是否返回正确的产品详细信息

### 3.3 测试实施

让我们创建一个LambdaTestEcommerceTests类，通过编写自动化测试脚本来实现测试场景。该类扩展了BaseTest类以继承其字段和方法，并在测试类中重用它：

```java
public class LambdaTestEcommerceTests extends BaseTest {
    @Test
    public void whenUserSearchesForAProduct_thenSearchResultsShouldBeDisplayed() {
        String productName = "iPhone";
        driver.get("https://ecommerce-playground.lambdatest.io/");
        HomePage homePage = new HomePage(driver);
        SearchResultPage searchResultPage = homePage.searchProduct(productName);
        searchResultPage.verifySearchResultPageHeader(productName);
        this.status = "passed";
    }
}
```

测试中使用了两个页面对象类，第一个是HomePage类，它包含与Web应用程序主页相关的所有页面对象和操作：

```java
public class HomePage {
    private RemoteWebDriver driver;

    public HomePage(RemoteWebDriver driver) {
        this.driver = driver;
    }

    public SearchResultPage searchProduct(String productName) {
        WebElement searchBox = driver.findElement(By.name("search"));
        searchBox.sendKeys(productName);
        WebElement searchBtn = driver.findElement(By.cssSelector("button.type-text"));
        searchBtn.click();
        return new SearchResultPage(driver);
    }
}
```

searchProduct()方法接收productName作为方法参数，并搜索具有提供的名称的产品。成功搜索产品后，将显示搜索结果页面；因此，此方法返回SearchResultPage类的新实例：

```java
public class SearchResultPage {
    private RemoteWebDriver driver;

    public SearchResultPage(RemoteWebDriver driver) {
        this.driver = driver;
    }

    public void verifySearchResultPageHeader(String productName) {
        String pageHeader = driver.findElement(By.tagName("h1"))
                .getText();
        assertEquals(pageHeader, "Search - " + productName);
    }
}
```

verifySearchResultPageHeader()方法检查搜索页面的标题是否包含所搜索产品的文本。

### 3.4 测试执行

让我们创建一个testng.xml文件，用于在LambdaTest云平台上执行Selenium回归测试：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Selenium GitHub Actions demo">
    <test name="Searching a product from the HomePage">
        <classes>
            <class name="cn.tuyucheng.taketoday.LambdaTestEcommerceTests">
                <methods>
                    <include name="whenUserSearchesForAProduct_thenSearchResultsShouldBeDisplayed"/>
                </methods>
            </class>
        </classes>
    </test>
</suite>
```

### 3.5 设置GitHub Actions

以下是设置和配置GitHub Actions的一些先决条件：

- GitHub帐户
- GitHub仓库

一旦我们拥有了GitHub帐户和创建了仓库，我们就可以转到“Actions”选项卡并配置Java with Maven Action工作流：

![](/assets/images/2025/automatedtest/seleniumregressiontestsgithubactions01.png)

### 3.6 创建GitHub Actions工作流

GitHub会自动为我们生成工作流文件，我们可以选择继续使用GitHub提供的相同工作流，也可以添加自己的内容。

由于我们需要在LambdaTest云平台上运行Selenium回归测试，因此将根据要求更新工作流程的内容：

```yaml
name: Java CI with Maven

on:
    push:
        branches:
            - "main"
            - "test-*"
    pull_request:
        branches:
            - "main"
            - "test-*"

jobs:
    build:
        runs-on: ubuntu-latest
        env:
            LT_USERNAME: ${{ secrets.LT_USERNAME }}
            LT_ACCESS_KEY: ${{ secrets.LT_ACCESS_KEY }}

        steps:
            - uses: actions/checkout@v4
            - name: Set up JDK 17
              uses: actions/setup-java@v4
              with:
                  java-version: '17'
                  distribution: 'temurin'
                  cache: maven
            - name: Build and Test with Maven
              run: mvn clean install
```

我们需要添加LambdaTest用户名和访问Key作为在LambdaTest云网格上运行测试的机密。此外，我们将更新Maven命令以mvn clean install，因为它会一次性下载所有依赖、构建项目并运行测试。

**可以通过导航到GitHub Settings > Secrets and Variables > Actions将机密值添加到仓库**：

![](/assets/images/2025/automatedtest/seleniumregressiontestsgithubactions02.png)

在“Actions Secrets and Variables”页面上，我们单击“New Repository Secrets”并添加秘密变量及其值。

### 3.7 触发GitHub Actions工作流

**我们可以使用工作流文件中的on关键字来触发单个事件或多个事件的GitHub Actions工作流程**。

让我们按照以下步骤在将代码推送到任何分支或提出拉取请求时触发工作流：

1. 我们将工作流文件添加到GitHub仓库，将分支命名为maven_github_actions，以使on push branches/on pull_request branches名称与maven_*匹配。

2. 我们在maven_github_actions分支中进行代码更改，然后，我们将代码推送到分支上，并向主分支发出拉取请求。成功推送代码后，工作流会自动触发，我们可以在GitHub仓库的“Actions”选项卡上检查其状态：

![](/assets/images/2025/automatedtest/seleniumregressiontestsgithubactions03.png)

单击工作流运行，我们可以查看有关工作流作业的详细信息。

工作流运行完成后，我们可以在LambdaTest Web自动化仪表板上查看Selenium回归测试结果：

![](/assets/images/2025/automatedtest/seleniumregressiontestsgithubactions04.png)

构建任务实际上在用户选择的配置机器上运行，此机器的名称在jobs块内的工作流中使用runs-on关键字设置。

在我们的例子中，我们使用ubuntu-latest机器运行GitHub Actions工作流。

## 4. 总结

GitHub Actions有助于设置自动化管道并运行各种工作流，包括回归测试。它会在多个事件上触发工作流，例如将代码推送到仓库或通过拉取请求将代码合并到主分支。这些工作流程运行可快速提供有关构建的反馈，使我们能够在将应用程序发布到生产之前采取纠正措施。