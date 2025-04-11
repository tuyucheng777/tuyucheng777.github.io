---
layout: post
title:  在Spring中使用BeanUtils.copyProperties复制特定字段
category: spring
copyright: spring
excerpt: Spring
---

## 1. 概述

在使用Java应用程序时，我们经常需要将数据从一个对象复制到另一个对象。[Spring框架](https://spring.io/projects/spring-framework)的BeanUtils.copyProperties方法是一个常用的函数，用于将属性从一个Bean复制到另一个Bean。然而，**该方法默认会复制所有匹配的属性**。

有时，我们需要复制特定的字段，例如当源对象和目标对象具有不同的结构，或者我们只需要特定操作的一部分字段时。**为了复制特定字段而不是所有字段，我们需要自定义此行为**。

在本教程中，我们将探讨仅复制特定字段及其示例的不同方法。

## 2. 使用ignoreProperties选项

**copyProperties方法允许第3个参数ignoreProperties，它指定不应从源对象复制到目标对象的字段**，我们可以将一个或多个属性名称作为[可变参数](https://www.baeldung.com/java-pass-collection-varargs-parameter#whats-a-varargs-parameter)传递以排除(例如address、age)。

在这里，我们将尝试将属性从SourceBean复制到TargetBean：

```java
public class SourceBean {
    private String name;
    private String address;
    private int age;

    // constructor and getters/setters here
}

public class TargetBean {
    private String name;
    private String address;
    private int age;

    // constructor and getters/setters here
}
```

让我们看下面的例子，其中传递给BeanUtils.copyProperties的第3个参数address表示sourceBean中的**address属性不应该复制到targetBean**：

```java
public class BeanUtilsCopyPropertiesUnitTest {
    @Test
    public void givenObjects_whenUsingIgnoreProperties_thenCopyProperties() {
        SourceBean sourceBean = new SourceBean("Peter", 30, "LA");
        TargetBean targetBean = new TargetBean();

        BeanUtils.copyProperties(sourceBean, targetBean, "address");
        assertEquals(targetBean.getName(), sourceBean.getName());
        assertEquals(targetBean.getAge(), sourceBean.getAge());
        assertNull(targetBean.getAddress());
    }
}
```

在上面的例子中，targetBean对象中的address字段为null。

## 3. 使用自定义包装器

我们可以创建一个实用程序方法，允许我们提及要复制的特定字段，**此方法将包装BeanUtils.copyProperties，并允许我们指定要复制的字段，同时排除其他字段**：

```java
public static void copySpecifiedProperties(Object source, Object target, Set<String> props) {
    String[] excludedProperties = Arrays.stream(BeanUtils.getPropertyDescriptors(source.getClass()))
        .map(PropertyDescriptor::getName)
        .filter(name -> !props.contains(name))
        .toArray(String[]::new);
    BeanUtils.copyProperties(source, target, excludedProperties);
}
```

在下面的例子中，目标是将name和age字段从sourceBean复制到targetBean，并排除复制address字段：

```java
@Test
public void givenObjects_whenUsingCustomWrapper_thenCopyProperties() {
    SourceBean sourceBean = new SourceBean("Peter", 30, "LA");
    TargetBean targetBean = new TargetBean();
    BeanUtilsCopyProperties.copySpecifiedProperties(sourceBean, targetBean, new HashSet<>(Arrays.asList("name", "age")));
    assertEquals(targetBean.getName(), sourceBean.getName());
    assertEquals(targetBean.getAge(), sourceBean.getAge());
    assertNull(targetBean.getAddress());
}
```

## 4. 使用中间DTO对象

这种方法需要创建一个中间DTO对象，用于过滤源对象中的特定字段，然后再将它们复制到目标对象。我们首先将源对象的字段复制到中间[DTO](https://www.baeldung.com/java-dto-pattern)对象(该对象是经过过滤的表示形式)，然后，我们将过滤后的字段从中间DTO对象复制到目标对象。

我们来看一下中间对象TempDTO的结构：

```java
public class TempDTO {
    public String name;
    public int age;
}
```

现在，我们首先将特定字段从sourceBean复制到tempDTO实例，然后从tempDTO复制到targetBean：

```java
@Test
public void givenObjects_whenUsingIntermediateObject_thenCopyProperties() {
    SourceBean sourceBean = new SourceBean("Peter", 30, "LA");
    TempDTO tempDTO = new TempDTO();
    BeanUtils.copyProperties(sourceBean, tempDTO);

    TargetBean targetBean = new TargetBean();
    BeanUtils.copyProperties(tempDTO, targetBean);
    assertEquals(targetBean.getName(), sourceBean.getName());
    assertEquals(targetBean.getAge(), sourceBean.getAge());
    assertNull(targetBean.getAddress());
}
```

## 5. 总结

在本文中，我们探讨了使用Java中的BeanUtils.copyProperties复制特定字段的各种方法，ignoreProperties参数提供了一种简单的解决方案，而自定义包装器和中间对象方法则为处理需要忽略的长属性列表提供了更大的灵活性。

中间对象方法运行速度稍慢，因为它执行两次复制操作，但它确保关注点的干净分离，防止不必要的数据复制，并允许灵活和选择性的字段映射。