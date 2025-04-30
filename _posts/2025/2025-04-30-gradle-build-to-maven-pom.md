---
layout: post
title:  将Gradle构建文件转换为Maven POM
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

在本教程中，我们将了解如何将Gradle构建文件转换为Maven POM文件，我们将使用Gradle 7.2版本作为示例，并探索一些可用的自定义选项。

## 2. Gradle构建文件

让我们从标准Gradle Java项目gradle-to-maven开始，其中包含以下build.gradle文件：

```groovy
repositories {
    mavenCentral()
}

group = ''
version = '0.0.1'

apply plugin: 'java'

dependencies {
    implementation 'org.slf4j:slf4j-api:1.7.25'
    testImplementation 'junit:junit:4.12'
}
```

## 3. Maven插件

Gradle附带一个[Maven插件](https://docs.gradle.org/current/userguide/maven_plugin.html)，该插件支持将Gradle文件转换为Maven POM文件，它还可以将工件部署到Maven仓库。

要使用此功能，我们将Maven Publish插件添加到我们的build.gradle文件中：

```groovy
apply plugin: 'maven-publish'
```

该插件使用Gradle文件中的组和版本，并将它们添加到POM文件中。此外，它还会自动从目录名称中获取artifactId。

该插件还会自动添加发布任务，为了进行转换，我们需要在POM文件中添加一个发布任务的基本定义：

```groovy
publishing {
    publications {
        customLibrary(MavenPublication) {
            from components.java
        }
    }

    repositories {
        maven {
            name = 'sampleRepo'
            url = layout.buildDirectory.dir("repo")
        }
    }
}
```

现在我们可以将customLibrary发布到基于本地目录的仓库以用于演示目的：

```shell
gradle publish
```

运行上述命令将创建一个包含以下子目录的构建目录：

- libs：包含名为\${artifactId}-\${version}.jar的jar
- **publications/customLibrary：包含已转换的POM文件，文件名为pom-default.xml**
- tmp/jar：包含清单
- repo：用作包含已发布工件的仓库的文件系统位置

生成的POM文件将如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.tuyucheng.taketoday</groupId>
    <artifactId>gradle-to-maven</artifactId>
    <version>0.0.1</version>
    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
</project>
```

请注意，**test范围依赖未包含在POM中，并且默认runtime范围分配给所有其他依赖**。

发布任务还将此POM文件和JAR上传到指定的仓库。

## 4. 自定义Maven插件

在某些情况下，自定义生成的POM文件中的项目信息可能会很有用，我们来看一下。

### 4.1 groupId、artifactId和version

可以在publications/{publicationName}块中处理groupId、artifactId和POM版本的更改：

```groovy
publishing {
    publications {
        customLibrary(MavenPublishing) {
            groupId = '.sample'
            artifactId = 'gradle-maven-converter'
            version = '0.0.1-maven'
        }
    }
}
```

运行发布任务现在会生成包含上面提供的信息的POM文件：

```xml
<groupId>cn.tuyucheng.taketoday.sample</groupId>
<artifactId>gradle-maven-converter</artifactId>
<version>0.0.1-maven</version>
```

### 4.2 自动生成内容

Maven插件还可以轻松更改任何生成的POM元素，例如，要将默认运行时范围更改为compile，我们可以将以下闭包添加到pom.withXml方法中：

```groovy
pom.withXml {
    asNode()
        .dependencies
        .dependency
        .findAll { dependency ->
            // find all dependencies with runtime scope
            dependency.scope.text() == 'runtime'
        }
        .each { dependency ->
            // set the scope to 'compile'
            dependency.scope*.value = 'compile'
        }
}
```

这将改变生成的POM文件中所有依赖的范围：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
    <scope>compile</scope>
</dependency>
```

### 4.3 附加信息

最后，如果我们想添加额外的信息，**我们可以将这些Maven支持的元素包含到pom函数中**。

让我们添加一些许可证信息：

```groovy
...
pom {
    licenses {
        license {
            name = 'The Apache License, Version 2.0'
            url = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
        }
    }
}
...
```

我们现在可以看到添加到POM的许可证信息：

```xml
... 
<licenses>
    <license>
        <name>The Apache License, Version 2.0</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
    </license>
</licenses>
...
```

## 5. 总结

在本快速教程中，我们学习了如何将Gradle构建文件转换为Maven POM。