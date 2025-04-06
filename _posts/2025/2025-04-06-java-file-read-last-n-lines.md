---
layout: post
title:  使用Java读取文件最后N行
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在本文中，我们将了解如何使用不同的标准Java包和Apache Commons IO库从文件中读取最后N行。

## 2. 样本数据

我们将在本教程的所有示例中使用下面定义的示例数据和参数。

让我们首先创建一个名为data.txt的简单文件，将其用作输入文件：

```text
line 1
line 2
line 3
line 4
line 5
line 6
line 7
line 8
line 9
line 10
```

此外，我们将使用以下示例值：要读取的最后N行数、要验证的输出和文件路径：

```java
private static final String FILE_PATH = "src/test/resources/data.txt";
private static final int LAST_LINES_TO_READ = 3;
private static final String OUTPUT_TO_VERIFY = "line 8\nline 9\nline 10";
```

## 3. 使用BufferedReader

让我们探索一下BufferedReader类，它允许我们逐行读取文件。**它的优点是不将整个文件存储在内存中**，我们可以使用FIFO结构的队列，在读取文件时，只要队列大小达到要读取的行数，我们就会开始删除第一个元素：

```java
@Test
public void givenFile_whenUsingBufferedReader_thenExtractedLastLinesCorrect() throws IOException {
    try (BufferedReader br = Files.newBufferedReader(Paths.get(FILE_PATH))) {
        Queue<String> queue = new LinkedList<>();
        String line;
        while ((line = br.readLine()) != null){
            if (queue.size() >= LAST_LINES_TO_READ) {
                queue.remove();
            }
            queue.add(line);
        }

        assertEquals(OUTPUT_TO_VERIFY, String.join("\n", queue));
    }
}
```

## 4. 使用Scanner

我们可以使用Scanner类以类似的方法实现相同的结果：

```java
@Test
public void givenFile_whenUsingScanner_thenExtractedLastLinesCorrect() throws IOException {
    try (Scanner scanner = new Scanner(new File(FILE_PATH))) {
        Queue<String> queue = new LinkedList<>();
        while (scanner.hasNextLine()){
            if (queue.size() >= LAST_LINES_TO_READ) {
                queue.remove();
            }
            queue.add(scanner.nextLine());
        }

        assertEquals(OUTPUT_TO_VERIFY, String.join("\n", queue));
    }
}
```

## 5. 使用NIO2 Files

如果我们要处理大文件，我们可以使用Files类，它的Lines方法提供了一个流来逐行读取文件。之后，我们将使用类似的Queue方法来读取所需的内容：

```java
@Test
public void givenLargeFile_whenUsingFilesAPI_thenExtractedLastLinesCorrect() throws IOException{
    try (Stream<String> lines = Files.lines(Paths.get(FILE_PATH))) {
        Queue<String> queue = new LinkedList<>();
        lines.forEach(line -> {
            if (queue.size() >= LAST_LINES_TO_READ) {
                queue.remove();
            }
            queue.add(line);
        });

        assertEquals(OUTPUT_TO_VERIFY, String.join("\n", queue));
    }
}
```

## 6. 使用Apache Commons IO

我们可以使用[Apache Commons IO](https://www.baeldung.com/apache-commons-io)库提供的FileUtils类和ReversedLinesFileReader类。

### 6.1 使用FileUtils类读取文件

此类提供了readLines方法，我们可以使用它来读取列表中的整个文件，**这会导致整个文件内容存储在内存中**，我们可以遍历列表并读取所需的内容：

```java
@Test
public void givenFile_whenUsingFileUtils_thenExtractedLastLinesCorrect() throws IOException{
    File file = new File(FILE_PATH);
    List<String> lines = FileUtils.readLines(file, "UTF-8");
    StringBuilder stringBuilder = new StringBuilder();
    for (int i = (lines.size() - LAST_LINES_TO_READ); i < lines.size(); i++) {
        stringBuilder.append(lines.get(i)).append("\n");
    }

    assertEquals(OUTPUT_TO_VERIFY, stringBuilder.toString().trim());
}
```

### 6.2 使用ReversedLinesFileReader类读取文件

此类允许使用其readLines方法以相反的顺序读取文件，**这有助于直接从最后读取所需的内容，而无需应用任何其他逻辑或跳过文件内容**：

```java
@Test
public void givenFile_whenUsingReverseFileReader_thenExtractedLastLinesCorrect() throws IOException{
    File file = new File(FILE_PATH);
    try (ReversedLinesFileReader rlfReader = new ReversedLinesFileReader(file, StandardCharsets.UTF_8)) {
        List<String> lastLines = rlfReader.readLines(LAST_LINES_TO_READ);
        StringBuilder stringBuilder = new StringBuilder();
        Collections.reverse(lastLines);
        lastLines.forEach(
          line -> stringBuilder.append(line).append("\n")
        );

        assertEquals(OUTPUT_TO_VERIFY, stringBuilder.toString().trim());
    }
}
```

## 7. 总结

在本文中，我们研究了从文件中读取最后N行的不同方法，我们应该考虑应用程序是否可以承受更多的CPU使用率或更多的内存使用率来选择方法。