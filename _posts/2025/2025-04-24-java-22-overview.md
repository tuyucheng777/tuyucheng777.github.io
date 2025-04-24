---
layout: post
title:  Java 22简介
category: java-new
copyright: java-new
excerpt: Java 22
---

## 1. 简介

在本教程中，我们将深入探讨最新的Java版本[Java 22](https://openjdk.org/projects/jdk/22/)，该版本现已全面推出。

## 2. Java语言更新

让我们来讨论一下此版本中Java语言的所有新变化。

### 2.1 未命名变量和模式-JEP 456

我们经常会定义一些在代码中未使用的临时变量或模式变量，通常，这是由于语言限制，删除它们是被禁止的或会带来副作用。异常、[Switch模式](https://www.baeldung.com/java-switch-pattern-matching)和Lambda表达式就是我们在特定作用域内定义变量或模式但从未使用过它们的例子：
```java
try {
    int number = someNumber / 0;
} catch (ArithmeticException exception) {
    System.err.println("Division by zero");
}

switch (obj) {
    case Integer i -> System.out.println("Is an integer");
    case Float f -> System.out.println("Is a float");
    case String s -> System.out.println("Is a String");
    default -> System.out.println("Default");
}

try (Connection connection = DriverManager.getConnection(url, user, pwd)) {
    LOGGER.info(STR."""
        DB Connection successful
        URL = \{url}
        usr = \{user}
        pwd = \{pwd}""");
} catch (SQLException e) {}
```

**[未命名变量](https://www.baeldung.com/java-unnamed-patterns-variables)(_)非常适合此类场景，它使变量的意图明确，它们不能在代码中传递，也不能使用或赋值**，让我们重写前面的示例：
```java
try {
    int number = someNumber / 0;
} catch (ArithmeticException _) {
    System.err.println("Division by zero");
}

switch (obj) {
    case Integer _ -> System.out.println("Is an integer");
    case Float _ -> System.out.println("Is a float");
    case String _ -> System.out.println("Is a String");
    default -> System.out.println("Default");
}

try (Connection _ = DriverManager.getConnection(url, user, pwd)) {
    LOGGER.info(STR."""
        DB Connection successful
        URL = \{url}
        usr = \{user}
        pwd = \{pwd}""");
} catch (SQLException e) {
    LOGGER.warning("Exception");
}
```

### 2.2 super()之前的语句-JEP 447

长期以来，Java不允许我们在子类的构造函数中调用super()之前放置任何语句，假设我们有一个Shape类系统和两个从Shape类扩展而来的类Square和Circle，在子类构造函数中，第一个语句是对super()的调用：
```java
public class Square extends Shape {
    int sides;
    int length;

    Square(int sides, int length) {
        super(sides, length);
        // some other code
    }
}
```

**当我们需要在调用super()之前执行某些验证时，这种情况会很不方便。在此版本中，这个问题得到了解决**：
```java
public class Square extends Shape {
    int sides;
    int length;

    Square(int sides, int length) {
        if (sides != 4 && length <= 0) {
            throw new IllegalArgumentException("Cannot form Square");
        }
        super(sides, length);
    }
}
```

需要注意的是，我们放在super()之前的语句不能访问实例变量或执行方法。我们可以使用它来执行验证，此外，在调用基类构造函数之前，我们还可以利用这一点来转换派生类中接收到的值。

这是Java的预览功能。

## 3. 字符串模板-JEP 459

Java 22引入了Java流行的字符串模板功能的第二个预览版，[字符串模板](https://www.baeldung.com/java-21-string-templates)允许嵌入文字文本以及表达式和模板处理器以产生特定结果。它们也是其他字符串组合技术的更安全、更高效的替代方案。

在此版本中，字符串模板继续处于预览状态，自第一个预览以来对API进行了小幅更新。**对模板表达式的类型进行了新的更改，以使用模板处理器中相应process()方法的返回类型**。

## 4. 隐式声明的类和实例main方法-JEP 463

**Java现在支持使用其标准模板编写程序，而无需定义显式类或主方法**，我们通常定义类的方式如下：
```java
class MyClass {
    public static void main(String[] args) {
    }
}
```

开发人员现在可以简单地创建一个具有main()方法定义的新文件，如下所示并开始编码：
```java
void main() {
    System.out.println("This is an implicitly declared class without any constructs");

    int x = 165;
    int y = 100;

    System.out.println(y + x);
}
```

**我们可以使用其文件名来编译它，未命名的类位于未命名的包中，而包又位于未命名的模块中**。

## 5. 库

Java 22还带来了新的库并更新了一些现有的库。

### 5.1 外部函数和内存API-JEP 454

Java 22在Project Loom中经过几次孵化器迭代后最终确定了[外部函数](https://www.baeldung.com/java-foreign-memory-access)和内存API，**此API允许开发人员调用外部函数(即JVM生态系统之外的函数)并访问JVM之外的内存**。

它允许我们访问其他运行时和语言的库，这是[JNI](https://www.baeldung.com/jni)(Java本机接口)所做的事情，但效率更高、性能更高、更安全。此JEP为在JVM运行的所有平台上调用本机库提供了更广泛的支持。**此外，该API功能广泛且更易读，并提供了跨多种内存类型(例如堆和瞬态内存)对无限大小的结构化和非结构化数据进行操作的方法**。

**我们将使用新的外部函数和内存API对C的strlen()函数进行本机调用来计算字符串的长度**：
```java
public long getLengthUsingNativeMethod(String string) throws Throwable {
    SymbolLookup stdlib = Linker.nativeLinker().defaultLookup();
    MethodHandle strlen = Linker.nativeLinker()
            .downcallHandle(
                    stdlib.find("strlen").orElseThrow(),
                    of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS));

    try (Arena offHeap = Arena.ofConfined()) {
        MemorySegment str = offHeap.allocateFrom(string);

        long len = (long) strlen.invoke(str);
        System.out.println("Finding String length using strlen function: " + len);
        return len;
    }
}
```

### 5.2 类文件API-JEP 457

**Class File API标准化了读取、解析和转换Java .class文件的过程，此外，它还旨在最终弃用JDK内部的第三方ASM库副本**。

Class File API提供了几个强大的API，可以有选择地转换和修改类中的元素和方法。作为示例，让我们看看如何利用API删除以test_开头的类文件中的方法：
```java
ClassFile cf = ClassFile.of();
ClassModel classModel = cf.parse(PATH);
byte[] newBytes = cf.build(classModel.thisClass()
        .asSymbol(), classBuilder -> {
    for (ClassElement ce : classModel) {
        if (!(ce instanceof MethodModel mm && mm.methodName()
                .stringValue()
                .startsWith(PREFIX))) {
            classBuilder.with(ce);
        }
    }
});
```

这段代码解析源类文件的字节，并通过仅提取满足给定条件的方法(由MethodModel类型表示)对其进行转换，生成的类文件省略了原始类的test_something()方法，可用于验证。

### 5.3 Stream Gatherers-JEP 461

JEP 461通过Stream::gather(Gatherer)在[Stream API](https://www.baeldung.com/java-streams)中支持自定义中间操作，由于内置的流中间操作有限，开发者长期以来一直希望能够支持更多操作。**借助此增强功能，Java允许我们创建自定义中间操作**。

**我们可以通过在流上链接gather()方法并为其提供Gatherer(java.util.stream.Gatherer接口的一个实例)来实现这一点**。

让我们使用Stream Gatherers通过滑动窗口方法将元素列表分成3组：
```java
public List<List<String>> gatherIntoWindows(List<String> countries) {
    List<List<String>> windows = countries
        .stream()
        .gather(Gatherers.windowSliding(3))
        .toList();
    return windows;
}

// Input List: List.of("India", "Poland", "UK", "Australia", "USA", "Netherlands")
// Output: [[India, Poland, UK], [Poland, UK, Australia], [UK, Australia, USA], [Australia, USA, Netherlands]]
```

此预览功能包含5个内置收集器：

- fold
- mapConcurrent
- scan
- windowFixed
- windowSliding

**该API还允许开发人员定义自定义的Gatherer**。

### 5.4 结构化并发-JEP 462

**[结构化并发](https://www.baeldung.com/java-structured-concurrency)API是Java 19中的一项孵化功能，在Java 21中作为预览功能引入，并在Java 22中回归，没有任何新的变化**。

此API的目标是在Java并发任务中引入结构化和协调，结构化并发API旨在通过引入一种编码风格模式来改进并发程序的开发，该模式旨在减少并发编程的常见陷阱和缺点。

**该API简化了错误传播，减少了取消延迟，并提高了可靠性和可观察性**。

### 5.5 作用域值-JEP 464

Java 21引入了[Scoped Values](https://www.baeldung.com/java-20-scoped-values) API作为预览功能以及结构化并发，**该API将原封不动地迁移到Java 22的第二个预览版中**。

**作用域值支持在线程内和线程间存储和共享不可变数据，作用域值引入了一种新类型ScopedValue<\>**，我们只需写入一次值，它们便在整个生命周期内保持不变。

Web请求和服务器代码通常使用ScopedValues，它们被定义为公共静态字段，并允许数据对象在方法之间传递，而无需定义为显式参数。

在下面的示例中，让我们看看如何验证用户并将其上下文作为ScopedValue跨多个实例存储：
```java
private void serve(Request request) {
    User loggedInUser = authenticateUser(request);
    if (loggedInUser) {
        ScopedValue.where(LOGGED_IN_USER, loggedInUser)
                .run(() -> processRequest(request));
    }
}

// In a separate class

private void processRequest(Request request) {
    System.out.println("Processing request" + ScopedValueExample.LOGGED_IN_USER.get());
}
```

**来自唯一用户的多次登录尝试将用户信息范围限定在其唯一线程内**：
```text
Processing request :: User :: 46
Processing request :: User :: 23
```

### 5.6 Vector API(第七次孵化)-JEP 460

**Java 16引入了[Vector API](https://www.baeldung.com/java-vector-api)，而Java 22带来了其第七个孵化版本，此更新提供了性能改进和小更新。以前，向量访问仅限于堆MemorySegments，它们由字节数组支持**，现在已更新为由原始元素类型数组支持。

此更新是低级更新，不会以任何方式影响API的使用。

## 6. 工具更新

Java 22对Java构建文件工具进行了更新。

### 6.1 多文件源程序-JEP 458

Java 11引入了执行单个Java文件而无需使用javac命令显式编译它的功能，这非常高效且快速，缺点是当存在依赖的Java源文件时，我们无法发挥它的优势。

**从Java 22开始，现在可以运行多文件Java程序了**：
```java
public class MainApp {
    public static void main(String[] args) {
        System.out.println("Hello");
        MultiFileExample mm = new MultiFileExample();
        mm.ping(args[0]);
    }
}

public class MultiFileExample {
    public void ping(String s) {
        System.out.println("Ping from Second File " + s);
    }
}
```

我们可以直接运行MainApp，而无需明确运行javac：
```shell
$ java --source 22 --enable-preview MainApp.java "Test"

Hello
Ping from Second File Test
```

请记住以下几点：

- **当类分散在多个源文件中时，编译顺序无法保证**
- **编译主程序引用的类的.java文件**
- 源文件中不允许有重复的类，否则会出错
- 我们可以通过–class-path选项来使用预编译的程序或库

## 7. 性能

Java此次迭代带来的性能更新是对[G1垃圾回收](https://www.baeldung.com/jvm-garbage-collectors)机制的增强。

### 7.1 G1垃圾回收器的区域固定-JEP 423

固定是通知JVM的底层垃圾回收器不要从内存中删除特定对象的过程，**不支持此功能的垃圾回收器通常会暂停垃圾收集，直到JVM收到指示释放关键对象为止**。

这个问题主要出现在JNI临界区中，垃圾回收器中缺少固定功能会影响其延迟、性能和JVM中的总体内存消耗。

**在Java 22中，G1垃圾回收器终于支持区域固定，这样一来，在使用JNI时，Java线程就无需暂停G1 GC**。

## 8. 总结

Java 22为Java带来了大量更新、增强功能和新的预览功能。