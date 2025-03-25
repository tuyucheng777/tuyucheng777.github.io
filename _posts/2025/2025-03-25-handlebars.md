---
layout: post
title:  Handlebars模版引擎介绍
category: libraries
copyright: libraries
excerpt: Handlebars
---

## 1. 概述

在本教程中，我们将研究[Handlebars.java](https://jknack.github.io/handlebars.java/)库以简化模板管理。

## 2. Maven依赖

让我们从添加[handlebars](https://search.maven.org/search?q=g:com.github.jknack%2Ba:handlebars)依赖开始：

```xml
<dependency>
    <groupId>com.github.jknack</groupId>
    <artifactId>handlebars</artifactId>
    <version>4.1.2</version>
</dependency>
```

## 3. 一个简单的模板

Handlebars模板可以是任何类型的文本文件，**它由{{name}}和{{#each people}}等标签组成**。

然后我们通过传递上下文对象(如Map或其他对象)来填充这些标签。

### 3.1 使用this

**要将单个字符串值传递给我们的模板，我们可以使用任何Object作为上下文**，我们还必须在我们的模板中使用{{this}}标签。

然后Handlebars在上下文对象上调用toString方法并用结果替换标签：

```java
@Test
public void whenThereIsNoTemplateFile_ThenCompilesInline() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compileInline("Hi {{this}}!");
    
    String templateString = template.apply("Tuyucheng");
    
    assertThat(templateString).isEqualTo("Hi Tuyucheng!");
}
```

在上面的示例中，我们首先创建了一个Handlebars实例，这是我们的API入口点。

然后，我们将模板赋予该实例。在这里，**我们只是内联传递模板**，但稍后我们会看到一些更强大的方法。

最后，我们为编译后的模板提供上下文。{{this}}将最终调用toString，这就是我们看到“Hi Tuyucheng”的原因。

### 3.2 将Map作为上下文对象传递

我们刚刚看到了如何为我们的上下文发送一个字符串，现在让我们尝试一个Map：

```java
@Test
public void whenParameterMapIsSupplied_thenDisplays() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compileInline("Hi {{name}}!");
    Map<String, String> parameterMap = new HashMap<>();
    parameterMap.put("name", "Tuyucheng");
    
    String templateString = template.apply(parameterMap);
    
    assertThat(templateString).isEqualTo("Hi Tuyucheng!");
}
```

与前面的示例类似，我们编译模板，然后传递上下文对象，但这次是作为Map传递。

另外，请注意我们使用的是{{name}}而不是{{this}}，**这意味着我们的Map必须包含键name**。

### 3.3 将自定义对象作为上下文对象传递

**我们还可以将自定义对象传递给我们的模板**：

```java
public class Person {
    private String name;
    private boolean busy;
    private Address address = new Address();
    private List<Person> friends = new ArrayList<>();

    public static class Address {
        private String street;
    }
}
```

使用Person类，我们将获得与前面示例相同的结果：

```java
@Test
public void whenParameterObjectIsSupplied_ThenDisplays() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compileInline("Hi {{name}}!");
    Person person = new Person();
    person.setName("Tuyucheng");
    
    String templateString = template.apply(person);
    
    assertThat(templateString).isEqualTo("Hi Tuyucheng!");
}
```

我们模板中的{{name}}将深入到我们的Person对象并获取name字段的值。

## 4. 模板加载器

到目前为止，我们已经使用了在代码中定义的模板。但是，这不是唯一的选择，我们还**可以从文本文件中读取模板**。

Handlebars.java为从类路径、文件系统或Servlet上下文中读取模板提供特殊支持，**默认情况下，Handlebars扫描类路径以加载给定的模板**：

```java
@Test
public void whenNoLoaderIsGiven_ThenSearchesClasspath() throws IOException {
    Handlebars handlebars = new Handlebars();
    Template template = handlebars.compile("greeting");
    Person person = getPerson("Tuyucheng");
    
    String templateString = template.apply(person);
    
    assertThat(templateString).isEqualTo("Hi Tuyucheng!");
}
```

因此，因为我们调用了compile而不是compileInline，这提示Handlebars在类路径上查找/greeting.hbs。

但是，我们也可以使用ClassPathTemplateLoader配置这些属性：

```java
@Test
public void whenClasspathTemplateLoaderIsGiven_ThenSearchesClasspathWithPrefixSuffix() throws IOException {
    TemplateLoader loader = new ClassPathTemplateLoader("/handlebars", ".html");
    Handlebars handlebars = new Handlebars(loader);
    Template template = handlebars.compile("greeting");
    // ... same as before
}
```

在这种情况下，我们**告诉Handlebars在classpath中查找/handlebars/greeting.html**。

最后，我们可以链接多个TemplateLoader实例：

```java
@Test
public void whenMultipleLoadersAreGiven_ThenSearchesSequentially() throws IOException {
    TemplateLoader firstLoader = new ClassPathTemplateLoader("/handlebars", ".html");
    TemplateLoader secondLoader = new ClassPathTemplateLoader("/templates", ".html");
    Handlebars handlebars = new Handlebars().with(firstLoader, secondLoader);
    // ... same as before
}
```

因此，这里我们有两个加载器，这意味着Handlebars将在两个目录中搜索greeting模板。

## 5. 内置助手

内置助手在编写模板时为我们提供了额外的功能。

### 5.1 with助手

**with帮助程序更改当前上下文**：

```html
{{#with address}}
<h4>I live in {{street}}</h4>
{{/with}}
```

在我们的示例模板中，{{#with address}}标签开始该部分，{{/with}}标签结束该部分。

**本质上，我们正在深入研究当前上下文对象(比方说person)，并将address设置为with部分的本地上下文**。此后，此部分中的每个字段引用都将加上person.address。

因此，{{street}}标签将保存person.address.street的值：

```java
@Test
public void whenUsedWith_ThenContextChanges() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("with");
    Person person = getPerson("Tuyucheng");
    person.getAddress().setStreet("World");
    
    String templateString = template.apply(person);
    
    assertThat(templateString).contains("<h4>I live in World</h4>");
}
```

我们正在编译模板并将Person实例分配为上下文对象。请注意，Person类有一个address字段，这是我们提供给with助手的字段。

**虽然我们进入了上下文对象的一个级别，但如果上下文对象有多个嵌套级别，则可以更深入地进行**。

### 5.2 each助手

**each助手会遍历一个集合**：

```html
{{#each friends}}
<span>{{name}} is my friend.</span>
{{/each}}
```

由于使用{{#each friends}}和{{/each}}标签开始和结束迭代部分，Handlebars将遍历上下文对象的friends字段。

```java
@Test
public void whenUsedEach_ThenIterates() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("each");
    Person person = getPerson("Tuyucheng");
    Person friend1 = getPerson("Java");
    Person friend2 = getPerson("Spring");
    person.getFriends().add(friend1);
    person.getFriends().add(friend2);
    
    String templateString = template.apply(person);
    
    assertThat(templateString)
        .contains("<span>Java is my friend.</span>", "<span>Spring is my friend.</span>");
}
```

在示例中，我们将两个Person实例分配给上下文对象的friends字段。因此，Handlebars在最终输出中重复了两次HTML部分。

### 5.3 if助手

最后，**if助手提供条件渲染**。

```html
{{#if busy}}
<h4>{{name}} is busy.</h4>
{{else}}
<h4>{{name}} is not busy.</h4>
{{/if}}
```

在我们的模板中，我们根据busy字段提供不同的消息。

```java
@Test
public void whenUsedIf_ThenPutsCondition() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("if");
    Person person = getPerson("Tuyucheng");
    person.setBusy(true);
    
    String templateString = template.apply(person);
    
    assertThat(templateString).contains("<h4>Tuyucheng is busy.</h4>");
}
```

编译模板后，我们设置上下文对象。由于busy字段为true，最终输出变为“<h4\>Tuyucheng is busy.</h4\>”。

## 6. 自定义模板助手

我们还可以创建自己的自定义助手。

### 6.1 Helper

**Helper接口使我们能够创建模板助手**。

作为第一步，我们必须提供Helper的实现：

```java
new Helper<Person>() {
    @Override
    public Object apply(Person context, Options options) throws IOException {
        String busyString = context.isBusy() ? "busy" : "available";
        return context.getName() + " - " + busyString;
    }
}
```

我们可以看到，Helper接口只有一个方法，它接收context和options对象。出于我们的目的，我们将输出Person的name和busy字段。

**创建助手后，我们还必须向Handlebars注册我们的自定义助手**：

```java
@Test
public void whenHelperIsCreated_ThenCanRegister() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    handlebars.registerHelper("isBusy", new Helper<Person>() {
        @Override
        public Object apply(Person context, Options options) throws IOException {
            String busyString = context.isBusy() ? "busy" : "available";
            return context.getName() + " - " + busyString;
        }
    });

    // implementation details
}
```

在我们的示例中，我们使用Handlebars.registerHelper()方法以isBusy的名称注册我们的助手。

**作为最后一步，我们必须使用助手的名称在模板中定义一个标签**：

```html
{{#isBusy this}}{{/isBusy}}
```

请注意，每个助手都有一个开始和结束标签。

### 6.2 辅助方法

**当我们使用Helper接口时，我们只能创建一个助手。相反，助手源类使我们能够定义多个模板助手**。

此外，我们不需要实现任何特定的接口。我们只需在一个类中编写辅助方法，然后HandleBars使用反射提取助手定义：

```java
public class HelperSource {

    public String isBusy(Person context) {
        String busyString = context.isBusy() ? "busy" : "available";
        return context.getName() + " - " + busyString;
    }

    // Other helper methods
}
```

由于辅助源可以包含多个助手实现，因此注册与单个助手注册不同：

```java
@Test
public void whenHelperSourceIsCreated_ThenCanRegister() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    handlebars.registerHelpers(new HelperSource());
    
    // Implementation details
}
```

我们使用Handlebars.registerHelpers()方法注册助手，此外，**辅助方法的名称将成为助手标签的名称**。

## 7. 模板重用

Handlebars库提供了多种方法来重用我们现有的模板。

### 7.1 模板包含

**模板包含是重用模板的方法之一，它有利于模板的组合**。

```html
<h4>Hi {{name}}!</h4>
```

这是header模板的内容-header.html。

为了在另一个模板中使用它，我们必须引用header模板。

```html
{{>header}}
<p>This is the page {{name}}</p>
```

我们有page模板page.html，其中包括使用{{>header}}的页眉模板。

当Handlebars.java处理模板时，最终的输出也会包含header的内容：

```java
@Test
public void whenOtherTemplateIsReferenced_ThenCanReuse() throws IOException {
    Handlebars handlebars = new Handlebars(templateLoader);
    Template template = handlebars.compile("page");
    Person person = new Person();
    person.setName("Tuyucheng");
    
    String templateString = template.apply(person);
    
    assertThat(templateString)
        .contains("<h4>Hi Tuyucheng!</h4>", "<p>This is the page Tuyucheng</p>");
}
```

### 7.2 模板继承

作为组合的替代方案，**Handlebars提供了模板继承**。

我们可以使用{{#block}}和{{#partial}}标签实现继承关系：

```html
<html>
<body>
{{#block "intro"}}
  This is the intro
{{/block}}
{{#block "message"}}
{{/block}}
</body>
</html>
```

通过这样做，messagebase模板有两个块-intro和message。

要应用继承，我们需要使用{{#partial}}覆盖其他模板中的这些块：

```html
{{#partial "message" }}
  Hi there!
{{/partial}}
{{> messagebase}}
```

这是simplemessage模板，请注意，我们包含了messagebase模板并覆盖了message块。

## 8. 总结

在本教程中，我们研究了使用Handlebars.java来创建和管理模板。

我们从基本的标签使用开始，然后研究了加载Handlebars模板的不同选项。

我们还研究了提供大量功能的模板助手。最后，我们研究了重用模板的不同方法。