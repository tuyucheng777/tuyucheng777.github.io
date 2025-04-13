---
layout: post
title:  使用poi-tl模板生成MS Word文档
category: apache
copyright: apache
excerpt: poi-tl
---

## 1. 概述

[poi-tl](https://github.com/Sayi/poi-tl)库是一个基于[Apache POI](https://www.baeldung.com/java-microsoft-word-with-apache-poi)的开源Java库，它简化了使用模板生成Word文档的过程。**poi-tl库是一个Word模板引擎，可以根据Word模板和数据创建新文档**。

我们可以在模板中指定样式，通过模板生成的文档将保留指定的样式，模板是声明式的，并且纯粹基于标签，针对图像、文本、表格等元素提供不同的标签模式。poi-tl库还支持自定义插件，以便根据需要构建文档。

在本文中，我们将介绍模板中可以使用的不同标签以及模板中自定义插件的使用。

## 2. 依赖

为了使用poi-tl库，我们将其Maven依赖添加到项目中：

```xml
<dependency>
    <groupId>com.deepoove</groupId>
    <artifactId>poi-tl</artifactId>
    <version>1.12.2</version>
</dependency>
```

可以在Maven Central中找到[poi-tl](https://mvnrepository.com/artifact/com.deepoove/poi-tl)库的最新版本。

## 3. 配置

我们使用ConfigureBuilder类来构建配置：

```java
ConfigureBuilder builder = Configure.builder();
...
XWPFTemplate template = XWPFTemplate.compile(...).render(templateData);
template.writeAndClose(...);
```

模板文件采用Word.docx文件格式，**首先，我们使用XWPFTemplate类中的compile方法编译模板。模板引擎编译模板，并在新的docx文件中相应地渲染templateData**。此处，writeAndClose创建一个新文件，并以模板中指定的格式写入指定的数据。templateData是一个[HashMap](https://www.baeldung.com/java-hashmap)实例，以String为键，Object为值。

我们可以根据自己的喜好配置模板引擎。

### 3.1 标签前缀和后缀

模板引擎使用花括号{{}}来表示标签，我们可以将其设置为${}或任何其他我们想要的形式：

```java
builder.buildGramer("${", "}");
```

### 3.2 标签类型

模板引擎默认为模板标签定义了标识符，例如：@表示图片标签，#表示表格标签等等；标签的标识符可以配置：

```java
builder.addPlugin('@', new TableRenderPolicy());
builder.addPlugin('#', new PictureRenderPolicy());
```

### 3.3 标签名称格式

标签名称默认支持不同字符、字母、数字、下划线的组合，使用正则表达式配置标签名称规则：

```java
builder.buildGrammerRegex("[\\w]+(\\.[\\w]+)*");
```

我们将在后续章节中介绍插件和错误处理等配置。

让我们假设本文的其余部分采用默认配置。

## 4. 模板标签

poi-tl库模板没有任何变量赋值或循环；它们完全基于标签。Map或[DataModel](https://www.baeldung.com/java-dto-pattern)将要渲染的数据与poi-tl库中的模板标签关联起来，在最初的示例中，我们将看到[Map](https://www.baeldung.com/java-map-vs-hashmap)和DataModel。

**标签由两个花括号括起来的标签名称组成**：

```text
{{tagName}}
```

让我们了解一下基本标签。

### 4.1 文本

**文本标签代表文档中的普通文本，标签名称用两个花括号括起来**：

```text
{{authorname}}
```

这里我们声明了一个名为authorname的文本标签，我们将其添加到template.docx文件中，然后，我们将数据值添加到Map中：

```java
this.templateData.put("authorname", Texts.of("John")
    .color("000000")
    .bold()
    .create());
```

模板数据渲染器在生成的文档中将文本标签authorname替换为指定的值John，此外，指定的样式(例如颜色和粗体)也会应用于生成的文档中。

### 4.2 图像

**对于图像标签，标签名称前面带有@**：

```text
{{@companylogo}}
```

这里我们定义了一个图片类型的companylogo标签，为了显示图片，我们将companylogo添加到数据Map中，并指定要显示的图片的路径。

```java
templateData.put("companylogo", "logo.png");
```

### 4.3 编号

对于编号列表，标签名称前面带有\*：

```java
{{*bulletlist}}
```

在这个模板中，我们声明一个名为bulletlist的编号列表，并添加列表数据：

```java
List<String> list = new ArrayList<String>();
list.add("Plug-in grammar");
// ...

NumberingBuilder builder = Numberings.of(NumberingFormat.DECIMAL);
for(String s:list) {
    builder.addItem(s);
}
NumberingRenderData renderData = builder.create();	
this.templateData.put("list", renderData);
```

不同的编号格式，如NumberingFormat.DECIMAL、NumberingFormat.LOWER_LETTER、NumberingFormat.LOWER_ROMAN等，配置列表编号样式。

### 4.4 章节

**开始和结束标签用于标记章节，开始标签中，名称前面带有?；结束标签中，名称前面带有/**：

```java
{{?students}} {{name}} {{/students}}
```

我们创建一个名为“students”的版块，并在版块内添加标签name，我们将版块数据添加到Map中：

```java
Map<String, Object> students = new LinkedHashMap<String, Object>();
students.put("name", "John");
students.put("name", "Mary");
students.put("name", "Disarray");
this.templateData.put("students", students);
```

### 4.5 表格

**让我们使用模板在Word文档中生成表格结构，对于表格结构，标签名称前面带有#**：

```java
{{#table0}}
```

我们添加一个名为table0的表格标签，并使用Tables类方法添加表格数据：

```java
templateData.put("table0", Tables.of(new String[][] { new String[] { "00", "01" }, new String[] { "10", "11" } })
    .border(BorderStyle.DEFAULT)
    .create());
```

Rows.of()方法可以单独定义行，并为表格行添加边框等样式：

```java
RowRenderData row01 = Rows.of("Col0", "col1", "col2")
    .center()
    .bgColor("4472C4")
    .create();
RowRenderData row01 = Rows.of("Col10", "col11", "col12")
    .center()
    .bgColor("4472C4")
    .create();
templateData.put("table3", Tables.of(row01, row11)
    .create());
```

这里，table3有两行，行边框设置为颜色4472C4。

让我们使用MergeCellRule来创建单元格合并规则：

```java
MergeCellRule rule = MergeCellRule.builder()
    .map(Grid.of(1, 0), Grid.of(1, 2))
    .build();
templateData.put("table3", Tables.of(row01, row11)
    .mergeRule(rule)
    .create());
```

此处，单元格合并规则将表格第二行开始的单元格1和2合并，同样，也可以使用table标签的其他自定义规则。

### 4.5 嵌套

我们可以在一个模板中添加另一个模板，即模板嵌套。**对于嵌套的标签，标签名称前面会加一个+**：

```text
{{+nested}}
```

这会在模板中声明一个名为nested的嵌套标签，然后，我们设置要为嵌套文档渲染的数据：

```java
List<Address> subData = new ArrayList<>();
subData.add(new Address("Florida,USA"));
subData.add(new Address("Texas,USA"));
templateData.put("nested", Includes.ofStream(WordDocumentEditor.class.getClassLoader().getResourceAsStream("nested.docx")).setRenderModel(subData).create());
```

这里，我们从nested.docx加载模板，并为嵌套模板设置渲染数据；poi-tl库模板引擎将嵌套模板渲染为嵌套标签要显示的值或数据。

### 4.6 使用DataModel进行模板渲染

DataModel也可以为模板渲染数据，让我们创建一个Person类：

```java
public class Person {
    private String name;
    private int age;
    // ...
}
```

我们可以使用数据模型设置模板标签值：

```java
templateData.put("person", new Person("Jimmy", 35));
```

这里我们设置了一个名为person的数据模型，该模型具有name和age属性，**模板使用.运算符访问属性值**：

```text
{{person.name}}
{{person.age}}
```

类似地，我们可以在数据模型中使用不同类型的数据。

## 5. 插件

插件允许我们在模板标签位置执行预定义的函数，使用插件，我们可以在模板中的所需位置执行几乎任何操作。poi-tl库具有默认插件，无需显式配置。默认插件负责处理文本、图像、表格等的渲染。

此外，还有一些内置插件需要配置才能使用，我们也可以开发自己的插件，称为自定义插件；让我们来了解一下内置插件和自定义插件。

### 5.1 使用内置插件

对于注释，poi-tl提供了一个内置插件，用于在Word文档中注释，我们使用CommentRenderPolicy配置注释标签：

```java
builder.bind("comment", new CommentRenderPolicy());
```

这会在模板引擎中将comment标签注册为注释渲染器。

我们来看看评论插件CommentRenderPolicy的使用：

```java
CommentRenderData comment = Comments.of("SampleExample")
    .signature("John", "S", LocaleUtil.getLocaleCalendar())
    .comment("Authored by John")
    .create();
templateData.put("comment", comment);
```

模板引擎识别comment标签并将指定的注释放置在生成的文档中。

类似地，可以使用其他可用的[插件](https://deepoove.com/poi-tl/#plugin-list)。

### 5.2 自定义插件

我们可以为模板引擎创建自定义插件，数据可以根据自定义需求在文档中呈现。

**要定义自定义插件，我们需要实现RenderPolicy接口或扩展抽象类AbstractRenderPolicy**：

```java
public class SampleRenderPolicy implements RenderPolicy {
    @Override
    public void render(ElementTemplate eleTemplate, Object data, XWPFTemplate template) {
        XWPFRun run = ((RunTemplate) eleTemplate).getRun();
        String text = "Sample plugin " + String.valueOf(data);
        run.setText(textVal, 0);
    }
}
```

这里，我们使用SampleRenderPolicy类创建一个示例自定义插件，然后配置模板引擎以识别自定义插件：

```java
ConfigureBuilder builder = Configure.builder();
builder.bind("sample", new SampleRenderPolicy());
templateData.put("sample", "custom-plugin");
```

此配置使用名为sample的标签注册了我们的自定义插件，模板引擎会将模板中的sample标签替换为文本Sample plugin custom-plugin。

类似地，我们可以通过扩展AbstractRenderPolicy来开发更多定制的插件。

## 6. 日志记录

要为poi-tl库启用日志记录，我们可以使用[Logback](https://www.baeldung.com/logback)库，我们在logback.xml中添加poi-tl库的记录器：

```xml
<logger name="com.deepoove.poi" level="debug" additivity="false">
    <appender-ref ref="STDOUT" />
</logger>
```

此配置启用poi-tl库中包com.deepoove.poi的日志记录：

```text
18:01:15.488 [main] INFO  c.d.poi.resolver.TemplateResolver - Resolve the document start...
18:01:15.503 [main] DEBUG c.d.poi.resolver.RunningRunBody - {{title}}
18:01:15.503 [main] DEBUG c.d.poi.resolver.RunningRunBody - [Start]:The run position of {{title}} is 0, Offset in run is 0
18:01:15.503 [main] DEBUG c.d.poi.resolver.RunningRunBody - [End]:The run position of {{title}} is 0, Offset in run is 8
...

18:01:19.661 [main] INFO  c.d.poi.resolver.TemplateResolver - Resolve the document start...
18:01:19.685 [main] INFO  c.d.poi.resolver.TemplateResolver - Resolve the document end, resolve and create 0 MetaTemplates.
18:01:19.685 [main] INFO  c.deepoove.poi.render.DefaultRender - Successfully Render template in 4126 millis

```

我们可以通过日志来观察模板标签是如何解析的以及数据是如何呈现的。

## 7. 错误处理

**poi-tl库支持自定义引擎在发生错误时的行为**。

在几种无法计算标签的场景下，比如模板中引用了不存在的变量，或者级联谓词不是哈希，比如{{student.name}}当student的值为null时，无法计算name的值。

poi-tl库可以配置发生此错误时的计算结果。

默认情况下，标签值被认为是null，当我们需要严格检查模板是否存在人为错误时，可以抛出异常：

```java
builder.useDefaultEL(true);
```

poi-tl库的默认行为是清除标签，如果我们不想对标签执行任何操作：

```java
builder.setValidErrorHandler(new DiscardHandler());
```

要执行严格验证，直接抛出异常：

```java
builder.setValidErrorHandler(new AbortHandler());
```

## 8. 模板生成模板

模板引擎不仅可以生成文档，还可以生成新的模板，比如我们可以将原来的文本标签拆分成文本标签和表格标签：

```javascript
Configure config = Configure.builder().bind("title", new DocumentRenderPolicy()).build();

Map<String, Object> data = new HashMap<>();

DocumentRenderData document = Documents.of()
    .addParagraph(Paragraphs.of("{{title}}").create())
    .addParagraph(Paragraphs.of("{{#table}}").create())
    .create();
data.put("title", document);
```

这里我们给DocumentRenderPolicy配置了一个名为title的标签，并创建了一个DocumentRenderData对象，形成了模板结构。

模板引擎识别title标签，并生成包含放入数据的模板结构化document的Word文档。

## 9. 总结

在本文中，我们学习了如何使用poi-tl库模板的功能创建Word文档，我们还讨论了poi-tl库中不同类型的标签、日志记录和错误处理。