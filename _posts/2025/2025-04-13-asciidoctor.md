---
layout: post
title:  Java中的Asciidoctor简介
category: libraries
copyright: libraries
excerpt: Asciidoctor
---

## 1. 简介

在本文中，我们将简要介绍如何在Java中使用Asciidoctor，我们将演示如何从AsciiDoc文档生成HTML5或PDF。

## 2. 什么是AsciiDoc？

AsciiDoc是一种文本文档格式，**它可用于编写文档、书籍、网页、手册页等**。

由于其高度可配置，AsciiDoc文档可以转换为许多其他格式，如HTML、PDF、手册页、EPUB等。

AsciiDoc语法非常简单，因此它变得非常流行，并得到了各种浏览器插件、编程语言插件和其他工具的广泛支持。

要了解有关该工具的更多信息，我们建议阅读[官方文档](https://asciidoc.org/)，你可以在其中找到许多有用的资源来学习正确的语法和将AsciiDoc文档导出为其他格式的方法。

## 3. 什么是Asciidoctor？

**[Asciidoctor](http://asciidoctor.org/)是一个文本处理器**，用于将AsciiDoc文档转换为HTML、PDF和其他格式。它使用Ruby编写，并打包为RubyGem。

如上所述，AsciiDoc是一种非常流行的编写文档的格式，因此你可以轻松地在许多GNU Linux发行版(如Ubuntu、Debian、Fedora和Arch)中找到Asciidoctor作为标准包。

由于我们想在JVM上使用Asciidoctor，我们将讨论AsciidoctorJ-它是使用Java的Asciidoctor。

## 4. 依赖

要在我们的应用程序中包含AsciidoctorJ包，需要以下pom.xml条目：

```xml
<dependency>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctorj</artifactId>
    <version>2.5.7</version>
</dependency>
<dependency>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctorj-pdf</artifactId>
    <version>2.3.4</version>
</dependency>
```

最新版本的库可以在[这里](https://mvnrepository.com/artifact/org.asciidoctor/asciidoctorj)和[这里](https://mvnrepository.com/artifact/org.asciidoctor/asciidoctorj-pdf)找到。

## 5. AsciidoctorJ API

AsciidoctorJ的入口点是Asciidoctor Java接口。

这些方法是：

- convert：从String或Stream解析AsciiDoc文档并将其转换为提供的格式类型
- convertFile：从提供的File对象解析AsciiDoc文档并将其转换为提供的格式类型
- convertFiles：与前一个相同，但方法接收多个File对象
- convertDirectory：解析提供的文件夹中的所有AsciiDoc文档并将其转换为提供的格式类型

### 5.1 代码中的API使用

要创建Asciidoctor实例，你需要从提供的工厂方法中检索实例：

```java
import static org.asciidoctor.Asciidoctor.Factory.create;
import org.asciidoctor.Asciidoctor;
..
//some code
..
Asciidoctor asciidoctor = create();
```

通过检索到的实例，我们可以非常轻松地转换AsciiDoc文档：

```java
String output = asciidoctor
    .convert("Hello _Tuyucheng_!", new HashMap<String, Object>());
```

如果我们想从文件系统转换文本文档，我们将使用convertFile方法：

```java
String output = asciidoctor
    .convertFile(new File("tuyucheng.adoc"), new HashMap<String, Object>());
```

为了转换多个文件，convertFiles方法接收List对象作为第一个参数，并返回String对象数组。

更有趣的是如何使用AsciidoctorJ转换整个目录。

如上所述，要转换整个目录，我们应该调用convertDirectory方法。该方法会扫描提供的路径，并搜索所有带有AsciiDoc扩展名(.adoc、.ad、.asciidoc、.asc)的文件并进行转换。要扫描所有文件，应向该方法传入DirectoryWalker的实例。

目前，Asciidoctor提供了上述接口的两个内置实现：

- **AsciiDocDirectoryWalker**：转换指定文件夹及其子文件夹中的所有文件，忽略所有以“_”开头的文件。
- **GlobDirectoryWalker**：按照glob表达式转换给定文件夹的所有文件

```java
String[] result = asciidoctor.convertDirectory(
    new AsciiDocDirectoryWalker("src/asciidoc"),
    new HashMap<String, Object>());
```

另外，**我们可以使用提供的java.io.Reader和java.io.Writer接口调用convert方法**，Reader接口作为源，Writer接口用于写入转换后的数据：

```java
FileReader reader = new FileReader(new File("sample.adoc"));
StringWriter writer = new StringWriter();
 
asciidoctor.convert(reader, writer, options().asMap());
 
StringBuffer htmlBuffer = writer.getBuffer();
```

### 5.2 PDF生成

**要从Asciidoc文档生成PDF文件，我们需要在选项中指定生成文件的类型**。如果仔细查看前面的示例，你会注意到任何convert方法的第二个参数都是一个Map-它代表选项对象。

我们将in_place选项设置为true，以便我们的文件自动生成并保存到文件系统：

```java
Map<String, Object> options = options()
    .inPlace(true)
    .backend("pdf")
    .asMap();

String outfile = asciidoctor.convertFile(new File("tuyucheng.adoc"), options);
```

## 6. Maven插件

在上一节中，我们展示了如何使用Java实现直接生成PDF文件，本节将展示如何在Maven构建过程中生成PDF文件；Gradle和Ant也提供类似的插件。

要在构建期间启用PDF生成，你需要将此依赖添加到pom.xml中：

```xml
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>2.2.2</version>
    <dependencies>
        <dependency>
            <groupId>org.asciidoctor</groupId>
            <artifactId>asciidoctorj-pdf</artifactId>
            <version>2.3.4</version>
        </dependency>
    </dependencies>
</plugin>
```

最新版本的Maven插件依赖可以在[这里](https://mvnrepository.com/search?q=asciidoctor-maven-plugin)找到。

### 6.1 使用

要在构建中使用该插件，你必须在pom.xml中定义它：

```xml
<plugin>
    <executions>
        <execution>
            <id>output-html</id> 
            <phase>generate-resources</phase> 
            <goals>
                <goal>process-asciidoc</goal> 
            </goals>
        </execution>
    </executions>
</plugin>
```

由于插件没有在任何特定阶段运行，因此你必须设置要启动它的阶段。

与Asciidoctorj插件一样，我们也可以在这里使用各种选项来生成PDF。

让我们快速浏览一下基本选项，同时你可以在[文档](https://github.com/asciidoctor/asciidoctor-maven-plugin)中找到其他选项：

- **sourceDirectory**：Asciidoc文档所在目录的位置
- **outputDirectory**：你要存储生成的PDF文件的目录位置
- **backend**：Asciidoctor输出的类型，对于PDF生成，请设置为pdf

这是如何在插件中定义基本选项的示例：

```xml
<plugin>
    <configuration>
        <sourceDirectory>src/main/doc</sourceDirectory>
        <outputDirectory>target/docs</outputDirectory>
        <backend>pdf</backend>
    </configuration>
</plugin>
```

运行构建后，可以在指定的输出目录中找到PDF文件。

## 7. 总结

AsciiDoc非常易于使用和理解，它是管理文档和其他文档的非常强大的工具。

在本文中，我们演示了一种从AsciiDoc文档生成HTML和PDF文件的简单方法。