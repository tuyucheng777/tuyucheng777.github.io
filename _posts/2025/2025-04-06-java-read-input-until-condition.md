---
layout: post
title:  读取用户输入直到满足条件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

当我们编写Java应用程序来接收用户输入时，可能有两种情况：单行输入和多行输入。

在单行输入的情况下，处理起来非常简单，我们读取输入，直到看到换行符。但是，我们需要以不同的方式管理多行用户输入。

在本教程中，我们将讨论如何在Java中处理多行用户输入。

## 2. 解决问题的思路

在Java中，我们可以使用[Scanner](https://www.baeldung.com/java-scanner)类[从用户输入中读取数据](https://www.baeldung.com/java-console-input-output#reading-from-systemin)。因此，从用户输入中读取数据对我们来说并不困难。但是，如果我们允许用户输入多行数据，我们应该知道用户什么时候给出了我们应该接收的所有数据。换句话说，我们需要一个事件来知道什么时候应该停止读取用户输入。

一种常用的方法是我们检查用户发送的数据，如果数据符合定义的条件，我们将停止读取输入数据。在实践中，此条件可能因要求而异。

**解决这个问题的一个想法是编写一个[无限循环](https://www.baeldung.com/infinite-loops-java)来逐行读取用户输入，在循环中，我们检查用户发送的每一行**。一旦满足条件，我们就中断无限循环：

```java
while (true) {
    String line = ... //get one input line
    if (matchTheCondition(line)) {
        break;
    }
    // ... save or use the input data ...
}
```

接下来，让我们创建一个方法来实现我们的想法。

## 3. 使用无限循环解决问题

为简单起见，**在本教程中，一旦我们的应用程序收到字符串”bye”(不区分大小写)，我们就停止读取输入**。

因此，按照我们之前讲过的思路，我们可以创建一个方法来解决这个问题：

```java
public static List<String> readUserInput() {
    List<String> userData = new ArrayList<>();
    System.out.println("Please enter your data below: (send 'bye' to exit) ");
    Scanner input = new Scanner(System.in);
    while (true) {
        String line = input.nextLine();
        if ("bye".equalsIgnoreCase(line)) {
            break;
        }
        userData.add(line);
    }
    return userData;
}
```

如上面的代码所示，readUserInput方法从System.in读取用户输入并将数据存储在userData List中。

一旦我们收到用户的“bye”，我们就会中断无限的while循环。换句话说，我们停止读取用户输入并返回userData以供进一步处理。

接下来，我们在[main方法](https://www.baeldung.com/java-main-method)中调用readUserInput方法：

```java
public static void main(String[] args) {
    List<String> userData = readUserInput();
    System.out.printf("User Input Data:\n%s", String.join("\n", userData));
}
```

在main方法中我们可以看到，调用readUserInput之后，打印出接收到的用户输入数据。

现在，让我们启动应用程序，看看它是否按预期工作。

当应用程序启动时，它会等待我们的输入并提示：

```text
Please enter your data below: (send 'bye' to exit)
```

所以，让我们发送一些文本并在最后发送“bye”：

```text
Hello there,
Today is 19. Mar. 2022.
Have a nice day!
bye
```

输入”bye”并按下Enter后，应用程序输出我们收集到的用户输入数据并退出：

```text
User Input Data:
Hello there,
Today is 19. Mar. 2022.
Have a nice day!
```

正如我们所见，该方法按预期工作。

## 4. 对解决方案进行单元测试

我们已经解决了问题并进行了手动测试，但是，我们可能需要不时调整方法以适应一些新的要求。因此，如果我们能够自动测试该方法就好了。

编写单元测试来测试readUserInput方法与常规测试有点不同，这是因为**当调用readUserInput方法时，应用程序被阻塞并等待用户输入**。

接下来我们先看看测试方法，然后再说明问题是如何解决的：

```java
@Test
public void givenDataInSystemIn_whenCallingReadUserInputMethod_thenHaveUserInputData() {
    String[] inputLines = new String[]{
            "The first line.",
            "The second line.",
            "The last line.",
            "bye",
            "anything after 'bye' will be ignored"
    };
    String[] expectedLines = Arrays.copyOf(inputLines, inputLines.length - 2);
    List<String> expected = Arrays.stream(expectedLines).collect(Collectors.toList());

    InputStream stdin = System.in;
    try {
        System.setIn(new ByteArrayInputStream(String.join("\n", inputLines).getBytes()));
        List<String> actual = UserInputHandler.readUserInput();
        assertThat(actual).isEqualTo(expected);
    } finally {
        System.setIn(stdin);
    }
}
```

现在，让我们快速浏览一下该方法并了解其工作原理。

一开始，我们创建了一个字符串数组inputLines来保存我们想要用作用户输入的行。然后，我们初始化了预期的List，其中包含预期的数据。

接下来，棘手的部分来了，在我们将当前System.in对象备份到stdin变量中之后，我们通过调用System.setIn方法重新分配了系统标准输入。

在这种情况下，**我们要使用inputLines数组来模拟用户输入**。

因此，我们将[数组转换为InputStream](https://www.baeldung.com/convert-byte-array-to-input-stream)(在本例中为ByteArrayInputStream对象)，并将InputStream对象重新分配为系统标准输入。

然后，我们可以调用目标方法并测试结果是否符合预期。

最后，**我们不应该忘记恢复原来的stdin对象作为系统标准输入**。因此，我们将System.setIn(stdin);放在[finally](https://www.baeldung.com/java-finally-keyword)块中，以确保它无论如何都会被执行。

如果我们运行测试方法，它就会通过，无需任何人工干预。

## 5. 总结

在本文中，我们探讨了如何编写Java方法来读取用户输入，直到满足条件为止。

两个关键技术是：

-   使用标准Java API中的Scanner类读取用户输入
-   在无限循环中检查每一行输入；如果条件满足，则中断循环

此外，我们还讨论了如何编写测试方法来自动测试我们的解决方案。