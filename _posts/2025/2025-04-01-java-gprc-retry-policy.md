---
layout: post
title:  配置gRPC请求的重试策略
category: rpc
copyright: rpc
excerpt: gRPC
---

## 1. 概述

在本教程中，我们将讨论在[gRPC](https://www.baeldung.com/grpc-introduction)(由Google开发的[远程过程调用](https://www.baeldung.com/cs/remote-vs-local-procedure-calls)框架)中实现重试策略的各种方法。gRPC可与多种编程语言互操作，但我们将重点介绍Java实现。

## 2. 重试的重要性

应用程序越来越依赖于分布式架构，这种方法有助于通过水平扩展处理繁重的工作负载，它还可以提高高可用性。然而，它也引入了更多的潜在故障点。因此，在开发具有多个微服务的应用程序时，[容错能力](https://www.baeldung.com/cs/high-availability-vs-fault-tolerance)至关重要。

**由于各种因素，RPC可能会暂时失败**：

- **网络延迟或连接中断**
- **由于内部错误，服务器未响应**
- **系统资源繁忙**
- **下游服务繁忙或不可用**
- **其他相关问题**

重试是一种故障处理机制，重试策略可以帮助根据某些条件自动重新尝试失败的请求。它还可以定义客户端可以重试多长时间或多久一次，这种简单的模式可以帮助处理瞬时故障并提高可靠性。

## 3. RPC失败阶段

首先让我们了解远程过程调用(RPC)可能失败的地方：

![](/assets/images/2025/rpc/javagprcretrypolicy01.png)

客户端应用程序发起请求，gRPC客户端库将该请求发送到服务器。收到请求后，gRPC服务器库将请求转发给服务器应用程序的逻辑。

**RPC可能会在各个阶段失败**：

1. **离开客户端前**
2. **在服务器中但在到达服务器应用程序逻辑之前**
3. **在服务器应用程序逻辑中**

## 4. gRPC中的重试支持

由于[重试](https://grpc.io/docs/guides/retry/)是一种重要的恢复机制，gRPC会在特殊情况下自动重试失败的请求，并允许开发人员定义重试策略以实现更好的控制。

### 4.1 透明重试

我们必须明白，只有在请求尚未到达应用服务器逻辑的情况下，gRPC才能安全地重新尝试失败的请求。除此之外，gRPC无法保证事务的幂等性。让我们来看看整体[透明重试路径](https://github.com/grpc/proposal/blob/master/A6-client-retries.md)：

![](/assets/images/2025/rpc/javagprcretrypolicy02.png)

**如前所述，内部重试可以在离开客户端或服务器之前但在到达服务器应用程序逻辑之前安全地进行，此重试策略称为透明重试**。一旦服务器应用程序成功处理请求，它就会返回响应并且不再尝试重试。

当RPC到达gRPC服务器库时，gRPC可以执行一次重试，因为多次重试会增加网络负载。但是，当RPC无法离开客户端时，它可能会无限次重试。

### 4.2 重试策略

为了让开发人员拥有更多控制权，gRPC支持在单个服务或方法级别为其应用程序配置适当的重试策略。一旦请求越过第2阶段，它就属于可配置重试策略的管辖范围。服务所有者或发布者可以借助[服务配置](https://grpc.io/docs/guides/service-config)(一个JSON文件)来配置其RPC的重试策略。

**服务所有者通常使用名称解析服务(例如DNS)将服务配置分发给gRPC客户端。但是，如果名称解析不提供服务配置，服务使用者或开发人员可以通过编程方式进行配置**。

gRPC支持多个重试参数：

|   配置名称    |                                            描述                                            |
|:---------:|:----------------------------------------------------------------------------------------:|
|  maxAttempts   |                               最大RPC尝试次数，包括原始请求<br/>默认最大值为5                               |
|   initialBackoff    |                                       重试之间的初始退避延迟                                        |
|   maxBackoff    |                             它对指数退避增长设置了上限这<br/>是强制性的，并且必须大于0                             |
|   backoffMultiplier    |                   每次重试后，退避值都会乘以该值，当乘数大于1时，退避值将呈指数增加<br/>这是强制性的，并且必须大于0                   |
|  retryableStatusCodes   | 如果gRPC调用失败且状态匹配，则将自动重试<br/>服务所有者在设计可重试的方法时应小心谨慎，方法应该是幂等的，或者只应在未对服务器进行任何更改的RPC的错误状态码上允许重试 |

值得注意的是，gRPC客户端使用initialBackoff、maxBackoff和backoffMultiplier参数在重试请求之前随机化延迟。

有时，服务器可能会在响应元数据中发送一条指令，不要重试或在一段时间延迟后尝试请求。这称为服务器回推。

现在我们已经讨论了gRPC的透明和基于策略的重试功能，让我们总结一下[gRPC如何整体管理重试](https://github.com/grpc/proposal/blob/master/A6-client-retries.md)：

![](/assets/images/2025/rpc/javagprcretrypolicy03.png)

## 5. 以编程方式应用重试策略

**假设我们有一项服务，可以通过调用向手机发送短信的底层通知服务向公民广播消息，政府使用此服务发布紧急情况公告**。使用此服务的客户端应用程序必须具有重试策略，以减轻由于瞬时故障导致的错误。

让我们进一步探讨这一点。

### 5.1 高层设计

首先我们看一下broadcast.proto文件中的接口定义：

```protobuf
syntax = "proto3";
option java_multiple_files = true;
option java_package = "cn.tuyucheng.taketoday.grpc.retry";
package retryexample;

message NotificationRequest {
    string message = 1;
    string type = 2;
    int32 messageID = 3;
}

message NotificationResponse {
    string response = 1;
}

service NotificationService {
    rpc notify(NotificationRequest) returns (NotificationResponse){}
}
```

**broadcast.proto文件定义了NotificationService，它有一个远程方法notify()和两个DTO，分别是NotificationRequest和NotificationResponse**。

总的来说，让我们看看gRPC应用程序的客户端和服务器端使用的类：

![](/assets/images/2025/rpc/javagprcretrypolicy04.png)

稍后，我们可以使用broadcast.proto文件生成实现NotificationService的支持Java源代码，Maven插件会生成类NotificationRequest、NotificationResponse和NotificationServiceGrpc。

服务端的GrpcBroadcastingServer类使用ServerBuilder类注册NotificationServiceImpl来广播消息，**客户端的GrpcBroadcastingClient类使用gRPC库的[ManagedChannel](https://grpc.github.io/grpc-java/javadoc/io/grpc/ManagedChannel.html)类来管理通道，执行RPC操作**。

服务配置文件retry-service-config.json概述了重试策略：

```json
{
     "methodConfig": [
         {
             "name": [
                 {
                      "service": "retryexample.NotificationService",
                      "method": "notify"
                 }
              ],
             "retryPolicy": {
                 "maxAttempts": 5,
                 "initialBackoff": "0.5s",
                 "maxBackoff": "30s",
                 "backoffMultiplier": 2,
                 "retryableStatusCodes": [
                     "UNAVAILABLE"
                 ]
             }
         }
     ]
}
```

前面我们了解了重试策略，例如maxAttempts、指数退避参数和retryableStatusCodes。当客户端调用前面在broadcast.proto文件中定义的NotificationService中的远程过程notify()时，gRPC框架会强制执行重试设置。

### 5.2 实施重试策略

让我们看一下GrpcBroadcastingClient类：

```java
public class GrpcBroadcastingClient {
    protected static Map<String, ?> getServiceConfig() {
        return new Gson().fromJson(new JsonReader(new InputStreamReader(GrpcBroadcastingClient.class.getClassLoader()
                .getResourceAsStream("retry-service-config.json"), StandardCharsets.UTF_8)), Map.class);
    }

    public static NotificationResponse broadcastMessage() {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8080)
                .usePlaintext()
                .disableServiceConfigLookUp()
                .defaultServiceConfig(getServiceConfig())
                .enableRetry()
                .build();
        return sendNotification(channel);
    }

    public static NotificationResponse sendNotification(ManagedChannel channel) {
        NotificationServiceGrpc.NotificationServiceBlockingStub notificationServiceStub = NotificationServiceGrpc
                .newBlockingStub(channel);

        NotificationResponse response = notificationServiceStub.notify(NotificationRequest.newBuilder()
                .setType("Warning")
                .setMessage("Heavy rains expected")
                .setMessageID(generateMessageID())
                .build());
        channel.shutdown();
        return response;
    }
}
```

broadcast()方法使用必要的配置构建ManagedChannel对象。然后，我们将其传递给sendNotification()，后者进一步调用存根上的notify()方法。

**[ManagedChannelBuilder](https://grpc.github.io/grpc-java/javadoc/io/grpc/ManagedChannelBuilder.html)类中在设置由重试策略组成的服务配置中起着至关重要的作用的方法包括**：

- **disableServiceConfigLookup()：明确禁用通过名称解析进行的服务配置查找**
- **enableRetry()：启用每个方法的重试配置**
- **defaultServiceConfig()：明确设置服务配置**

getServiceConfig()方法从retry-service-config.json文件中读取服务配置并返回其内容的Map表示，随后，此Map被传递给ManagedChannelBuilder类中的defaultServiceConfig()方法。

最后，在创建ManagedChannel对象后，我们调用类型为NotificationServiceGrpc.NotificationServiceBlockingStub的notificationServiceStub对象的notify()方法来广播消息，此策略也适用于非阻塞存根。

**建议使用专用类来创建ManagedChannel对象，这样可以进行集中管理，包括配置重试策略**。

为了演示重试功能，服务器中的NotificationServiceImpl类设计为随机停止服务。让我们来看看GrpcBroadcastingClient的实际操作：

```java
@Test
void whenMessageBroadCasting_thenSuccessOrThrowsStatusRuntimeException() {
    try {
        NotificationResponse notificationResponse = GrpcBroadcastingClient.sendNotification(managedChannel);
        assertEquals("Message received: Warning - Heavy rains expected", notificationResponse.getResponse());
    } catch (Exception ex) {
        assertTrue(ex instanceof StatusRuntimeException);
    }
}
```

**该方法调用GrpcBroadcastingClient类上的sendNotification()来调用服务器端远程过程来广播消息，我们可以检查日志来验证重试**：

![](/assets/images/2025/rpc/javagprcretrypolicy04.png)

## 6. 总结

在本文中，我们探索了gRPC库中的重试策略功能。通过JSON文件以声明方式设置策略的能力是一项强大的功能，但是，我们应该将其用于测试场景，或者在名称解析期间服务配置不可用时使用它。

**重试失败的请求可能会导致不可预测的结果，因此我们在设置幂等事务时应小心谨慎**。