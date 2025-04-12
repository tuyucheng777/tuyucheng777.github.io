---
layout: post
title:  Apache OpenNLP简介
category: apache
copyright: apache
excerpt: Apache OpenNLP
---

## 1. 概述

Apache OpenNLP是一个开源自然语言处理Java库。

它具有用于命名实体识别、句子检测、POS标记和标记化等用例的API。

在本教程中，我们将了解如何将此API用于不同的用例。

## 2. Maven设置

首先，我们需要将主要依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.opennlp</groupId>
    <artifactId>opennlp-tools</artifactId>
    <version>1.8.4</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/search?q=opennlp-tools)上找到最新的稳定版本。

某些用例需要经过训练的模型，你可以在[此处](http://opennlp.sourceforge.net/models-1.5/)下载预定义模型，并在[此处](http://opennlp.apache.org/models.html)获取有关这些模型的详细信息。

## 3. 句子检测

让我们首先了解什么是句子。

**句子检测是指识别句子的起始和结束**，这通常取决于所使用的语言，这也称为“句子边界消歧”(SBD)。 

在某些情况下，**由于句号字符的歧义性，句子检测相当具有挑战性**。句号通常表示句子的结束，但也可能出现在电子邮件地址、缩写、小数点以及许多其他地方。

与大多数NLP任务一样，对于句子检测，我们需要一个训练有素的模型作为输入，我们希望它驻留在/resources文件夹中。

为了实现句子检测，我们加载模型并将其传递给SentenceDetectorME的一个实例。然后，我们只需将文本传递给sentDetect()方法，即可在句子边界处进行拆分：

```java
@Test
public void givenEnglishModel_whenDetect_thenSentencesAreDetected() throws Exception {
    String paragraph = "This is a statement. This is another statement."
            + "Now is an abstract word for time, "
            + "that is always flying. And my email address is google@gmail.com.";

    InputStream is = getClass().getResourceAsStream("/models/en-sent.bin");
    SentenceModel model = new SentenceModel(is);

    SentenceDetectorME sdetector = new SentenceDetectorME(model);

    String[] sentences = sdetector.sentDetect(paragraph);
    assertThat(sentences).contains(
            "This is a statement.",
            "This is another statement.",
            "Now is an abstract word for time, that is always flying.",
            "And my email address is google@gmail.com.");
}
```

注意：Apache OpenNLP中的许多类名都使用后缀“ME”，表示基于“最大熵”的算法。

## 4. 标记化

现在我们可以将文本语料库分成句子，我们就可以开始更详细地分析句子。

**标记化的目标是将句子分成更小的部分**，称为标记(token)。通常，这些标记是单词、数字或标点符号。

OpenNLP中有三种类型的标记器。

### 4.1 使用TokenizerME

在这种情况下，我们首先需要加载模型，可以从[这里](http://opennlp.sourceforge.net/models-1.5/)下载模型文件，将其放在/resources文件夹中，然后从那里加载。

接下来，我们将使用加载的模型创建TokenizerME的实例，并使用tokenize()方法对任何字符串执行标记化：

```java
@Test
public void givenEnglishModel_whenTokenize_thenTokensAreDetected() throws Exception {
    InputStream inputStream = getClass()
            .getResourceAsStream("/models/en-token.bin");
    TokenizerModel model = new TokenizerModel(inputStream);
    TokenizerME tokenizer = new TokenizerME(model);
    String[] tokens = tokenizer.tokenize("Tuyucheng is a Spring Resource.");

    assertThat(tokens).contains("Tuyucheng", "is", "a", "Spring", "Resource", ".");
}
```

我们看到，分词器已将所有单词和句点字符识别为单独的分词，此分词器也可与自定义训练模型一起使用。

### 4.2 WhitespaceTokenizer

顾名思义，这个标记器只是使用空格字符作为分隔符将句子拆分成标记：

```java
@Test
public void givenWhitespaceTokenizer_whenTokenize_thenTokensAreDetected() throws Exception {
    WhitespaceTokenizer tokenizer = WhitespaceTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("Tuyucheng is a Spring Resource.");

    assertThat(tokens).contains("Tuyucheng", "is", "a", "Spring", "Resource.");
}
```

我们可以看到，句子已被空格分隔，因此我们得到“Resource.”(末尾带有句点字符)作为单个标记，而不是单词“Resource”和句点字符的两个不同标记。

### 4.3 SimpleTokenizer

这个分词器比WhitespaceTokenizer稍微复杂一点，它会将句子拆分成单词、数字和标点符号。这是默认行为，不需要任何模型：

```java
@Test
public void givenSimpleTokenizer_whenTokenize_thenTokensAreDetected() throws Exception {
    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer
            .tokenize("Tuyucheng is a Spring Resource.");

    assertThat(tokens)
            .contains("Tuyucheng", "is", "a", "Spring", "Resource", ".");
}
```

## 5. 命名实体识别

现在我们已经了解了标记化，让我们看一下基于成功标记化的第一个用例：命名实体识别(NER)。

**NER的目标是在给定文本中查找命名实体，例如人物、地点、组织和其他命名事物**。

OpenNLP使用预定义的模型来处理人名、日期和时间、地点以及组织。我们需要使用TokenNameFinderModel加载该模型，并将其传递给NameFinderME的实例。然后，我们就可以使用find()方法在给定文本中查找命名实体了：

```java
@Test
public void
givenEnglishPersonModel_whenNER_thenPersonsAreDetected() throws Exception {
    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer
            .tokenize("John is 26 years old. His best friend's "
                    + "name is Leonard. He has a sister named Penny.");

    InputStream inputStreamNameFinder = getClass()
            .getResourceAsStream("/models/en-ner-person.bin");
    TokenNameFinderModel model = new TokenNameFinderModel(
            inputStreamNameFinder);
    NameFinderME nameFinderME = new NameFinderME(model);
    List<Span> spans = Arrays.asList(nameFinderME.find(tokens));

    assertThat(spans.toString())
            .isEqualTo("[[0..1) person, [13..14) person, [20..21) person]");
}
```

正如我们在断言中看到的，结果是一个Span对象列表，其中包含组成文本中命名实体的标记的开始和结束索引。

## 6. 词性标注

另一个需要标记列表作为输入的用例是词性标注。

**词性(POS)标识单词的类型**，OpenNLP使用以下标记来表示不同的词性：

- **NN**：名词，单数或集合
- **DT**：限定词
- **VB**：动词，基本形式
- **VBD**：动词，过去式
- **VBZ**：动词，第三人称单数现在时
- **IN**：介词或从属连词
- **NNP**：专有名词，单数
- **TO**：“to”这个词
- **JJ**：形容词

这些标签与宾州树木库(Penn Tree Bank)中定义的标签相同，完整列表请参阅[此列表](https://www.ling.upenn.edu/courses/Fall_2003/ling001/penn_treebank_pos.html)。

与NER示例类似，我们加载适当的模型，然后在一组标记上使用POSTaggerME及其方法tag()来标记句子：

```java
@Test
public void givenPOSModel_whenPOSTagging_thenPOSAreDetected() throws Exception {
    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("John has a sister named Penny.");

    InputStream inputStreamPOSTagger = getClass()
            .getResourceAsStream("/models/en-pos-maxent.bin");
    POSModel posModel = new POSModel(inputStreamPOSTagger);
    POSTaggerME posTagger = new POSTaggerME(posModel);
    String[] tags = posTagger.tag(tokens);

    assertThat(tags).contains("NNP", "VBZ", "DT", "NN", "VBN", "NNP", ".");
}
```

tag()方法将标记映射到POS标签列表中，示例中的结果如下：

1. “John”：NNP(专有名词)
2. “has”：VBZ(动词)
3. “a”：DT(限定词)
4. “sister”：NN(名词)
5. “named”：VBZ(动词)
6. “Penny”：NNP(专有名词)
7. “.”：时期

## 7. 词形还原

现在我们有了句子中标记的词性信息，我们可以进一步分析文本。

词形还原是将具有时态、性别、语气或其他信息的词形映射到该词的基本形式(也称为“词干”)的过程。

**词形还原器以一个词条及其词性标记作为输入**，并返回该词的词干。因此，在进行词形还原之前，句子应该先经过一个词条还原器和词性标注器。

Apache OpenNLP提供两种类型的词形还原：

- **统计**：需要使用训练数据构建一个词形还原模型来查找给定单词的词干
- **基于词典**：需要一个包含单词、POS标签和相应词干的所有有效组合的词典

对于统计词形还原，我们需要训练一个模型，而对于词典词形还原，我们只需要一个像[这样](https://raw.githubusercontent.com/richardwilly98/elasticsearch-opennlp-auto-tagging/master/src/main/resources/models/en-lemmatizer.dict)的词典文件。

让我们看一个使用字典文件的代码示例：

```java
@Test
public void givenEnglishDictionary_whenLemmatize_thenLemmasAreDetected() throws Exception {
    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("John has a sister named Penny.");

    InputStream inputStreamPOSTagger = getClass()
            .getResourceAsStream("/models/en-pos-maxent.bin");
    POSModel posModel = new POSModel(inputStreamPOSTagger);
    POSTaggerME posTagger = new POSTaggerME(posModel);
    String[] tags = posTagger.tag(tokens);
    InputStream dictLemmatizer = getClass()
            .getResourceAsStream("/models/en-lemmatizer.dict");
    DictionaryLemmatizer lemmatizer = new DictionaryLemmatizer(
            dictLemmatizer);
    String[] lemmas = lemmatizer.lemmatize(tokens, tags);

    assertThat(lemmas)
            .contains("O", "have", "a", "sister", "name", "O", "O");
}
```

可以看到，我们得到了每个token的词干。“O”表示由于该词是专有名词，因此无法确定词干。因此，我们没有得到“John”和“Penny”的词干。

但我们已经确定了句子中其他单词的词干：

- has：have
- a：a
- sister：sister
- named：name

## 8. 分块

词性信息在分块过程中也至关重要-将句子划分为具有语法意义的词组，例如名词组或动词组。

与之前类似，我们在调用chunk()方法之前对句子进行标记并对标记进行词性标注：

```java
@Test
public void givenChunkerModel_whenChunk_thenChunksAreDetected() throws Exception {

    SimpleTokenizer tokenizer = SimpleTokenizer.INSTANCE;
    String[] tokens = tokenizer.tokenize("He reckons the current account deficit will narrow to only 8 billion.");

    InputStream inputStreamPOSTagger = getClass()
            .getResourceAsStream("/models/en-pos-maxent.bin");
    POSModel posModel = new POSModel(inputStreamPOSTagger);
    POSTaggerME posTagger = new POSTaggerME(posModel);
    String[] tags = posTagger.tag(tokens);

    InputStream inputStreamChunker = getClass()
            .getResourceAsStream("/models/en-chunker.bin");
    ChunkerModel chunkerModel
            = new ChunkerModel(inputStreamChunker);
    ChunkerME chunker = new ChunkerME(chunkerModel);
    String[] chunks = chunker.chunk(tokens, tags);
    assertThat(chunks).contains(
            "B-NP", "B-VP", "B-NP", "I-NP",
            "I-NP", "I-NP", "B-VP", "I-VP",
            "B-PP", "B-NP", "I-NP", "I-NP", "O");
}
```

如我们所见，每个token都会从词块划分器中得到一个输出。“B”表示词块的开始，“I”表示词块的延续，“O”表示没有词块。

解析示例的输出，我们得到6个块：

1. “He”：名词短语
2. “reckons”：动词短语
3. “the current account deficit”：名词短语
4. “will narrow”：动词短语
5. “to”：介词短语
6. “only 8 billion”：名词短语

## 9. 语言检测

除了已经讨论过的用例之外，**OpenNLP还提供了语言检测API，可以识别特定文本的语言**。 

为了进行语言检测，我们需要一个训练数据文件，该文件包含每行特定语言的句子，每行都标记了正确的语言，以便为机器学习算法提供输入。

可以在[此处](https://github.com/apache/opennlp/blob/master/opennlp-tools/src/test/resources/opennlp/tools/doccat/DoccatSample.txt)下载语言检测的示例训练数据文件。

我们可以将训练数据文件加载到LanguageDetectorSampleStream中，定义一些训练数据参数，创建模型，然后使用该模型检测文本的语言：

```java
@Test
public void givenLanguageDictionary_whenLanguageDetect_thenLanguageIsDetected() throws FileNotFoundException, IOException {
    InputStreamFactory dataIn
            = new MarkableFileInputStreamFactory(new File("src/main/resources/models/DoccatSample.txt"));
    ObjectStream lineStream = new PlainTextByLineStream(dataIn, "UTF-8");
    LanguageDetectorSampleStream sampleStream
            = new LanguageDetectorSampleStream(lineStream);
    TrainingParameters params = new TrainingParameters();
    params.put(TrainingParameters.ITERATIONS_PARAM, 100);
    params.put(TrainingParameters.CUTOFF_PARAM, 5);
    params.put("DataIndexer", "TwoPass");
    params.put(TrainingParameters.ALGORITHM_PARAM, "NAIVEBAYES");

    LanguageDetectorModel model = LanguageDetectorME
            .train(sampleStream, params, new LanguageDetectorFactory());

    LanguageDetector ld = new LanguageDetectorME(model);
    Language[] languages = ld
            .predictLanguages("estava em uma marcenaria na Rua Bruno");
    assertThat(Arrays.asList(languages))
            .extracting("lang", "confidence")
            .contains(
                    tuple("pob", 0.9999999950605625),
                    tuple("ita", 4.939427661577956E-9),
                    tuple("spa", 9.665954064665144E-15),
                    tuple("fra", 8.250349924885834E-25));
}
```

结果是最可能的语言列表以及置信度分数。

而且，借助丰富的模型，我们可以通过这种类型的检测实现非常高的准确度。

## 10. 总结

我们在这里探索了很多OpenNLP的有趣功能。我们重点介绍了一些用于执行NLP任务的有趣特性，例如词形还原、词性标注、分词、句子检测、语言检测等等。