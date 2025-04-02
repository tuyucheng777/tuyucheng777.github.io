---
layout: post
title:  Java中的“Code too large”编译错误
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述 

**当Java方法超过65535字节时，我们会得到编译错误“code too large”**。在本文中，我们将讨论为什么会出现此错误以及如何修复它。

## 2. JVM约束

[Code_attribute](https://docs.oracle.com/javase/specs/jvms/se16/html/jvms-4.html#jvms-4.7.3)是JVM规范的method_info结构中的一个可变长度的表，此结构包含方法的JVM指令，该方法可以是常规方法或实例、类或接口的初始化方法：

```text
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    {   
        u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    }
    exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

属性code_length指定方法中代码的长度：

```text
code_length
The value of the code_length item gives the number of bytes in the code array for this method.
The value of code_length must be greater than zero (as the code array must not be empty) and less than 65536.
```

从上面可以看出，**JVM规范规定方法的代码长度必须小于65536字节，因此这意味着方法的大小不能超过65535字节**。

## 3. 为什么会出现问题

现在我们知道了方法的大小限制，让我们看看可能导致如此大的方法的情况：

-   代码生成器：大多数大型方法都是使用某些代码生成器(如[ANTLR解析器](https://www.baeldung.com/java-antlr))的结果
-   初始化方法：GUI初始化可能添加许多细节，如布局、事件监听器等，所有这些都在一个方法中
-   JSP页面：包含类的单个方法中的所有代码
-   [代码检测](https://www.baeldung.com/java-instrumentation)：在运行时将字节码添加到已编译的类中
-   数组初始化器：初始化非常大的数组的方法，如下所示：

```java
String[][] largeStringArray = new String[][] {
        { "java", "code", "exceeded", "65355", "bytes" },
        { "alpha", "beta", "gamma", "delta", "epsilon" },
        { "one", "two", "three", "four", "five" },
        { "uno", "dos", "tres", "cuatro", "cinco" },

        //More values
};
```

## 4. 如何修复错误

正如我们所指出的，错误的根本原因是方法超过了65535字节的阈值。因此，**将出错的方法重构为几个更小的方法将为我们解决问题**。

对于数组初始化，我们可以拆分数组或从文件加载，我们也可以使用[静态初始化器](https://www.baeldung.com/java-initialization#2-static-initialization-block)。即使我们使用代码生成器，仍然可以重构代码。对于大型JSP文件，我们可以使用jsp:include指令并将其分解为更小的单元。

上面的问题都比较容易处理，但是**当我们在代码中添加检测后出现“code too large”错误时，事情就变得复杂了**。如果我们拥有代码，我们仍然可以重构该方法。但是当我们从第三方库中得到这个错误时，我们就陷入了困境。通过降低检测级别，我们也许能够解决这个问题。

## 5. 总结

在本文中，我们讨论了“code too large”错误的原因和可能的解决方案，我们始终可以参考[JVM规范的Code_Attributes部分](https://docs.oracle.com/javase/specs/jvms/se16/html/jvms-4.html#jvms-4.7.3)来找到关于这个约束的更多细节。