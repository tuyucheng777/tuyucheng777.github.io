---
layout: post
title:  设置并使用Spock与Gradle
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

[Spock框架](https://docs.spockframework.org/)是一个针对Java和[Groovy](https://www.baeldung.com/groovy-language)应用程序的测试和规范框架，[Gradle](https://docs.gradle.org/current/userguide/userguide.html)是一个流行的构建工具，也是[Maven](https://www.baeldung.com/maven)的替代品。

在本教程中，我们将展示如何使用Gradle设置项目并添加Spock测试依赖，我们还将快速入门并逐步过渡到使用Gradle构建流程将Spock与Spring完全集成。

## 2. 使用Spock和Gradle

我们需要创建一个Gradle项目并添加Spock依赖。

### 2.1 设置Gradle项目

首先，我们在系统上[安装Gradle](https://gradle.org/install/)，然后可以使用gradle init命令[初始化](https://docs.gradle.org/current/userguide/part1_gradle_init.html#part1_begin)Gradle项目。例如，创建应用程序或使用[Java](https://docs.gradle.org/8.5/userguide/building_java_projects.html#sec:building_jvm_components)或Kotlin的库时，有不同的选项可供选择。

无论如何，Gradle项目总是从以下位置获取配置：

- **build.gradle**：它包含有关构建过程的信息，例如Java版本或用于实现或测试的库，我们将其称为构建文件。
- **settings.gradle**：它添加了项目相关的信息，例如项目名称或子模块结构，我们将其称为设置文件。

Gradle使用JVM插件实现项目的编译、测试和捆绑功能。

如果我们选择Java，我们将使用“[java](https://docs.gradle.org/8.5/userguide/java_plugin.html#java_plugin)”插件使其保持简单，因为这是它最终将被扩展的内容。

让我们检查一下Java 17项目的简单构建框架：

```groovy
plugins {
    id 'java'
}

repositories {
    mavenCentral()
}

dependencies {
    // test dependencies
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

test {
    useJUnitPlatform()

    testLogging {
        events "started", "passed", "skipped", "failed"
    }
}
```

该文件是核心组件，定义了构建项目所需的任务。

我们添加了[JUnit5](https://www.baeldung.com/junit-5)测试依赖：

```groovy
testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
```

我们需要在测试任务中使用useJUnitPlatform()规范来运行测试，我们还添加了一些用于测试日志记录的属性，以便在任务运行时获得输出：

```groovy
test {
    useJUnitPlatform()
    testLogging {
        events "started", "passed", "skipped", "failed"
    }
}
```

我们还看到了我们的项目如何使用例如mavenCentral()仓库下载依赖：

```groovy
repositories {
    mavenCentral()
}
```

值得注意的是，有些人可能会发现该配置比具有基于XML的pom.xml构建配置的Maven项目更具可读性。

最后，我们来看看设置文件：

```groovy
rootProject.name = 'spring-boot-testing-spock'
```

这非常简单，仅配置项目名称。但是，它可以包含相关信息，例如子模块包含或插件定义。

我们可以查看[Gradle DSL参考](https://docs.gradle.org/current/dsl/index.html)以获取有关构建或设置脚本的更多信息。

### 2.2 添加Spock依赖

我们需要两个简单的步骤将Spock添加到我们的Gradle项目中：

- 添加“groovy”插件
- 将Spock添加到测试依赖

让我们看一下构建文件：

```groovy
plugins {
    id 'java'
    id 'groovy'
}

repositories {
    mavenCentral()
}

dependencies {
    // Spock test dependencies
    testImplementation 'org.spockframework:spock-core:2.4-M1-groovy-4.0'
    // Junit dependencies
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

test {
    useJUnitPlatform()
    testLogging {
        events "started", "passed", "skipped", "failed"
    }
}
```

**我们必须通过添加[org.spockframework:spock-core](https://mvnrepository.com/artifact/org.spockframework/spock-core/)测试依赖来更新我们的依赖**：

```groovy
testImplementation 'org.spockframework:spock-core:2.4-M1-groovy-4.0'
```

值得注意的是，我们不需要像Maven项目那样配置[GMavenPlus插件](https://groovy.github.io/GMavenPlus/)。

### 2.3 运行测试

我们可以用Spock进行不同类型的[测试](https://www.baeldung.com/groovy-spock)，我们来看一个简单的测试用例：

```groovy
class SpockTest extends Specification {
    def "one plus one should equal two"() {
        expect:
        1 + 1 == 2
    }
}
```

每个测试都必须扩展Specification类，此外，测试使用Groovy def语法定义为函数。

如果我们习惯使用Java编程，**则需要记住，Spock测试默认位于不同的包中，并具有另一个类扩展名**。如果没有其他指定，我们必须将测试放在test/groovy文件夹中。此外，该类将具有.groovy扩展名，例如SpockTest.groovy。

要运行测试，我们需要使用IDE或在命令行执行测试任务：

```shell
gradle test
```

让我们检查一些示例输出：

```text
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :test

SpockTest > one plus one should equal two STARTED

SpockTest > one plus one should equal two PASSED

BUILD SUCCESSFUL in 15s
7 actionable tasks: 7 executed
```

Gradle使用缓存系统，只会重新运行自上次执行以来发生变化的测试。

## 3. 使用Spock、Gradle和Spring

我们可能想将Spock添加到[Spring](https://www.baeldung.com/spring-spock-testing)项目中，Spock有一个专门的[模块](https://spockframework.org/spock/docs/2.2-SNAPSHOT/modules.html#_spring_module)来实现这一点。

让我们先来看看使用基本Spring配置的Spock，稍后，我们还将介绍Spring Boot的设置。从现在开始，为了简洁起见，我们将省略构建文件中的java和test部分，因为它们不会改变。

### 3.1 Spock和Spring

假设我们有一个Spring项目，并且想要切换或采用Spock进行测试。

现在，构建文件中的依赖结构变得越来越复杂，因此让我们正确地注释掉每个部分：

```groovy
plugins {
    id 'java'
    id 'groovy'
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring implementation dependencies
    implementation 'org.springframework:spring-web:6.1.0'

    // Spring test dependencies
    testImplementation 'org.springframework:spring-test:6.1.0'

    // Spring Spock test dependencies
    testImplementation 'org.spockframework:spock-spring:2.4-M1-groovy-4.0'

    // Spock Core test dependencies
    testImplementation 'org.spockframework:spock-core:2.4-M1-groovy-4.0'

    // Junit Test dependencies
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```

我们添加了[org.springframework:spring-web](https://mvnrepository.com/artifact/org.springframework/spring-web/)来演示一个简单的Spring依赖，此外，如果我们想要使用Spring测试功能，则必须添加[org.springframework:spring-test](https://mvnrepository.com/artifact/org.springframework/spring-test/)测试依赖：

```groovy
// Spring implementation dependencies
implementation 'org.springframework:spring-web:6.1.0'

// Spring test dependencies
testImplementation 'org.springframework:spring-test:6.1.0'
```

**最后，让我们添加[org.spockframework:spock-spring](https://mvnrepository.com/artifact/org.spockframework/spock-spring/)依赖，这是集成Spock和Spring所需的唯一依赖**：

```groovy
// Spring Spock test dependencies
testImplementation 'org.spockframework:spock-spring:2.4-M1-groovy-4.0'
```

### 3.2 Spock和Spring Boot

我们只需要将之前的Spring的基本依赖替换成Spring Boot的基本依赖即可。

让我们看一下构建文件：

```groovy
plugins {
    id 'java'
    id 'groovy'
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring implementation dependencies
    implementation 'org.springframework.boot:spring-boot-starter-web:3.0.0'

    // Spring Test dependencies
    testImplementation 'org.springframework.boot:spring-boot-starter-test:3.0.0'

    // Spring Spock Test dependencies
    testImplementation 'org.spockframework:spock-spring:2.4-M1-groovy-4.0'

    // Spring Core Test dependencies
    testImplementation 'org.spockframework:spock-core:2.4-M1-groovy-4.0'

    // Junit Test dependencies
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```

值得注意的是，我们添加Spring Boot依赖：

```groovy
// Spring implementation dependencies
implementation 'org.springframework.boot:spring-boot-starter-web:3.0.0'

// Spring Test dependencies
testImplementation 'org.springframework.boot:spring-boot-starter-test:3.0.0'
```

### 3.3 Spring和Gradle依赖管理

**让我们使用[Spring依赖管理](https://docs.spring.io/dependency-management-plugin/docs/current/reference/html/)来完成设置，使我们的配置更加紧凑和易于维护**。这样，我们采用类似于[Maven](https://www.baeldung.com/spring-maven-bom)项目的BOM风格，并且仅在一个位置声明版本。

我们可以通过几种方法来实现这一点。

让我们看一下第一个选项：

```groovy
plugins {
    id 'java'
    id 'groovy'
    id "org.springframework.boot" version "3.0.0"
    id 'io.spring.dependency-management' version '1.0.14.RELEASE'
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring implementation dependencies
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // Spring Test dependencies
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    // Spring Spock Test dependencies
    testImplementation 'org.spockframework:spock-spring:2.4-M1-groovy-4.0'

    // Spring Core Test dependencies
    testImplementation 'org.spockframework:spock-core:2.4-M1-groovy-4.0'
}
```

在这种情况下，我们仅添加以下插件：

```groovy
id "org.springframework.boot" version "3.2.1"
id 'io.spring.dependency-management' version '1.0.14.RELEASE'
```

值得注意的是，JUnit5依赖现在由Spring Boot拉取，无需指定它们。

最后，如果我们想要更新Spring Boot版本，只需要在org.springframework.boot插件中进行替换即可。

让我们看看第二种选择：

```groovy
plugins {
    id 'java'
    id 'groovy'
    id 'io.spring.dependency-management' version '1.1.4'
}

repositories {
    mavenCentral()
}

dependencyManagement {
    imports {
        mavenBom 'org.springframework.boot:spring-boot-dependencies:3.2.1'
    }
}

dependencies {
    // Spring implementation dependencies
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // Test implementation
    testImplementation(
        'junit:junit',
        'org.spockframework:spock-core:2.4-M1-groovy-4.0',
        'org.spockframework:spock-spring:2.4-M1-groovy-4.0',
        'org.springframework.boot:spring-boot-starter-test',
    )
}

```

**现在，我们已经用dependencyManagement部分替换了org.springframework.boot插件，作为我们依赖的单一入口点**：

```groovy
dependencyManagement {
    imports {
        mavenBom 'org.springframework.boot:spring-boot-dependencies:3.2.1'
    }
}
```

值得注意的是，我们将testImplementation折叠在一个输入中，并添加JUnit4依赖，以防我们的项目仍然使用它或与JUnit5混合：

```groovy
// Test implementation
testImplementation(
    'junit:junit',
    'org.spockframework:spock-core:2.4-M1-groovy-4.0',
    'org.spockframework:spock-spring:2.4-M1-groovy-4.0',
    'org.springframework.boot:spring-boot-starter-test',
)
```

## 4. 总结

在本教程中，我们了解了如何使用Spock和Gradle搭建Java项目，并了解了如何添加Spring和Spring Boot依赖。Gradle提供了强大的构建支持，并简化了项目脚本的配置。同样，Spock也是一款优秀的测试工具，因为它易于设置，并且规范面向数据驱动测试和基于交互的测试，而非像JUnit那样采用断言式的测试方式。