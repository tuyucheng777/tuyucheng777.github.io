---
layout: post
title:  将Spring Bean设置为Null
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将学习如何将Spring上下文中的Bean设置为null。这在某些情况下可能很有用，例如当我们不想提供Mock时进行测试。此外，在使用一些可选功能时，我们可能希望避免创建实现并传递null。

此外，通过这种方式，如果我们想推迟在Bean生命周期之外选择所需实现的决定，我们可以创建占位符。最后，此技术可能是弃用过程中的第一步，其中涉及从上下文中删除特定的Bean。

## 2. 组件设置

有多种方法可以将Bean设置为null，具体取决于上下文的配置方式。**我们将考虑[XML](https://www.baeldung.com/spring-xml-injection)、[注解](https://www.baeldung.com/spring-bean-annotations)[和Java配置](https://www.baeldung.com/spring-application-context#1-java-based-configuration)**。我们将使用包含两个类的简单设置：

```java
@Component
public class MainComponent {
    private SubComponent subComponent;
    public MainComponent(final SubComponent subComponent) {
        this.subComponent = subComponent;
    }
    public SubComponent getSubComponent() {
        return subComponent;
    }
    public void setSubComponent(final SubComponent subComponent) {
        this.subComponent = subComponent;
    }
}
```

我们将展示如何在[Spring上下文](https://www.baeldung.com/spring-application-context#1-java-based-configuration)中将SubComponent设置为null：

```java
@Component
public class SubComponent {}
```

## 3. 使用占位符的XML配置

在XML配置中，我们可以使用特殊的占位符来标识null值：

```xml
<beans>
    <bean class="cn.tuyucheng.taketoday.nullablebean.MainComponent" name="mainComponent">
        <constructor-arg>
            <null/>
        </constructor-arg>
    </bean>
</beans>
```

此配置将提供以下结果：

```java
@Test
void givenNullableXMLContextWhenCreatingMainComponentThenSubComponentIsNull() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("nullable-application-context.xml");
    MainComponent bean = context.getBean(MainComponent.class);
    assertNull(bean.getSubComponent());
}
```

## 4. 使用SpEL进行XML配置

我们可以使用XML中的[SpEL](https://www.baeldung.com/spring-expression-language)获得类似的结果，与之前的配置有一些差异：

```xml
<beans>
    <bean class="cn.tuyucheng.taketoday.nullablebean.MainComponent" name="mainComponent">
        <constructor-arg value="#{null}"/>
    </bean>
</beans>
```

与上一个测试类似，我们可以识别出SubComponent为null：

```java
@Test
void givenNullableSpELXMLContextWhenCreatingMainComponentThenSubComponentIsNull() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("nullable-spel-application-context.xml");
    MainComponent bean = context.getBean(MainComponent.class);
    assertNull(bean.getSubComponent());
}
```

## 5. 使用SpEL和属性进行XML配置

改进先前解决方案的方法之一是将Bean名称存储在属性文件中。这样，我们就可以在需要时传递null值，而无需更改配置：

```properties
nullableBean = null
```

XML配置将使用[PropertyPlaceholderConfigurer](https://www.baeldung.com/spring-git-information)读取属性：

```xml
<beans>
    <bean id="propertyConfigurer"
          class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:nullable.properties"/>
    </bean>
    <bean class="cn.tuyucheng.taketoday.nullablebean.MainComponent" name="mainComponent">
        <constructor-arg value="#{ ${nullableBean} }"/>
    </bean>
    <bean class="cn.tuyucheng.taketoday.nullablebean.SubComponent" name="subComponent"/>
</beans>
```

但是，我们应该在SpEL表达式中使用属性占位符，以便正确读取值。因此，我们将SubComponent初始化为null：

```java
@Test
void givenNullableSpELXMLContextWithNullablePropertiesWhenCreatingMainComponentThenSubComponentIsNull() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("nullable-configurable-spel-application-context.xml");
    MainComponent bean = context.getBean(MainComponent.class);
    assertNull(bean.getSubComponent());
}
```

要提供实现，我们只需更改属性：

```properties
nullableBean = subComponent
```

## 6. Java配置中的Null供应器

不可能直接从用[@Bean](https://www.baeldung.com/spring-bean)标注的方法返回null，这就是为什么我们需要以某种方式包装它。我们可以使用[Supplier](https://www.baeldung.com/java-8-functional-interfaces#Suppliers)来做到这一点：

```java
@Bean
public Supplier<SubComponent> subComponentSupplier() {
    return () -> null;
}
```

从技术上讲，我们可以使用任何类来包装null值，但使用Supplier更惯用。在null的情况下，我们不关心Supplier可能会被调用几次。然而，如果我们想为普通的Bean实现类似的解决方案，我们必须确保Supplier在需要单例的情况下提供相同的实例。

该解决方案也将为我们提供正确的行为：

```java
@Test
void givenNullableSupplierContextWhenCreatingMainComponentThenSubComponentIsNull() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(NullableSupplierConfiguration.class);
    MainComponent bean = context.getBean(MainComponent.class);
    assertNull(bean.getSubComponent());
}
```

请注意，仅从@Bean返回null可能会制造问题：

```java
@Bean
public SubComponent subComponent() {
    return null;
}
```

在这种情况下，上下文将失败并显示[UnsatisfiedDependencyException](https://www.baeldung.com/spring-unsatisfied-dependency):

```java
@Test
void givenNullableContextWhenCreatingMainComponentThenSubComponentIsNull() {
    assertThrows(UnsatisfiedDependencyException.class, () ->  new AnnotationConfigApplicationContext(NullableConfiguration.class));
}
```

## 7. 使用Optional

使用[Optional](https://www.baeldung.com/java-optional)时，Spring会自动识别该Bean可以不在上下文中并传递null无需任何额外配置：

```java
@Bean
public MainComponent mainComponent(Optional<SubComponent> optionalSubComponent) {
    return new MainComponent(optionalSubComponent.orElse(null));
}
```

如果Spring在上下文中找不到SubComponent，它将传递一个空Optional：

```java
@Test
void givenOptionableContextWhenCreatingMainComponentThenSubComponentIsNull() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(OptionableConfiguration.class);
    MainComponent bean = context.getBean(MainComponent.class);
    assertNull(bean.getSubComponent());
}
```

## 8. 非必需的自动装配

使用null作为Bean值的另一种方法是将其声明为非必需。但是，此方法仅适用于非构造函数注入：

```java
@Component
public class NonRequiredMainComponent {
    @Autowired(required = false)
    private NonRequiredSubComponent subComponent;
    public NonRequiredSubComponent getSubComponent() {
        return subComponent;
    }
    public void setSubComponent(final NonRequiredSubComponent subComponent) {
        this.subComponent = subComponent;
    }
}
```

该依赖关系对于组件的正常运行不是必需的：

```java
@Test
void givenNonRequiredContextWhenCreatingMainComponentThenSubComponentIsNull() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(NonRequiredConfiguration.class);
    NonRequiredMainComponent bean = context.getBean(NonRequiredMainComponent.class);
    assertNull(bean.getSubComponent());
}
```

## 9. 使用@Nullable

此外，我们可以使用@Nullable注解来标识我们期望该Bean可能是无效的。[Spring](https://www.baeldung.com/spring-null-safety-annotations#the-nullable-annotation)和[Jakarta](https://www.baeldung.com/java-avoid-null-check#codeanalysis)注解都可以用于此：

```java
@Component
public class NullableMainComponent {
    private NullableSubComponent subComponent;
    public NullableMainComponent(final @Nullable NullableSubComponent subComponent) {
        this.subComponent = subComponent;
    }
    public NullableSubComponent getSubComponent() {
        return subComponent;
    }
    public void setSubComponent(final NullableSubComponent subComponent) {
        this.subComponent = subComponent;
    }
}
```

我们不需要将NullableSubComponent识别为Spring组件：

```java
public class NullableSubComponent {}
```

Spring上下文将根据@Nullable注解将其设置为null：

```java
@Test
void givenContextWhenCreatingNullableMainComponentThenSubComponentIsNull() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(NullableJavaConfiguration.class);
    NullableMainComponent bean = context.getBean(NullableMainComponent.class);
    assertNull(bean.getSubComponent());
}
```

## 10. 总结

在Spring上下文中使用null并不是最常见的做法，但有时可能是合理的。但是，将Bean设置为null的过程可能不是很直观。

在本文中，我们学习了如何以多种方式解决这个问题。