---
layout: post
title:  解析Java源代码并提取方法
category: java
copyright: java
excerpt: Java Sun
---

## 1. 简介

在本文中，我们将研究JavaCompiler API。**我们将了解这个API是什么、可以用它做什么以及如何使用它来提取源文件中定义的方法的详细信息**。

## 2. JavaCompiler API

**Java 6引入了ToolProvider机制，使我们能够访问各种内置JVM工具，其中包括JavaCompiler**。这与[javac](https://www.baeldung.com/javac)应用程序中的功能相同，但仅可通过编程方式使用。

利用JavaCompiler，我们可以编译Java源代码。但是，我们也可以在编译过程中从代码中提取信息。

为了访问JavaCompiler，我们需要使用ToolProvider，如果可用，它将为我们提供一个实例：

```java
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
```

请注意，无法保证JavaCompiler可用，这取决于所使用的JVM以及它提供的工具。

**但是，查询Java代码而不是简单地编译它是依赖于实现的。在本文中，我们假设使用Oracle编译器，并且[tools.jar](https://www.baeldung.com/java-build-compiler-plugin#setup)文件在类路径上可用**。请注意，自Java 9以来，此文件默认不再可用，因此我们需要确保有合适的版本可供使用。

## 3. 处理Java代码

**一旦JavaCompiler实例可用，我们就可以处理一些Java代码。我们需要一个适当的JavaFileManager实例和一个适当的JavaFileObject实例集合来执行此操作**，我们究竟如何做这两件事取决于我们希望处理的代码的来源。

如果我们想要处理以文件形式存在于磁盘上的代码，我们可以依赖JVM工具。特别是，JavaCompiler实例提供访问权限的StandardJavaFileManager正是用于此目的：

```java
StandardJavaFileManager fileManager = compiler.getStandardFileManager(null, null, StandardCharsets.UTF_8);
```

一旦我们得到这个，我们就可以使用它来访问我们想要处理的文件：

```java
Iterable<? extends JavaFileObject> compilationUnits = fileManager.getJavaFileObjectsFromFiles(Arrays.asList(new File(filename)));
```

如果需要，我们可以使用这些的其他实例。例如，如果我们想[处理保存在局部变量中的代码](https://www.baeldung.com/java-string-compile-execute-code)，

一旦我们有了这些，我们就可以处理我们的文件：

```java
JavacTask javacTask = (JavacTask) compiler.getTask(null, fileManager, null, null, null, compilationUnits);
Iterable<? extends CompilationUnitTree> compilationUnitTrees = javacTask.parse();
```

**请注意，我们将编译器的结果.getTask()转换为JavacTask实例，此类存在于tools.jar文件中，是查询已处理Java源代码的入口点**。然后，我们使用它来将输入文件解析为CompilationUnitTree类型的集合，每个集合都代表我们提供给编译器的文件。

## 4. 编译单元详情

**一旦我们走到这一步，我们就可以获得编译单元的解析细节-也就是我们已经处理过的源文件**。

我们能做的第一件事就是查询顶层细节。例如，我们可以使用getPackageName()查看它代表什么包，并使用getImports()获取导入列表。**我们还可以使用getTypeDecls()获取所有顶层声明的列表**-这通常意味着类定义，但可以是Java语言支持的任何内容。

我们会注意到，返回的所有内容都是Tree接口的实现。整个编译单元以树结构表示，允许适当嵌套内容。例如，如果方法已经嵌套在另一个类中，则可以将类定义[嵌套在方法](https://www.baeldung.com/java-nested-classes#1-local-classes)中。

这给我们带来的一个好处是，树结构实现了[访问者模式](https://www.baeldung.com/java-visitor-pattern)，这允许我们拥有可以查询结构的任何实例的代码，而无需事先知道它是什么。

这非常有用，因为getTypeDecls()返回任意Tree类型的集合，所以我们现在不知道我们正在处理什么：

```java
for (Tree tree : compilationUnitTree.getTypeDecls()) {
    tree.accept(new SimpleTreeVisitor() {
        @Override
        public Object visitClass(ClassTree classTree, Object o) {
            System.out.println("Found class: " + classTree.getSimpleName());
            return null;
        }
    }, null);
}
```

我们还可以通过直接查询来确定Tree实例的类型。我们所有的Tree实例都有一个getKind()方法，该方法从Kind枚举中返回适当的值。例如，类定义将返回Kind.CLASS以表明它们属于该类型。

如果我们不想使用访问者模式，我们可以使用它并自己转换值：

```java
for (Tree tree : compilationUnitTree.getTypeDecls()) {
    if (tree.getKind() == Tree.Kind.CLASS) {
        ClassTree classTree = (ClassTree) tree;
        System.out.println("Found class: " + classTree.getSimpleName());
    }
}
```

## 5. 类详情

**一旦我们获得了ClassTree实例的访问权限(无论我们如何管理它)，我们就可以开始查询有关类定义的详细信息**。这包括类名称、超类、接口列表等类级详细信息。

**我们还可以使用getMembers()获取类成员的详细信息，这包括可以作为类成员的任何内容，例如方法、字段、嵌套类等**，任何允许直接写入类主体的内容都将由此返回。

这与我们在CompilationUnitTree.getTypeDecls()中看到的一样，我们可以在其中获得不同类型的混合。因此，我们需要以类似的方式处理它，使用访问者模式或getKind()方法。

例如，我们可以从一个类中提取所有方法：

```java
for (Tree member : classTree.getMembers()) {
    member.accept(new SimpleTreeVisitor(){
        @Override
        public Object visitMethod(MethodTree methodTree, Object o) {
            System.out.println("Found method: " + methodTree.getName());
            return null;
        }
    }, null);
}
```

## 6. 方法详细信息

**如果我们愿意，我们可以查询MethodTree实例以获取有关方法本身的更多信息**。正如我们所期望的那样，我们可以获得有关方法签名的所有详细信息。这包括方法名称、参数、返回类型和throws子句，还包括泛型类型参数、修饰符等详细信息，甚至(如果[方法存在于注解类中](https://www.baeldung.com/java-custom-annotation#2-field-level-annotation-example))还包括默认值。

与往常一样，我们在这里给出的所有内容都是Tree或某个子类。例如，方法参数始终是VariableTree实例，因为这是该位置上唯一合法的东西。然后我们可以将它们视为源文件的任何其他部分。

例如，我们可以打印出某个方法的一些细节：

```java
System.out.println("Found method: " + classTree.getSimpleName() + "." + methodTree.getName());
System.out.println("Return value: " + methodTree.getReturnType());
System.out.println("Parameters: " + methodTree.getParameters());
```

这将产生如下输出：

```text
Found method: ExtractJavaLiveTest.visitClassMethods
Return value: void
Parameters: ClassTree classTree
```

## 7. 方法主体

**但我们可以更进一步，MethodTree实例让我们能够以语句集合的形式访问已解析的方法主体**。

与API中的其他任何地方相比，这里最能体现“一切都是树”这一事实的好处。在Java中，有各种具有特殊细节的语句，甚至包括一些包含其他语句的语句。

例如，以下Java代码是一条语句：

```java
for (Tree statement : methodTree.getBody().getStatements()) {
    System.out.println("Found statement: " + statement);
}
```

此语句是“[增强for循环](https://www.baeldung.com/java-for-loop#foreach)”，包括：

- 变量声明：Tree语句
- 表达式：methodTree.getBody().getStatements()
- 嵌套语句：包含System.out.println(“Found statement: ” + statement)的块；

我们的JavaCompiler将其表示为EnhancedForLoopTree实例，这使我们能够访问这些不同的详细信息。**Java中可以使用的每种不同类型的语句都由StatementTree的子类表示，这使我们能够再次获取相关详细信息**。

## 8. 面向未来

Java非常重视向后兼容性，然而，向前兼容性管理得不太好。这意味着Java代码可能会使用我们程序不期望的语法。例如，Java 5引入了增强的for循环。如果我们期望代码比这更旧，我们会惊讶地看到其中的一个。

然而，这一切意味着我们必须为可能意想不到的Tree实例做好准备。根据我们所做的具体工作，这可能是一个严重的问题，也可能根本不是一个问题。但一般来说，如果我们尝试解析比我们预期的版本更新的Java代码，我们应该做好失败的准备。

## 9. 总结

我们已经了解了如何使用JavaCompiler API解析一些Java源代码并从中获取信息。特别是，我们已经了解了如何从源文件获取组成方法体的各个语句。