---
layout: post
title:  Java反射中修饰符的AccessFlag
category: java-reflect
copyright: java-reflect
excerpt: 反射
---

## 1. 概述

Java中的[反射](https://www.baeldung.com/java-reflection)是一个强大的功能，它允许我们操作不同的成员，例如类、接口、字段和方法。此外，使用反射，我们可以在编译时实例化类、调用方法和访问字段，而无需知道其类型。

在本教程中，我们将首先探索[JVM](https://www.baeldung.com/jvm-series)访问标志。然后，我们将了解如何使用它们。最后，我们将研究修饰符和访问标志之间的区别。

## 2. JVM访问标志

让我们首先了解JVM访问标志。

[Java虚拟机规范](https://docs.oracle.com/javase/specs/jvms/se22/html/)定义了JVM中编译类的结构，它由单个[ClassFile](https://docs.oracle.com/javase/specs/jvms/se22/html/jvms-4.html#jvms-4.1)组成：

```text
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

ClassFile中包含了access_flags项，简单来说，**access_flags是一个掩码，它由各种标志组成，用于定义类的访问权限和其他属性**。

此外，ClassFile由[field_info](https://docs.oracle.com/javase/specs/jvms/se22/html/jvms-4.html#jvms-4.5)和[method_info](https://docs.oracle.com/javase/specs/jvms/se22/html/jvms-4.html#jvms-4.6)项组成，每个项都包含其access_flags项。

[Javassist](https://www.baeldung.com/javassist)和[ASM](https://www.baeldung.com/java-asm)等库使用JVM访问标志来操作Java字节码。

除了Java之外，JVM还支持其他语言，例如[Kotlin](https://www.baeldung.com/kotlin/kotlin-overview)或[Scala](https://www.baeldung.com/scala/scala-basics-tutorial)，每种语言都定义了各自的修饰符。例如，Java中的[Modifier](https://docs.oracle.com/en%2Fjava%2Fjavase%2F22%2Fdocs%2Fapi%2F%2F/java.base/java/lang/reflect/Modifier.html)类包含所有特定于Java编程语言的修饰符。此外，我们在使用反射时通常依赖从这些类中检索到的信息。

但是，当需要将修饰符转换为JVM访问标志时，问题就出现了，让我们进一步分析一下原因。

## 3. 修饰符的访问标志

诸如[可变参数](https://www.baeldung.com/java-varargs)和[transient](https://www.baeldung.com/java-transient-keyword)、[volatile](https://www.baeldung.com/java-volatile)和bridge之类的修饰符使用相同的整数位掩码，**为了解决不同修饰符之间的位冲突，[Java 20](https://www.baeldung.com/java-20-new-features)引入了[AccessFlag](https://docs.oracle.com/en%2Fjava%2Fjavase%2F22%2Fdocs%2Fapi%2F%2F/java.base/java/lang/reflect/AccessFlag.html)枚举，它包含了我们可以在类、字段或方法中使用的所有修饰符**。

该枚举对JVM访问标志进行建模，以便于修饰符和访问标志之间的映射。如果没有AccessFlag枚举，我们需要考虑元素的上下文来确定使用哪个修饰符，尤其是对于那些具有精确位表示的修饰符。

为了查看AccessFlag的实际作用，让我们创建具有几种方法的AccessFlagDemo类，每种方法使用不同的修饰符：

```java
public class AccessFlagDemo {
    public static final void staticFinalMethod() {
    }

    public void varArgsMethod(String... args) {
    }

    public strictfp void strictfpMethod() {
    }
}
```

接下来，让我们检查一下staticFinalMethod()方法中使用的访问标志：

```java
@Test
void givenStaticFinalMethod_whenGetAccessFlag_thenReturnCorrectFlags() throws Exception {
    Class<?> clazz = Class.forName(AccessFlagDemo.class.getName());

    Method method = clazz.getMethod("staticFinalMethod");
    Set<AccessFlag> accessFlagSet = method.accessFlags();

    assertEquals(3, accessFlagSet.size());
    assertTrue(accessFlagSet.contains(AccessFlag.PUBLIC));
    assertTrue(accessFlagSet.contains(AccessFlag.STATIC));
    assertTrue(accessFlagSet.contains(AccessFlag.FINAL));
}
```

这里，我们调用了accessFlags()方法，该方法返回一个包装成不可修改集合的EnumSet。**在内部，该方法使用getModifiers()方法，并根据可应用标志的位置返回访问标志**。我们的方法包含三个访问标志：PUBLIC、STATIC和FINAL。

此外，**从Java 17开始，[strictfp](https://www.baeldung.com/java-strictfp)修饰符是多余的，不再编译到字节码中**：

```java
@Test
void givenStrictfpMethod_whenGetAccessFlag_thenReturnOnlyPublicFlag() throws Exception {
    Class<?> clazz = Class.forName(AccessFlagDemo.class.getName());
    Method method = clazz.getMethod("strictfpMethod");

    Set<AccessFlag> accessFlagSet = method.accessFlags();

    assertEquals(1, accessFlagSet.size());
    assertTrue(accessFlagSet.contains(AccessFlag.PUBLIC));
}
```

我们可以看到，strictfpMethod()包含一个访问标志。

## 4. getModifiers()与accessFlags()方法

在Java中使用反射时，我们经常使用[getModifiers()](https://www.baeldung.com/java-reflection#3-class-modifiers)方法来检索在类、接口、方法或字段上定义的所有修饰符：

```java
@Test
void givenStaticFinalMethod_whenGetModifiers_thenReturnIsStaticTrue() throws Exception {
    Class<?> clazz = Class.forName(AccessFlagDemo.class.getName());
    Method method = clazz.getMethod("staticFinalMethod");

    int methodModifiers = method.getModifiers();

    assertEquals(25, methodModifiers);
    assertTrue(Modifier.isStatic(methodModifiers));
}
```

**getModifiers()方法返回一个表示已编码修饰符标志的整数值**，我们调用了Modifier类中定义的isStatic()方法来检查该方法是否包含[静态](https://www.baeldung.com/java-static)修饰符。此外，Java还会解码方法内部的标志，以确定该方法是否为静态方法。

此外，**值得注意的是，访问标志与Java中定义的修饰符并不相同**。某些访问标志和修饰符具有一对一映射关系，例如[public](https://www.baeldung.com/java-public-keyword)。然而，某些修饰符(例如[seal](https://www.baeldung.com/java-sealed-classes-interfaces))没有指定的访问标志。同样，我们无法将某些访问标志(例如synthesized)映射到相应的修饰符值。

进一步说，**由于一些修饰符共享精确的位表示，如果我们不考虑修饰符的使用上下文，我们可能会得出错误的总结**。

让我们在varArgsMethod()上调用Modifier.toString()：

```java
@Test
void givenVarArgsMethod_whenGetModifiers_thenReturnPublicTransientModifiers() throws Exception {
    Class<?> clazz = Class.forName(AccessFlagDemo.class.getName());
    Method method = clazz.getMethod("varArgsMethod", String[].class);

    int methodModifiers = method.getModifiers();

    assertEquals("public transient", Modifier.toString(methodModifiers));
}
```

该方法返回一个public transient值，如果不考虑上下文，我们可能会认为varArgsMethod()是transient，但这并不准确。

另一方面，访问标志会考虑位的来源上下文，因此，它提供了正确的信息：

```java
@Test
void givenVarArgsMethod_whenGetAccessFlag_thenReturnPublicVarArgsFlags() throws Exception {
    Class<?> clazz = Class.forName(AccessFlagDemo.class.getName());
    Method method = clazz.getMethod("varArgsMethod", String[].class);

    Set<AccessFlag> accessFlagSet = method.accessFlags();

    assertEquals("[PUBLIC, VARARGS]", accessFlagSet.toString());
}
```

## 5. 总结

在本文中，我们了解了JVM访问标志是什么以及如何使用它们。

总而言之，JVM访问标志包含有关运行时成员(例如类、方法和字段)的访问权限和其他属性的信息，我们可以利用访问标志来获取有关特定元素修饰符的准确信息。