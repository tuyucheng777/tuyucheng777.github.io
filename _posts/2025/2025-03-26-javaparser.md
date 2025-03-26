---
layout: post
title:  JavaParser简介
category: libraries
copyright: libraries
excerpt: JavaParser
---

## 1. 简介

在本文中，我们将了解[JavaParser](https://github.com/javaparser/javaparser)库。我们将了解它是什么、可以用它做什么以及如何使用它。

## 2. 什么是JavaParser？

JavaParser是一个用于处理Java源代码的开源库，**它允许我们将Java源代码解析为[抽象语法树](https://en.wikipedia.org/wiki/Abstract_syntax_tree)(AST)。完成此操作后，我们可以分析解析后的代码、对其进行操作，甚至编写新代码**。

使用JavaParser，我们可以解析用Java编写的源代码，最高版本为Java 18。这包括所有稳定的语言功能，但可能不包括任何[预览功能](https://www.baeldung.com/java-preview-features)。

## 3. 依赖

**在我们可以使用JavaParser之前，我们需要在我们的构建中包含[最新版本](https://mvnrepository.com/artifact/com.github.javaparser/javaparser-core)，在撰写本文时是[3.25.10](https://mvnrepository.com/artifact/com.github.javaparser/javaparser-core/3.25.10)**。

我们需要包含的主要依赖是javaparser-core，如果使用Maven，我们可以在pom.xml文件中包含此依赖：

```xml
<dependency>
    <groupId>com.github.javaparser</groupId>
    <artifactId>javaparser-core</artifactId>
    <version>3.25.10</version>
</dependency>
```

或者如果我们使用Gradle，可以将其包含在我们的build.gradle文件中：

```groovy
implementation("com.github.javaparser:javaparser-core:3.25.10")
```

此时，我们已准备好开始在我们的应用程序中使用它。

另外还有两个依赖可用，依赖com.github.javaparser:javaparser-symbol-solver-core提供了一种分析已解析AST的方法，以查找Java元素与其声明之间的关系。依赖com.github.javaparser:javaparser-core-serialization提供了一种将已解析AST序列化为JSON或从JSON序列化的方法。

## 4. 解析Java代码

一旦我们在应用程序中设置了依赖，我们就可以开始了。**Java代码的解析始终从StaticJavaParser类开始**，这为我们提供了几种不同的代码解析机制，具体取决于我们要解析的内容以及代码来自何处。

### 4.1 解析源文件

**我们首先要看的是解析整个源文件，我们可以使用StaticJavaParser.parse()方法来实现这一点**。有几种重载替代方案允许我们以不同的方式提供源代码-直接作为字符串、作为本地文件系统上的文件或作为某些资源的InputStream或Reader。所有这些方法的工作方式都相同，并且都是提供要解析的代码的便捷方法。

让我们看看它的实际效果。在这里，我们将尝试解析提供的源代码并生成CompilationUnit作为结果：

```java
CompilationUnit parsed = StaticJavaParser.parse("class TestClass {}");
```

这代表我们的AST并允许我们检查和操作解析的代码。

### 4.2 解析语句

**我们可以解析的代码范围的另一端是单个语句，我们使用StaticJavaParser.parseStatement()方法执行此操作**。与源文件不同，该方法只有一个版本，它接收包含要解析的语句的单个字符串。

此方法返回代表已解析语句的Statement对象：

```java
Statement parsed = StaticJavaParser.parseStatement("final int answer = 42;");
```

### 4.3 解析其他结构

**JavaParser还可以解析许多其他构造，涵盖Java 18之前的整个Java语言。每个构造都有一个单独的专用解析方法，并返回表示解析代码的适当类型**。例如，我们可以使用parseAnnotation()来解析注解，使用parseImport()来解析导入语句，使用parseBlock()来解析语句块等等。

在内部，JavaParser将使用完全相同的代码来解析我们代码的各个部分。例如，当使用parseBlock()解析块时，JavaParser最终将使用与parseStatement()直接调用相同的代码，这意味着我们可以依靠这些不同的解析方法对相同的代码子集进行相同的工作。

**我们确实需要确切地知道我们正在解析的代码类型，以便选择正确的解析方法**。例如，使用parseStatement()方法解析类定义将会失败。

### 4.4 畸形代码

**如果解析失败，JavaParser将抛出ParseProblemException，准确指出代码中存在什么问题**。例如，如果我们尝试解析格式错误的类定义，则会得到类似以下内容的结果：

```java
ParseProblemException parseProblemException = assertThrows(ParseProblemException.class,
    () -> StaticJavaParser.parse("class TestClass"));

assertEquals(1, parseProblemException.getProblems().size());
assertEquals("Parse error. Found <EOF>, expected one of  \"<\" \"extends\" \"implements\" \"permits\" \"{\"", 
    parseProblemException.getProblems().get(0).getMessage());
```

从这个错误信息中我们可以看出，问题在于类定义错误。在Java中，这样的语句后面必须跟一个“<”-对于泛型定义，extends或implements关键字，或者跟一个“{”来开始类的实际主体。

## 5. 分析解析后的代码

**一旦解析了某些代码，我们就可以开始分析它并从中学习。这类似于正在运行的应用程序内的[反射](https://www.baeldung.com/java-reflection)，只不过是针对已解析的源代码而不是当前正在运行的代码**。

### 5.1 访问已解析元素

**一旦我们解析了一些源代码，我们就可以查询AST来访问单个元素**。具体如何操作取决于我们想要访问的元素以及我们解析的内容。

例如，如果我们已经将源文件解析为CompilationUnit，那么我们可以使用getClassByName()访问我们期望存在的类：

```java
Optional<ClassOrInterfaceDeclaration> cls = compilationUnit.getClassByName("TestClass");
```

请注意，这将返回一个Optional<ClassOrInterfaceDeclaration\>。使用Optional是因为我们不能保证该类型存在于此编译单元中。在其他情况下，我们可能能够保证元素的存在。例如，一个类将始终有一个名称，因此ClassOrInterfaceDeclaration.getName()不需要返回Optional。

在每个阶段，我们只能直接访问当前正在处理的最外层元素。例如，如果我们从解析源文件中获得CompilationUnit，那么我们可以访问包声明、导入语句和顶级类型，但无法访问这些类型中的成员。但是，一旦我们访问其中一种类型，我们就可以访问其中的成员。

### 5.2 迭代已解析的元素

**在某些情况下，我们可能不知道我们解析的代码中到底存在哪些元素，或者我们只是想处理某种类型的所有元素，而不仅仅是一个**。

我们的每个AST类型都可以访问一系列适当的嵌套元素，具体如何工作取决于我们想要处理什么。例如，我们可以使用以下命令从CompilationUnit中提取所有导入语句：

```java
NodeList<ImportDeclaration> imports = compilationUnit.getImports();
```

不需要Optional，因为这保证会返回结果。但是，如果没有导入，则此结果可能为空列表。

一旦完成此操作，我们就可以将其视为任何集合。NodeList类型正确实现了java.util.List，因此我们可以像处理任何其他列表一样处理它。

### 5.3 迭代整个AST

除了从解析后的代码中提取一种类型的元素之外，我们还可以遍历整个解析后的树。**JavaParser中的所有AST类型都实现了[访问者模式](https://www.baeldung.com/java-visitor-pattern)，允许我们使用自定义访问者访问解析后的源代码中的每个元素**：

```java
compilationUnit.accept(visitor, arg);
```

然后，我们可以使用两种标准类型的访问者。这两种访问者都针对每种可能的AST类型都有一个visit()方法，该方法接收传递到accept()调用的状态参数。

**其中最简单的是VoidVisitor<A\>，它对每个AST类型都有一个方法，没有返回值**。然后我们有一个适配器类型-VoidVisitorAdapter，它为我们提供了一个标准实现，以帮助确保正确调用整个树。

然后我们只需要实现我们感兴趣的方法-例如：

```java
compilationUnit.accept(new VoidVisitorAdapter<Object>() {
    @Override
    public void visit(MethodDeclaration n, Object arg) {
        super.visit(n, arg);

        System.out.println("Method: " + n.getName());
    }
}, null);
```

这将为源文件中的每个方法名称输出一条日志消息，无论它们位于何处。这会在整个树结构上递归，这意味着这些方法可以位于顶级类、内部类，甚至是其他方法内的匿名类中。

**另一种方法是GenericVisitor<R, A\>，它的工作原理与VoidVisitor类似，只是它的visit()方法具有返回值**。我们这里还有适配器类，具体取决于我们想要如何收集每个方法的返回值。例如，GenericListVisitorAdaptor将强制我们将每个方法的返回类型改为List<R\>，并将所有这些列表合并在一起：

```java
List<String> allMethods = compilationUnit.accept(new GenericListVisitorAdapter<String, Object>() {
    @Override
    public List<String> visit(MethodDeclaration n, Object arg) {
        List<String> result = super.visit(n, arg);
        result.add(n.getName().asString());
        return result;
    }
}, null);
```

这将返回一个包含整个树中每个方法的名称的列表。

## 6. 输出解析后的代码

**除了解析和分析代码之外，我们还可以将其再次输出为字符串**。这在很多情况下都很有用-例如，如果我们想提取并仅输出代码的特定部分。

**实现此目的的最简单方法是使用标准toString()方法，我们所有的AST类型都正确实现了此方法，并将生成格式化的代码**。请注意，这可能与我们解析代码时的格式不完全相同，但它仍将遵循相对标准的约定。

例如，如果我们解析以下代码：

```java
package cn.tuyucheng.taketoday.javaparser;
import java.util.List;
class TestClass {
private List<String> doSomething()  {}
private class Inner {
private String other() {}
}
}
```

当我们格式化它时，我们将得到以下输出：

```java
package cn.tuyucheng.taketoday.javaparser;

import java.util.List;

class TestClass {

    private List<String> doSomething() {
    }

    private class Inner {

        private String other() {
        }
    }
}
```

**我们可以使用另一种方法来格式化代码，即使用DefaultPrettyPrinterVisitor**。这是一个处理格式化的标准访问者类，这让我们可以配置输出格式化的某些方面。例如，如果我们想用2个空格而不是4个空格缩进，我们可以这样写：

```java
DefaultPrinterConfiguration printerConfiguration = new DefaultPrinterConfiguration();
printerConfiguration.addOption(new DefaultConfigurationOption(DefaultPrinterConfiguration.ConfigOption.INDENTATION,
    new Indentation(Indentation.IndentType.SPACES, 2)));
DefaultPrettyPrinterVisitor visitor = new DefaultPrettyPrinterVisitor(printerConfiguration);

compilationUnit.accept(visitor, null);
String formatted = visitor.toString();
```

## 7. 操作解析后的代码

**一旦我们将一些代码解析为AST，我们就可以对其进行更改**。由于这现在只是一个Java对象模型，我们可以将其视为任何其他对象模型，并且JavaParser让我们能够自由地更改它的大多数方面。

结合将AST输出为工作源代码的能力，我们就可以操作解析后的代码，对其进行更改，并以某种形式提供输出。这对于IDE插件、代码编译步骤等非常有用。

我们可以以任何可以访问适当AST元素的方式使用它-无论是直接访问它们，使用访问者进行迭代，还是其他任何有意义的方式。

例如，如果我们想将一段代码中的每个方法名称都大写，那么我们可以执行以下操作：

```java
compilationUnit.accept(new VoidVisitorAdapter<Object>() {
    @Override
    public void visit(MethodDeclaration n, Object arg) {
        super.visit(n, arg);
        
        String oldName = n.getName().asString();
        n.setName(oldName.toUpperCase());
    }
}, null);
```

这使用一个简单的访问者来访问源树中的每个方法声明，并使用setName()方法为每个方法赋予一个新名称，新名称只是大写的旧名称。

完成此操作后，AST就会更新。然后我们可以根据需要对其进行格式化，新格式化的代码将反映我们的更改。

## 8. 总结

我们在这里看到了JavaParser的简要介绍，我们展示了如何开始使用它以及我们可以用它实现的一些功能。