---
layout: post
title:  从特定Java类生成Avro Schema
category: kafka
copyright: kafka
excerpt: Avro
---

## 1. 简介

在本教程中，我们将讨论从现有Java类生成[Avro](https://www.baeldung.com/java-apache-avro)模式的不同选项，虽然这不是标准的工作流程，但这种转换方式也可能发生，并且最好以最简单的方式了解现有库的使用方法。

## 2. 什么是Avro？

在深入研究将现有类转换回模式的细微差别之前，让我们先回顾一下什么是Avro。

**根据文档，它是一个数据序列化系统，能够按照预定义的模式对数据进行序列化和反序列化，这也是该系统的核心，模式本身以JSON格式表示**。更多关于Avro的信息，请参阅已发布的[指南](https://www.baeldung.com/java-apache-avro)。

## 3. 从现有Java类生成Avro Schema的动机

使用Avro的标准工作流程包括定义模式，然后用所选语言生成类，**虽然这是最常用的方式，但也可以倒推，从项目中现有的类生成Avro Schema**。

假设我们正在使用一个遗留系统，并希望通过消息代理发送数据，我们决定使用Avro作为序列化/反序列化解决方案。在仔细研究代码后，我们发现通过发送现有类表示的数据，可以快速满足新规则的要求。

手动将Java代码转换为Avro JSON模式会非常繁琐，相反，我们可以使用现有的库来帮我们完成这项工作，从而节省时间。

## 4. 使用Avro反射API生成Avro Schema

**第一个允许我们将现有Java类快速转换为Avro模式的选项是使用Avro Reflection API**，要使用此API，我们需要确保我们的项目依赖[Avro库](https://mvnrepository.com/artifact/org.apache.avro/avro)：

```xml
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>1.12.0</version>
</dependency>
```

### 4.1 简单记录

假设我们想使用反射API来获取一个简单的Java记录：

```java
record SimpleBankAccount(String bankAccountNumber) {
}
```

**我们可以使用ReflectData的单例实例为任何给定的Java类生成一个org.apache.avro.Schema对象**，然后，我们可以调用Schema实例的toString()方法，将Avro模式转换为JSON字符串。

为了验证生成的字符串是否符合我们的期望，我们可以使用[JsonUnit](https://www.baeldung.com/jsonunit-assertj-json-unit-test)：

```java
@Test
void whenConvertingSimpleRecord_thenAvroSchemaIsCorrect() {
    Schema schema = ReflectData.get()
            .getSchema(SimpleBankAccount.class);
    String jsonSchema = schema.toString();

    assertThatJson(jsonSchema).isEqualTo("""
            {
                "type" : "record",
                "name" : "SimpleBankAccount",
                "namespace" : "cn.tuyucheng.taketoday.apache.avro.model",
                "fields" : [ {
                    "name" : "bankAccountNumber",
                    "type" : "string"
                } ]
            }
            """);
}
```

**尽管我们为了简单起见使用了Java记录，但这对于普通的Java对象也同样有效**。

### 4.2 可空字段

让我们在Java记录中添加另一个String字段，我们可以使用@org.apache.avro.reflect.Nullable注解将其标记为可选：

```java
record BankAccountWithNullableField(
        String bankAccountNumber,
        @Nullable String reference
) {
}
```

如果我们重复测试，我们可以预期reference的可空性将会得到体现：

```java
@Test
void whenConvertingRecordWithNullableField_thenAvroSchemaIsCorrect() {
    Schema schema = ReflectData.get()
            .getSchema(BankAccountWithNullableField.class);
    String jsonSchema = schema.toString(true);

    assertThatJson(jsonSchema).isEqualTo("""
            {
                "type" : "record",
                "name" : "BankAccountWithNullableField",
                "namespace" : "cn.tuyucheng.taketoday.apache.avro.model",
                "fields" : [ {
                    "name" : "bankAccountNumber",
                    "type" : "string"
                }, {
                    "name" : "reference",
                    "type" : [ "null", "string" ],
                    "default" : null
                } ]
            }
            """);
}
```

**我们可以看到，在新字段上应用@Nullable注解使得生成的模式联合中的reference字段为null**。

### 4.3 忽略字段

Avro库还允许我们在生成schema时忽略某些字段，例如，我们不想通过网络传输敏感信息。**为此，只需在特定字段上使用@AvroIgnore注解即可**：

```java
record BankAccountWithIgnoredField(
        String bankAccountNumber,
        @AvroIgnore String reference
) {
}
```

因此，生成的模式将与我们第一个示例中的模式相匹配。

### 4.4 覆盖字段名称

默认情况下，生成的模式中的字段名称直接来自Java字段名，虽然这是默认行为，但可以进行调整：

```java
record BankAccountWithOverriddenField(
        String bankAccountNumber,
        @AvroName("bankAccountReference") String reference
) {
}
```

**从我们的记录的此版本生成的模式使用bankAccountReference而不是reference**：

```json
{
    "type" : "record",
    "name" : "BankAccountWithOverriddenField",
    "namespace" : "cn.tuyucheng.taketoday.apache.avro.model",
    "fields" : [ {
        "name" : "bankAccountNumber",
        "type" : "string"
    }, {
        "name" : "bankAccountReference",
        "type" : "string"
    } ]
}
```

### 4.5 具有多种实现的字段

有时，我们的类可能包含一个类型为子类型的字段。

我们假设AccountReference是一个具有两个实现的接口-为了简洁起见，我们可以坚持使用Java记录：

```java
interface AccountReference {
    String reference();
}

record PersonalBankAccountReference(
        String reference,
        String holderName
) implements AccountReference {
}

record BusinessBankAccountReference(
        String reference,
        String businessEntityId
) implements AccountReference {
}
```

在我们的BankAccountWithAbstractField中，我们使用@org.apache.avro.reflect.Union注解指示AccountReference字段支持的实现：

```java
record BankAccountWithAbstractField(
        String bankAccountNumber,
        @Union({ PersonalBankAccountReference.class, BusinessBankAccountReference.class })
        AccountReference reference
) {
}
```

**因此，生成的Avro模式将包含一个联合，允许分配这两个类中的任意一个，而不是限制我们只分配一个**：

```json
{
    "type" : "record",
    "name" : "BankAccountWithAbstractField",
    "namespace" : "cn.tuyucheng.taketoday.apache.avro.model",
    "fields" : [ {
        "name" : "bankAccountNumber",
        "type" : "string"
    }, {
        "name" : "reference",
        "type" : [ {
            "type" : "record",
            "name" : "PersonalBankAccountReference",
            "namespace" : "cn.tuyucheng.taketoday.apache.avro.model.BankAccountWithAbstractField",
            "fields" : [ {
                "name" : "holderName",
                "type" : "string"
            }, {
                "name" : "reference",
                "type" : "string"
            } ]
        }, {
            "type" : "record",
            "name" : "BusinessBankAccountReference",
            "namespace" : "cn.tuyucheng.taketoday.apache.avro.model.BankAccountWithAbstractField",
            "fields" : [ {
                "name" : "businessEntityId",
                "type" : "string"
            }, {
                "name" : "reference",
                "type" : "string"
            } ]
        } ]
    } ]
}
```

### 4.6 逻辑类型

Avro支持逻辑类型，**这些是模式级别的原始类型，但包含额外的提示，用于告诉代码生成器应该使用哪个类来表示特定字段**。

例如，如果我们的模型使用时间字段或UUID，我们可以利用逻辑类型功能：

```java
record BankAccountWithLogicalTypes(
        String bankAccountNumber,
        UUID reference,
        LocalDateTime expiryDate
) {
}
```

此外，我们将配置ReflectData实例，添加所需的Conversion对象。我们可以创建自己的Conversion对象，也可以使用系统自带的Conversion对象：

```java
@Test
void whenConvertingRecordWithLogicalTypes_thenAvroSchemaIsCorrect() {
    ReflectData reflectData = ReflectData.get();
    reflectData.addLogicalTypeConversion(new Conversions.UUIDConversion());
    reflectData.addLogicalTypeConversion(new TimeConversions.LocalTimestampMillisConversion());

    String jsonSchema = reflectData.getSchema(BankAccountWithLogicalTypes.class).toString();
  
    // verify schema
}
```

因此，当我们生成并验证模式时，我们会注意到新字段将包含一个logicalType字段：

```json
{
    "type" : "record",
    "name" : "BankAccountWithLogicalTypes",
    "namespace" : "cn.tuyucheng.taketoday.apache.avro.model",
    "fields" : [ {
        "name" : "bankAccountNumber",
        "type" : "string"
    }, {
        "name" : "expiryDate",
        "type" : {
            "type" : "long",
            "logicalType" : "local-timestamp-millis"
        }
    }, {
        "name" : "reference",
        "type" : {
            "type" : "string",
            "logicalType" : "uuid"
        }
    } ]
}
```

## 5. 使用Jackson生成Avro Schema

尽管Avro反射API很有用，并且应该能够满足不同的甚至复杂的需求，但了解替代方案总是值得的。

**在我们的例子中，我们刚刚试验过的库的替代方案是Jackson Dataformats Binary库，特别是其[与Avro相关的子模块](https://github.com/FasterXML/jackson-dataformats-binary/tree/2.18/avro)**。

首先，让我们将[jackson-core](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core)和[jackson-dataformat-avro](https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat/jackson-dataformat-avro/2.18.2)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.17.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-avro</artifactId>
    <version>2.17.2</version>
</dependency>
```

### 5.1 简单转换

让我们通过编写一个简单的转换器来探索[Jackson](https://www.baeldung.com/jackson-object-mapper-tutorial)的功能，此实现的优势在于它使用了众所周知的Java API。事实上，Jackson是最广泛使用的库之一，而直接使用的Avro API则相对较少。

我们将创建AvroMapper和AvroSchemaGenerator实例并使用它们来检索org.apache.avro.Schema实例。

从那里，我们只需调用toString()方法，就像前面的例子一样：

```java
@Test
void whenConvertingRecord_thenAvroSchemaIsCorrect() throws JsonMappingException {
    AvroMapper avroMapper = new AvroMapper();
    AvroSchemaGenerator avroSchemaGenerator = new AvroSchemaGenerator();

    avroMapper.acceptJsonFormatVisitor(SimpleBankAccount.class, avroSchemaGenerator);
    Schema schema = avroSchemaGenerator.getGeneratedSchema().getAvroSchema();
    String jsonSchema = schema.toString();

    assertThatJson(jsonSchema).isEqualTo("""
            {
                "type" : "record",
                "name" : "SimpleBankAccount",
                "namespace" : "cn.tuyucheng.taketoday.apache.avro.model",
                "fields" : [ {
                    "name" : "bankAccountNumber",
                    "type" : [ "null", "string" ]
                } ]
            }
            """);
}
```

### 5.2 Jackson注解

如果我们比较为SimpleBankAccount生成的两个模式，我们会注意到一个关键的区别：使用Jackson生成的模式将bankAccountNumber字段标记为可空，这是因为Jackson的工作方式与Avro反射不同。

**Jackson不太依赖反射，为了能够识别要迁移到模式的字段，它要求类具有访问器**。此外，需要注意的是，默认行为假定该字段可空。**如果我们不希望该字段在模式中可空，则需要使用@JsonProperty(required = true)注解**。

让我们创建该类的不同变体并利用此注解：

```java
record JacksonBankAccountWithRequiredField(
        @JsonProperty(required = true) String bankAccountNumber
) {
}
```

由于应用于原始Java类的所有Jackson注解仍然有效，因此我们需要仔细检查转换的结果。

### 5.3 逻辑类型感知转换器

**Jackson与Avro反射类似，默认情况下不考虑逻辑类型**，因此，我们需要显式启用此功能。让我们通过对AvroMapper和AvroSchemaGenerator对象进行一些小的调整来实现这一点：

```java
@Test
void whenConvertingRecordWithRequiredField_thenAvroSchemaIsCorrect() throws JsonMappingException {
    AvroMapper avroMapper = AvroMapper.builder()
            .addModule(new AvroJavaTimeModule())
            .build();

    AvroSchemaGenerator avroSchemaGenerator = new AvroSchemaGenerator()
            .enableLogicalTypes();

    avroMapper.acceptJsonFormatVisitor(BankAccountWithLogicalTypes.class, avroSchemaGenerator);
    Schema schema = avroSchemaGenerator.getGeneratedSchema()
            .getAvroSchema();
    String jsonSchema = schema.toString();

    // verify schema
}
```

通过这些修改，我们将能够观察在为Temporal对象生成的Avro模式中使用的逻辑类型特征。

## 6. 总结

在本文中，我们展示了几种从现有Java类生成Avro模式的方法，你可以使用标准的Avro反射API，也可以使用Jackson及其二进制Avro模块。

**尽管Avro的方式及其API不太为大众所知，但它似乎是一种比使用Jackson更可预测的解决方案，如果将其纳入我们正在进行的主要项目中，很容易导致错误**。