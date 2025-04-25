---
layout: post
title:  配置项目以排除某些Sonar违规行为
category: staticanalysis
copyright: staticanalysis
excerpt: Sonar
---

## 1. 概述

在构建过程中，我们可以使用各种工具来报告源代码的质量，[SonarQube](https://www.baeldung.com/sonar-qube)就是这样一个工具，它可以执行静态代码分析。

有时我们可能不同意返回的结果，因此，我们可能希望**排除一些被SonarQube错误标记的代码**。

在本简短教程中，我们将了解如何禁用Sonar检查。虽然可以更改SonarQube服务器上的规则集，但我们仅关注如何在项目源代码和配置中控制单个检查。

## 2. 违规示例

让我们看一个例子：

```java
public void printStringToConsoleWithDate(String str) {
    System.out.println(LocalDateTime.now().toString() + " " + str);
}
```

默认情况下，由于违反了[java:S106规则](https://rules.sonarsource.com/java/RSPEC-106)，SonarQube将此代码报告为代码异味：

![](/assets/images/2025/staticanalysis/sonarexcludeviolations01.png)

但是，假设对于这个特定的类，我们决定使用[System.out进行日志记录](https://www.baeldung.com/java-system-out-println-vs-loggers)是有效的，也许这是一个轻量级的实用程序，它将在容器中运行，并且不需要整个日志库来记录到stdout。

需要注意的是，在SonarQube用户界面中，也可以将违规行为标记为误报。但是，如果在多台服务器上分析了代码，或者重构后该行代码移至另一个类，则违规行为将再次出现。

有时我们希望在源代码存储库中进行排除，以便它们能够持续存在。

那么，让我们看看如何通过配置项目从SonarQube报告中排除此代码。

## 3. 使用//NOSONAR

**我们可以通过在末尾添加//NOSONAR来禁用一行代码**：

```java
System.out.println(LocalDateTime.now()
    .toString() + " " + str); //NOSONAR lightweight logging
```

行尾的//NOSONAR标签会抑制所有可能引发的问题，此方法适用于[SonarQube支持的大多数语言](https://docs.sonarqube.org/latest/faq/)。

我们还可以在NOSONAR之后添加一些额外的评论，解释我们为什么禁用该检查。

让我们继续，看看Java中禁用检查的特定方法。

## 4. 使用@SuppressWarnings

### 4.1 注解代码

在Java中，**我们可以使用内置的[@SuppressWarnings](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/SuppressWarnings.html)注解排除Sonar检查**。

我们可以标注该函数：

```java
@SuppressWarnings("java:S106")
public void printStringToConsoleWithDate(String str) {
    System.out.println(LocalDateTime.now().toString() + " " + str);
}
```

这与抑制编译器警告的方式完全相同，我们所要做的就是指定规则标识符，在本例中为java:S106。

### 4.2 如何获取标识符

我们可以使用SonarQube用户界面获取规则标识符，查看违规行为时，可以点击“Why is this an issue?”：

![](/assets/images/2025/staticanalysis/sonarexcludeviolations02.png)

它向我们展示了定义，由此我们可以在右上角找到规则标识符：

![](/assets/images/2025/staticanalysis/sonarexcludeviolations03.png)

## 5. 使用sonar-project.properties

**我们还可以使用[分析属性](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)在sonar-project.properties文件中定义排除规则**。

让我们定义并将sonar-project.properties文件添加到我们的资源目录中：

```properties
sonar.issue.ignore.multicriteria=e1

sonar.issue.ignore.multicriteria.e1.ruleKey=java:S106
sonar.issue.ignore.multicriteria.e1.resourceKey=**/SonarExclude.java
```

我们刚刚声明了第一个[多条件规则](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)，名为e1。我们为SonarExclude类排除了java:S106规则，我们的定义可以**混合使用规则标识符和[文件匹配模式](https://docs.sonarqube.org/latest/project-administration/narrowing-the-focus/#header-1)进行排除**，分别在以e1名称标签开头的ruleKey和resourceKey属性中。

使用这种方法，我们可以构建一个复杂的配置，排除多个文件中的特定规则：

```properties
sonar.issue.ignore.multicriteria=e1,e2

# Console usage - ignore a single class
sonar.issue.ignore.multicriteria.e1.ruleKey=java:S106
sonar.issue.ignore.multicriteria.e1.resourceKey=**/SonarExclude.java
# Too many parameters - ignore the whole package
sonar.issue.ignore.multicriteria.e2.ruleKey=java:S107
sonar.issue.ignore.multicriteria.e2.resourceKey=cn/tuyucheng/taketoday/sonar/*.java
```

我们刚刚定义了multicriteria的一个子集，我们通过添加第二个定义来扩展配置，并将其命名为e2。然后，我们将这两个规则合并到一个子集中，并用逗号分隔名称。

## 6. 禁用Maven

所有分析属性也可以使用[Maven属性](https://maven.apache.org/pom.html#Properties)来应用，[Gradle](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_system_properties)中也提供类似的机制。

### 6.1 Maven中的多条件 

回到示例，让我们修改pom.xml：

```xml
<properties>
    <sonar.issue.ignore.multicriteria>e1</sonar.issue.ignore.multicriteria>
    <sonar.issue.ignore.multicriteria.e1.ruleKey>java:S106</sonar.issue.ignore.multicriteria.e1.ruleKey>
    <sonar.issue.ignore.multicriteria.e1.resourceKey>
        **/SonarExclude.java
    </sonar.issue.ignore.multicriteria.e1.resourceKey>
</properties>
```

此配置的工作方式与在sonar-project.properties文件中使用的完全相同。

### 6.2 缩小焦点

有时，分析的项目可能包含一些我们想要排除并[缩小](https://docs.sonarqube.org/latest/project-administration/narrowing-the-focus/)SonarQube检查范围的生成代码。

让我们通过在pom.xml中定义sonar.exclusions来排除我们的类：

```xml
<properties>
    <sonar.exclusions>**/SonarExclude.java</sonar.exclusions>
</properties>
```

在这种情况下，我们根据文件名排除了单个文件，除该文件之外的所有文件都将执行检查。

我们还可以使用文件匹配模式。让我们通过定义以下内容来排除整个包：

```xml
<properties>
    <sonar.exclusions>cn/tuyucheng/taketoday/sonar/*.java</sonar.exclusions>
</properties>
```

另一方面，通过使用sonar.inclusions属性，我们可以要求SonarQube仅分析项目文件的特定子集：

```xml
<properties>
    <sonar.inclusions>cn/tuyucheng/taketoday/sonar/*.java</sonar.inclusions>
</properties>
```

此代码片段仅定义对来自cn.tuyucheng.taketoday.sonar包的java文件的分析。

最后，我们还可以定义sonar.skip值：

```xml
<properties>
    <sonar.skip>true</sonar.skip>
</properties>
```

这会将整个Maven模块排除在SonarQube检查之外。

## 7. 总结

在本文中，我们讨论了抑制我们的代码上的某些SonarQube分析的不同方法。

我们首先从排除单行检查开始；然后，我们讨论了内置的@SuppressWarnings注解以及通过特定规则进行排除，这需要我们找到规则的标识符。

我们还研究了如何配置分析属性，我们尝试了多准则和sonar-project.properties文件。

最后，我们将属性移至pom.xml并审查了其他缩小焦点的方法。