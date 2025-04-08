---
layout: post
title:  使用camel-jackson解组JSON数组
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

[Apache Camel](https://www.baeldung.com/apache-camel-intro)是一个强大的开源集成框架，实现了许多已知的[企业集成模式](https://www.baeldung.com/camel-integration-patterns)。

通常，在使用Camel进行消息路由时，我们会希望使用众多受支持的可插拔[数据格式](https://camel.apache.org/manual/latest/data-format.html)之一。鉴于JSON在许多现代API和数据服务中很流行，它成为一种常见的选择。

在本教程中，**我们将介绍几种使用camel-jackson组件将[JSON数组](https://www.baeldung.com/jackson-collection-array)解组为Java对象列表的方法**。

## 2. 依赖

首先，让我们将[camel-jackson-starter](https://mvnrepository.com/artifact/org.apache.camel.springboot/camel-jackson-starter)依赖添加到我们的 pom.xml中：

```xml
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
    <artifactId>camel-jackson-starter</artifactId>
    <version>3.21.0</version>
</dependency>
```

然后，我们还将专门为我们的单元测试添加[camel-test-spring-junit5](https://mvnrepository.com/artifact/org.apache.camel/camel-test-spring-junit5)依赖：

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring-junit5</artifactId>
    <version>3.21.0</version>
    <scope>test</scope>
</dependency>
```

## 3. 水果域类

在本教程中，我们将使用几个轻量级[POJO](https://www.baeldung.com/java-pojo-class)对象来模拟我们的水果域。

让我们继续定义一个具有id和name的类来代表水果：

```java
public class Fruit {

    private String name;
    private int id;

    // standard getter and setters
}
```

接下来，我们将定义一个容器来保存Fruit对象列表：

```java
public class FruitList {

    private List<Fruit> fruits;

    public List<Fruit> getFruits() {
        return fruits;
    }

    public void setFruits(List<Fruit> fruits) {
        this.fruits = fruits;
    }
}
```

在接下来的几节中，**我们将看到如何将表示水果列表的JSON字符串解组到这些域类中。最终，我们要寻找的是可以使用的List<Fruit\>类型的变量**。

## 4. 解组JSON FruitList

在第一个例子中，我们将使用JSON格式表示一个简单的水果列表：

```json
{
    "fruits": [
        {
            "id": 100,
            "name": "Banana"
        },
        {
            "id": 101,
            "name": "Apple"
        }
    ]
}
```

首先，**我们应该强调这个JSON代表一个包含名为fruits的属性的对象，该属性包含我们的数组**。

现在让我们设置Apache Camel[路由](https://www.baeldung.com/apache-camel-intro#domain-specific-language)来执行反序列化：

```java
@Bean
RoutesBuilder route() {
    return new RouteBuilder() {
        @Override
        public void configure() throws Exception {
            from("direct:jsonInput").unmarshal(new JacksonDataFormat(FruitList.class))
                    .to("mock:marshalledObject");
        }
    };
}
```

在此示例中，我们使用名为jsonInput的direct端点。接下来，我们调用unmarshal方法，该方法将Camel交换器上收到的每个消息主体反序列化为指定的数据格式。

**我们使用JacksonDataFormat类和FruitList自定义解组类型**，这本质上是[Jackson ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)的一个简单包装器，允许我们可以将其与JSON进行编组。

最后，我们将unmarshal方法的结果发送到名为marshalledObject的mock端点。正如我们将要看到的，这就是我们将如何测试我们的路由以查看它是否正常工作。

考虑到这一点，让我们继续编写第一个单元测试：

```java
@CamelSpringBootTest
@SpringBootTest
public class FruitListJacksonUnmarshalUnitTest {

    @Autowired
    private ProducerTemplate template;

    @EndpointInject("mock:marshalledObject")
    private MockEndpoint mock;

    @Test
    public void givenJsonFruitList_whenUnmarshalled_thenSuccess() throws Exception {
        mock.setExpectedMessageCount(1);
        mock.message(0).body().isInstanceOf(FruitList.class);

        String json = readJsonFromFile("/json/fruit-list.json");
        template.sendBody("direct:jsonInput", json);
        mock.assertIsSatisfied();

        FruitList fruitList = mock.getReceivedExchanges().get(0).getIn().getBody(FruitList.class);
        assertNotNull("Fruit lists should not be null", fruitList);

        List<Fruit> fruits = fruitList.getFruits();
        assertEquals("There should be two fruits", 2, fruits.size());

        Fruit fruit = fruits.get(0);
        assertEquals("Fruit name", "Banana", fruit.getName());
        assertEquals("Fruit id", 100, fruit.getId());

        fruit = fruits.get(1);
        assertEquals("Fruit name", "Apple", fruit.getName());
        assertEquals("Fruit id", 101, fruit.getId());
    }
}
```

让我们回顾一下测试的关键部分，以了解正在发生的事情：

- 首先，我们添加@CamelSpringBootTest注解，这是一个使用Spring Boot测试Camel的有用注解。
- 然后我们使用@EndpointInject注入MockEndpoint，此注解允许我们设置Mock端点，而无需从Camel上下文中手动查找它们。在这里，我们将mock:marshalledObject设置为Camel端点的URI，它指向名为marshalledObject的mock端点。
- 之后，我们设置测试期望，我们的mock端点应该接收一条消息，消息类型应该是FruitList。
- **现在我们准备将JSON输入文件作为字符串发送到我们之前定义的direct端点，我们使用ProducerTemplate类将消息发送到端点**。
- 在我们检查mock期望已经得到满足后，我们可以自由地检索FruitList并检查内容是否符合预期。

此测试确认我们的路由正常运行，并且JSON正在按预期进行解组。

## 5. 解组JSON Fruit数组

另一方面，我们可能希望避免使用容器对象来保存我们的Fruit对象，我们可以修改JSON以直接保存水果数组：

```json
[
    {
        "id": 100,
        "name": "Banana"
    },
    {
        "id": 101,
        "name": "Apple"
    }
]
```

这一次，我们的路由几乎相同，但我们将其设置为专门与JSON数组一起使用：

```java
@Bean
RoutesBuilder route() {
    return new RouteBuilder() {
        @Override
        public void configure() throws Exception {
            from("direct:jsonInput").unmarshal(new ListJacksonDataFormat(Fruit.class))
                    .to("mock:marshalledObject");
        }
    };
}
```

我们可以看到，与之前的示例唯一的不同是，我们使用了ListJacksonDataFormat类和Fruit的自定义解组类型。**这是一种Jackson数据格式类型，我们可以使用它来处理列表**。

同样，我们的单元测试非常相似：

```java
@Test
public void givenJsonFruitArray_whenUnmarshalled_thenSuccess() throws Exception {
    mock.setExpectedMessageCount(1);
    mock.message(0).body().isInstanceOf(List.class);

    String json = readJsonFromFile("/json/fruit-array.json");
    template.sendBody("direct:jsonInput", json);
    mock.assertIsSatisfied();

    @SuppressWarnings("unchecked")
    List<Fruit> fruitList = mock.getReceivedExchanges().get(0).getIn().getBody(List.class);
    assertNotNull("Fruit lists should not be null", fruitList);

    assertEquals("There should be two fruits", 2, fruitList.size());

    Fruit fruit = fruitList.get(0);
    assertEquals("Fruit name", "Banana", fruit.getName());
    assertEquals("Fruit id", 100, fruit.getId());

    fruit = fruitList.get(1);
    assertEquals("Fruit name", "Apple", fruit.getName());
    assertEquals("Fruit id", 101, fruit.getId());
}
```

然而，与我们在上一节中看到的测试相比，有两个细微的差别：

- 我们首先设置我们的mock期望，以直接使用List.class来包含主体。
- 当我们以List.class形式检索消息正文时，我们会收到有关类型安全的标准警告-因此使用@SuppressWarnings("unchecked")。

## 6. 总结

在这篇短文中，我们看到了两种使用Camel消息路由和Camel-jackson组件解组JSON数组的简单方法，这两种方法之间的主要区别在于JacksonDataFormat反序列化为对象类型，而ListJacksonDataFormat反序列化为列表类型。