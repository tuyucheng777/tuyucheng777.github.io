---
layout: post
title:  使用反射设置字段值
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在[上一篇文章](https://www.baeldung.com/java-reflection-read-private-field-value)中，我们讨论了如何从Java中的其他类读取私有字段的值。但是，在某些情况下，我们需要设置字段的值，例如在某些我们无权访问字段的库中。

在本快速教程中，我们将讨论如何使用[反射](https://www.baeldung.com/java-reflection)API设置Java中不同类的字段的值。

**请注意，我们将在这里的示例中使用与我们[上一篇文章](https://www.baeldung.com/java-reflection-read-private-field-value)中使用的相同的Person类**。

## 2. 设置原始类型字段

我们可以**使用Field#setXxx方法设置原始类型字段**。

### 2.1 设置整数字段

我们可以使用setByte、setShort、setInt和setLong方法分别设置byte、short、int和long字段：

```java
@Test
public void whenSetIntegerFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field ageField = person.getClass()
        .getDeclaredField("age");
    ageField.setAccessible(true);

    byte age = 26;
    ageField.setByte(person, age);
    Assertions.assertEquals(age, person.getAge());

    Field uidNumberField = person.getClass()
        .getDeclaredField("uidNumber");
    uidNumberField.setAccessible(true);

    short uidNumber = 5555;
    uidNumberField.setShort(person, uidNumber);
    Assertions.assertEquals(uidNumber, person.getUidNumber());

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    int pinCode = 411057;
    pinCodeField.setInt(person, pinCode);
    Assertions.assertEquals(pinCode, person.getPinCode());

    Field contactNumberField = person.getClass()
        .getDeclaredField("contactNumber");
    contactNumberField.setAccessible(true);

    long contactNumber = 123456789L;
    contactNumberField.setLong(person, contactNumber);
    Assertions.assertEquals(contactNumber, person.getContactNumber());
}
```

**也可以对原始类型进行[拆箱操作](https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)**：

```java
@Test
public void whenDoUnboxing_thenSuccess() throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    Integer pinCode = 411057;
    pinCodeField.setInt(person, pinCode);
    Assertions.assertEquals(pinCode, person.getPinCode());
}
```

**原始数据类型的setXxx方法也支持[向下兼容](https://www.baeldung.com/java-primitive-conversions#widening-primitive-conversions)**：

```java
@Test
public void whenDoNarrowing_thenSuccess() throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    short pinCode = 4110;
    pinCodeField.setInt(person, pinCode);
    Assertions.assertEquals(pinCode, person.getPinCode());
}
```

### 2.2 设置浮点类型字段

要设置float和double字段，我们需要分别使用setFloat和setDouble方法：

```java
@Test
public void whenSetFloatingTypeFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field heightField = person.getClass()
        .getDeclaredField("height");
    heightField.setAccessible(true);

    float height = 6.1242f;
    heightField.setFloat(person, height);
    Assertions.assertEquals(height, person.getHeight());

    Field weightField = person.getClass()
        .getDeclaredField("weight");
    weightField.setAccessible(true);

    double weight = 75.2564;
    weightField.setDouble(person, weight);
    Assertions.assertEquals(weight, person.getWeight());
}
```

### 2.3 设置字符字段

要设置字符字段，我们可以使用setChar方法：

```java
@Test
public void whenSetCharacterFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field genderField = person.getClass()
        .getDeclaredField("gender");
    genderField.setAccessible(true);

    char gender = 'M';
    genderField.setChar(person, gender);
    Assertions.assertEquals(gender, person.getGender());
}
```

### 2.4 设置布尔字段

类似地，**我们可以使用setBoolean方法来设置布尔字段**：

```java
@Test
public void whenSetBooleanFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field activeField = person.getClass()
        .getDeclaredField("active");
    activeField.setAccessible(true);

    activeField.setBoolean(person, true);
    Assertions.assertTrue(person.isActive());
}
```

## 3. 设置对象字段

我们可以使用Field#set方法设置对象字段：

```java
@Test
public void whenSetObjectFields_thenSuccess() throws Exception {
    Person person = new Person();

    Field nameField = person.getClass()
        .getDeclaredField("name");
    nameField.setAccessible(true);

    String name = "Umang Budhwar";
    nameField.set(person, name);
    Assertions.assertEquals(name, person.getName());
}
```

## 4. 异常

现在，让我们讨论一下JVM在设置字段时可能抛出的异常。

### 4.1 IllegalArgumentException

**如果我们使用的setXxx变量与目标字段的类型不兼容，JVM将抛出IllegalArgumentException**。在我们的示例中，如果我们写入nameField.setInt(person, 26)，JVM将抛出此异常，因为该字段的类型为String而不是int或Integer：

```java
@Test
public void givenInt_whenSetStringField_thenIllegalArgumentException() throws Exception {
    Person person = new Person();
    Field nameField = person.getClass()
        .getDeclaredField("name");
    nameField.setAccessible(true);

    Assertions.assertThrows(IllegalArgumentException.class, () -> nameField.setInt(person, 26));
}
```

正如我们已经看到的，setXxx方法支持原始类型的向下兼容。需要注意的是，**我们需要提供正确的目标才能成功缩小范围**。否则，JVM会抛出IllegalArgumentException：

```java
@Test
public void givenInt_whenSetLongField_thenIllegalArgumentException() throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    long pinCode = 411057L;

    Assertions.assertThrows(IllegalArgumentException.class, () -> pinCodeField.setLong(person, pinCode));
}
```

### 4.2 IllegalAccessException

**如果我们尝试设置没有访问权限的私有字段，那么JVM将抛出IllegalAccessException**。在上面的例子中，如果我们没有编写语句nameField.setAccessible(true)，那么JVM将抛出异常：

```java
@Test
public void whenFieldNotSetAccessible_thenIllegalAccessException() throws Exception {
    Person person = new Person();
    Field nameField = person.getClass()
        .getDeclaredField("name");

    Assertions.assertThrows(IllegalAccessException.class, () -> nameField.set(person, "Umang Budhwar"));
}
```

## 5. 总结

在本教程中，我们了解了如何在Java中修改或设置一个类中另一个类的私有字段的值，并介绍了JVM可能抛出的异常及其原因。