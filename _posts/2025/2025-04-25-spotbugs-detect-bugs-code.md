---
layout: post
title:  SpotBugs简介
category: staticanalysis
copyright: staticanalysis
excerpt: SpotBugs
---

## 1. 概述

识别Java程序中的错误是软件开发中的一项关键挑战，**[SpotBugs](https://github.com/spotbugs/spotbugs)是一款开源静态分析工具，用于查找Java代码中的错误。它基于Java字节码而非源代码进行操作，以识别代码中的潜在问题，例如错误、性能问题或不良实践**。SpotBugs是[FindBugs](https://www.baeldung.com/intro-to-findbugs)的继承者，并基于其功能进行构建，提供更详细、更精确的错误检测。在本文中，我们将探讨如何在Java项目中设置SpotBugs，并将其集成到IDE和Maven构建中。 

## 2. 错误模式

**SpotBugs可检查400多种[错误模式](https://spotbugs.readthedocs.io/en/stable/bugDescriptions.html#bug-descriptions)，例如空指针引用、无限递归循环、Java库的不当使用以及死锁**。SpotBugs会根据多种变量对错误模式进行分类，例如违规类型、类别、错误等级以及发现过程的可信度；SpotBugs有十个错误模式类别：

- **不良实践**：检测不良的编码实践，这些实践可能不会立即导致问题，但可能会在将来引发问题。例如，哈希码和相等问题、可克隆习语、丢弃异常、可序列化问题以及对finalize的误用。
- **正确性**：识别可能不正确且可能导致运行时错误(例如可能的错误)的代码。
- **实验性**：指相对较新的、实验性的或尚未完全验证的错误模式。
- **国际化**：检测Java代码中与国际化和区域设置相关的潜在问题。
- **恶意代码漏洞**：标记可能被攻击者利用的代码。
- **多线程正确性**：检查多线程代码中的潜在问题，例如竞争条件和死锁。
- **虚假随机噪声**：旨在用作数据挖掘实验中的控制，而不是用于查找软件中的实际错误。
- **性能**：识别不一定不正确但可能效率低下的代码。
- **安全性**：突出显示代码中的安全漏洞。
- **可疑代码**：查找那些不一定错误但可疑且可能存在问题的代码，这些代码令人困惑、异常或以导致错误的编写方式编写。示例包括无效的本地存储、Switch失败、未经确认的强制类型转换以及对已知为空的值进行冗余的空值检查。

SpotBugs的一项功能是能够将错误按严重程度进行分类，**SpotBugs会根据1到20的数字等级对错误进行排序，排名代表问题的严重程度**；数字等级可以分为以下几类：

- **最恐怖(高优先级)**：等级1至4
- **恐怖(中等优先级)**：等级5至9
- **麻烦(低优先级)**：等级10至14
- **值得关注(信息性)**：排名15至20

## 3. SpotBugs Maven插件

**Spotbugs可以作为独立应用程序使用，也可以通过[Maven](https://www.baeldung.com/maven)、[Gradle](https://www.baeldung.com/gradle)、Eclipse和 IntelliJ等多种集成方式使用**。在本节中，我们将重点介绍Maven集成。

### 3.1 Maven配置

让我们首先在pom.xml的<plugins\>部分导入[spotbugs-maven-plugin](https://mvnrepository.com/artifact/com.github.spotbugs/spotbugs-maven-plugin)插件：

```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.5.0</version>
    <dependencies>
        <dependency>
	    <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs</artifactId>
	    <version>4.8.5</version>
        </dependency>
    </dependencies>
</plugin>
```

### 3.2 生成报告

添加插件后，我们可以打开终端并运行以下命令：

```shell
mvn spotbugs:check
```

这将对我们的源代码进行分析，然后输出我们需要修复的警告列表。要生成[错误报告](https://www.baeldung.com/cs/write-good-bug-reports)，我们需要一些代码，为了简单起见，我们将使用[GitHub](https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries-2)上提供的项目，假设我们使用以下类：

```java
public class Application {

    public static final String NAME = "Name: ";

    private Application() {
    }

    public static String readName() {
        Scanner scanner = new Scanner(System.in);
        String input = scanner.next();
        return NAME.concat(input);
    }
}
```

现在，我们可以运行mvn spotbugs:check命令了，运行过程中检测到了以下情况：

```text
[INFO] BugInstance size is 1
[INFO] Error size is 0
[INFO] Total bugs: 1
[ERROR] High: Found reliance on default encoding in cn.tuyucheng.taketoday.systemin.Application.readName(): new java.util.Scanner(InputStream) [cn.tuyucheng.taketoday.systemin.Application] At Application.java:[line 13] DM_DEFAULT_ENCODING
```

从该报告中，我们可以看到我们的Application类有一个高优先级的错误。

### 3.3 查看结果

SpotBugs会在target/spotbugsXml.xml中生成XML格式的报告，为了获得更美观的报告，我们需要在SpotBugs插件中添加htmlOutput配置：

```xml
<configuration>
    <htmlOutput>true</htmlOutput>
</configuration>
```

现在，我们运行mvn clean install和mvn spotbugs:check命令，然后，我们导航到target/spotbugs.html并打开文件，直接在浏览器中查看结果：

![](/assets/images/2025/staticanalysis/spotbugsdetectbugscode01.png)

此外，我们还可以使用Maven命令mvn spotbugs:gui在Spotbugs GUI中查看错误详情：

![](/assets/images/2025/staticanalysis/spotbugsdetectbugscode02.png)

有了这些反馈，我们现在可以更新代码，主动修复错误。

### 3.4 修复错误

我们的Application类存在[DM_DEFAULT_ENCODING](https://spotbugs.readthedocs.io/en/latest/bugDescriptions.html#dm-reliance-on-default-encoding-dm-default-encoding)错误，该错误表明在执行I/O操作时使用了默认字符编码，如果默认编码在不同环境或平台上有所不同，则可能导致意外行为。通过显式指定字符编码，我们可以确保无论平台的默认编码如何，行为都保持一致。为了修复此问题，我们可以将StandardCharsets添加到Scanner类：

```java
public static String readName() {
    Scanner scanner = new Scanner(System.in, StandardCharsets.UTF_8.displayName());
    String input = scanner.next();
    return NAME.concat(input);
}
```

添加之前建议的更改后，再次运行mvn spotbugs:check将生成无错误的报告：

```text
[INFO] BugInstance size is 0
[INFO] Error size is 0
[INFO] No errors/warnings found
```

## 4. SpotBugs IntelliJ IDEA插件

**IntelliJ SpotBugs插件提供静态字节码分析功能，可在IntelliJ IDEA中查找Java代码中的错误**，该插件在底层使用了SpotBugs。

### 4.1 安装

要在IntelliJ IDEA中安装SpotBugs插件，我们打开应用程序并导航到“Welcome Screen”，如果项目已打开，我们转到“File -> Settings”(或在macOS上转到“IntelliJ IDEA -> Preferences”)，在设置窗口中，我们选择“Plugins”，然后转到“Marketplace”选项卡。使用搜索栏，我们找到“SpotBugs”，找到后单击“Install”按钮。安装完成后，我们重新启动IntelliJ IDEA以激活插件：

![](/assets/images/2025/staticanalysis/spotbugsdetectbugscode03.png)

要手动安装SpotBugs插件，我们首先从[JetBrains插件库](https://plugins.jetbrains.com/plugin/14014-spotbugs/versions#tabs)下载插件，下载插件zip文件后，我们打开IntelliJ IDEA并导航到“Welcome Screen”，然后转到“File > Settings”。在设置窗口中，我们选择“Plugins”，单击右上角的“齿轮图标”，然后选择“Install Plugin from Disk”。然后，我们导航到下载SpotBugs插件zip文件的位置，选择它，然后单击“OK”。最后，我们重新启动IntelliJ IDEA以激活该插件。

### 4.2 报告浏览

为了在IDEA中启动静态分析，我们可以点击SpotBugs-IDEA面板中的“Analyze Current File”：

![](/assets/images/2025/staticanalysis/spotbugsdetectbugscode04.png)

然后检查结果：我们可以使用屏幕截图左侧的第二列命令，根据各种因素(例如错误类别、级别、软件包、优先级或错误等级)对错误进行分组，还可以通过点击第三列命令中的“export”按钮，将报告导出为XML/HTML格式。

### 4.3 配置

IntelliJ IDEA的 SpotBugs插件提供了各种首选项和设置，允许用户自定义分析流程并管理错误报告的方式，下图显示了SpotBugs插件的首选项：

![](/assets/images/2025/staticanalysis/spotbugsdetectbugscode05.png)

以下是SpotBugs插件中常用首选项的概述：

- **错误类别和优先级**：我们可以配置SpotBugs检测的错误类别，我们还可以设置错误检测的最低级别，以便我们专注于更关键的问题。
- **过滤文件**：此选项卡允许我们管理代码库的哪些部分需要纳入分析或排除分析，我们可以定义模式，以从分析中排除或包含特定的包、类或方法。
- **注解**：该插件支持为类、方法和字段配置注解，这些注解可用于抑制特定警告或向SpotBugs提供额外信息。
- **检测器**：它提供与SpotBugs使用的检测机制相关的设置，我们可以根据需要启用或禁用特定的错误检测器。例如，可以通过启用安全相关的检测器并禁用其他检测器来关注安全问题。

这些首选项和设置提供了灵活性，并可控制SpotBugs如何分析代码和报告潜在问题，帮助开发人员保持高代码质量并有效地解决错误。

## 5. SpotBugs Eclipse插件

在本节中，我们将重点介绍SpotBugs Eclipse插件，深入研究其安装过程以及在Eclipse IDE中的使用。

### 5.1 安装

我们首先启动Eclipse IDE并导航到菜单栏中的“Help”。之后，我们选择“Eclipse Marketplace”，在Marketplace窗口中，我们在搜索框中输入“SpotBugs”并按“Enter”，当搜索结果出现时，我们找到SpotBugs并点击旁边的“Install”按钮。我们按照安装向导完成安装过程，最后，我们重新启动Eclipse以激活插件：

![](/assets/images/2025/staticanalysis/spotbugsdetectbugscode06.png)

或者，如果SpotBugs在“Eclipse Marketplace”中不可用，或者我们更喜欢通过更新站点安装它，我们可以打开Eclipse并转到顶部的“Help”菜单。从下拉菜单中，我们选择“Install New Software”，然后在出现的对话框中单击“Add”按钮，在“Add Repository”对话框中，我们输入存储库的名称，例如“SpotBugs”，并在“Location”字段中输入https://spotbugs.github.io/eclipse URL。点击“OK”后，我们在可用软件列表中勾选“SpotBugs”旁边的复选框，点击“Next”，按照提示完成安装。最后，我们重启Eclipse即可激活插件。

### 5.2 报告浏览

要使用SpotBugs Eclipse插件对项目进行静态分析，我们需要在“Project Explorer”中右键单击该项目，然后导航到“SpotBugs -> Find Bugs”。之后，Eclipse会在“Bug Explorer”窗口下显示结果。

### 5.3 配置

我们可以通过“Window -> Preferences -> Java -> SpotBugs”来检查配置界面：

![](/assets/images/2025/staticanalysis/spotbugsdetectbugscode07.png)

我们可以自由地取消选中不需要的类别，提高要报告的最低等级，指定要报告的最低置信度，并自定义错误等级的标记。在“Detector configuration”选项卡下，Eclipse SpotBugs插件允许我们自定义启用或禁用哪些错误检测器，此选项卡提供了可用检测器的详细列表，每个检测器都旨在识别代码中特定类型的潜在问题。在“Filter files”面板下，我们可以创建自定义文件过滤器，允许我们在SpotBugs分析中包含或排除特定文件或目录，此功能可以对要分析的代码库部分进行微调控制，确保不相关或不必要的文件不会包含在错误检测过程中。

## 6. 总结

在本文中，我们讨论了在Java项目中使用SpotBugs的基本要点。SpotBugs是一款静态分析工具，它通过检查字节码来帮助识别代码中的潜在错误，我们学习了如何使用Maven(Java生态系统中广泛使用的构建自动化工具)配置SpotBugs，以便在构建过程中自动检测问题；此外，我们还演示了将SpotBugs集成到常用开发环境(包括IntelliJ IDEA和Eclipse)的过程，以确保我们能够轻松地将错误检测功能融入到我们的工作流程中。