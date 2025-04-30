---
layout: post
title:  Gradle源集
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

[源集](https://www.baeldung.com/gradle)为我们提供了一种在Gradle项目中构建源代码的强大方法。

在本快速教程中，我们将了解如何使用它们。

## 2. 默认源集

在介绍默认设置之前，我们先来解释一下什么是源集。顾名思义，**源集代表源文件的逻辑分组**。

我们将介绍Java项目的配置，但这些概念也适用于其他Gradle项目类型。

### 2.1 默认项目布局

让我们从一个简单的项目结构开始：

```text
source-sets 
  ├── src 
  │    ├── main 
  │    │    └── java 
  │    │        ├── SourceSetsMain.java
  │    │        └── SourceSetsObject.java
  │    └── test 
  │         └── java 
  │             └── SourceSetsTest.java
  └── build.gradle 
```

现在让我们看一下build.gradle：

```groovy
apply plugin : "java"
description = "Source Sets example"
test {
    testLogging {
        events "passed", "skipped", "failed"
    }
}
dependencies {   
    implementation('org.apache.httpcomponents:httpclient:4.5.12')
    testImplementation('junit:junit:4.12')
}
```

**Java插件假定src/main/java和src/test/java作为默认源目录**。 

让我们制定一个简单的实用任务：

```groovy
task printSourceSetInformation(){
    doLast{
        sourceSets.each { srcSet ->
            println "["+srcSet.name+"]"
            print "-->Source directories: "+srcSet.allJava.srcDirs+"\n"
            print "-->Output directories: "+srcSet.output.classesDirs.files+"\n"
            println ""
        }
    }
}
```

我们这里只打印了一些源集属性，你可以随时查看完整的[JavaDoc](https://docs.gradle.org/current/javadoc/org/gradle/api/tasks/SourceSet.html)以获取更多信息。

让我们运行它并看看得到了什么：

```shell
$ ./gradlew printSourceSetInformation

> Task :source-sets:printSourceSetInformation
[main]
-->Source directories: [.../source-sets/src/main/java]
-->Output directories: [.../source-sets/build/classes/java/main]

[test]
-->Source directories: [.../source-sets/src/test/java]
-->Output directories: [.../source-sets/build/classes/java/test]
```

请注意，**我们有两个默认源集：main和test**。

### 2.2 默认配置

**Java插件还会自动为我们创建一些默认的Gradle[配置](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.Configuration.html)**。

它们遵循特殊的命名约定：<sourceSetName\><configurationName\>。

我们使用它们在build.gradle中声明依赖：

```groovy
dependencies { 
    implementation('org.apache.httpcomponents:httpclient:4.5.12') 
    testImplementation('junit:junit:4.12') 
}
```

请注意，我们指定的是implementation，而不是mainImplementation，这是命名约定的一个例外。

**默认情况下，testImplementation配置扩展了实现并继承了其所有依赖和输出**。

让我们改进辅助任务并看看这是关于什么的：

```groovy
task printSourceSetInformation(){

    doLast{
        sourceSets.each { srcSet ->
            println "["+srcSet.name+"]"
            print "-->Source directories: "+srcSet.allJava.srcDirs+"\n"
            print "-->Output directories: "+srcSet.output.classesDirs.files+"\n"
            print "-->Compile classpath:\n"
            srcSet.compileClasspath.files.each { 
                print "  "+it.path+"\n"
            }
            println ""
        }
    }
}
```

让我们看一下输出：

```text
[main]
// same output as before
-->Compile classpath:
  .../httpclient-4.5.12.jar
  .../httpcore-4.4.13.jar
  .../commons-logging-1.2.jar
  .../commons-codec-1.11.jar

[test]
// same output as before
-->Compile classpath:
  .../source-sets/build/classes/java/main
  .../source-sets/build/resources/main
  .../httpclient-4.5.12.jar
  .../junit-4.12.jar
  .../httpcore-4.4.13.jar
  .../commons-logging-1.2.jar
  .../commons-codec-1.11.jar
  .../hamcrest-core-1.3.jar
```

测试源集在其编译类路径中包含main的输出，还包括其依赖。

接下来，让我们创建单元测试：

```java
public class SourceSetsTest {

    @Test
    public void whenRun_ThenSuccess() {
        SourceSetsObject underTest = new SourceSetsObject("lorem","ipsum");
        
        assertThat(underTest.getUser(), is("lorem"));
        assertThat(underTest.getPassword(), is("ipsum"));
    }
}
```

这里我们测试一个存储两个值的简单POJO，我们可以直接使用它，因为**main输出位于我们的test类路径中**。

接下来，让我们从Gradle运行它：

```text
./gradlew clean test

> Task :source-sets:test

cn.tuyucheng.taketoday.test.SourceSetsTest > whenRunThenSuccess PASSED
```

## 3. 自定义源集

到目前为止，我们已经了解了一些合理的默认值。但是，在实践中，我们经常需要自定义源集，尤其是在集成测试中。

这是因为我们可能希望只在集成测试类路径上包含特定的测试库，我们也可能希望独立于单元测试执行它们。

### 3.1 定义自定义源集

让我们为集成测试创建一个单独的源目录：

```text
source-sets 
  ├── src 
  │    └── main 
  │         ├── java 
  │         │    ├── SourceSetsMain.java
  │         │    └── SourceSetsObject.java
  │         ├── test 
  │         │    └── SourceSetsTest.java
  │         └── itest 
  │              └── SourceSetsITest.java
  └── build.gradle 
```

接下来，**让我们使用sourceSets构造在build.gradle中对其进行配置**：

```groovy
sourceSets {
    itest {
        java {
        }
    }
}
dependencies {
    implementation('org.apache.httpcomponents:httpclient:4.5.12')
    testImplementation('junit:junit:4.12')
}
// other declarations omitted
```

注意，我们没有指定任何自定义目录，这是因为我们的文件夹与新源集(itest)的名称匹配。

我们**可以使用srcDirs属性自定义包含哪些目录**：

```groovy
sourceSets{
    itest {
        java {
            srcDirs("src/itest")
        }
    }
}
```

还记得我们一开始的辅助任务吗？让我们重新运行它，看看它打印了什么：

```shell
$ ./gradlew printSourceSetInformation

> Task :source-sets:printSourceSetInformation
[itest]
-->Source directories: [.../source-sets/src/itest/java]
-->Output directories: [.../source-sets/build/classes/java/itest]
-->Compile classpath:
  .../source-sets/build/classes/java/main
  .../source-sets/build/resources/main

[main]
 // same output as before

[test]
 // same output as before
```

### 3.2 分配源集特定的依赖

还记得默认配置吗？现在我们也获得了itest源集的一些配置。

**让我们使用itestImplementation来分配一个新的依赖**：


```groovy
dependencies {
    implementation('org.apache.httpcomponents:httpclient:4.5.12')
    testImplementation('junit:junit:4.12')
    itestImplementation('com.google.guava:guava:29.0-jre')
}
```

这仅适用于集成测试。

让我们修改之前的测试并将其添加为集成测试：

```java
public class SourceSetsItest {

    @Test
    public void givenImmutableList_whenRun_ThenSuccess() {
        SourceSetsObject underTest = new SourceSetsObject("lorem", "ipsum");
        List someStrings = ImmutableList.of("Tuyucheng", "is", "cool");

        assertThat(underTest.getUser(), is("lorem"));
        assertThat(underTest.getPassword(), is("ipsum"));
        assertThat(someStrings.size(), is(3));
    }
}
```

为了能够运行它，我们需要**定义一个使用编译输出的自定义测试任务**：

```groovy
// source sets declarations

// dependencies declarations 

task itest(type: Test) {
    description = "Run integration tests"
    group = "verification"
    testClassesDirs = sourceSets.itest.output.classesDirs
    classpath = sourceSets.itest.runtimeClasspath
}
```

**这些声明在[配置阶段](https://docs.gradle.org/current/userguide/build_lifecycle.html)进行评估**，因此，它们的顺序很重要。

例如，在声明之前我们不能在任务主体中引用itest源集。

让我们看看运行测试会发生什么：

```shell
$ ./gradlew clean itest

// some compilation issues

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':source-sets:compileItestJava'.
> Compilation failed; see the compiler error output for details.
```

与上次运行不同，这次我们遇到了编译错误，那么发生了什么？

这个新的源集创建了一个独立的配置。

换句话说，**itestImplementation不会继承JUnit依赖，也不会获取main的输出**。

让我们在Gradle配置中修复这个问题：

```groovy
sourceSets{
    itest {
        compileClasspath += sourceSets.main.output
        runtimeClasspath += sourceSets.main.output
        java {
        }
    }
}

// dependencies declaration
configurations {
    itestImplementation.extendsFrom(testImplementation)
    itestRuntimeOnly.extendsFrom(testRuntimeOnly)
}
```

现在让我们重新运行集成测试：

```shell
$ ./gradlew clean itest

> Task :source-sets:itest

cn.tuyucheng.taketoday.itest.SourceSetsItest > givenImmutableList_whenRun_ThenSuccess PASSED
```

测试通过。

### 3.3 Eclipse IDE处理

到目前为止，我们已经了解了如何直接在Gradle中处理源集。但是，大多数情况下，我们会使用IDE(例如Eclipse)。

当我们导入项目时，我们遇到一些编译问题：

![](/assets/images/2025/gradle/gradlesourcesets01.png)

但是，如果我们从Gradle运行集成测试，则不会出现任何错误：

```shell
$ ./gradlew clean itest

> Task :source-sets:itest

cn.tuyucheng.taketoday.itest.SourceSetsItest > givenImmutableList_whenRun_ThenSuccess PASSED
```

那么发生了什么？在这种情况下，guava依赖属于itestImplementation。

不幸的是，**Eclipse Buildship Gradle插件不能很好地处理这些自定义配置**。

让我们在build.gradle中修复这个问题：

```groovy
apply plugin: "eclipse"

// previous declarations

eclipse {
    classpath {
        plusConfigurations+=[configurations.itestCompileClasspath] 
    } 
}

```

我们将配置附加到Eclipse类路径，如果我们刷新项目，编译问题就消失了。

但是，这种方法有一个缺点：IDE不区分配置，这意味着我们可以轻松地在test源中导入guava(这是我们特别想避免的)。

## 4. 总结

在本教程中，我们介绍了Gradle源集的基础知识。

然后我们解释了自定义源集如何工作以及如何在Eclipse中使用它们。