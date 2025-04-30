---
layout: post
title:  在Gradle中排除传递依赖
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1.概述

[Gradle](https://www.baeldung.com/gradle)是一个构建自动化工具，用于管理和自动化构建、测试和部署应用程序的过程。

使用基于[Groovy](https://www.baeldung.com/groovy-language)或[Kotlin](https://www.baeldung.com/kotlin/kotlin-overview)的领域特定语言(DSL)定义构建任务，可以轻松自动定义和管理项目中所需的库依赖。

在本教程中，我们将具体讨论在Gradle中排除传递依赖的几种方法。

## 2. 什么是传递依赖？

假设我们使用一个依赖于另一个库B的库A，那么**B被称为A的传递依赖**。默认情况下，**当我们包含A时，Gradle会自动将B添加到项目的类路径中**，这样即使我们没有明确地将B添加为依赖，B中的代码也可以在我们的项目中使用。

为了更清楚地说明，让我们使用一个真实的例子，我们在项目中定义[Google Guava](https://www.baeldung.com/guava-guide)：

```groovy
dependencies {
    // ...
    implementation 'com.google.guava:guava:31.1-jre'
}
```

如果Google Guava与其他库有依赖关系，那么Gradle将自动包含这些其他库。

要查看我们在项目中使用的依赖，我们可以打印它们：

```shell
./gradlew <module-name>:dependencies
```

在这种情况下，我们使用一个名为cluding-transitive-dependencies的模块：

```shell
./gradlew excluding-transitive-dependencies:dependencies
```

让我们看看输出：

```text
testRuntimeClasspath - Runtime classpath of source set 'test'.
\--- com.google.guava:guava:31.1-jre
     +--- com.google.guava:failureaccess:1.0.1
     +--- com.google.guava:listenablefuture:9999.0-empty-to-avoid-conflict-with-guava
     +--- com.google.code.findbugs:jsr305:3.0.2
     +--- org.checkerframework:checker-qual:3.12.0
     +--- com.google.errorprone:error_prone_annotations:2.11.0
     \--- com.google.j2objc:j2objc-annotations:1.3
```

我们可以看到一些我们没有明确定义的库，但是Gradle自动添加了它们，因为Google Guava需要它们。

然而，有时我们可能有充分的理由排除传递依赖关系。

## 3. 为什么要排除传递依赖？

让我们回顾一下为什么我们可能想要排除传递依赖关系的几个充分理由：

- **避免安全问题**：例如，Firestore Firebase SDK 24.4.0或Dagger 2.44对Google Guava 31.1-jre具有传递依赖性，而后者存在[安全漏洞问题](https://devhub.checkmarx.com/cve-details/CVE-2023-2976/)。
- **避免不必要的依赖**：某些库可能会带来与我们的应用无关的依赖，但是，我们应该谨慎考虑-例如，当我们需要完全排除传递依赖时，无论版本号如何。
- **缩减应用大小**：通过排除未使用的传递依赖，我们可以减少打包到应用中的库数量，从而减小输出文件(JAR、WAR、APK)的大小。我们还可以使用[ProGuard](https://www.baeldung.com/cs/code-obfuscation-benefits#:~:text=the%20following%20tools%3A-,Proguard,-%2C%20an%20open%2Dsource)等工具，通过移除未使用的代码、优化字节码、混淆类名和方法名以及移除不必要的资源来显著缩减应用大小。此过程可以在不牺牲功能的情况下，使应用更小、更快、更高效。

因此，Gradle还提供了排除依赖的机制。

### 3.1 解决版本冲突

**我们不建议排除传递依赖来解决版本冲突，因为[Gradle](https://docs.gradle.org/current/userguide/dependency_resolution.html#sec:conflict-resolution)已经有很好的机制来处理这个问题**。

当存在两个或多个相同的依赖时，Gradle只会选择一个，如果它们的版本不同，则[默认情况下会选择最新版本](https://docs.gradle.org/current/userguide/dependency_resolution.html#sec:conflict-resolution:~:text=Gradle%20will%20consider%20all%20requested%20versions%2C%20wherever%20they%20appear%20in%20the%20dependency%20graph.%20By%20default%2C%20it%20will%20select%20the%20highest%20version.%20More%20information%20on%20version%20ordering%20here.)。如果我们仔细观察，就会在日志中看到这种行为：

```text
+--- org.hibernate.orm:hibernate-core:7.0.0.Beta1
|    +--- jakarta.persistence:jakarta.persistence-api:3.2.0-M2
|    +--- jakarta.transaction:jakarta.transaction-api:2.0.1
|    +--- org.jboss.logging:jboss-logging:3.5.0.Final <-------------------+ same version
|    +--- org.hibernate.models:hibernate-models:0.8.6                     |
|    |    +--- io.smallrye:jandex:3.1.2 -> 3.2.0    <------------------+  |
|    |    \--- org.jboss.logging:jboss-logging:3.5.0.Final +-----------|--+
|    +--- io.smallrye:jandex:3.2.0     +-------------------------------+ latest version
|    +--- com.fasterxml:classmate:1.5.1
|    |    \--- jakarta.activation:jakarta.activation-api:2.1.0 -> 2.1.1 <---+
|    +--- org.glassfish.jaxb:jaxb-runtime:4.0.2                             |
|    |    \--- org.glassfish.jaxb:jaxb-core:4.0.2                           |
|    |         +--- jakarta.xml.bind:jakarta.xml.bind-api:4.0.0 (*)         |
|    |         +--- jakarta.activation:jakarta.activation-api:2.1.1   +-----+ latest version
|    |         +--- org.eclipse.angus:angus-activation:2.0.0                
```

我们可以看到，一些已识别的依赖是相同的。例如，org.jboss.logging:jboss-logging:3.5.0.Final出现了两次，但由于它们是同一版本，因此Gradle只会包含一份。

同时，对于jakarta.activation:jakarta.activation-api，会发现两个版本2.1.0和 2.1.1，Gradle会选择最新版本，即2.1.1。对于io.smallrye:jandex，也会选择3.2.0。

**但有时我们不想使用最新版本，我们可以强制Gradle选择我们严格需要的版本**：

```groovy
implementation("io.smallrye:jandex") {
    version {
        strictly '3.1.2'
    }
}
```

在这里，即使找到另一个甚至更新的版本，Gradle仍然会选择版本3.1.2。

我们可以声明具有特定版本或[版本范围](https://docs.gradle.org/current/userguide/rich_versions.html)的依赖来定义我们的项目可以使用的依赖的可接受版本。

## 4. 排除传递依赖

我们可以在各种情况下排除传递依赖，为了更清晰易懂，我们将使用一些我们可能熟悉的库的真实示例。

### 4.1 排除组

当我们定义依赖时，例如Google Guava，如果我们查看，依赖具有如下格式：

```text
com.google.guava : guava : 31.1-jre
----------------   -----   --------
        ^            ^        ^
        |            |        |
      group        module   version
```

如果我们查看第2部分的输出，我们会看到Google Guava依赖的五个模块。它们是com.google.code.findbugs、com.google.errorprone、com.google.guava、com.google.j2objc和org.checkerframework。

我们将排除com.google.guava组，其中包含guava、failaccess和listenablefuture模块：

```groovy
dependencies {
    // ...
    implementation ('com.google.guava:guava:31.1-jre') {
        exclude group: 'com.google.guava'
    }
}
```

这将排除com.google.guava组中除guava之外的所有模块，因为它是一个主模块。

### 4.2 排除特定模块

要排除特定的模块依赖，我们可以使用目标路径。例如，当我们使用[Hibernate](https://www.baeldung.com/learn-jpa-hibernate)库时，我们只需要排除org.glassfish.jaxb:txw2模块：

```groovy
dependencies {
    // ...
    implementation ('org.hibernate.orm:hibernate-core:7.0.0.Beta1') {
        exclude group: 'org.glassfish.jaxb', module : 'txw2'
    }
}
```

这意味着即使Hibernate依赖于txw2模块，我们也不会在项目中包含该模块。

### 4.3 排除多个模块

Gradle还允许我们在单个依赖语句中排除多个模块：

```groovy
dependencies {
    // ...
    testImplementation platform('org.junit:junit-bom:5.10.0')
    testImplementation ('org.junit.jupiter:junit-jupiter') {
        exclude group: 'org.junit.jupiter', module : 'junit-jupiter-api'
        exclude group: 'org.junit.jupiter', module : 'junit-jupiter-params'
        exclude group: 'org.junit.jupiter', module : 'junit-jupiter-engine'
    }
}
```

在此示例中，我们从org.junit-jupiter依赖中排除了junit-jupiter-api、junit-jupiter-params和junit-jupiter-engine模块。

通过这种机制，我们可以对更多的模块排除情况做同样的事情：

```groovy
dependencies {
    // ...
    implementation('com.google.android.gms:play-services-mlkit-face-detection:17.1.0') {
        exclude group: 'androidx.annotation', module: 'annotation'
        exclude group: 'android.support.v4', module: 'core'
        exclude group: 'androidx.arch.core', module: 'core'
        exclude group: 'androidx.collection', module: 'collection'
        exclude group: 'androidx.coordinatorlayout', module: 'coordinatorlayout'
        exclude group: 'androidx.core', module: 'core'
        exclude group: 'androidx.viewpager', module: 'viewpager'
        exclude group: 'androidx.print', module: 'print'
        exclude group: 'androidx.localbroadcastmanager', module: 'localbroadcastmanager'
        exclude group: 'androidx.loader', module: 'loader'
        exclude group: 'androidx.lifecycle', module: 'lifecycle-viewmodel'
        exclude group: 'androidx.lifecycle', module: 'lifecycle-livedata'
        exclude group: 'androidx.lifecycle', module: 'lifecycle-common'
        exclude group: 'androidx.fragment', module: 'fragment'
        exclude group: 'androidx.drawerlayout', module: 'drawerlayout'
        exclude group: 'androidx.legacy.content', module: 'legacy-support-core-utils'
        exclude group: 'androidx.cursoradapter', module: 'cursoradapter'
        exclude group: 'androidx.customview', module: 'customview'
        exclude group: 'androidx.documentfile.provider', module: 'documentfile'
        exclude group: 'androidx.interpolator', module: 'interpolator'
        exclude group: 'androidx.exifinterface', module: 'exifinterface'
    }
}
```

此示例从Google ML Kit依赖中排除了各种模块，以避免包含项目中默认已包含的某些模块。

### 4.4 排除所有传递模块

有时我们可能只需要使用主模块而不使用任何其他依赖，或者，我们可能需要明确指定所使用的每个依赖的版本。

transitive = false语句将告诉Gradle不要自动包含我们使用的库中的传递依赖：

```groovy
dependencies {
    // ...
    implementation('org.hibernate.orm:hibernate-core:7.0.0.Beta1') {
        transitive = false
    }
}
```

这意味着只有Hibernate Core本身将被添加到项目中，而无需任何其他依赖。

### 4.5 从每个配置中排除

除了在依赖声明中排除传递依赖之外，我们还可以在配置级别这样做。

我们可以使用[configuration.configureEach{}](https://docs.gradle.org/current/javadoc/org/gradle/api/DomainObjectCollection.html#all(org.gradle.api.Action):~:text=to%20be%20executed-,configureEach,-void%C2%A0configureEach%E2%80%8B)，它使用给定的操作配置集合中的每个元素。

此方法在Gradle 4.9及更高版本中可用，作为[all()](https://docs.gradle.org/current/javadoc/org/gradle/api/DomainObjectCollection.html#all(org.gradle.api.Action):~:text=to%20be%20executed-,configureEach,-void%C2%A0configureEach%E2%80%8B)的推荐替代方案。

尝试一下：

```groovy
dependencies { 
    // ...
    testImplementation 'org.mockito:mockito-core:3.+'
}

configurations.configureEach {
    exclude group: 'net.bytebuddy', module: 'byte-buddy-agent'
}
```

这意味着我们将net.bytebuddy组的byte-buddy-agent模块从使用该依赖的所有配置中排除。

### 4.6 在特定配置中排除

有时我们需要通过指定特定配置来排除依赖，当然，Gradle也允许这样做：

```groovy
configurations.testImplementation {
    exclude group: 'org.junit.jupiter', module : 'junit-jupiter-engine'
}

configurations.testCompileClasspath {
    exclude group : 'com.google.j2objc', module : 'j2objc-annotations'
}

configurations.annotationProcessor {
    exclude group: 'com.google.guava'
}
```

显然，Gradle允许以这种方式排除依赖。我们可以在Classpath中使用exclude来排除特定的配置，例如testImplementation、testCompileClasspath、annotationProcessor等等。

## 5. 总结

排除传递依赖主要有三个原因：避免安全问题、避免我们不需要的库以及减少应用程序大小。

在本文中，我们讨论了在Gradle中排除传递依赖的方法，从按组、特定模块或多个模块排除，到排除所有传递依赖。我们也可以在配置级别进行排除，但是，排除操作必须经过深思熟虑。