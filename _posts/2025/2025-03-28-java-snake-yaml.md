---
layout: post
title:  使用SnakeYAML解析YAML
category: libraries
copyright: libraries
excerpt: SnakeYAML
---

## 1. 概述

在本教程中，我们将学习如何使用SnakeYAML库**将Java对象序列化为YAML文档，反之亦然**。

## 2. 项目设置

为了在我们的项目中使用[SnakeYAML](https://bitbucket.org/snakeyaml/snakeyaml/src/master/)，我们将添加以下Maven依赖(最新版本可在[此处](https://mvnrepository.com/artifact/org.yaml/snakeyaml)找到)：

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>2.2</version>            
</dependency>
```

## 3. 入口点

Yaml类是API的入口点：

```java
Yaml yaml = new Yaml();
```

由于该实现不是线程安全的，因此不同的线程必须有自己的Yaml实例。

## 4. 加载YAML文档

该库支持从String或InputStream加载文档，此处的大部分代码示例都基于对InputStream的解析。

让我们首先定义一个简单的YAML文档，并将该文件命名为customer.yaml：

```yaml
firstName: "John"
lastName: "Doe"
age: 20
```

### 4.1 基本用法

现在我们将使用Yaml类解析上述YAML文档：

```java
Yaml yaml = new Yaml();
InputStream inputStream = this.getClass()
    .getClassLoader()
    .getResourceAsStream("customer.yaml");
Map<String, Object> obj = yaml.load(inputStream);
System.out.println(obj);
```

上述代码生成以下输出：

```text
{firstName=John, lastName=Doe, age=20}
```

load()方法默认返回一个Map实例，每次查询Map对象都需要我们提前知道属性键名，而且遍历嵌套属性也不是一件容易的事情。

### 4.2 自定义类型

**该库还提供了一种将文档作为自定义类加载的方法**，此选项可以轻松遍历内存中的数据。

让我们定义一个Customer类并尝试再次加载文档：

```java
public class Customer {

    private String firstName;
    private String lastName;
    private int age;

    // getters and setters
}
```

假设要反序列化的YAML文档为已知类型，我们可以在文档中指定明确的全局标签。

让我们更新文档并将其存储在新文件customer_with_type.yaml中：

```text
!!cn.tuyucheng.taketoday.snakeyaml.Customer
firstName: "John"
lastName: "Doe"
age: 20
```

请注意文档中的第一行，它包含有关加载时要使用的类的信息。

现在我们将更新上面使用的代码，并将新文件名作为输入传递：

```java
Yaml yaml = new Yaml();
InputStream inputStream = this.getClass()
    .getClassLoader()
    .getResourceAsStream("yaml/customer_with_type.yaml");
Customer customer = yaml.load(inputStream);
```

load()方法现在返回Customer类型的实例，**这种方法的缺点是必须将该类型导出为库才能在需要时使用**。

尽管如此，我们可以使用不需要导出库的显式本地标签。

**加载自定义类型的另一种方法是使用Constructor类**，这样，我们可以指定要解析的YAML文档的根类型。让我们创建一个以Customer类型为根类型的Constructor实例，并将其传递给Yaml实例。

现在加载customer.yaml，我们将获取Customer对象：

```java
Yaml yaml = new Yaml(new Constructor(Customer.class));
```

### 4.3 隐式类型

**如果没有为给定的属性定义类型，则库会自动将值转换为隐式类型**。

例如：

```text
1.0 -> Float
42 -> Integer
2009-03-30 -> Date
```

让我们使用测试用例来测试这种隐式类型转换：

```java
@Test
public void whenLoadYAML_thenLoadCorrectImplicitTypes() {
   Yaml yaml = new Yaml();
   Map<Object, Object> document = yaml.load("3.0: 2018-07-22");
 
   assertNotNull(document);
   assertEquals(1, document.size());
   assertTrue(document.containsKey(3.0d));   
}
```

### 4.4 嵌套对象和集合

**给定一个顶级类型，库会自动检测嵌套对象的类型(除非它们是接口或抽象类)**，并将文档反序列化为相关的嵌套类型。

让我们将Contact和Address详细信息添加到customer.yaml，并将新文件保存为customer_with_contact_details_and_address.yaml。 

现在我们将解析新的YAML文档：

```yaml
firstName: "John"
lastName: "Doe"
age: 31
contactDetails:
   - type: "mobile"
     number: 123456789
   - type: "landline"
     number: 456786868
homeAddress:
   line: "Xyz, DEF Street"
   city: "City Y"
   state: "State Y"
   zip: 345657
```

Customer类也应该反映这些变化，以下是更新后的类：

```java
public class Customer {
    private String firstName;
    private String lastName;
    private int age;
    private List<Contact> contactDetails;
    private Address homeAddress;    
    // getters and setters
}
```

让我们看看Contact和Address类是什么样子的：

```java
public class Contact {
    private String type;
    private int number;
    // getters and setters
}
```

```java
public class Address {
    private String line;
    private String city;
    private String state;
    private Integer zip;
    // getters and setters
}
```

现在我们将使用给定的测试用例测试YAML#load()：

```java
@Test
public void whenLoadYAMLDocumentWithTopLevelClass_thenLoadCorrectJavaObjectWithNestedObjects() {
    Yaml yaml = new Yaml(new Constructor(Customer.class, new LoaderOptions()));
    InputStream inputStream = this.getClass()
            .getClassLoader()
            .getResourceAsStream("yaml/customer_with_contact_details_and_address.yaml");
    Customer customer = yaml.load(inputStream);

    assertNotNull(customer);
    assertEquals("John", customer.getFirstName());
    assertEquals("Doe", customer.getLastName());
    assertEquals(31, customer.getAge());
    assertNotNull(customer.getContactDetails());
    assertEquals(2, customer.getContactDetails().size());

    assertEquals("mobile", customer.getContactDetails()
            .get(0)
            .getType());
    assertEquals(123456789, customer.getContactDetails()
            .get(0)
            .getNumber());
    assertEquals("landline", customer.getContactDetails()
            .get(1)
            .getType());
    assertEquals(456786868, customer.getContactDetails()
            .get(1)
            .getNumber());
    assertNotNull(customer.getHomeAddress());
    assertEquals("Xyz, DEF Street", customer.getHomeAddress()
            .getLine());
}
```

### 4.5 类型安全集合

当给定Java类的一个或多个属性是类型安全(泛型)集合时，指定TypeDescription以便识别正确的参数化类型非常重要。

假设一个Customer有多个Contact，并尝试加载它：

```yaml
firstName: "John"
lastName: "Doe"
age: 31
contactDetails:
   - { type: "mobile", number: 123456789}
   - { type: "landline", number: 123456789}
```

为了加载该文档，**我们可以在顶级类中为给定的属性指定TypeDescription**：

```java
Constructor constructor = new Constructor(Customer.class);
TypeDescription customTypeDescription = new TypeDescription(Customer.class);
customTypeDescription.addPropertyParameters("contactDetails", Contact.class);
constructor.addTypeDescription(customTypeDescription);
Yaml yaml = new Yaml(constructor);
```

### 4.6 加载多个文档

在某些情况下，单个文件中有多个YAML文档，我们想要解析所有这些文档。Yaml类提供了loadAll()方法来执行此类解析。

默认情况下，该方法返回Iterable<Object\>的实例，其中每个对象都是Map<String, Object\>类型。如果需要自定义类型，那么我们可以使用上面讨论的构造函数实例。 

考虑单个文件中的以下文档：

```yaml
---
firstName: "John"
lastName: "Doe"
age: 20
---
firstName: "Jack"
lastName: "Jones"
age: 25
```

我们可以使用loadAll()方法解析上述内容，如下面的代码示例所示：

```java
@Test
public void whenLoadMultipleYAMLDocuments_thenLoadCorrectJavaObjects() {
    Yaml yaml = new Yaml(new Constructor(Customer.class, new LoaderOptions()));
    InputStream inputStream = this.getClass()
            .getClassLoader()
            .getResourceAsStream("yaml/customers.yaml");

    int count = 0;
    for (Object object : yaml.loadAll(inputStream)) {
        count++;
        assertTrue(object instanceof Customer);
    }
    assertEquals(2,count);
}
```

## 5. 转储YAML文档

该库还提供了一种将给定的Java对象转储到YAML文档中的方法，输出可以是字符串或指定的文件/流。

### 5.1 基本用法

我们将从一个简单的示例开始，将Map<String, Object\>的实例转储到YAML文档(String)：

```java
@Test
public void whenDumpMap_thenGenerateCorrectYAML() {
    Map<String, Object> data = new LinkedHashMap<String, Object>();
    data.put("name", "Silenthand Olleander");
    data.put("race", "Human");
    data.put("traits", new String[] { "ONE_HAND", "ONE_EYE" });
    Yaml yaml = new Yaml();
    StringWriter writer = new StringWriter();
    yaml.dump(data, writer);
    String expectedYaml = "name: Silenthand Olleander\nrace: Human\ntraits: [ONE_HAND, ONE_EYE]\n";

    assertEquals(expectedYaml, writer.toString());
}
```

上述代码产生以下输出(请注意，使用LinkedHashMap的实例会保留输出数据的顺序)：

```text
name: Silenthand Olleander
race: Human
traits: [ONE_HAND, ONE_EYE]
```

### 5.2 自定义Java对象

我们还可以选择**将自定义Java类型转储到输出流中**，但是，这会将全局显式标记添加到输出文档中：

```java
@Test
public void whenDumpACustomType_thenGenerateCorrectYAML() {
    Customer customer = new Customer();
    customer.setAge(45);
    customer.setFirstName("Greg");
    customer.setLastName("McDowell");
    Yaml yaml = new Yaml();
    StringWriter writer = new StringWriter();
    yaml.dump(customer, writer);        
    String expectedYaml = "!!cn.tuyucheng.taketoday.snakeyaml.Customer {age: 45, contactDetails: null, firstName: Greg,\n  homeAddress: null, lastName: McDowell}\n";

    assertEquals(expectedYaml, writer.toString());
}
```

采用上述方法，我们仍然将标签信息转储到YAML文档中。

这意味着我们必须将我们的类导出为库，以供任何反序列化它的消费者使用。为了避免输出文件中出现标签名称，我们可以使用该库提供的dumpAs()方法。

因此，在上面的代码中，我们可以调整以下内容以删除标签：

```java
yaml.dumpAs(customer, Tag.MAP, null);
```

## 6. 总结

本文阐述了如何使用SnakeYAML库将Java对象序列化为YAML，反之亦然。