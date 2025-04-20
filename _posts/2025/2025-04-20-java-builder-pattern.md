---
layout: post
title:  在Java中实现构建器模式
category: designpattern
copyright: designpattern
excerpt: 构建器模式
---

## 1. 简介

在软件开发过程中，我们经常会遇到创建具有众多属性的对象令人望而生畏的情况，杂乱的构造函数会降低代码的可读性，这正是构建器模式的闪光点。**构建器模式是一种[创建型设计模式](https://www.baeldung.com/creational-design-patterns)，它将复杂对象的构造与其表示分离，从而提供了一种更清晰、更灵活的对象创建方法**。

## 2. 构建器模式的优点

在深入编码之前，让我们快速回顾一下使用[构建器模式](https://www.baeldung.com/cs/builder-pattern-vs-factory-pattern)的优势：

- 灵活性：通过将构造过程与实际对象表示分离，构建器模式允许我们创建具有不同配置的对象，而不会因多个构造函数或Setter而导致代码库混乱。
- 可读性：构建器模式提供了流式的接口，使我们的代码更具可读性；这使我们和其他开发人员能够一目了然地了解复杂对象的构造过程。
- 不变性：构建器可以在构建完成后通过创建不可变对象来强制不变性；这确保了线程安全并防止了意外的修改。

## 3. 经典构建器模式

**在构建器模式的经典实现中，我们创建一个单独的构建者内部类，该内部类包含用于设置构造对象各个属性的方法**。这种结构化方法有助于实现顺序配置过程，确保代码清晰易用。此外，它还增强了代码的组织性和可读性，使其更易于理解和维护：

```java
public class Post {
    private final String title;

    private final String text;

    private final String category;

    Post(Builder builder) {
        this.title = builder.title;
        this.text = builder.text;
        this.category = builder.category;
    }

    public String getTitle() {
        return title;
    }

    public String getText() {
        return text;
    }

    public String getCategory() {
        return category;
    }

    public static class Builder {
        private String title;
        private String text;
        private String category;

        public Builder title(String title) {
            this.title = title;
            return this;
        }

        public Builder text(String text) {
            this.text = text;
            return this;
        }

        public Builder category(String category) {
            this.category = category;
            return this;
        }

        public Post build() {
            return new Post(this);
        }
    }
}
```

在构建器类中，我们声明了与外部类相同的字段集。Builder类提供了流式的方法来设置Post的每个属性，**此外，它还包含一个build()方法来创建Post实例**。

现在，我们可以使用Builder来创建一个新对象：

```java
Post post = new Post.Builder()
    .title("Java Builder Pattern")
    .text("Explaining how to implement the Builder Pattern in Java")
    .category("Programming")
    .build();
```

## 4. 泛型构建器模式

在Java 8中，[Lambda表达式](https://www.baeldung.com/java-8-lambda-expressions-tips)和方法引用开辟了新的可能性，包括更通用的构建器模式。我们的实现引入了GenericBuilder类，它可以利用泛型构造各种类型的对象：

```java
public class GenericBuilder<T> {
    private final Supplier<T> supplier;

    private GenericBuilder(Supplier<T> supplier) {
        this.supplier = supplier;
    }

    public static <T> GenericBuilder<T> of(Supplier<T> supplier) {
        return new GenericBuilder<>(supplier);
    }

    public <P> GenericBuilder<T> with(BiConsumer<T, P> consumer, P value) {
        return new GenericBuilder<>(() -> {
            T object = supplier.get();
            consumer.accept(object, value);
            return object;
        });
    }

    public T build() {
        return supplier.get();
    }
}
```

此类遵循流式接口，首先使用of()方法创建初始对象实例。然后，使用with()方法使用Lambda表达式或方法引用设置对象属性。

**GenericBuilder提供了灵活性和可读性，使我们能够简洁地构造每个对象，同时确保类型安全**。此模式展现了Java 8的强大表达能力，是解决复杂构造任务的优雅解决方案。

然而，这个解决方案的一个很大的缺点是它基于类的Setter方法。**这意味着我们的属性不再像上例中那样是final的，从而失去了构建器模式提供的不变性**。

对于我们的下一个示例，我们将创建一个新的GenericPost类，它由默认的无参数构造函数、Getter和Setter组成：

```java
public class GenericPost {

    private String title;

    private String text;

    private String category;

    // getters and setters
}
```

现在，我们可以使用GenericBuilder来创建GenericPost：

```java
Post post = GenericBuilder.of(GenericPost::new)
    .with(GenericPost::setTitle, "Java Builder Pattern")
    .with(GenericPost::setText, "Explaining how to implement the Builder Pattern in Java")
    .with(GenericPost::setCategory, "Programming")
    .build();
```

## 5. Lombok构建器

[Lombok](https://www.baeldung.com/intro-to-project-lombok)是一个库，它通过自动生成常用方法(如Getter、Setter、equals、hashCode甚至构造函数)来简化Java代码。

Lombok最受赞赏的功能之一是它对构建器模式的支持，**通过使用@Builder标注一个类，Lombok可以生成一个包含流式方法的构建器类来设置属性**。此注解消除了手动实现构建器类的需要，从而显著减少了代码的冗余。

要使用Lombok，我们需要从[Maven中央仓库](https://mvnrepository.com/artifact/org.projectlombok/lombok)导入依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.32</version>
</dependency>
```

现在，我们可以使用@Builder注解创建一个新的LombokPost类：

```java
@Builder
@Getter
public class LombokPost {
    private String title;
    private String text;
    private String category;
}
```

我们还使用了@Setter和@Getter注解来避免样板代码，然后我们可以使用现成的构建器模式来创建新对象：

```java
LombokPost lombokPost = LombokPost.builder()
    .title("Java Builder Pattern")
    .text("Explaining how to implement the Builder Pattern in Java")
    .category("Programming")
    .build();
```

## 6. 总结

Java 8中的构建器模式简化了对象构造，并提升了代码的可读性。借助经典构建器模式、泛型构建器模式和Lombok构建器模式等变体，我们可以根据特定需求定制方法。通过采用这种模式并利用Lombok等工具，我们可以编写更简洁、更高效的代码，从而推动软件开发的创新和成功。