---
layout: post
title:  使用Java获取SOAP消息中的特定部分
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在设计用于数据交换的[API](https://www.baeldung.com/cs/api-endpoints)时，我们经常采用[REST或SOAP](https://www.baeldung.com/cs/rest-vs-soap)架构方法。在使用SOAP协议的情况下，有时我们需要从SOAP消息中提取一些特定数据以进行进一步处理。

在本教程中，我们将学习如何在Java中获取SOAP消息的特定部分。

## 2. SOAPMessage类

在深入研究之前，让我们简单检查一下[SOAPMessage](https://jakarta.ee/specifications/platform/10/apidocs/jakarta/xml/soap/soapmessage)类的结构，该类是所有SOAP消息的根类：

![](/assets/images/2025/webmodules/javasoapmsgspecificpart01.png)

该类由两个主要部分组成-SOAP部分和可选附件部分。前者包含SOAP信封，其中包含我们收到的实际消息。此外，信封本身由标头和正文元素组成。

**从Java 11开始，Java EE(包括[JAX-WS](https://www.baeldung.com/jax-ws)和SAAJ模块)已从JDK中删除，不再是标准发行版的一部分**。要使用Jakarta EE 9及更高版本成功处理SOAP消息，我们需要在pom.xml中添加[Jakarta SOAP with Attachment API](https://mvnrepository.com/artifact/jakarta.xml.soap/jakarta.xml.soap-api)和[Jakarta SOAP Implementation](https://mvnrepository.com/artifact/com.sun.xml.messaging.saaj/saaj-impl)依赖：

```xml
<dependency>
    <groupId>jakarta.xml.soap</groupId>
    <artifactId>jakarta.xml.soap-api</artifactId>
    <version>3.0.1</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.messaging.saaj</groupId>
    <artifactId>saaj-impl</artifactId>
    <version>3.0.3</version>
</dependency>
```

## 3. 示例

接下来，让我们创建一个将在本教程中使用的[XML消息](https://www.baeldung.com/java-xml)：

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:be="http://www.tuyucheng.com/soap/">
    <soapenv:Header>
        <be:Username>tuyucheng</be:Username>
    </soapenv:Header>
    <soapenv:Body>
        <be:ArticleRequest>
            <be:Article>
                <be:Name>Working with JUnit</be:Name>
            </be:Article>
        </be:ArticleRequest>
    </soapenv:Body>
</soapenv:Envelope>
```

## 4. 从SOAP消息中获取标头和正文

接下来，让我们看看如何从SOAP消息中提取标头和正文元素。

**根据SOAPMessage类层次结构，要获取实际的SOAP消息，我们首先需要获取SOAP部分，然后获取信封**：

```java
InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream("soap-message.xml");
SOAPMessage soapMessage = MessageFactory.newInstance().createMessage(new MimeHeaders(), inputStream);
SOAPPart part = soapMessage.getSOAPPart();
SOAPEnvelope soapEnvelope = part.getEnvelope();
```

现在，要获取标头元素，我们可以调用getHeader()方法：

```java
SOAPHeader soapHeader = soapEnvelope.getHeader();
```

类似地，我们可以通过调用getBody()方法提取body元素：

```java
SOAPBody soapBody = soapEnvelope.getBody();
```

## 5. 从SOAP消息中获取特定元素

现在我们已经讨论了检索基本元素，让我们探索如何从SOAP消息中提取特定部分。

### 5.1 根据标签名获取元素

**我们可以使用getElementsByTagName()方法来获取特定元素**，该方法返回一个NodeList。此外，Node是所有DOM组件的主要数据类型。换句话说，所有元素、属性和文本内容都被视为Node类型。

我们从XML中提取Name元素：

```java
@Test
void whenGetElementsByTagName_thenReturnCorrectBodyElement() throws Exception {
    SOAPEnvelope soapEnvelope = getSoapEnvelope();
    SOAPBody soapBody = soapEnvelope.getBody();
    NodeList nodes = soapBody.getElementsByTagName("be:Name");
    assertNotNull(nodes);

    Node node = nodes.item(0);
    assertNotNull(node);
    assertEquals("Working with JUnit", node.getTextContent());
}
```

**这里需要注意的是，我们需要将命名空间前缀传递给方法才能让它起作用**。

同样，我们可以使用相同的方法从SOAP标头中获取元素：

```java
@Test
void whenGetElementsByTagName_thenReturnCorrectHeaderElement() throws Exception {
    SOAPEnvelope soapEnvelope = getSoapEnvelope();
    SOAPHeader soapHeader = soapEnvelope.getHeader();
    NodeList nodes = soapHeader.getElementsByTagName("be:Username");
    assertNotNull(nodes);

    Node node = nodes.item(0);
    assertNotNull(node);
    assertEquals("tuyucheng", node.getTextContent());
}
```

### 5.2 迭代子节点

从特定元素获取值的另一种方法是遍历子节点。

让我们看看如何迭代body元素的子节点：

```java
@Test
void whenGetElementUsingIterator_thenReturnCorrectBodyElement() throws Exception {
    SOAPEnvelope soapEnvelope = getSoapEnvelope();
    SOAPBody soapBody = soapEnvelope.getBody();
    NodeList childNodes = soapBody.getChildNodes();

    for (int i = 0; i < childNodes.getLength(); i++) {
        Node node = childNodes.item(i);
        if ("Name".equals(node.getLocalName())) {
            String name = node.getTextContent();
            assertEquals("Working with JUnit", name);
        }
    }
}
```

### 5.3 使用XPath

接下来我们来看看如何使用[XPath](https://www.baeldung.com/java-xpath)来提取元素。简单来说，XPath是用来描述XML部分的语法。**此外，它还与XPath表达式配合使用，我们可以在特定条件下使用它来检索元素**。

首先，让我们创建一个新的XPath实例：

```java
XPathFactory xPathFactory = XPathFactory.newInstance();
XPath xpath = xPathFactory.newXPath();
```

为了有效地处理命名空间，让我们定义命名空间上下文：

```typescript
xpath.setNamespaceContext(new NamespaceContext() {
    @Override
    public String getNamespaceURI(String prefix) {
        if ("be".equals(prefix)) {
            return "http://www.tuyucheng.com/soap/";
        }
        return null;
    }

    // other methods
});
```

这样，XPath就知道在哪里寻找我们的数据。

接下来，让我们定义检索Name元素值的XPath表达式：

```java
XPathExpression expression = xpath.compile("//be:Name/text()");
```

在这里，我们使用路径表达式和返回节点文本内容的text()函数的组合创建了XPath表达式。

最后，让我们调用valuate()方法来检索匹配表达式的结果：

```java
String name = (String) expression.evaluate(soapBody, XPathConstants.STRING); 
assertEquals("Working with JUnit", name);
```

**此外，我们可以创建一个忽略命名空间的表达式**：

```java
@Test
void whenGetElementUsingXPathAndIgnoreNamespace_thenReturnCorrectResult() throws Exception {
    SOAPBody soapBody = getSoapBody();
    XPathFactory xPathFactory = XPathFactory.newInstance();
    XPath xpath = xPathFactory.newXPath();
    XPathExpression expression = xpath.compile("//*[local-name()='Name']/text()");

    String name = (String) expression.evaluate(soapBody, XPathConstants.STRING);
    assertEquals("Working with JUnit", name);
}
```

我们在表达式中使用了local-name()函数来忽略命名空间，因此，表达式选择任何具有局部名称Name的元素，而不考虑命名空间前缀。

## 6. 总结

在本文中，我们学习了如何从Java中的SOAP消息中获取特定部分。

总而言之，有多种方法可以从SOAP消息中检索某些元素。我们探索了多种方法，例如通过标签名称搜索元素、迭代子节点以及使用XPath表达式。