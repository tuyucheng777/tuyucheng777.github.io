---
layout: post
title:  探索jrecreate
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. EJDK介绍

EJDK(嵌入式Java开发工具包)是由Oracle推出的，用于解决为所有可用的嵌入式平台提供二进制文件的问题。我们可以从此处的[Oracle站点](http://www.oracle.com/technetwork/java/embedded/downloads/java-embedded-java-se-download-359230.html)下载最新的EJDK。

简而言之，它包含用于创建特定于平台的JRE的工具。

## 2. jrecreate

EJDK为Windows提供了jrecreate.bat，为Unix/Linux平台提供了jrecreate.sh。该工具有助于为我们希望使用的平台组装自定义JRE，并被引入到：

-   尽量减少Oracle为每个平台发布的二进制文件
-   使为其他平台创建自定义JRE变得容易

在Unix/Linux中，使用以下语法执行jrecreate命令：

```shell
$jrecreate.sh -<option>/--<option> <argument-if-any>
```

在Windows中：

```shell
$jrecreate.bat -<option>/--<option> <argument-if-any>
```

请注意，我们可以为单个JRE创建添加多个选项。现在，让我们看一下该工具可用的一些选项。

## 3. jrecreate选项

### 3.1 Destination

destination选项是必需的，它指定应该在其中创建目标JRE的目录：

```shell
$jrecreate.sh -d /SampleJRE
```

运行上述命令后，将在指定位置创建默认JRE，命令行输出为：

```text
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJRE
    target: linux_i586
    vm: all
    runtime: jre
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: []
}

Target JRE Size is 55,205 KB (on disk usage may be greater).
Embedded JRE created successfully
```

从上面的结果可以看出，目标JRE已在指定的目标目录下创建，所有其他选项均采用默认值。

### 3.2 Profile

profile选项用于管理目标JRE的大小，profile定义要包含的API的功能。如果未指定profile选项，则该工具将默认包含所有JRE API：

```shell
$jrecreate.sh -d /SampleJRECompact1/ -p compact1
```

将会创建一个具有compact1 profile的JRE，我们也可以在命令中使用––profile代替-p，命令行输出将显示以下结果：

```text
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJRECompact1
    target: linux_i586
    vm: minimal
    runtime: compact1 profile
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: []
}

Target JRE Size is 10,808 KB (on disk usage may be greater).
Embedded JRE created successfully
```

在上面的结果中，请注意runtime选项的值为compact1。另请注意，结果JRE的大小略低于11MB，低于上一个示例中的55MB。

profile设置有3个可用选项：compact1、compact2和compact3。

### 3.3 JVM

jvm选项用于根据用户的需要使用特定的JVM自定义目标JRE，默认情况下，如果未指定profile和jvm选项，则它包括所有可用的JVM(客户端、服务器和最小)：

```shell
$jrecreate.sh -d /SampleJREClientJVM/ --vm client
```

将创建一个带有client jvm的JRE，命令行输出将显示以下结果：

```text
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJREClientJVM
    target: linux_i586
    vm: Client
    runtime: jre
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: []
}

Target JRE Size is 46,217 KB (on disk usage may be greater).
Embedded JRE created successfully
```

在上面的结果中，请注意vm选项的值为Client。我们还可以使用此选项指定其他JVM，如server和minimal。

### 3.4 Extension

extension选项用于包含对目标JRE的各种允许的扩展，默认情况下，不添加任何扩展：

```shell
$jrecreate.sh -d /SampleJRESunecExt/ -x sunec
```

将创建带有扩展名sunec(椭圆曲线密码安全提供程序)的JRE。我们也可以在命令中使用––extension代替-x，命令行输出将显示以下结果：

```shell
Building JRE using Options {
    ejdk-home: /installDir/ejdk1.8.0/bin/..
    dest: /SampleJRESunecExt
    target: linux_i586
    vm: all
    runtime: jre
    debug: false
    keep-debug-info: false
    no-compression: false
    dry-run: false
    verbose: false
    extension: [sunec]
}

Target JRE Size is 55,462 KB (on disk usage may be greater).
Embedded JRE created successfully
```

在上面的结果中，请注意extension选项的值为sunec，使用此选项可以添加多个扩展。

### 3.5 其他选项

除了上面讨论的主要选项之外，jrecreate还为用户提供了更多选项：

-   ––help：显示jrecreate工具的命令行选项摘要
-   ––debug：创建具有调试支持的JRE
-   ––keep-debug-info：保留类和未签名的JAR文件中的调试信息
-   ––dry-run：执行试运行，但不实际创建JRE
-   ––no-compression：使用未签名的JAR文件以未压缩格式创建JRE
-   ––verbose：显示所有jrecreate命令的详细输出

## 4. 总结

在本教程中，我们学习了EJDK的基础知识，以及如何使用jrecreate工具生成特定于平台的JRE。