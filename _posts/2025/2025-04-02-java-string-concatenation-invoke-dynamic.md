---
layout: post
title:  使用Invoke Dynamic进行字符串拼接
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

编译器和运行时倾向于优化一切，即使是最小且看似不太重要的部分。当谈到这些优化时，JVM和Java有很多优势。

在本文中，我们将评估其中一种相对较新的优化：使用invokedynamic进行[字符串拼接](https://www.baeldung.com/java-strings-concatenation)。

## 2. Java 9之前

在Java 9之前，重要的字符串拼接是使用[StringBuilder](https://www.baeldung.com/java-string-builder-string-buffer)实现的。例如，让我们考虑以下方法：

```java
String concat(String s, int i) {
    return s + i;
}
```

这个简单代码的字节码如下(使用javap -c)：

```text
java.lang.String concat(java.lang.String, int);
  Code:
     0: new           #2      // class StringBuilder
     3: dup
     4: invokespecial #3      // Method StringBuilder."<init>":()V
     7: aload_0
     8: invokevirtual #4      // Method StringBuilder.append:(LString;)LStringBuilder;
    11: iload_1
    12: invokevirtual #5      // Method StringBuilder.append:(I)LStringBuilder;
    15: invokevirtual #6      // Method StringBuilder.toString:()LString;
```

在这里，Java 8编译器使用StringBuilder拼接方法输入，即使我们没有在代码中使用StringBuilder。

公平地说，**使用StringBuilder拼接字符串非常高效且经过精心设计**。

让我们看看Java 9如何改变这个实现以及这种改变的动机是什么。

## 3. Invoke Dynamic

从Java 9开始，作为[JEP 280](https://openjdk.java.net/jeps/280)的一部分，字符串拼接现在使用[invokedynamic](https://www.baeldung.com/java-invoke-dynamic)。

**更改背后的主要动机是实现更动态的实现**，也就是说，可以在不更改字节码的情况下更改拼接策略。这样，客户端无需重新编译即可从新的优化策略中获益。

还有其他优点。例如，invokedynamic的字节码更优雅、更稳定、更小。

### 3.1 总体情况

在深入了解这种新方法的工作原理之前，让我们从更广泛的角度来看它。

例如，假设我们要通过将另一个String与int拼接来创建一个新的String，**我们可以将其视为接收String和int然后返回拼接后的String的函数**。

对于此示例，新方法的工作方式如下：

-   准备描述拼接的函数签名，例如(String, int) -> String
-   准备拼接的实际参数，例如，如果我们要拼接“The answer is ”和42，那么这些值将是参数
-   调用bootstrap方法并将函数签名、参数和一些其他参数传递给它
-   为该函数签名生成实际实现并将其封装在MethodHandle中
-   调用生成的函数来创建最终的拼接字符串

![](/assets/images/2025/javajvm/javastringconcatenationinvokedynamic01.png)

简而言之，**字节码在编译时定义了规范。然后bootstrap方法在运行时将实现链接到该规范**，这反过来又可以在不触及字节码的情况下更改实现。

在本文中，我们将揭示与每个步骤相关的细节。

首先，让我们看看与bootstrap方法的链接是如何工作的。

## 4. 链接

让我们看看Java 9+编译器如何为相同的方法生成字节码：

```text
java.lang.String concat(java.lang.String, int);
  Code:
     0: aload_0
     1: iload_1
     2: invokedynamic #7,  0   // InvokeDynamic #0:makeConcatWithConstants:(LString;I)LString;
     7: areturn
```

**与简单的StringBuilder方法相反，该方法使用的指令数量明显少得多**。

在这个字节码中，(LString;I)LString签名非常有趣，它需要一个String和一个int(I代表int)并返回拼接的字符串。这是因为该方法将一个String和一个int拼接在一起。

**与其他invokedynamic实现类似，大部分逻辑从编译时移到运行时**。

为了查看运行时逻辑，让我们检查bootstrap方法表(使用javap -c -v)：

```text
BootstrapMethods:
  0: #25 REF_invokeStatic java/lang/invoke/StringConcatFactory.makeConcatWithConstants:
    (Ljava/lang/invoke/MethodHandles$Lookup;
     Ljava/lang/String;
     Ljava/lang/invoke/MethodType;
     Ljava/lang/String;
     [Ljava/lang/Object;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #31 \u0001\u0001
```

在这种情况下，当JVM第一次看到invokedynamic指令时，它会调用[makeConcatWithConstants](https://github.com/openjdk/jdk14u/blob/8c9ab998b758a18e65e2a1cebcc608860ae43931/src/java.base/share/classes/java/lang/invoke/StringConcatFactory.java#L593) bootstrap方法，bootstrap方法又会返回一个指向拼接逻辑的[ConstantCallSite](https://github.com/openjdk/jdk14u/blob/master/src/java.base/share/classes/java/lang/invoke/ConstantCallSite.java)。

![](/assets/images/2025/javajvm/javastringconcatenationinvokedynamic02.png)

在传递给bootstrap方法的参数中，有两个尤为突出：

-   Ljava/lang/invoke/MethodType表示字符串拼接签名，在本例中，它是(LString;I)LString，因为我们将整数与字符串拼接
-   \u0001\u0001是构造字符串的配方(稍后会详细介绍)

## 5. 配方

为了更好地理解配方的作用，让我们考虑一个简单的数据类：

```java
public class Person {

    private String firstName;
    private String lastName;

    // constructor

    @Override
    public String toString() {
        return "Person{" +
                "firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                '}';
    }
}
```

为了生成字符串表示，JVM将firstName和lastName字段作为参数传递给invokedynamic指令：

```text
 0: aload_0
 1: getfield      #7        // Field firstName:LString;
 4: aload_0
 5: getfield      #13       // Field lastName:LString;
 8: invokedynamic #16,  0   // InvokeDynamic #0:makeConcatWithConstants:(LString;LString;)L/String;
 13: areturn
```

这一次，bootstrap方法表看起来有点不同：

```text
BootstrapMethods:
  0: #28 REF_invokeStatic StringConcatFactory.makeConcatWithConstants // truncated
    Method arguments:
      #34 Person{firstName=\'\u0001\', lastName=\'\u0001\'} // The recipe
```

如上所示，**配方表示拼接字符串的基本结构**。例如，前面的配方包括：

-   常量字符串，例如“Person”，这些文字值将按原样出现在拼接的字符串中
-   两个\u0001标签来表示普通参数，它们将被实际参数替换，例如firstName

我们可以将配方视为包含静态部分和变量占位符的模板字符串。

**使用配方可以显著减少传递给bootstrap方法的参数数量，因为我们只需要传递所有动态参数和一个配方**。

## 6. 字节码风格

新的拼接方法有两种字节码风格，到目前为止，我们熟悉一种风格：调用makeConcatWithConstants bootstrap方法并传递配方。**这种风格被称为使用常量的indy，是Java 9的默认风格**。

**第二种风格不使用配方，而是将所有内容作为参数传递**。也就是说，它不区分常量部分和动态部分，并将它们全部作为参数传递。

**要使用第二种风格，我们应该将-XDstringConcat=indy选项传递给Java编译器**。例如，如果我们用这个标志编译同一个Person类，那么编译器会生成以下字节码：

```text
public java.lang.String toString();
    Code:
       0: ldc           #16      // String Person{firstName=\'
       2: aload_0
       3: getfield      #7       // Field firstName:LString;
       6: bipush        39
       8: ldc           #18      // String , lastName=\'
      10: aload_0
      11: getfield      #13      // Field lastName:LString;
      14: bipush        39
      16: bipush        125
      18: invokedynamic #20,  0  // InvokeDynamic #0:makeConcat:(LString;LString;CLString;LString;CC)LString;
      23: areturn
```

这一次，bootstrap方法是[makeConcat](https://github.com/openjdk/jdk14u/blob/8c9ab998b758a18e65e2a1cebcc608860ae43931/src/java.base/share/classes/java/lang/invoke/StringConcatFactory.java#L472)。此外，拼接签名有7个参数，每个参数代表toString的一部分：

-   第一个参数表示firstName变量之前的部分-“Person{firstName=\'”字面量 
-   第二个参数是firstName字段的值
-   第三个参数是单引号字符
-   第四个参数是下一个变量之前的部分-“，lastName=\'”
-   第五个参数是lastName字段
-   第六个参数是单引号字符
-   最后一个参数是右大括号

这样，bootstrap方法就有足够的信息来链接适当的拼接逻辑。

非常有趣的是，**还可以回到Java 9之前的世界并使用带有-XDstringConcat=inline编译器选项的StringBuilder**。

## 7. 策略

**bootstrap方法最终提供了指向实际拼接逻辑的MethodHandle**，在撰写本文时，有6种不同的[策略](https://github.com/openjdk/jdk14u/blob/8c9ab998b758a18e65e2a1cebcc608860ae43931/src/java.base/share/classes/java/lang/invoke/StringConcatFactory.java#L136)可以生成此逻辑：

-   BC_SB或“字节码StringBuilder”策略在运行时生成相同的StringBuilder字节码，然后它通过Unsafe.defineAnonymousClass方法加载生成的字节码。
-   BC_SB_SIZED策略将尝试猜测StringBuilder的必要容量，除此之外，它与以前的方法相同。猜测容量可能有助于StringBuilder在不调整底层byte[]大小的情况下执行拼接。
-   BC_SB_SIZED_EXACT是一个基于StringBuilder的字节码生成器，可以精确计算所需的存储空间。要计算确切的大小，首先，它将所有参数转换为String。
-   MH_SB_SIZED基于MethodHandle，最终调用StringBuilder API进行拼接，该策略还对所需容量进行了有根据的猜测。
-   MH_SB_SIZED_EXACT与前一个类似，只是它能够完全准确地计算所需的容量。
-   MH_INLINE_SIZE_EXACT预先计算所需的存储空间并直接维护其byte[]以存储拼接结果。这个策略是内联的，因为它复制了StringBuilder在内部所做的事情。

**默认策略是MH_INLINE_SIZE_EXACT，但是，我们可以使用-Djava.lang.invoke.stringConcat=<strategyName\>系统属性更改此策略**。 

## 8. 总结

在这篇详细的文章中，我们了解了新的字符串拼接是如何实现的，以及使用这种方法的优势。