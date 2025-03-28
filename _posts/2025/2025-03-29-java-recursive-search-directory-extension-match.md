---
layout: post
title:  Java中按扩展名在指定目录中查找文件
category: libraries
copyright: libraries
excerpt: Java
---

## 1. 简介

在本快速教程中，我们将了解一些使用核心Java和外部库在目录(包括子目录)中搜索与特定扩展名匹配的文件的替代方法，我们将从简单的数组和列表介绍到流和其他较新的方法。

## 2. 设置过滤器

**由于我们需要按扩展名过滤文件，因此让我们从一个简单的Predicate实现开始**。我们需要进行一些输入清理，以确保我们符合大多数用例，例如是否接受以点开头的扩展名：

```java
public class MatchExtensionPredicate implements Predicate<Path> {

    private final String extension;

    public MatchExtensionPredicate(String extension) {
        if (!extension.startsWith(".")) {
            extension = "." + extension;
        }
        this.extension = extension.toLowerCase();
    }

    @Override
    public boolean test(Path path) {
        if (path == null) {
            return false;
        }
        return path.getFileName()
                .toString()
                .toLowerCase()
                .endsWith(extension);
    }
}
```

我们首先编写构造函数，在扩展名前添加一个点(如果扩展名中没有点)。然后，我们将其转换为小写。这样，当我们将其与其他文件进行比较时，我们就可以确保它们的大小写相同。**最后，我们通过获取Path的文件名并将其转换为小写来实现test()。最重要的是，我们检查它是否以我们要查找的扩展名结尾**。

## 3. 使用Files.listFiles()遍历目录

我们的第一个示例将使用自Java诞生以来就存在的方法：Files.listFiles()，让我们首先实例化一个List来存储结果并列出目录中的所有文件：

```java
List<File> find(File startPath, String extension) {
    List<File> matches = new ArrayList<>();

    File[] files = startPath.listFiles();
    if (files == null) {
        return matches;
    }

    // ...
}
```

**就其本身而言，listFiles()并不递归操作，因此对于每个元素，如果我们确定它是一个目录，我们就开始递归**：

```java
MatchExtensionPredicate filter = new MatchExtensionPredicate(extension);
for (File file : files) {
    if (file.isDirectory()) {
        matches.addAll(find(file, extension));
    } else if (filter.test(file.toPath())) {
        matches.add(file);
    }
}

return matches;
```

我们还实例化了过滤器，并且只有当前文件通过了test()实现，才会将其添加到列表中。**最终，我们将获得与过滤器匹配的所有结果**。请注意，这可能会导致目录树太深时出现StackOverflowError，而包含太多文件的目录则会出现OutOfMemoryError。我们稍后会看到性能更好的选项。

## 4. 从Java 7开始使用Files.walkFileTree()遍历目录

从Java 7开始，我们有了[NIO2 API](https://www.baeldung.com/java-nio-2-file-api)，它包含许多实用程序，例如Files类和使用Path类处理文件的新方法。**使用walkFileTree()可以让我们毫不费力地递归遍历目录**，此方法只需要一个起始Path和一个[FileVisitor](https://www.baeldung.com/java-nio2-file-visitor)实现：

```java
List<Path> find(Path startPath, String extension) throws IOException {
    List<Path> matches = new ArrayList<>();

    Files.walkFileTree(startPath, new SimpleFileVisitor<Path>() {

        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attributes) {
            if (new MatchExtensionPredicate(extension).test(file)) {
                matches.add(file);
            }
            return FileVisitResult.CONTINUE;
        }

        @Override
        public FileVisitResult visitFileFailed(Path file, IOException exc) {
            return FileVisitResult.CONTINUE;
        }
    });
    return matches;
}
```

FileVisitor包含几个事件的回调：进入目录前、离开目录后、访问文件时以及访问失败时。**但是，使用SimpleFileVisitor，我们只需实现我们感兴趣的回调**。在本例中，它使用visitFile()访问文件。因此，对于访问的每个文件，我们都会根据Predicate对其进行测试，并将其添加到匹配文件列表中。

此外，我们实现visitFileFailed()以始终返回FileVisitResult.CONTINUE。**这样，即使发生异常(例如拒绝访问)，我们也可以继续搜索文件**。

## 5. 从Java 8开始使用Files.walk()进行流式传输

Java 8包含了一种与[Stream API](https://www.baeldung.com/java-8-streams)集成的更简单的遍历目录的方法，以下是使用Files.walk()的方法：

```java
Stream<Path> find(Path startPath, String extension) throws IOException {
    return Files.walk(startPath)
            .filter(new MatchExtensionPredicate(extension));
}
```

**不幸的是，第一次抛出异常时，这个方法就失效了，目前还[没有办法](https://bugs.openjdk.org/browse/JDK-8039910)处理**。所以，让我们尝试一种不同的方法。我们将从实现一个包含Consumer<Path\>的FileVisitor开始，**这次，我们将使用这个Consumer对文件匹配项执行任何我们想做的事情，而不是将它们累积在List中**：

```java
public class SimpleFileConsumerVisitor extends SimpleFileVisitor<Path> {

    private final Predicate<Path> filter;
    private final Consumer<Path> consumer;

    public SimpleFileConsumerVisitor(MatchExtensionPredicate filter, Consumer<Path> consumer) {
        this.filter = filter;
        this.consumer = consumer;
    }

    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attributes) {
        if (filter.test(file)) {
            consumer.accept(file);
        }
        return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
        return FileVisitResult.CONTINUE;
    }
}
```

最后，让我们修改find()方法以使用它：

```java
void find(Path startPath, String extension, Consumer<Path> consumer) throws IOException {
    MatchExtensionPredicate filter = new MatchExtensionPredicate(extension);
    Files.walkFileTree(startPath, new SimpleFileConsumerVisitor(filter, consumer));
}
```

请注意，我们必须返回到Files.walkFileTree()才能使用我们的FileVisitor实现。

## 6. 使用Apache Commons IO的FileUtils.iterateFiles()

另一个有用的选项是[Apache Commons IO](https://www.baeldung.com/apache-commons-io)中的FileUtils.iterateFiles()，它返回一个Iterator。让我们包含它的[依赖](https://mvnrepository.com/artifact/commons-io/commons-io)：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.15.1</version>
</dependency>
```

**有了依赖后，我们可以使用WildcardFileFilter代替MatchExtensionPredicate**：

```java
Iterator<File> find(Path startPath, String extension) {
    if (!extension.startsWith(".")) {
        extension = "." + extension;
    }
    return FileUtils.iterateFiles(
        startPath.toFile(), 
        WildcardFileFilter.builder().setWildcards("*" + extension).get(), 
        TrueFileFilter.INSTANCE);
}
```

我们首先确保扩展名符合预期格式，检查是否需要在前面添加一个点，这样我们的方法在传递“.extension”或仅传递“extension”时才有效。

与其他方法一样，它只需要一个起始目录。但是，由于这是一个较旧的API，因此它需要一个File而不是Path。最后一个参数是一个可选的目录过滤器，但是，如果未指定，它会忽略子目录。因此，我们包含一个TrueFileFilter.INSTANCE以确保访问整个目录树。

## 7. 总结

在本文中，我们探讨了根据指定扩展名在目录及其子目录中搜索文件的各种方法。我们首先设置了一个与Predicate匹配的灵活扩展，然后，我们介绍了不同的技术，从传统的Files.listFiles()和Files.walkFileTree()方法到Java 8中引入的更现代的替代方案，例如Files.walk()。此外，我们还从不同的角度探讨了Apache Commons IO的FileUtils.iterateFiles()的用法。