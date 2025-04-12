---
layout: post
title:  如何在Avro中处理默认值
category: apache
copyright: apache
excerpt: Apache Avro
---

## 1. 简介

在本教程中，我们将探索[Apache Avro](https://www.baeldung.com/java-apache-avro)数据序列化/反序列化框架。**此外，我们还将学习如何在初始化和序列化对象时使用默认值来处理模式定义**。

## 2. 什么是Avro？

Apache Avro是一种比传统数据格式化方式更强大的替代方案，它通常使用[JSON](https://www.baeldung.com/java-json)来定义模式。此外，Avro最常用的用例包括Apache Kafka、Hive或Impala，Avro非常适合实时处理大量数据(写入密集型、大数据操作)。

**我们将Avro视为由模式定义，并且该模式以JSON编写**。

Avro的优点是：

- 数据自动压缩(只需更少的CPU资源)
- 数据是完全类型化的(我们稍后会看到如何声明每个属性的类型)
- 模式伴随数据
- 文档嵌入在模式中
- 得益于JSON，可以用任何语言读取数据
- 安全模式演化

## 3. Avro设置

首先，让我们添加适当的[Avro](https://mvnrepository.com/artifact/org.apache.avro/avro) Maven依赖：

```xml
<dependencies> 
    <dependency> 
        <groupId>org.apache.avro</groupId> 
        <artifactId>avro</artifactId> 
        <version>1.11.3</version> 
    </dependency> 
</dependencies>
```

接下来，我们将配置[avro-maven-plugin](https://mvnrepository.com/artifact/org.apache.avro/avro-maven-plugin)来帮助我们生成代码：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.avro</groupId>
            <artifactId>avro-maven-plugin</artifactId>
            <version>1.11.3</version>
            <configuration>
                <sourceDirectory>${project.basedir}/src/main/java/cn/tuyucheng/taketoday/avro/</sourceDirectory>
                <outputDirectory>${project.basedir}/src/main/java/cn/tuyucheng/taketoday/avro/</outputDirectory>
                <stringType>String</stringType>
            </configuration>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>schema</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

现在让我们定义一个示例模式，Avro会使用它来生成示例类。该模式是一个JSON格式的对象定义，存储在一个文本文件中，我们必须确保该文件的扩展名为.avsc；在本例中，我们将该文件命名为car.avsc。

初始模式如下所示：

```json
{
    "namespace": "generated.avro",
     "type": "record",
     "name": "Car",
     "fields": [
         {  "name": "brand",
            "type": "string"
         },
         {  "name": "number_of_doors",
            "type": "int"
         },
         {  "name": "color",
            "type": "string"
         }
     ]
}
```

让我们更详细地看一下这个模式，namespace是生成的record类将被添加到的位置，[Record](https://www.baeldung.com/java-record-keyword)是一种特殊的Java类，它帮助我们用比普通类更少的样板代码来建模普通的数据聚合。总的来说，Avro支持六种[复杂类型](https://avro.apache.org/docs/1.11.1/specification/#complex-types)：记录(record)、枚举(enum)、数组(array)、映射(map)、联合(union)和固定(fixed)。

在我们的示例中，type是一个record，name是类的名称，fields是它的属性及其类型；**这里我们处理默认值**。

## 4. Avro默认值

**Avro的一个重要特性是，可以使用union将字段设为可选，在这种情况下，字段默认值为null；或者，当字段尚未初始化时，可以为其指定一个特定的默认值**。因此，我们要么拥有一个默认为null的可选字段，要么使用我们在模式中指定的默认值初始化该字段。

现在，让我们看一下配置默认值的新模式：

```json
{
    "namespace": "generated.avro",
    "type": "record",
    "name": "Car",
    "fields": [
        {   "name": "brand",
            "type": "string",
            "default": "Dacia"
        },
        {   "name": "number_of_doors",
            "type": "int",
            "default": 4
        },
        {   "name": "color",
            "type": ["null", "string"],
            "default": null
        }
    ]
}
```

我们看到属性有两种类型：String和int；我们还注意到，属性除了type之外还有一个default属性，这允许属性类型不进行初始化，而是默认为指定的值。

**为了在初始化对象时使用默认值，我们必须使用Avro生成类的newBuilder()方法**。正如我们在下面的测试中看到的，我们使用了构建器设计模式，并通过它初始化了必需的属性。

我们也来看一下测试：

```java
@Test
public void givenCarJsonSchema_whenCarIsSerialized_thenCarIsSuccessfullyDeserialized() throws IOException {
    Car car = Car.newBuilder()
        .build();

    SerializationDeserializationLogic.serializeCar(car);
    Car deserializedCar = SerializationDeserializationLogic.deserializeCar();

    assertEquals("Dacia", deserializedCar.getBrand());
    assertEquals(4, deserializedCar.getNumberOfDoors());
    assertNull(deserializedCar.getColor());
}
```

我们实例化了一个新的car对象，并仅设置了color属性，这也是唯一一个必需的属性。检查属性，我们发现brand初始化为Dacia，number_of_doors初始化为4(两者均已从模式中分配默认值)，color默认为null。

此外，在字段中添加可选语法(union)会强制其采用该值；因此，即使字段是int类型，默认值也将为null。当我们想要确保字段未被设置时，这很有用：

```json
{ 
    "name": "number_of_wheels", 
    "type": ["null", "int"], 
    "default": null 
}
```

## 5. 总结

Avro的创建是为了满足大数据处理环境中高效序列化的需求。

在本文中，我们了解了Apache的数据序列化/反序列化框架Avro。此外，我们还讨论了它的优势和设置方法，最重要的是，我们学习了如何配置模式以接收默认值。