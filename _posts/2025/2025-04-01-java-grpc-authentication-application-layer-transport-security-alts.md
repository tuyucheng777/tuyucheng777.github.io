---
layout: post
title:  使用ALTS在Java中进行gRPC身份验证
category: rpc
copyright: rpc
excerpt: gRPC
---

## 1. 概述

在本教程中，我们将探讨[ALTS(应用层传输安全性)](https://cloud.google.com/docs/security/encryption-in-transit/application-layer-transport-security)在[gRPC](https://www.baeldung.com/grpc-introduction)应用程序中的作用。众所周知，在分布式架构中确保身份验证和数据安全很困难，但至关重要。

ALTS是Google定制的内置相互身份验证和传输加密解决方案，仅在Google的云基础架构中提供。ALTS简化了gRPC服务之间的身份验证和数据加密，只需进行少量代码更改即可启用。因此，它在开发人员中很受欢迎，因为他们可以更专注于编写业务逻辑。

## 2. ALTS和TLS之间的主要区别

**[ALTS与TLS](https://www.baeldung.com/cs/ssl-vs-tls)类似，但具有针对Google基础架构优化的不同信任模型**，让我们快速看一下它们之间的主要区别：

|  特征   |                    ALTS                     |                         TLS                         |
|:-----:|:-------------------------------------------:|:---------------------------------------------------:|
| 信任模型  |             基于身份的依赖GCP IAM服务帐户              |                 基于证书，需要证书管理，包括续订和撤销                 |
|  设计   |                     更简单                     |                         复杂                          |
| 使用上下文 |          用于保护在Google数据中心运行的gRPC服务           |         用于保护网页浏览(HTTPS)、电子邮件、即时通讯、VoIP等的安全          |
| 消息序列化 |              使用Protocol Buffer              |                  使用ASN.1编码的X.509证书                  |
|  性能   |                  设计用于一般用途                   |            针对Google数据中心的低延迟、高吞吐量通信进行了优化             |

## 3. 使用ALTS的示例应用程序

**ALTS功能在Google Cloud Platform(GCP)上默认启用，它使用[GCP服务帐号](https://cloud.google.com/iam/docs/service-account-overview)来保护gRPC服务之间的RPC调用。具体来说，它在Google基础架构内的[Google Compute Engine](https://cloud.google.com/products/compute)或[Kubernetes Engine(GKE)](https://cloud.google.com/kubernetes-engine)上运行**。

假设一家医院有一个手术室(OT)预订系统，由前端和后端服务组成：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts01.png)

OT预订系统包括在Google Cloud Platform(GCP)中运行的两项服务，前端服务对后端服务进行远程过程调用，我们将使用gRPC框架开发服务。考虑到数据的敏感性，我们将利用GCP中的内置ALTS功能对传输数据进行身份验证和加密。

首先我们定义[protobuf](https://www.baeldung.com/google-protocol-buffer) ot_booking.proto文件：

```protobuf
syntax = "proto3";

package otbooking;

option java_multiple_files = true;
option java_package = "cn.tuyucheng.taketoday.grpc.alts.otbooking";

service OtBookingService {
    rpc getBookingInfo(BookingRequest) returns (BookingResponse) {}
}

message BookingRequest {
    string patientID = 1;
    string doctorID = 2;
    string description = 3;
}

message BookingResponse {
    string bookingDate = 1;
    string condition = 2;
}
```

基本上，我们在protobuf文件中使用RPC getBookingInfo()声明了一个服务OtBookingService，以及两个DTO BookingRequest和BookingResponse。

接下来我们看一下这个应用程序的重要类：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts02.png)

[Maven](https://www.baeldung.com/maven)插件编译protobuf文件并自动生成一些类，例如OtBookingServiceGrpc、OtBookingServiceImplBase、BookingRequest和BookingResponse，**我们将使用gRPC库类[AltsChannelBuilder](https://javadoc.io/doc/io.grpc/grpc-alts/latest/io/grpc/alts/AltsChannelBuilder.html)来启用ALTS在客户端创建[ManagedChannel](https://grpc.github.io/grpc-java/javadoc/io/grpc/ManagedChannel.html)对象**。最后，我们将使用OtBookingServiceGrpc生成OtBookingServiceBlockingStub来调用在服务器端运行的RPC getBookingInfo()方法。

**与AltsChannelBuilder一样，AltsServerBuilder类帮助在服务器端启用ALTS。我们注册拦截器[ClientAuthInterceptor](https://javadoc.io/doc/io.grpc/grpc-alts/latest/io/grpc/alts/AltsServerBuilder.html)来帮助验证客户端**。最后，我们将OtBookingService注册到[io.grpc.Server](https://grpc.github.io/grpc-java/javadoc/io/grpc/Server.html)对象，然后启动该服务。

此外，我们将在下一节讨论具体实现方式。

## 4. 使用ALTS实现应用程序

让我们实现之前讨论过的类。然后，我们将通过在GCP虚拟机上运行服务来进行演示。

### 4.1 先决条件

由于ALTS是GCP的内置功能，我们必须配置一些云资源来运行示例应用程序。

首先，我们将创建两个IAM服务账户，分别与前端和后端服务器关联：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts03.png)

然后，我们将创建两个分别托管前端和后端服务的虚拟机：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts04.png)

虚拟机prod-booking-client-vm与prod-ot-booking-client-svc服务帐户相关联。同样，prod-booking-service-vm与prod-ot-booking-svc服务帐户相关联。服务帐户充当服务器的身份，ALTS使用它们进行授权和加密。

### 4.2 实现

让我们首先从pom.xml文件开始，添加[Maven依赖](https://mvnrepository.com/artifact/io.grpc/grpc-alts/)：

```xml
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-alts</artifactId>
    <version>1.63.0</version>
</dependency>
```

然后，我们将实现后端，从AltsBookingServer类开始：

```java
public class AltsOtBookingServer {
    public static void main(String[] args) throws IOException, InterruptedException {
        final String CLIENT_SERVICE_ACCOUNT = args[0];
        Server server = AltsServerBuilder.forPort(8080)
                .intercept(new ClientAuthInterceptor(CLIENT_SERVICE_ACCOUNT))
                .addService(new OtBookingService())
                .build();
        server.start();
        server.awaitTermination();
    }
}
```

gRPC提供了一个特殊的类AltsServerBuilder，用于在ALTS模式下配置服务器。我们在服务器上注册了ClientAuthInterceptor，以便在所有RPC到达OtBookingService类中的端点之前拦截它们。

我们来看一下ClientAuthInterceptor类：

```java
public class ClientAuthInterceptor implements ServerInterceptor {
    String clientServiceAccount = null;
    public ClientAuthInterceptor(String clientServiceAccount) {
        this.clientServiceAccount = clientServiceAccount;
    }

    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> serverCall, Metadata metadata,
                                                                 ServerCallHandler<ReqT, RespT> serverCallHandler) {
        Status status = AuthorizationUtil.clientAuthorizationCheck(serverCall,
                Lists.newArrayList(this.clientServiceAccount));
        if (!status.isOk()) {
            serverCall.close(status, new Metadata());
        }
        return serverCallHandler.startCall(serverCall, metadata);
    }
}
```

所有的RPC都会命中ClientAuthInterceptor中的Intercept()方法。然后，我们调用gRPC库类AuthorizationUtil中的clientAuthorizationCheck()方法来授权客户端服务账户。最后，只有当授权成功时，RPC才会继续进行。

接下来我们看一下前端服务：

```java
public class AltsOtBookingClient {
    public static void main(String[] args) {
        final String SERVER_ADDRESS = args[0];
        final String SERVER_ADDRESS_SERVICE_ACCOUNT = args[1];
        ManagedChannel managedChannel = AltsChannelBuilder.forTarget(SERVER_ADDRESS)
                .addTargetServiceAccount(SERVER_ADDRESS_SERVICE_ACCOUNT)
                .build();
        OtBookingServiceGrpc.OtBookingServiceBlockingStub OTBookingServiceStub = OtBookingServiceGrpc
                .newBlockingStub(managedChannel);
        BookingResponse bookingResponse = OTBookingServiceStub.getBookingInfo(BookingRequest.newBuilder()
                .setPatientID("PT-1204")
                .setDoctorID("DC-3904")
                .build());
        managedChannel.shutdown();
    }
}
```

与AltsServerBuilder类似，gRPC提供了一个AltsChannelBuilder类，用于在客户端启用ALTS。我们可以多次调用addTargetServiceAccount()方法来添加多个潜在目标服务帐户。此外，我们通过调用存根上的getBookingInfo()方法来启动RPC。

**同一个服务账户可以关联多个虚拟机，为服务水平扩展提供了一定的灵活性和敏捷性**。

### 4.3 在Google Compute Engine上运行

让我们登录两台服务器，然后克隆托管演示gRPC服务源代码的GitHub仓库：

```shell
git clone https://github.com/tuyucheng777/taketoday-tutorial4j.git
```

克隆后，我们将编译taketoday-tutorial4j/grpc目录中的代码：

```shell
mvn clean compile
```

编译成功后，我们将在prod-booking-service-vm中启动后端服务：

```shell
mvn exec: java -Dexec.mainClass="cn.tuyucheng.taketoday.grpc.alts.server.AltsOtBookingServer" \
-Dexec.arguments="prod-ot-booking-client-svc@grpc-alts-demo.iam.gserviceaccount.com"
```

我们使用前端客户端的服务帐户作为参数运行了AltsOtBookingServer类。

一旦服务启动并运行，我们将从虚拟机prod-booking-client-vm上运行的前端服务启动RPC：

```shell
mvn exec:java -Dexec.mainClass="cn.tuyucheng.taketoday.grpc.alts.client.AltsOtBookingClient" \
-Dexec.arguments="10.128.0.2:8080,prod-ot-booking-svc@grpc-alts-demo.iam.gserviceaccount.com"
```

我们用两个参数运行了AltsOtBookingClient类，第一个参数是运行后端服务的目标服务器，第二个参数是与后端服务器关联的服务帐户。

命令成功运行，服务在对客户端进行身份验证后返回响应：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts05.png)

假设我们禁用客户端服务帐户：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts06.png)

因此，ALTS阻止RPC到达后端服务：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts07.png)

RPC失败，状态为UNAVAILABLE。

现在，让我们禁用后端服务器的服务账户：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts08.png)

令人惊讶的是，RPC成功通过，但重新启动服务器后，它像之前的情况一样失败了：

![](/assets/images/2025/rpc/javagrpcauthenticationapplicationlayertransportsecurityalts09.png)

**看起来ALTS之前一直在缓存服务账户状态，但是在服务器重启后，RPC失败并显示状态UNKNOWN**。

## 5. 总结

在本文中，我们深入研究了支持ALTS的gRPC Java库。只需极少的代码，即可在gRPC服务中启用ALTS。借助GCP IAM服务帐户，它还可以更灵活地控制gRPC服务的授权。

**但是，由于它是开箱即用的，因此它只能在GCP基础架构中使用。因此，要在GCP基础架构之外运行gRPC服务，gRPC中的TLS支持至关重要，并且必须手动配置**。