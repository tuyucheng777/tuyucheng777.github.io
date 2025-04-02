---
layout: post
title:  Java中的构造函数返回类型
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本快速教程中，我们将重点关注Java中构造函数的返回类型。

首先，我们将熟悉Java和JVM中对象初始化的工作原理。然后，我们将深入了解对象初始化和赋值的底层工作原理。

## 2. 实例初始化

让我们从一个空类开始：

```java
public class Color {}
```

在这里，我们将从此类创建一个实例并将其分配给某个变量：

```java
Color color = new Color();
```

编译完这个简单的Java代码片段后，让我们通过javap -c命令看一下它的字节码：

```text
0: new           #7                  // class Color
3: dup
4: invokespecial #9                  // Method Color."<init>":()V
7: astore_1
```

当我们在Java中实例化一个对象时，JVM会执行以下操作：

1.  首先，它在其进程空间中为新对象[找到一块空间](https://alidg.me/blog/2019/6/21/tlab-jvm)。
2.  然后，JVM执行系统初始化过程；在此步骤中，它以默认状态创建对象。字节码中的new操作码实际上负责这一步。
3.  最后，它使用构造函数和其他初始化块初始化对象；在这种情况下，invokespecial操作码调用构造函数。

如上所示，默认构造函数的方法签名为：

```text
Method Color."<init>":()V
```

**<init\>是JVM中[实例初始化方法](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.9)的名称**。在本例中，<init\>是一个函数：

-   不接收任何输入(方法名称后的空括号)
-   不会将任何值推送到操作数栈(由V表示，通常与void关联)

总之，虽然构造函数的字节码表示显示返回描述符为V，但说Java中的构造函数具有void返回类型是不准确的。相反，**Java中的构造函数根本没有返回类型**。

那么，再看一下我们的简单任务：

```java
Color color = new Color();
```

现在我们知道了构造函数是如何工作的，让我们看看赋值是如何工作的。

## 3. 任务分配如何进行

JVM是一个基于栈的虚拟机，每个栈由[栈帧](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html#jvms-2.6)组成。简单来说，每个栈帧对应一个方法调用。事实上，JVM使用新的方法调用创建栈帧，并在完成工作后销毁它们：

![](/assets/images/2025/javajvm/javaconstructorreturntype01.png)

每个栈帧使用一个数组来存储局部变量和一个操作数栈来存储部分结果。鉴于此，让我们再看一下字节码：

```text
0: new           #7                // class Color
3: dup
4: invokespecial #9               // Method Color."<init>":()V
7: astore_1
```

任务的执行方式如下：

-   new指令创建Color实例并将其引用压入操作数栈
-   dup操作码复制操作数栈中的最后一项
-   invokespecial获取重复的引用并将其用于初始化，在此之后，操作数栈上只剩下原始引用
-   astore_1存储对局部变量数组索引1的原始引用，前缀“a”表示要存储的项是对象引用，“1”是数组索引

从现在开始，**局部变量数组中的第二项(索引1)是对新创建对象的引用**。因此，我们不会丢失引用，并且赋值确实有效-即使构造函数不返回任何内容。

## 4. 总结

在这个快速教程中，我们学习了JVM如何创建和初始化我们的类实例。此外，我们还了解了实例初始化的底层工作原理。

为了更详细地了解JVM，查看其[规范](https://docs.oracle.com/javase/specs/jvms/se14/html/index.html)总是一个好主意。