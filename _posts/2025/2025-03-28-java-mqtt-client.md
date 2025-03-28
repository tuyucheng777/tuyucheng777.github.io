---
layout: post
title:  Java中的MQTT客户端
category: libraries
copyright: libraries
excerpt: Eclipse Paho
---

## 1. 概述

在本教程中，我们将了解如何使用[Eclipse Paho项目](https://www.eclipse.org/paho/)提供的库在Java项目中添加MQTT消息传递。

## 2. MQTT入门

**MQTT(MQ遥测传输)是一种消息传递协议**，旨在满足以简单轻量的方式与低功耗设备(例如工业应用中使用的设备)传输数据的需求。

随着IoT(物联网)设备的日益普及，MQTT的使用也日益增多，并最终被OASIS和ISO进行标准化。

该协议支持单一消息模式，即发布-订阅模式：客户端发送的每条消息都包含一个关联的“主题”，代理会使用该主题将其路由到订阅的客户端。主题名称可以是简单的字符串，如“oiltemp”，也可以是路径字符串“motor/1/rpm”。

为了接收消息，客户端使用其确切名称或包含支持的通配符之一的字符串(“#”表示多级主题，“+”表示单级)订阅一个或多个主题。

## 3. 项目设置

为了将Paho库包含在Maven项目中，我们必须添加以下依赖：

```xml
<dependency>
  <groupId>org.eclipse.paho</groupId>
  <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
  <version>1.2.0</version>
</dependency>
```

 可以从Maven Central下载最新版本的[Eclipse Paho](https://mvnrepository.com/artifact/org.eclipse.paho/org.eclipse.paho.client.mqttv3) Java库模块。

## 4. 客户端设置

使用Paho库时，为了从MQTT代理发送和/或接收消息，我们需要做的第一件事就是获取IMqttClient接口的实现。此接口包含应用程序建立与服务器的连接、发送和接收消息所需的所有方法。

Paho提供了此接口的两个实现，一个是异步实现(MqttAsyncClient)，另一个是同步实现(MqttClient)。在本例中，我们将重点介绍同步版本，因为它的语义更简单。

设置过程本身分为两个步骤：首先，我们创建MqttClient类的实例，然后将其连接到我们的服务器。以下小节详细介绍了这些步骤。

### 4.1 创建一个新的IMqttClient实例

以下代码片段显示如何创建一个新的IMqttClient同步实例：

```java
String publisherId = UUID.randomUUID().toString();
IMqttClient publisher = new MqttClient("tcp://iot.eclipse.org:1883",publisherId);
```

在这种情况下，**我们使用最简单的构造函数，它接收我们的MQTT代理的端点地址和客户端标识符，以唯一标识我们的客户端**。

在我们的例子中，我们使用了随机UUID，因此每次运行时都会生成一个新的客户端标识符。

Paho还提供了额外的构造函数，我们可以使用这些构造函数来自定义用于存储未确认消息的持久化机制和/或用于运行协议引擎实现所需的后台任务的ScheduledExecutorService。

**我们使用的服务器端点是由Paho项目托管的公共MQTT代理**，它允许任何拥有互联网连接的人测试客户端，而无需任何身份验证。

### 4.2 连接服务器

我们新创建的MqttClient实例尚未连接到服务器，**我们通过调用其connect()方法进行连接**，并可选择传递允许我们自定义协议某些方面的MqttConnectOptions实例。

具体来说，我们可以使用这些选项来传递附加信息，例如安全凭证、会话恢复模式、重新连接模式等。

MqttConnectionOptions类将这些选项公开为简单属性，我们可以使用常规设置方法进行设置。我们只需设置场景所需的属性-其余属性将采用默认值。

用于建立与服务器的连接的代码通常如下所示：

```java
MqttConnectOptions options = new MqttConnectOptions();
options.setAutomaticReconnect(true);
options.setCleanSession(true);
options.setConnectionTimeout(10);
publisher.connect(options);
```

在这里，我们定义连接选项以便：

- 如果发生网络故障，库将自动尝试重新连接服务器
- 它将丢弃上次运行中未发送的消息
- 连接超时设置为10秒

## 5. 发送消息

使用已连接的MqttClient发送消息非常简单，**我们使用其中一种publish()方法变体将有效负载(始终是字节数组)发送到给定主题**，并使用以下服务质量选项之一：

- 0：“最多一次”语义，也称为“即发即弃”。当消息丢失是可以接受的时，请使用此选项，因为它不需要任何类型的确认或持久化
- 1：“至少一次”语义，当消息丢失不可接受且订阅者可以处理重复消息时，请使用此选项
- 2：“恰好一次”语义，当消息丢失不可接收且订阅者无法处理重复消息时，请使用此选项

在我们的示例项目中，EngineTemperatureSensor类扮演模拟传感器的角色，每次调用其call()方法时都会产生新的温度读数。

该类实现了Callable接口，因此我们可以轻松地将它与java.util.concurrent包中提供的ExecutorService实现之一一起使用：

```java
public class EngineTemperatureSensor implements Callable<Void> {

    // ... private members omitted

    public EngineTemperatureSensor(IMqttClient client) {
        this.client = client;
    }

    @Override
    public Void call() throws Exception {
        if ( !client.isConnected()) {
            return null;
        }
        MqttMessage msg = readEngineTemp();
        msg.setQos(0);
        msg.setRetained(true);
        client.publish(TOPIC,msg);
        return null;
    }

    private MqttMessage readEngineTemp() {
        double temp =  80 + rnd.nextDouble() * 20.0;
        byte[] payload = String.format("T:%04.2f",temp)
                .getBytes();
        return new MqttMessage(payload);
    }
}
```

**MqttMessage封装了有效负载本身、请求的服务质量以及消息的retained标志**。此标志向代理指示它应保留此消息，直到被订阅者消费为止。

我们可以使用此功能来实现“最后已知的良好”行为，因此当新订阅者连接到服务器时，它将立即收到保留的消息。

## 6. 接收消息

为了从MQTT代理接收消息，我们需要使用其中一种subscribe()方法变体，它允许我们指定：

- 一个或多个用于接收消息的主题过滤器
- 相关的QoS
- 处理收到的消息的回调处理程序

在以下示例中，我们将展示如何向现有IMqttClient实例添加消息监听器以接收来自给定主题的消息。我们使用CountDownLatch作为回调和主执行线程之间的同步机制，每次有新消息到达时都会将其递减。

在示例代码中，我们使用了不同的IMqttClient实例来接收消息。我们这样做只是为了更清楚地说明哪个客户端执行什么操作，但这不是Paho的限制-如果你愿意，可以使用同一个客户端来发布和接收消息：

```java
CountDownLatch receivedSignal = new CountDownLatch(10);
subscriber.subscribe(EngineTemperatureSensor.TOPIC, (topic, msg) -> {
    byte[] payload = msg.getPayload();
    // ... payload handling omitted
    receivedSignal.countDown();
});    
receivedSignal.await(1, TimeUnit.MINUTES);
```

上面使用的subscribe()变体将IMqttMessageListener实例作为其第二个参数。

在我们的例子中，我们使用一个简单的Lambda函数来处理有效负载并递减计数器。如果在指定的时间窗口(1分钟)内没有足够的消息到达，await()方法将抛出异常。

使用Paho时，我们不需要明确确认收到消息。如果回调正常返回，Paho会认为消费成功并向服务器发送确认。

**如果回调抛出Exception，客户端将被关闭。请注意，这将导致丢失所有以QoS级别0发送的消息**。

一旦客户端重新连接并再次订阅该主题，服务器将重新发送以QoS级别1或2发送的消息。

## 7. 总结

在本文中，我们演示了如何使用Eclipse Paho项目提供的库在Java应用程序中添加对MQTT协议的支持。

该库处理所有低级协议细节，使我们能够专注于解决方案的其他方面，同时留下良好的空间来定制其内部功能的重要方面，例如消息持久化。