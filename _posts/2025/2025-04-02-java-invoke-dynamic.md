---
layout: post
title:  JVM中Invoke Dynamic的介绍
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

Invoke Dynamic(也称为Indy)是[JSR 292](https://jcp.org/en/jsr/detail?id=292)的一部分，旨在增强JVM对动态类型语言的支持。在Java 7中首次发布后，invokedynamic操作码被JRuby等基于JVM的动态语言甚至Java等静态类型语言广泛使用。

在本教程中，我们将揭开invokedynamic的神秘面纱，看看它如何帮助库和语言设计者实现多种形式的动态性。

## 2. 认识Invoke Dynamic

让我们从一个简单的[Stream API](https://www.baeldung.com/java-streams)调用链开始：

```java
public class Main {

    public static void main(String[] args) {
        long lengthyColors = List.of("Red", "Green", "Blue")
                .stream().filter(c -> c.length() > 3).count();
    }
}
```

**起初，我们可能认为Java创建了一个派生自Predicate的匿名内部类，然后将该实例传递给filter方法。但是，我们错了**。

### 2.1 字节码

为了验证这个假设，我们可以看一下生成的字节码：

```shell
javap -c -p Main
// truncated
// class names are simplified for the sake of brevity 
// for instance, Stream is actually java/util/stream/Stream
0: ldc               #7             // String Red
2: ldc               #9             // String Green
4: ldc               #11            // String Blue
6: invokestatic      #13            // InterfaceMethod List.of:(LObject;LObject;)LList;
9: invokeinterface   #19,  1        // InterfaceMethod List.stream:()LStream;
14: invokedynamic    #23,  0        // InvokeDynamic #0:test:()LPredicate;
19: invokeinterface  #27,  2        // InterfaceMethod Stream.filter:(LPredicate;)LStream;
24: invokeinterface  #33,  1        // InterfaceMethod Stream.count:()J
29: lstore_1
30: return
```

尽管我们这么想，但实际上**并不存在匿名内部类**，当然，没有人将此类的实例传递给filter方法。 

令人惊讶的是，invokedynamic指令以某种方式负责创建Predicate实例。

### 2.2 Lambda特定方法

此外，Java编译器还生成了以下看起来很有趣的静态方法：

```shell
private static boolean lambda$main$0(java.lang.String);
    Code:
       0: aload_0
       1: invokevirtual #37                 // Method java/lang/String.length:()I
       4: iconst_3
       5: if_icmple     12
       8: iconst_1
       9: goto          13
      12: iconst_0
      13: ireturn
```

此方法将字符串作为输入，然后执行以下步骤：

-   计算输入长度(invokevirtual on length)
-   将长度与常量3(if_icmple和iconst_3)进行比较
-   如果长度小于或等于3，则返回false

有趣的是，这实际上等同于我们传递给filter方法的Lambda：

```java
c -> c.length() > 3
```

**因此，Java没有创建匿名内部类，而是创建了一个特殊的静态方法，并以某种方式通过invokedynamic调用该方法**。 

在本文中，我们将了解此调用的内部工作原理。但首先，让我们定义invokedynamic试图解决的问题。

### 2.3 问题

在Java 7之前，JVM只有4种方法调用类型：调用普通类方法的invokevirtual，调用静态方法的invokestatic，调用接口方法的invokeinterface，调用构造函数或私有方法的invokespecial。

**尽管存在差异，但所有这些调用都有一个简单的特征：它们有几个预定义的步骤来完成每个方法调用，并且我们无法通过自定义行为来丰富这些步骤**。

此限制有两种主要的解决方法：一种在编译时，另一种在运行时。前者通常由[Scala](https://alidg.me/blog/2020/2/9/java14-records-in-depth#scalas-case-class)或[Koltin](https://alidg.me/blog/2020/2/9/java14-records-in-depth#kotlins-data-class)等语言使用，而后者是JRuby等基于JVM的动态语言的首选解决方案。

运行时方法通常是基于反射的，因此效率低下。

另一方面，编译时解决方案通常依赖于编译时的代码生成。这种方法在运行时更有效，但是，它有点脆弱，并且还可能导致启动时间变慢，因为要处理的字节码更多。

现在我们已经对问题有了更好的理解，让我们看看解决方案在内部是如何工作的。

## 3. 幕后

**invokedynamic让我们以任何我们想要的方式引导方法调用过程**。也就是说，当JVM第一次看到invokedynamic操作码时，它会调用一种称为bootstrap方法的特殊方法来初始化调用过程：

![](/assets/images/2025/javajvm/javainvokedynamic01.png)

bootstrap方法是我们为设置调用过程而编写的一段普通Java代码。因此，它可以包含任何逻辑。

**一旦bootstrap方法正常完成，它应该返回一个[CallSite](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/CallSite.html)的实例，这个CallSite封装了以下信息**：

-   指向JVM应执行的实际逻辑的指针，这应该表示为[MethodHandle](https://www.baeldung.com/java-method-handles)。 
-   表示返回的CallSite有效性的条件。

**从现在开始，每次JVM再次看到这个特定的操作码时，它都会跳过慢速路径，直接调用底层可执行文件**。此外，JVM将继续跳过慢速路径，直到CallSite中的条件发生变化。

相当于反射API，JVM可以完全看穿MethodHandle并且会尝试对其进行优化，因此性能更好。

### 3.1 Bootstrap方法表

我们再看一下生成的invokedynamic字节码：

```text
14: invokedynamic #23,  0  // InvokeDynamic #0:test:()Ljava/util/function/Predicate;
```

**这意味着该特定指令应从bootstrap方法表中调用第一个bootstrap方法(#0部分)**。此外，它还提到了一些要传递给bootstrap方法的参数：

-   test是Predicate中唯一的抽象方法
-   ()Ljava/util/function/Predicate表示JVM中的方法签名：该方法不接收任何输入并返回Predicate接口的实例

为了查看Lambda示例的bootstrap方法表，我们应该将-v选项传递给javap：

```shell
javap -c -p -v Main
// truncated
// added new lines for brevity
BootstrapMethods:
  0: #55 REF_invokeStatic java/lang/invoke/LambdaMetafactory.metafactory:
    (Ljava/lang/invoke/MethodHandles$Lookup;
     Ljava/lang/String;
     Ljava/lang/invoke/MethodType;
     Ljava/lang/invoke/MethodType;
     Ljava/lang/invoke/MethodHandle;
     Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #62 (Ljava/lang/Object;)Z
      #64 REF_invokeStatic Main.lambda$main$0:(Ljava/lang/String;)Z
      #67 (Ljava/lang/String;)Z
```

所有Lambda的bootstrap方法都是[LambdaMetafactory](https://github.com/openjdk/jdk/blob/a445b66e58a30577dee29cacb636d4c14f0574a2/src/java.base/share/classes/java/lang/invoke/LambdaMetafactory.java#L315)类中的[metafactory](https://github.com/openjdk/jdk/blob/a445b66e58a30577dee29cacb636d4c14f0574a2/src/java.base/share/classes/java/lang/invoke/LambdaMetafactory.java#L227)静态方法。

**与所有其他bootstrap方法类似，这个方法至少需要3个参数**，如下所示：

-   Ljava/lang/invoke/MethodHandles$Lookup参数表示invokedynamic的查找上下文
-   Ljava/lang/String表示调用站点中的方法名称-在这个例子中，方法名称是test
-   Ljava/lang/invoke/MethodType是调用站点的动态方法签名-在本例中，它是()Ljava/util/function/Predicate

**除了这3个参数之外，bootstrap方法还可以选择性地接收一个或多个额外参数**。在这个例子中，这些是额外的：

-   (Ljava/lang/Object;)Z是一个被[擦除](https://www.baeldung.com/java-type-erasure)的方法签名，它接收一个Object实例并返回一个布尔值。
-   REF_invokeStatic Main.lambda\$main\$0:(Ljava/lang/String;)Z是指向实际Lambda逻辑的MethodHandle。
-   (Ljava/lang/String;)Z是一个非擦除的方法签名，它接收一个字符串并返回一个布尔值。

**简而言之，JVM会将所有需要的信息传递给bootstrap方法，bootstrap方法将依次使用该信息创建适当的Predicate实例**。然后，JVM将该实例传递给filter方法。

### 3.2 不同类型的CallSite

一旦JVM第一次看到此示例中的invokedynamic，它就会调用bootstrap方法。在撰写本文时，**Lambda bootstrap方法将使用[InnerClassLambdaMetafactory](https://github.com/openjdk/jdk/blob/a445b66e58a30577dee29cacb636d4c14f0574a2/src/java.base/share/classes/java/lang/invoke/InnerClassLambdaMetafactory.java#L194)在运行时为Lambda生成内部类**。 

然后bootstrap方法将生成的内部类封装在一种称为[ConstantCallSite](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/ConstantCallSite.html)的特殊类型的CallSite中，这种类型的CallSite在设置后永远不会改变。**因此，在为每个Lambda首次设置后，JVM将始终使用快速路径直接调用Lambda逻辑**。

尽管这是最有效的invokedynamic类型，但它肯定不是唯一可用的选项。事实上，Java提供了[MutableCallSite](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/MutableCallSite.html)和[VolatileCallSite](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/VolatileCallSite.html)以适应更动态的需求。

### 3.3 优点

因此，为了实现Lambda表达式，Java不是在编译时创建匿名内部类，而是在运行时通过invokedynamic创建它们。

有人可能会反对将内部类的生成推迟到运行时。但是，与简单的编译时解决方案相比，invokedynamic方法有一些优势。

首先，JVM在第一次使用Lambda之前不会生成内部类。因此，在第一次执行Lambda之前，**我们无需为与内部类相关的额外占用空间付出代价**。

此外，许多链接逻辑都从字节码移到了bootstrap方法中。因此，**invokedynamic字节码通常比替代解决方案小得多**。较小的字节码可以提高启动速度。

假设更新版本的Java带有更高效的bootstrap方法实现，**那么我们的invokedynamic字节码就可以利用这一改进而无需重新编译**，这样我们就可以实现某种转发二进制兼容性。基本上，我们可以在不同的策略之间切换而无需重新编译。

最后，用Java编写bootstrap和链接逻辑通常比遍历AST生成一段复杂的字节码更容易。因此，invokedynamic可以(主观上)不那么脆弱。

## 4. 更多示例

Lambda表达式并不是唯一的特性，Java当然也不是唯一使用invokedynamic的语言。在本节中，我们将熟悉一些InvokeDynamic的其他示例。

### 4.1 Java 14：记录

[记录](https://www.baeldung.com/java-record-keyword)是[Java 14](https://openjdk.java.net/projects/jdk/14/)中的一项新预览功能，它提供了一种简洁的语法来声明应该是哑数据持有者的类。

这是一个简单的记录示例：

```java
public record Color(String name, int code) {}
```

给定这个简单的单行代码，Java编译器会为访问器方法、toString、equals和hashcode生成适当的实现。 

**为了实现toString、equals或hashcode，Java使用invokedynamic**。例如，equals的字节码如下：

```text
public final boolean equals(java.lang.Object);
    Code:
       0: aload_0
       1: aload_1
       2: invokedynamic #27,  0  // InvokeDynamic #0:equals:(LColor;Ljava/lang/Object;)Z
       7: ireturn
```

另一种解决方案是找到所有记录字段，并在编译时根据这些字段生成equals逻辑。**字段越多，字节码就越长**。

相反，Java在运行时调用bootstrap方法来链接适当的实现。因此，**无论字段数量有多少，字节码长度都会保持不变**。

仔细查看字节码可以发现，bootstrap方法是[ObjectMethods#bootstrap](https://github.com/openjdk/jdk/blob/827e5e32264666639d36990edd5e7d0b7e7c78a9/src/java.base/share/classes/java/lang/runtime/ObjectMethods.java#L338)：

```text
BootstrapMethods:
  0: #42 REF_invokeStatic java/lang/runtime/ObjectMethods.bootstrap:
    (Ljava/lang/invoke/MethodHandles$Lookup;
     Ljava/lang/String;
     Ljava/lang/invoke/TypeDescriptor;
     Ljava/lang/Class;
     Ljava/lang/String;
     [Ljava/lang/invoke/MethodHandle;)Ljava/lang/Object;
    Method arguments:
      #8 Color
      #49 name;code
      #51 REF_getField Color.name:Ljava/lang/String;
      #52 REF_getField Color.code:I
```

### 4.2 Java 9：字符串拼接

在Java 9之前，重要的字符串拼接是使用StringBuilder实现的。作为[JEP 280](https://openjdk.java.net/jeps/280)的一部分，字符串拼接现在使用invokedynamic。 例如，让我们拼接一个常量字符串和一个随机变量：

```java
"random-" + ThreadLocalRandom.current().nextInt();
```

以下是此示例的字节码：

```text
0: invokestatic  #7          // Method ThreadLocalRandom.current:()LThreadLocalRandom;
3: invokevirtual #13         // Method ThreadLocalRandom.nextInt:()I
6: invokedynamic #17,  0     // InvokeDynamic #0:makeConcatWithConstants:(I)LString;
```

此外，字符串拼接的bootstrap方法位于[StringConcatFactory](https://github.com/openjdk/jdk/blob/827e5e32264666639d36990edd5e7d0b7e7c78a9/src/java.base/share/classes/java/lang/invoke/StringConcatFactory.java#L593)类中：

```text
BootstrapMethods:
  0: #30 REF_invokeStatic java/lang/invoke/StringConcatFactory.makeConcatWithConstants:
    (Ljava/lang/invoke/MethodHandles$Lookup;
     Ljava/lang/String;
     Ljava/lang/invoke/MethodType;
     Ljava/lang/String;
     [Ljava/lang/Object;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #36 random-\u0001
```

## 5. 总结

在本文中，我们首先熟悉了Indy试图解决的问题。

然后，通过一个简单的Lambda表达式示例，我们了解了invokedynamic的内部工作原理。

最后，我们列举了最近几个Java版本中indy的其他几个例子。