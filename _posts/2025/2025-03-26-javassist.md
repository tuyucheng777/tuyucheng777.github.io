---
layout: post
title:  Javassist简介
category: libraries
copyright: libraries
excerpt: Javasisst
---

## 1. 概述

在本文中，我们将研究[Javasisst](https://jboss-javassist.github.io/javassist/)库。

简单地说，该库通过使用高级API比JDK中的API更简单，使操作Java字节码的过程更简单。

## 2. Maven依赖

要将Javassist库添加到我们的项目中，我们需要将[javassist](https://mvnrepository.com/artifact/javassist/javassist)添加到pom中：

```xml
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>${javaassist.version}</version>
</dependency>

<properties>
    <javaassist.version>3.21.0-GA</javaassist.version>
</properties>
```

## 3. 什么是字节码？

从非常高的层次来看，每个Java类都是以纯文本格式编写并编译为字节码的-一个可以由Java虚拟机处理的指令集，JVM将字节码指令翻译成机器级汇编指令。

假设我们有一个Point类：

```java
public class Point {
    private int x;
    private int y;

    public void move(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // standard constructors/getters/setters
}
```

编译后，将创建包含字节码的Point.class文件。我们可以通过执行javap命令来查看该类的字节码：

```shell
javap -c Point.class
```

这将打印以下输出：

```text
public class cn.tuyucheng.taketoday.javasisst.Point {
  public cn.tuyucheng.taketoday.javasisst.Point(int, int);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iload_1
       6: putfield      #2                  // Field x:I
       9: aload_0
      10: iload_2
      11: putfield      #3                  // Field y:I
      14: return

  public void move(int, int);
    Code:
       0: aload_0
       1: iload_1
       2: putfield      #2                  // Field x:I
       5: aload_0
       6: iload_2
       7: putfield      #3                  // Field y:I
      10: return
}
```

所有这些指令都由Java语言指定；其中有[大量可用](https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings)。

我们来分析一下move()方法的字节码指令：

-   aload_0指令将引用从局部变量0加载到堆栈
-   iload_1从局部变量1加载int值
-   putfield设置我们对象的字段x，所有操作对于字段y都是类比的
-   最后一条指令是return

每行Java代码都使用适当的指令编译为字节码，Javassist库使处理该字节码变得相对容易。

## 4. 生成Java类

Javassist库可用于生成新的Java class文件。

假设我们要生成一个实现java.lang.Cloneable接口的JavassistGeneratedClass类，我们希望该类有一个int类型的id字段。ClassFile用于创建新的class文件，FieldInfo用于向类添加新字段：

```java
ClassFile cf = new ClassFile(false, "cn.tuyucheng.taketoday.JavassistGeneratedClass", null);
cf.setInterfaces(new String[] {"java.lang.Cloneable"});

FieldInfo f = new FieldInfo(cf.getConstPool(), "id", "I");
f.setAccessFlags(AccessFlag.PUBLIC);
cf.addField(f);
```

在我们创建一个JavassistGeneratedClass.class之后，我们可以断言它实际上有一个id字段：

```java
ClassPool classPool = ClassPool.getDefault();
Field[] fields = classPool.makeClass(cf).toClass().getFields();
 
assertEquals(fields[0].getName(), "id");
```

## 5. 加载类的字节码指令

如果我们想加载一个已经存在的类方法的字节码指令，我们可以获取该类特定方法的一个CodeAttribute。然后我们可以得到一个CodeIterator来迭代该方法的所有字节码指令。

让我们加载Point类的move()方法的所有字节码指令：

```java
ClassPool cp = ClassPool.getDefault();
ClassFile cf = cp.get("cn.tuyucheng.taketoday.javasisst.Point")
    .getClassFile();
MethodInfo minfo = cf.getMethod("move");
CodeAttribute ca = minfo.getCodeAttribute();
CodeIterator ci = ca.iterator();

List<String> operations = new LinkedList<>();
while (ci.hasNext()) {
    int index = ci.next();
    int op = ci.byteAt(index);
    operations.add(Mnemonic.OPCODE[op]);
}

assertEquals(operations,
    Arrays.asList(
    "aload_0", 
    "iload_1", 
    "putfield", 
    "aload_0", 
    "iload_2",  
    "putfield", 
    "return"));
```

我们可以通过将字节码聚合到operations列表来查看move()方法的所有字节码指令，如上面的断言所示。

## 6. 向现有类字节码添加字段

假设我们要在现有类的字节码中添加一个int类型的字段，我们可以使用ClassPoll加载该类并向其中添加一个字段：

```java
ClassFile cf = ClassPool.getDefault()
    .get("cn.tuyucheng.taketoday.javasisst.Point").getClassFile();

FieldInfo f = new FieldInfo(cf.getConstPool(), "id", "I");
f.setAccessFlags(AccessFlag.PUBLIC);
cf.addField(f);
```

我们可以使用反射来验证id字段是否存在于Point类中：

```java
ClassPool classPool = ClassPool.getDefault();
Field[] fields = classPool.makeClass(cf).toClass().getFields();
List<String> fieldsList = Stream.of(fields)
    .map(Field::getName)
    .collect(Collectors.toList());
 
assertTrue(fieldsList.contains("id"));
```

## 7. 向类字节码添加构造函数

我们可以使用addInvokespecial()方法将构造函数添加到前面示例之一中提到的现有类。

我们可以通过调用java.lang.Object类的<init\>方法来添加无参数构造函数：

```java
ClassFile cf = ClassPool.getDefault()
    .get("cn.tuyucheng.taketoday.javasisst.Point").getClassFile();
Bytecode code = new Bytecode(cf.getConstPool());
code.addAload(0);
code.addInvokespecial("java/lang/Object", MethodInfo.nameInit, "()V");
code.addReturn(null);

MethodInfo minfo = new MethodInfo(cf.getConstPool(), MethodInfo.nameInit, "()V");
minfo.setCodeAttribute(code.toCodeAttribute());
cf.addMethod(minfo);
```

我们可以通过遍历字节码来检查新创建的构造函数是否存在：

```java
CodeIterator ci = code.toCodeAttribute().iterator();
List<String> operations = new LinkedList<>();
while (ci.hasNext()) {
    int index = ci.next();
    int op = ci.byteAt(index);
    operations.add(Mnemonic.OPCODE[op]);
}

assertEquals(operations, Arrays.asList("aload_0", "invokespecial", "return"));
```

## 8. 总结

在本文中，我们介绍了Javassist库，它的目的是使字节码操作更容易。

我们专注于核心功能并从Java代码生成class文件；我们还对已经创建的Java类进行了一些字节码操作。