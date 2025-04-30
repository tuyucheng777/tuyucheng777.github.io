---
layout: post
title:  Gradle Lint插件简介
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本文中，我们将探讨[gradle-lint](https://github.com/nebula-plugins/gradle-lint-plugin)插件。

首先，我们将了解何时使用它。然后，我们将介绍插件的配置选项。接下来，我们将使用一些预定义的规则。最后，我们将生成Lint报告。

## 2. 什么是Gradle Lint插件？

**[Gradle Lint插件](https://github.com/nebula-plugins/gradle-lint-plugin)有助于对[Gradle](https://www.baeldung.com/gradle)配置文件进行lint操作**，它在我们的整个代码库中强制执行构建脚本结构，该插件可以保持[Gradle Wrapper](https://www.baeldung.com/gradle#gradle-wrapper)版本为最新版本，防止构建文件中出现不良做法，并[删除未使用的依赖](https://www.baeldung.com/gradle-finding-unused-dependencies)。 

实际上，我们会使用预定义规则或编写自定义规则。然后，我们会配置插件，将其视为违规或忽略，linter会在大多数Gradle任务结束时运行。

**默认情况下，它不会直接修改代码，而是显示警告**。这很有用，因为其他Gradle任务不会因为该插件而失败。此外，这些警告不会丢失，因为它们显示在日志末尾。

此外，[gradle-lint](https://github.com/nebula-plugins/gradle-lint-plugin)提供了fixGradleLint命令来自动修复大多数lint违规行为。

**Gradle Lint插件内部利用[Groovy AST](https://groovy-lang.org/metaprogramming.html#_available_ast_transformations)和[Gradle模型](https://github.com/nebula-plugins/gradle-lint-plugin/blob/main/src/main/groovy/com/netflix/nebula/lint/rule/GradleModelAware.groovy)来应用Lint规则**，这表明该插件与Groovy AST紧密耦合，因此，[该插件不支持Kotlin构建脚本](https://github.com/nebula-plugins/gradle-lint-plugin/issues/166)。

## 3. 设置

**有三种方法可以设置Gradle Lint插件：在build.gradle中，[初始化脚本](https://docs.gradle.org/current/userguide/init_scripts.html)或[脚本插件](https://docs.gradle.org/current/userguide/plugins.html#sec:script_plugins)**，让我们逐一探索一下。

### 3.1 使用build.gradle

让我们将[gradle-lint](https://mvnrepository.com/artifact/com.netflix.nebula/gradle-lint-plugin)插件添加到我们的build.gradle中：

```groovy
plugins {
    id "nebula.lint" version "17.8.0"
}
```

**使用多模块项目时，应将插件应用于根项目**，由于我们将使用[多模块项目](https://www.baeldung.com/gradle#multi-projects)，因此我们将插件应用于根项目：

```groovy
allprojects {
    apply plugin :"nebula.lint"
    gradleLint {
        rules = [] // we'll add rules here
    }
}
```

稍后，我们将在gradleLint.rules数组中添加lint规则。

### 3.2 使用初始化脚本

除了build.gradle，我们还可以使用[初始化脚本](https://docs.gradle.org/current/userguide/init_scripts.html)来配置我们的插件：

```groovy
import com.netflix.nebula.lint.plugin.GradleLintPlugin

initscript {
    repositories { mavenCentral() }
    dependencies {
        classpath 'com.netflix.nebula:gradle-lint-plugin:17.8.0'
    }
}

allprojects {
    apply plugin: GradleLintPlugin
    gradleLint {
        rules=[]
    }
}
```

这个lint.gradle脚本与我们之前的设置完全相同，为了应用它，我们将–init-script标志传递给我们的任务：

```shell
./gradlew build --init-script lint.gradle
```

**一个有趣的用例是根据任务运行的环境传递不同的初始化脚本**，缺点是我们必须始终传递–init-script标志。

### 3.3 使用脚本插件

让我们将[脚本插件](https://docs.gradle.org/current/userguide/plugins.html#sec:script_plugins)直接应用到我们的build.gradle中：

```groovy
plugins{
    id "nebula.lint" version "17.8.0"
}
apply from: "gradle-lint-intro.gradle"
```

gradle-lint-intro.gradle内容将被注入，就好像它是构建脚本的一部分一样：

```groovy
allprojects {
    apply plugin: "nebula.lint"
    gradleLint {
        rules= []
    }
}
```

在本文中，我们将把gradle-lint配置保留在build.gradle脚本中。

## 4. 配置

让我们回顾一下grade-lint插件中可用的不同配置选项。

### 4.1 执行

**执行Gradle Lint插件的命令是./gradlew lintGradle**，我们可以单独调用它，也可以在其他任务中调用它。

默认情况下，插件会在除以下任务之外的所有任务结束时自动运行-help、tasks、dependencies、dependencyInsight、components、model、projects、wrapper和properties：

```shell
./gradlew build
```

此构建任务以调用./gradlew lintGradle结束，linter会将违规行为显示为警告。

此外，**我们可以通过应用skipForTask方法阻止linter在特定任务中运行**：

```groovy
gradleLint {
    skipForTask('build')
}
```

在这里，我们阻止插件在构建任务期间运行。

**如果我们想为所有任务禁用该插件，我们可以使用alwaysRun标志**：

```groovy
gradleLint {
    alwaysRun = false
}
```

当我们想要在特定时间点单独调用./gradlew lintGradle时，这特别有用。

**值得注意的是，单独调用插件会将lint违规标记为错误，而不是警告**。 

### 4.2 规则定义

Gradle Lint插件允许我们配置两组规则：违规规则(通过rules和criticalRules选项)和忽略规则(使用excludedRules、ignore和fixme属性)，让我们来探索一下。

首先，rules属性接受一个Lint规则数组。**当满足规则定义的违规行为时，linter会发出警告**。但是，如果Gradle Lint插件单独运行，这些警告会导致构建失败。

接下来，我们来探索一下criticalRules选项。**如果我们希望规则违规触发任务失败，我们会向gradleLint.criticalRules属性添加一条规则**，该插件提供了一个特定的criticalLintGradle任务，仅用于lint关键规则：

```shell
./gradlew criticalLintGradle -PgradleLint.criticalRules=undeclared-dependency
```

这里，-PgradleLint.criticalRules选项将undeclared-dependency规则添加到criticalRules中。然后，criticalLintGradle任务仅对criticalRules中定义的规则进行lint操作。

继续，**excludedRules选项接受要忽略的规则列表**，当我们想要忽略组规则中的特定规则时，它会派上用场：

```groovy
gradleLint { 
    rules= ['all-dependency']
    excludedRules= ['undeclared-dependency']
 }
```

我们已经从之前的[all-dependency](https://github.com/nebula-plugins/gradle-lint-plugin/blob/main/src/main/resources/META-INF/lint-rules/all-dependency.properties)组规则中排除了未声明的依赖规则，[组规则](https://github.com/nebula-plugins/gradle-lint-plugin/wiki/Grouping-Rules)允许我们一次性应用一组规则。

最后，我们可以通过将构建文件的部分内容包装在ignore属性中来阻止linter扫描它们：

```groovy
dependencies {
    testImplementation('junit:junit:4.13.1')
    gradleLint.ignore('dependency-parentheses') {
        implementation("software.amazon.awssdk:sts")
    }
}
```

我们特意保护了[aws-sts](https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-sts)免受依赖括号规则的影响，运行linter时，它不会被列出：

```shell
warning   dependency-parentheses             parentheses are unnecessary for dependencies
gradle-lint-intro/build.gradle:11
testImplementation('junit:junit:4.13.1')
```

dependency-parentheses警告不要在Gradle依赖声明中不必要地使用括号。

此外，如果我们想暂时忽略某条规则，我们可以将ignore属性替换为fixme属性：

```groovy
gradleLint.fixme('2029-04-23', 'dependency-parentheses') {
    implementation('software.amazon.awssdk:sts')
}
```

在这里，[aws-sts](https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-sts)依赖将在2029年4月23日之后触发未使用的依赖违规。

**需要注意的是，[ignore和fixme属性仅对已解析的依赖有效，它们对构建文件中包装的声明无效](https://github.com/nebula-plugins/gradle-lint-plugin/issues/349#issuecomment-925097474)**：

```groovy
dependencies {
    gradleLint.ignore('unused-dependency') {
        implementation "software.amazon.awssdk:sts"
    }
}
```

我们特意保护了[aws-sts](https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-sts)免受未使用依赖规则的影响，我们不会直接在代码中使用它。但出于某种原因，AWS [WebIdentityTokenCredentialsProvider](https://stackoverflow.com/questions/67137283/does-webidentitytokencredentialsprovider-need-sts-module)需要它。不幸的是，linter无法捕获该规则，因为它不会评估未解析的依赖。

### 4.3 覆盖配置属性

Gradle Lint允许我们在命令行中覆盖某些配置属性，我们可以通过使用-PgradleLint选项以及任何有效的文件配置选项来实现。

我们将[unused-dependency](https://www.baeldung.com/gradle-finding-unused-dependencies)规则添加到我们初始的空rules数组中：

```shell
gradleLint { 
    rules= ['unused-dependency'] 
}
```

接下来，让我们通过命令行排除它，以防止linter应用它：

```shell
./gradlew lintGradle -PgradleLint.excludedRules=unused-dependency
```

这相当于在配置文件中设置excludedRules=[“unused-dependency”\]。

我们可以将此模式用于所有其他属性，尤其是在[CI环境](https://www.baeldung.com/cs/continuous-integration-deployment-delivery)中运行时，多个值应以逗号分隔。

## 5. 内置规则

**Gradle Lint插件有几个[内置的lint规则](https://github.com/nebula-plugins/gradle-lint-plugin/tree/main/src/main/resources/META-INF/lint-rules)**，让我们来探索其中的一些。

### 5.1 最小依赖版本规则

首先，我们来看一下[minimum-dependency-version](https://github.com/nebula-plugins/gradle-lint-plugin/wiki/Minimum-Dependency-Version-Rule)规则，该规则会显示依赖的版本严格低于预定义版本的情况，从而导致违规：

```shell
gradleLint { 
    rules= ['minimum-dependency-version'] 
}
```

我们需要在minVersions属性中提供以逗号分隔的最低依赖版本列表，但是，截至撰写本文时，[GradleLintExtension](https://github.com/nebula-plugins/gradle-lint-plugin/blob/280804cbbdcce703284921b34fe8719a0a84f661/src/main/groovy/com/netflix/nebula/lint/plugin/GradleLintExtension.groovy#L23)还没有minVersions属性，因此，我们可以通过命令行来设置它：

```shell
./gradlew lintGradle -PgradleLint.minVersions=junit:junit:5.0.0
```

这里，linter警告不要使用低于5.0.0的[junit](https://mvnrepository.com/artifact/junit/junit)版本：

```shell
> Task :lintGradle FAILED

This project contains lint violations. A complete listing of the violations follows.
Because none were serious, the build's overall status was unaffected.

warning   minimum-dependency-version         junit:junit is below the minimum version of 5.0.0 (no auto-fix available). See https://github.com/nebula-plugins/gradle-lint-plugin/wiki/Minimum-Dependency-Version-Rule for more details

? 1 problem (0 errors, 1 warning)

To apply fixes automatically, run fixGradleLint, review, and commit the changes.
```

运行./gradlew fixGradleLint可有效将[junit](https://mvnrepository.com/artifact/junit/junit)版本更新至5.0.0。

### 5.2 未声明依赖规则

接下来，我们将学习[undeclared-dependency](https://github.com/nebula-plugins/gradle-lint-plugin/blob/main/src/main/groovy/com/netflix/nebula/lint/rule/dependency/UndeclaredDependencyRule.groovy)规则，它确保我们在代码中直接使用显式声明的[传递依赖](https://www.baeldung.com/maven-dependency-scopes#transitive-dependency)：

```shell
gradleLint { 
    rules= ['undeclared-dependency'] 
}
```

现在，让我们执行lintGradle任务：

```shell
> Task :lintGradle FAILED

This project contains lint violations. A complete listing of the violations follows. 
Because none were serious, the build's overall status was unaffected.

warning   undeclared-dependency              one or more classes in org.hamcrest:hamcrest-core:1.3 are required by your code directly

warning   undeclared-dependency              one or more classes in org.apache.httpcomponents:httpcore:4.4.13 are required by your code directly

? 2 problems (0 errors, 2 warnings)

To apply fixes automatically, run fixGradleLint, review, and commit the changes.
```

我们可以看到我们的代码直接需要两个依赖，运行fixlintGradle任务后，linter会添加它们：

```shell
> Task :fixLintGradle

This project contains lint violations. A complete listing of my attempt to fix them follows. Please review and commit the changes.

fixed          undeclared-dependency              one or more classes in org.hamcrest:hamcrest-core:1.3 are required by your code directly

fixed          undeclared-dependency              one or more classes in org.apache.httpcomponents:httpcore:4.4.13 are required by your code directly

Corrected 2 lint problems
```

此外，截至撰写本文时，还[无法将自定义规则提供给插件](https://github.com/nebula-plugins/gradle-lint-plugin/issues/371)，另一种方法是分叉库并直接添加自定义规则。

最后，使用过时的规则时应格外小心，[archaic-wrapper规则](https://github.com/nebula-plugins/gradle-lint-plugin/pull/217)已于2018年被移除，文档中的某些部分尚未更新。

## 6. 生成报告

现在，让我们使用generateGradleLintReport任务生成报告：

```shell
./gradlew generateGradleLintReport
```

默认情况下，报告格式为HTML，插件将所有报告文件放在build/reports/gradleLint文件夹中。我们的HTML报告将命名为gradle-5.html，其中gradle-5是我们的根项目名称。

其他可用选项是xml和text：

```shell
gradleLint {
    reportFormat = 'text'
    reportOnlyFixableViolations = true
}
```

我们使用了reportFormat属性，直接以纯文本形式输出报告。此外，由于已激活reportOnlyFixableViolations标志，我们的报告仅包含可修复的违规行为。

## 7. 总结

在本文中，我们探索了[gradle-lint](https://github.com/nebula-plugins/gradle-lint-plugin)插件。首先，我们了解了它的实用功能。然后，我们列出了不同的配置选项。接下来，我们使用了一些预定义规则。最后，我们生成了lint报告。