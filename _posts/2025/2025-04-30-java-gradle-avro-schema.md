---
layout: post
title:  使用Gradle从Avro模式生成Java类
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本教程中，我们将学习如何从[Apache Avro](https://www.baeldung.com/java-apache-avro)模式生成Java类。

首先，我们将熟悉两种方法：使用现有的[Gradle插件](https://github.com/davidmc24/gradle-avro-plugin)和为构建脚本实现自定义任务。然后，我们将分析每种方法的优缺点，并了解它们最适合哪些场景。

## 2. Apache Avro入门

我们主要关注如何从Apache Avro模式生成Java类。在深入探讨代码生成的复杂性之前，我们先简单回顾一下基本概念。

### 2.1 Apache Avro模式定义

首先，让我们准备处理Avro格式所需的依赖，**我们需要[apache.avro模块](https://mvnrepository.com/artifact/org.apache.avro/avro/)来进行数据序列化和反序列化，因此我们将它添加到libs.version.toml和build.gradle文件中**：

```groovy
# libs.versions.toml

[versions]
// project dependencies versions
avro = "1.11.0"

[libraries]
// project libratirs
avro = {module = "org.apache.avro:avro", version.ref = "avro"}
```

```groovy
# build.gradle

dependencies {
    implementation libs.avro
    // project dependencies
}
```

下一步是定义Avro模式，为了演示，我们准备两个模式，分别用于本教程中使用的每个方法：

- /src/main/avro/user.avsc：用于Gradle插件方法
- /src/main/custom/pet.avsc：用于自定义Gradle任务方法

我们将模式放在单独的文件夹中，以维护正确的文件夹结构，**这也有助于避免出现ClassAlreadyExists异常，并确保Gradle构建系统能够正确识别和处理我们的Avro模式定义**。

上面的文件夹结构也会影响模式定义，User模式属于avro命名空间：

```json
{
    "type": "record",
    "name": "User",
    "namespace": "avro",
    "fields": [
      {
        "name": "firstName",
        "type": "string"
      },
      {
        "name": "lastName",
        "type": "string"
      },
      {
        "name": "phoneNumber",
        "type": "string"
      }
    ]
}
```

类似地，让我们在自定义命名空间下定义一个Pet模式：

```json
{
    "type": "record",
    "name": "Pet",
    "namespace": "custom",
    "fields": [
        {
            "name": "petId",
            "type": "string"
        },
        {
            "name": "name",
            "type": "string"
        },
        {
            "name": "species",
            "type": "string"
        },
        {
            "name": "age",
            "type": "int"
        }
    ]
}
```

选择合适的命名空间对于防止Java类生成过程中的命名冲突至关重要，因此，我们将遵循广为接受的做法，利用文件夹层次结构来确定命名空间标识符。

## 3. Java类生成

现在我们已经定义了模式，是时候编译它们了。

### 3.1 在命令行中使用Avro-Tools

**开箱即用，[Apache Avro Framework](https://avro.apache.org/docs/)提供了avro-tools jar等工具来生成代码**：

```shell
java -jar /path/to/avro-tools-1.11.1.jar compile schema <schema file> <destination>
```

然而，虽然了解avro-tools的功能可以为我们提供自定义解决方案的基础，**但这种方法对于大多数实际场景来说并不方便，因为主要要求是在构建脚本执行期间生成代码**。

### 3.2 使用开源Avro Gradle插件

将代码生成集成到我们的构建中的可能解决方案之一是使用[davidmc24的开源avro-gradle-plugin](https://github.com/davidmc24/gradle-avro-plugin)。

我们只需要导入依赖并通过包含插件ID来扩展build.gradle文件，让我们使用[官方发布页面](https://github.com/davidmc24/gradle-avro-plugin/releases)中的最新版本：

```groovy
# libs.versions.toml

[plugins]
avro = { id = "com.github.davidmc24.gradle.plugin.avro", version = "1.9.1" }
```

```groovy
# build.gradle

plugins {
    id 'java'
    alias libs.plugins.avro
}
```

此后，该库即可使用。

该插件默认使用/src/main/avro目录作为源，并将生成的类存储在/build/generated-main-avro-java中，我们可以通过覆盖GenerateAvroJavaTask来自定义此行为：

```groovy
def generateAvro = tasks.register("generateAvro", GenerateAvroJavaTask) {
    source("src/<custom>")
    outputDir = file("dest/avro")
}
```

乍一看，这种方法似乎非常灵活且易于使用，**但是，该项目已被归档。因此，它可能不适用于商业用途，因为该库不太可能进一步更新**。对于此类用例，最好利用Apache Avro工具库的功能实现自定义Gradle任务。

### 3.3 实现自定义Gradle任务

自定义Gradle代码生成任务的理念在于利用Apache Avro框架和avro-tools jar提供的强大机制，为此，我们需要相应地更新libs.versions.toml文件：

```toml
# libs.versions.toml

[versions]
avro = "1.11.0"

[libraries]
avro = {module = "org.apache.avro:avro", version.ref = "avro"}
avro-tools = {module = "org.apache.avro:avro-tools", version.ref = "avro"}
```

**Avro和Avro-tools库的版本应该相同，以防止发生冲突**。

此外，我们需要更新构建脚本，将avro-tools jar添加到类路径。构建过程的时间至关重要，通常，构建脚本按顺序执行，按照脚本中指定的顺序解析依赖并执行任务。

在Avro模式代码生成上下文中，负责此任务的自定义Gradle任务需要在构建过程的早期访问Avro-tools库，即在加载常规依赖之前：

```groovy
# build.gradle

buildscript {
    dependencies {
        classpath libs.avro.tools
    }
}

def avroSchemasDir = "src/main/custom"
def avroCodeGenerationDir = "build/generated-main-avro-custom-java"

// Add the generated Avro Java code to the Gradle source files.
sourceSets.main.java.srcDirs += [avroCodeGenerationDir]
```

在此步骤中，我们还可以定义源目录和输出目录，并将它们添加到sourceSets以确保Gradle脚本可以访问它们。

驱动我们自定义Gradle任务的主引擎是SpecificCompilerTool，此类是Avro代码生成过程的核心，提供类似于执行我们之前看到的命令的功能：

```shell
java -jar /path/to/avro-tools-1.11.1.jar compile schema <schema file> <destination> [..args]
```

我们可以自定义编码和字段可见性等参数。官方文档提供了有关[SpecificCompilerTool](https://avro.apache.org/docs/1.11.1/api/java/org/apache/avro/tool/SpecificCompilerTool.html)的更多信息：

```groovy
tasks.register('customAvroCodeGeneration') {
    // Define the task inputs and outputs for the Gradle up-to-date checks.
    inputs.dir(avroSchemasDir)
    outputs.dir(avroCodeGenerationDir)
    // The Avro code generation logs to the standard streams. Redirect the standard streams to the Gradle log.
    logging.captureStandardOutput(LogLevel.INFO);
    logging.captureStandardError(LogLevel.ERROR)
    doLast {
        new SpecificCompilerTool().run(System.in, System.out, System.err, List.of(
                "-encoding", "UTF-8",
                "-string",
                "-fieldVisibility", "private",
                "-noSetters",
                "schema", "$projectDir/$avroSchemasDir".toString(), "$projectDir/$avroCodeGenerationDir".toString()
        ))
    }
}
```

最后，为了将代码生成包含在构建流程中，让我们添加对customAvroCodeGeneration的依赖：

```groovy
tasks.withType(JavaCompile).configureEach {
    // Make Java compilation tasks depend on the Avro code generation task.
    dependsOn('customAvroCodeGeneration')
}
```

因此，每当调用构建命令时，我们都会触发Avro代码生成作业。

## 4. 总结

总结本文，我们熟悉了从Avro模式生成Java代码的两种方法。

第一种方法利用开源的avro-gradle-plugin，它提供了灵活性，并可以无缝集成到Gradle项目中。**但是，由于它已被归档，因此其商业用途的适用性可能受到限制**。

第二种方法涉及实现一个自定义的Gradle任务来扩展avro-tools库，**这种方法的优势在于，它引入的依赖关系极少，仅限于Apache Avro框架固有的依赖，此策略有助于最大限度地降低因使用不兼容的库版本而导致潜在冲突的风险**。此外，Gradle任务可以控制生成流程，在编译为Java类之前可能需要进行额外检查的用例中可能会有所帮助。例如，在构建管道中添加自定义验证等，这种方法提供了可靠性和稳定性，使其非常适合需要关键依赖管理的生产环境。