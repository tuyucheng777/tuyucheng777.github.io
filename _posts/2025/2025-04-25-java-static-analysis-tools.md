---
layout: post
title:  Eclipse和IntelliJ IDEA中的Java静态分析工具
category: staticanalysis
copyright: staticanalysis
excerpt: IntelliJ IDEA
---

## 1. 概述

在对[FindBugs的介绍](https://www.baeldung.com/intro-to-findbugs)中，我们研究了FindBugs作为静态分析工具的功能以及如何将其直接集成到Eclipse和IntelliJ IDEA等IDE中。

在本文中，我们将研究一些Java的替代静态分析工具-以及它们如何与Eclipse和IntelliJ IDEA集成。

## 2. PMD

让我们从PMD开始。

这个成熟且相当完善的工具可以分析源代码中可能存在的错误、次优代码和其他不良做法；它还可以查看所分析代码库的更高级指标，例如圈复杂度。

### 2.1 与Eclipse集成

PMD插件可以直接从Eclipse Marketplace安装，你也可以从[此处](https://pmd.sourceforge.io/pmd-5.8.1/usage/integrations.html#Eclipse)手动下载，安装完成后，我们可以直接从IDE运行PMD检查：

![](/assets/images/2025/staticanalysis/javastaticanalysistools01.png)

值得注意的是，我们可以在项目级别或单个类级别运行PMD。

结果如下所示-不同颜色表示不同级别的发现，从“警告”到“阻止”，严重程度依次递增：

![](/assets/images/2025/staticanalysis/javastaticanalysistools02.png)

我们可以通过右键单击每个条目并从上下文菜单中选择“show details”来深入了解其详细信息，Eclipse将显示问题的简要描述以及可能的解决方法：

![](/assets/images/2025/staticanalysis/javastaticanalysistools03.png)

你还可以更改PMD扫描的配置——我们可以在菜单中进行更改，在Window -> Preferences -> PMD下启动配置页面，在这里，我们可以配置扫描参数、规则集、结果显示设置等。

如果我们需要停用项目的一些特定规则-我们可以简单地将它们从扫描中删除：

![](/assets/images/2025/staticanalysis/javastaticanalysistools04.png)

### 2.2 与IntelliJ集成

当然，IntelliJ有一个类似的PMD插件-可以从[JetBrains插件商店](https://plugins.jetbrains.com/plugin/1137-pmdplugin)下载并安装。

我们可以在IDE中直接运行插件-通过右键单击我们需要扫描的源并从上下文菜单中选择PMD扫描：

![](/assets/images/2025/staticanalysis/javastaticanalysistools05.png)

结果会立即显示，但与Eclipse不同的是，如果我们尝试打开描述，它将打开一个浏览器，其中包含一个用于查找信息的公共网页：

![](/assets/images/2025/staticanalysis/javastaticanalysistools06.png)

我们可以从设置页面设置PMD插件的行为，方法是前往File -> Settings -> other settings -> PMD查看配置页面，在设置页面中，我们可以加载包含我们自定义测试规则的自定义规则集来配置规则集。

## 3. JaCoCo

JaCoCo是一个测试覆盖率工具，用于跟踪代码库中的单元测试覆盖率。简而言之，该工具使用多种策略来计算覆盖率，例如：代码行、类、方法等。

### 3.1 与Eclipse集成

JaCoCo可以直接从[应用市场](http://marketplace.eclipse.org/content/eclemma-java-code-coverage)安装，官方网站上也提供了安装[链接](http://www.eclemma.org/installation.html)。

![](/assets/images/2025/staticanalysis/javastaticanalysistools07.png)

该工具可以从项目级别执行到单个方法级别，Eclipse插件使用不同的配色方案来精确定位测试用例覆盖的代码部分和未覆盖的代码部分：

![](/assets/images/2025/staticanalysis/javastaticanalysistools08.png)

我们的方法是将两个提供的整数参数相除并返回结果，如果第二个参数为0，则返回整数数据类型的最大值。

在我们的测试用例中，我们仅测试第二个参数为0的情况：

![](/assets/images/2025/staticanalysis/javastaticanalysistools09.png)

在本例中，我们可以看到第6行被标记为黄色。在我们的简单测试中，只有if条件的一个分支被测试并运行。因此，它没有被完全测试，因此被标记为黄色。

此外，第7行显示为绿色，表示该行已通过全面测试。最后，第9行以红色突出显示，这意味着该行完全未经我们的单元测试测试。

我们可以看到测试覆盖率的摘要，其中显示了类级别和包级别的单元测试覆盖了多少代码：

![](/assets/images/2025/staticanalysis/javastaticanalysistools10.png)

### 3.2 与IntelliJ IDEA集成

JaCoCo默认与最新的IntelliJ IDEA发行版捆绑在一起，因此无需单独安装插件。

执行单元测试时，我们可以选择需要使用的覆盖率运行器，我们可以在项目级别或类级别运行测试用例：

![](/assets/images/2025/staticanalysis/javastaticanalysistools11.png)

与Eclipse类似，JaCoCo使用不同的颜色方案来显示覆盖率结果。

![](/assets/images/2025/staticanalysis/javastaticanalysistools12.png)

我们可以看到测试覆盖率的摘要，其中显示了类级别和包级别的单元测试覆盖了多少代码。

![](/assets/images/2025/staticanalysis/javastaticanalysistools13.png)

## 4. Cobertura

最后，值得一提的是Cobertura-它同样用于跟踪代码库中的单元测试覆盖率。

在撰写本文时，最新版本的Eclipse不支持Cobertura插件；该插件可以与早期版本的Eclipse一起使用。

同样，IntelliJ IDEA没有可以执行Cobertura覆盖的官方插件。

## 5. 总结

我们研究了三种常用静态分析工具与Eclipse和IntelliJ IDEA的集成，FindBug已在之前的[FindBugs简介](https://www.baeldung.com/intro-to-findbugs)中介绍过。