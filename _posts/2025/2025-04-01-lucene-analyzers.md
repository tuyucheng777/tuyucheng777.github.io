---
layout: post
title:  Lucene分词器指南
category: libraries
copyright: libraries
excerpt: Lucene
---

## 1. 概述

Lucene分词器用于在索引和搜索文档时分词文本。

我们在[入门教程](https://www.baeldung.com/lucene)中简要提到了分词器。

在本教程中，**我们将讨论常用的分词器、如何构建自定义分词器以及如何为不同的文档字段分配不同的分词器**。

## 2. Maven依赖

首先，我们需要将这些依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-core</artifactId>
    <version>7.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>7.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-analyzers-common</artifactId>
    <version>7.4.0</version>
</dependency>
```

最新的Lucene版本可以在[这里](https://mvnrepository.com/artifact/org.apache.lucene/lucene-core)找到。

## 3. Lucene分词器

Lucene分词器将文本拆分成标记。

**分词器主要由分词器和过滤器组成**，不同的分词器由不同的分词器和过滤器组合而成。

为了演示常用分词器之间的区别，我们将使用以下方法：

```java
public List<String> analyze(String text, Analyzer analyzer) throws IOException{
    List<String> result = new ArrayList<String>();
    TokenStream tokenStream = analyzer.tokenStream(FIELD_NAME, text);
    CharTermAttribute attr = tokenStream.addAttribute(CharTermAttribute.class);
    tokenStream.reset();
    while(tokenStream.incrementToken()) {
        result.add(attr.toString());
    }
    return result;
}
```

此方法使用给定的分词器将给定的文本转换为标记列表。

## 4. 常见的Lucene分词器 

现在我们来了解一下一些常用的Lucene分词器。

### 4.1 StandardAnalyzer

我们将从最常用的分词器StandardAnalyzer开始：

```java
private static final String SAMPLE_TEXT = "This is tuyucheng.com Lucene Analyzers test";

@Test
public void whenUseStandardAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new StandardAnalyzer());

    assertThat(result, contains("tuyucheng.com", "lucene", "analyzers","test"));
}
```

请注意，StandardAnalyzer可以识别URL和电子邮件。

此外，它还会删除停用词并将生成的标记变为小写。

### 4.2 StopAnalyzer

StopAnalyzer由LetterTokenizer、LowerCaseFilter和StopFilter组成：

```java
@Test
public void whenUseStopAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new StopAnalyzer());

    assertThat(result, contains("tuyucheng", "com", "lucene", "analyzers", "test"));
}
```

在这个例子中，LetterTokenizer按非字母字符分割文本，而StopFilter从标记列表中删除停用词。

但是，与StandardAnalyzer不同，StopAnalyzer无法识别URL。

### 4.3 SimpleAnalyzer

SimpleAnalyzer由LetterTokenizer和LowerCaseFilter组成：

```java
@Test
public void whenUseSimpleAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new SimpleAnalyzer());

    assertThat(result, contains("this", "is", "tuyucheng", "com", "lucene", "analyzers", "test"));
}
```

这里，SimpleAnalyzer不会删除停用词，它也无法识别URL。

### 4.4 WhitespaceAnalyzer

WhitespaceAnalyzer仅使用WhitespaceTokenizer，它通过空格字符分割文本：

```java
@Test
public void whenUseWhiteSpaceAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new WhitespaceAnalyzer());

    assertThat(result, contains("This", "is", "tuyucheng.com", "Lucene", "Analyzers", "test"));
}
```

### 4.5 KeywordAnalyzer

KeywordAnalyzer将输入标记为单个标记：

```java
@Test
public void whenUseKeywordAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new KeywordAnalyzer());

    assertThat(result, contains("This is tuyucheng.com Lucene Analyzers test"));
}
```

KeywordAnalyzer对于像ids和zipcodes这样的字段很有用。

### 4.6 语言分词器

还有针对不同语言的特殊分词器，如EnglishAnalyzer、FrenchAnalyzer和SpanishAnalyzer：

```java
@Test
public void whenUseEnglishAnalyzer_thenAnalyzed() throws IOException {
    List<String> result = analyze(SAMPLE_TEXT, new EnglishAnalyzer());

    assertThat(result, contains("tuyucheng.com", "lucen", "analyz", "test"));
}
```

在这里，我们使用由StandardTokenizer、StandardFilter、EnglishPossessiveFilter、LowerCaseFilter、StopFilter和PorterStemFilter组成的EnglishAnalyzer。

## 5. 自定义分词器 

接下来，让我们看看如何构建自定义分词器，我们将以两种不同的方式构建相同的自定义分词器。

在第一个例子中，**我们将使用CustomAnalyzer构建器从预定义的标记器和过滤器构建我们的分词器**：

```java
@Test
public void whenUseCustomAnalyzerBuilder_thenAnalyzed() throws IOException {
    Analyzer analyzer = CustomAnalyzer.builder()
            .withTokenizer("standard")
            .addTokenFilter("lowercase")
            .addTokenFilter("stop")
            .addTokenFilter("porterstem")
            .addTokenFilter("capitalization")
            .build();
    List<String> result = analyze(SAMPLE_TEXT, analyzer);

    assertThat(result, contains("Tuyucheng.com", "Lucen", "Analyz", "Test"));
}
```

我们的分词器与EnglishAnalyzer非常相似，但它将标记大写。

在第二个示例中，**我们将通过扩展Analyzer抽象类并重写createComponents()方法来构建相同的分词器**：

```java
public class MyCustomAnalyzer extends Analyzer {

    @Override
    protected TokenStreamComponents createComponents(String fieldName) {
        StandardTokenizer src = new StandardTokenizer();
        TokenStream result = new StandardFilter(src);
        result = new LowerCaseFilter(result);
        result = new StopFilter(result,  StandardAnalyzer.STOP_WORDS_SET);
        result = new PorterStemFilter(result);
        result = new CapitalizationFilter(result);
        return new TokenStreamComponents(src, result);
    }
}
```

如果需要，我们还可以创建自定义标记器或过滤器，并将其添加到自定义分词器中。

现在，让我们看看自定义分词器的实际运行情况-我们将在此示例中使用[InMemoryLuceneIndex](https://www.baeldung.com/lucene)：

```java
@Test
public void givenTermQuery_whenUseCustomAnalyzer_thenCorrect() {
    InMemoryLuceneIndex luceneIndex = new InMemoryLuceneIndex(
            new RAMDirectory(), new MyCustomAnalyzer());
    luceneIndex.indexDocument("introduction", "introduction to lucene");
    luceneIndex.indexDocument("analyzers", "guide to lucene analyzers");
    Query query = new TermQuery(new Term("body", "Introduct"));

    List<Document> documents = luceneIndex.searchIndex(query);
    assertEquals(1, documents.size());
}
```

## 6. PerFieldAnalyzerWrapper

最后，**我们可以使用PerFieldAnalyzerWrapper为不同的字段分配不同的分词器**。

首先，我们需要定义analyzerMap来将每个分词器映射到特定字段：

```java
Map<String,Analyzer> analyzerMap = new HashMap<>();
analyzerMap.put("title", new MyCustomAnalyzer());
analyzerMap.put("body", new EnglishAnalyzer());
```

我们将“title”映射到我们的自定义分词器，将“body”映射到EnglishAnalyzer。

接下来，让我们通过提供analyzerMap和默认Analyzer来创建我们的PerFieldAnalyzerWrapper：

```java
PerFieldAnalyzerWrapper wrapper = new PerFieldAnalyzerWrapper(new StandardAnalyzer(), analyzerMap);
```

现在，让我们测试一下：

```java
@Test
public void givenTermQuery_whenUsePerFieldAnalyzerWrapper_thenCorrect() {
    InMemoryLuceneIndex luceneIndex = new InMemoryLuceneIndex(new RAMDirectory(), wrapper);
    luceneIndex.indexDocument("introduction", "introduction to lucene");
    luceneIndex.indexDocument("analyzers", "guide to lucene analyzers");

    Query query = new TermQuery(new Term("body", "introduct"));
    List<Document> documents = luceneIndex.searchIndex(query);
    assertEquals(1, documents.size());

    query = new TermQuery(new Term("title", "Introduct"));
    documents = luceneIndex.searchIndex(query);
    assertEquals(1, documents.size());
}
```

## 7. 总结

我们讨论了流行的Lucene分词器、如何构建自定义分词器以及如何针对每个字段使用不同的分词器。