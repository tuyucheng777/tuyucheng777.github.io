---
layout: post
title:  使用Java JMS读取和写入IBM MQ队列
category: messaging
copyright: messaging
excerpt: IBM MQ
---

## 1. 简介

在本教程中，我们将学习如何使用Java JMS(Java消息服务)从IBM MQ队列读取和写入消息。

## 2. 设置环境

为了避免手动安装和配置的复杂性，我们可以在[Docker](https://www.baeldung.com/ops/docker-guide)容器内运行IBM MQ，我们可以使用以下命令以基本配置运行容器：

```shell
docker run -d --name my-mq -e LICENSE=accept -e MQ_QMGR_NAME=QM1 MQ_QUEUE_NAME=QUEUE1 -p 1414:1414 -p 9443:9443 ibmcom/mq
```

接下来，我们需要在pom.xml文件中添加[IBM MQ客户端](https://mvnrepository.com/artifact/com.ibm.mq/com.ibm.mq.allclient/)：

```xml
<dependency>
    <groupId>com.ibm.mq</groupId>
    <artifactId>com.ibm.mq.allclient</artifactId>
    <version>9.4.0.0</version>
</dependency>
```

## 3. 配置JMS连接

首先，我们需要使用QueueConnectionFactory设置JMS连接，用于创建与队列管理器的连接：

```java
public class JMSSetup {
    public QueueConnectionFactory createConnectionFactory() throws JMSException {
        MQQueueConnectionFactory factory = new MQQueueConnectionFactory();
        factory.setHostName("localhost");
        factory.setPort(1414);
        factory.setQueueManager("QM1");
        factory.setChannel("SYSTEM.DEF.SVRCONN"); 
        
        return factory;
    }
}
```

我们首先创建一个MQQueueConnectionFactory实例，用于配置和创建与IBM MQ服务器的连接。**我们将主机名设置为localhost，因为MQ服务器在Docker容器内本地运行**，端口1414已从Docker容器映射到主机。

然后我们使用默认通道SYSTEM.DEF.SVRCONN，**这是客户端连接到IBM MQ的通用通道**。

## 4. 将消息写入IBM MQ队列

在本节中，我们将介绍向IBM MQ队列发送消息的过程。

### 4.1 建立JMS连接

首先，我们创建MessageSender类，该类负责建立与IBM MQ服务器的连接、管理会话以及处理消息发送操作。我们声明了QueueConnectionFactory、QueueConnection、QueueSession和QueueSender的实例变量，用于与IBM MQ服务器交互。

以下是IBM MQ连接设置、会话创建和消息发送的示例实现：

```java
public class MessageSender {
    private QueueConnectionFactory factory;
    private QueueConnection connection;
    private QueueSession session;
    private QueueSender sender;

    public MessageSender() throws JMSException {
        factory = new JMSSetup().createConnectionFactory();
        connection = factory.createQueueConnection();
        session = connection.createQueueSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue("QUEUE1");
        sender = session.createSender(queue);
        connection.start();
    }

    // ...
}
```

接下来，在MessageSender的构造函数中，我们使用JMSSetup类初始化QueueConnectionFactory。然后，**该工厂用于创建QueueConnection**，此连接允许我们与IBM MQ服务器进行交互。

连接建立后，我们使用createQueueSession()创建一个QueueSession，这个会话允许我们与队列进行通信。**这里我们传递false来指示该会话是非事务性的，并传递Session.AUTO_ACKNOWLEDGE来表示在收到消息时自动确认**。

之后，我们定义具体的队列“QUEUE1”，并创建一个QueueSender来处理消息的发送。**最后，我们启动连接，以确保会话处于活跃状态并准备好传输消息**。

### 4.2 写入文本消息

现在我们已经建立了连接、创建了会话、定义了队列并创建了消息生产者，我们准备向队列发送文本消息：

```java
public void sendMessage(String messageText) {
    try {
        TextMessage message = session.createTextMessage();
        message.setText(messageText);
        sender.send(message);
    } catch (JMSException e) {
        // handle exception
    } finally {
        // close resources
    }
}
```

首先，我们创建一个sendMessage()方法，该方法接收一个messageText参数，**该方法负责向队列发送一条文本消息**，它创建一个TextMessage对象，并使用setText()方法设置消息内容。

接下来，我们使用QueueSender对象的send()方法将消息发送到定义的队列，这种设计可以实现高效的消息传输，因为只要MessageSender对象存在，连接和会话就会一直保持打开状态。

### 4.3 消息类型

除了TextMessage之外，IBM MQ还支持多种其他消息类型，以满足不同的用例。例如，我们可以发送以下内容：

- BytesMessage：消息以字节的形式保存原始二进制数据
- ObjectMessage：消息携带序列化的Java对象
- MapMessage：包含键值对的消息
- StreamMessage：包含原始数据类型流的消息

## 5. 从IBM MQ队列读取消息

现在我们已经向队列发送了消息，让我们探索如何从队列中读取消息。

### 5.1 建立JMS连接并创建会话

首先，我们需要建立连接并创建会话，类似于发送消息时的操作。我们首先创建一个MessageReceiver类，**该类负责处理与IBM MQ服务器的连接，并设置消息消费所需的组件**：

```java
public class MessageReceiver {
    private QueueConnectionFactory factory;
    private QueueConnection connection;
    private QueueSession session;
    private QueueReceiver receiver;

    public MessageReceiver() throws JMSException {
        factory = new JMSSetup().createConnectionFactory();
        connection = factory.createQueueConnection();
        session = connection.createQueueSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue("QUEUE1");
        receiver = session.createReceiver(queue);
        connection.start();
    }

    // ...
}
```

在这个类中，我们首先创建一个QueueConnectionFactory来建立与IBM MQ服务器的连接。**然后，我们使用此连接创建一个QueueSession，以便与队列进行交互**。

最后，我们定义特定的队列“QUEUE1”，并创建一个QueueReceiver来处理来自队列的传入消息。

### 5.2 读取消息

一旦连接、会话和接收器设置完毕，我们就可以开始从队列接收消息了，我们使用QueueReceiver的receive()方法从指定的队列中提取消息：

```java
public void receiveMessage() {
    try {
        Message message = receiver.receive(1000);
        if (message instanceof TextMessage) {
            TextMessage textMessage = (TextMessage) message;
        } else {
            // ...
        }
    } catch (JMSException e) {
        // handle exception
    } finally {
        // close resources
    }
}
```

在receiveMessage()方法中，我们使用receive()函数等待来自队列的消息，超时时间为1000毫秒。**收到消息后，我们会检查该消息是否为TextMessage类型**。

如果是，我们可以使用getText()方法检索实际的消息内容，该方法以字符串形式返回文本内容。

## 6. 消息属性和标头

在本节中，我们将讨论在发送或接收消息时可以使用的一些常用[消息属性和标头](https://www.ibm.com/docs/en/svdi/7.2.0.3?topic=connector-jms-headers-properties)。

### 6.1 消息属性

消息属性可用于存储和检索消息正文以外的附加信息，**这对于过滤消息或向消息添加上下文数据特别有用**，以下是在发送消息时设置自定义属性的方法：

```java
TextMessage message = session.createTextMessage();
message.setText(messageText);

message.setStringProperty("OrderID", "12345");
```

接下来，我们可以在接收消息时检索属性：

```java
Message message = receiver.receive(1000);
if (message instanceof TextMessage) {
    TextMessage textMessage = (TextMessage) message;
    String orderID = message.getStringProperty("OrderID");
} 
```

### 6.2 消息头

**消息标头提供包含消息元数据的预定义字段**，一些常用的消息标头包括：

- JMSMessageID：JMS提供程序为每条消息分配的唯一标识符，我们可以使用此ID来跟踪和记录消息
- JMSExpiration：定义消息过期时间(毫秒)，如果消息未在此时间内送达，则会被丢弃
- JMSTimestamp：消息发送的时间
- JMSPriority：消息的优先级

让我们看看如何在接收消息时检索消息头：

```java
Message message = receiver.receive(1000);

if (message instanceof TextMessage) {
    TextMessage textMessage = (TextMessage) message;
    String messageId = message.getJMSMessageID();
    long timestamp = message.getJMSTimestamp();
    long expiration = message.getJMSExpiration();
    int priority = message.getJMSPriority();
}
```

## 7. 使用Mockito进行Mock测试

在本节中，我们将使用[Mockito](https://www.baeldung.com/mockito-annotations) Mock依赖项，并验证MessageSender和MessageReceiver类的交互。**首先，我们使用@Mock注解创建依赖项的Mock实例**。

接下来，我们验证sendMessage()方法是否能与Mock的QueueSender正确交互，我们Mock了QueueSender的send()方法，并验证TextMessage是否正确创建：

```java
String messageText = "Hello Tuyucheng! Nice to meet you!";
doNothing().when(sender).send(any(TextMessage.class));

messageSender.sendMessage(messageText);

verify(sender).send(any(TextMessage.class));
verify(textMessage).setText(messageText);
```

最后，我们验证receiveMessage()方法是否与Mock的QueueReceiver正确交互。我们Mock了receive()方法，使其返回一个预定义的TextMessage，并且消息文本也按预期检索到：

```java
when(receiver.receive(anyLong())).thenReturn(textMessage);
when(textMessage.getText()).thenReturn("Hello Tuyucheng! Nice to meet you!");

messageReceiver.receiveMessage();
verify(textMessage).getText();
```

## 8. 总结

在本文中，我们探讨了设置JMS连接、会话以及消息生产者/接收器以与IBM MQ队列交互的过程，我们还介绍了IBM MQ支持的几种消息类型。此外，我们还重点介绍了自定义属性和标头如何增强消息处理。