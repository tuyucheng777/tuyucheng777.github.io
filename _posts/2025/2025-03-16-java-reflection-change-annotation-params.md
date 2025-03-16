---
layout: post
title:  在运行时更改注解参数
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

注解，一种可以添加到Java代码中的元数据形式。这些注解可以在编译时处理并嵌入到类文件中，也可以在运行时使用反射保留和访问。

在本文中，我们将讨论如何使用反射在运行时更改注解值，我们将在此示例中使用类级注解。

## 2. 注解

Java允许使用现有注解创建新注解，最简单的形式是，注解表示为@符号后跟注解名称：

```java
@Override
```

让我们创建自己的注解Greeter：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Greeter {    
    public String greet() default ""; 
}
```

现在，我们将创建一个使用类级注解的Java类Greetings：

```java
@Greeter(greet="Good morning")
public class Greetings {}
```

现在，我们将使用反射来访问注解值。Java类Class提供了一种方法getAnnotation来访问类的注解：

```java
Greeter greetings = Greetings.class.getAnnotation(Greeter.class);
System.out.println("Hello there, " + greetings.greet() + " !!");
```

## 3. 修改注解

Java类Class维护一个用于管理注解的Map-注解类作为键，注解对象作为值：

```java
Map<Class<? extends Annotation>, Annotation> map;
```

我们将更新此Map以在运行时更改注解，访问此Map的方法在不同的JDK实现中有所不同，我们将针对JDK 7和JDK 8进行讨论。

### 3.1 JDK 7实现

Java类Class具有字段annotations，由于这是一个私有字段，因此要访问它，我们必须将字段的可访问性设置为true。Java提供了方法getDeclaredField来通过其名称访问任何字段：

```java
Field annotations = Class.class.getDeclaredField(ANNOTATIONS);
annotations.setAccessible(true);
```

现在，让我们访问Greeter类的注解Map：

```java
 Map<Class<? extends Annotation>, Annotation> map = annotations.get(targetClass);
```

现在，这是包含有关所有注解及其值对象的信息的映射。我们想要更改Greeter注解值，这可以通过更新Greeter类的注解对象来实现：

```java
map.put(targetAnnotation, targetValue);
```

### 3.2 JDK 8实现

Java 8实现将注解信息存储在AnnotationData类中，我们可以使用annotationData方法访问此对象。我们将annotationData方法的可访问性设置为true，因为它是一个私有方法：

```java
Method method = Class.class.getDeclaredMethod(ANNOTATION_METHOD, null);
method.setAccessible(true);
```

现在，我们可以访问annotations字段。由于此字段也是私有字段，我们将可访问性设置为true：

```java
Field annotations = annotationData.getClass().getDeclaredField(ANNOTATIONS);
annotations.setAccessible(true);
```

此字段具有注解缓存映射，用于存储注解类和值对象，让我们对其进行修改：

```java
Map<Class<? extends Annotation>, Annotation> map = annotations.get(annotationData); 
map.put(targetAnnotation, targetValue);
```

## 4. 应用程序

让我们来看这个例子：

```java
Greeter greetings = Greetings.class.getAnnotation(Greeter.class);
System.err.println("Hello there, " + greetings.greet() + " !!");
```

这将是问候语“Good morning”，因为这是我们为注解提供的值。

现在，我们将再创建一个Greeter类型的对象，其值为“Good evening”：

```java
Greeter targetValue = new DynamicGreeter("Good evening");
```

让我们用新值更新注解Map：

```java
alterAnnotationValueJDK8(Greetings.class, Greeter.class, targetValue);
```

让我们再次检查问候语值：

```java
greetings = Greetings.class.getAnnotation(Greeter.class);
System.err.println("Hello there, " + greetings.greet() + " !!");
```

它将返回“Good evening”。

## 5. 总结

Java实现使用两个数据字段来存储注解数据：annotations和declaredAnnotations。两者之间的区别是：第一个还存储来自父类的注解，而后者仅存储当前类的注解。

由于getAnnotation的实现在JDK 7和JDK 8中有所不同，因此我们在这里使用annotations字段Map以简化操作。