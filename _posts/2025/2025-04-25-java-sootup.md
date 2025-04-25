---
layout: post
title:  SootUp简介
category: staticanalysis
copyright: staticanalysis
excerpt: SootUp
---

## 1. 简介

在本文中，我们将探讨[SootUp](https://soot-oss.github.io/SootUp/latest/)库。SootUp是一个用于对JVM代码进行静态分析的库，可以使用原始源代码或编译后的JVM字节码，它是对[Soot库](https://github.com/soot-oss/soot)的全面改造，旨在使其更加模块化、可测试、可维护且易于使用。

## 2. 依赖

**在使用SootUp之前，我们需要在我们的构建中包含[最新版本](https://mvnrepository.com/artifact/org.soot-oss)，在撰写本文时为[1.3.0](https://mvnrepository.com/artifact/org.soot-oss/sootup.core/1.3.0)**。

```xml
<dependency>
    <groupId>org.soot-oss</groupId>
    <artifactId>sootup.core</artifactId>
    <version>1.3.0</version>
</dependency>
<dependency>
    <groupId>org.soot-oss</groupId>
    <artifactId>sootup.java.core</artifactId>
    <version>1.3.0</version>
</dependency>
<dependency>
    <groupId>org.soot-oss</groupId>
    <artifactId>sootup.java.sourcecode</artifactId>
    <version>1.3.0</version>
</dependency>
<dependency>
    <groupId>org.soot-oss</groupId>
    <artifactId>sootup.java.bytecode</artifactId>
    <version>1.3.0</version>
</dependency>
<dependency>
    <groupId>org.soot-oss</groupId>
    <artifactId>sootup.jimple.parser</artifactId>
    <version>1.3.0</version>
</dependency>
```

这里我们有几个不同的依赖，那么它们都起什么作用呢？

- org.soot-uss:sootup.core是核心库。
- org.soot-uss:sootup.java.core是使用Java的核心模块。
- org.soot-uss:sootup.java.sourcecode是分析Java源代码的模块。
- org.soot-uss:sootup.java.bytecode是用于分析编译后的Java字节码的模块。
- org.soot-uss:sootup.jimple.parser是用于解析[Jimple](https://soot-oss.github.io/SootUp/v1.3.0/jimple/)的模块-SootUp用于表示Java的中间表示。

不幸的是，没有可用的[BOM](https://www.baeldung.com/spring-maven-bom)依赖，因此我们需要单独管理这些依赖的每个版本。

## 3. 什么是Jimple？

**SootUp可以分析多种不同格式的代码-包括Java源代码、编译的字节码，甚至是JVM内部的类**。

为此，**它将各种输入转换为称为Jimple的中间表示**。

Jimple的存在是为了表示所有可以用Java源代码或字节码实现的功能，但以一种更易于分析的方式，这意味着它在某些方面刻意与这两种可能的输入有所不同。

JVM字节码的某些值访问方式是基于栈的，这在运行时非常高效，但在分析方面却非常困难。Jimple的代码表示将其转换为完全基于变量的方式，这样可以实现完全相同的功能，同时更容易理解。

相反，Java源代码也是基于变量的，但其嵌套结构也使其更难分析，这对于开发人员来说更容易处理，但对于软件工具来说更难分析，Jimple将其表示转换为扁平结构。

Jimple也作为一种语言存在，我们可以自己读写代码。例如，Java源代码：

```java
public void demoMethod() {
    System.out.println("Inside method.");
}
```

也可以写成Jimple形式，如下所示：

```java
public void demoMethod() {
    java.io.PrintStream $stack1;
    target.exercise1.DemoClass this;

    this := @this: target.exercise1.DemoClass;
    $stack1 = <java.lang.System: java.io.PrintStream out>;

    virtualinvoke $stack1.<java.io.PrintStream: void println(java.lang.String)>("Inside method.");
    return;
}
```

这看起来更冗长，但我们可以看到它具有相同的功能，如果我们需要以这种格式存储和转换代码，SootUp提供了直接解析和生成此Jimple代码的功能。

当我们分析代码时，无论原始代码是什么，它都会被转换成这种结构以供我们使用。然后，我们将处理与此结构直接相关的类型，例如SootClass、SootField、SootMethod等。

## 4. 分析代码

**在使用SootUp进行任何操作之前，我们需要分析一些代码，具体方法是创建一个AnalysisInputLocation的实例，并围绕它构建一个JavaView**。

我们创建的AnalysisInputLocation的具体类型取决于我们想要分析的代码的来源。

最简单易用，但可能本身用处最小的，就是能够分析JVM本身的类，我们可以使用JrtFileSystemAnalysisInputLocation类来实现这一点：

```java
AnalysisInputLocation inputLocation = new JrtFileSystemAnalysisInputLocation();
```

更有用的是，**我们可以使用OTFCompileAnalysisInputLocation分析源文件**：

```java
AnalysisInputLocation inputLocation = new OTFCompileAnalysisInputLocation(
    Path.of("src/test/java/cn/tuyucheng/taketoday/sootup/AnalyzeUnitTest.java"));
```

这也有一个替代构造函数，用于一次性分析整个源文件列表：

```java
AnalysisInputLocation inputLocation = new OTFCompileAnalysisInputLocation(List.of(.....));
```

我们还可以使用它来分析内存中作为字符串的源代码：

```java
Path javaFile = Path.of("src/test/java/cn/tuyucheng/taketoday/sootup/AnalyzeUnitTest.java");
String javaContents = Files.readString(javaFile);

AnalysisInputLocation inputLocation = new OTFCompileAnalysisInputLocation("AnalyzeUnitTest.java", javaContents);
```

**最后，我们可以分析已经编译好的字节码，这是使用JavaClassPathAnalysisInputLocation完成的，我们可以将其指向任何可以被视为类路径的内容-包括JAR文件或包含类文件的目录**。

```java
AnalysisInputLocation inputLocation = new JavaClassPathAnalysisInputLocation("target/classes");
```

还有其他几种标准方法可以访问我们想要分析的代码，包括直接解析Jimple表示或读取Android APK文件。

一旦我们获得了AnalysisInputLocation实例，我们就可以围绕它创建一个JavaView：

```java
JavaView view = new JavaView(inputLocation);
```

这样我们就可以访问输入中存在的所有类型。

## 5. 访问类

**一旦我们分析了代码并围绕它构建了JavaView实例，我们就可以开始访问代码的详细信息了，首先要访问类**。

如果我们知道我们想要的确切类名，我们可以直接使用完全限定的类名来访问它。SootUp使用各种Signature类来描述我们想要访问的元素，在这种情况下，我们需要一个ClassType实例，幸运的是，我们可以使用SootUp提供的IdentifierFactory轻松生成一个完全限定的类名：

```java
IdentifierFactory identifierFactory = view.getIdentifierFactory();
ClassType javaClass = identifierFactory.getClassType("cn.tuyucheng.taketoday.sootup.ClassUnitTest");
```

一旦我们构建了ClassType实例，我们就可以使用它来访问此类的详细信息：

```java
Optional<JavaSootClass> sootClass = view.getClass(javaClass);
```

这里返回一个Optional<JavaSootClass\>，因为这个类可能在我们的视图中不存在。或者，我们有一个getClassOrThrow()方法，它直接返回一个SootClass-JavaSootClass的超类，但如果这个类在我们的JavaView中不存在，就会抛出异常：

```java
SootClass sootClass = view.getClassOrThrow(javaClass);
```

**一旦我们得到了SootClass实例，我们就可以用它来检查类的细节**。这让我们能够确定类本身的细节，比如它的可见性、它是具体类还是抽象类等等：

```java
assertTrue(classUnitTest.isPublic());
assertTrue(classUnitTest.isConcrete());
assertFalse(classUnitTest.isFinal());
assertFalse(classUnitTest.isEnum());
```

我们还可以导航已解析的代码，例如通过访问类的超类或接口：

```java
Optional<? extends ClassType> superclass = sootClass.getSuperclass();
Set<? extends ClassType> interfaces = sootClass.getInterfaces();
```

注意，这些方法返回的是ClassType而不是SootClass实例，这是因为无法保证实际的类定义是我们视图的一部分，而只是类的名称。

## 6. 访问字段和方法

**除了类本身之外，我们还可以访问类的内容，例如字段和方法**。

如果我们已经有一个可用的SootClass，那么我们可以直接查询它来找到字段和方法：

```java
Set<? extends SootField> fields = sootClass.getFields();
Set<? extends SootMethod> methods = sootClass.getMethods();
```

与我们从一个类导航到另一个类不同，这可以安全地返回字段或方法的整个表示，因为它们保证在我们的视图中。

如果我们确切知道要查找的内容，也可以直接访问它。例如，要访问某个字段，我们只需要知道它的名称：

```java
Optional<? extends SootField> field = sootClass.getField("aField");
```

访问方法稍微复杂一些，因为我们需要知道方法名称和参数类型：

```java
Optional<? extends SootMethod> method = sootClass.getMethod("someMethod", List.of());
```

如果我们的方法需要参数，那么我们需要从IdentifierFactory中提供一个Type实例列表：

```java
Optional<? extends SootMethod> method = sootClass.getMethod("anotherMethod",
    List.of(identifierFactory.getClassType("java.lang.String")));
```

这样，当我们有重载方法时，就可以获取正确的实例。我们还可以列出所有同名的重载方法：

```java
Set<? extends SootMethod> method = sootClass.getMethodsByName("someMethod");
```

和以前一样，一旦我们获得了SootMethod或SootField实例，我们就可以使用它来检查详细信息：

```java
assertTrue(sootMethod.isPrivate());
assertFalse(sootMethod.isStatic());
```

## 7. 分析方法主体

**一旦我们获得了SootMethod实例，我们就可以用它来分析方法体本身，这意味着方法签名、方法中的局部变量以及调用图本身**。

在我们做任何这些之前，我们需要访问方法主体本身：

```java
Body methodBody = sootMethod.getBody();
```

使用这个，我们现在可以访问方法主体的所有细节。

### 7.1 访问局部变量

**我们可以做的第一件事是访问方法中可用的任何局部变量**：

```java
Set<Local> methodLocals = methodBody.getLocals();
```

这使我们能够访问方法中可访问的所有变量，此列表可能并非预期的那样，它实际上是来自该方法的Jimple表示的变量列表，因此会包含解析过程中的一些额外条目，并且可能不包含原始变量名。

例如，以下方法有5个局部变量：

```java
private void someMethod(String name) {
    var capitals = name.toUpperCase();
    System.out.println("Hello, " + capitals);
}
```

这些都是：

- this
- I1：方法参数。
- I2：变量“capitals”。
- $stack3：指向System.out的局部变量。
- $stack4：表示“Hello, ” + capitals的局部变量。

\$stack3和\$stack4局部变量由Jimple表示生成，并不直接存在于原始代码中。

### 7.2 访问方法语句图

**除了局部变量之外，我们还可以分析整个方法语句图，这是该方法将执行的每个语句的详细信息**：

```java
StmtGraph<?> stmtGraph = methodBody.getStmtGraph();
List<Stmt> stmts = stmtGraph.getStmts();
```

这给出了该方法将执行的所有语句的列表，按执行顺序排列，每个语句都将实现Stmt接口，表示该方法可以执行的操作。

例如，我们之前的方法将产生这样的结果：

![](/assets/images/2025/staticanalysis/javasootup01.png)

**这看起来比我们实际写的代码要多得多——实际代码只有两行，这是因为这是我们代码的Jimple表示**，但我们可以分解一下，看看到底发生了什么。

我们从两个JIdentityStmt实例开始，它们代表传递给我们方法的值-this值和我们之前看到的作为第一个参数的I1。

接下来，我们有三个JAssignStmt实例，它们表示对方法中变量的赋值。在本例中，我们将I1.toUpperCase()的结果赋值给I2，将System.out的值赋值给\$stack3，并将“Hello, ” + I2的结果赋值给\$stack4。

此后，我们得到了一个JInvokeStmt实例，这表示调用\$stack3上的println()方法，并将\$stack4的值传递给它。

最后，我们有一个JReturnVoidStmt实例，它表示方法结束时的隐式返回。

这是一个非常简单的方法，没有分支或控制语句，但我们可以清楚地看到，该方法所做的所有操作都在这里体现。对于我们在Java应用程序中可以实现的任何功能，情况也是如此。

## 8. 总结

以上是对SootUp的简要介绍，这个库还能发挥更多功能。