---
layout: post
title:  使用反射在运行时调用方法
category: java-new
copyright: java-new
excerpt: Java 11
---

## 1. 概述

在这篇简短的文章中，我们将快速了解如何**使用Java反射API在运行时调用方法**。

## 2. 准备

让我们创建一个简单的类，我们将在下面的示例中使用它：

```java
public class Operations {
    public double publicSum(int a, double b) {
        return a + b;
    }

    public static double publicStaticMultiply(float a, long b) {
        return a * b;
    }

    private boolean privateAnd(boolean a, boolean b) {
        return a && b;
    }

    protected int protectedMax(int a, int b) {
        return a > b ? a : b;
    }
}
```

## 3. 获取Method对象

首先，我们需要获取一个反映我们要调用的方法的Method对象。Class对象表示定义方法的类型，它提供了两种方法来执行此操作。

### 3.1 getMethod()

我们可以使用getMethod()来查找该类或其任何超类的任何公共方法。

基本上，它接收方法名称作为第一个参数，然后是方法参数的类型：

```java
Method sumInstanceMethod = Operations.class.getMethod("publicSum", int.class, double.class);

Method multiplyStaticMethod = Operations.class.getMethod("publicStaticMultiply", float.class, long.class);
```

### 3.2 getDeclaredMethod()

我们可以使用getDeclaredMethod()来获取任何类型的方法。这包括公共、受保护、默认访问，甚至私有方法，但不包括继承的方法。

它接收与getMethod()相同的参数：

```java
Method andPrivateMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);
```

```java
Method maxProtectedMethod = Operations.class.getDeclaredMethod("protectedMax", int.class, int.class);
```

## 4. 调用方法

有了Method实例，我们现在可以调用invoke()来执行底层方法并获取返回的对象。

### 4.1 实例方法

要调用实例方法，invoke()的第一个参数必须是反映被调用方法的Method实例：

```java
@Test
public void givenObject_whenInvokePublicMethod_thenCorrect() {
    Method sumInstanceMethod = Operations.class.getMethod("publicSum", int.class, double.class);

    Operations operationsInstance = new Operations();
    Double result = (Double) sumInstanceMethod.invoke(operationsInstance, 1, 3);

    assertThat(result, equalTo(4.0));
}
```

### 4.2 静态方法

由于这些方法不需要调用实例，我们可以将null作为第一个参数传递：

```java
@Test
public void givenObject_whenInvokeStaticMethod_thenCorrect() {
    Method multiplyStaticMethod = Operations.class.getDeclaredMethod("publicStaticMultiply", float.class, long.class);

    Double result = (Double) multiplyStaticMethod.invoke(null, 3.5f, 2);

    assertThat(result, equalTo(7.0));
}
```

## 5. 方法可访问性

**默认情况下，并非所有反射方法都可以访问**。这意味着JVM在调用它们时会强制执行访问控制检查。

例如，如果我们尝试在其定义类之外调用私有方法或从子类或其类的包之外调用受保护方法，我们将得到IllegalAccessException：

```java
@Test(expected = IllegalAccessException.class)
public void givenObject_whenInvokePrivateMethod_thenFail() {
    Method andPrivateMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);

    Operations operationsInstance = new Operations();
    Boolean result = (Boolean) andPrivateMethod.invoke(operationsInstance, true, false);

    assertFalse(result);
}

@Test(expected = IllegalAccessException.class)
public void givenObject_whenInvokeProtectedMethod_thenFail() {
    Method maxProtectedMethod = Operations.class.getDeclaredMethod("protectedMax", int.class, int.class);

    Operations operationsInstance = new Operations();
    Integer result = (Integer) maxProtectedMethod.invoke(operationsInstance, 2, 4);
    
    assertThat(result, equalTo(4));
}
```

### 5.1 AccessibleObject#setAccesible

**通过在反射方法对象上调用setAccessible(true)，JVM抑制访问控制检查并允许我们调用该方法而不抛出异常**：

```java
@Test
public void givenObject_whenInvokePrivateMethod_thenCorrect() throws Exception {
    Method andPrivatedMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);
    andPrivatedMethod.setAccessible(true);

    Operations operationsInstance = new Operations();
    Boolean result = (Boolean) andPrivatedMethod.invoke(operationsInstance, true, false);

    assertFalse(result);
}
```

### 5.2 AccessibleObject#canAccess

[Java 9](https://www.baeldung.com/new-java-9)提供了一种全新的方法来**检查调用者是否可以访问反射的方法对象**。

为此，它提供了canAccess作为已弃用的方法[isAccessible](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AccessibleObject.html#isAccessible())的替代品。

让我们看看它的实际效果：

```java
@Test
public void givenObject_whenInvokePrivateMethod_thenCheckAccess() throws Exception {
    Operations operationsInstance = new Operations();
    Method andPrivatedMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);
    boolean isAccessEnabled = andPrivatedMethod.canAccess(operationsInstance);
 
    assertFalse(isAccessEnabled);
 }
```

在使用setAccessible(true)将accessible标志设置为true之前，我们可以使用canAccess检查调用者是否已经可以访问反射方法。

### 5.3 AccessibleObject#trySetAccessible

trySetAccessible是另一个方便的方法，我们可以使用它来使反射对象可访问。

这种新方法的好处是，**如果无法启用访问，它会返回false**。但是，旧方法setAccessible(true)在失败时会 抛出InaccessibleObjectException。

让我们举例说明trySetAccessible方法的使用：

```java
@Test
public void givenObject_whenInvokePublicMethod_thenEnableAccess() throws Exception {
    Operations operationsInstance = new Operations();
    Method andPrivatedMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);
    andPrivatedMethod.trySetAccessible();
    boolean isAccessEnabled = andPrivatedMethod.canAccess(operationsInstance);
        
    assertTrue(isAccessEnabled);
}
```

## 6. 总结

在这篇快速文章中，我们了解了如何在运行时通过反射调用类的实例和静态方法。我们还展示了如何更改反射方法对象上的可访问标志，以在调用私有和受保护方法时抑制Java访问控制检查。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-11-2)上获得。