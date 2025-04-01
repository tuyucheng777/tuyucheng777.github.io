---
layout: post
title:  Java Protobuf中打包重复字段
category: rpc
copyright: rpc
excerpt: gRPC
---

## 1. 概述

在本教程中，我们将讨论Google的[Protocol Buffer(protobuf)](https://www.baeldung.com/google-protocol-buffer)消息中的打包重复字段。Protocol Buffer有助于定义高度优化的语言中立和平台中立的数据结构，以实现极其高效的序列化。在protobuf中，repeated关键字有助于定义可以容纳多个值的字段。

**此外，为了在重复字段的序列化过程中实现更高的优化，protobuf引入了新的选项packed，它采用一种特殊的编码技术，进一步减小消息的大小**。

让我们对此进行进一步探讨。

## 2. 重复字段

在讨论repeated字段的packed选项之前，让我们先了解一下标签repeat的含义。让我们考虑一个proto文件repeat.proto：

```protobuf
syntax = "proto3";
option java_multiple_files = true;
option java_package = "cn.tuyucheng.taketoday.grpc.repeated";
package repeated;

message PackedOrder {
    int32 orderId = 1;
    repeated int32 productIds = 2 [packed = true];
}

message UnpackedOrder {
    int32 orderId = 1;
    repeated int32 productIds = 2 [packed = false];
}

service OrderService {
    rpc createOrder(UnpackedOrder) returns (UnpackedOrder){}
}
```

该文件定义了两种消息类型(DTO) PackedOrder和UnpackedOrder，以及一个名为OrderService的服务。**productIds字段上的repeat标签强调它可以具有多个整型值，类似于集合或数组**。从protobuf v2.1.0开始，repeated字段的packed选项默认为true。因此，为了禁用packed行为，我们现在明确使用选项packed = false来专注于repeat功能。

有趣的是，如果我们修改repeated字段并添加packed = true选项，我们不需要调整代码即可使其工作。唯一的区别是内部gRPC库在序列化过程中如何对字段进行编码，我们将在后面的部分中讨论这个问题。

让我们定义具有RPC createOrder()的OrderService：

```java
public class OrderService extends OrderServiceGrpc.OrderServiceImplBase {
    @Override
    public void createOrder(UnpackedOrder unpackedOrder, StreamObserver<UnpackedOrder> responseObserver) {
        List productIds = unpackedOrder.getProductIdsList();
        if(validateProducts(productIds)) {
            int orderID = insertOrder(unpackedOrder);
            UnpackedOrder createdUnpackedOrder = UnpackedOrder.newBuilder(unpackedOrder)
                    .setOrderId(orderID)
                    .build();
            responseObserver.onNext(createdUnpackedOrder);
            responseObserver.onCompleted();
        }
    }
}
```

**protoc Maven插件自动生成方法getProductIdsList()，用于获取repeated字段中的元素列表。无论字段是打包的还是未打包的，此方法均适用**。最后，我们在UnpackedOrder对象中设置生成的orderID，并将其返回给客户端。

现在让我们调用RPC：

```java
@Test
void whenUnpackedRepeatedProductIds_thenCreateUnpackedOrderAndInvokeRPC() {
    UnpackedOrder.Builder unpackedOrderBuilder = UnpackedOrder.newBuilder();
    unpackedOrderBuilder.setOrderId(1);
    Arrays.stream(fetchProductIds()).forEach(unpackedOrderBuilder::addProductIds);
    UnpackedOrder unpackedOrderRequest = unpackedOrderBuilder.build();
    UnpackedOrder unpackedOrderResponse = orderClientStub.createOrder(unpackedOrderRequest);
    assertInstanceOf(Integer.class, unpackedOrderResponse.getOrderId());
}
```

当我们使用protoc Maven插件编译代码时，它会为proto文件中定义的UnpackedOrder消息类型生成Java类文件。我们在遍历Stream时多次调用方法addProductIds()来填充UnpackedOrder对象中的repeated字段productIds。**通常，在编译proto文件期间，会为所有repeated字段名称创建一个类似的方法，并以文本add为前缀。这适用于所有repeated字段，无论是packed还是unpacked**。

此后，我们调用RPC createOrder()返回字段orderId。

## 3. 打包重复字段

到目前为止，我们知道打包重复字段与重复字段的主要区别在于序列化之前的编码过程。要理解编码技术，让我们首先看看如何序列化proto文件中定义的PackedOrder和UnpackedOrder消息类型：

```java
void serializeObject(String file, GeneratedMessageV3 object) throws IOException {
    try(FileOutputStream fileOutputStream = new FileOutputStream(file)) {
        object.writeTo(fileOutputStream);
    }
}
```

方法serializeObject()调用[GeneratedMessageV3类型对象中的writeTo()](https://javadoc.io/doc/com.google.protobuf/protobuf-java/3.25.3/com/google/protobuf/GeneratedMessageV3.html)方法将其序列化到文件系统。

PackedOrder和UnpackedOrder消息类型从其父类GeneratedMessageV3继承writeTo()方法。因此，我们将使用serializeObject()方法将其实例写入文件系统：

```java
@Test
void whenSerializeUnpackedOrderAndPackedOrderObject_thenSizeofPackedOrderObjectIsLess() throws IOException {
    UnpackedOrder.Builder unpackedOrderBuilder = UnpackedOrder.newBuilder();
    unpackedOrderBuilder.setOrderId(1);
    Arrays.stream(fetchProductIds()).forEach(unpackedOrderBuilder::addProductIds);
    UnpackedOrder unpackedOrder = unpackedOrderBuilder.build();
    String unpackedOrderObjFileName = FOLDER_TO_WRITE_OBJECTS + "unpacked_order.bin";

    serializeObject(unpackedOrderObjFileName, unpackedOrder);

    PackedOrder.Builder packedOrderBuilder = PackedOrder.newBuilder();
    packedOrderBuilder.setOrderId(1);
    Arrays.stream(fetchProductIds()).forEach(packedOrderBuilder::addProductIds);
    PackedOrder packedOrder = packedOrderBuilder.build();
    String packedOrderObjFileName = FOLDER_TO_WRITE_OBJECTS + "packed_order.bin";

    serializeObject(packedOrderObjFileName, packedOrder);

    long sizeOfUnpackedOrderObjectFile = getFileSize(unpackedOrderObjFileName);
    long sizeOfPackedOrderObjectFile = getFileSize(packedOrderObjFileName);
    long sizeReductionPercentage = (sizeOfUnpackedOrderObjectFile - sizeOfPackedOrderObjectFile) * 100 / sizeOfUnpackedOrderObjectFile;
    logger.info("Packed field saved {}% over unpacked field", sizeReductionPercentage);
    assertTrue(sizeOfUnpackedOrderObjectFile > sizeOfPackedOrderObjectFile);
}
```

首先，我们通过向每个对象添加相同的产品ID集来创建unpackedOrder和packedOrder对象。然后，我们对这两个对象进行序列化并比较它们的文件大小。该程序还使用打包版本的productID来计算对象中文件大小的减少百分比。**正如预期的那样，包含unpackedOrder对象的文件大于包含packedOrder对象的文件**。

现在让我们看看程序的控制台输出：

```shell
Packed field saved 29% over unpacked field
```

**此示例包含20个产品ID，表明packedOrder对象的文件大小减少了29%。此外，随着产品ID的增加，节省量会不断增加并最终趋于稳定**。

当然，打包repeated字段会带来更好的性能。但是，我们只能对原始数字类型使用packed选项。

## 4. 编码后的解包字段与打包字段

之前，我们创建了两个文件unpacked_order.bin和packed_order.bin，分别对应UnpackedOrder和PackedOrder对象。我们将使用[protoscope工具](https://github.com/protocolbuffers/protoscope)检查这两个文件的编码内容。**Protoscope是一种简单、人性化且可编辑的语言，可帮助我们查看传输中消息的低级Protobuf有线格式**。

让我们检查一下unpacked_order.bin的内容：

```text
#cat unpacked_order.bin | protoscope -explicit-wire-types
1:VARINT 1
2:VARINT 266
2:VARINT 629
2:VARINT 725
2:VARINT 259
2:VARINT 353
2:VARINT 746
more elements...
```

protoscope命令将编码的Protocol Buffer转储为文本。在文本中，字段及其值以键值格式表示，其中键是repeat.proto文件中定义的字段编号。**键为2的productId字段重复出现，其每个值均表示为VARINT[有线格式](https://protobuf.dev/programming-guides/encoding/#structure)类型，这意味着由键值对定义的每个记录都是单独编码的**。

类似地，我们来看看protoscope文本格式的packed-order.bin的内容：

```text
#cat packed_order.bin | protoscope -explicit-wire-types -explicit-length-prefixes
1:VARINT 1
2:LEN 38 `fc06c0058e047293069702ea04c203ba0165c005d601da02dc02a307a804f101ca019a02df03`
```

有趣的是，一旦我们在productId字段上启用packed选项，gRPC库就会将它们一起编码以进行序列化。它将其表示为具有38个十六进制字节的单个LEN有线格式记录：

```text
fc 06 c0 05 8e 04 72 93 06 97 02 ea 04 c2 03 ba 01 65 c0 05 d6 01 da 02 dc 02 a3 07 a8 04 f1 01 ca 01 9a 02 df 03
```

关于[protobuf消息的编码](https://protobuf.dev/programming-guides/encoding/)我们就不讨论了，官方网站上已经有详细的介绍，大家也可以参考[其他网站](https://clement-jean.github.io/packed_vs_unpacked_repeated_fields/)来详细了解编码算法。

## 5. 总结

在本文中，我们探讨了protobuf中重复字段的packed选项。打包字段的元素被一起编码，因此它们的大小大大减少，这可以通过更快的序列化来提高性能。**需要注意的是，我们只能将原始数字有线类型(例如VARINT、I32或I64类型)声明为packed**。