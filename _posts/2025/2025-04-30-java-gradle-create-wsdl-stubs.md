---
layout: post
title:  使用Gradle生成WSDL存根
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

简而言之，**[Web服务描述语言(WSDL)](https://www.baeldung.com/jax-ws#web-services-definition-language-wsdl)是一种基于XML的语言，用于描述Web服务所提供的功能**。WSDL存根是从WSDL文件生成的代理类，使与Web服务交互变得更加容易，而无需手动创建和管理[SOAP](https://www.baeldung.com/java-soap-web-service)消息。

在本教程中，我们将学习如何使用Gradle生成WSDL存根。此外，我们还将查看一个示例WSDL文件，并从中生成存根。

## 2. 示例设置

首先，让我们创建一个新的Gradle项目，用于从WSDL文件生成WSDL存根。接下来，我们将为WSDL文件创建目录结构：

```shell
$ mkdir -p src/main/resources/wsdl
```

我们将使用一个公共的WSDL文件，该文件可以将数字转换为相应的单词，让我们下载该WSDL文件并将其放在wsdl文件夹中：

```shell
$ curl -o src/main/resources/wsdl/NumberConversion.wsdl https://www.dataaccess.com/webservicesserver/numberconversion.wso?WSDL
```

上述命令从[dataacess.com](https://www.dataaccess.com/webservicesserver/numberconversion.wso?WSDL)下载WSDL文件并将其放在指定的文件夹中。

在下一节中，我们将配置build.gradle来生成我们可以在示例程序中交互的类。

## 3. Gradle配置

要从WSDL文件生成Java类，我们需要一个使用Apache CXF库的插件，[com.github.bjornvester.wsdl2java](https://github.com/bjornvester/wsdl2java-gradle-plugin)就是这样一个插件，我们将在本教程中使用它。这个插件简化了流程，并允许我们配置gradle.build：

```groovy
plugins {
    id 'java'
    id("com.github.bjornvester.wsdl2java") version "1.2"
}
```

该项目需要两个插件，Java插件帮助我们编译代码、运行测试和创建JAR文件，WSDL插件帮助我们从WSDL文件生成Java类。众所周知，WSDL文件是描述Web服务的XML文档。

我们可以使用wsdl2java扩展配置WSDL插件：

```groovy
wsdl2java {
    // ...
}
```

另外，我们可以配置CXF版本：

```groovy
wsdl2java {
    cxfVersion.set("3.4.4")
}
```

**默认情况下，该插件会为resources文件夹中的所有WSDL文件创建存根**，我们还可以通过指定其位置来配置它，以便为特定的WSDL文件创建存根：

```groovy
wsdl2java {
    // ...
    includes = [
        "src/main/resources/wsdl/NumberConversion.wsdl",
    ]
    // ... 
}
```

**此外，生成的类保存在build/generated/sources/wsdl2java文件夹中**，但我们可以通过指定自己的文件夹来覆盖它：

```groovy
wsdl2java {
    // ...
    generatedSourceDir.set(layout.projectDirectory.dir("src/generated/wsdl2java"))
    // ... 
}
```

在上面的代码中，我们指定存储生成的类的位置，而不是使用默认文件夹。

配置完成后，我们需要运行Gradle wsdl2java命令来生成存根：

```shell
$ ./gradlew wsdl2java
```

上面的命令生成了Java类，我们现在可以在程序中与它们交互。

## 4. 从WSDL文件生成WSDL存根

首先，让我们检查示例项目的build.gradle文件：

```groovy
plugins {
    id 'java'
    id("com.github.bjornvester.wsdl2java") version "1.2"
}
repositories {
    mavenCentral()
}
dependencies {
    implementation 'com.sun.xml.ws:jaxws-ri:4.0.1'
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.2'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.2'
}
test {
    useJUnitPlatform()
}
wsdl2java {
    cxfVersion.set("3.4.4")
}
```

上面的示例代码展示了如何配置WSDL插件以使用CXF版本3.4.4，该插件在默认位置生成存根，并在src/main/resources/wsdl中查找WSDL文件，这正是我们之前放置WSDL文件的位置。

此外，我们需要[Java API for XML Web Services(JAX-WS)](https://www.baeldung.com/jax-ws)[依赖](https://mvnrepository.com/artifact/com.sun.xml.ws/jaxws-ri)来与服务交互并执行单元测试。

要从WSDL文件生成Java类，我们可以执行Gradle wsdl2java命令：

```shell
$ ./gradlew wsdl2java
```

以下是生成的Java类：

![](/assets/images/2025/gradle/javagradlecreatewsdlstubs01.png)

生成的类存储在默认位置，接下来，让我们通过编写单元测试来与这些类进行交互：

```java
@Test
public void givenNumberConversionService_whenConvertingNumberToWords_thenReturnCorrectWords() {
    NumberConversion service = new NumberConversion();
    NumberConversionSoapType numberConversionSoapType = service.getNumberConversionSoap();
    String numberInWords = numberConversionSoapType.numberToWords(BigInteger.valueOf(10000000));
        
    assertEquals("ten million", numberInWords);
}
```

在上面的示例单元测试中，我们创建了一个NumberConversion的新实例，并在服务对象上调用getNumberConversionSoap()方法来获取对NumberConversionSoapType对象的引用。

此外，我们调用numberConversionSoapType上的numberToWords()方法并将值1000000作为参数传递。

最后，我们断言预期值等于输出。

## 5. 总结

在本文中，我们学习了如何使用Gradle生成WSDL存根。此外，我们还了解了如何自定义插件配置，例如指定CXF版本和生成类的输出目录。此外，我们还讨论了如何通过编写单元测试与生成的类进行交互。