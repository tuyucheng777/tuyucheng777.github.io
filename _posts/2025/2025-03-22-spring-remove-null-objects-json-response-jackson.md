---
layout: post
title:  使用Spring和Jackson时删除JSON响应中的空对象
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

[JSON](https://www.baeldung.com/java-json)是[RESTful](https://www.baeldung.com/rest-with-spring-series)应用程序的事实标准，[Spring](https://www.baeldung.com/spring-tutorial)使用[Jackson](https://www.baeldung.com/jackson)库将对象与JSON无缝转换。然而，有时，我们希望自定义转换并提供特定的规则。

其中之一就是忽略响应或请求中的空值。**这可能会带来性能优势**，因为我们不需要来回发送空值。此外，这可以使我们的API更加简单。

在本教程中，我们将学习如何利用Jackson映射来简化REST交互。

## 2. 空值

在发送或接收请求时，我们经常会看到值设置为null。但是，通常它不会为我们提供任何有用的信息，因为在大多数情况下，这是未定义变量或字段的默认值。

此外，我们允许在JSON中传递空值，这使得验证过程变得复杂。如果值不存在，我们可以跳过验证并将其设置为默认值。但是，如果值存在，我们需要进行额外的检查来确定它是否为空，以及是否可以将其转换为某种合理的表示形式。

Jackson提供了一种直接在我们的类中配置它的便捷方法，**我们将使用[Include.NON_NULL](https://www.baeldung.com/jackson-ignore-null-fields)**。如果规则适用于所有字段，则可以在类级别使用它，或者我们可以在字段、Getter和Setter上更精细地使用它。让我们考虑以下Employee类：

```java
@JsonInclude(Include.NON_NULL)
public class Employee {
    private String lastName;
    private String firstName;
    private long id;
    // constructors, getters and setters
}
```

如果任何字段为null，并且我们仅讨论引用字段，则它们不会包含在生成的JSON：

```java
@ParameterizedTest
@MethodSource
void giveEndpointWhenSendEmployeeThanReceiveThatUserBackIgnoringNullValues(Employee expected) throws Exception {
    MvcResult result = sendRequestAndGetResult(expected, USERS);
    String response = result.getResponse().getContentAsString();
    validateJsonFields(expected, response);
}

private void validateJsonFields(Employee expected, String response) throws JsonProcessingException {
    JsonNode jsonNode = mapper.readTree(response);
    Predicate<Field> nullField = s -> isFieldNull(expected, s);
    List<String> nullFields = filterFieldsAndGetNames(expected, nullField);
    List<String> nonNullFields = filterFieldsAndGetNames(expected, nullField.negate());
    nullFieldsShouldBeMissing(nullFields, jsonNode);
    nonNullFieldsShouldNonBeMissing(nonNullFields, jsonNode);
}
```

有时，我们希望为null之类的字段类似的行为，Jackson也提供了一种处理它们的方法。

## 3. 缺失值

空[Optional](https://www.baeldung.com/java-optional)从技术上讲是非空值。但是，为请求或响应中不存在的值传递包装器没有什么意义。前面的注解不会处理这种情况，并且会尝试添加一些有关包装器本身的信息：

```json
{
    "lastName": "John",
    "firstName": "Doe",
    "id": 1,
    "salary": {
        "empty": true,
        "present": false
    }
}
```

假设我们Employee都可以公开自己的工资，只要他们愿意：

```java
@JsonInclude(Include.NON_ABSENT)
public class Employee {
    private String lastName;
    private String firstName;
    private long id;
    private Optional<Salary> salary;
    // constructors, getters and setters
}
```

我们可以使用返回null值的自定义Getter和Setter来处理它，**但是，这会使API变得复杂，并且首先会忽略使用Optional背后的想法**。要忽略空Options，我们可以使用Include.NON_ABSENT：

```java
private void validateJsonFields(Employee expected, String response) throws JsonProcessingException {
    JsonNode jsonNode = mapper.readTree(response);
    Predicate<Field> nullField = s -> isFieldNull(expected, s);
    Predicate<Field> absentField = s -> isFieldAbsent(expected, s);
    List<String> nullOrAbsentFields = filterFieldsAndGetNames(expected, nullField.or(absentField));
    List<String> nonNullAndNonAbsentFields = filterFieldsAndGetNames(expected, nullField.negate().and(absentField.negate()));
    nullFieldsShouldBeMissing(nullOrAbsentFields, jsonNode);
    nonNullFieldsShouldNonBeMissing(nonNullAndNonAbsentFields, jsonNode);
}
```

**Include.NON_ABSENT处理空Optional值和null这样我们就可以将它用于这两种场景**。

## 4. Empty值

我们应该在生成的JSON中包含空字符串或空集合吗？在大多数情况下，这没有意义。**将它们设置为null或用Optional包装它们可能不是一个好主意，并且会使与对象的交互变得复杂**。

让我们考虑一些有关我们Employee的其他信息，由于我们在一家国际组织中工作，因此可以合理地假设员工可能想要添加其姓名的拼音版本。此外，他们可能会提供一个或多个电话号码，以便其他人与他们取得联系：

```java
@JsonInclude(Include.NON_EMPTY)
public class Employee {
    private String lastName;
    private String firstName;
    private long id;
    private Optional<Salary> salary;
    private String phoneticName = "";
    private List<PhoneNumber> phoneNumbers = new ArrayList<>();
    // constructors, getters and setters
}
```

**我们可以使用Include.NON_EMPTY排除空值**，此配置忽略null以及不存在的值：

```java
private void validateJsonFields(Employee expected, String response) throws JsonProcessingException {
    JsonNode jsonNode = mapper.readTree(response);
    Predicate<Field> nullField = s -> isFieldNull(expected, s);
    Predicate<Field> absentField = s -> isFieldAbsent(expected, s);
    Predicate<Field> emptyField = s -> isFieldEmpty(expected, s);
    List<String> nullOrAbsentOrEmptyFields = filterFieldsAndGetNames(expected, nullField.or(absentField).or(emptyField));
    List<String> nonNullAndNonAbsentAndNonEmptyFields = filterFieldsAndGetNames(expected,
            nullField.negate().and(absentField.negate().and(emptyField.negate())));
    nullFieldsShouldBeMissing(nullOrAbsentOrEmptyFields, jsonNode);
    nonNullFieldsShouldNonBeMissing(nonNullAndNonAbsentAndNonEmptyFields, jsonNode);
}
```

正如前面提到的，所有这些注解都可以更精细地使用，我们甚至可以针对不同的领域应用不同的策略。**此外，我们可以全局配置映射器将此规则应用于任何转换**。

## 5. 自定义映射器

如果上述策略不够灵活，无法满足我们的需求或需要支持特定约定，我们应该使用Include.CUSTOM或实现[自定义序列化器](https://www.baeldung.com/jackson-custom-serialization)：

```java
public class CustomEmployeeSerializer extends StdSerializer<Employee> {
    @Override
    public void serialize(Employee employee, JsonGenerator gen, SerializerProvider provider) throws IOException {
        gen.writeStartObject();
        // Custom logic to serialize other fields
        gen.writeEndObject();
    }
}
```

## 6. 总结

Jackson和Spring可以帮助我们以最少的配置开发RESTul应用程序，包含策略可以简化我们的API并减少样板代码量。同时，如果默认解决方案限制过多或不灵活，我们可以扩展使用自定义映射器或过滤器。