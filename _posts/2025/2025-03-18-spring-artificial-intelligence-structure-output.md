---
layout: post
title:  Spring AI中的结构化输出指南
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 简介

通常，在使用大语言模型([LLM](https://www.baeldung.com/cs/large-language-models))时，我们并不期望得到结构化的响应。此外，我们已经习惯了它们不可预测的行为，这通常会导致输出并不总是符合我们的预期。但是，有一些方法可以增加生成结构化响应的可能性(尽管不是100%的概率)，甚至可以将这些响应解析为可用的代码结构。

**在本教程中，我们将探索Spring AI和简化和优化此过程的工具，使其更易于访问和直接**。

## 2. 聊天模型简介

**让我们对AI模型进行提示的基本结构是ChatModel接口**：

```java
public interface ChatModel extends Model<Prompt, ChatResponse> {
    default String call(String message) {
        // implementation is skipped
    }

    @Override
    ChatResponse call(Prompt prompt);
}
```

call()方法的作用是向模型发送消息并接收响应，仅此而已。人们自然会期望提示和响应为String类型，但是，现代模型实现通常具有更复杂的结构，可以进行更精细的调整，从而增强模型的可预测性。例如，虽然可以使用接收String参数的默认call()方法，但使用Prompt更为实用。此Prompt可以包含多条消息或包含温度等选项，以调节模型的明显创造力。

我们可以自动装配ChatModel并直接调用它。例如，如果我们的依赖项中有OpenAI API的spring-ai-openai-spring-boot-starter，则OpenAiChatModel实现将自动装配。

## 3. 结构化输出API

为了以数据结构的形式获取输出，**Spring AI提供了使用结构化输出API包装ChatModel调用的工具**。此API的核心接口是StructuredOutputConverter：

```java
public interface StructuredOutputConverter<T> extends Converter<String, T>, FormatProvider {}
```

**它结合了另外两个接口，第一个是FormatProvider**：

```java
public interface FormatProvider {
    String getFormat();
}
```

在ChatModel的call()之前，getFormat()会准备提示，用所需的数据模式填充它，并具体描述应如何格式化数据以避免响应不一致。例如，要获取JSON格式的响应，它使用以下提示：

```java
public String getFormat() {
    String template = "Your response should be in JSON format.\n"
            + "Do not include any explanations, only provide a RFC8259 compliant JSON response following this format without deviation.\n"
            + "Do not include markdown code blocks in your response.\n
            + "Remove the ```json markdown from the output.\nHere is the JSON Schema instance your output must adhere to:\n```%s```\n";
    return String.format(template, this.jsonSchema);
}
```

这些说明通常附加在用户输入之后。

**第二个接口是Converter**：

```java
@FunctionalInterface
public interface Converter<S, T> {
    @Nullable
    T convert(S source);

    // default method
}
```

call()返回响应后，转换器将其解析为所需的T类型的数据结构，下面是StructuredOutputConverter工作原理的简单示意图：

![](/assets/images/2025/springai/springartificialintelligencestructureoutput01.png)

## 4. 可用的转换器

在本节中，我们将通过示例探索StructuredOutputConverter的可用实现。我们将通过为《龙与地下城》游戏生成角色来演示这一点：

```java
public class Character {
    private String name;
    private int age;
    private String race;
    private String characterClass;
    private String cityOfOrigin;
    private String favoriteWeapon;
    private String bio;
    
    // constructor, getters, and setters
}
```

**请注意，由于Jackson的ObjectMapper在后台使用，因此我们需要为我们的Bean设置空的构造函数**。

## 5. BeanOutputConverter

**BeanOutputConverter从模型的响应中生成指定类的实例**，它构造一个提示来指示模型生成符合RFC8259的JSON。让我们看看如何使用ChatClient API使用它：

```java
@Override
public Character generateCharacterChatClient(String race) {
    return ChatClient.create(chatModel).prompt()
        .user(spec -> spec.text("Generate a D&D character with race {race}")
            .param("race", race))
            .call()
            .entity(Character.class); // <-------- we call ChatModel.call() here, not on the line before
}
```

在此方法中，ChatClient.create(chatModel)实例化ChatClient，prompt()方法使用请求(ChatClientRequest)启动构建器链。在我们的例子中，我们只添加用户的文本。创建请求后，将调用call()方法，返回一个包含ChatModel和ChatClientRequest的新CallResponseSpec。然后，entity()方法根据提供的类型创建一个转换器，完成提示并调用AI模型。

我们可能注意到我们没有直接使用BeanOutputConverter，那是因为我们使用了一个类作为.entity()方法的参数，这意味着BeanOutputConverter将处理提示和转换。

为了获得更多控制，我们可以编写此方法的低级版本。在这里，我们将自己使用ChatModel.call()，这是我们事先自动装配的：

```java
@Override
public Character generateCharacterChatModel(String race) {
    BeanOutputConverter<Character> beanOutputConverter = new BeanOutputConverter<>(Character.class);

    String format = beanOutputConverter.getFormat();

    String template = """
            Generate a D&D character with race {race}
            {format}
            """;

    PromptTemplate promptTemplate = new PromptTemplate(template, Map.of("race", race, "format", format));
    Prompt prompt = new Prompt(promptTemplate.createMessage());
    Generation generation = chatModel.call(prompt).getResult();

    return beanOutputConverter.convert(generation.getOutput().getContent());
}
```

在上面的示例中，我们创建了BeanOutputConverter，提取了模型的格式指南，然后将这些指南添加到自定义提示中。我们使用PromptTemplate生成了最终提示，**PromptTemplate是Spring AI的核心提示模板组件**，它在底层使用StringTemplate引擎。然后，我们调用模型以获取结果Generation。Generation表示模型的响应：我们提取其内容，然后使用转换器将其转换为Java对象。

以下是我们使用转换器从OpenAI获得的真实响应示例：

```json
{
    name: "Thoren Ironbeard",
    age: 150,
    race: "Dwarf",
    characterClass: "Wizard",
    cityOfOrigin: "Sundabar",
    favoriteWeapon: "Magic Staff",
    bio: "Born and raised in the city of Sundabar, he is known for his skills in crafting and magic."
}
```

矮人巫师，真是罕见的景象！

## 6. 用于集合的Mapoutputconverter和Listoutputconverter

**MapOutputConverter和ListOutputConverter分别允许我们创建以Map和List形式构造的响应**，以下是使用MapOutputConverter的高级和低级代码示例：

```java
@Override
public Map<String, Object> generateMapOfCharactersChatClient(int amount) {
    return ChatClient.create(chatModel).prompt()
            .user(u -> u.text("Generate {amount} D&D characters, where key is a character's name")
                    .param("amount", String.valueOf(amount)))
            .call()
            .entity(new ParameterizedTypeReference<Map<String, Object>>() {});
}

@Override
public Map<String, Object> generateMapOfCharactersChatModel(int amount) {
    MapOutputConverter outputConverter = new MapOutputConverter();
    String format = outputConverter.getFormat();
    String template = """
            "Generate {amount} of key-value pairs, where key is a "Dungeons and Dragons" character name and value (String) is his bio.
            {format}
            """;
    Prompt prompt = new Prompt(new PromptTemplate(template, Map.of("amount", String.valueOf(amount), "format", format)).createMessage());
    Generation generation = chatModel.call(prompt).getResult();

    return outputConverter.convert(generation.getOutput().getContent());
}
```

我们在Map<String, Object\>中使用Object的原因是因为目前MapOutputConverter不支持泛型值，不过不用担心，稍后我们将构建自定义转换器来支持该功能。现在，让我们查看ListOutputConverter的示例，我们可以自由使用泛型：

```java
@Override
public List<String> generateListOfCharacterNamesChatClient(int amount) {
    return ChatClient.create(chatModel).prompt()
            .user(u -> u.text("List {amount} D&D character names")
                    .param("amount", String.valueOf(amount)))
            .call()
            .entity(new ListOutputConverter(new DefaultConversionService()));
}

@Override
public List<String> generateListOfCharacterNamesChatModel(int amount) {
    ListOutputConverter listOutputConverter = new ListOutputConverter(new DefaultConversionService());
    String format = listOutputConverter.getFormat();
    String userInputTemplate = """
            List {amount} D&D character names
            {format}
            """;
    PromptTemplate promptTemplate = new PromptTemplate(userInputTemplate,
            Map.of("amount", amount, "format", format));
    Prompt prompt = new Prompt(promptTemplate.createMessage());
    Generation generation = chatModel.call(prompt).getResult();
    return listOutputConverter.convert(generation.getOutput().getContent());
}
```

## 7. 转换器剖析或如何构建我们自己的转换器

让我们创建一个转换器，将AI模型中的数据转换为Map<String, V\>格式，其中V是泛型类型。与Spring提供的转换器一样，我们的容器将实现StructuredOutputConverter<T\>，这将要求我们添加方法convert()和getFormat()：

```java
public class GenericMapOutputConverter<V> implements StructuredOutputConverter<Map<String, V>> {
    private final ObjectMapper objectMapper; // to convert response
    private final String jsonSchema; // schema for the instructions in getFormat()
    private final TypeReference<Map<String, V>> typeRef; // type reference for object mapper

    public GenericMapOutputConverter(Class<V> valueType) {
        this.objectMapper = this.getObjectMapper();
        this.typeRef = new TypeReference<>() {};
        this.jsonSchema = generateJsonSchemaForValueType(valueType);
    }

    public Map<String, V> convert(@NonNull String text) {
        try {
            text = trimMarkdown(text);
            return objectMapper.readValue(text, typeRef);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Failed to convert JSON to Map<String, V>", e);
        }
    }

    public String getFormat() {
        String raw = "Your response should be in JSON format.\nThe data structure for the JSON should match this Java class: %s\n" +
                "For the map values, here is the JSON Schema instance your output must adhere to:\n```%s```\n" +
                "Do not include any explanations, only provide a RFC8259 compliant JSON response following this format without deviation.\n";
        return String.format(raw, HashMap.class.getName(), this.jsonSchema);
    }

    private ObjectMapper getObjectMapper() {
        return JsonMapper.builder()
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                .build();
    }

    private String trimMarkdown(String text) {
        if (text.startsWith("```json") && text.endsWith("```")) {
            text = text.substring(7, text.length() - 3);
        }
        return text;
    }

    private String generateJsonSchemaForValueType(Class<V> valueType) {
        try {
            JacksonModule jacksonModule = new JacksonModule();
            SchemaGeneratorConfig config = new SchemaGeneratorConfigBuilder(SchemaVersion.DRAFT_2020_12, OptionPreset.PLAIN_JSON)
                    .with(jacksonModule)
                    .build();
            SchemaGenerator generator = new SchemaGenerator(config);

            JsonNode jsonNode = generator.generateSchema(valueType);
            ObjectWriter objectWriter = new ObjectMapper().writer(new DefaultPrettyPrinter()
                    .withObjectIndenter(new DefaultIndenter().withLinefeed(System.lineSeparator())));

            return objectWriter.writeValueAsString(jsonNode);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Could not generate JSON schema for value type: " + valueType.getName(), e);
        }
    }
}
```

我们知道，getFormat()为AI模型提供了一条指令，它将遵循用户在对AI模型的最终请求中的提示。该指令指定了一个Map结构，并为值提供了我们自定义对象的模式。我们使用com.github.victools.jsonschema库生成了一个模式，Spring AI已在其转换器内部使用此库，这意味着我们不需要明确导入它。

由于我们请求JSON格式的响应，因此在convert()中，我们使用Jackson的ObjectMapper进行解析。因此，我们像Spring的BeanOutputConverter实现一样修剪Markdown。AI模型通常使用Markdown来包装代码片段，通过删除它，我们可以避免ObjectMapper出现异常。

之后，我们可以像这样使用我们的实现：

```java
@Override
public Map<String, Character> generateMapOfCharactersCustomConverter(int amount) {
    GenericMapOutputConverter<Character> outputConverter = new GenericMapOutputConverter<>(Character.class);
    String format = outputConverter.getFormat();
    String template = """
            "Generate {amount} of key-value pairs, where key is a "Dungeons and Dragons" character name and value is character object.
            {format}
            """;
    Prompt prompt = new Prompt(new PromptTemplate(template, Map.of("amount", String.valueOf(amount), "format", format)).createMessage());
    Generation generation = chatModel.call(prompt).getResult();

    return outputConverter.convert(generation.getOutput().getContent());
}

@Override
public Map<String, Character> generateMapOfCharactersCustomConverterChatClient(int amount) {
    return ChatClient.create(chatModel).prompt()
            .user(u -> u.text("Generate {amount} D&D characters, where key is a character's name")
                    .param("amount", String.valueOf(amount)))
            .call()
            .entity(new GenericMapOutputConverter<>(Character.class));
}
```

## 8. 总结

在本文中，我们探讨了如何使用大型语言模型(LLM)来生成结构化响应。通过利用StructuredOutputConverter，我们可以有效地将模型的输出转换为可用的数据结构。之后，我们讨论了BeanOutputConverter、MapOutputConverter和ListOutputConverter的用例，并为每个用例提供了实际示例。此外，我们还深入研究了创建自定义转换器来处理更复杂的数据类型。借助这些工具，将AI驱动的结构化输出集成到Java应用程序中变得更加易于访问和管理，从而提高了LLM响应的可靠性和可预测性。