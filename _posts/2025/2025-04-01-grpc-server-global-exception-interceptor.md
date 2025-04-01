---
layout: post
title:  在gRPC服务器中添加全局异常拦截器
category: rpc
copyright: rpc
excerpt: gRPC
---

## 1. 概述

在本教程中，我们将研究[拦截器](https://grpc.io/docs/guides/interceptors/)在[gRPC](https://www.baeldung.com/grpc-introduction)服务器应用程序中处理全局异常的作用。

**拦截器可以在请求到达RPC方法之前对其进行验证或操作**，因此，它们在处理应用程序的常见问题(如日志记录、安全、缓存、审计、身份验证和授权等)方面非常有用。

应用程序还可以使用拦截器作为全局异常处理程序。

## 2. 拦截器作为全局异常处理程序

**主要地，拦截器可以帮助处理两种类型的异常**：

- **处理无法处理的方法中逃逸的未知运行时异常**
- **处理从任何其他下游拦截器中逃逸的异常**

拦截器可以帮助创建一个以集中方式处理异常的框架，这样，应用程序就可以拥有一致的标准和强大的方法来处理异常。

他们可以用各种方式处理异常：

- 记录或保留异常以用于审计或报告目的
- 创建支持票
- 在将错误响应发送回客户端之前，对其进行修改或丰富

## 3. 全局异常处理程序的高层设计

**拦截器可以将传入的请求转发到目标RPC服务。但是，当目标RPC方法抛出异常时，它可以捕获该异常，然后进行适当的处理**。

假设有一个订单处理微服务，我们将在拦截器的帮助下开发一个全局异常处理程序，以捕获微服务中RPC方法逃逸的异常。此外，拦截器还会捕获任何下游拦截器逃逸的异常。然后，它调用票务服务在票务系统中开具票证。最后，将响应发送回客户端。

我们来看一下当请求在RPC端点失败时，请求的遍历路径：

![](/assets/images/2025/rpc/grpcserverglobalexceptioninterceptor01.png)

同样的我们在日志拦截器中看一下请求失败的时候，请求的遍历路径：

![](/assets/images/2025/rpc/grpcserverglobalexceptioninterceptor02.png)

首先，我们将开始在[protobuf](https://protobuf.dev/)文件order_processing.proto中定义订单处理服务的基类：

```protobuf
syntax = "proto3";

package orderprocessing;

option java_multiple_files = true;
option java_package = "cn.tuyucheng.taketoday.grpc.orderprocessing";

message OrderRequest {
    string product = 1;
    int32 quantity = 2;
    float price = 3;
}
message OrderResponse {
    string response = 1;
    string orderID = 2;
    string error = 3;
}
service OrderProcessor {
    rpc createOrder(OrderRequest) returns (OrderResponse){}
}
```

**order_processing.proto文件定义了OrderProcessor，它有一个远程方法createOrder()以及两个DTO：OrderRequest和OrderResponse**。

让我们看一下在接下来的部分中将要实现的主要类：

![](/assets/images/2025/rpc/grpcserverglobalexceptioninterceptor03.png)

稍后，我们可以使用order_processing.proto文件生成实现OrderProcessorImpl和GlobalExceptionInterceptor的支持Java源代码，Maven[插件](https://www.baeldung.com/maven-guide)生成OrderRequest、OrderResponse和OrderProcessorGrpc类。

我们将在实现部分讨论每个类。

## 4. 实现

我们将实现一个可以处理各种异常的拦截器。异常可能由于某些失败的逻辑而明确引发，也可能是由于某些不可预见的错误而引发的异常。

### 4.1 实现全局异常处理程序

**gRPC应用程序中的拦截器必须实现[ServerInterceptor](https://grpc.github.io/grpc-java/javadoc/io/grpc/ServerInterceptor.html)接口的InterceptCall()方法**：

```java
public class GlobalExceptionInterceptor implements ServerInterceptor {
    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> serverCall, Metadata headers,
                                                                 ServerCallHandler<ReqT, RespT> next) {
        ServerCall.Listener<ReqT> delegate = null;
        try {
            delegate = next.startCall(serverCall, headers);
        } catch(Exception ex) {
            return handleInterceptorException(ex, serverCall);
        }
        return new ForwardingServerCallListener.SimpleForwardingServerCallListener<ReqT>(delegate) {
            @Override
            public void onHalfClose() {
                try {
                    super.onHalfClose();
                } catch (Exception ex) {
                    handleEndpointException(ex, serverCall);
                }
            }
        };
    }

    private static <ReqT, RespT> void handleEndpointException(Exception ex, ServerCall<ReqT, RespT> serverCall) {
        String ticket = new TicketService().createTicket(ex.getMessage());
        serverCall.close(Status.INTERNAL
                .withCause(ex)
                .withDescription(ex.getMessage() + ", Ticket raised:" + ticket), new Metadata());
    }

    private <ReqT, RespT> ServerCall.Listener<ReqT> handleInterceptorException(Throwable t, ServerCall<ReqT, RespT> serverCall) {
        String ticket = new TicketService().createTicket(t.getMessage());
        serverCall.close(Status.INTERNAL
                .withCause(t)
                .withDescription("An exception occurred in a **subsequent** interceptor:" + ", Ticket raised:" + ticket), new Metadata());

        return new ServerCall.Listener<ReqT>() {
            // no-op
        };
    }
}
```

方法interceptCall()接收3个输入参数：

- [ServerCall](https://grpc.github.io/grpc-java/javadoc/io/grpc/ServerCall.html)：帮助接收响应消息
- [Metadata](https://grpc.github.io/grpc-java/javadoc/io/grpc/Metadata.html)：保存传入请求的元数据
- [ServerCallHandler](https://grpc.github.io/grpc-java/javadoc/io/grpc/ServerCallHandler.html)：帮助将传入的服务器调用分派到拦截器链中的下一个处理器

**该方法有两个[try–catch](https://www.baeldung.com/java-exceptions)块，第一个处理任何后续下游拦截器抛出的未捕获异常**。在catch块中，我们调用方法handleInterceptorException()，该方法为异常创建一张票。最后，它返回一个ServerCall.Listener对象，这是一个回调方法。

**类似地，第二个try–catch块处理从RPC端点抛出的未捕获异常**。interceptCall()方法返回ServerCall.Listener，它充当传入RPC消息的回调。具体来说，它返回[ForwardingServerCallListener.SimpleForwardingServerCallListener](https://grpc.github.io/grpc-java/javadoc/io/grpc/ForwardingServerCallListener.SimpleForwardingServerCallListener.html)的实例，是[ServerCall.Listener](https://grpc.github.io/grpc-java/javadoc/io/grpc/ServerCall.Listener.html)的子类。

**为了处理下游方法抛出的异常，我们重写了ForwardingServerCallListener.SimpleForwardingServerCallListener类中的onHalfClose()方法。客户端完成消息发送后，该方法就会被调用**。

在此方法中，super.onHalfClose()将请求转发到OrderProcessorImpl类中的RPC端点createOrder()。如果端点中存在未捕获的异常，我们将捕获该异常，然后调用handleEndpointException()来创建票据。最后，我们调用serverCall对象上的close()方法来关闭服务器调用并将响应发送回客户端。

### 4.2 注册全局异常处理程序

我们在启动过程中创建io.grpc.Server对象时注册拦截器：

```java
public class OrderProcessingServer {
    public static void main(String[] args) throws IOException, InterruptedException {
        Server server = ServerBuilder.forPort(8080)
                .addService(new OrderProcessorImpl())
                .intercept(new LogInterceptor())
                .intercept(new GlobalExceptionInterceptor())
                .build();
        server.start();
        server.awaitTermination();
    }
}
```

**我们将GlobalExceptionInterceptor对象传递给[io.grpc.ServerBuilder](https://grpc.github.io/grpc-java/javadoc/io/grpc/ServerBuilder.html)类的Intercept()方法**，这可确保对OrderProcessorImpl服务的任何RPC调用都通过GlobalExceptionInterceptor。同样，我们调用addService()方法来注册OrderProcessorImpl服务。最后，我们调用Server对象上的start()方法来启动服务器应用程序。

### 4.3 处理端点未捕获的异常

为了演示异常处理程序，我们首先看一下OrderProcessorImpl类：

```java
public class OrderProcessorImpl extends OrderProcessorGrpc.OrderProcessorImplBase {
    @Override
    public void createOrder(OrderRequest request, StreamObserver<OrderResponse> responseObserver) {
        if (!validateOrder(request)) {
            throw new StatusRuntimeException(Status.FAILED_PRECONDITION.withDescription("Order Validation failed"));
        } else {
            OrderResponse orderResponse = processOrder(request);

            responseObserver.onNext(orderResponse);
            responseObserver.onCompleted();
        }
    }

    private Boolean validateOrder(OrderRequest request) {
        int tax = 100 / 0;
        return false;
    }

    private OrderResponse processOrder(OrderRequest request) {
        return OrderResponse.newBuilder()
                .setOrderID("ORD-5566")
                .setResponse("Order placed successfully")
                .build();
    }
}
```

**RPC方法createOrder()首先验证订单，然后通过调用processOrder()方法来处理订单。在validateOrder()方法中，我们故意通过将数字除以0来强制产生运行时异常**。

现在，让我们运行该服务并看看它如何处理异常：

```java
@Test
void whenRuntimeExceptionInRPCEndpoint_thenHandleException() {
    OrderRequest orderRequest = OrderRequest.newBuilder()
            .setProduct("PRD-7788")
            .setQuantity(1)
            .setPrice(5000)
            .build();

    try {
        OrderResponse response = orderProcessorBlockingStub.createOrder(orderRequest);
    } catch (StatusRuntimeException ex) {
        assertTrue(ex.getStatus()
                .getDescription()
                .contains("Ticket raised:TKT"));
    }
}
```

我们创建OrderRequest对象，然后将其传递给客户端存根中的createOrder()方法。正如预期的那样，服务会抛出异常。当我们检查异常中的描述时，我们会发现其中嵌入了票据信息。因此，这表明GlobalExceptionInterceptor完成了它的工作。

这对于[流处理](https://www.baeldung.com/java-grpc-streaming)案例同样有效。

### 4.4 处理拦截器未捕获的异常

假设在GlobalExceptionInterceptor之后调用了第二个拦截器，LogInterceptor会记录所有传入的请求以供审计，我们来看看：

```java
public class LogInterceptor implements ServerInterceptor {
    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> serverCall, Metadata metadata,
                                                                 ServerCallHandler<ReqT, RespT> next) {
        logMessage(serverCall);
        ServerCall.Listener<ReqT> delegate = next.startCall(serverCall, metadata);
        return delegate;
    }

    private <ReqT, RespT> void logMessage(ServerCall<ReqT, RespT> call) {
        int result = 100 / 0;
    }
}
```

在LogInterceptor中，interceptCall()方法在将请求转发到RPC端点之前调用logMessage()来记录消息。**logMessage()方法故意执行除以0的操作以引发运行时异常，以展示GlobalExceptionInterceptor的功能**。

让我们运行该服务并看看它如何处理从LogInterceptor引发的异常：

```java
@Test
void whenRuntimeExceptionInLogInterceptor_thenHandleException() {
    OrderRequest orderRequest = OrderRequest.newBuilder()
            .setProduct("PRD-7788")
            .setQuantity(1)
            .setPrice(5000)
            .build();

    try {
        OrderResponse response = orderProcessorBlockingStub.createOrder(orderRequest);
    } catch (StatusRuntimeException ex) {
        assertTrue(ex.getStatus()
                .getDescription()
                .contains("An exception occurred in a **subsequent** interceptor:, Ticket raised:TKT"));
    }
    logger.info("order processing over");
}
```

首先，我们在客户端存根上调用createOrder()方法。这一次，GlobalExceptionInterceptor捕获了在第一个try–catch块中从LogInterceptor中逃逸的异常。随后，客户端接收异常，并在描述中嵌入票证信息。

## 5. 总结

在本文中，我们探讨了拦截器在gRPC框架中作为全局异常处理程序的作用。**它们是处理常见异常问题(例如日志记录、创建工单、丰富错误响应等)的绝佳工具**。