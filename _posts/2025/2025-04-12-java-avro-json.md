---
layout: post
title:  使用Java将Avro文件转换为JSON文件
category: apache
copyright: apache
excerpt: Apache Avro
---

## 1. 概述

[Apache Avro](https://www.baeldung.com/java-apache-avro)是一个广泛使用的数据序列化系统，由于其高效性和模式演化能力，在大数据应用中尤其受欢迎。在本教程中，我们将演示如何通过Avro将对象转换为JSON，以及如何将整个Avro文件转换为JSON文件，这对于数据检查和调试尤其有用。

在当今数据驱动的世界中，处理不同数据格式的能力至关重要；Apache Avro通常用于需要高性能和存储效率的系统，例如Apache Hadoop。

## 2. 配置

首先，让我们将Avro和JSON的依赖添加到pom.xml文件中。

我们为本教程添加了Apache Avro [1.11.1版本](https://mvnrepository.com/artifact/org.apache.avro/avro/1.11.1)：

```xml
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>1.11.1</version>
</dependency>
```

## 3. 将Avro对象转换为JSON

通过Avro将Java对象转换为JSON涉及多个步骤，包括：

- 推断/构建Avro模式
- 将Java对象转换为Avro GenericRecord，最后
- 将对象转换为JSON

我们将利用Avro的反射API从Java对象动态推断模式，而不是手动定义模式。

为了证明这一点，让我们创建一个具有两个int属性x和y的Point类：

```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // Getters and setters
}
```

让我们继续推断模式：

```java
public Schema inferSchema(Point p) {
    return ReflectData.get().getSchema(p.getClass());
}
```

我们定义了一个方法inferSchema，并使用ReflectData类的getSchema方法从Point对象推断出模式，该模式描述了字段x和y及其数据类型。

接下来，让我们从Point对象创建一个GenericRecord对象并将其转换为JSON：

```java
public String convertObjectToJson(Point p, Schema schema) {
    try {
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        GenericDatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(schema);
        GenericRecord genericRecord = new GenericData.Record(schema);
        genericRecord.put("x", p.getX());
        genericRecord.put("y", p.getY());
        Encoder encoder = EncoderFactory.get().jsonEncoder(schema, outputStream);
        datumWriter.write(genericRecord, encoder);
        encoder.flush();
        outputStream.close();
        return outputStream.toString();
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

convertObjectToJson方法使用提供的模式将Point对象转换为JSON字符串，首先，根据提供的模式创建一个GenericRecord对象，并用Point对象的数据填充该对象；然后，使用DatumWriter通过JsonEncoder对象将数据传递到ByteArrayOutputStream；最后，对OutputStream对象使用toString方法获取JSON字符串。

让我们验证一下生成的JSON的内容：

```java
private AvroFileToJsonFile avroFileToJsonFile;
private Point p;
private String expectedOutput;

@BeforeEach
public void setup() {
    avroFileToJsonFile = new AvroFileToJsonFile();
    p = new Point(2, 4);
    expectedOutput = "{\"x\":2,\"y\":4}";
}

@Test
public void whenConvertedToJson_ThenEquals() {
    String response = avroFileToJsonFile.convertObjectToJson(p, avroFileToJsonFile.inferSchema(p));
    assertEquals(expectedOutput, response);
}
```

## 4. 将Avro文件转换为JSON文件

将整个Avro文件转换为JSON文件的过程类似，但需要从文件中读取数据。当我们将Avro格式的数据存储在磁盘上，并需要将其转换为更易于访问的格式(例如JSON)时，这种情况很常见。

让我们首先定义一个方法writeAvroToFile，它将用于将一些Avro数据写入文件：

```java
public void writeAvroToFile(Schema schema, List<Point> records, File writeLocation) {
    try {
        if (writeLocation.exists()) {
            if (!writeLocation.delete()) {
                System.err.println("Failed to delete existing file.");
                return;
            }
        }
        GenericDatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(schema);
        DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<>(datumWriter);
        dataFileWriter.create(schema, writeLocation);
        for (Point record: records) {
            GenericRecord genericRecord = new GenericData.Record(schema);
            genericRecord.put("x", record.getX());
            genericRecord.put("y", record.getY());
            dataFileWriter.append(genericRecord);
        }
        dataFileWriter.close();
    } catch (IOException e) {
        e.printStackTrace();
        System.out.println("Error writing Avro file.");
    }
}
```

该方法根据提供的Schema将Point对象构造为GenericRecord实例，从而将其转换为Avro格式。GenericDatumWrite将这些记录序列化，然后使用DataFileWriter将其写入Avro文件。

让我们验证该文件是否已写入并且是否存在：

```java
private File dataLocation;
private File jsonDataLocation;
// ...

@BeforeEach
public void setup() {
    // Load files from the resources folder
    ClassLoader classLoader = getClass().getClassLoader();
    dataLocation = new File(classLoader.getResource("").getFile(), "data.avro");
    jsonDataLocation = new File(classLoader.getResource("").getFile(), "data.json");
    // ...
}

// ...

@Test
public void whenAvroContentWrittenToFile_ThenExist(){
    Schema schema = avroFileToJsonFile.inferSchema(p);
    avroFileToJsonFile.writeAvroToFile(schema, List.of(p), dataLocation);
    assertTrue(dataLocation.exists());
}
```

接下来，我们将从存储位置读取文件并将其以JSON格式写回另一个文件。

让我们创建一个名为readAvroFromFileToJsonFile的方法来处理这个问题：

```java
public void readAvroFromFileToJsonFile(File readLocation, File jsonFilePath) {
    DatumReader<GenericRecord> reader = new GenericDatumReader<>();
    try {
        DataFileReader<GenericRecord> dataFileReader = new DataFileReader<>(readLocation, reader);
        DatumWriter<GenericRecord> jsonWriter = new GenericDatumWriter<>(dataFileReader.getSchema());
        Schema schema = dataFileReader.getSchema();
        OutputStream fos = new FileOutputStream(jsonFilePath);
        JsonEncoder jsonEncoder = EncoderFactory.get().jsonEncoder(schema, fos);
        while (dataFileReader.hasNext()) {
            GenericRecord record = dataFileReader.next();
            System.out.println(record.toString());
            jsonWriter.write(record, jsonEncoder);
            jsonEncoder.flush();
        }
        dataFileReader.close();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

我们从readLocation读取Avro数据，并将其以JSON格式写入jsonFilePath。我们使用DataFileReader从Avro文件中读取GenericRecord实例，然后使用JsonEncoder和GenericDatumWriter将这些记录序列化为JSON格式。

让我们继续确认写入生成的文件的JSON内容：

```java
@Test
public void whenAvroFileWrittenToJsonFile_ThenJsonContentEquals() throws IOException {
    avroFileToJsonFile.readAvroFromFileToJsonFile(dataLocation, jsonDataLocation);
    String text = Files.readString(jsonDataLocation.toPath());
    assertEquals(expectedOutput, text);
}
```

## 5. 总结

在本文中，我们探讨了如何将Avro的内容写入文件、读取文件并将其存储为JSON格式的文件，并通过示例演示了整个过程。此外，值得注意的是，模式也可以存储在单独的文件中，而不是包含在数据中。