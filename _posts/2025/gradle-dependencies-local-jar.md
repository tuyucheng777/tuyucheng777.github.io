---
layout: post
title:  本地JAR文件作为Gradle依赖项
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本教程中，我们将重点介绍如何将本地JAR文件添加到我们的[Gradle](https://www.baeldung.com/gradle)依赖中。

## 2. 本地JAR

**在开始解释将本地JAR文件添加到Gradle的过程之前，有必要提一下，不建议手动添加公共仓库中提供的依赖**，像Gradle这样的构建系统存在的一个最重要的原因就是能够自动完成这类操作。在Gradle出现之前，我们习惯于下载JAR文件并将其放在libs文件夹中。现在，Gradle会自动为我们处理这些事情。

但是，Gradle仍然支持此过程用于特殊目的，例如自定义JAR文件。

## 3. 平面目录

如果我们想使用平面文件系统目录作为我们的仓库，我们需要将以下内容添加到我们的build.gradle文件中：

```groovy
repositories {
    flatDir {
        dirs 'lib1', 'lib2'
    }
}
```

这使得Gradle会在lib1和lib2中查找依赖，设置好平面目录后，我们就可以使用lib1或lib2文件夹中的本地JAR文件了：

```groovy
dependencies { implementation name: 'sample-jar-0.8.7' }
```

## 4. 文件集合

平面目录的另一种方法是直接提及文件而不使用flatdir：

```groovy
implementation files('libs/a.jar', 'libs/b.jar')
```

## 5. 文件树

我们可以告诉Gradle在特定目录中查找所有JAR文件，而无需缩小名称范围，当我们无法或不想将某些文件放入仓库时，这会很有用。但我们必须小心使用此功能，因为它也可能会添加不必要的依赖：

```groovy
implementation fileTree(dir: 'libs', include: '*.jar')
```

## 6. 使用IntelliJ

还有另一种利用本地jar文件的方法，首先，我们进入Project Structure：

![](/assets/images/2025/gradle/gradledependencieslocaljar01.png)

然后我们点击列表顶部的加号按钮并选择Java：

![](/assets/images/2025/gradle/gradledependencieslocaljar02.png)

然后会出现一个对话框，要求我们找到JAR文件。选择后，点击“OK”，这样我们的项目就可以访问存档中的方法和类了。

## 7. 总结

在本文中，我们研究了在Gradle项目中使用未托管在标准仓库中的JAR文件的各种方法。