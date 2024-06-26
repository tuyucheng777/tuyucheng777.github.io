---
layout: post
title:  Java中对象的内存布局
category: java-jvm
copyright: java-jvm
excerpt: Java JVM
---

## 1. 概述

在本教程中，我们将了解JVM如何在堆中布局对象和数组。

首先，我们将从一些理论开始。然后，我们将探讨不同情况下的不同对象和数组内存布局。

通常，运行时数据区的内存布局不是JVM规范的一部分，[由实现者自行决定](https://docs.oracle.com/javase/specs/jvms/se14/html/jvms-2.html)。因此，每个JVM实现可能有不同的策略来在内存中布局对象和数组。在本教程中，我们将重点介绍一个特定的JVM实现：HotSpot JVM。

我们也可以互换使用JVM和HotSpot JVM术语。

## 2. 普通对象指针(OOP)

**HotSpot JVM使用一种称为普通对象指针([OOPS](https://github.com/openjdk/jdk15/tree/master/src/hotspot/share/oops))的数据结构来表示指向对象的指针**。JVM中的所有指针(包括对象和数组)都基于一种称为[oopDesc](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/oop.hpp#L52)的特殊数据结构。每个oopDesc使用以下信息描述指针：

-   一个[标记字](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/oop.hpp#L56)
-   一个可能是压缩的[klass字](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/oop.hpp#L57)

[标记字](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/markWord.hpp#L33)描述对象头。**HotSpot JVM使用这个字来存储身份哈希码、偏向锁定模式、锁定信息和GC元数据**。

此外，标记字状态仅包含一个[uintptr_t](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/markWord.hpp#L96)，因此，**其大小在32位和64位架构中分别在4和8字节之间变化**。此外，有偏向的和正常的对象的标记字是不同的。但是，我们只会考虑普通对象，因为Java 15将[弃用偏向锁](https://openjdk.java.net/jeps/374)。

此外，[klass字](https://github.com/openjdk/jdk15/blob/bf1e6903a2499d0c2ab2f8703a1dc29046e8375d/src/hotspot/share/oops/klass.hpp#L54)封装了语言级别的类信息，例如类名、修饰符、超类信息等。

对于Java中的普通对象(表示为[instanceOop](https://github.com/openjdk/jdk15/blob/master/src/hotspot/share/oops/instanceOop.hpp))，**对象头由标记和klass字加上可能的对齐填充组成**。在对象头之后，可能有0个或多个对实例字段的引用。因此，在64位架构中至少有16个字节，因为标记有8个字节、4个字节的klass和另外4个字节用于填充。

**对于表示为[arrayOop](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/arrayOop.hpp#L35)的数组，对象标头除了标记、klass和填充之外，还包含一个4字节的数组长度**。同样，由于8个字节的标记、4个字节的klass和另外4个字节的数组长度，这将至少为16个字节。

现在我们已经对理论有了足够的了解，让我们看看内存布局在实践中是如何工作的。

## 3. 设置JOL

为了检查JVM中对象的内存布局，我们将广泛使用Java对象布局([JOL](https://openjdk.java.net/projects/code-tools/jol/))。因此，我们需要添加[jol-core](https://search.maven.org/artifact/org.openjdk.jol/jol-core)依赖：

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

## 4. 内存布局示例

让我们首先查看常规VM详细信息：

```java
System.out.println(VM.current().details());
```

这将打印：

```plaintext
# Running 64-bit HotSpot VM.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

这意味着引用占用4个字节，boolean和byte占用1个字节，short和char占用2个字节，int和float占用4个字节，最后，long和double占用8个字节。有趣的是，如果我们将它们用作数组元素，它们会消耗相同数量的内存。

此外，如果我们通过-XX:-UseCompressedOops禁用[压缩引用](https://www.baeldung.com/jvm-compressed-oops)，则只有引用大小更改为8字节：

```plaintext
# Field sizes by type: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 8, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```

### 4.1 基本

让我们考虑一个SimpleInt类：

```java
public class SimpleInt {
    private int state;
}
```

如果我们打印它的类布局：

```java
System.out.println(ClassLayout.parseClass(SimpleInt.class).toPrintable());
```

我们会看到类似以下内容：

```text
SimpleInt object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4    int SimpleInt.state                           N/A
Instance size: 16 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

如上所示，对象标头为12个字节，包括8个字节的标记和4个字节的klass。之后，我们有4个字节用于int state。总的来说，这个类的任何对象都会消耗16个字节。

此外，对象标头和状态没有值，因为我们正在解析类布局，而不是实例布局。

### 4.2 身份哈希码

[hashCode()](https://www.baeldung.com/java-hashcode)是所有Java对象的通用方法之一。**当我们没有为类声明hashCode()方法时，Java将为其使用身份哈希码**。

对象的身份哈希码在其生命周期内不会更改。因此，**HotSpot JVM在计算出该值后将其存储在标记字中**。

让我们看看对象实例的内存布局：

```java
SimpleInt instance = new SimpleInt();
System.out.println(ClassLayout.parseInstance(instance).toPrintable());
```

HotSpot JVM延迟计算身份哈希码：

```text
SimpleInt object internals:
 OFFSET  SIZE   TYPE DESCRIPTION               VALUE
      0     4        (object header)           01 00 00 00 (00000001 00000000 00000000 00000000) (1) # mark
      4     4        (object header)           00 00 00 00 (00000000 00000000 00000000 00000000) (0) # mark
      8     4        (object header)           9b 1b 01 f8 (10011011 00011011 00000001 11111000) (-134145125) # klass
     12     4    int SimpleInt.state           0
Instance size: 16 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

如上所示，标记字目前似乎还没有存储任何重要内容。

但是，如果我们在对象实例上调用[System.identityHashCode()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/System.html#identityHashCode(java.lang.Object))甚至Object.hashCode()，这将会改变：

```java
System.out.println("The identity hash code is " + System.identityHashCode(instance));
System.out.println(ClassLayout.parseInstance(instance).toPrintable());
```

现在，我们可以发现身份哈希码作为标记字的一部分：

```text
The identity hash code is 1702146597
SimpleInt object internals:
 OFFSET  SIZE   TYPE DESCRIPTION               VALUE
      0     4        (object header)           01 25 b2 74 (00000001 00100101 10110010 01110100) (1957831937)
      4     4        (object header)           65 00 00 00 (01100101 00000000 00000000 00000000) (101)
      8     4        (object header)           9b 1b 01 f8 (10011011 00011011 00000001 11111000) (-134145125)
     12     4    int SimpleInt.state           0
```

HotSpot JVM将身份哈希码存储为标记字中的“25 b2 74 65”。最高有效字节是65，因为JVM以小端格式存储该值。因此，要恢复十进制的哈希码值(1702146597)，我们必须以相反的顺序读取“25 b2 74 65”字节序列：

```text
65 74 b2 25 = 01100101 01110100 10110010 00100101 = 1702146597
```

### 4.3 对齐

默认情况下，JVM会向对象添加足够的填充以使其大小成为8的倍数。

例如，考虑SimpleLong类：

```java
public class SimpleLong {
    private long state;
}
```

如果我们解析类布局：

```java
System.out.println(ClassLayout.parseClass(SimpleLong.class).toPrintable());
```

然后JOL将打印内存布局：

```text
SimpleLong object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4        (alignment/padding gap)                  
     16     8   long SimpleLong.state                          N/A
Instance size: 24 bytes
Space losses: 4 bytes internal + 0 bytes external = 4 bytes total
```

如上图，对象标头和long state总共消耗了20个字节。**为了使这个大小成为8字节的倍数，JVM添加了4字节的填充**。

**我们还可以通过-XX:ObjectAlignmentInBytes调整标志更改默认对齐大小**。例如，对于同一个类，带有-XX:ObjectAlignmentInBytes=16的内存布局将是：

```text
SimpleLong object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4        (alignment/padding gap)                  
     16     8   long SimpleLong.state                          N/A
     24     8        (loss due to the next object alignment)
Instance size: 32 bytes
Space losses: 4 bytes internal + 8 bytes external = 12 bytes total
```

对象标头和long变量总共仍然占用20个字节。因此，我们应该再添加12个字节以使其成为16的倍数。

如上所示，它添加了4个内部填充字节以在偏移量16处开始long变量(启用更对齐的访问)。然后它将剩余的8个字节添加到long变量之后。

### 4.4 字段包装

**当一个类有多个字段时，JVM可能会以最小化填充浪费的方式分配这些字段**。例如，考虑FieldsArrangement类：

```java
public class FieldsArrangement {
    private boolean first;
    private char second;
    private double third;
    private int fourth;
    private boolean fifth;
}
```

字段声明顺序及其在内存布局中的顺序是不同的：

```text
OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0    12           (object header)                           N/A
     12     4       int FieldsArrangement.fourth                  N/A
     16     8    double FieldsArrangement.third                   N/A
     24     2      char FieldsArrangement.second                  N/A
     26     1   boolean FieldsArrangement.first                   N/A
     27     1   boolean FieldsArrangement.fifth                   N/A
     28     4           (loss due to the next object alignment)
```

这背后的主要动机是尽量减少填充浪费。

### 4.5 锁定

JVM还在标记字中维护锁信息，让我们看看实际效果：

```java
public class Lock {}
```

如果我们创建这个类的一个实例，它的内存布局将是：

```text
Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 
      4     4        (object header)                           00 00 00 00
      8     4        (object header)                           85 23 02 f8
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
```

但是，如果我们在这个实例上同步：

```java
synchronized (lock) {
    System.out.println(ClassLayout.parseInstance(lock).toPrintable());
}
```

内存布局更改为：

```text
Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           f0 78 12 03
      4     4        (object header)                           00 70 00 00
      8     4        (object header)                           85 23 02 f8
     12     4        (loss due to the next object alignment)
```

如上所示，**当我们持有监视器锁时，标记字的位模式会发生变化**。

### 4.6 年龄和任期

**为了将对象提升到老年代(当然是在分代[GC](https://www.baeldung.com/jvm-garbage-collectors)中)，JVM需要跟踪每个对象的存活数**。如前所述，JVM也在标记字内部维护此信息。

为了模拟minor GC，我们将通过将对象分配给volatile变量来创建大量垃圾。这样我们就可以防止JIT编译器可能[消除死代码](https://www.baeldung.com/java-microbenchmark-harness#dead-code-elimination)：

```java
volatile Object consumer;
Object instance = new Object();
long lastAddr = VM.current().addressOf(instance);
ClassLayout layout = ClassLayout.parseInstance(instance);

for (int i = 0; i < 10_000; i++) {
    long currentAddr = VM.current().addressOf(instance);
    if (currentAddr != lastAddr) {
        System.out.println(layout.toPrintable());
    }

    for (int j = 0; j < 10_000; j++) {
        consumer = new Object();
    }

    lastAddr = currentAddr;
}
```

**每次活动对象的地址发生变化时，这可能是因为较小的GC和幸存者空间之间的移动**。对于每次更改，我们还打印新的对象布局以查看老化的对象。

以下是标记字的前4个字节随时间的变化：

```plaintext
09 00 00 00 (00001001 00000000 00000000 00000000)
              ^^^^
11 00 00 00 (00010001 00000000 00000000 00000000)
              ^^^^
19 00 00 00 (00011001 00000000 00000000 00000000)
              ^^^^
21 00 00 00 (00100001 00000000 00000000 00000000)
              ^^^^
29 00 00 00 (00101001 00000000 00000000 00000000)
              ^^^^
31 00 00 00 (00110001 00000000 00000000 00000000)
              ^^^^
31 00 00 00 (00110001 00000000 00000000 00000000)
              ^^^^
```

### 4.7 虚假共享和@Contended

**jdk.internal.vm.annotation.Contended注解(或Java 8上的sun.misc.Contended)提示JVM隔离带注解的字段以避免[虚假共享](https://alidg.me/blog/2020/4/24/thread-local-random#false-sharing)**。

简而言之，Contended注解在每个标注字段周围添加一些填充，以将每个字段隔离在其自己的缓存行上。因此，这将影响内存布局。

为了更好地理解这一点，让我们考虑一个例子：

```java
public class Isolated {

    @Contended
    private int v1;

    @Contended
    private long v2;
}
```

如果我们检查这个类的内存布局，我们会看到类似这样的东西：

```text
Isolated object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12   128        (alignment/padding gap)                  
    140     4    int Isolated.i                                N/A
    144   128        (alignment/padding gap)                  
    272     8   long Isolated.l                                N/A
Instance size: 280 bytes
Space losses: 256 bytes internal + 0 bytes external = 256 bytes total
```

如上所示，JVM在每个带注解的字段周围添加了128字节的填充。**大多数现代机器中的缓存行大小约为64/128字节，因此需要128字节填充**。当然，我们可以使用-XX:ContendedPaddingWidth调整标志来控制Contended填充大小。

请注意Contended注解是JDK内部的，因此我们应该避免使用它。

此外，我们应该使用-XX:-RestrictContended调整标志运行我们的代码；否则，注解不会生效。基本上，默认情况下，此注解仅供内部使用，禁用RestrictContended将为公共API解锁此功能。

### 4.8 数组

**正如我们之前提到的，数组长度也是arrayOop的一部分**。例如，对于包含3个元素的布尔数组：

```java
boolean[] booleans = new boolean[3];
System.out.println(ClassLayout.parseInstance(booleans).toPrintable());
```

内存布局如下所示：

```text
[Z object internals:
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0     4           (object header)                           01 00 00 00 # mark
      4     4           (object header)                           00 00 00 00 # mark
      8     4           (object header)                           05 00 00 f8 # klass
     12     4           (object header)                           03 00 00 00 # array length
     16     3   boolean [Z.<elements>                             N/A
     19     5           (loss due to the next object alignment)
Instance size: 24 bytes
Space losses: 0 bytes internal + 5 bytes external = 5 bytes total
```

在这里，我们有16个字节的对象头，其中包含8个字节的标记字、4个字节的klass字和4个字节的长度。在对象头之后，我们有3个字节用于包含3个元素的布尔数组。

### 4.9 压缩引用

到目前为止，我们的示例是在启用了压缩引用的64位架构中执行的。

使用8字节对齐，我们最多可以使用[32GB的堆](https://www.baeldung.com/jvm-compressed-oops#2beyond-32-gb)压缩引用。**如果我们超出这个限制甚至手动禁用压缩引用，那么klass字将消耗8个字节而不是4个字节**。

让我们看看使用-XX:-UseCompressedOops调整标志禁用压缩oops时同一数组示例的内存布局：

```text
[Z object internals:
 OFFSET  SIZE      TYPE DESCRIPTION                               VALUE
      0     4           (object header)                           01 00 00 00 # mark
      4     4           (object header)                           00 00 00 00 # mark
      8     4           (object header)                           28 60 d2 11 # klass
     12     4           (object header)                           01 00 00 00 # klass
     16     4           (object header)                           03 00 00 00 # length
     20     4           (alignment/padding gap)                  
     24     3   boolean [Z.<elements>                             N/A
     27     5           (loss due to the next object alignment)
```

正如承诺的那样，现在klass字还有4个字节。

## 5. 总结

在本教程中，我们了解了JVM如何在堆中布置对象和数组。

要进行更详细的探索，强烈建议查看[JVM源代码的oops部分](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/)。此外，[Aleksey Shipilëv](https://shipilev.net/)在这方面有一篇[更深入的文章](https://shipilev.net/jvm/objects-inside-out/)。

此外，[JOL](https://hg.openjdk.java.net/code-tools/jol/file/tip/jol-samples/src/main/java/org/openjdk/jol/samples/)的更多示例作为项目源代码的一部分提供。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jvm-2)上获得。