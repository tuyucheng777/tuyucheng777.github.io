---
layout: post
title:  使用Twilio在Java中发送短信
category: saas
copyright: saas
excerpt: Twilio
---

## 1. 简介

发送短信是许多现代应用程序的重要组成部分，短信可以用于各种场景：双重身份验证、实时警报、聊天机器人等等。

在本教程中，我们将构建一个使用[Twilio](https://www.twilio.com/)发送短信的简单Java应用程序。

有许多服务提供SMS功能，例如[Vonage](https://www.vonage.com/)、[Plivo](https://www.plivo.com/)、[Amazon Simple Notification Service](https://aws.amazon.com/sns/)(SNS)、[Zapier](https://www.zapier.com/)等。

使用Twilio Java客户端，**我们只需几行代码即可发送短信**。

## 2. 设置Twilio

**首先，我们需要一个Twilio账户**，他们提供了一个试用账户，足以测试其平台的所有功能。

作为帐户设置的一部分，我们还必须创建一个电话号码，这很重要，因为试用帐户需要经过验证的电话号码才能发送消息。

Twilio为新账户提供了[快速设置教程](https://www.twilio.com/docs/sms/quickstart/java)，完成账户设置并验证电话号码后，我们就可以开始发送消息了。

## 3. TwiML简介

在编写示例应用程序之前，让我们快速了解一下Twilio服务使用的数据交换格式。

TwiML是一种基于XML的专有标记语言，TwiML消息中的元素反映了我们可以执行的与电话相关的各种操作：拨打电话、录制消息、发送消息等等。

以下是用于发送短信的TwiML消息示例：

```xml
<Response>
    <Message>
        <Body>Sample Twilio SMS</Body>
    </Message>
</Response>
```

以下是另一个拨打电话的TwiML消息示例：

```xml
<Response>
    <Dial>
        <Number>415-123-4567</Number>
    </Dial>
</Response>
```

这两个例子虽然简单，但却能让我们更好地理解TwiML的本质，它由易于记忆的动词和名词组成，并且与我们用手机执行的操作直接相关。

## 4. 使用Twilio在Java中发送短信

Twilio提供了一个功能丰富的Java客户端，方便用户与其服务进行交互。**我们无需从头编写代码来构建TwiML消息，只需使用开箱即用的Java客户端即可**。

### 4.1 依赖

我们可以直接从[Maven Central](https://mvnrepository.com/artifact/com.twilio.sdk/twilio)下载依赖，或者通过在我们的pom.xml文件中添加以下元素：

```xml
<dependency>
    <groupId>com.twilio.sdk</groupId>
    <artifactId>twilio</artifactId>
    <version>7.20.0</version>
</dependency>
```

### 4.2 发送短信

首先，让我们看一些示例代码：

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
Message message = Message.creator(
    new PhoneNumber("+12225559999"),
    new PhoneNumber(TWILIO_NUMBER),
    "Sample Twilio SMS using Java")
.create();
```

让我们将上述示例中的代码分解为几个关键部分：

- 需要调用一次Twilio.init()来使用我们唯一的Account Sid和Token设置Twilio环境
- Message对象是Java中与我们之前看到的TwiML<Message\>元素等价的对象
- Message.creator()需要3个参数：收件人电话号码、发件人电话号码和消息正文
- create()方法处理发送消息

### 4.3 发送彩信

**Twilio API还支持发送多媒体消息**，我们可以混合使用文本和图像，但接收手机必须支持多媒体消息功能：

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
Message message = Message.creator(
    new PhoneNumber("+12225559999"),
    new PhoneNumber(TWILIO_NUMBER),
    "Sample Twilio MMS using Java")
.setMediaUrl(
    Promoter.listOfOne(URI.create("http://www.domain.com/image.png")))
.create();
```

## 5. 跟踪邮件状态

在前面的例子中，我们没有确认消息是否真的送达；不过，**Twilio提供了一种机制，让我们可以判断消息是否送达成功**。

### 5.1 消息状态码

发送消息时，它会随时具有以下状态之一：

- Queued：Twilio已收到消息并将其放入队列等待发送
- Sending：服务器正在将你的消息发送到网络中最近的上游运营商
- Sent：消息已被最近的上游运营商成功接收
- Delivered：Twilio已收到上游运营商的消息送达确认，并且可能已收到目标手机(如果可用)的确认
- Failed：无法发送消息
- Undelivered：服务器已收到送达回执，表明邮件未送达

请注意，对于最后两种状态，我们可以找到具有更多具体详细信息的错误代码，以帮助我们解决交付问题。

Twilio Java客户端提供了同步和异步方法来获取状态，我们来看一下。

### 5.2 检查交付状态(同步)

一旦创建了Message对象，我们就可以调用Message.getStatus()来查看它当前处于哪种状态：

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
ResourceSet messages = Message.reader().read();
for (Message message : messages) {
    System.out.println(message.getSid() + " : " + message.getStatus());
}
```

请注意，Message.reader().read()会进行远程API调用，因此请谨慎使用。**默认情况下，它会返回我们发送的所有消息**，但我们可以通过电话号码或日期范围筛选返回的消息。

### 5.3 检查交付状态(异步)

由于检索消息状态需要远程API调用，因此可能需要很长时间。为了避免阻塞当前线程，Twilio Java客户端还提供了Message.getStatus().read()的异步版本。

```java
Twilio.init(ACCOUNT_SID, AUTH_TOKEN);
ListenableFuture<ResourceSet<Message>> future = Message.reader().readAsync();
Futures.addCallback(
    future,
    new FutureCallback<ResourceSet<Message>>() {
        public void onSuccess(ResourceSet<Message> messages) {
            for (Message message : messages) {
                System.out.println(message.getSid() + " : " + message.getStatus());
             }
         }
         public void onFailure(Throwable t) {
             System.out.println("Failed to get message status: " + t.getMessage());
         }
    });
```

这使用Guava ListenableFuture接口在不同的线程上处理来自Twilio的响应。

## 6. 总结

在本文中，我们学习了如何使用Twilio和Java发送SMS和MMS。

虽然TwiML是所有往返于Twilio服务器的消息的基础，但Twilio Java客户端使发送消息变得非常容易。