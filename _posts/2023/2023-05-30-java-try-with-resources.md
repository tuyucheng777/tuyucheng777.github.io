---
layout: post
title:  Java – try-with-resources
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

对try-with-resources的支持在Java 7中引入，允许我们声明要在try块中使用的资源，并确保资源将在该块执行后关闭。

声明的资源需要实现AutoCloseable接口。

## 2. 使用try-with-resources

简单地说，要自动关闭，必须在try中声明和初始化资源：

```java
try (PrintWriter writer = new PrintWriter(new File("test.txt"))) {
    writer.println("Hello World");
}
```

## 3. 将try-catch-finally替换为try-with-resources

使用新的try-with-resources功能的简单而明显的方法是替换传统且冗长的try-catch-finally块。

让我们比较以下代码示例。

第一个是典型的try-catch-finally块：

```java
Scanner scanner = null;
try {
    scanner = new Scanner(new File("test.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

这是使用try-with-resources的新的超级简洁解决方案：

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

## 4. try-with-resources多个资源

我们可以在try-with-resources块中通过用分号分隔它们来声明多个资源：

```java
try (Scanner scanner = new Scanner(new File("testRead.txt"));
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) {
    while (scanner.hasNext()) {
	writer.print(scanner.nextLine());
    }
}
```

## 5. 具有AutoCloseable的自定义资源 

要构造将由try-with-resources块正确处理的自定义资源，该类应实现Closeable或AutoCloseable接口并覆盖close方法：

```java
public class MyResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("Closed MyResource");
    }
}
```

## 6. 资源关闭顺序

首先定义/获取的资源将最后关闭。让我们看一下此行为的示例：

资源1：

```java
public class AutoCloseableResourcesFirst implements AutoCloseable {

    public AutoCloseableResourcesFirst() {
        System.out.println("Constructor -> AutoCloseableResources_First");
    }

    public void doSomething() {
        System.out.println("Something -> AutoCloseableResources_First");
    }

    @Override
    public void close() throws Exception {
        System.out.println("Closed AutoCloseableResources_First");
    }
}
```

资源2：

```java
public class AutoCloseableResourcesSecond implements AutoCloseable {

    public AutoCloseableResourcesSecond() {
        System.out.println("Constructor -> AutoCloseableResources_Second");
    }

    public void doSomething() {
        System.out.println("Something -> AutoCloseableResources_Second");
    }

    @Override
    public void close() throws Exception {
        System.out.println("Closed AutoCloseableResources_Second");
    }
}
```

代码：

```java
private void orderOfClosingResources() throws Exception {
    try (AutoCloseableResourcesFirst af = new AutoCloseableResourcesFirst();
        AutoCloseableResourcesSecond as = new AutoCloseableResourcesSecond()) {

        af.doSomething();
        as.doSomething();
    }
}
```

输出：

```text
Constructor -> AutoCloseableResources_First
Constructor -> AutoCloseableResources_Second
Something -> AutoCloseableResources_First
Something -> AutoCloseableResources_Second
Closed AutoCloseableResources_Second
Closed AutoCloseableResources_First
```

## 7. catch和finally

**try-with-resources块仍然可以有catch和finally块**，它们的工作方式与传统的try块相同。

## 8.Java 9-有效最终变量

在Java 9之前，我们只能在try-with-resources块中使用新变量：

```java
try (Scanner scanner = new Scanner(new File("testRead.txt")); 
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) { 
    // omitted
}
```

如上所示，这在声明多个资源时尤其冗长。从Java 9开始，作为[JEP 213](https://openjdk.java.net/jeps/213)的一部分，**我们现在可以在try-with-resources块中使用[final甚至有效final](https://www.baeldung.com/java-effectively-final)变量**：

```java
final Scanner scanner = new Scanner(new File("testRead.txt"));
PrintWriter writer = new PrintWriter(new File("testWrite.txt"))
try (scanner;writer) { 
    // omitted
}
```

简而言之，如果一个变量在第一次赋值后没有改变，那么它实际上是最终的，即使它没有明确标记为final。

如上所示，scanner变量被显式声明为final，因此我们可以将其与try-with-resources块一起使用。而writer变量虽然不是明确的final，但它在第一次赋值后不会改变。因此，我们也可以使用writer变量。

## 9. 总结

在本文中，我们讨论了如何使用try-with-resources以及如何用try-with-resources替换try、catch和finally。

我们还研究了使用AutoCloseable构建自定义资源以及资源关闭的顺序。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-2)上获得。