---
layout: post
title:  在Java中读取同一行中的多个输入
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

[Scanner](https://www.baeldung.com/java-scanner)类是Java中用于从控制台读取输入的有用工具，我们经常使用next()或nextLine()方法在单独的行上读取每个输入。但是，有时我们可能希望在同一行上读取多个输入。

**在本教程中，我们将探索实现此目的的不同方法，例如使用空格或自定义分隔符，甚至正则表达式**。

## 2. 在同一行读取多个输入

**要在同一行读取多个输入，我们可以使用Scanner类以及next()或nextLine()方法。但是，如果我们使用分隔符来分隔每个输入，效果会更好**。

### 2.1 使用空格作为分隔符

在同一行读取多个输入的一种方法是使用空格作为分隔符，以下是示例：

```java
Scanner scanner = new Scanner(System.in);
System.out.print("Enter two numbers: ");
int num1 = scanner.nextInt();
int num2 = scanner.nextInt();
System.out.println("You entered " + num1 + " and " + num2);
```

在此示例中，我们使用nextInt()方法从控制台读取两个整数。**由于我们在同一行读取它们，因此我们使用空格作为分隔符来分隔这两个整数**。

### 2.2 使用自定义分隔符

如果我们不想使用空格作为分隔符，我们可以通过在Scanner对象上调用setDelimiter()方法来使用自定义分隔符，以下是示例：

```java
Scanner scanner = new Scanner(System.in);
scanner.useDelimiter(";");
System.out.print("Enter two numbers separated by a semicolon: ");
int num1 = scanner.nextInt();
int num2 = scanner.nextInt();
System.out.println("You entered " + num1 + " and " + num2);
```

在此示例中，我们使用分号作为分隔符，而不是空格。**我们还调用setDelimiter()方法将分隔符设置为分号**。

### 2.3 使用正则表达式作为分隔符

除了使用空格或自定义分隔符外，我们还可以在读取同一行上的多个输入时使用正则表达式作为分隔符，**正则表达式是一种可以灵活而强大地匹配字符串的模式**。

例如，如果我们想在同一行读取多个以空格或逗号分隔的输入，我们可以使用以下代码：

```java
Scanner scanner = new Scanner(System.in);
scanner.useDelimiter("[\\s,]+");
System.out.print("Enter two numbers separated by a space or a comma: ");
int num1 = scanner.nextInt();
int num2 = scanner.nextInt();
System.out.println("You entered " + num1 + " and " + num2);
```

在本例中，我们使用正则表达式[\\\\s,\]+作为分隔符，此正则表达式匹配一个或多个空格或逗号。

## 3. 错误处理

在同一行读取多个输入时，[处理可能发生的错误](https://www.baeldung.com/java-exceptions)非常重要。**例如，如果用户输入了无效的输入(例如输入的是String而不是Integer)，程序将抛出异常**。

为了处理这个错误，我们可以使用try-catch块来捕获并妥善处理异常，以下是示例：

```java
Scanner scanner = new Scanner(System.in);
scanner.useDelimiter(";");
System.out.print("Enter two numbers separated by a semicolon: ");
try {
    int num1 = scanner.nextInt();
    int num2 = scanner.nextInt();
    System.out.println("You entered " + num1 + " and " + num2);
} catch (InputMismatchException e) {
    System.out.println("Invalid input. Please enter two integers separated by a semicolon.");
}
```

在此示例中，我们使用try-catch块来捕获用户输入无效输入时可能引发的[InputMismatchException](https://www.baeldung.com/java-scanner-integer#when-the-input-is-in-an-invalid-number-format)，如果捕获到此异常，我们将打印错误消息并要求用户重新输入。

## 4. 总结

在本文中，我们讨论如何使用Scanner类读取同一行上的多个输入。