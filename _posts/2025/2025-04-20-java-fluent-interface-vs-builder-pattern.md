---
layout: post
title:  Java中流式接口和构建器模式的区别
category: designpattern
copyright: designpattern
excerpt: 构建器模式
---

## 1. 概述

在本教程中，我们将讨论流式接口设计模式，并将其与构建器模式进行比较。在探索流式接口模式的过程中，我们会意识到构建器模式只是一种可能的实现。由此，我们可以深入探讨设计流式API的最佳实践，包括诸如不变性和接口隔离原则等考量。

## 2. 流式接口

**流式接口(Fluent Interface)是一种面向对象的API设计，它允许我们以可读且直观的方式将方法调用链接在一起**。为了实现它，我们需要声明返回同一类对象的方法。这样，我们就能够将多个方法调用链接在一起，该模式常用于构建DSL(领域特定语言)。

例如，Java 8的Stream API使用了流式接口模式，允许用户以非常声明式的方式操作数据流。让我们看一个简单的例子，观察一下每一步之后是如何返回一个新的Stream的：

```java
Stream<Integer> numbers = Stream.of(1,3,4,5,6,7,8,9,10);

Stream<String> processedNumbers = numbers.distinct()
    .filter(nr -> nr % 2 == 0)
    .skip(1)
    .limit(4)
    .map(nr -> "#" + nr)
    .peek(nr -> System.out.println(nr));

String result = processedNumbers.collect(Collectors.joining(", "));
```

我们可以注意到，首先我们需要创建一个实现流式API模式的对象，在我们的例子中，这是通过静态方法Stream.of()实现的。之后，我们通过它的公共API进行操作，可以注意到每个方法都返回同一个类。最后，我们以一个返回不同类型的方法结束整个操作链。在我们的例子中，这是一个返回String的Collector。

## 3. 构建器设计模式

**[构建器设计模式](https://www.baeldung.com/cs/builder-pattern-vs-factory-pattern)是一种创建型设计模式，它将复杂对象的构造与其表示分离。构建器类实现了流式接口模式，并允许逐步创建对象**。

让我们看一下构建器设计模式的简单用法：

```java
User.Builder userBuilder = User.builder();

userBuilder = userBuilder
    .firstName("John")
    .lastName("Doe")
    .email("jd@gmail.com")
    .username("jd_2000")
    .id(1234L);

User user = userBuilder.build();
```

我们应该能够理解上例中讨论的所有步骤，流式接口设计模式由User.Builder类实现，该类使用User.builder()方法创建。之后，我们链接多个方法调用，指定User的各种属性，每个步骤都返回同一个类型：User.Builder。最后，我们通过build()方法调用退出流式接口，该方法实例化并返回User。**因此，我们可以肯定地说，构建器模式是流式API模式唯一可能的实现**。

## 4. 不变性

**如果我们想要创建一个具有流式接口的对象，我们需要考虑其[不可变性](https://www.baeldung.com/java-immutable-object)**。上一节中的User.Builder并不是一个不可变的对象，它会改变其内部状态，并且始终返回同一个实例-它自己：

```java
public static class Builder {
    private String firstName;
    private String lastName;
    private String email;
    private String username;
    private Long id;

    public Builder firstName(String firstName) {
        this.firstName = firstName;
        return this;
    }

    public Builder lastName(String lastName) {
        this.lastName = lastName;
        return this;
    }

    // other methods

    public User build() {
        return new User(firstName, lastName, email, username, id);
    }
}
```

另一方面，只要它们具有相同的类型，每次都可以返回一个新实例。让我们创建一个具有流式API的类，用于生成HTML：

```java
public class HtmlDocument {
    private final String content;

    public HtmlDocument() {
        this("");
    }

    public HtmlDocument(String html) {
        this.content = html;
    }

    public String html() {
        return format("<html>%s</html>", content);
    }

    public HtmlDocument header(String header) {
        return new HtmlDocument(format("%s <h1>%s</h1>", content, header));
    }

    public HtmlDocument paragraph(String paragraph) {
        return new HtmlDocument(format("%s <p>%s</p>", content, paragraph));
    }

    public HtmlDocument horizontalLine() {
        return new HtmlDocument(format("%s <hr>", content));
    }

    public HtmlDocument orderedList(String... items) {
        String listItems = stream(items).map(el -> format("<li>%s</li>", el)).collect(joining());
        return new HtmlDocument(format("%s <ol>%s</ol>", content, listItems));
    }
}
```

在本例中，我们将通过直接调用构造函数来获取Fluent类的实例。大多数方法都返回一个HtmlDocument对象，并遵循该模式，我们可以使用html()方法来结束整个调用链并获取结果字符串：

```java
HtmlDocument document = new HtmlDocument()
    .header("Principles of O.O.P.")
    .paragraph("OOP in Java.")
    .horizontalLine()
    .paragraph("The main pillars of OOP are:")
    .orderedList("Encapsulation", "Inheritance", "Abstraction", "Polymorphism");
String html = document.html();

assertThat(html).isEqualToIgnoringWhitespace(
  "<html>"
  +  "<h1>Principles of O.O.P.</h1>"
  +  "<p>OOP in Java.</p>"
  +  "<hr>"
  +  "<p>The main pillars of OOP are:</p>"
  +  "<ol>"
  +     "<li>Encapsulation</li>"
  +     "<li>Inheritance</li>"
  +     "<li>Abstraction</li>"
  +     "<li>Polymorphism</li>"
  +   "</ol>"
  + "</html>"
);
```

此外，由于HtmlDocument是不可变的，因此链中的每次方法调用都会生成一个新的实例。换句话说，如果我们在文档中添加一个段落，那么这个带有标题的文档将变成另一个对象：

```java
HtmlDocument document = new HtmlDocument()
    .header("Principles of O.O.P.");
HtmlDocument updatedDocument = document
    .paragraph("OOP in Java.");

assertThat(document).isNotEqualTo(updatedDocument);
```

## 5. 接口隔离原则

**[接口隔离原则](https://www.baeldung.com/java-interface-segregation)，也就是SOLID中的“I”，教导我们避免使用大型接口**。为了完全遵循这一原则，API的客户端不应该依赖任何它从未使用过的方法。

构建流式接口时，我们必须关注API的公共方法数量。我们可能会忍不住添加越来越多的方法，导致对象变得非常庞大。例如，Stream API就有40多个公共方法。让我们来看看流式接口HtmlDocument的公共API是如何演变的，为了保留前面的示例，我们将为本节创建一个新类：

```java
public class LargeHtmlDocument {
    private final String content;
    // constructors

    public String html() {
        return format("<html>%s</html>", content);
    }
    public LargeHtmlDocument header(String header) { ... }
    public LargeHtmlDocument headerTwo(String header) { ... }
    public LargeHtmlDocument headerThree(String header) { ... }
    public LargeHtmlDocument headerFour(String header) { ... }
    
    public LargeHtmlDocument unorderedList(String... items) { ... }
    public LargeHtmlDocument orderedList(String... items) { ... }
    
    public LargeHtmlDocument div(Object content) { ... }
    public LargeHtmlDocument span(Object content) { ... }
    public LargeHtmlDocument paragraph(String paragraph) { .. }
    public LargeHtmlDocument horizontalLine() { ...}
    // other methods
}
```

有很多解决方案可以缩小接口，其中之一是将方法分组，并用小而内聚的对象组合成HtmlDocument。例如，我们可以将API限制为3个方法：head()、body()和footer()，并使用对象组合来创建文档。请注意，这些小对象本身是如何暴露出流式的API的。

```java
String html = new LargeHtmlDocument()
        .head(new HtmlHeader(Type.PRIMARY, "title"))
        .body(new HtmlDiv()
                .append(new HtmlSpan()
                        .paragraph("learning OOP from John Doe")
                        .append(new HorizontalLine())
                        .paragraph("The pillars of OOP:")
                )
                .append(new HtmlList(ORDERED, "Encapsulation", "Inheritance", "Abstraction", "Polymorphism"))
        )
        .footer(new HtmlDiv()
                .paragraph("trademark John Doe")
        )
        .html();
```

## 6. 总结

在本文中，我们学习了流式API设计，我们探讨了构建器模式如何只是流式接口模式的一种实现。然后，我们深入研究了流式的API，并讨论了不可变性的问题。最后，我们解决了大型接口的问题，并学习了如何拆分API以符合接口隔离原则。