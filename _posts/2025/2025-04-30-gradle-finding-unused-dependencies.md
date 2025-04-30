---
layout: post
title:  查找未使用的Gradle依赖
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

有时在开发过程中，我们可能会添加比我们使用的更多的依赖。

在本快速教程中，我们将了解如何使用[Gradle Nebula Lint](https://github.com/nebula-plugins/gradle-lint-plugin)插件来识别和修复这些问题。

## 2. 设置和配置

我们在示例中使用多模块Gradle 5设置。

**该插件仅适用于基于Groovy的构建文件**。

我们在根项目构建文件中进行配置：

```groovy
plugins {
    id "nebula.lint" version "16.9.0"
}

description = "Gradle 5 root project"

allprojects {
    apply plugin :"java"
    apply plugin :"nebula.lint"
    gradleLint {
        rules=['unused-dependency']
    }
    group = "cn.tuyucheng.taketoday"
    version = "0.0.1"
    sourceCompatibility = "1.8"
    targetCompatibility = "1.8"

    repositories {
        jcenter()
    }
}
```

**目前我们只能在多项目构建时这样配置**，这意味着我们无法在每个模块中单独应用它。

接下来，让我们配置模块依赖：

```groovy
description = "Gradle Unused Dependencies example"

dependencies {
    implementation('com.google.guava:guava:29.0-jre')
    testImplementation('junit:junit:4.12')
}
```

现在让我们在模块源中添加一个简单的主类：

```java
public class UnusedDependencies {

    public static void main(String[] args) {
        System.out.println("Hello world");
    }
}
```

稍后我们将在此基础上进行构建，并了解插件的工作原理。

## 3. 检测场景和报告

该插件搜索输出jar来检测是否使用了依赖。

但是，**根据[不同的条件](https://github.com/nebula-plugins/gradle-lint-plugin/wiki/Unused-Dependency-Rule)，它可能给我们不同的结果**。

我们将在下一节中探讨更有趣的案例。

### 3.1 未使用的依赖

现在我们已经完成了设置，让我们看看基本的用例，我们感兴趣的是未使用的依赖。

让我们运行lintGradle任务：

```shell
$ ./gradlew lintGradle

> Task :lintGradle FAILED
# failure output omitted

warning   unused-dependency                  this dependency is unused and can be removed
unused-dependencies/build.gradle:6
implementation('com.google.guava:guava:29.0-jre')

✖ 1 problem (0 errors, 1 warning)

To apply fixes automatically, run fixGradleLint, review, and commit the changes.
# some more failure output
```

我们的compileClasspath配置中有一个未使用的依赖(guava)。

如果我们按照插件的建议运行fixGradleLint任务，**依赖将自动从我们的build.gradle中删除**。

但是，让我们使用一些带有依赖关系的虚拟逻辑：

```java
public static void main(String[] args) {
    System.out.println("Hello world");
    useGuava();
}

private static void useGuava() {
    List<String> list = ImmutableList.of("Tuyucheng", "is", "cool");
    System.out.println(list.stream().collect(Collectors.joining(" ")));
}
```

如果我们重新运行它，就不会再出现错误：

```shell
$ ./gradlew lintGradle

BUILD SUCCESSFUL in 559ms
3 actionable tasks: 1 executed, 2 up-to-date
```

### 3.2 使用传递依赖

现在让我们包含另一个依赖：

```groovy
dependencies {
    implementation('com.google.guava:guava:29.0-jre')
    implementation('org.apache.httpcomponents:httpclient:4.5.12')
    testImplementation('junit:junit:4.12')
}
```

这次，让我们使用一些传递依赖的东西：

```java
public static void main(String[] args) {
    System.out.println("Hello world");
    useGuava();
    useHttpCore();
}

// other methods

private static void useHttpCore() {
    SSLContextBuilder.create();
}
```

让我们看看会发生什么：

```shell
$ ./gradlew lintGradle

> Task :lintGradle FAILED
# failure output omitted 

warning   unused-dependency                  one or more classes in org.apache.httpcomponents:httpcore:4.4.13 
are required by your code directly (no auto-fix available)
warning   unused-dependency                  this dependency is unused and can be removed 
unused-dependencies/build.gradle:8
implementation('org.apache.httpcomponents:httpclient:4.5.12')

✖ 2 problems (0 errors, 2 warnings)
```

我们收到两个错误。第一个错误大致告诉我们应该直接引用httpcore。

我们的示例中的SSLContextBuilder实际上是它的一部分。

第二个错误表明我们没有使用httpclient中的任何内容。

**如果我们使用传递依赖，插件会告诉我们使其成为直接依赖**。

让我们看一下依赖树：

```shell
$ ./gradlew unused-dependencies:dependencies --configuration compileClasspath

> Task :unused-dependencies:dependencies

------------------------------------------------------------
Project :unused-dependencies - Gradle Unused Dependencies example
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
+--- com.google.guava:guava:29.0-jre
|    +--- com.google.guava:failureaccess:1.0.1
|    +--- com.google.guava:listenablefuture:9999.0-empty-to-avoid-conflict-with-guava
|    +--- com.google.code.findbugs:jsr305:3.0.2
|    +--- org.checkerframework:checker-qual:2.11.1
|    +--- com.google.errorprone:error_prone_annotations:2.3.4
|    \--- com.google.j2objc:j2objc-annotations:1.3
\--- org.apache.httpcomponents:httpclient:4.5.12
     +--- org.apache.httpcomponents:httpcore:4.4.13
     +--- commons-logging:commons-logging:1.2
     \--- commons-codec:commons-codec:1.11
```

本例中我们可以看到httpcore是由httpclient引入的。

### 3.3 通过反射使用依赖

当我们使用反射时会怎样？

让我们稍微增强一下我们的例子：

```java
public static void main(String[] args) {
    System.out.println("Hello world");
    useGuava();
    useHttpCore();
    useHttpClientWithReflection();
}

// other methods

private static void useHttpClientWithReflection() {
    try {
        Class<?> httpBuilder = Class.forName("org.apache.http.impl.client.HttpClientBuilder");
        Method create = httpBuilder.getMethod("create", null);
        create.invoke(httpBuilder, null);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

现在让我们重新运行Gradle任务：

```shell
$ ./gradlew lintGradle

> Task :lintGradle FAILED
# failure output omitted

warning   unused-dependency                  one or more classes in org.apache.httpcomponents:httpcore:4.4.13 
are required by your code directly (no auto-fix available)

warning   unused-dependency                  this dependency is unused and can be removed
unused-dependencies/build.gradle:9
implementation('org.apache.httpcomponents:httpclient:4.5.12')

✖ 2 problems (0 errors, 2 warnings)
```

发生了什么？我们使用了依赖(httpclient)中的HttpClientBuilder，但仍然出现错误。

**如果我们使用具有反射的库，插件将无法检测其使用情况**。

结果，我们可以看到相同的两个错误。

一般来说，我们应该配置诸如runtimeOnly这样的依赖。

### 3.4 生成报告

对于大型项目来说，终端返回的错误数量变得难以处理。

让我们配置插件来为我们提供报告：

```groovy
allprojects {
    apply plugin :"java"
    apply plugin :"nebula.lint"
    gradleLint {
        rules=['unused-dependency']
        reportFormat = 'text'
    }
    // other  details omitted
}
```

让我们运行generateGradleLintReport任务并检查我们的构建输出：

```shell
$ ./gradlew generateGradleLintReport
# task output omitted

$ cat unused-dependencies/build/reports/gradleLint/unused-dependencies.txt

CodeNarc Report - Jun 20, 2020, 3:25:28 PM

Summary: TotalFiles=1 FilesWithViolations=1 P1=0 P2=3 P3=0

File: /home/user/tutorials/gradle-5/unused-dependencies/build.gradle
    Violation: Rule=unused-dependency P=2 Line=null Msg=[one or more classes in org.apache.httpcomponents:httpcore:4.4.13 
                                                         are required by your code directly]
    Violation: Rule=unused-dependency P=2 Line=9 Msg=[this dependency is unused and can be removed] 
                                                 Src=[implementation('org.apache.httpcomponents:httpclient:4.5.12')]
    Violation: Rule=unused-dependency P=2 Line=17 Msg=[this dependency is unused and can be removed] 
                                                  Src=[testImplementation('junit:junit:4.12')]

[CodeNarc (http://www.codenarc.org) v0.25.2]
```

现在它可以检测testCompileClasspath配置上未使用的依赖。

不幸的是，这是插件的不一致行为，结果，我们现在得到了三个错误。

## 4. 总结

在本教程中，我们了解了如何在Gradle构建中查找未使用的依赖。

首先，我们解释了常规设置。之后，我们探讨了不同依赖报告的错误及其用法。

最后，我们了解了如何生成基于文本的报告。