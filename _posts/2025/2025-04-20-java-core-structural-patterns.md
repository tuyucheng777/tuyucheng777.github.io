---
layout: post
title:  Java核心中的结构型模式
category: designpattern
copyright: designpattern
excerpt: 结构型模式
---

## 1. 概述

**[结构型设计模式](https://www.baeldung.com/design-patterns-series)通过识别大型对象之间的关系来简化其结构设计，它们描述了组合类和对象的常用方法**，使它们成为可重复的解决方案。

[GoF](https://www.pearson.com/us/higher-education/program/Gamma-Design-Patterns-Elements-of-Reusable-Object-Oriented-Software/PGM14333.html)描述了7种这样的结构型方式或模式，**在本快速教程中，我们将通过示例了解Java的一些核心库是如何采用这些模式的**。

## 2. [适配器](https://www.baeldung.com/java-adapter-pattern)

顾名思义，**适配器充当中介，将原本不兼容的接口转换为客户端期望的接口**。

当我们想要采用一个源代码无法修改的现有类并使其与另一个类一起工作时，这很有用。

JDK的集合框架提供了许多适配器模式的示例：

```java
List<String> musketeers = Arrays.asList("Athos", "Aramis", "Porthos");
```

这里，**[Arrays#asList](https://www.baeldung.com/java-arrays-aslist-vs-new-arraylist)帮助我们将Array适配为List**。

I/O框架也广泛使用了这种模式，例如，我们来看下面的代码片段，它将一个[InputStream映射到Reader](https://www.baeldung.com/java-convert-inputstream-to-reader)对象：

```java
InputStreamReader input = new InputStreamReader(new FileInputStream("input.txt"));
```

## 3. [桥接](https://www.baeldung.com/java-structural-design-patterns#bridge)

**桥接模式允许抽象和实现之间的分离，以便它们可以彼此独立开发，但仍然有一种方式或桥梁来共存和交互**。

Java中的一个例子是[JDBC API](https://www.baeldung.com/java-jdbc)，它充当数据库(例如Oracle、MySQL和PostgreSQL)与其特定实现之间的链接。

JDBC API是一组标准接口，例如Driver、Connection和ResultSet等，这使得不同的数据库供应商可以拥有各自的实现。

例如，要创建与数据库的连接，我们会写：

```java
Connection connection = DriverManager.getConnection(url);
```

这里，url是一个可以代表任何数据库供应商的字符串。

举个例子，对于PostgreSQL，我们可能写：

```java
String url = "jdbc:postgresql://localhost/demo";
```

对于MySQL：

```java
String url = "jdbc:mysql://localhost/demo";
```

## 4. [复合](https://www.baeldung.com/java-composite-pattern)

此模式处理树状的对象结构，在这棵树中，单个对象，甚至整个层次结构，都以相同的方式处理。简而言之，**此模式以分层的方式排列对象，以便客户端可以无缝地与整体或部分对象交互**。

AWT/Swing中的嵌套容器是Java核心中复合模式的绝佳应用示例，java.awt.Container对象本质上是一个根组件，它可以包含其他组件，从而形成嵌套组件的树形结构。

考虑以下代码片段：

```java
JTabbedPane pane = new JTabbedPane();
pane.addTab("1", new Container());
pane.addTab("2", new JButton());
pane.addTab("3", new JCheckBox());
```

这里使用的所有类-即JTabbedPane、JButton、JCheckBox和JFrame都是Container的后代，正如我们所见，**此代码片段在第二行中处理树或Container的根，其方式与处理其子级的方式相同**。

## 5. [装饰器](https://www.baeldung.com/java-decorator-pattern)

**当我们想要在不修改原始对象本身的情况下增强对象的行为时，这种模式就会发挥作用**，这是通过向对象添加相同类型的包装器来为其附加额外的责任来实现的。

这种模式最常见的用法之一可以在[java.io](https://www.baeldung.com/java-download-file#using-java-io)包中找到：

```java
BufferedInputStream bis = new BufferedInputStream(new FileInputStream(new File("test.txt")));
while (bis.available() > 0) {
    char c = (char) bis.read();
    System.out.println("Char: " + c);
}
```

这里，**BufferedInputStream装饰了FileInputStream，使其能够缓冲输入**。值得注意的是，这两个类都拥有InputStream作为共同的祖先，这意味着装饰对象和被装饰对象属于同一类型，这无疑是装饰器模式的体现。

## 6. [外观](https://www.baeldung.com/java-facade-pattern)

根据定义，“外观”一词指的是对象的人造或虚假外观。应用于编程中，**它同样意味着为一组复杂的对象提供另一张面孔-或者更确切地说，接口**。

当我们想要简化或隐藏子系统或框架的复杂性时，这种模式就会发挥作用。

Faces API的ExternalContext是外观模式的绝佳示例，它内部使用了HttpServletRequest、HttpServletResponse和HttpSession等类。简而言之，它是一个让Faces API完全不了解其底层应用环境的类。

让我们看看[Primefaces](https://www.primefaces.org/docs/api/5.3/org/primefaces/component/export/PDFExporter.html#writePDFToResponse(javax.faces.context.ExternalContext,java.io.ByteArrayOutputStream,java.lang.String))如何使用它来写入HttpResponse，而实际上并不了解它：

```java
protected void writePDFToResponse(ExternalContext externalContext, ByteArrayOutputStream baos, String fileName) throws IOException, DocumentException {
    externalContext.setResponseContentType("application/pdf");
    externalContext.setResponseHeader("Expires", "0");
    // set more relevant headers
    externalContext.setResponseContentLength(baos.size());
    externalContext.addResponseCookie(
            Constants.DOWNLOAD_COOKIE, "true", Collections.<String, Object>emptyMap());
    OutputStream out = externalContext.getResponseOutputStream();
    baos.writeTo(out);
    // do cleanup
}
```

正如我们在这里看到的，我们直接使用ExternalContext作为外观来设置响应头、实际响应和Cookie，HTTPResponse不在其中。

## 7. [享元](https://www.baeldung.com/java-flyweight)

**享元模式通过回收对象来减轻它们的负担(或内存占用)**，换句话说，如果我们拥有可以共享状态的不可变对象，那么按照此模式，我们可以缓存它们以提高系统性能。

在Java的所有[Number](https://www.baeldung.com/java-number-class)类中都可以发现享元的身影。

用于创建任何数据类型的包装类的对象的valueOf方法旨在缓存值并在需要时返回它们。

例如，[Integer](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Integer.html)有一个静态类IntegerCache，它帮助其[valueOf](https://www.baeldung.com/java-convert-string-to-int-or-integer#integervalueof)方法始终缓存-128到127范围内的值：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high) {
        return IntegerCache.cache[i + (-IntegerCache.low)];
    }
    return new Integer(i);
}
```

## 8. [代理](https://www.baeldung.com/java-proxy-pattern)

**此模式为另一个复杂对象提供代理或替代**，虽然听起来与外观模式类似，但实际上两者有所不同，因为外观模式为客户端提供了不同的交互接口，而代理模式的接口与其隐藏的对象的接口相同。

使用此模式，在创建原始对象之前或之后对其执行任何操作变得很容易。

JDK提供了一个开箱即用的[java.lang.reflect.Proxy](https://www.baeldung.com/java-dynamic-proxies)类用于代理实现：

```java
Foo proxyFoo = (Foo) Proxy.newProxyInstance(Foo.class.getClassLoader(), new Class<?>[] { Foo.class }, handler);
```

上面的代码片段为接口Foo创建了一个代理proxyFoo。

## 9. 总结

在这个简短的教程中，我们看到了在核心Java中实现的结构设计模式的实际用法。

总而言之，我们简要定义了这七种模式各自的含义，然后通过代码片段逐一理解它们。