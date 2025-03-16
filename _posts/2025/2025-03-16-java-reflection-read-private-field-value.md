---
layout: post
title:  从Java中的另一个类读取私有字段的值
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在本快速教程中，我们将讨论如何从Java中的不同类访问[私有](https://www.baeldung.com/java-private-keyword)字段的值。

在开始本教程之前，我们需要了解private访问修饰符可以防止意外滥用字段。但是，如果我们想访问它们，可以使用[反射](https://www.baeldung.com/java-reflection)API来实现。

## 2. 示例

让我们定义一个具有一些私有字段的示例类Person：

```java
public class Person {

    private String name = "John";
    private byte age = 30;
    private short uidNumber = 5555;
    private int pinCode = 452002;
    private long contactNumber = 123456789L;
    private float height = 6.1242f;
    private double weight = 75.2564;
    private char gender = 'M';
    private boolean active = true;

    // getters and setters
}
```

## 3. 使私有字段可访问

为了使任何私有字段可访问，**我们必须调用Field#setAccessible方法**：

```java
Person person = new Person(); 
Field nameField = person.getClass().getDeclaredField("name"); 
nameField.setAccessible(true);
```

在上面的例子中，我们首先使用**Class#getDeclaredField**方法指定要检索的字段-name。然后我们使用nameField.setAccessible(true)使该字段可访问。

## 4. 访问私有原始类型字段

我们可以**使用Field#getXxx方法访问原始类型的私有字段**。

### 4.1 访问整数字段

我们可以使用getByte、getShort、getInt和getLong方法分别访问byte、short、int和long字段：

```java
@Test
public void whenGetIntegerFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field ageField = person.getClass().getDeclaredField("age");
    ageField.setAccessible(true);

    byte age = ageField.getByte(person);
    Assertions.assertEquals(30, age);

    Field uidNumberField = person.getClass().getDeclaredField("uidNumber");
    uidNumberField.setAccessible(true);

    short uidNumber = uidNumberField.getShort(person);
    Assertions.assertEquals(5555, uidNumber);

    Field pinCodeField = person.getClass().getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    int pinCode = pinCodeField.getInt(person);
    Assertions.assertEquals(452002, pinCode);

    Field contactNumberField = person.getClass().getDeclaredField("contactNumber");
    contactNumberField.setAccessible(true);

    long contactNumber = contactNumberField.getLong(person);
    Assertions.assertEquals(123456789L, contactNumber);
}
```

**也可以对原始类型执行[自动装箱](https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)**：

```java
@Test
public void whenDoAutoboxing_thenSuccess() throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass().getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    Integer pinCode = pinCodeField.getInt(person);
    Assertions.assertEquals(452002, pinCode);
}
```

**原始数据类型的getXxx方法也支持[扩展](https://www.baeldung.com/java-primitive-conversions#widening-primitive-conversions)**：

```java
@Test
public void whenDoWidening_thenSuccess() throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass().getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    Long pinCode = pinCodeField.getLong(person);
    Assertions.assertEquals(452002L, pinCode);
}
```

### 4.2 访问浮点类型字段

要访问float和double字段，我们需要分别使用getFloat和getDouble方法：

```java
@Test
public void whenGetFloatingTypeFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field heightField = person.getClass().getDeclaredField("height");
    heightField.setAccessible(true);

    float height = heightField.getFloat(person);
    Assertions.assertEquals(6.1242f, height);
    
    Field weightField = person.getClass().getDeclaredField("weight");
    weightField.setAccessible(true);

    double weight = weightField.getDouble(person);
    Assertions.assertEquals(75.2564, weight);
}
```

### 4.3 访问字符字段

要访问char字段，我们可以使用getChar方法：

```java
@Test
public void whenGetCharacterFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field genderField = person.getClass().getDeclaredField("gender");
    genderField.setAccessible(true);

    char gender = genderField.getChar(person);
    Assertions.assertEquals('M', gender);
}
```

### 4.4 访问布尔字段

类似地，我们可以使用getBoolean方法来访问布尔字段：

```java
@Test
public void whenGetBooleanFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field activeField = person.getClass().getDeclaredField("active");
    activeField.setAccessible(true);

    boolean active = activeField.getBoolean(person);
    Assertions.assertTrue(active);
}
```

## 5. 访问对象的私有字段

我们可以**使用Field#get方法访问对象的私有字段**，需要注意的是，**通用get方法返回一个Object，因此我们需要将其转换为目标类型才能使用该值**：

```java
@Test
public void whenGetObjectFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field nameField = person.getClass().getDeclaredField("name");
    nameField.setAccessible(true);

    String name = (String) nameField.get(person);
    Assertions.assertEquals("John", name);
}
```

## 6. 异常

现在，让我们讨论一下JVM在访问私有字段时可能抛出的异常。

### 6.1 IllegalArgumentException

**如果我们使用的getXxx访问器与目标字段的类型不兼容，JVM将抛出IllegalArgumentException**。在我们的示例中，如果我们写入nameField.getInt(person)，JVM将抛出此异常，因为该字段的类型为String而不是int或Integer：

```java
@Test
public void givenInt_whenSetStringField_thenIllegalArgumentException() throws Exception {
    Person person = new Person();
    Field nameField = person.getClass().getDeclaredField("name");
    nameField.setAccessible(true);

    Assertions.assertThrows(IllegalArgumentException.class, () -> nameField.getInt(person));
}
```

正如我们已经看到的，getXxx方法支持扩展原始类型。需要注意的是，我们需要提供正确的目标才能成功扩展。否则，JVM会抛出IllegalArgumentException：

```java
@Test
public void givenInt_whenGetLongField_thenIllegalArgumentException() throws Exception {
    Person person = new Person();
    Field contactNumberField = person.getClass().getDeclaredField("contactNumber");
    contactNumberField.setAccessible(true);

    Assertions.assertThrows(IllegalArgumentException.class, () -> contactNumberField.getInt(person));
}
```

### 6.2 IllegalAccessException

如果我们尝试访问没有访问权限的字段，JVM将抛出IllegalAccessException。在上面的例子中，如果我们没有编写语句nameField.setAccessible(true)，则JVM将抛出异常：

```java
@Test
public void whenFieldNotSetAccessible_thenIllegalAccessException() throws Exception {
    Person person = new Person();
    Field nameField = person.getClass().getDeclaredField("name");

    Assertions.assertThrows(IllegalAccessException.class, () -> nameField.get(person));
}
```

### 6.3 NoSuchFieldException

**如果我们尝试访问Person类中不存在的字段，那么JVM可能会抛出NoSuchFieldException**：

```java
Assertions.assertThrows(NoSuchFieldException.class,
    () -> person.getClass().getDeclaredField("firstName"));
```

### 6.4 NullPointerException

最后，正如你所期望的，**如果我们将字段名称传递为null，JVM将抛出NullPointerException**：

```java
Assertions.assertThrows(NullPointerException.class,
    () -> person.getClass().getDeclaredField(null));
```

## 7. 总结

在本教程中，我们了解了如何访问一个类中另一个类的私有字段，并了解了JVM可能抛出的异常及其原因。