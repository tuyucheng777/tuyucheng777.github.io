---
layout: post
title:  LangChain简介
category: libraries
copyright: libraries
excerpt: LLM
---

## 1. 简介

在本教程中，我们将研究[LangChain](https://www.langchain.com/)的细节，这是一个由[语言模型](https://www.baeldung.com/cs/nlp-language-models)驱动的应用程序开发框架。我们将首先收集有关语言模型的基本概念，这将有助于本教程。

尽管LangChain主要提供Python和JavaScript/TypeScript版本，但也可以选择在Java中使用LangChain。我们将讨论LangChain作为框架的构建块，然后继续在Java中尝试它们。

## 2. 背景

在深入探讨为什么我们需要一个由语言模型驱动的应用程序构建框架之前，我们必须首先了解什么是语言模型。我们还将介绍使用语言模型时遇到的一些典型复杂问题。

### 2.1 大语言模型

**语言模型是自然语言的概率模型**，可以生成一系列单词的概率。[大语言模型](https://www.baeldung.com/cs/large-language-models)(LLM)是一种以规模大为特征的语言模型，它们是人工神经网络，可能具有数十亿个参数。

**LLM通常使用[自监督](https://en.wikipedia.org/wiki/Self-supervised_learning)和[半监督学习](https://en.wikipedia.org/wiki/Weak_supervision)技术对大量未标记数据进行预训练**。然后，使用微调和快速工程等各种技术将预训练模型调整为特定任务：

![](/assets/images/2025/libraries/javalangchainbasics01.png)

这些LLM能够执行多种自然语言处理任务，如语言翻译和内容摘要，**它们还能够执行内容创建等生成任务**。因此，它们在回答问题等应用中非常有价值。

**几乎所有主要的云服务提供商都已将大型语言模型纳入其服务产品中**。例如，[Microsoft Azure](https://azure.microsoft.com/en-us/solutions/ai)提供Llama 2和OpenAI GPT-4等LLM。Amazon [Bedrock](https://aws.amazon.com/bedrock/)提供AI21 Labs、Anthropic、Cohere、Meta和Stability AI的模型。

### 2.2 提示工程

LLM是基于大量文本数据进行训练的基础模型，因此，它们可以捕捉人类语言固有的语法和语义。但是，它们 **必须经过调整才能执行我们希望它们执行的特定任务**。

[提示工程](https://en.wikipedia.org/wiki/Prompt_engineering)是适应LLM的最快方法之一。这是一个构建文本的过程，可供LLM解释和理解。在这里，我们使用自然语言文本来描述我们期望LLM执行的任务：

![](/assets/images/2025/libraries/javalangchainbasics02.png)

我们创建的提示可**帮助LLM进行[情境学习](https://en.wikipedia.org/wiki/Prompt_engineering#In-context_learning)**，这是暂时的。我们可以使用提示工程来促进LLM的安全使用，并构建新功能，例如使用领域知识和外部工具增强LLM。

这是一个活跃的研究领域，新技术不断涌现。然而，**像[思路提示](https://www.promptingguide.ai/techniques/cot)这样的技术已经变得相当流行**，这里的理念是让LLM在给出最终答案之前，通过一系列中间步骤来解决问题。

### 2.3 词嵌入

正如我们所见，LLM能够处理大量自然语言文本。如果我们将自然语言中的单词表示为[词向量](https://en.wikipedia.org/wiki/Word_embedding)，LLM的性能将大大提高。这是一个能够编码单词含义的实值向量。

通常，词向量是使用[Tomáš Mikolov的Word2vec](https://en.wikipedia.org/wiki/Word2vec)或[斯坦福大学的GloVe](https://nlp.stanford.edu/projects/glove/)等算法生成的。GloVe是一种无监督学习算法，它基于语料库中汇总的全局单词共现统计数据进行训练：

![](/assets/images/2025/libraries/javalangchainbasics03.png)

在提示工程中，我们将提示转换为其词向量，使模型能够更好地理解和响应提示。此外，它还非常有助于增强我们为模型提供的上下文，从而使模型能够提供更多上下文答案。

例如，我们可以从现有数据集生成词向量并将其存储在向量数据库中。此外，我们可以使用用户提供的输入对该向量数据库进行语义搜索。然后，我们可以将搜索结果用作模型的附加上下文。

## 3. LangChain的LLM技术栈

正如我们已经看到的，**创建有效的提示**是成功利用任何应用程序中的LLM功能的关键要素。这包括使与语言模型的交互具有上下文感知能力，并能够依赖语言模型进行推理。

为此，我们需要执行多项任务，**例如创建提示模板、调用语言模型以及向语言模型提供来自多个来源的用户特定数据**。为了简化这些任务，我们需要一个像LangChain这样的框架作为我们LLM技术栈的一部分：

![](/assets/images/2025/libraries/javalangchainbasics04.png)

该框架还有助于开发需要链接多个语言模型并能够回忆过去与语言模型交互的信息的应用程序。然后，还有更复杂的用例，涉及使用语言模型作为推理引擎。

最后，我们可以执行日志记录、监控、流式传输和其他维护和故障排除的基本任务。LLM技术栈正在快速发展，以解决其中许多问题。然而，LangChain正迅速成为LLM技术栈中一个有价值的组成部分。

## 4. Java版LangChain

**[LangChain](https://www.langchain.com/)于2022年作为开源项目推出**，并很快通过社区支持获得了发展势头。它最初由Harrison Chase用Python开发，很快就成为AI领域增长最快的初创公司之一。

**2023年初，在Python版本之后，出现了一个JavaScript/TypeScript版本的LangChain**。它很快就变得非常流行，并开始支持多种JavaScript环境，如Node.js、Web浏览器、CloudFlare工作器、Vercel/Next.js、Deno和Supabase Edge函数。

不幸的是，没有适用于Java/Spring应用程序的官方Java版LangChain。不过，有一个Java版的LangChain社区版本，名为[LangChain4j](https://github.com/langchain4j/langchain4j)。它适用于Java 8或更高版本，并支持Spring Boot 2和3。

LangChain的各种依赖可在[Maven Central](https://mvnrepository.com/artifact/dev.langchain4j/langchain4j/0.23.0)中找到。根据我们使用的功能，我们可能需要在应用程序中添加一个或多个依赖：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>0.23.0</version>
</dependency>
```

例如，在本教程的后续部分中，我们还需要支持[与OpenAI模型集成](https://mvnrepository.com/artifact/dev.langchain4j/langchain4j-open-ai/0.23.0)的依赖、提供[对嵌入的支持](https://mvnrepository.com/artifact/dev.langchain4j/langchain4j-embeddings/0.23.0)以及[类似all-MiniLM-L6-v2的句子转换器模型](https://mvnrepository.com/artifact/dev.langchain4j/langchain4j-embeddings-all-minilm-l6-v2-q/0.23.0)。

LangChain4j的设计目标与LangChain类似，它提供了一个简单而连贯的抽象层以及众多实现。它已经支持OpenAI等多家语言模型提供商和Pinecone等嵌入存储提供商。

但是，由于LangChain和LangChain4j都在快速发展，Python或JS/TS版本中可能支持Java版本中尚不支持的功能。不过，基本概念、总体结构和词汇大致相同。

## 5. LangChain的构建模块

LangChain为我们的应用程序提供了多种价值主张，可用作模块组件，模块化组件提供有用的抽象以及用于处理语言模型的实现集合。让我们用Java中的示例来讨论其中一些模块。

### 5.1 模型I/O

在使用任何语言模型时，我们都需要与之交互的能力。LangChain提供了必要的构建块，例如模板化提示以及动态选择和管理模型输入的能力。此外，我们可以使用输出解析器从模型输出中提取信息：

![](/assets/images/2025/libraries/javalangchainbasics05.png)

提示模板是用于生成语言模型提示的预定义方案，可能包括说明、[少量示例](https://www.promptingguide.ai/techniques/fewshot)和特定上下文：

```java
PromptTemplate promptTemplate = PromptTemplate
    .from("Tell me a {{adjective}} joke about {{content}}..");
Map<String, Object> variables = new HashMap<>();
variables.put("adjective", "funny");
variables.put("content", "computers");
Prompt prompt = promptTemplate.apply(variables);
```

这里，我们创建一个能够接受多个变量的提示模板。变量是我们从用户输入中接收并提供给提示模板的内容。

LangChain支持集成两种类型的模型，即语言模型和聊天模型。聊天模型也由语言模型支持，但提供聊天功能：

```java
ChatLanguageModel model = OpenAiChatModel.builder()
    .apiKey(<OPENAI_API_KEY>)
    .modelName(GPT_3_5_TURBO)
    .temperature(0.3)
    .build();
String response = model.generate(prompt.text());
```

这里我们用特定的OpenAI模型和相关的API Key创建一个聊天模型，我们可以通过免费注册从[OpenAI](https://openai.com/)获取API Key，参数temperature用于控制模型输出的随机性。

最后，语言模型的输出可能不够结构化，无法呈现。LangChain提供输出解析器，帮助我们构建语言模型响应-例如，从输出中提取信息作为Java中的POJO。

### 5.2 记忆

通常，利用LLM的应用程序具有对话界面。**任何对话的一个重要方面是能够引用对话中先前介绍的信息**，存储有关过去交互的信息的能力称为记忆：

![](/assets/images/2025/libraries/javalangchainbasics06.png)

LangChain提供了向应用程序添加内存的关键推动因素。例如，我们需要能够从内存中读取以增强用户输入。然后，我们需要能够将当前运行的输入和输出写入内存：

```java
ChatMemory chatMemory = TokenWindowChatMemory
    .withMaxTokens(300, new OpenAiTokenizer(GPT_3_5_TURBO));
chatMemory.add(userMessage("Hello, my name is Kumar"));
AiMessage answer = model.generate(chatMemory.messages()).content();
System.out.println(answer.text()); // Hello Kumar! How can I assist you today?
chatMemory.add(answer);
chatMemory.add(userMessage("What is my name?"));
AiMessage answerWithName = model.generate(chatMemory.messages()).content();
System.out.println(answer.text()); // Your name is Kumar.
chatMemory.add(answerWithName);
```

在这里，我们使用TokenWindowChatMemory实现了固定窗口聊天内存，它允许我们读取和写入与语言模型交换的聊天消息。

**LangChain还提供更复杂的数据结构和算法**，以便从内存中返回选定的消息，而不是返回所有内容。例如，它支持返回过去几条消息的摘要，或者仅返回与当前运行相关的消息。

### 5.3 检索

大型语言模型通常是在大量文本语料库上进行训练的。因此，它们在一般任务中非常高效，但在特定领域的任务中可能不太有用。为此，我们需要检索相关的外部数据并在生成步骤中将其传递给语言模型。

此过程称为[检索增强生成(RAG)](https://www.promptingguide.ai/techniques/rag)，它有助于将模型建立在相关且准确的信息上，并让我们深入了解模型的生成过程。LangChain提供了创建RAG应用程序所需的构建块：

![](/assets/images/2025/libraries/javalangchainbasics07.png)

首先，LangChain提供了文档加载器，用于从存储位置检索文档。然后，有转换器可用于准备文档以供进一步处理。例如，我们可以让它将大文档拆分成较小的块：

```java
Document document = FileSystemDocumentLoader.loadDocument("simpson's_adventures.txt");
DocumentSplitter splitter = DocumentSplitters.recursive(100, 0, new OpenAiTokenizer(GPT_3_5_TURBO));
List<TextSegment> segments = splitter.split(document);
```

这里，我们使用FileSystemDocumentLoader从文件系统加载文档。然后，我们使用OpenAiTokenizer将该文档拆分成更小的块。

为了提高检索效率，文档通常会被转换成其嵌入并存储在向量数据库中。LangChain支持多种嵌入提供程序和方法，并与几乎所有流行的向量存储集成：

```java
EmbeddingModel embeddingModel = new AllMiniLmL6V2EmbeddingModel();
List<Embedding> embeddings = embeddingModel.embedAll(segments).content();
EmbeddingStore<TextSegment> embeddingStore = new InMemoryEmbeddingStore<>();
embeddingStore.addAll(embeddings, segments);
```

在这里，我们使用AllMiniLmL6V2EmbeddingModel创建文档段的嵌入。然后，我们将嵌入存储在内存向量存储中。

现在，我们已将外部数据作为嵌入存储在向量存储中，我们已准备好从中检索数据。LangChain支持多种检索算法，例如简单的语义搜索和复杂的算法，例如集成检索器：

```java
String question = "Who is Simpson?";
//The assumption here is that the answer to this question is contained in the document we processed earlier.
Embedding questionEmbedding = embeddingModel.embed(question).content();
int maxResults = 3;
double minScore = 0.7;
List<EmbeddingMatch<TextSegment>> relevantEmbeddings = embeddingStore
    .findRelevant(questionEmbedding, maxResults, minScore);
```

我们创建用户问题的嵌入，然后使用问题嵌入从向量存储中检索相关匹配项。现在，我们可以将检索到的相关匹配项作为上下文发送，方法是将它们添加到我们打算发送给模型的提示中。

## 6. LangChain的复杂应用

到目前为止，我们已经了解了如何使用单个组件来创建具有语言模型的应用程序。LangChain还提供组件来构建更复杂的应用程序，例如，我们可以使用链和代理来构建具有增强功能的更具自适应性的应用程序。

### 6.1 链

通常，**一个应用程序需要按特定顺序调用多个组件**。这就是LangChain中所谓的链。它简化了更复杂应用程序的开发，并使其更易于调试、维护和改进。

这对于组合多个链以形成更复杂的应用程序也很有用，这些应用程序可能需要与多个语言模型的接口。LangChain提供了创建此类链的便捷方法，并提供了许多预建链：

```java
ConversationalRetrievalChain chain = ConversationalRetrievalChain.builder()
    .chatLanguageModel(chatModel)
    .retriever(EmbeddingStoreRetriever.from(embeddingStore, embeddingModel))
    .chatMemory(MessageWindowChatMemory.withMaxMessages(10))
    .promptTemplate(PromptTemplate
        .from("Answer the following question to the best of your ability: {{question}}\n\nBase your answer on the following information:\n{{information}}"))
    .build();
```

这里我们使用了预先构建的链ConversationalRetrievalChain，它允许我们将聊天模型与检索器以及内存和提示模板一起使用。现在，我们可以简单地使用该链来执行用户查询：

```java
String answer = chain.execute("Who is Simpson?");
```

该链带有默认的内存和提示模板，我们可以覆盖这些模板。创建自定义链也相当容易，创建链的能力使得实现复杂应用程序的模块化实现变得更加容易。

### 6.2 代理

LangChain还提供更强大的结构，例如代理。与链不同，代理使用语言模型作为推理引擎来确定要采取哪些操作以及按什么顺序进行。我们还可以为代理提供访问正确工具的权限，以执行必要的操作。

在LangChain4j中，代理可作为AI服务使用，以声明方式定义复杂的AI行为。让我们看看是否可以提供计算器作为AI服务的工具，并启用语言模型来执行计算。

首先，我们将定义一个包含一些基本计算器功能的类，并用自然语言描述每个功能，以便模型理解：

```java
public class AIServiceWithCalculator {
    static class Calculator {
        @Tool("Calculates the length of a string")
        int stringLength(String s) {
            return s.length();
        }

        @Tool("Calculates the sum of two numbers")
        int add(int a, int b) {
            return a + b;
        }
    }
}
```

然后，我们将定义AI服务构建的接口。这里非常简单，但它也可以描述更复杂的行为：

```java
interface Assistant {
    String chat(String userMessage);
}
```

现在，我们将使用刚刚定义的接口和创建的工具从LangChain4j提供的构建器工厂构建一个AI服务：

```java
Assistant assistant = AiServices.builder(Assistant.class)
    .chatLanguageModel(OpenAiChatModel.withApiKey(<OPENAI_API_KEY>))
    .tools(new Calculator())
    .chatMemory(MessageWindowChatMemory.withMaxMessages(10))
    .build();
```

就这样！我们现在可以开始将包含一些要执行的计算的问题发送到我们的语言模型：

```java
String question = "What is the sum of the numbers of letters in the words \"language\" and \"model\"?";
String answer = assistant.chat(question);
System.out.prtintln(answer); // The sum of the numbers of letters in the words "language" and "model" is 13. 
```

当我们运行此代码时，我们会发现语言模型现在能够执行计算。

值得注意的是，语言模型在执行某些需要时间和空间概念或执行复杂算术程序的任务时会遇到困难。但是，我们始终可以为模型补充必要的工具来解决这个问题。

## 7. 总结

在本教程中，我们介绍了创建由大型语言模型驱动的应用程序的一些基本要素。此外，我们还讨论了将LangChain等框架作为开发此类应用程序的技术堆栈的一部分的价值。