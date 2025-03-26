---
layout: post
title:  Java中的NLP库概述
category: libraries
copyright: libraries
excerpt: NLP
---

## 1. 概述

**自然语言处理(NLP)是[人工智能(AI)](https://www.baeldung.com/java-ai)的一个分支，它使计算机能够像人类一样理解书面或口头语言**。在人工智能革命时代，它具有多种应用。

在本教程中，我们将探索Java中的不同NLP库，并了解如何使用Apache OpenNLP和Stanford CoreNLP实现一些NLP任务。

## 2. 什么是NLP？

NLP使计算机能够以类似于人类的方式处理文本和单词，它将计算语言学与[统计学](https://www.baeldung.com/cs/ml-statistics-significance/)、[深度学习](https://www.baeldung.com/cs/machine-learning-vs-deep-learning)和[机器学习](https://www.baeldung.com/cs/machine-learning-how-to-start)结合在一起。

人类每天都通过各种媒体在线互动，在此过程中，他们分享不同类型的数据，例如文本、语音、图像等。这些数据对于理解人类的行为和习惯至关重要，因此，它们被用来训练计算机模仿人类智能。

**NLP使用数据来训练机器模仿人类的语言行为**。为此，它遵循由以下几个步骤组成的过程：

1. 它将文本分成更小的单位，例如句子或单词。
2. 它对文本进行标记，这意味着它为每个单词分配一个唯一的标识符。
3. 它会删除停用词，即对文本没有太多意义的常用词，例如“the”、“a”、“and”等。
4. 它对文本进行词干提取或词形还原，这意味着它将每个单词简化为其词根形式或词典形式。
5. 它用词性来标记每个单词。
6. 它用命名实体标记每个单词，例如人、地点、组织等。

## 3. NLP的用例

NLP是众多现代现实世界应用中机器智能背后的驱动力。

**机器翻译就是一个示例用例**，我们有可以将一种特定语言翻译成另一种语言的系统，其中一个例子就是谷歌翻译。推动机器翻译的技术基于NLP算法。

此外，**另一个常见的用例是垃圾邮件检测**。大多数流行的电子邮件服务提供商使用垃圾邮件检测器来确定收到的邮件是否是垃圾邮件，垃圾邮件检测应用NLP文本分类技术根据语言模式识别垃圾邮件。

此外，**人工智能聊天机器人现在非常普遍**，流行的例子包括Siri、Google Assistant、Alexa等。这些应用程序使用语音识别和自然语言来识别语音中的模式并以适当、有用的评论做出回应。

**NLP是这些应用程序的核心逻辑，因为它使它们能够处理自然语言输入和输出(例如文本和语音)，并理解它们背后的含义和意图**。

## 4. OpenNLP

[Apache OpenNLP](https://opennlp.apache.org/)是一个利用机器学习处理自然语言文本的工具包，它为标记化、分段、语音标记等常见NLP任务提供支持。

**Apache OpenNLP的主要目标是提供对NLP任务的支持，并为多种语言提供大量预构建的模型**。此外，它还提供了命令行接口(CLI)，方便进行实验和训练。

Apache OpenNLP提供各种预构建模型供[下载](https://opennlp.apache.org/models.html)，让我们使用预构建模型实现一个简单的语言检测器。首先，让我们将OpenNLP[依赖](https://mvnrepository.com/artifact/org.apache.opennlp/opennlp-tools)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.opennlp</groupId>
    <artifactId>opennlp-tools</artifactId>
    <version>2.1.1</version>
</dependency>
```

接下来我们使用[langdetect-183.bin](https://www.apache.org/dyn/closer.cgi/opennlp/models/langdetect/1.8.3/langdetect-183.bin)预构建模型来实现语言检测器：

```java
@Test
void givenTextInEnglish_whenDetectLanguage_thenReturnsEnglishLanguageCode() {
    String text = "the dream my father told me";
    LanguageDetectorModel model;

    try (InputStream modelIn = new FileInputStream("langdetect-183.bin")) {
        model = new LanguageDetectorModel(modelIn);
    } catch (IOException e) {
        return;
    }

    LanguageDetectorME detector = new LanguageDetectorME(model);
    Language language = detector.predictLanguage(text);
    assertEquals("eng", language.getLang());
}
```

在上面的示例中，我们从OpenNLP获取预构建的语言检测模型并将其放在根目录中。然后，我们定义输入数据。接下来，我们加载语言检测器模型。最后，我们创建一个新的LanguageDetectorME实例并尝试检测语言，我们使用返回语言测试预期语言。

## 5. Stanford NLP

[Stanford NLP](https://nlp.stanford.edu/)小组协助开发允许机器处理、生成和理解人类文本和语言的算法。

**CoreNLP是斯坦福NLP小组用Java编写的一套程序，可以执行各种NLP任务，如标记化、词性标记、词形还原等。它可以通过命令行、Java代码或调用服务器来使用**。

让我们看一个使用Stanford CoreNLP执行标记化的示例，我们需要将其[依赖](https://mvnrepository.com/artifact/edu.stanford.nlp/stanford-corenlp)添加到pom.xml中：

```xml
<dependency>
    <groupId>edu.stanford.nlp</groupId>
    <artifactId>stanford-corenlp</artifactId>
    <version>4.5.3</version>
</dependency>
```

接下来，让我们进行标记化：

```java
@Test
void givenSampleText_whenTokenize_thenExpectedTokensReturned() {
    Properties props = new Properties();
    props.setProperty("annotators", "tokenize");
    StanfordCoreNLP pipeline = new StanfordCoreNLP(props);

    String text = "The german shepard display an act of kindness";
    Annotation document = new Annotation(text);
    pipeline.annotate(document);

    List<CoreMap> sentences = document.get(CoreAnnotations.SentencesAnnotation.class);
    StringBuilder tokens = new StringBuilder();

    for (CoreMap sentence : sentences) {
        for (CoreLabel token : sentence.get(CoreAnnotations.TokensAnnotation.class)) {
            String word = token.get(CoreAnnotations.TextAnnotation.class);
            tokens.append(word).append(" ");
        }
    }
    assertEquals("The german shepard display an act of kindness", tokens.toString().trim());
}
```

在上面的例子中，我们设置了带有标记化注释器的StanfordCoreNLP对象。接下来，我们创建一个新的Annotation实例。最后，我们实现从示例句子生成标记的逻辑。

## 6. CogComp NLP

**[CogComp NLP](http://cogcomp.github.io/cogcomp-nlp/pipeline/)是由认知计算小组开发的自然语言处理(NLP)库的集合**，它为NLP任务提供了各种工具和模块，例如标记化、词形还原、词性标注等。

CogComp NLP既可以用作命令行工具，也可以用作Java API。CogComp NLP中最受欢迎的模块之一是cogcomp-nlp-pipeline，它对给定的文本执行基本的NLP任务。但是，cogcomp-nlp-pipeline仅适用于纯英文文本。

另一个模块是similarity，它测量文本或其他对象之间的相似度并返回分数。

## 7. GATE

[General Architecture for Text Engineering(GATE)](https://gate.ac.uk/)是一个能够解决文本分析和语言处理问题的工具包，它是开发处理人类语言的软件组件的绝佳基础设施。此外，它还是一个很棒的NLP工具包。

该工具包拥有庞大的开发人员和研究人员社区，他们使用它进行信息提取、情感分析、社交媒体挖掘和生物医学文本处理。

**GATE通过提供语言处理软件的架构来帮助开发人员和研究人员。此外，它还提供了实现该架构的类库**。

## 8. Apache UIMA

[Unstructured Information Management Applications(UIMA)](https://uima.apache.org/)是可以处理和分析大量非结构化数据(包括文本、音频和视频)的软件系统。**它们有助于创建可以从内容中检测情绪、实体和其他类型信息的组件**，这些组件是用Java或C++编写的。

此外，Apache UIMA是一个框架，它使我们能够使用UIMA组件构建应用程序并处理大量非结构化数据，它帮助我们从数据中提取相关信息并将其用于各种目的。

## 9. MALLET

[MAchine Learning for LangaugE Toolkit(MALLET)](https://mimno.github.io/Mallet/)是一个Java包，它为NLP任务提供各种工具和算法，例如文档分类、主题建模和序列标记。MALLET中包含的算法之一是朴素贝叶斯算法，该算法在NLP中广泛用于文本分类和情感分析。

MALLET是一个开源Java软件包，提供各种文本分析工具。其中一个工具是主题建模，它可以在大量未标记的文本文档中发现主要主题。

**此外，MALLET还可以将文本文档转换为可用于机器学习的数值向量**。此外，它既可以用作命令行工具，也可以用作直接Java API。

## 10. 总结

在本文中，我们了解了有关NLP的关键内容以及NLP的用例。此外，我们还了解了不同的Java NLP库和工具包。此外，我们还分别查看了使用CoreNLP和OpenNLP进行标记化和句子检测的示例。