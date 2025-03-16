---
layout: post
title:  在Java中使用反射实例化内部类
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在本教程中，我们将讨论使用[反射API](https://www.baeldung.com/java-reflection)在Java中实例化[内部类或嵌套类](https://www.baeldung.com/java-nested-classes)。

**反射API在需要读取Java类的结构并动态实例化类的情况下尤为重要，具体场景包括扫描注解、使用Bean名称查找和实例化Java Bean等等**。一些流行的库(如Spring和Hibernate)和代码分析工具广泛使用它。

与普通类相比，实例化内部类更具挑战性。让我们进一步探索。

## 2. 内部类编译

要在内部类上使用Java反射API，我们必须了解编译器如何处理它。因此，作为示例，我们首先定义一个Person类，我们将使用它来演示内部类的实例化：

```java
public class Person {
    String name;
    Address address;

    public Person() {
    }

    public class Address {
        String zip;

        public Address(String zip) {
            this.zip = zip;
        }
    }

    public static class Builder {
    }
}
```

Person类有两个内部类，Address和Builder。Address类是非静态的，因为在现实世界中，地址通常与人的实例相关联。但是，Builder是静态的，因为它是创建Person实例所必需的。因此，在我们实例化Person类之前，它必须存在。

**编译器为内部类创建单独的类文件，而不是将它们嵌入到外部类中**。在本例中，我们看到编译器总共创建了三个类：

![](/assets/images/2025/javareflect/javareflectioninstantiateinnerclass01.png)

编译器生成了Person类，有趣的是，它还创建了两个名为Person\$Address和Person\$Builder的内部类。

下一步是找出内部类中的构造函数：

```java
@Test
void givenInnerClass_whenUseReflection_thenShowConstructors() {
    final String personBuilderClassName = "cn.tuyucheng.taketoday.reflection.innerclass.Person$Builder";
    final String personAddressClassName = "cn.tuyucheng.taketoday.reflection.innerclass.Person$Address";
    assertDoesNotThrow(() -> logConstructors(Class.forName(personAddressClassName)));
    assertDoesNotThrow(() -> logConstructors(Class.forName(personBuilderClassName)));
}

static void logConstructors(Class<?> clazz) {
    Arrays.stream(clazz.getDeclaredConstructors())
        .map(c -> formatConstructorSignature(c))
        .forEach(logger::info);
}

static String formatConstructorSignature(Constructor<?> constructor) {
    String params = Arrays.stream(constructor.getParameters())
        .map(parameter -> parameter.getType().getSimpleName() + " " + parameter.getName())
        .collect(Collectors.joining(", "));
    return constructor.getName() + "(" + params + ")";
}
```

Class.forName()接收内部类的完全限定名称并返回Class对象。此外，通过此Class对象，我们使用方法logConstructors()获取构造函数的详细信息：

```text
cn.tuyucheng.taketoday.reflection.innerclass.Person$Address(Person this$0, String zip)
cn.tuyucheng.taketoday.reflection.innerclass.Person$Builder()
```

**令人惊讶的是，在非静态Person\$Address类的构造函数中，编译器注入了this\$0，其中包含对封闭Person类的引用作为第一个参数。但静态类Person$Builder在构造函数中没有对外部类的引用**。

在实例化内部类时，我们会牢记Java编译器的这种行为。

## 3. 实例化静态内部类

**实例化静态内部类与使用方法[Class.forName(String className)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html#forName(java.lang.String))实例化任何普通类几乎类似**：

```java
@Test
void givenStaticInnerClass_whenUseReflection_thenInstantiate() throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException,
      InstantiationException, IllegalAccessException {
    final String personBuilderClassName = "cn.tuyucheng.taketoday.reflection.innerclass.Person$Builder";
    Class<Person.Builder> personBuilderClass = (Class<Person.Builder>) Class.forName(personBuilderClassName);
    Person.Builder personBuilderObj = personBuilderClass.getDeclaredConstructor().newInstance();
    assertTrue(personBuilderObj instanceof Person.Builder);
}
```

我们将内部类的完全限定名称“cn.tuyucheng.taketoday.reflection.innerclass.Person$Builder”传递给Class.forName()，然后我们在Person.Builder类的构造函数上调用[newInstance()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/Constructor.html#newInstance(java.lang.Object...))方法来获取personBuilderObj。

## 4. 实例化非静态内部类

正如我们之前看到的，**Java编译器将对封闭类的引用作为第一个参数注入到非静态内部类的构造函数中**。

有了这些知识，让我们尝试实例化Person.Address类：

```java
@Test
void givenNonStaticInnerClass_whenUseReflection_thenInstantiate() throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException,
      InstantiationException, IllegalAccessException {
    final String personClassName = "cn.tuyucheng.taketoday.reflection.innerclass.Person";
    final String personAddressClassName = "cn.tuyucheng.taketoday.reflection.innerclass.Person$Address";

    Class<Person> personClass = (Class<Person>) Class.forName(personClassName);
    Person personObj = personClass.getConstructor().newInstance();

    Class<Person.Address> personAddressClass = (Class<Person.Address>) Class.forName(personAddressClassName);

    assertThrows(NoSuchMethodException.class, () -> personAddressClass.getDeclaredConstructor(String.class));
    
    Constructor<Person.Address> constructorOfPersonAddress = personAddressClass.getDeclaredConstructor(Person.class, String.class);
    Person.Address personAddressObj = constructorOfPersonAddress.newInstance(personObj, "751003");
    assertTrue(personAddressObj instanceof Person.Address);
}
```

首先，我们创建了Person对象。然后，我们将内部类的完全限定名称“cn.tuyucheng.taketoday.reflection.innerclass.Person\$Address”传递给Class.forName()。接下来，我们从personAddressClass中获取构造函数Address(Person this\$0, String zip)。

最后，我们使用personObj和751003参数在构造函数上调用newInstance()方法来获取personAddressObj。

我们还看到方法personAddressClass.getDeclaredConstructor(String.class)由于缺少第一个参数this\$0而引发NoSuchMethodException。

## 5. 总结

在本文中，我们讨论了Java反射API来实例化静态和非静态内部类。我们发现编译器将内部类视为外部类，而不是外部类中的嵌入类。

此外，非静态内部类的构造函数默认将外部类对象作为第一个参数。但是，我们可以像任何普通类一样实例化静态类。