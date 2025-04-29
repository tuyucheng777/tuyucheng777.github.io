---
layout: post
title:  Avro中的枚举值序列化
category: apache
copyright: apache
excerpt: Avro
---

## 1. 简介

[Apache Avro](https://www.baeldung.com/java-apache-avro)是一个数据序列化框架，它提供了丰富的数据结构以及紧凑、快速的二进制数据格式。在Java应用程序中使用Avro时，我们经常需要序列化[枚举](https://www.baeldung.com/a-guide-to-java-enums)值，如果处理不当，这可能会非常棘手。

在本教程中，我们将探索如何使用Avro正确序列化Java枚举值。此外，我们还将解决在Avro中使用枚举时可能遇到的常见问题。

## 2. 理解Avro枚举序列化

在Avro中，枚举由一个名称和一组符号定义，**[序列化](https://www.baeldung.com/java-serialization)Java枚举时，我们必须确保模式中的枚举定义与Java枚举定义匹配**。这一点很重要，因为Avro会在序列化过程中验证枚举值。

Avro采用基于模式的方法，这意味着模式定义了数据的结构，包括字段名称、类型，以及(如果是枚举)允许的符号值。因此，模式充当了序列化器和反序列化器之间的契约，从而有助于确保数据一致性。

让我们首先向项目添加必要的Avro Maven[依赖](https://mvnrepository.com/artifact/org.apache.avro/avro/)：

```xml
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>1.12.0</version>
</dependency>
```

## 3. 在Avro模式中定义枚举

首先，让我们看看在创建Avro模式时如何正确定义枚举：

```java
Schema colorEnum = SchemaBuilder.enumeration("Color")
    .namespace("cn.tuyucheng.taketoday.apache.avro")
    .symbols("UNKNOWN", "GREEN", "RED", "BLUE");
```

这将创建一个包含4个可用值的枚举模式，**命名空间有助于防止命名冲突，此外，符号定义了有效的枚举值**。

现在，让我们在记录模式中使用这个枚举：

```java
Schema recordSchema = SchemaBuilder.record("ColorRecord")
    .namespace("cn.tuyucheng.taketoday.apache.avro")
    .fields()
    .name("color")
    .type(colorEnum)
    .noDefault()
    .endRecord();
```

此初始化创建了一个记录模式ColorRecord，其中有一个名为color的字段，属于我们之前定义的Enum类型。

## 4. 序列化枚举值

现在我们已经定义了枚举模式，让我们探索如何序列化枚举值。

**在本节中，我们将讨论基本枚举序列化的标准方法。此外，我们还将解决在联合类型中处理枚举的常见挑战，这常常会引起混淆**。

### 4.1 基本枚举序列化的正确方法

**为了正确序列化枚举值，我们需要创建一个EnumSymbol对象**。因此，我们将使用适当的枚举模式(colorEnum)：

```java
public void serializeEnumValue() throws IOException {
    GenericRecord record = new GenericData.Record(recordSchema);
    GenericData.EnumSymbol colorSymbol = new GenericData.EnumSymbol(colorEnum, "RED");
    record.put("color", colorSymbol);

    DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(recordSchema);
    try (DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<>(datumWriter)) {
        dataFileWriter.create(recordSchema, new File("color.avro"));
        dataFileWriter.append(record);
    }
}
```

首先，我们基于recordSchema创建一个GenericRecord。接下来，我们用枚举模式(colorEnum)和值“RED”创建一个EnumSymbol。最后，我们将其添加到记录中，并使用DatumWriter和DataFileWriter将其序列化到临时文件中。

现在，让我们测试一下我们的实现：

```java
@Test
void whenSerializingEnum_thenSuccess() throws IOException {
    File file = tempDir.resolve("color.avro").toFile();

    serializeEnumValue();

    DatumReader<GenericRecord> datumReader = new GenericDatumReader<>(recordSchema);
    try (DataFileReader<GenericRecord> dataFileReader = new DataFileReader<>(file, datumReader)) {
        GenericRecord result = dataFileReader.next();
        assertEquals("RED", result.get("color").toString());
    }
}
```

该测试确认我们可以成功序列化和反序列化枚举值。

### 4.2 使用枚举处理联合类型

现在，让我们看看如何处理可能面临的常见问题-在联合类型中序列化枚举：

```java
Schema colorEnum = SchemaBuilder.enumeration("Color")
    .namespace("cn.tuyucheng.taketoday.apache.avro")
    .symbols("UNKNOWN", "GREEN", "RED", "BLUE");
    
Schema unionSchema = SchemaBuilder.unionOf()
    .type(colorEnum)
    .and()
    .nullType()
    .endUnion();
    
Schema recordWithUnionSchema = SchemaBuilder.record("ColorRecordWithUnion")
    .namespace("cn.tuyucheng.taketoday.apache.avro")
    .fields()
    .name("color")
    .type(unionSchema)
    .noDefault()
    .endRecord();
```

我们定义了一个联合模式，它可以是枚举类型，也可以为null。当字段为可选类型时，这种模式很常见。接下来，我们创建了一个包含使用此联合类型的字段的记录模式。

因此，当我们在联合中序列化枚举时，我们仍然会使用EnumSymbol，但使用正确的模式引用：

```java
GenericRecord record = new GenericData.Record(recordWithUnionSchema);
GenericData.EnumSymbol colorSymbol = new GenericData.EnumSymbol(colorEnum, "RED");
record.put("color", colorSymbol);
```

**这里需要注意的一点是，我们创建EnumSymbol时使用的是枚举模式，而不是联合模式**。这是一个常见的错误，会导致序列化错误。

现在，让我们测试联合处理的实现：

```java
@Test
void whenSerializingEnumInUnion_thenSuccess() throws IOException {
    File file = tempDir.resolve("colorUnion.avro").toFile();

    GenericRecord record = new GenericData.Record(recordWithUnionSchema);
    GenericData.EnumSymbol colorSymbol = new GenericData.EnumSymbol(colorEnum, "GREEN");
    record.put("color", colorSymbol);

    DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(recordWithUnionSchema);
    try (DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<>(datumWriter)) {
        dataFileWriter.create(recordWithUnionSchema, file);
        dataFileWriter.append(record);
    }

    DatumReader<GenericRecord> datumReader = new GenericDatumReader<>(recordWithUnionSchema);
    try (DataFileReader<GenericRecord> dataFileReader = new DataFileReader<>(file, datumReader)) {
        GenericRecord result = dataFileReader.next();
        assertEquals("GREEN", result.get("color").toString());
    }
}
```

此外，我们还可以测试在联合中处理[null](https://www.baeldung.com/java-null)：

```java
@Test
void whenSerializingNullInUnion_thenSuccess() throws IOException {
    File file = tempDir.resolve("colorNull.avro").toFile();

    GenericRecord record = new GenericData.Record(recordWithUnionSchema);
    record.put("color", null);

    DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(recordWithUnionSchema);
    assertDoesNotThrow(() -> {
        try (DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<>(datumWriter)) {
            dataFileWriter.create(recordWithUnionSchema, file);
            dataFileWriter.append(record);
        }
    });
}
```

## 5. 使用枚举进行模式演化

在处理枚举时，模式演进是一个特别敏感的领域，因为添加或删除枚举值可能会导致兼容性问题。**在本节中，我们将探讨如何根据需求变化更新数据结构**，我们将重点介绍如何使用枚举类型，以及如何通过合理的默认值配置来保持向后兼容性。

### 5.1 添加新的枚举值

当我们需要扩展模式时，添加新的枚举值需要仔细考虑，我们需要考虑兼容性问题。**因此，为了向后兼容，添加默认值至关重要**：

```java
@Test
void whenSchemaEvolution_thenDefaultValueUsed() throws IOException {
    String evolvedSchemaJson = "{\"type\":\"record\",
                                 \"name\":\"ColorRecord\",
                                 \"namespace\":\"cn.tuyucheng.taketoday.apache.avro\",
                                 \"fields\":
                                   [{\"name\":\"color\",
                                     \"type\":
                                        {\"type\":\"enum\",
                                         \"name\":\"Color\",
                                     \"symbols\":[\"UNKNOWN\",\"GREEN\",\"RED\",\"BLUE\",\"YELLOW\"],
                                         \"default\":\"UNKNOWN\"
                                   }}]
                                 }";
    
    Schema evolvedRecordSchema = new Schema.Parser().parse(evolvedSchemaJson);
    Schema evolvedEnum = evolvedRecordSchema.getField("color").schema();
    
    File file = tempDir.resolve("colorEvolved.avro").toFile();

    GenericRecord record = new GenericData.Record(evolvedRecordSchema);
    GenericData.EnumSymbol colorSymbol = new GenericData.EnumSymbol(evolvedEnum, "YELLOW");
    record.put("color", colorSymbol);

    DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(evolvedRecordSchema);
    try (DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<>(datumWriter)) {
        dataFileWriter.create(evolvedRecordSchema, file);
        dataFileWriter.append(record);
    }
    
    String originalSchemaJson = "{\"type\":\"record\",
                                  \"name\":\"ColorRecord\",
                                  \"namespace\":\"cn.tuyucheng.taketoday.apache.avro\",
                                  \"fields\":[{
                                     \"name\":\"color\",
                                     \"type\":
                                         {\"type\":\"enum\",
                                          \"name\":\"Color\",
                                          \"symbols\":[\"UNKNOWN\",\"GREEN\",\"RED\",\"BLUE\"],
                                          \"default\":\"UNKNOWN\"}}]
                                 }";
    
    Schema originalRecordSchema = new Schema.Parser().parse(originalSchemaJson);
    
    DatumReader<GenericRecord> datumReader = 
                    new GenericDatumReader<>(evolvedRecordSchema, originalRecordSchema);
    try (DataFileReader<GenericRecord> dataFileReader = new DataFileReader<>(file, datumReader)) {
        GenericRecord result = dataFileReader.next();
        assertEquals("UNKNOWN", result.get("color").toString());
    }
}
```

现在，让我们分析一下上面的代码。我们改进了模式(evolvedSchemaJson)，并添加了一个新的符号“YELLOW”。接下来，我们创建了一个带有“YELLOW”枚举值的记录，并将其写入文件中。

然后，我们创建了一个“原始模式”(originalSchemaJson)，但默认值保持不变。为了避免忘记，我们之前已经指出，添加默认值对于向后兼容非常重要。

最后，当我们使用原始模式读取数据时，我们验证使用默认值“UNKNOWN”而不是“YELLOW”。

**为了正确地使用枚举进行模式演化，我们需要在枚举类型级别而不是字段级别指定默认值**。在我们的示例中，这就是为什么我们使用JSON字符串来定义模式，因为它使我们能够直接控制结构。

## 6. 总结

在本文中，我们探讨了如何使用Apache Avro正确地序列化枚举值，我们研究了基本的枚举序列化、枚举联合的处理以及解决模式演变的挑战。

在Avro中使用枚举时，我们应该记住一些关键点。首先，我们需要使用正确的命名空间和符号来定义枚举模式。使用GenericData.EnumSymbol并添加合适的枚举模式引用非常重要。

此外，对于联合类型，我们使用枚举模式而不是联合模式创建枚举符号。

最后，关于模式演变，我们需要将默认值放在枚举类型级别以实现适当的兼容性。