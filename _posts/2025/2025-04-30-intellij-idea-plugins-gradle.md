---
layout: post
title:  使用Gradle编写IntelliJ IDEA插件
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

过去几年，JetBrains的[IntelliJ](https://www.jetbrains.com/idea/)迅速成为Java开发者的首选IDE，在我们最新的[Java现状报告](https://www.baeldung.com/java-in-2019)中，61%的受访者选择IntelliJ作为IDE，相比去年的55%有所上升。

IntelliJ对Java开发人员如此有吸引力的一个特点是**能够使用插件扩展和创建新功能**。

在本教程中，我们将学习如何使用新推荐的Gradle方式编写IntelliJ插件，并演示几种扩展IDE的方法，本文是[上一篇](https://www.baeldung.com/intellij-new-custom-plugin)介绍如何使用Plugin Devkit创建相同插件的文章的改编。

## 2. 插件的主要类型

最常见的插件类型包括以下功能：

- Custom language support：能够编写、解释和编译用不同语言编写的代码
- Framework integration：支持Spring等第三方框架
- Tool integration：与Gradle等外部工具集成
- User interface add-ons：新菜单项、工具窗口、进度条等

**插件通常分为多个类别**，例如，IntelliJ自带的[Git插件](https://www.jetbrains.com/help/idea/using-git-integration.html)会与系统上安装的git可执行文件进行交互，该插件提供其工具窗口和弹出菜单项，同时还集成到项目创建工作流程、首选项窗口等。

## 3. 创建插件

有两种创建插件的方式，对于使用[Gradle](https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html#using-gradle)的新项目，我们将使用推荐的方式，而不是使用其[Plugin Devkit](https://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html#using-devkit)。

通过使用New > Project菜单可以创建基于Gradle的插件。

![](/assets/images/2025/gradle/intellijideapluginsgradle01.png)

请注意，我们必须包含Java和IntelliJ平台插件，以确保所需的插件类在类路径上可用。

截至撰写本文时，**我们只能使用JDK 8编写IntelliJ插件**。

## 4. 示例插件

我们将创建一个插件，以便从IDE中的多个区域快速访问流行的Stack Overflow网站，它将包括：

- 工具菜单项，用于访问“提问”页面
- 文本编辑器和控制台输出中的弹出菜单项，用于在Stack Overflow上搜索突出显示的文本

### 4.1 创建操作

**操作是访问插件的最常见方式**，操作由IDE中的事件触发，例如点击菜单项或工具栏按钮。

创建操作的第一步是创建一个扩展AnAction的Java类，对于我们的Stack Overflow插件，我们将创建两个操作。

第一个操作在新浏览器窗口中打开“提问”页面：

```java
public class AskQuestionAction extends AnAction {
    @Override
    public void actionPerformed(AnActionEvent e) {
        BrowserUtil.browse("https://stackoverflow.com/questions/ask");
    }
}
```

我们使用内置的BrowserUtil类来处理在不同的操作系统和浏览器上打开网页的所有细微差别。

我们需要两个参数来在StackOverflow上执行搜索：语言标签和要搜索的文本。

为了获取语言标签，我们将使用[程序结构接口](http://www.jetbrains.org/intellij/sdk/docs/basics/architectural_overview/psi.html)(PSI)。**此API会解析项目中的所有文件，并提供一种编程式的方式来检查它们**。

在这种情况下，我们使用PSI来确定文件的编程语言：

```java
Optional<PsiFile> psiFile = Optional.ofNullable(e.getData(LangDataKeys.PSI_FILE));
String languageTag = psiFile.map(PsiFile::getLanguage)
    .map(Language::getDisplayName)
    .map(String::toLowerCase)
    .map(lang -> "[" + lang + "]")
    .orElse("");
```

为了获取要搜索的文本，我们将使用Editor API来检索屏幕上突出显示的文本：

```java
Editor editor = e.getRequiredData(CommonDataKeys.EDITOR);
CaretModel caretModel = editor.getCaretModel();
String selectedText = caretModel.getCurrentCaret().getSelectedText();
```

尽管此操作对于编辑器和控制台窗口来说是相同的，但访问所选文本的方式相同。

现在，我们可以将所有这些放在一个actionPerformed声明中：

```java
@Override
public void actionPerformed(@NotNull AnActionEvent e) {
    Optional<PsiFile> psiFile = Optional.ofNullable(e.getData(LangDataKeys.PSI_FILE));
    String languageTag = psiFile.map(PsiFile::getLanguage)
        .map(Language::getDisplayName)
        .map(String::toLowerCase)
        .map(lang -> "[" + lang + "]")
        .orElse("");

    Editor editor = e.getRequiredData(CommonDataKeys.EDITOR);
    CaretModel caretModel = editor.getCaretModel();
    String selectedText = caretModel.getCurrentCaret().getSelectedText();

    BrowserUtil.browse("https://stackoverflow.com/search?q=" + languageTag + selectedText);
}
```

此操作还重写了第二个名为update的方法，该方法允许我们在不同条件下启用或禁用该操作。在本例中，如果没有选定文本，我们将禁用搜索操作：

```java
Editor editor = e.getRequiredData(CommonDataKeys.EDITOR);
CaretModel caretModel = editor.getCaretModel();
e.getPresentation().setEnabledAndVisible(caretModel.getCurrentCaret().hasSelection());
```

### 4.2 注册操作

编写完操作后，**我们需要将其注册到IDE中**，有两种方法可以做到这一点。

第一种方法是使用plugin.xml文件，该文件是在我们开始新项目时为我们创建的。

默认情况下，该文件将有一个空的<actions\>元素，我们将在其中添加操作：

```xml
<actions>
    <action
            id="StackOverflow.AskQuestion.ToolsMenu"
            class="cn.tuyucheng.taketoday.intellij.stackoverflowplugin.AskQuestionAction"
            text="Ask Question on Stack Overflow"
            description="Ask a Question on Stack Overflow">
        <add-to-group group-id="ToolsMenu" anchor="last"/>
    </action>
    <action
            id="StackOverflow.Search.Editor"
            class="cn.tuyucheng.taketoday.intellij.stackoverflowplugin.SearchAction"
            text="Search on Stack Overflow"
            description="Search on Stack Overflow">
        <add-to-group group-id="EditorPopupMenu" anchor="last"/>
    </action>
    <action
            id="StackOverflow.Search.Console"
            class="cn.tuyucheng.taketoday.intellij.stackoverflowplugin.SearchAction"
            text="Search on Stack Overflow"
            description="Search on Stack Overflow">
        <add-to-group group-id="ConsoleEditorPopupMenu" anchor="last"/>
    </action>
</actions>
```

使用XML文件注册操作将确保它们在IDE启动期间注册，这通常是更好的选择。

注册操作的第二种方法是以编程方式使用ActionManager类：

```java
ActionManager.getInstance().registerAction("StackOverflow.SearchAction", new SearchAction());
```

这样做的好处是，我们可以动态地注册操作。例如，如果我们编写一个插件来与远程API集成，我们可能希望根据调用的API版本注册一组不同的操作。

这种方法的缺点是操作不会在启动时注册，我们必须创建一个ApplicationComponent实例来管理操作，这需要更多的代码和XML配置。

## 5. 测试插件

与任何程序一样，编写IntelliJ插件也需要测试，对于像我们编写的这种小型插件，只需确保插件能够编译，并且我们创建的操作在点击时能够按预期工作即可。

我们可以通过打开Gradle工具窗口并执行runIde任务来手动测试(和调试)我们的插件：

![](/assets/images/2025/gradle/intellijideapluginsgradle02.png)

这将启动一个新的IntelliJ实例，并激活我们的插件，这样我们就可以点击之前创建的不同菜单项，并确保打开正确的Stack Overflow页面。

如果我们希望进行更传统的单元测试，IntelliJ提供了一个[无头环境](http://www.jetbrains.org/intellij/sdk/docs/basics/testing_plugins.html)来运行单元测试。我们可以使用任何我们想要的测试框架编写测试，并且测试会使用IDE中真实的、未模拟的组件运行。

## 6. 部署插件

Gradle插件提供了一种简单的方法来打包插件，以便我们安装和分发它们，只需打开Gradle工具窗口并执行buildPlugin任务即可，这将在build/distributions目录中生成一个ZIP文件。

生成的ZIP文件包含加载到IntelliJ所需的代码和配置文件，我们可以将其安装在本地，也可以将其发布到[插件库](http://www.jetbrains.org/intellij/sdk/docs/plugin_repository/index.html)供其他人使用。

下面的截图展示了Stack Overflow中新菜单项的运行情况：

![](/assets/images/2025/gradle/intellijideapluginsgradle03.png)

## 7. 总结

在本文中，我们开发了一个简单的插件，重点介绍了如何增强IntelliJ IDE。

虽然我们主要使用操作，但IntelliJ插件SDK提供了多种向IDE添加新功能的方法。如需进一步了解，请查看其[官方入门指南](http://www.jetbrains.org/intellij/sdk/docs/welcome.html)。