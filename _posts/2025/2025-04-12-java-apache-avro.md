---
layout: post
title:  Apache Avro指南
category: apache
copyright: apache
excerpt: Apache Avro
---

## 1. 概述

数据序列化是一种将数据转换为二进制或文本格式的技术，目前有多种系统可用于此目的，[Apache Avro](https://avro.apache.org/)就是其中一种数据序列化系统。

**Avro是一个独立于语言、基于模式的数据序列化库**，它使用模式来执行序列化和反序列化。此外，Avro使用JSON格式来指定数据结构，这使其功能更加强大。

在本教程中，我们将进一步探讨Avro设置、执行序列化的Java API以及Avro与其他数据序列化系统的比较。

我们将主要关注作为整个系统基础的模式创建。

## 2. Apache Avro

Avro是一个独立于语言的序列化库，Avro使用其核心组件之一的模式来实现这一点，**它将模式存储在文件中，以便进一步处理数据**。

Avro最适合大数据处理，它因其更快的处理速度在Hadoop和Kafka领域颇受欢迎。

Avro创建一个数据文件，将数据和模式保存在其元数据部分。最重要的是，它提供了丰富的数据结构，这使得它比其他类似的解决方案更受欢迎。

要使用Avro进行序列化，我们需要遵循下面提到的步骤。

## 3. 问题陈述

我们先定义一个名为AvroHttRequest的类，用于示例，该类包含原始类型属性和复杂类型属性：

```java
class AvroHttpRequest {
    private long requestTime;
    private ClientIdentifier clientIdentifier;
    private List<String> employeeNames;
    private Active active;
}
```

这里，requestTime是一个原始值，ClientIdentifier是另一个表示复杂类型的类，employeeNames也是一个复杂类型。Active是一个枚举，用于描述给定的员工列表是否处于活跃状态。

**我们的目标是使用Apache Avro序列化和反序列化AvroHttRequest类**。

## 4. Avro数据类型

在继续之前，让我们讨论一下Avro支持的数据类型。

Avro支持两种类型的数据：

- 原始类型：Avro支持所有原始类型，我们使用原始类型名称来定义给定字段的类型。例如，一个包含字符串的值在schema中应声明为{"type": "string"}
- 复杂类型：Avro支持6种复杂类型：记录、枚举、数组、映射、联合和固定

例如，在我们的问题陈述中，ClientIdentifier是一个记录。

在这种情况下，ClientIdentifier的模式应如下所示：

```json
{
    "type":"record",
    "name":"ClientIdentifier",
    "namespace":"cn.tuyucheng.taketoday.avro.model",
    "fields":[
        {
            "name":"hostName",
            "type":"string"
        },
        {
            "name":"ipAddress",
            "type":"string"
        }
    ]
}
```

## 5. 使用Avro

首先，让我们将所需的Maven依赖添加到pom.xml文件中。

我们应该包括以下依赖：

- Apache Avro：核心组件
- 编译器：用于Avro IDL和Avro特定Java APIT的Apache Avro编译器
- 工具：包括Apache Avro命令行工具和实用程序
- 适用于Maven项目的Apache Avro Maven插件

本教程使用的是1.8.2版本。

但是，始终建议在[Maven Central](https://mvnrepository.com/artifact/org.apache.avro/avro)上查找最新版本：

```xml
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-compiler</artifactId>
    <version>1.8.2</version>
</dependency>
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>1.8.2</version>
</dependency>
```

添加Maven依赖后，下一步将是：

- 模式创建
- 在我们的程序中读取模式
- 使用Avro序列化数据
- 最后，反序列化数据

## 6. 模式创建

Avro使用JSON格式描述其模式，给定的Avro模式主要有4个属性：

- type：描述模式的类型，无论是复杂类型还是原始值
- namespace：描述给定模式所属的命名空间
- name：模式的名称
- fields：描述与给定模式关联的字段，**字段可以是原始类型，也可以是复杂类型**

创建模式的一种方法是编写JSON表示，正如我们在前面的部分中看到的那样。

我们还可以使用SchemaBuilder创建模式，这无疑是一种更好、更有效的创建方式。

### 6.1 SchemaBuilder实用程序

org.apache.avro.SchemaBuilder类对于创建模式很有用。

首先，让我们为ClientIdentifier创建模式：

```java
Schema clientIdentifier = SchemaBuilder.record("ClientIdentifier")
    .namespace("cn.tuyucheng.taketoday.avro.model")
    .fields()
    .requiredString("hostName")
    .requiredString("ipAddress")
    .endRecord();
```

现在，让我们使用它来创建一个avroHttpRequest模式：

```java
Schema avroHttpRequest = SchemaBuilder.record("AvroHttpRequest")
    .namespace("cn.tuyucheng.taketoday.avro.model")
    .fields().requiredLong("requestTime")
    .name("clientIdentifier")
        .type(clientIdentifier)
        .noDefault()
    .name("employeeNames")
        .type()
        .array()
        .items()
        .stringType()
        .arrayDefault(null)
    .name("active")
        .type()
        .enumeration("Active")
        .symbols("YES","NO")
        .noDefault()
    .endRecord();
```

这里需要注意的是，我们已将clientIdentifier指定为clientIdentifier字段的类型，在这种情况下，用于定义类型的clientIdentifier与我们之前创建的模式相同。

### 6.2 使用模式对象

如前所述，我们可以利用SchemaBuilder的流式API声明式地生成一个org.apache.avro.Schema对象，**之后，我们可以调用toString()方法获取Schema的JSON结构**。

让我们验证一下为ClientIdentifier记录创建的Schema实例是否生成了正确的JSON，我们可以使用像[JsonUnit](https://www.baeldung.com/jsonunit-assertj-json-unit-test)这样的专用断言库来实现这一点：

```java
@Test
void whenCallingSchemaToString_thenReturnJsonAvroSchema() {
    Schema clientIdSchema = clientIdentifierSchema();

    assertThatJson(clientIdSchema.toString())
            .isEqualTo("""
                    {
                        "type":"record",
                        "name":"ClientIdentifier",
                        "namespace":"cn.tuyucheng.taketoday.avro.model",
                        "fields":[
                            {
                                "name":"hostName",
                                "type":"string"
                            },
                            {
                                "name":"ipAddress",
                                "type":"string"
                            }
                        ]
                    }
                    """);
}
```

不用说，我们可以做同样的事情来为AvroHttpRequest记录生成Avro模式。

**然后，我们可以将这些生成的模式保存为src/main/resources下的.avsc文件**，这样我们稍后就可以将这些文件与avro-maven-plugin插件一起使用。

## 7. 读取模式

我们可以使用Schema实例来创建org.apache.avro.generic.GenericRecord对象，**GenericRecord API允许我们以基于模式的格式存储数据，而无需预定义的Java类**。

**然而，更流行的方法是使用.avro模式文件来创建Avro类**；创建类后，我们就可以使用它们来序列化和反序列化对象，创建Avro类有两种方法：

- 以编程方式生成Avro类：可以使用SchemaCompiler生成类，我们可以使用一些API来生成Java类，可以在GitHub上找到生成类的代码。
- 使用Maven插件生成类

我们可以使用[avro-maven-plugin](https://mvnrepository.com/artifact/org.apache.avro/avro-maven-plugin)基于.avsc文件生成Java类，让我们将该插件添加到pom.xml中：

```xml
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>${avro.version}</version>
    <executions>
        <execution>
            <id>schemas</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>schema</goal>
                <goal>protocol</goal>
                <goal>idl-protocol</goal>
            </goals>
            <configuration>
                <sourceDirectory>${project.basedir}/src/main/resources/</sourceDirectory>
                <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**现在，我们可以简单地运行“mvn clean install”，插件会在generate-sources阶段根据我们的.avsc文件生成Java类**。

## 8. 使用Avro进行序列化和反序列化

当我们完成生成模式后，让我们继续探索序列化部分。

Avro支持两种数据序列化格式：JSON格式和二进制格式。

首先，我们将重点关注JSON格式，然后讨论二进制格式。

在继续之前，我们应该先了解几个关键接口，我们可以使用下面的接口和类进行序列化：

DatumWriter<T\>：我们应该用它来向给定的模式写入数据，在本例中，我们将使用SpecificDatumWriter的实现，不过DatumWriter也有其他实现，包括GenericDatumWriter、Json.Writer、ProtobufDatumWriter、ReflectDatumWriter和ThriftDatumWriter。

Encoder：Encoder用于定义前面提到的格式，EncoderFactory提供两种类型的编码器：二进制编码器和JSON编码器。

DatumReader<D\>：用于反序列化的单一接口，同样，它有多个实现，但我们在示例中将使用SpecificDatumReader；其他实现包括GenericDatumReader、Json.ObjectReader、Json.Reader、ProtobufDatumReader、ReflectDatumReader和ThriftDatumReader。

Decoder：Decoder在反序列化数据时使用，DecoderFactory提供两种类型的解码器：二进制解码器和JSON解码器。

接下来，让我们看看Avro中序列化和反序列化是如何发生的。

### 8.1 序列化

我们将以AvroHttpRequest类为例，并使用Avro对其进行序列化。

首先我们将其序列化为JSON格式：

```java
public byte[] serializeAvroHttpRequestJSON(AvroHttpRequest request) {
    DatumWriter<AvroHttpRequest> writer = new SpecificDatumWriter<>(AvroHttpRequest.class);
    byte[] data = new byte[0];
    ByteArrayOutputStream stream = new ByteArrayOutputStream();
    Encoder jsonEncoder = null;
    try {
        jsonEncoder = EncoderFactory.get().jsonEncoder(AvroHttpRequest.getClassSchema(), stream);
        writer.write(request, jsonEncoder);
        jsonEncoder.flush();
        data = stream.toByteArray();
    } catch (IOException e) {
        logger.error("Serialization error:" + e.getMessage());
    }
    return data;
}
```

让我们看一下此方法的测试用例：

```java
@Test
public void givenJSONEncoder_whenSerialized_thenObjectGetsSerialized(){
    byte[] data = serializer.serializeAvroHttpRequestJSON(request);
    assertTrue(Objects.nonNull(data));
    assertTrue(data.length > 0);
}
```

这里我们使用了jsonEncoder方法并将模式传递给它。

如果我们想使用二进制编码器，我们需要用binaryEncoder()替换jsonEncoder()方法：

```java
Encoder jsonEncoder = EncoderFactory.get().binaryEncoder(stream,null);
```

### 8.2 反序列化

为此，我们将使用上面提到的DatumReader和Decoder接口。

正如我们使用EncoderFactory来获取Encoder一样，同样我们将使用DecoderFactory来获取Decoder对象。

让我们使用JSON格式反序列化数据：

```java
public AvroHttpRequest deSerializeAvroHttpRequestJSON(byte[] data) {
    DatumReader<AvroHttpRequest> reader = new SpecificDatumReader<>(AvroHttpRequest.class);
    Decoder decoder = null;
    try {
        decoder = DecoderFactory.get().jsonDecoder(AvroHttpRequest.getClassSchema(), new String(data));
        return reader.read(null, decoder);
    } catch (IOException e) {
        logger.error("Deserialization error:" + e.getMessage());
    }
}
```

让我们看一下测试用例：

```java
@Test
public void givenJSONDecoder_whenDeserialize_thenActualAndExpectedObjectsAreEqual(){
    byte[] data = serializer.serializeAvroHttpRequestJSON(request);
    AvroHttpRequest actualRequest = deSerializer.deSerializeAvroHttpRequestJSON(data);
    assertEquals(actualRequest,request);
    assertTrue(actualRequest.getRequestTime().equals(request.getRequestTime()));
}
```

类似地，我们可以使用二进制解码器：

```java
Decoder decoder = DecoderFactory.get().binaryDecoder(data, null);
```

## 9. 总结

Apache Avro在处理大数据时特别有用，它提供二进制和JSON格式的数据序列化，可根据用例使用。

Avro序列化过程速度更快，而且节省空间；Avro不会为每个字段保留字段类型信息，而是在模式中创建元数据。

最后但并非最不重要的一点是，Avro与各种编程语言都有很好的绑定，这让它具有优势。