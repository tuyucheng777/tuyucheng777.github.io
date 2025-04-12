---
layout: post
title:  使用对象列表创建Avro Schema
category: apache
copyright: apache
excerpt: Avro
---

## 1. 概述

[Apache Avro](https://www.baeldung.com/java-apache-avro)是一个数据序列化框架，它提供强大的数据结构和轻量级、快速的二进制数据格式。

在本教程中，我们将探讨如何创建一个Avro模式，当它转换为对象时，会包含其他对象的列表。

## 2. 目标

**假设我们要开发一个表示父子关系的Avro模式**，因此，我们需要一个包含Child对象列表的Parent类。

Java代码如下所示：

```java
public class Child {
    String name;
}

public class Parent {
    List<Child> children;
}
```

**我们的目标是创建一个Avro模式，将其结构转换为这些对象**。

在我们研究解决方案之前，让我们快速回顾一下Avro的一些基础知识：

- [Avro模式使用JSON](https://www.baeldung.com/java-json/)定义
- type字段指的是数据类型(例如，记录，数组，字符串)
- fields数组定义记录的结构

## 3. 创建Avro Schema

为了正确说明Avro中的父子关系，我们需要使用[record](https://www.baeldung.com/java-record-keyword/)和array类型的组合。

该模式如下所示：

```json
{
    "namespace": "cn.tuyucheng.taketoday.apache.avro.generated",
    "type": "record",
    "name": "Parent",
    "fields": [
        {
            "name": "children",
            "type": {
                "type": "array",
                "items": {
                    "type": "record",
                    "name": "Child",
                    "fields": [
                        {"name": "name", "type": "string"}
                    ]
                }
            }
        }
    ]
}
```

我们首先定义了一个Parent类型的记录，在Parent记录中，我们定义了一个children字段，该字段是一个[数组](https://www.baeldung.com/java-arrays-guide)类型，允许我们存储多个Child对象。数组类型的items属性详细说明了数组中每个元素部分的结构，在我们的例子中，这是一个Child记录。如我们所见，Child记录只有一个字符串类型的属性name。 

## 4. 在Java中使用Schema

**定义好Avro模式后，我们将使用它来生成Java类**，当然，我们会使用Avro Maven插件来完成这项工作，以下是父pom文件中的配置：

```xml
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>1.11.3</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>schema</goal>
            </goals>
            <configuration>
                <sourceDirectory>src/main/java/cn/tuyucheng/taketoday/apache/avro/schemas</sourceDirectory>
                <outputDirectory>src/main/java</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

为了让Avro生成我们的类，我们需要运行Maven生成源命令(mvn clean generate-sources)或转到Maven工具窗口的插件部分并运行avro插件的avro：schema目标：

![](/assets/images/2025/apache/avroschemalistobjects01.png)

这样，Avro就会根据提供的schema在提供的namespace中创建Java类，namespace属性还会在生成的类顶部添加包名称。

## 5. 使用生成的类

**新创建的类提供了设置方法，用于获取和设置子列表**，如下所示：

```java
@Test
public void whenAvroSchemaWithListOfObjectsIsUsed_thenObjectsAreSuccessfullyCreatedAndSerialized() throws IOException {
    Parent parent = new Parent();
    List<Child> children = new ArrayList();

    Child child1 = new Child();
    child1.setName("Alice");
    children.add(child1);

    Child child2 = new Child();
    child2.setName("Bob");
    children.add(child2);

    parent.setChildren(children);

    SerializationDeserializationLogic.serializeParent(parent);
    Parent deserializedParent = SerializationDeserializationLogic.deserializeParent();

    assertEquals("Alice", deserializedParent.getChildren().get(0).getName());
}
```

从上面的测试中，我们看到我们创建了一个新的Parent，我们可以这样做，或者使用builder()函数。这篇关于[Avro默认值](https://www.baeldung.com/java-avro-default-values)的文章演示了如何使用builder()模式。

然后，我们创建两个Child对象，并将它们添加到Parent的children属性中。最后，我们对该对象进行序列化和反序列化，并比较其中一个名称。

## 6. 总结

在本文中，我们研究了如何创建包含对象列表的Avro模式。此外，我们还详细介绍了如何定义带有子记录列表属性的父记录，这是一种在Avro中表示复杂数据结构的方法。此外，当我们需要处理对象集合或分层数据时，这种方法尤其有用。

最后，Avro模式非常灵活，我们可以配置它们来设置更复杂的数据结构。我们可以组合不同的类型和嵌套结构来复制我们的数据模型。