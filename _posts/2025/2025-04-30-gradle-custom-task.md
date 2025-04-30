---
layout: post
title:  Gradle中的自定义任务
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本文中，我们将介绍如何在Gradle中创建自定义任务，我们将展示如何使用构建脚本或自定义任务类型来定义新的任务。

有关Gradle的介绍，请参阅[本文](https://www.baeldung.com/gradle)，它包含Gradle的基础知识以及本文最重要的内容-Gradle任务介绍。

## 2. build.gradle中的自定义任务定义

要创建一个简单的Gradle任务，我们需要将其定义添加到我们的build.gradle文件中：

```groovy
task welcome {
    doLast {
        println 'Welcome in the Tuyucheng!'
    }
}
```

上述任务的主要目标是打印文本“Welcome in the Tuyucheng!”，我们可以通过运行gradle tasks–all命令来检查此任务是否可用：

```shell
gradle tasks --all
```

该任务位于其他任务组下的列表中：

```text
Other tasks
-----------
welcome
```

它可以像任何其他Gradle任务一样执行：

```shell
gradle welcome
```

输出正如预期的那样——“Welcome in the Tuyucheng!”消息。

备注：如果未设置选项–all，则属于“其他”类别的任务不可见。自定义Gradle任务可以属于与“其他”不同的组，并且可以包含描述。

## 3. 设置组和描述

有时按功能对任务进行分组会很方便，这样它们就可以在一个类别下显示，我们可以通过**定义一个group属性来快速为自定义任务设置组**：

```groovy
task welcome {
    group 'Sample category'
    doLast {
        println 'Welcome on the Tuyucheng!'
    }
}
```

现在，当我们运行Gradle命令列出所有可用任务(不再需要–all选项)时，我们将在新组下看到我们的任务：

```text
Sample category tasks
---------------------
welcome
```

但是，让其他人了解任务的职责也是有益的，我们可以创建一个包含简短信息的描述：

```groovy
task welcome {
    group 'Sample category'
    description 'Tasks which shows a welcome message'
    doLast {
        println 'Welcome in the Tuyucheng!'
    }
}
```

当我们打印可用任务列表时，输出将如下所示：

```text
Sample category tasks
---------------------
welcome - Tasks which shows a welcome message
```

这种任务定义称为临时定义。

进一步说，创建一个可自定义且可重复使用的任务非常有益，我们将介绍如何根据类型创建任务，以及如何为该任务的用户提供一些自定义功能。

## 4. 在build.gradle中定义Gradle任务类型

上面的“welcome”任务无法自定义，因此在大多数情况下，它没什么用。我们可以运行它，但如果我们在不同的项目(或子项目)中需要它，那么我们需要复制并粘贴它的定义。

我们可以**通过创建任务类型来快速启用任务的自定义**，只需在构建脚本中定义一个任务类型即可：

```groovy
class PrintToolVersionTask extends DefaultTask {
    String tool

    @TaskAction
    void printToolVersion() {
        switch (tool) {
            case 'java':
                println System.getProperty("java.version")
                break
            case 'groovy':
                println GroovySystem.version
                break
            default:
                throw new IllegalArgumentException("Unknown tool")
        }
    }
}
```

**自定义任务类型是一个简单的Groovy类，它扩展了DefaultTask类**-该类定义了标准任务的实现。我们也可以扩展其他任务类型，但在大多数情况下，DefaultTask类是合适的选择。

PrintToolVersionTask任务包含tool属性，可以通过该任务的实例进行自定义：

```groovy
String tool
```

我们可以根据需要添加任意数量的属性-请记住它只是一个简单的Groovy类字段。

此外，它还包含一个用@TaskAction标注的方法，它定义了这个任务正在做什么。在这个简单的例子中，它根据给定的参数值打印已安装的Java或Groovy版本。

要根据创建的任务类型运行自定义任务，我们需要创建此类型的新任务实例：

```groovy
task printJavaVersion(type : PrintToolVersionTask) {
    tool 'java'
}
```

最重要的部分是：

- 我们的任务是PrintToolVersionTask类型，因此执行时它将触发用@TaskAction标注的方法中定义的操作
- 我们添加了一个自定义工具属性值(java)，它将由PrintToolVersionTask使用

当我们运行上述任务时，输出符合预期(取决于安装的Java版本)：

```text
> Task :printJavaVersion 
9.0.1
```

现在让我们创建一个打印已安装的Groovy版本的任务：

```groovy
task printGroovyVersion(type : PrintToolVersionTask) {
    tool 'groovy'
}
```

它使用与我们之前定义的相同的任务类型，但具有不同的工具属性值。当我们执行此任务时，它会打印Groovy版本：

```text
> Task :printGroovyVersion 
2.4.12
```

**如果自定义任务不太多，我们可以像上面那样直接在build.gradle文件中定义它们**。但是，如果自定义任务太多，build.gradle文件就会变得难以阅读和理解。

幸运的是，Gradle为此提供了一些解决方案。

## 5. 在buildSrc文件夹中定义任务类型

**我们可以在位于项目根目录的buildSrc文件夹中定义任务类型**，Gradle会编译其中的所有内容，并将类型添加到类路径中，以便我们的构建脚本可以使用它。

我们之前定义的任务类型(PrintToolVersionTask)可以移到buildSrc/src/main/groovy/cn/tuyucheng/taketoday/PrintToolVersionTask.groovy中，我们只需要在移动后的类中添加一些来自Gradle API的导入即可。

我们可以在buildSrc文件夹中定义无限数量的任务类型，这样更易于维护和阅读，并且任务类型声明与任务实例不在同一个位置。

我们可以像使用构建脚本中直接定义的类型一样使用这些类型，我们只需记住添加适当的导入即可。

## 6. 在插件中定义任务类型

我们可以在自定义Gradle插件中定义自定义任务类型。请参阅[这篇](https://www.baeldung.com/gradle-create-plugin)文章，其中介绍了如何定义自定义Gradle插件，其定义在：

- build.gradle文件
- buildSrc文件夹与其他Groovy类相同

当我们定义此插件的依赖时，这些自定义任务将可用于我们的构建。请注意，除了自定义任务类型外，临时任务也可用。

## 7. 总结

在本教程中，我们介绍了如何在Gradle中创建自定义任务，你可以在build.gradle文件中使用许多可用的插件，它们将提供你需要的各种自定义任务类型。