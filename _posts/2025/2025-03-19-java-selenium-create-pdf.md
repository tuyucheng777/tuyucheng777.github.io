---
layout: post
title:  如何使用Selenium生成PDF
category: automatedtest
copyright: automatedtest
excerpt: Selenium
---

## 1. 概述

在本教程中，我们将探讨如何使用Selenium 4的ChromeDriver类中提供的print()方法从网页生成PDF文件。print()方法提供了一种将网页内容直接捕获到PDF文件的简单方法。

我们将介绍如何使用Chrome和Firefox浏览器生成PDF，并演示如何使用PrintOptions类自定义PDF输出。这包括调整方向、页面大小、比例和边距等参数，以使PDF符合特定要求。

此外，我们的实际示例将涉及使用Java和[JUnit测试](https://www.baeldung.com/junit-5)打印网页内容。

## 2. 设置和配置

配置环境需要两个依赖：[Selenium Java](https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java)和[WebDriverManager](https://mvnrepository.com/artifact/io.github.bonigarcia/webdrivermanager/)。**Selenium Java提供了必要的自动化框架，以便以编程方式与Web浏览器交互并对其进行控制**。

**WebDriverManager通过自动处理浏览器驱动程序的下载和配置来简化浏览器驱动程序的管理**，此设置对于顺利执行我们的自动化测试和Web交互至关重要。

让我们添加Maven依赖项：

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

## 3. 使用Chrome和Selenium生成PDF

在本节中，**我们将展示如何使用Chrome中的Selenium WebDriver将网页转换为PDF文件**。我们的目标是捕获[Baeldung Java Weekly](https://baeldung.com/library/java-web-weekly)的内容并将其保存为PDF文档。

让我们编写一个JUnit测试来在项目的根目录中创建一个名为Baeldung_Weekly.pdf的PDF文件：

```java
@Test
public void whenNavigatingToBaeldung_thenPDFIsGenerated() throws IOException {
    ChromeDriver driver = new ChromeDriver();
    driver.get("https://www.baeldung.com/library/java-web-weekly");
    Pdf pdf = driver.print(new PrintOptions());
    byte[] pdfContent = Base64.getDecoder().decode(pdf.getContent());
    Files.write(Paths.get("./Baeldung_Weekly.pdf"), pdfContent);
    assertTrue(Files.exists(Paths.get("./Baeldung_Weekly.pdf")), "PDF file should be created");
    driver.quit();
}
```

运行测试后，ChromeDriver打开网页并使用print()方法创建PDF。系统解码Base64编码的PDF并将其保存为Baeldung_Weekly.pdf，然后检查文件是否存在以确认成功生成PDF。最后，使用driver.quit()方法关闭浏览器。**关闭浏览器始终很重要，以确保没有资源被挂起**。

此外，**即使Chrome在无头模式(即浏览器无需GUI即可运行的模式)下运行，print()方法也能无缝运行**。

我们可以**使用options.addArguments(“–headless”)方法启用无头模式**，以在没有GUI的情况下在后台运行浏览器：

```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless");

ChromeDriver driver = new ChromeDriver(options);
```

## 4. 使用Firefox和Selenium生成PDF

**Firefox浏览器也支持print()方法**，让我们探索如何使用Firefox和Selenium WebDriver自动生成PDF。与以前一样，目标是捕获URL中提供的页面并使用Firefox将其保存为PDF文档。

让我们编写一个JUnit测试来在项目当前目录中生成一个名为Firefox_Weekly.pdf的PDF文件：

```java
@Test
public void whenNavigatingToBaeldungWithFirefox_thenPDFIsGenerated() throws IOException {
    FirefoxDriver driver = new FirefoxDriver(new FirefoxOptions());
    driver.get("https://www.baeldung.com/library/java-web-weekly");
    Pdf pdf = driver.print(new PrintOptions());
    byte[] pdfContent = Base64.getDecoder().decode(pdf.getContent());
    Files.write(Paths.get("./Firefox_Weekly.pdf"), pdfContent);
    assertTrue(Files.exists(Paths.get("./Firefox_Weekly.pdf")), "PDF file should be created");
    driver.quit();
}
```

测试执行后，启动FirefoxDriver，打开并导航到指定的网址，然后print()方法从网页生成PDF。系统将生成的PDF以Base64格式编码，解码为二进制格式，并将其存储为Firefox_Weekly.pdf。

**测试通过检查PDF是否在文件系统中存在来确认PDF的创建，此验证确保我们使用Firefox浏览器从网页成功生成PDF文件**。

## 5. 使用PrintOptions自定义PDF输出

使用print()方法时，我们可以决定输出PDF文档的外观。在本节中，我们将了解如何通过使用[PrintOptions](https://www.selenium.dev/selenium/docs/api/java/org/openqa/selenium/print/PrintOptions.html)类自定义PDF输出来增强Selenium WebDriver中print()方法的功能。

**PrintOptions类是Selenium API的一部分，允许在将网页呈现为PDF时对其进行详细调整**。让我们了解一下PrintOptions提供的众多选项中的几个-方向、页面大小、比例和边距：

```java
PrintOptions options = new PrintOptions();
        
options.setOrientation(PrintOptions.Orientation.LANDSCAPE);
options.setScale(1.5);
options.setPageSize(new PageSize(100, 100));
options.setPageMargin(new PageMargin(2, 2, 2, 2));

Pdf pdf = driver.print(options);
```

在代码片段中，**PrintOptions类自定义了print()方法生成的PDF输出**。setOrientation()设置页面方向，setScale()调整内容大小，setPageSize()指定自定义页面大小，setPageMargin()定义每边的边距。

## 6. 总结

在本文中，我们介绍了如何使用Selenium 4的print()方法从网页生成PDF文件。我们通过在Chrome和Firefox上尝试print()演示了在不同平台上实现相同功能的能力。

此外，我们还探索了可通过PrintOptions类提供的自定义选项，该选项允许自定义输出PDF文档以满足特定要求。