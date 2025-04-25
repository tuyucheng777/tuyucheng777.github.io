---
layout: post
title:  使用Java中的Error Prone库捕获常见错误
category: staticanalysis
copyright: staticanalysis
excerpt: Error Prone
---

## 1. 简介

**确保代码质量对于成功部署应用程序至关重要**，错误和漏洞会严重影响软件的功能和稳定性，有一个实用工具可以帮助识别此类错误：[Error Prone](https://errorprone.info/index)。

Error Prone是Google内部维护和使用的库，它帮助Java开发人员在编译阶段检测和修复常见的编程错误。

在本教程中，我们将探索Error Prone库的功能，从安装到定制，以及它在提高代码质量和稳健性方面提供的好处。

## 2. 安装

该库可在[Maven Central仓库](https://mvnrepository.com/artifact/com.google.errorprone/error_prone_core)中找到，我们将添加一个新的构建配置来配置我们的应用程序编译器，以运行Error Prone检查：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.12.1</version>
            <configuration>
                <release>17</release>
                <encoding>UTF-8</encoding>
                <compilerArgs>
                    <arg>-XDcompilePolicy=simple</arg>
                    <arg>-Xplugin:ErrorProne</arg>
                </compilerArgs>
                <annotationProcessorPaths>
                    <path>
                        <groupId>com.google.errorprone</groupId>
                        <artifactId>error_prone_core</artifactId>
                        <version>2.23.0</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

由于JDK 16版本对[JDK内部机制](https://openjdk.org/jeps/396)进行了强封装，我们需要添加一些参数才能运行该插件。如果.mvn/jvm.config文件不存在，可以创建一个新的文件，并添加插件所需的参数：

```properties
--add-exports jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED
--add-exports jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED
--add-exports jdk.compiler/com.sun.tools.javac.main=ALL-UNNAMED
--add-exports jdk.compiler/com.sun.tools.javac.model=ALL-UNNAMED
--add-exports jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED
--add-exports jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED
--add-exports jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED
--add-exports jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED
--add-opens jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED
--add-opens jdk.compiler/com.sun.tools.javac.comp=ALL-UNNAMED
```

如果我们的[maven-compiler-plugin](https://www.baeldung.com/maven-compiler-plugin)使用外部可执行文件或启用了maven-toolchains-plugin，我们应该将exports和opens添加为compilerArgs：

```xml
<compilerArgs>
    // ...
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED</arg>
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED</arg>
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.main=ALL-UNNAMED</arg>
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.model=ALL-UNNAMED</arg>
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED</arg>
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED</arg>
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED</arg>
    <arg>-J--add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED</arg>
    <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED</arg>
    <arg>-J--add-opens=jdk.compiler/com.sun.tools.javac.comp=ALL-UNNAMED</arg>
</compilerArgs>
```

## 3. 错误模式

识别和理解常见的错误模式对于维护软件的稳定性和可靠性至关重要，通过在开发过程的早期识别这些模式，我们可以主动实施策略来预防错误，并提高代码的整体质量。

### 3.1 预定义的错误模式

**该插件包含500多个[预定义的错误模式](https://errorprone.info/bugpatterns)**，其中一个错误是[DeadException](https://errorprone.info/bugpattern/DeadException)，我们将举例说明：

```java
public static void main(String[] args) {
    if (args.length == 0 || args[0] != null) {
        new IllegalArgumentException();
    }
    // other operations with args[0]
}
```

在上面的代码中，我们希望确保程序接收的参数非空。否则，我们应该抛出IllegalArgumentException异常。然而，由于粗心大意，我们创建了这个异常，却忘记了抛出它。很多情况下，如果没有错误检查工具，这种情况可能会被忽略。

我们可以使用maven clean verify命令对代码运行Error Prone检查，如果这样做，我们将收到以下编译错误：

```text
[ERROR] /C:/Dev/incercare_2/src/main/java/org/example/Main.java:[6,12] [DeadException] Exception created but not thrown
    (see https://errorprone.info/bugpattern/DeadException)
  Did you mean 'throw new IllegalArgumentException();'?
```

我们可以看到该插件不仅检测到了我们的错误而且还为我们提供了解决方案。

### 3.2 自定义错误模式

**Error Prone的另一个显著特点是它支持创建自定义错误检查器**，这些自定义错误检查器使我们能够根据特定的代码库定制工具，并高效地解决特定领域的问题。

要创建自定义检查，我们需要初始化一个新项目，我们将其命名为my-bugchecker-plugin，首先，我们添加错误检查器的配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.12.1</version>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>com.google.auto.service</groupId>
                        <artifactId>auto-service</artifactId>
                        <version>1.0.1</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>

<dependencies>
    <dependency>
        <groupId>com.google.errorprone</groupId>
        <artifactId>error_prone_annotation</artifactId>
        <version>2.23.0</version>
    </dependency>
    <dependency>
        <groupId>com.google.errorprone</groupId>
        <artifactId>error_prone_check_api</artifactId>
        <version>2.23.0</version>
    </dependency>
    <dependency>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service-annotations</artifactId>
        <version>1.0.1</version>
    </dependency>
</dependencies>
```

这次我们添加了更多依赖，如我们所见，除了Error Prone依赖之外，我们还添加了[Google AutoService](https://www.baeldung.com/google-autoservice)，Google AutoService是在Google Auto项目下开发的开源代码生成器工具，它将发现并加载我们的自定义检查。

现在我们将创建自定义检查，它将验证我们的代码库中是否有任何空方法：

```java
@AutoService(BugChecker.class)
@BugPattern(name = "EmptyMethodCheck", summary = "Empty methods should be deleted", severity = BugPattern.SeverityLevel.ERROR)
public class EmptyMethodChecker extends BugChecker implements BugChecker.MethodTreeMatcher {

    @Override
    public Description matchMethod(MethodTree methodTree, VisitorState visitorState) {
        if (methodTree.getBody()
                .getStatements()
                .isEmpty()) {
            return describeMatch(methodTree, SuggestedFix.delete(methodTree));
        }
        return Description.NO_MATCH;
    }
}
```

首先，注解BugPattern包含错误的名称、简短摘要和严重程度。其次，BugChecker本身是MethodTreeMatcher的一个实现，因为我们想要匹配具有空主体的方法。最后，如果方法树主体没有任何语句，则matchMethod()中的逻辑应该返回匹配结果。

**要在另一个项目中使用我们的自定义错误检查器，我们应该将其编译成一个单独的JAR文件**。具体操作如下：运行maven clean install命令，之后，我们将生成的JAR文件添加到主项目的构建配置中，并将其作为依赖添加到commentProcessorPaths中：

```xml
<annotationProcessorPaths>
    <path>
        <groupId>com.google.errorprone</groupId>
        <artifactId>error_prone_core</artifactId>
        <version>2.23.0</version>
    </path>
    <path>
        <groupId>com.baeldung</groupId>
        <artifactId>my-bugchecker-plugin</artifactId>
        <version>1.0-SNAPSHOT</version>
    </path>
</annotationProcessorPaths>
```

这样，我们的错误检查器也变得可重用了。现在，如果我们编写一个带有空方法的新类：

```java
public class ClassWithEmptyMethod {

    public void theEmptyMethod() {
    }
}
```

如果我们再次运行maven clean verify命令，我们将收到错误：

```text
[EmptyMethodCheck] Empty methods should be deleted
```

## 4. 自定义检查

Google Error Prone是一款非常棒的工具，它可以帮助我们在代码引入BUG之前就将其消除。但是，有时它对我们的代码来说可能过于苛刻，假设我们想为空函数抛出异常，但仅限于此。我们可以添加[SuppressWarnings](https://www.baeldung.com/java-suppresswarnings)注解，并在注解中写入我们想要绕过的检查名称：

```java
@SuppressWarnings("EmptyMethodCheck")
public void emptyMethod() {}
```

**不建议抑制警告，但在某些情况下可能需要，例如在使用未实现与我们的项目相同代码标准的外部库时**。

除此之外，我们还可以使用额外的编译器参数来控制所有检查的严重性：

- -Xep:EmptyMethodCheck：打开EmptyMethodCheck，并使用BugPattern注解中的严重性级别
- -Xep:EmptyMethodCheck：OFF关闭EmptyMethodCheck检查
- -Xep:EmptyMethodCheck：WARN将EmptyMethodCheck检查作为警告打开
- -Xep:EmptyMethodCheck：ERROR将EmptyMethodCheck检查视为错误

我们还有一些适用于所有检查的全局严重性更改标志：

- -XepAllErrorsAsWarnings
- -XepAllSuggestionsAsWarnings
- -XepAllDisabledChecksAsWarnings
- -XepDisableAllChecks
- -XepDisableAllWarnings
- -XepDisableWarningsInGeneratedCode

我们还可以将自定义编译器标志与全局编译器标志结合起来：

```xml
<compilerArgs>
    <arg>-XDcompilePolicy=simple</arg>
    <arg>-Xplugin:ErrorProne -XepDisableAllChecks -Xep:EmptyMethodCheck:ERROR</arg>
</compilerArgs>
```

通过如上所述配置我们的编译器，我们将禁用除我们创建的自定义检查之外的所有检查。

## 5. 重构代码

我们的插件与其他静态[代码分析程序](https://www.baeldung.com/java-static-code-analysis-tutorial)的一个区别在于它能够对代码库进行修补，**除了在标准编译阶段识别错误之外，Error Prone还可以提供建议的替换方法**，正如我们在第三点中看到的，当Error Prone发现DeadException时，它还会建议相应的修复方法：

```shell
Did you mean 'throw new IllegalArgumentException();'?
```

在这种情况下，Error Prone建议通过添加throw关键字来解决这个问题。我们也可以使用Error Prone根据建议的替换修改源代码，这在我们首次将Error Prone强制添加到现有代码库时非常有用，要激活此功能，我们需要在编译器调用中添加两个编译器标志：

- –XepPatchChecks：后面跟着我们要修补的检查，如果检查没有建议修复，则不会执行任何操作。
- -XepPatchLocation：生成包含修复程序的补丁文件的位置。

因此，我们可以像这样重写我们的编译器配置：

```xml
<compilerArgs>
    <arg>-XDcompilePolicy=simple</arg>
    <arg>-Xplugin:ErrorProne -XepPatchChecks:DeadException,EmptyMethodCheck -XepPatchLocation:IN_PLACE</arg>
</compilerArgs>
```

我们将告诉编译器修复DeadException和我们自定义的EmptyMethodCheck，我们将位置设置为IN_PLACE，这意味着它将应用源代码中的更改。

现在，如果我们对有缺陷的类运行maven clean verify命令：

```java
public class BuggyClass {

    public static void main(String[] args) {
        if (args.length == 0 || args[0] != null) {
             new IllegalArgumentException();
        }
    }

    public void emptyMethod() {
    }
}
```

它将重构该类：

```java
public class BuggyClass {

    public static void main(String[] args) {
        if (args.length == 0 || args[0] != null) {
             throw new IllegalArgumentException();
        }
    }
}
```

## 6. 总结

总而言之，Error Prone是一款功能强大的工具，它将有效的错误识别与可自定义的配置相结合。它使开发人员能够无缝地执行编码标准，并通过自动建议替换来促进高效的代码重构。