---
layout: post
title:  使用反射从Java类中检索字段
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在本教程中，我们将使用[反射](https://www.baeldung.com/java-reflection)检查Java应用程序的运行时结构。具体来说，我们将使用Java反射API。首先，我们检查Java类的字段，然后查看该类从超类继承的字段。

## 2. 从Java类中检索字段

让我们看看如何检索类的字段，无论它们的可见性如何。

现在，假设我们有一个Person类，它有两个字段，分别表示一个人的lastName和firstName：

```java
public class Person {
    protected String lastName;
    private String firstName;
}
```

因此，我们有protected和private字段，这些字段不会对其他类显式可见。不过，**我们可以使用Class::getDeclaredFields方法获取所有声明的字段**，getDeclaredFields()方法返回一个Field对象数组以及字段的所有元数据：

```java
List<Field> allFields = Arrays.asList(Person.class.getDeclaredFields());

assertEquals(2, allFields.size());
Field lastName = allFields.stream()
    .filter(field -> field.getName().equals(LAST_NAME_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found"));
assertEquals(String.class, lastName.getType());
Field firstName = allFields.stream()
    .filter(field -> field.getName().equals(FIRST_NAME_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found"));
assertEquals(String.class, firstName.getType());
```

为了测试该方法，我们只需根据字段名称过滤返回的数组并检查它们的类型。

## 3. 检索继承的字段

现在让我们看看如何获取Java类的继承字段。

让我们用额外的public字段扩展Person类并将其命名为Employee：

```java
public class Employee extends Person {
    public static final String LABEL = "employee";
    public int employeeId;
}
```

### 3.1 在简单类层次结构中检索继承的字段

类似地，在Employee类上调用getDeclaredFields()将仅返回employeeId和LABEL字段，但是我们想要Person超类的字段。当然，我们可以在Person和Employee类上使用getDeclaredFields()方法，并将它们的结果合并到一个数组中。但是，如果我们不想明确指定超类怎么办？在这种情况下，我们可以使用Java反射API方法Class::getSuperclass()。

**getSuperClass()方法返回我们类的超类，而无需明确引用超类的名称**。现在，我们可以从超类中获取所有字段并将它们合并到一个数组中：

```java
List<Field> personFields = Arrays.asList(Employee.class.getSuperclass().getDeclaredFields());
List<Field> employeeFields = Arrays.asList(Employee.class.getDeclaredFields());
List<Field> allFields = Stream.concat(personFields.stream(), employeeFields.stream())
    .collect(Collectors.toList());

assertEquals(4, allFields.size());
Field lastNameField = allFields.stream()
    .filter(field -> field.getName().equals(LAST_NAME_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found"));
assertEquals(String.class, lastNameField.getType());
Field firstNameField = allFields.stream()
    .filter(field -> field.getName().equals(FIRST_NAME_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found"));
assertEquals(String.class, firstNameField.getType());
Field employeeIdField = allFields.stream()
    .filter(field -> field.getName().equals(EMPLOYEE_ID_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found"));
assertEquals(employeeIdField.getType(), int.class);
Field employeeTypeField = allFields.stream()
    .filter(field -> field.getName().equals(EMPLOYEE_TYPE_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found"));
assertEquals(String.class, employeeTypeField.getType());

```

我们可以在这里看到收集了Person的两个字段和Employee的两个字段。

反射是一种非常强大的工具，但应谨慎使用。例如，我们不建议使用类的私有字段，因为我们通常打算隐藏它们并阻止使用它们。我们只应使用可以继承的字段，例如公共和受保护的字段。

### 3.2 过滤Public和Protected字段

不幸的是，Java API中没有一种方法允许我们从类及其超类中仅获取公共和受保护的字段，Class::getFields()方法通过返回类及其超类的所有公共字段(但不返回受保护的字段)来实现我们的目标。因此，我们必须使用.getDeclaredFields()方法，并使用Field类上的getModifiers()方法过滤其结果。

**getModifiers()方法返回一个表示字段修饰符的int值，该值是2^0到2^7之间的整数**。具体来说，public是2^0，static是2^3。因此，在public和static字段上调用getModifiers()方法将返回9。

我们可以忽略实现细节，直接使用Modifiers类提供的辅助方法。具体来说，我们将分别使用.isPublic()和.isProtected()方法仅收集公共字段和受保护字段：

```java
List<Field> personFields = Arrays.stream(Employee.class.getSuperclass().getDeclaredFields())
    .filter(f -> Modifier.isPublic(f.getModifiers()) || Modifier.isProtected(f.getModifiers()))
    .collect(Collectors.toList());

assertEquals(1, personFields.size());

Field personField = personFields.stream()
    .filter(field -> field.getName().equals(LAST_NAME_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found"));
assertEquals(String.class, personField.getType());
```

在我们的测试中，我们过滤掉了除公共和受保护字段之外的所有字段。我们可以自由组合修饰符的方法来获取特定字段。例如，要仅获取Employee类的公共静态最终字段，我们必须过滤那些特定的修饰符：

```java
List<Field> publicStaticField = Arrays.stream(Employee.class.getDeclaredFields())
    .filter(field -> Modifier.isStatic(field.getModifiers()) && Modifier.isPublic(field.getModifiers()) && Modifier.isFinal(field.getModifiers()))
    .collect(Collectors.toList());

assertEquals(1, publicStaticField.size());
Field employeeTypeField = publicStaticField.get(0);
assertEquals(EMPLOYEE_TYPE_FIELD, employeeTypeField.getName());
```

### 3.3 在深层类层次结构中检索继承的字段

到目前为止，我们一直在处理单个类层次结构。如果我们有更深的类层次结构并想要收集所有继承的字段，该怎么办？

例如，如果我们有一个Employee的子类或一个Person的超类，则获取整个层次结构的字段还需要检查超类的超类。

为此，我们可以创建一个贯穿整个层次结构的实用方法，为我们构建完整的结果：

```java
List<Field> getAllFields(Class clazz) {
    if (clazz == null) {
        return Collections.emptyList();
    }

    List<Field> result = new ArrayList<>(getAllFields(clazz.getSuperclass()));
    List<Field> filteredFields = Arrays.stream(clazz.getDeclaredFields())
        .filter(f -> Modifier.isPublic(f.getModifiers()) || Modifier.isProtected(f.getModifiers()))
        .collect(Collectors.toList());
    result.addAll(filteredFields);
    return result;
}
```

此递归方法将通过类层次结构搜索公共和受保护的字段，并返回列表中找到的所有字段。

让我们通过对新的MonthEmployee类进行一个小测试来说明这一点，该类扩展了我们之前示例中的Employee类：

```java
public class MonthEmployee extends Employee {
    protected double reward;
}
```

这个类定义了一个新字段reward，考虑到所有的层次结构类，我们的方法应该给我们三个字段定义-Person::lastName、Employee::employeeId和MonthEmployee::reward。

我们在MonthEmployee上调用getAllFields()方法：

```java
List<Field> allFields = getAllFields(MonthEmployee.class);

assertEquals(4, allFields.size());

assertFalse(allFields.stream().anyMatch(field -> field.getName().equals(FIRST_NAME_FIELD)));
assertEquals(String.class, allFields.stream().filter(field -> field.getName().equals(LAST_NAME_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found")).getType());
assertEquals(int.class, allFields.stream().filter(field -> field.getName().equals(EMPLOYEE_ID_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found")).getType());
assertEquals(double.class, allFields.stream().filter(field -> field.getName().equals(MONTH_EMPLOYEE_REWARD_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found")).getType());
assertEquals(String.class, allFields.stream().filter(field -> field.getName().equals(EMPLOYEE_TYPE_FIELD))
    .findFirst().orElseThrow(() -> new RuntimeException("Field not found")).getType());
```

正如预期的那样，我们从类层次结构中收集了所有公共和受保护的字段。

## 4. 总结

在本文中，我们了解了如何使用Java反射API检索Java类的字段。

我们首先学习了如何检索类的已声明字段。之后，我们了解了如何检索其超类字段。然后，我们学习了如何过滤非公共和非受保护字段。

最后，我们看到了如何应用所有这些来收集多类层次结构的继承字段。