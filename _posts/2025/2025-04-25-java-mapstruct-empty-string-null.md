---
layout: post
title:  如何使用MapStruct将空字符串映射到Null
category: libraries
copyright: libraries
excerpt: MapStruct
---

## 1. 概述

在本教程中，我们将学习如何自定义[MapStruct](https://www.baeldung.com/mapstruct)映射器，将空字符串转换为null，我们将研究几个选项，每个选项都提供不同级别的控制和自定义。

## 2. 示例对象

在开始之前，我们需要创建两个对象，用于在示例和测试中进行映射。为了简单起见，我们将使用Student作为映射对象：

```java
class Student {
    String firstName;
    String lastName;
}
```

为了将我们的对象映射到其中，让我们创建一个Teacher：

```java
class Teacher {
    String firstName;
    String lastName;
}
```

我们在这里将事情保持得非常基础；两个类都具有相同名称的相同属性，因此默认情况下，我们的映射器无需额外的注解即可工作。

## 3. 所有字符串的全局映射器

对于我们的第一个映射器，我们来看一个能够涵盖所有情况和我们创建的每种映射的解决方案。如果我们的应用程序总是希望将String转换为null，并且不出现任何异常，那么这个方案就非常有用。

所有示例中映射器的基本结构都相同，我们需要一个带有@Mapper注解的接口，一个可以使用该[接口](https://www.baeldung.com/java-interfaces)的实例，以及最终的mapper方法：

```java
@Mapper
interface EmptyStringToNullGlobal {
    EmptyStringToNullGlobal INSTANCE = Mappers.getMapper(EmptyStringToNullGlobal.class);

    Teacher toTeacher(Student student);
}
```

对于基本映射来说，这已经够用了。但是，如果我们想将空字符串转换为null，则需要向接口添加另一个映射器：

```java
String mapEmptyString(String string) {
    return string != null && !string.isEmpty() ? string : null;
}
```

**在这个方法中，我们告诉MapStruct，如果处理的String已经为null或为空，则返回null**。否则，它返回原始String。

MapStruct会对所有遇到的String使用此方法，因此如果我们在接口中添加更多映射器，它们也会自动使用此方法。**只要我们始终需要这种行为，此选项就很简单、可自动复用，并且只需极少的额外代码**。它也是一个多功能的解决方案，因为它可以让我们做比这里关注的转换更多的事情。

为了演示这个工作原理，让我们创建一个简单的测试：

```java
@Test
void givenAMapperWithGlobalNullHandling_whenConvertingEmptyString_thenOutputNull() {
    EmptyStringToNullGlobal globalMapper = EmptyStringToNullGlobal.INSTANCE;

    Student student = new Student("Steve", "");

    Teacher teacher = globalMapper.toTeacher(student);
    assertEquals("Steve", teacher.firstName);
    assertNull(teacher.lastName);
}
```

在这个测试中，我们获取了映射器的一个实例。然后，我们创建了一个Student对象，名字为Steve，中间名是一个空字符串。使用映射器后，我们可以看到创建的Teacher对象的名字和预期的一样，是Steve。中间名也为空，这证明映射器的工作符合我们的预期。

## 4. 使用@Condition注解

### 4.1 @Condition的基本用法

我们将要研究的第二个选项是使用[@Condition注解](https://www.baeldung.com/java-mapstruct-Bean-types-conditional)，此注解允许我们创建MapStruct调用的方法，以检查某个属性是否应该映射。**由于MapStruct会将未映射的字段默认为null，因此我们可以利用它来实现目标**。这次，我们的映射器接口看起来与以前相同，但用一个新方法替换了之前的String映射器：

```java
@Mapper
interface EmptyStringToNullCondition {
    EmptyStringToNullCondition INSTANCE = Mappers.getMapper(EmptyStringToNullCondition.class);

    Teacher toTeacher(Student student);

    @Condition
    default boolean isNotEmpty(String value) {
        return value != null && !value.isEmpty();
    }
}
```

这里的方法isNotEmpty()每次在MapStruct遇到需要处理的字符串时都会被调用，如果返回false，则不会映射该值，目标对象中的字段将为null。如果返回true，则该值将被映射。

让我们创建一个测试来确认它是否按预期工作：

```java
@Test
void givenAMapperWithConditionAnnotationNullHandling_whenConvertingEmptyString_thenOutputNull() {
    EmptyStringToNullCondition conditionMapper = EmptyStringToNullCondition.INSTANCE;

    Student student = new Student("Steve", "");

    Teacher teacher = conditionMapper.toTeacher(student);
    assertEquals("Steve", teacher.firstName);
    assertNull(teacher.lastName);
}
```

这个测试与我们之前讨论的上一个选项的测试几乎完全相同，我们只是使用了新的映射器，并检查是否得到了相同的结果。

### 4.2 检查目标和源属性名称

@Condition注解选项 比我们之前的全局映射器示例更具可定制性，我们可以在isNotEmpty()方法签名中添加两个额外的注解参数：targetPropertyName和sourcePropertyName：

```java
@Condition
boolean isNotEmpty(String value, @TargetPropertyName String targetPropertyName, @SourcePropertyName String sourcePropertyName) {
    if (sourcePropertyName.equals("lastName")) {
        return value != null && !value.isEmpty();
    }
    return true;
}
```

这些新参数提供了要映射到的字段名称，我们可以用它们对某些String进行特殊处理，甚至可以将其他String完全排除在检查范围之外。在本例中，我们专门查找了名为lastName的源属性，并且仅在找到它时才应用检查。**如果我们想在大多数情况下应用空字符串到null的映射，但有少数例外情况，或者我们很少需要应用它，那么此选项非常有用**。

如果我们需要查找的异常或字段太多，代码就会变得非常混乱，我们应该考虑其他方法。这种方法还限制于创建的对象中的字段为空或为源值；我们根本无法更改该值。

## 5. 使用表达式

最后，让我们看看最有针对性的方案。我们可以在映射器中使用表达式，一次只影响一个映射方法。因此，如果我们不想广泛使用这种行为，这种方法是最佳选择。要使用表达式，只需将其添加到方法的@Mapping注解中即可，我们的映射器接口如下所示：

```java
@Mapper
public interface EmptyStringToNullExpression {
    EmptyStringToNullExpression INSTANCE = Mappers.getMapper(EmptyStringToNullExpression.class);

    @Mapping(target = "lastName", expression = "java(student.lastName.isEmpty() ? null : student.lastName)")
    Teacher toTeacher(Student student);
}
```

这个选项赋予了我们很大的权力，我们定义了我们感兴趣的目标字段，并提供了Java代码来告诉MapStruct如何映射它。我们的Java代码会检查lastName字段是否为空；如果是，则返回null，否则返回原始的lastName。

**这里的优点和缺点在于，我们已经非常明确地说明了这将影响哪些字段**，firstName字段根本不会被处理，更不用说我们可能定义的其他映射器中的任何String了。**因此，对于我们很少需要像这样映射String的应用程序来说，这是一个理想的选择**。

最后，让我们测试基于表达式的选项并确认它有效：

```java
@Test
void givenAMapperUsingExpressionBasedNullHandling_whenConvertingEmptyString_thenOutputNull() {
    EmptyStringToNullExpression expressionMapper = EmptyStringToNullExpression.INSTANCE;

    Student student = new Student("Steve", "");

    Teacher teacher = expressionMapper.toTeacher(student);
    assertEquals("Steve", teacher.firstName);
    assertNull(teacher.lastName);
}
```

我们再次测试了相同的操作，只是使用了表达式映射器。lastName字段再次按预期映射到null，而firstName字段保持不变。

## 6. 总结

在本文中，我们探讨了使用MapStruct将String转换为null的三种方案。首先，我们了解到，如果我们希望始终无例外地执行此行为，那么使用全局映射器是一种简单的选择。然后，我们研究了使用@Condition注解来获得类似的结果，但如果需要，还可以选择更多控制。最后，我们了解了如果我们很少希望发生某种映射，如何使用表达式来定位单个字段。