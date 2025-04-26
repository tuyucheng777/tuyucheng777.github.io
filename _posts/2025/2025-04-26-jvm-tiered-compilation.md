---
layout: post
title:  JVM中的分层编译
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

JVM在运行时[解释](https://www.baeldung.com/java-compiled-interpreted)并执行[字节码](https://www.baeldung.com/java-class-view-bytecode)，此外，它还利用即时(JIT)编译来提高性能。

在早期版本的Java中，我们必须在Hotspot JVM提供的两种JIT编译器之间手动选择。一种针对更快的应用程序启动进行了优化，而另一种则实现了更佳的整体性能。Java 7引入了分层编译，以便兼顾两者的优势。

在本教程中，我们将介绍客户端和服务器JIT编译器，我们将回顾分层编译及其5个编译级别。最后，我们将通过跟踪编译日志来了解方法编译的工作原理。

## 2. JIT编译器

**JIT编译器会将频繁执行的字节码部分编译为本机代码**，这些部分称为热点，因此得名热点JVM。因此，Java的运行性能可以媲美完全编译型语言，让我们来看看JVM中两种可用的JIT编译器。

### 2.1 C1–客户端编译器

**客户端编译器(也称为C1)是一种针对更快启动时间进行优化的JIT编译器**，它会尝试尽快优化和编译代码。

过去，我们曾将C1用于短期应用程序以及启动时间是一项重要非功能性需求的应用程序。在Java 8之前的版本中，我们必须指定-client标志才能使用C1编译器，但是，如果我们使用Java 8或更高版本，此标志将不起作用。

### 2.2 C2–服务器编译器

**服务器编译器(也称为C2)是一种针对整体性能进行了优化的JIT编译器**，与C1相比，C2会在更长的时间内观察和分析代码，这使得C2能够对编译后的代码进行更好的优化。

过去，我们曾使用C2编译器来处理长时间运行的服务器端应用程序。在Java 8之前的版本中，我们必须指定-server标志才能使用C2编译器，不过，此标志在Java 8或更高版本中将不再生效。

值得注意的是，[Graal](https://www.baeldung.com/graal-java-jit-compiler) JIT编译器自Java 10起也可用，作为C2的替代方案。与C2不同，Graal可以在即时(JIT)和[AOT](https://www.baeldung.com/ahead-of-time-compilation)编译模式下运行，以生成原生代码。

## 3. 分层编译

C2编译器编译相同的方法通常需要更多时间并消耗更多内存，但是，它生成的原生代码比C1编译器优化得更好。

分层编译概念最早是在Java 7中引入的，**其目标是混合使用C1和C2编译器，以实现快速启动和良好的长期性能**。

### 3.1 两全其美

应用程序启动时，JVM会首先解释所有字节码并收集相关的性能分析信息，JIT编译器会利用收集到的性能分析信息来查找热点。

首先，JIT编译器使用C1编译频繁执行的代码段，以快速达到原生代码的性能。之后，当获得更多性能分析信息时，C2就会启动。C2会使用更激进、更耗时的优化方法重新编译代码，以提升性能：

![](/assets/images/2025/javajvm/jvmtieredcompilation01.png)

综上所述，**C1性能提升速度更快，而C2基于更多热点信息，性能提升效果更佳**。

### 3.2 精确分析

分层编译的另一个好处是可以获得更准确的性能分析信息，在分层编译之前，JVM仅在解释期间收集性能分析信息。

启用分层编译后，**JVM还会收集C1编译代码的分析信息**，由于编译后的代码性能更佳，因此JVM可以收集更多的分析样本。

### 3.3 代码缓存

[代码缓存](https://www.baeldung.com/jvm-code-cache)是JVM存储所有编译成本机代码的字节码的内存区域，分层编译使需要缓存的代码量增加了四倍。

从Java 9开始，JVM将代码缓存分为三个区域：

- 非方法段：JVM内部相关代码(约5MB，可通过-XX:NonNMethodCodeHeapSize配置)
- 分析的代码段：C1编译的代码，其生命周期可能较短(默认情况下约为122MB，可通过-XX:ProfiledCodeHeapSize配置)
- 未分析的段：C2编译的代码，具有可能较长的生命周期(默认情况下为122MB，可通过-XX:NonProfiledCodeHeapSize配置)

**分段代码缓存有助于提高代码局部性并减少内存碎片**，从而提高整体性能。

### 3.4 去优化

尽管C2编译的代码经过高度优化且寿命长，但它也可能被去优化，这会导致JVM暂时回滚到解释执行模式。

**当编译器的乐观假设被证明是错误的时候，就会发生去优化**-例如，当分析信息与方法行为不匹配时：

![](/assets/images/2025/javajvm/jvmtieredcompilation02.png)

在我们的示例中，一旦热路径发生变化，JVM就会对编译和内联的代码进行取消优化。

## 4. 编译级别

尽管JVM只使用一个解释器和两个JIT编译器，但编译级别却有五种，其背后的原因是C1编译器可以在三个不同的级别上运行，这三个级别之间的区别在于执行的分析量。

### 4.1 级别0–解释代码

**最初，JVM解释执行所有Java代码**，在此初始阶段，其性能通常不如编译型语言。

但是，JIT编译器在预热阶段后启动，并在运行时编译热门代码，JIT编译器利用在此阶段收集的分析信息来执行优化。

### 4.2 级别1–简单的C1编译代码

**在此级别，JVM使用C1编译器编译代码**，但不收集任何性能分析信息，JVM将级别1用于那些被视为无关紧要的方法。

由于方法复杂度较低，C2编译不会使其速度更快。因此，JVM认为，对于无法进一步优化的代码，收集性能分析信息毫无意义。

### 4.3 级别2–有限的C1编译代码

在级别2中，JVM使用带有轻量级性能分析的C1编译器来编译代码。当C2队列已满时，JVM会使用此级别，目标是尽快编译代码以提高性能。

随后，JVM会在级别3上重新编译代码，并使用完整的性能分析。最后，当C2队列不再繁忙时，JVM会在级别4上重新编译代码。

### 4.4 级别3–完整的C1编译代码

在级别3上，JVM使用具有完整性能分析功能的C1编译器来编译代码。级别3是默认编译路径的一部分，因此，**JVM在所有情况下都会使用它，除了一些简单方法或编译器队列已满的情况**。

JIT编译中最常见的场景是解释代码直接从0级跳转到3级。

### 4.5 级别4–C2编译代码

在此级别，JVM使用C2编译器编译代码，以实现最佳的长期性能。级别4也是默认编译路径的一部分，JVM使用此级别编译除简单方法之外的所有方法。

由于级别4的代码被视为已完全优化，JVM将停止收集性能分析信息。但是，它也可能决定取消优化代码，并将其恢复到级别0。

## 5. 编译参数

从Java 8开始，分层编译默认启用，强烈建议使用它，除非有充分的理由禁用它。

### 5.1 禁用分层编译

我们可以通过设置–XX:-TieredCompilation标志来禁用分层编译，设置此标志后，JVM将不会在编译级别之间切换。因此，我们需要选择要使用的JIT编译器：C1还是C2。

除非明确指定，否则JVM会根据CPU决定使用哪个JIT编译器。对于多核处理器或64位虚拟机，JVM将选择C2，为了禁用C2并仅使用C1且不产生性能分析开销，我们可以应用-XX:TieredStopAtLevel=1参数。

要完全禁用两个JIT编译器，并使用解释器运行所有内容，我们可以应用-Xint标志。但是，需要注意的是，**禁用JIT编译器会对性能产生负面影响**。

### 5.2 设置级别阈值

**编译阈值是指代码编译前方法调用的次数**，在分层编译的情况下，我们可以为2-4级编译设置这些阈值。例如，我们可以设置参数-XX:Tier4CompileThreshold=10000。

为了检查特定Java版本使用的默认阈值，我们可以使用-XX:+PrintFlagsFinal标志运行Java：

```shell
java -XX:+PrintFlagsFinal -version | grep CompileThreshold
intx CompileThreshold = 10000
intx Tier2CompileThreshold = 0
intx Tier3CompileThreshold = 2000
intx Tier4CompileThreshold = 15000
```

我们应该注意，**当启用分层编译时，JVM不使用通用CompileThreshold参数**。

## 6. 方法编译

现在让我们看一下方法编译生命周期：

![](/assets/images/2025/javajvm/jvmtieredcompilation03.png)

总而言之，JVM最初会解释某个方法，直到其调用次数达到Tier3CompileThreshold。然后，**它会使用C1编译器编译该方法，同时继续收集性能分析信息**。最后，当该方法的调用次数达到Tier4CompileThreshold时，JVM会使用C2编译器编译该方法。最终，JVM可能会决定对C2编译的代码进行反优化，这意味着整个过程将重复进行。

### 6.1 编译日志

默认情况下，JIT编译日志是禁用的，要启用它们，我们可以设置-XX:+PrintCompilation标志；编译日志的格式如下：

- 时间戳：自应用程序启动以来的毫秒数
- 编译ID：每个编译方法的增量ID
- 属性：编译状态有五个可能的值：
  - %：发生栈内替换
  - s：该方法已同步
  - !：该方法包含异常处理程序
  - b：编译发生在阻塞模式下
  - n：编译将包装器转换为本机方法
- 编译级别：0至4之间
- 方法名称
- 字节码大小
- 去优化指标-具有两个可能的值：
  - 未入围者：标准C1去优化或编译器的乐观假设被证明是错误的
  - 僵尸进程：垃圾回收器释放代码缓存空间的清理机制

### 6.2 示例

让我们用一个简单的例子来演示方法编译的生命周期，首先，我们创建一个实现JSON格式化程序的类：

```java
public class JsonFormatter implements Formatter {

    private static final JsonMapper mapper = new JsonMapper();

    @Override
    public <T> String format(T object) throws JsonProcessingException {
        return mapper.writeValueAsString(object);
    }
}
```

接下来，我们将创建一个实现相同接口但实现XML格式化程序的类：

```java
public class XmlFormatter implements Formatter {

    private static final XmlMapper mapper = new XmlMapper();

    @Override
    public <T> String format(T object) throws JsonProcessingException {
        return mapper.writeValueAsString(object);
    }
}
```

现在，我们将编写一个使用两种不同格式化程序实现的方法。在循环的前半部分，我们将使用JSON实现，然后在其余部分切换到XML实现：

```java
public class TieredCompilation {

    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 1_000_000; i++) {
            Formatter formatter;
            if (i < 500_000) {
                formatter = new JsonFormatter();
            } else {
                formatter = new XmlFormatter();
            }
            formatter.format(new Article("Tiered Compilation in JVM", "Tuyucheng"));
        }
    }
}
```

最后，我们将设置-XX:+PrintCompilation标志，运行main方法，并观察编译日志。

### 6.3 审查日志

让我们关注三个自定义类及其方法的日志输出。

前两条日志记录表明，JVM在级别3编译了main方法和format方法的JSON实现。因此，这两个方法都是由C1编译器编译的，C1编译后的代码替换了最初解释的版本：

```text
567  714       3       cn.tuyucheng.taketoday.tieredcompilation.JsonFormatter::format (8 bytes)
687  832 %     3       cn.tuyucheng.taketoday.tieredcompilation.TieredCompilation::main @ 2 (58 bytes)
```

几百毫秒后，JVM在级别4上编译了这两个方法。因此，C2编译的版本取代了之前用C1编译的版本：

```text
659  800       4       cn.tuyucheng.taketoday.tieredcompilation.JsonFormatter::format (8 bytes)
807  834 %     4       cn.tuyucheng.taketoday.tieredcompilation.TieredCompilation::main @ 2 (58 bytes)
```

仅仅几毫秒之后，我们就看到了第一个去优化的例子。这里，JVM将C1编译版本标记为过时(而非新版本)：

```text
812  714       3       cn.tuyucheng.taketoday.tieredcompilation.JsonFormatter::format (8 bytes)   made not entrant
838 832 % 3 cn.tuyucheng.taketoday.tieredcompilation.TieredCompilation::main @ 2 (58 bytes) made not entrant
```

过了一会儿，我们会注意到另一个去优化的例子，这条日志很有趣，因为JVM将完全优化的C2编译版本标记为过时(而不是进入者)，这意味着**当JVM检测到完全优化的代码不再有效时，它会回滚该代码**：

```text
1015  834 %     4       cn.tuyucheng.taketoday.tieredcompilation.TieredCompilation::main @ 2 (58 bytes)   made not entrant
1018  800       4       cn.tuyucheng.taketoday.tieredcompilation.JsonFormatter::format (8 bytes)   made not entrant
```

接下来，我们将首次看到format方法的XML实现。JVM在级别3编译了它，同时编译了main方法：

```text
1160 1073       3       cn.tuyucheng.taketoday.tieredcompilation.XmlFormatter::format (8 bytes)
1202 1141 %     3       cn.tuyucheng.taketoday.tieredcompilation.TieredCompilation::main @ 2 (58 bytes)
```

几百毫秒后，JVM在级别4编译了这两个方法。然而，这一次，main方法使用的是XML实现：

```text
1341 1171       4       cn.tuyucheng.taketoday.tieredcompilation.XmlFormatter::format (8 bytes)
1505 1213 %     4       cn.tuyucheng.taketoday.tieredcompilation.TieredCompilation::main @ 2 (58 bytes
```

与之前相同，几毫秒后，JVM将C1编译版本标记为过时(不再进入)：

```text
1492 1073       3       cn.tuyucheng.taketoday.tieredcompilation.XmlFormatter::format (8 bytes)   made not entrant
1508 1141 %     3       cn.tuyucheng.taketoday.tieredcompilation.TieredCompilation::main @ 2 (58 bytes)   made not entrant
```

JVM继续使用级别4编译方法直到我们的程序结束。

## 7. 总结

在本文中，我们探讨了JVM中的分层编译概念。我们回顾了两种JIT编译器，以及分层编译如何结合这两种编译器来实现最佳效果。我们还了解了五个编译级别，并学习了如何使用JVM参数控制它们。

在示例中，我们通过观察编译日志探索了完整的方法编译生命周期。