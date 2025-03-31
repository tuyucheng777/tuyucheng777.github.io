---
layout: post
title:  使用Lucene进行简单文件搜索
category: libraries
copyright: libraries
excerpt: Lucene
---

## 1. 概述

Apache Lucene是一个全文搜索引擎，可供各种编程语言使用。要开始使用Lucene，请参阅[此处](https://www.baeldung.com/lucene)的介绍文章。

在这篇简短的文章中，我们将索引一个文本文件并在该文件中搜索示例字符串和文本片段。

## 2. Maven设置

让我们首先添加必要的依赖：

```xml
<dependency>        
    <groupId>org.apache.lucene</groupId>          
    <artifactId>lucene-core</artifactId>
    <version>7.1.0</version>
</dependency>
```

最新版本可以在[这里](https://mvnrepository.com/artifact/org.apache.lucene/lucene-core)找到。

此外，为了解析我们的搜索查询，我们需要：

```xml
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>7.1.0</version>
</dependency>
```

可以在[这里](https://mvnrepository.com/artifact/org.apache.lucene/lucene-queryparser)查看最新版本。

## 3. 文件系统目录

**为了索引文件，我们首先需要创建一个文件系统索引**。

Lucene提供了FSDirectory类来创建文件系统索引：

```java
Directory directory = FSDirectory.open(Paths.get(indexPath));
```

这里的indexPath是目录的位置，**如果目录不存在，Lucene将创建它**。

Lucene提供了抽象FSDirectory类的三个具体实现：SimpleFSDirectory、NIOFSDirectory和MMapDirectory，它们每个都可能针对特定环境存在特殊问题。

例如，**SimpleFSDirectory的并发性能较差，因为当多个线程读取同一个文件时它会阻塞**。

类似地，**NIOFSDirectory和MMapDirectory实现分别面临Windows中的文件通道问题和内存释放问题**。

**为了克服这种环境特性，Lucene提供了FSDirectory.open()方法**。调用时，它会尝试根据环境选择最佳实现。

## 4. 索引文本文件

一旦我们创建了索引目录，我们就可以继续将文件添加到索引中：

```java
public void addFileToIndex(String filepath) {
    Path path = Paths.get(filepath);
    File file = path.toFile();
    IndexWriterConfig indexWriterConfig = new IndexWriterConfig(analyzer);
    Directory indexDirectory = FSDirectory
            .open(Paths.get(indexPath));
    IndexWriter indexWriter = new IndexWriter(indexDirectory, indexWriterConfig);
    Document document = new Document();

    FileReader fileReader = new FileReader(file);
    document.add(new TextField("contents", fileReader));
    document.add(new StringField("path", file.getPath(), Field.Store.YES));
    document.add(new StringField("filename", file.getName(), Field.Store.YES));

    indexWriter.addDocument(document);
    indexWriter.close();
}
```

在这里，我们创建一个文档，其中包含两个名为“path”和“filename”的StringField以及一个名为“contents”的TextField。

注意，我们将fileReader实例作为第2个参数传递给TextField，使用IndexWriter将文档添加到索引中。

TextField或StringField构造函数中的第3个参数表示是否也存储该字段的值。

**最后，我们调用IndexWriter的close()来正常关闭并释放索引文件的锁**。

## 5. 搜索索引文件

现在让我们搜索已索引的文件：

```java
public List<Document> searchFiles(String inField, String queryString) {
    Query query = new QueryParser(inField, analyzer)
            .parse(queryString);
    Directory indexDirectory = FSDirectory
            .open(Paths.get(indexPath));
    IndexReader indexReader = DirectoryReader
            .open(indexDirectory);
    IndexSearcher searcher = new IndexSearcher(indexReader);
    TopDocs topDocs = searcher.search(query, 10);

    return topDocs.scoreDocs.stream()
            .map(scoreDoc -> searcher.doc(scoreDoc.doc))
            .collect(Collectors.toList());
}
```

现在让我们测试一下功能：

```java
@Test
public void givenSearchQueryWhenFetchedFileNamehenCorrect(){
    String indexPath = "/tmp/index";
    String dataPath = "/tmp/data/file1.txt";

    Directory directory = FSDirectory
            .open(Paths.get(indexPath));
    LuceneFileSearch luceneFileSearch = new LuceneFileSearch(directory, new StandardAnalyzer());

    luceneFileSearch.addFileToIndex(dataPath);

    List<Document> docs = luceneFileSearch
            .searchFiles("contents", "consectetur");

    assertEquals("file1.txt", docs.get(0).get("filename"));
}
```

注意我们如何在位置indexPath中创建文件系统索引并为file1.txt建立索引。

然后，我们只需在“contents”字段中搜索字符串“consectetur”。

## 6. 总结

本文简要演示了如何使用Apache Lucene来索引和搜索文本。要了解有关Lucene的索引、搜索和查询的更多信息，请参阅我们的[Lucene简介文章](https://www.baeldung.com/lucene)。