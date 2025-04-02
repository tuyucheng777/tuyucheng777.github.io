---
layout: post
title:  Java main()方法解释
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

每个程序都需要一个开始执行的地方；对于Java程序来说，那就是main方法。

我们习惯于在代码会话期间编写main方法，以至于我们甚至不注意它的细节。在这篇简短的文章中，我们将分析此方法并展示其他一些写法。

## 2. 通用签名

最常见的main方法模板是：

```java
public static void main(String[] args) { }
```

这就是我们学习它的方式，也是IDE为我们自动完成代码的方式。但这不是该方法可以采用的唯一形式，我们可以使用一些有效的变体，但并不是每个开发人员都会注意这个事实。

在深入研究这些方法签名之前，让我们回顾一下常见签名中每个关键字的含义：

-   public：访问修饰符，表示全局可见
-   static：方法可以直接从类中访问，我们不必实例化一个对象来获得引用并使用它
-   void：表示此方法不返回值
-   main：方法的名称，这是JVM在执行Java程序时查找的标识符

至于args参数，它表示方法接收到的值，这就是我们在第一次启动程序时将参数传递给程序的方式。

参数args是一个String数组，在以下示例中：

```java
java CommonMainMethodSignature foo bar
```

我们正在执行一个名为CommonMainMethodSignature的Java程序并传递2个参数：foo和bar，这些值可以在main方法内部作为args[0\](以foo为值)和args[1\](以bar为值)访问。

在下一个示例中，我们检查args以决定是加载测试参数还是生产参数：

```java
public static void main(String[] args) {
    if (args.length > 0) {
        if (args[0].equals("test")) {
            // load test parameters
        } else if (args[0].equals("production")) {
            // load production parameters
        }
    }
}
```

永远要记住，IDE也可以向程序传递参数。

## 3. 编写main()方法的不同方式

让我们来看看编写main方法的一些不同方法，尽管它们不是很常见，但它们是有效的签名。

请注意，这些都不是特定于main方法的，它们可以与任何Java方法一起使用，但它们也是main方法的有效部分。

方括号可以放在String附近，就像在通用模板中一样，或者放在两边的args附近：

```java
public static void main(String []args) { }
```

```java
public static void main(String args[]) { }
```

参数可以表示为可变参数：

```java
public static void main(String...args) { }
```

我们甚至可以为main()方法添加strictfp，用于在处理浮点值时实现处理器之间的兼容性：

```java
public strictfp static void main(String[] args) { }
```

synchronized和final也是main方法的有效关键字，但它们在这里不起作用。

另一方面，可以将final应用于args以防止修改数组：

```java
public static void main(final String[] args) { }
```

为了结束这些示例，我们还可以使用上述所有关键字编写main方法(当然，你可能永远不会在实际应用程序中使用)：

```java
final static synchronized strictfp void main(final String[] args) { }
```

## 4. 有多个main()方法

我们还**可以在应用程序中定义多个main方法**。

事实上，有些人将它用作验证单个类的原始测试技术(尽管像JUnit这样的测试框架更适用于此目的)。

为了指定JVM应该执行哪个main方法作为我们应用程序的入口点，我们使用MANIFEST.MF文件。在清单中，我们可以指明主类：

```text
Main-Class: mypackage.ClassWithMainMethod
```

这主要在创建可执行.jar文件时使用，我们通过位于META-INF/MANIFEST.MF(以UTF-8编码)的清单文件指示哪个类具有启动执行的main方法。

## 5. 总结

本教程描述了main方法的细节和它可以采用的一些其他形式，甚至是大多数开发人员不太常见的形式。

请记住，尽管我们展示的所有示例在语法方面都是有效的，但它们只是用于教育目的，大多数时候我们将坚持使用通用签名来完成我们的工作。