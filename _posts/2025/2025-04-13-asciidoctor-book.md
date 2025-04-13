---
layout: post
title:  使用Asciidoctor生成一本书
category: libraries
copyright: libraries
excerpt: OpenPDF
---

## 1. 简介

在这篇简短的文章中，我们将演示如何从AsciiDoc文档生成一本书，以及如何使用各种样式选项自定义书。

如果你不熟悉Java中的AsciiDoc，可以阅读我们的[AsciiDoctor简介](https://www.baeldung.com/asciidoctor)。

## 2. 后端书籍类型

使用AsciiDoctorj生成书籍最简单的方法是使用Maven，就像前面提到的那样，**唯一的区别是你必须指定doctype标签并将其设置为“book”**。

```xml
<backend>pdf</backend>
<doctype>book</doctype>
```

通过定义的文档类型，AsciiDoctorj知道你想要创建一本书，因此它会创建：

- 标题页
- 目录
- 第一页正文内容
- 部分和章节

为了获得提到的部分，Asciidoc文档应该具有定义的标题、章节和其他对于书籍来说正常的部分。

## 3. 定义自定义样式

在写书时，我们很自然地想要使用一些自定义样式，可以使用在简单的YAML文件中定义的AsciiDoc特定格式化语言来实现。

例如，以下代码片段将定义书中每一页的外观。我们希望采用纵向模式，在A4纸张格式上，顶部和底部边距为0.75英寸，侧面边距为1英寸：

```text
page:
    layout: portrait
    margin: [0.75in, 1in, 0.75in, 1in]
    size: A4
```

另外，我们可以为书的页脚和页眉定义自定义样式：

```plaintext
header:
    height: 0.5in
    line_height: 1
    recto_content:
        center: '{document-title}'
    verso_content:
        center: '{document-title}'

footer:
    height: 0.5in
    line_height: 1
    recto_content:
        right: '{chapter-title} | *{page-number}*'
    verso_content:
        left: '*{page-number}* | {chapter-title}
```

更多格式化选项可以在AsciiDoctorj的[Github页面](https://github.com/asciidoctor/asciidoctor-pdf/blob/master/docs/theming-guide.adoc)上找到。

为了在书籍生成过程中包含自定义主题，我们必须定义样式文件所在的路径，该位置在pom.xml的属性部分中指定：

```xml
<pdf-stylesdir>${project.basedir}/src/themes</pdf-stylesdir>
<pdf-style>custom</pdf-style>
```

第一行定义了我们样式定义的路径，第二行指定了不带扩展名的文件名。

经过这些更改，我们的pom.xml如下所示：

```xml
<configuration>
    <sourceDirectory>src/docs/asciidoc</sourceDirectory>
    <outputDirectory>target/docs/asciidoc</outputDirectory>
    <attributes>
        <pdf-stylesdir>${project.basedir}/src/themes</pdf-stylesdir>
        <pdf-style>custom</pdf-style>
    </attributes>
    <backend>pdf</backend>
    <doctype>book</doctype>
</configuration>
```

## 4. 生成书籍

要生成你的书，你只需在项目目录中运行Maven，生成的书可以在target/docs/asciidoctor/目录中找到。

## 5. 总结

在本教程中，我们向你展示了如何使用Maven生成具有简单风格装饰的书籍。