---
layout: post
title:  使用Infer进行静态代码分析
category: staticanalysis
copyright: staticanalysis
excerpt: Infer
---

## 1. 简介

在软件开发领域，确保代码质量非常重要，尤其是对于复杂且庞大的代码库，**[Infer](https://fbinfer.com/)等静态代码分析工具为我们提供了检测代码库中潜在错误和漏洞的技术，防止它们变成严重问题**。

在本教程中，我们将探讨代码分析的基础知识，探索Infer的功能，并提供将其纳入我们的开发工作流程的实用见解。

## 2. 静态代码分析

**静态分析是一种无需执行程序即可自动检查源代码的调试方法**，此过程有助于识别潜在缺陷、安全漏洞和可维护性问题，静态代码分析通常由第三方工具(如著名的[SonarQube](https://www.sonarsource.com/products/sonarqube/))进行，自动化时非常简单。

**通常，它发生在早期开发阶段**。编写代码后，应运行静态代码分析器来检查代码，它将根据标准中定义的编码规则或自定义预定义规则进行检查，一旦代码通过静态代码分析器运行，分析器就会确定代码是否符合设定的规则。

在基本的企业环境中，它通常是[持续集成(CI)](https://www.baeldung.com/cs/continuous-integration-deployment-delivery)流程的一部分，每次提交时，都会触发一项作业来构建应用程序、运行测试和分析代码，确保在部署之前符合法规、安全性和保障性。

## 3. Infer

**Infer是一个用[OCaml](https://ocaml.org/)编写的Java、C、C++和Objective-C静态代码分析工具**，它最初由Facebook开发，并于[2015年开源](https://engineering.fb.com/2015/06/11/developer-tools/open-sourcing-facebook-infer-identify-bugs-before-you-ship/)。从那时起，它在Facebook之外越来越受欢迎，被其他大公司采用。

对于Java和Android代码，它会检查诸如空指针异常、资源泄漏和并发竞争条件等问题。对于C、C++和Objective-C，它会检测诸如空指针问题、内存泄漏、编码约定违规以及对不可用API的调用等问题。

**为了高效处理大型代码库，Infer采用了[分离逻辑和双向推理](https://fbinfer.com/docs/separation-logic-and-bi-abduction)等技术**，分离逻辑允许它独立分析代码存储的小部分，避免需要一次处理整个内存空间。双向推理是一种逻辑推理方法，可帮助Infer发现有关代码不同部分的属性，从而使其在后续分析中仅关注修改过的部分。

通过结合这些方法，该工具可以在短时间内发现由数百万行代码构建的应用程序修改中的复杂问题。

### 3.1 推断阶段

无论输入语言是什么，Infer都分为两个主要阶段运行：[捕获阶段](https://fbinfer.com/docs/infer-workflow#1-the-capture-phase)和[分析阶段](https://fbinfer.com/docs/infer-workflow#2-the-analysis-phase)。

**在捕获阶段，Infer会捕获编译命令，将要分析的文件翻译成其内部的中间语言**。此翻译类似于编译，因此Infer从编译过程中获取信息来执行自己的翻译，这也是我们使用编译命令调用Infer的原因，例如：
```shell
infer run -- javac File.java
```

因此，文件会像往常一样进行编译，Infer还会将它们转换为在第二阶段进行分析。Infer将中间文件存储在结果目录中，该目录默认为infer-out/，在调用infer命令的文件夹中创建。

另外，我们可以使用capture子命令仅调用捕获阶段：
```shell
infer capture -- javac File.java
```

**在分析阶段，Infer会单独分析infer-out/中的文件，每个函数和方法都会单独分析。如果Infer在分析某个方法或函数时遇到错误，它会停止对该特定实体的分析，但会继续分析其他实体**。因此，典型的工作流程包括在代码上运行Infer、解决已发现的错误，然后重新运行Infer以发现其他问题或确认修复。

检测到的错误会在标准输出和名为infer-out/report.txt的文件中报告，推断过滤器并突出显示最有可能是真实的错误。

我们可以使用analyze子命令仅调用分析阶段：
```shell
infer analyze
```

### 3.2 推断全局和差异工作流

默认情况下，Infer将删除之前的infer-out/目录(如果存在)，这会导致[全局工作流程](https://fbinfer.com/docs/infer-workflow#global-workflow)，每次都会分析整个项目。

向Infer传递–reactive或-r可防止删除infer-out/目录，从而导致[差异化工作流程](https://fbinfer.com/docs/infer-workflow#differential-workflow)。例如，移动应用程序使用增量构建系统，其中代码随着一系列代码更改而发展。对于这些更改，只分析项目中的当前更改而不是每次都分析整个项目是有意义的，因此，我们可以利用Infer的响应模式并切换到差异化工作流程。

### 3.3 分析项目

要使用Infer分析文件，我们可以使用javac和clang等编译器。此外，我们还可以使用gcc，尽管Infer内部将使用clang。此外，我们可以将Infer与各种构建系统一起使用。

Java的一个流行构建系统是[Maven](https://www.baeldung.com/maven)，我们可以通过以下方式与Infer一起使用：
```shell
infer run -- mvn <maven target>
```

另外，我们可以将Infer与[Gradle](https://www.baeldung.com/gradle)一起使用：
```shell
infer run -- gradle <gradle task>
```

### 3.4 CI的推荐流程

**Infer建议使用差异工作流程进行持续集成(CI)**，因此，流程将是确定修改的文件并以反应模式运行分析。此外，如果我们想运行多个分析器，那么分离捕获阶段会更有效率，这样所有分析器都可以使用结果。

## 4. 运行Infer

我们有多种使用Infer的选项：二进制版本、从源代码构建Infer或[Docker](https://www.baeldung.com/ops/docker-guide)镜像，[Infer入门](https://fbinfer.com/docs/getting-started)页面介绍了如何获取和运行Infer。

现在，我们可以创建一些虚拟Java代码片段并进行分析，我们不会只介绍Infer可以识别的少数问题，你**可以在[此处](https://fbinfer.com/docs/all-issue-types)找到Infer可以检测到的所有类型问题的完整列表**。

### 4.1 空引用
```java
public class NullPointerDereference {
    public static void main(String[] args) {
        NullPointerDereference.nullPointerDereference();
    }

    private static void nullPointerDereference() {
        String str = null;
        int length = str.length();
    }
}
```

如果我们根据此代码进行推断，我们将得到以下输出：
```text
Analyzed 1 file

Found 1 issue

./NullPointerDereference.java:11: error: NULL_DEREFERENCE
  object str last assigned on line 10 could be null and is dereferenced at line 11
   9.       private static void nullPointerDereference() {
  10.           String str = null;
  11. >         int length = str.length();
  12.       }
  13.   }
  24.   

Summary of the reports

  NULL_DEREFERENCE: 1
```

### 4.2 资源泄漏
```java
public class ResourceLeak {
    public static void main(String[] args) throws IOException {
        ResourceLeak.resourceLeak();
    }

    private static void resourceLeak() throws IOException {
        FileOutputStream stream;
        try {
            File file = new File("randomName.txt");
            stream = new FileOutputStream(file);
        } catch (IOException e) {
            return;
        }
        stream.write(0);
    }
}
```

现在，通过运行Infer，我们可以看到检测到了资源泄漏：
```text
Analyzed 1 file

Found 1 issue

./ResourceLeak.java:21: error: RESOURCE_LEAK
   resource of type java.io.FileOutputStream acquired to stream by call to FileOutputStream(...) at line 17 is not released after line 21
  19.               return;
  20.           }
  21. >         stream.write(0);
  22.       }
  23.   }
  24.   

Summary of the reports

  RESOURCE_LEAK: 1
```

### 4.3 除以0
```java
public class DivideByZero {
    public static void main(String[] args) {
        DivideByZero.divideByZero();
    }

    private static void divideByZero() {
        int dividend = 5;
        int divisor = 0;
        int result = dividend / divisor;
    }
}
```

我们的代码显示以下输出：
```text
Analyzed 1 file

Found 1 issue

./DivideByZero.java:9: error: DIVIDE_BY_ZERO
  The denominator for division is zero, which triggers an Arithmetic exception.
  6. private static void divideByZero() {
  7.     int dividend = 5;
  8.     int divisor = 0;
  9. >    int result = dividend / divisor;
  10. }

Summary of the reports

DIVIDE_BY_ZERO: 1
```

## 5. 总结

正如我们在本教程中发现的那样，静态代码分析工具对于软件开发过程至关重要，其中一种工具是Infer，它以能够检测各种编程语言中的各种问题而脱颖而出。**通过利用Infer进行静态代码分析，我们可以主动解决错误和漏洞，从而开发出更可靠、更安全的软件应用程序**。