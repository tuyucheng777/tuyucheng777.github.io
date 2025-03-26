---
layout: post
title:  使用Picocli创建Java命令行程序
category: libraries
copyright: libraries
excerpt: Picocli
---

## 1. 简介

在本教程中，我们将介绍[picocli库](https://picocli.info/)，它使我们能够轻松地在Java中创建命令行程序。

我们将首先开始创建一个Hello World命令。然后，我们将通过部分git命令来深入了解该库的主要功能。

## 2. Hello World命令

让我们从简单的事情开始：Hello World命令！

首先，我们需要将[picocli](https://mvnrepository.com/artifact/info.picocli/picocli)依赖添加到项目pom.xml中：

```xml
<dependency>
    <groupId>info.picocli</groupId>
    <artifactId>picocli</artifactId>
    <version>3.9.6</version>
</dependency>
```

如我们所见，我们将使用该库的4.7.0版本。

现在依赖项已设置完毕，让我们创建Hello World命令。为此，**我们将使用库中的@Command注解**：

```java
@Command(
        name = "hello",
        description = "Says hello"
)
public class HelloWorldCommand {
}
```

正如我们所见，注解可以带参数，我们这里只使用其中两个，它们的目的是为自动帮助消息提供有关当前命令和文本的信息。

目前，我们无法使用此命令做很多事情。为了让它做一些事情，我们需要添加一个调用方便的CommandLine.run(Runnable, String[])方法的main方法。这需要两个参数：一个我们的命令实例，因此必须实现Runnable接口，以及一个表示命令参数(选项、参数和子命令)的字符串数组：

```java
public class HelloWorldCommand implements Runnable {
    public static void main(String[] args) {
        CommandLine.run(new HelloWorldCommand(), args);
    }

    @Override
    public void run() {
        System.out.println("Hello World!");
    }
}
```

现在，当我们运行main方法时，我们会看到控制台输出“Hello World!”

[打包成jar](https://www.baeldung.com/java-create-jar)后，我们可以使用java命令运行我们的Hello World命令：

```shell
java -cp "pathToPicocliJar;pathToCommandJar" cn.tuyucheng.taketoday.picoli.helloworld.HelloWorldCommand
```

毫不奇怪，这也会输出“Hello World!”字符串到控制台。

## 3. 具体用例

现在我们已经了解了基础知识，我们将深入研究picocli库。为了做到这一点，我们将部分重现一个流行的命令：git。

当然，目的不是实现git命令的行为，而是重现git命令的可能性-存在哪些子命令以及特定子命令有哪些选项可用。

首先，我们必须像创建Hello World命令一样创建一个GitCommand类：

```java
@Command
public class GitCommand implements Runnable {
    public static void main(String[] args) {
        CommandLine.run(new GitCommand(), args);
    }

    @Override
    public void run() {
        System.out.println("The popular git command");
    }
}
```

## 4. 添加子命令

git命令提供了很多[子命令](https://picocli.info/#_subcommands)-add、commit、remote等等。我们将在这里关注add和commit。

因此，我们的目标是将这两个子命令声明给主命令，Picocli提供了三种方法来实现这一点。

### 4.1 在类上使用@Command注解

**@Command注解提供了通过subcommands参数注册子命令的可能性**：

```java
@Command(
    subcommands = {
        GitAddCommand.class,
        GitCommitCommand.class
    }
)
```

在我们的例子中，我们添加了两个新类：GitAddCommand和GitCommitCommand，两者都用@Command标注并实现Runnable。**为它们命名很重要，因为picocli将使用这些名称来识别要执行的子命令**：

```java
@Command(
        name = "add"
)
public class GitAddCommand implements Runnable {
    @Override
    public void run() {
        System.out.println("Adding some files to the staging area");
    }
}
```

```java
@Command(
        name = "commit"
)
public class GitCommitCommand implements Runnable {
    @Override
    public void run() {
        System.out.println("Committing files in the staging area, how wonderful?");
    }
}
```

因此，如果我们以add作为参数运行我们的主命令，控制台将输出“Adding some files to the staging area”。

### 4.2 在方法上使用@Command注解

声明子命令的另一种方法是**在GitCommand类中创建表示这些命令的@Command注解方法**：

```java
@Command(name = "add")
public void addCommand() {
    System.out.println("Adding some files to the staging area");
}

@Command(name = "commit")
public void commitCommand() {
    System.out.println("Committing files in the staging area, how wonderful?");
}
```

这样，我们可以直接在方法中实现业务逻辑，而不必创建单独的类来处理它。

### 4.3 以编程方式添加子命令

**最后，picocli为我们提供了以编程方式注册子命令的可能性**。这个有点棘手，因为我们必须创建一个CommandLine对象来包装我们的命令，然后向其中添加子命令：

```java
CommandLine commandLine = new CommandLine(new GitCommand());
commandLine.addSubcommand("add", new GitAddCommand());
commandLine.addSubcommand("commit", new GitCommitCommand());
```

之后，我们仍然需要运行命令，但是我们**不能再使用CommandLine.run()方法**了。现在，我们必须在新创建的CommandLine对象上调用parseWithHandler()方法：

```java
commandLine.parseWithHandler(new RunLast(), args);
```

我们应该注意RunLast类的使用，它告诉picocli运行最具体的子命令。picocli还提供了另外两个命令处理程序：RunFirst和RunAll。前者运行最顶层的命令，而后者运行所有命令。

当使用便捷方法CommandLine.run()时，默认使用RunLast处理程序。

## 5. 使用@Option注解管理选项

### 5.1 无参数选项

现在让我们看看如何为我们的命令添加一些选项。实际上，我们想告诉我们的add命令它应该添加所有修改过的文件。为此，我们将在GitAddCommand类中添加一个带有[@Option](https://picocli.info/#_options)注解的字段：

```java
@Option(names = {"-A", "--all"})
private boolean allFiles;

@Override
public void run() {
    if (allFiles) {
        System.out.println("Adding all files to the staging area");
    } else {
        System.out.println("Adding some files to the staging area");
    }
}
```

正如我们所看到的，注解带有一个names参数，它给出了选项的不同名称。因此，使用-A或–all调用add命令会将allFiles字段设置为true。因此，如果我们使用该选项运行该命令，控制台将显示“Adding all files to the staging area”。

### 5.2 带参数的选项

正如我们刚刚看到的，对于没有参数的选项，它们的存在或不存在总是被评估为一个boolean值。

但是，可以注册带有参数的选项。**我们可以通过将字段声明为不同类型来简单地做到这一点**，让我们向commit命令中添加一个message选项：

```java
@Option(names = {"-m", "--message"})
private String message;

@Override
public void run() {
    System.out.println("Committing files in the staging area, how wonderful?");
    if (message != null) {
        System.out.println("The commit message is " + message);
    }
}
```

毫不奇怪，当给定message选项时，该命令将在控制台上显示提交消息。在本文的后面，我们将介绍库处理哪些类型以及如何处理其他类型。

### 5.3 具有多个参数的选项

但是现在，如果我们希望命令接收多条消息，就像真正的[git commit](https://git-scm.com/docs/git-commit)命令那样呢？不用担心，让我们**将字段设为数组或Collection**，这样就大功告成了：

```java
@Option(names = {"-m", "--message"})
private String[] messages;

@Override
public void run() {
    System.out.println("Committing files in the staging area, how wonderful?");
    if (messages != null) {
        System.out.println("The commit message is");
        for (String message : messages) {
            System.out.println(message);
        }
    }
}
```

现在，我们可以多次使用message选项：

```shell
commit -m "My commit is great" -m "My commit is beautiful"
```

但是，我们可能还希望只提供一次选项，并用正则表达式分隔符分隔不同的参数。因此，我们可以使用@Option注解的split参数：

```java
@Option(names = {"-m", "--message"}, split = ",")
private String[] messages;
```

现在，我们可以通过-m “My commit is great”,”My commit is beautiful”来达到与上面相同的结果。

### 5.4 必选项

**有时，我们可能有一个必需的选项。required参数(默认为false)允许我们这样做**：

```java
@Option(names = {"-m", "--message"}, required = true)
private String[] messages;
```

现在，如果不指定message选项就无法调用commit命令。如果我们尝试这样做，picocli将打印错误：

```text
Missing required option '--message=<messages>'
Usage: git commit -m=<messages> [-m=<messages>]...
  -m, --message=<messages>
```

## 6. 管理位置参数

### 6.1 捕获位置参数

现在，让我们关注我们的add命令，因为它还不是很强大。我们只能决定添加所有文件，但如果我们想添加特定文件怎么办？

我们可以使用另一个选项来做到这一点，但这里更好的选择是使用位置参数。实际上，**位置参数旨在捕获占据特定位置的命令参数，既不是子命令也不是选项**。

在我们的示例中，这将使我们能够执行以下操作：

```shell
add file1 file2
```

为了捕获位置参数，我们将使用[@Parameters](https://picocli.info/#_positional_parameters)注解：

```java
@Parameters
private List<Path> files;

@Override
public void run() {
    if (allFiles) {
        System.out.println("Adding all files to the staging area");
    }

    if (files != null) {
        files.forEach(path -> System.out.println("Adding " + path + " to the staging area"));
    }
}
```

现在，我们之前的命令将打印：

```text
Adding file1 to the staging area
Adding file2 to the staging area
```

### 6.2 捕获位置参数的子集

由于注解的index参数，可以更细粒度地确定要捕获的位置参数。索引从0开始，因此，如果我们定义：

```java
@Parameters(index="2..*")
```

这将捕获与选项或子命令不匹配的参数，从第三个到最后一个。

索引可以是范围或单个数字，代表单个位置。

## 7. 关于类型转换

正如我们在本教程前面看到的那样，picocli本身可以处理一些类型转换。例如，它将多个值映射到数组或Collections，但它也可以将参数映射到特定类型，就像我们将Path类用于add命令时一样。

**事实上，picocli带有一堆[预处理类型](https://picocli.info/#_built_in_types)。这意味着我们可以直接使用这些类型，而不必考虑自己转换它们**。

但是，我们可能需要将我们的命令参数映射到已经处理过的类型以外的类型。对我们来说幸运的是，**这可以通过[ITypeConverter](https://picocli.info/#_custom_type_converters)接口和CommandLine#registerConverter方法实现，该方法将类型与转换器关联起来**。

假设我们想将config子命令添加到git命令，但我们不希望用户更改不存在的配置元素。因此，我们决定将这些元素映射到一个枚举：

```java
public enum ConfigElement {
    USERNAME("user.name"),
    EMAIL("user.email");

    private final String value;

    ConfigElement(String value) {
        this.value = value;
    }

    public String value() {
        return value;
    }

    public static ConfigElement from(String value) {
        return Arrays.stream(values())
                .filter(element -> element.value.equals(value))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("The argument "
                        + value + " doesn't match any ConfigElement"));
    }
}
```

另外，在我们新创建的GitConfigCommand类中，让我们添加两个位置参数：

```java
@Parameters(index = "0")
private ConfigElement element;

@Parameters(index = "1")
private String value;

@Override
public void run() {
    System.out.println("Setting " + element.value() + " to " + value);
}
```

这样，我们确保用户无法更改不存在的配置元素。

最后，我们必须注册我们的转换器。美妙的是，如果使用Java 8或更高版本，我们甚至不必创建实现ITypeConverter接口的类，**只需将Lambda或方法引用传递给registerConverter()方法即可**：

```java
CommandLine commandLine = new CommandLine(new GitCommand());
commandLine.registerConverter(ConfigElement.class, ConfigElement::from);

commandLine.parseWithHandler(new RunLast(), args);
```

这发生在GitCommand main()方法中。请注意，我们不得不放弃方便的CommandLine.run()方法。

当与未处理的配置元素一起使用时，该命令将显示帮助消息以及一条信息，告诉我们无法将参数转换为ConfigElement：

```text
Invalid value for positional parameter at index 0 (<element>): 
cannot convert 'user.phone' to ConfigElement 
(java.lang.IllegalArgumentException: The argument user.phone doesn't match any ConfigElement)
Usage: git config <element> <value>
      <element>
      <value>
```

## 8. 与Spring Boot集成

事实上，我们可能在Spring Boot环境中开发，并希望在我们的命令行程序中从中受益。为此，我****们必须创建一个实现CommandLineRunner接口的SpringBootApplication：

```java
@SpringBootApplication
public class Application implements CommandLineRunner {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
    }
}
```

另外，**让我们用Spring @Component注解来标注我们所有的命令和子命令**，并在我们的Application中自动装配所有这些：

```java
private GitCommand gitCommand;
private GitAddCommand addCommand;
private GitCommitCommand commitCommand;
private GitConfigCommand configCommand;

public Application(GitCommand gitCommand, GitAddCommand addCommand,
                   GitCommitCommand commitCommand, GitConfigCommand configCommand) {
    this.gitCommand = gitCommand;
    this.addCommand = addCommand;
    this.commitCommand = commitCommand;
    this.configCommand = configCommand;
}
```

请注意，我们必须自动装配每个子命令。不幸的是，这是因为目前picocli还不能在以声明式方式声明(使用注解)时从Spring上下文中检索子命令。因此，我们必须以编程方式自己进行注入：

```java
@Override
public void run(String... args) {
    CommandLine commandLine = new CommandLine(gitCommand);
    commandLine.addSubcommand("add", addCommand);
    commandLine.addSubcommand("commit", commitCommand);
    commandLine.addSubcommand("config", configCommand);

    commandLine.parseWithHandler(new CommandLine.RunLast(), args);
}
```

现在，我们的命令行程序可以完美地与Spring组件配合使用。因此，我们可以创建一些服务类并在我们的命令中使用它们，让Spring负责依赖注入。

## 9.总结

在本文中，我们了解了picocli库的一些关键特性。我们学习了如何创建一个新命令并向其添加一些子命令，我们了解了处理选项和位置参数的多种方法。另外，我们还学习了如何实现我们自己的类型转换器来使我们的命令强类型化。最后，我们了解了如何在Spring Boot引入我们的命令。

当然，关于它还有很多东西有待发现。该库提供了[完整的文档](https://picocli.info/)。