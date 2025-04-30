---
layout: post
title:  如何在Gradle中配置条件依赖
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本教程中，我们将了解如何在Gradle项目中配置条件依赖。

## 2. 项目设置

我们将为演示建立一个多模块项目，首先访问[start.spring.io](https://start.spring.io/)并创建根项目conditional-dependency-demo，我们将使用Gradle、Java以及Spring Boot。

我们还添加两个提供者模块，provider1和provider2，以及两个消费者模块，consumer1和consumer2：

![](/assets/images/2025/gradle/gradleconditionaldependencies01.png)

## 3. 配置条件依赖

假设，根据项目属性，我们想要包含两个提供程序模块中的一个。对于consumer1模块，如果指定了属性isLocal，则希望包含provider1模块。否则，应该包含provider2模块。

为此，我们在consumer1模块的gradle.settings.kts文件中添加以下内容：

```groovy
plugins {
    id("java")
}

group = "cn.tuyucheng.taketoday.gradle"
version = "0.0.1-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter-api:5.7.0")
    testRuntimeOnly ("org.junit.jupiter:junit-jupiter-engine:5.7.0")

    if (project.hasProperty("isLocal")) {
        implementation("cn.tuyucheng.taketoday.gradle:provider1")
    } else {
        implementation("cn.tuyucheng.taketoday.gradle:provider2")
    }
}

tasks.getByName<Test>("test") {
    useJUnitPlatform()
}
```

现在，让我们运行dependencies任务来查看选择了哪个提供程序模块：

```shell
gradle -PisLocal dependencies --configuration implementation
```

```text
> Task :consumer1:dependencies

------------------------------------------------------------
Project ':consumer1'
------------------------------------------------------------

implementation - Implementation only dependencies for source set 'main'. (n)
\--- cn.tuyucheng.taketoday.gradle:provider1 (n)

(n) - Not resolved (configuration is not meant to be resolved)

A web-based, searchable dependency report is available by adding the --scan option.

BUILD SUCCESSFUL in 591ms
1 actionable task: 1 executed
```

可以看到，传递属性会导致provider1模块被引入。现在，让我们在不指定任何属性的情况下运行dependencies任务：

```shell
gradle dependencies --configuration implementation
```

```text
> Task :consumer1:dependencies

------------------------------------------------------------
Project ':consumer1'
------------------------------------------------------------

implementation - Implementation only dependencies for source set 'main'. (n)
\--- cn.tuyucheng.taketoday.gradle:provider2 (n)

(n) - Not resolved (configuration is not meant to be resolved)

A web-based, searchable dependency report is available by adding the --scan option.

BUILD SUCCESSFUL in 649ms
1 actionable task: 1 executed
```

我们可以看到，provider2现已被包含在内。

## 4. 通过模块替换配置条件依赖

让我们看看另一种通过依赖替换来有条件地配置依赖的方法，对于我们的consumer2模块，如果指定了isLocal属性，我们希望包含provider2模块。否则，应该使用模块provider1。

让我们将以下配置添加到我们的consumer2模块来实现这一目标：

```groovy
plugins {
    id("java")
}

group = "cn.tuyucheng.taketoday.gradle"
version = "0.0.1-SNAPSHOT"

repositories {
    mavenCentral()
}

configurations.all {
    resolutionStrategy.dependencySubstitution {
        if (project.hasProperty("isLocal"))
            substitute(project("cn.tuyucheng.taketoday.gradle:provider1"))
              .using(project(":provider2"))
              .because("Project property override(isLocal).")
    }
}

dependencies {
    implementation(project(":provider1"))

    testImplementation("org.junit.jupiter:junit-jupiter-api:5.7.0")
    testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine:5.7.0")
}

tasks.getByName<Test>("test") {
    useJUnitPlatform()
}
```

现在，如果我们再次运行相同的命令，应该会得到类似的结果，首先，我们指定isLocal属性来运行：

```shell
gradle -PisLocal dependencies --configuration compilePath
```

```text
> Task :consumer2:dependencies

------------------------------------------------------------
Project ':consumer2'
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
\--- project :provider1 -> project :provider2

A web-based, searchable dependency report is available by adding the --scan option.

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

果然，我们看到provider1项目被provider2项目替换了，现在让我们尝试一下不指定属性的情况：

```shell
gradle dependencies --configuration compilePath
```

```text
> Task :consumer2:dependencies

------------------------------------------------------------
Project ':consumer2'
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
\--- project :provider1

A web-based, searchable dependency report is available by adding the --scan option.

BUILD SUCCESSFUL in 623ms
1 actionable task: 1 executed
```

正如预期的那样，这次没有发生替换，并且包含了provider1。

## 5. 两种方法的区别

正如我们在上面的演示中所看到的，这两种方法都帮助我们实现了有条件地配置依赖的目标，让我们来谈谈这两种方法之间的一些区别。

首先，**与第二种方法相比，直接编写条件逻辑看起来更简单，配置更少**。

其次，**虽然第二种方法涉及更多配置，但它似乎更符合惯例**。在第二种方法中，我们利用了Gradle本身提供的替换机制，它还允许我们指定替换的原因。此外，我们可以在日志中注意到替换的发生，而第一种方法则没有此类信息：

```shell
compileClasspath - Compile classpath for source set 'main'. 
\--- project :provider1 -> project :provider2
```

我们还注意到，在第一种方法中，不需要依赖解析，我们可以通过以下方式获得结果：

```shell
gradle -PisLocal dependencies --configuration implementation
```

而在第二种方法中，如果我们检查实现配置，我们将看不到预期的结果，原因是它仅在依赖解析发生时才有效。因此，它可以通过compilePath配置来实现：

```shell
gradle -PisLocal dependencies --configuration compilePath
```

## 6. 总结

在本文中，我们了解了在Gradle中条件配置依赖的两种方法，并分析了两者之间的区别。