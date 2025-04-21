---
layout: post
title:  如何在Avro中序列化和反序列化日期
category: apache
copyright: apache
excerpt: Avro
---

## 1. 简介

在本教程中，我们将探索使用[Apache Avro](https://www.baeldung.com/java-apache-avro)在Java中序列化和反序列化Date对象的不同方法。该框架是一个数据序列化系统，它提供紧凑、快速的二进制数据格式以及基于模式的数据定义。

**在Avro中处理日期时，我们面临一些挑战，因为Avro的类型结构本身并不支持Java的[Date](https://www.baeldung.com/java-8-date-time-intro)类**。现在，让我们更详细地了解一下Date序列化所面临的挑战。

## 2. 日期序列化的挑战

在开始之前，让我们将Avro[依赖](https://mvnrepository.com/artifact/org.apache.avro/avro)添加到Maven项目中：

```xml
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>1.12.0</version>
</dependency>
```

Avro的类型系统由基本类型组成：null、boolean、int、long、float、double、bytes和string。此外，还支持复杂类型：record、enum、array、map、union和Fixed。

现在，让我们看一个例子来了解为什么日期[序列化](https://www.baeldung.com/java-serialization)在Avro中会出现问题：

```java
public class DateContainer {
    private Date date;
    
    // Constructors, getters, and setters
}
```

**当我们尝试使用Avro的基于反射的序列化直接序列化此类时，默认行为会在内部将Date对象转换为long值(自纪元以来的毫秒数)**。

不幸的是，这个过程可能会导致精度问题。例如，反序列化的值可能与原始值相差几毫秒。

## 3. 实现日期序列化

接下来，我们将使用两种方法实现日期序列化和反序列化：使用带有GenericRecord的逻辑类型和使用Avro的转换API。

### 3.1 使用带有GenericRecord的逻辑类型

从Avro 1.8开始，框架提供了逻辑类型，这些逻辑类型为底层原始类型添加了必要且适当的含义。

因此，对于日期，我们有三种逻辑类型：

1. date：表示不带时间的日期，它以int形式存储(即自纪元以来的天数)
2. timestamp-millis：表示具有毫秒精度的时间戳，存储为long
3. timestamp-micros：表示具有微秒精度的时间戳，存储为long

现在，让我们看看如何在Avro模式中使用这些逻辑类型：

```java
public static Schema createDateSchema() {
    String schemaJson = 
        "{"
        + "\"type\": \"record\","
        + "\"name\": \"DateRecord\","
        + "\"fields\": ["
        + "  {\"name\": \"date\", \"type\": {\"type\": \"int\", \"logicalType\": \"date\"}},"
        + "  {\"name\": \"timestamp\", \"type\": {\"type\": \"long\", \"logicalType\": \"timestamp-millis\"}}"
        + "]"
        + "}";
    return new Schema.Parser().parse(schemaJson);
}
```

值得注意的是，我们将逻辑类型应用于底层原始类型，而不是直接应用于字段。

现在，让我们看看如何使用逻辑类型实现Date序列化：

```java
public static byte[] serializeDateWithLogicalType(LocalDate date, Instant timestamp) {
    Schema schema = createDateSchema();
    GenericRecord record = new GenericData.Record(schema);
    
    record.put("date", (int) date.toEpochDay());
    
    record.put("timestamp", timestamp.toEpochMilli());
    
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(schema);
    Encoder encoder = EncoderFactory.get().binaryEncoder(baos, null);
    
    datumWriter.write(record, encoder);
    encoder.flush();
    
    return baos.toByteArray();
}
```

让我们回顾一下上面的逻辑，我们将LocalDate转换为自纪元以来的天数，将timestamp转换为自纪元以来的毫秒数。这样，我们就可以使用逻辑类型了。

现在，让我们实现处理反序列化的方法：

```java
public static Pair<LocalDate, Instant> deserializeDateWithLogicalType(byte[] bytes) {
    Schema schema = createDateSchema();
    DatumReader<GenericRecord> datumReader = new GenericDatumReader<>(schema);
    Decoder decoder = DecoderFactory.get().binaryDecoder(bytes, null);
    
    GenericRecord record = datumReader.read(null, decoder);
    
    LocalDate date = LocalDate.ofEpochDay((int) record.get("date"));
    
    Instant timestamp = Instant.ofEpochMilli((long) record.get("timestamp"));
    
    return Pair.of(date, timestamp);
}
```

最后，测试一下我们的实现：

```java
@Test
void whenSerializingDateWithLogicalType_thenDeserializesCorrectly() {
    LocalDate expectedDate = LocalDate.now();
    Instant expectedTimestamp = Instant.now();

    byte[] serialized = serializeDateWithLogicalType(expectedDate, expectedTimestamp);
    Pair<LocalDate, Instant> deserialized = deserializeDateWithLogicalType(serialized);

    assertEquals(expectedDate, deserialized.getLeft());

    assertEquals(expectedTimestamp.toEpochMilli(), deserialized.getRight().toEpochMilli(), "Timestamps should match exactly at millisecond precision");
}
```

**从测试中我们可以看出，timestamp-millis逻辑类型保持了精度，并且时间戳与预期匹配**。此外，使用逻辑类型使我们的数据格式在模式定义中清晰可见，这对于模式开发和文档编写非常有价值。

### 3.2 使用Avro的转换API

**Avro提供了一个可以自动处理逻辑类型的转换API**，此API并非一种独立的方案，事实上，它建立在逻辑类型之上，有助于加快转换过程。

这样，我们就无需在Java类型和Avro内部表示之间手动转换了。此外，它还为转换过程增加了类型安全性。

现在，让我们实现自动处理逻辑类型的解决方案：

```java
public static byte[] serializeWithConversionApi(LocalDate date, Instant timestamp) {
    Schema schema = createDateSchema();
    GenericRecord record = new GenericData.Record(schema);

    Conversion<LocalDate> dateConversion = new org.apache.avro.data.TimeConversions.DateConversion();
    LogicalTypes.date().addToSchema(schema.getField("date").schema());

    Conversion<Instant> timestampConversion = new org.apache.avro.data.TimeConversions.TimestampMillisConversion();
    LogicalTypes.timestampMillis().addToSchema(schema.getField("timestamp").schema());

    record.put("date", dateConversion.toInt(date, schema.getField("date").schema(), LogicalTypes.date()));
    record.put("timestamp",
            timestampConversion.toLong(timestamp, schema.getField("timestamp").schema(),
                    LogicalTypes.timestampMillis()));

    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(schema);
    Encoder encoder = EncoderFactory.get().binaryEncoder(baos, null);

    datumWriter.write(record, encoder);
    encoder.flush();

    return baos.toByteArray();
}
```

与之前的方法不同，这次我们使用LogicalTypes.date()和LogicalTypes.timestampMillis()进行转换。

接下来，让我们实现处理反序列化的方法：

```java
public static Pair<LocalDate, Instant> deserializeWithConversionApi(byte[] bytes) {
    Schema schema = createDateSchema();
    DatumReader<GenericRecord> datumReader = new GenericDatumReader<>(schema);
    Decoder decoder = DecoderFactory.get().binaryDecoder(bytes, null);

    GenericRecord record = datumReader.read(null, decoder);

    Conversion<LocalDate> dateConversion = new DateConversion();
    LogicalTypes.date().addToSchema(schema.getField("date").schema());

    Conversion<Instant> timestampConversion = new TimestampMillisConversion();
    LogicalTypes.timestampMillis().addToSchema(schema.getField("timestamp").schema());

    int daysSinceEpoch = (int) record.get("date");
    long millisSinceEpoch = (long) record.get("timestamp");

    LocalDate date = dateConversion.fromInt(
            daysSinceEpoch,
            schema.getField("date").schema(),
            LogicalTypes.date()
    );

    Instant timestamp = timestampConversion.fromLong(
            millisSinceEpoch,
            schema.getField("timestamp").schema(),
            LogicalTypes.timestampMillis()
    );

    return Pair.of(date, timestamp);
}
```

最后我们来验证一下实现：

```java
@Test
void whenSerializingWithConversionApi_thenDeserializesCorrectly() {
    LocalDate expectedDate = LocalDate.now();
    Instant expectedTimestamp = Instant.now();

    byte[] serialized = serializeWithConversionApi(expectedDate, expectedTimestamp);
    Pair<LocalDate, Instant> deserialized = deserializeWithConversionApi(serialized);

    assertEquals(expectedDate, deserialized.getLeft());
    assertEquals(expectedTimestamp.toEpochMilli(), deserialized.getRight().toEpochMilli(), "Timestamps should match at millisecond precision");
}
```

## 4. 处理使用Date的遗留代码

目前，许多现有的Java应用程序仍在使用旧版java.util.Date类，对于此类代码库，我们需要一种策略来在使用Avro序列化时处理这些对象。

一个好方法是在序列化信息之前将旧日期转换为现代Java时间API：

```java
public static byte[] serializeLegacyDateAsModern(Date legacyDate) {
    Instant instant = legacyDate.toInstant();
    
    LocalDate localDate = instant.atZone(ZoneId.systemDefault()).toLocalDate();
    
    return serializeDateWithLogicalType(localDate, instant);
}
```

然后，我们可以使用前面提到的方法之一序列化日期，**这种方法使我们能够利用Avro的逻辑类型，同时仍然使用传统的Date对象**。

让我们测试一下我们的实现：

```java
@Test
void whenSerializingLegacyDate_thenConvertsCorrectly() {
    Date legacyDate = new Date();
    LocalDate expectedLocalDate = legacyDate.toInstant()
        .atZone(ZoneId.systemDefault())
        .toLocalDate();

    byte[] serialized = serializeLegacyDateAsModern(legacyDate);
    LocalDate deserialized = deserializeDateWithLogicalType(serialized).getKey();
    
    assertEquals(expectedLocalDate, deserialized);
}
```

## 5. 总结

在本文中，我们探索了使用Avro序列化Date对象的不同方法，我们学习了如何使用Avro的逻辑类型来正确表示日期和时间戳值。

对于大多数现代应用程序来说，使用Avro的转换API来处理其逻辑类型(通过java.time类)是最佳方法。通过这种组合，我们可以获得类型安全性，维护正确的语义，并与Avro的模式扩展功能兼容。