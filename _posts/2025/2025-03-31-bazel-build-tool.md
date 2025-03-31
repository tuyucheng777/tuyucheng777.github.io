---
layout: post
title:  使用Bazel构建Java应用程序
category: libraries
copyright: libraries
excerpt: Bazel
---

## 1. 概述

[Bazel](https://bazel.build/)是一个用于构建和测试源代码的开源工具，类似于Maven和Gradle，**它支持多种语言的项目并为多个平台构建输出**。

在本教程中，我们将介绍使用Bazel构建简单Java应用程序所需的步骤。为了说明，我们将从一个多模块Maven项目开始，然后使用Bazel构建源代码。

我们首先[安装Bazel](https://docs.bazel.build/versions/master/install.html)。

## 2. 项目结构

让我们创建一个多模块Maven项目：

```text
bazel (root)
    pom.xml
    WORKSPACE (bazel workspace)
    |—bazelapp
        pom.xml
        BUILD (bazel build file)
        |—src
            |—main
                |—java
            |—test
                |—java
    |—bazelgreeting
        pom.xml
        BUILD (bazel build file)
        |—src
            |—main
                |—java
            |—test
                |—java
```

**WORKSPACE文件的存在为Bazel设置了工作区**，一个项目中可能有一个或多个这样的文件。在我们的示例中，我们将只在顶级项目目录中保留一个文件。

下一个重要文件是**BUILD文件，其中包含构建规则**，它使用唯一的target名称来标识每条规则。

Bazel提供了灵活性，可以根据需要配置任意数量的BUILD文件，并配置为任意粒度级别，这意味着我们可以通过相应地配置BUILD规则来构建较少数量的Java类。为了简单起见，我们将在示例中保留最少的BUILD文件。

由于Bazel BUILD配置的输出通常是一个jar文件，我们将每个包含BUILD文件的目录称为构建包。

## 3. 构建文件

### 3.1 规则配置

现在是时候配置我们的第一个构建规则来构建Java二进制文件了，让我们在属于bazelapp模块的BUILD文件中配置一个：

```text
java_binary (
    name = "BazelApp",
    srcs = glob(["src/main/java/cn/tuyucheng/taketoday/*.java"]),
    main_class = "cn.tuyucheng.taketoday.BazelApp"
)
```

让我们逐一了解配置设置：

- java_binary：规则的名称，它需要额外的属性来构建二进制文件
- name：构建目标的名称
- srcs：文件位置模式数组，指示要构建哪些Java文件
- main_class：应用程序主类的名称(可选)

### 3.2 构建执行

现在我们可以构建应用程序了，从包含WORKSPACE文件的目录中，我们在shell中执行bazel build命令来构建我们的目标：

```shell
$Bazelbuild //bazelapp:BazelApp
```

最后一个参数是BUILD文件中配置的目标名称，其格式为“//<path_to_build>:<target_name\>”。

模式的第一部分“//”表示我们从工作区目录开始。接下来，“bazelapp”是从工作区目录到BUILD文件的相对路径。最后，“BazelApp”是要构建的目标名称。

### 3.3 构建输出

我们现在应该注意到上一步中的两个二进制输出文件：

```text
bazel-bin/bazelapp/BazelApp.jar
bazel-bin/bazelapp/BazelApp
```

BazelApp.jar包含所有类，而BazelApp是执行jar文件的包装脚本。

### 3.4 可部署JAR

我们可能需要将jar及其依赖运送到不同的位置进行部署。

上面部分的包装脚本将所有依赖(jar文件)指定为BazelApp.jar的启动命令的一部分。

但是，我们也可以制作一个包含所有依赖的fat jar：

```shell
$Bazelbuild //bazelapp:BazelApp_deploy.jar
```

**在目标名称中添加“_deploy”后缀会指示Bazel将所有依赖打包到jar中并准备部署**。

## 4. 依赖

到目前为止，我们仅使用bazelapp中的文件进行构建。但是，几乎每个应用程序都有依赖。

在本节中，我们将看到如何将依赖与jar文件一起打包。

### 4.1 构建库

不过，在执行此操作之前，我们需要一个bazelapp可以使用的依赖。

让我们创建另一个名为bazelgreeting的Maven模块，并使用java_library规则为新模块配置BUILD文件，我们将此目标命名为“greeter”：

```text
java_library (
    name = "greeter",
    srcs = glob(["src/main/java/cn/tuyucheng/taketoday/*.java"])
)
```

这里我们使用了java_library规则来创建库，构建此目标后，我们将获得libgreetings.jar文件：

```text
INFO: Found 1 target...
Target //bazelgreeting:greetings up-to-date:
  bazel-bin/bazelgreeting/libgreetings.jar
```

### 4.2 配置依赖

要在bazelapp中使用greeter，我们需要一些额外的配置。首先，我们需要让包对bazelapp可见。我们可以通过在greeter包的java_library规则中添加visibility属性来实现这一点：

```text
java_library (
    name = "greeter",
    srcs = glob(["src/main/java/cn/tuyucheng/taketoday/*.java"]),
    visibility = ["//bazelapp:__pkg__"]
)
```

visibility属性使得当前包对于数组中列出的包可见。

现在在bazelapp包中，我们必须配置对greeter包的依赖，让我们使用deps属性来执行此操作：

```text
java_binary (
    name = "BazelApp",
    srcs = glob(["src/main/java/cn/tuyucheng/taketoday/*.java"]),
    main_class = "cn.tuyucheng.taketoday.BazelApp",
    deps = ["//bazelgreeting:greeter"]
)
```

deps属性使当前包依赖于数组中列出的包。

## 5. 外部依赖

我们可以处理具有多个工作区且相互依赖的项目。或者，我们可以从远程位置导入库。我们可以将这些外部依赖分类为：

- **本地依赖**：我们在同一个工作区内管理它们，就像我们在上一节中看到的那样，或者跨越多个工作区
- **HTTP档案**：我们通过HTTP从远程位置导入库

有许多Bazel规则可用于管理[外部依赖](https://docs.bazel.build/versions/master/external.html)，我们将在后续部分中了解如何从远程位置导入jar文件。

### 5.1 HTTP URL位置

为了举例，我们将[Apache Commons Lang](https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.jar)导入到我们的应用程序中。由于我们必须从HTTP位置导入此jar，因此我们将使用[http_jar](https://docs.bazel.build/versions/master/repo/http.html#http_jar)规则。我们将首先从Bazel HTTP构建定义中加载规则，并使用Apache Commons的位置在WORKSPACE文件中对其进行配置：

```text
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_jar")

http_jar (
    name = "apache-commons-lang",
    url = "https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.12.0/commons-lang3-3.12.0.jar"
)
```

我们必须在“bazelapp”包的BUILD文件中添加依赖：

```text
deps = ["//bazelgreeting:greeter", "@apache-commons-lang//jar"]
```

注意，我们需要指定与WORKSPACE文件中http_jar规则中使用的名称相同的名称。

### 5.2 Maven依赖

管理单个jar文件成为一项繁琐的任务。或者，我们可以使用WORKSPACE文件中的[rules_jvm_external](https://github.com/bazelbuild/rules_jvm_external)规则配置Maven仓库，这将使我们能够从仓库中获取项目中所需的任意数量的依赖。

首先，我们必须使用WORKSPACE文件中的http_archive规则从远程位置导入rules_jvm_external规则：

```text
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

RULES_JVM_EXTERNAL_TAG = "2.0.1"
RULES_JVM_EXTERNAL_SHA = "55e8d3951647ae3dffde22b4f7f8dee11b3f70f3f89424713debd7076197eaca"

http_archive(
    name = "rules_jvm_external",
    strip_prefix = "rules_jvm_external-%s" % RULES_JVM_EXTERNAL_TAG,
    sha256 = RULES_JVM_EXTERNAL_SHA,
    url = "https://github.com/bazelbuild/rules_jvm_external/archive/%s.zip" % RULES_JVM_EXTERNAL_TAG,
)
```

接下来，我们将使用maven_install规则并配置Maven仓库URL和所需的工件：

```text
load("@rules_jvm_external//:defs.bzl", "maven_install")

maven_install(
    artifacts = [
        "org.apache.commons:commons-lang3:3.12.0" ], 
    repositories = [ 
        "https://repo1.maven.org/maven2", 
    ] )
```

最后，我们在BUILD文件中添加依赖：

```text
deps = ["//bazelgreeting:greeter", "@maven//:org_apache_commons_commons_lang3"]
```

它使用下划线(_)字符来解析工件的名称。

## 6. 总结

在本教程中，我们学习了使用Bazel构建工具构建Maven风格Java项目的基本配置。