---
layout: post
title: 使用AssertJ比较时忽略字段
category: assertion
copyright: assertion
excerpt: AssertJ
---

## 1. 简介

使用[AssertJ](https://www.baeldung.com/introduction-to-assertj)进行比较时忽略字段是一项强大的功能，它有助于通过仅关注对象的相关部分来简化测试。在处理具有动态或不相关值的字段的对象时，此功能非常有用。

在本文中，我们将介绍AssertJ流式API中的各种方法。首先，我们将设置正确的依赖并提供一个简单的Employee类示例。然后，我们将探索使用AssertJ提供的方法在对象比较中忽略必填字段的各种用例。

## 2. Maven依赖和示例设置

让我们在pom.xml中添加[assertj-guava](https://mvnrepository.com/artifact/org.assertj/assertj-guava)依赖项：

```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-guava</artifactId>
    <version>3.4.0</version>
</dependency>
```

接下来，我们将构建一个具有以下字段的简单Employee类：

```java
public class Employee {
    public Long id;
    public String name;
    public String department;
    public String homeAddress;
    public String workAddress;
    public LocalDate dateOfBirth;
    public Double grossSalary;
    public Double netSalary;
    ...// constructor
    ...//getters and setters
}
```

在接下来的部分中，我们将重点介绍AssertJ流式API提供的方法，以便在各种情况下忽略Employee类测试中的所需字段。

## 3. 使用ignoringFields()

首先，让我们看看如何在比较实际对象和预期对象时指定要忽略的一个或多个字段。在这里，我们可以使用ignoringFields()方法，它允许我们指定在比较期间应忽略哪些字段。**当我们想要排除可能在实例之间有所不同但与测试无关的字段时，这特别有用**。

让我们使用前面创建的Employee类作为示例，我们将创建Employee类的两个实例，然后使用AssertJ对它们进行比较，同时忽略一些字段：

```java
@Test
public void givenEmployeesWithDifferentAddresses_whenComparingIgnoringSpecificFields_thenEmployeesAreEqual() {
    // Given
    Employee employee1 = new Employee();
    employee1.id = 1L;
    employee1.name = "John Doe";
    employee1.department = "Engineering";
    employee1.homeAddress = "123 Home St";
    employee1.workAddress = "456 Work Ave";
    employee1.dateOfBirth = LocalDate.of(1990, 1, 1);
    employee1.grossSalary = 100000.0;
    employee1.netSalary = 75000.0;

    Employee employee2 = new Employee();
    employee2.id = 2L;
    employee2.name = "John Doe";
    employee2.department = "Engineering";
    employee2.homeAddress = "789 Home St";
    employee2.workAddress = "101 Work Ave";
    employee2.dateOfBirth = LocalDate.of(1990, 1, 1);
    employee2.grossSalary = 110000.0;
    employee2.netSalary = 80000.0;

    // When & Then
    Assertions.assertThat(employee1)
        .usingRecursiveComparison()
        .ignoringFields("id", "homeAddress", "workAddress", "grossSalary", "netSalary")
        .isEqualTo(employee2);
}
```

这里，我们在比较两个Employee对象时忽略id、 homeAddress、workAddress、grossSalary和netSalary字段，这意味着即使employee1和employee2的所有这些忽略字段都不同，相等性也仍然成立。

## 4. 使用ignoringFieldsMatchingRegexes() 

**AssertJ还提供了ignoringFieldsMatchingRegexes()方法，允许我们根据[正则表达式](https://www.baeldung.com/regular-expressions-java)忽略字段**。当我们想要排除多个具有相似名称的字段时，这很有用。

假设我们有一个Employee类，其中homeAddress、workAddress等字段在比较中应该被忽略。同样，我们希望忽略所有以“Salary”结尾的字段：

```java
@Test
public void givenEmployeesWithDifferentSalaries_whenComparingIgnoringFieldsMatchingRegex_thenEmployeesAreEqual() {
    Employee employee1 = new Employee();
    employee1.id = 1L;
    employee1.name = "Jane Smith";
    employee1.department = "Marketing";
    employee1.homeAddress = "123 Home St";
    employee1.workAddress = "456 Work Ave";
    employee1.dateOfBirth = LocalDate.of(1990, 1, 1);
    employee1.grossSalary = 95000.0;
    employee1.netSalary = 70000.0;

    Employee employee2 = new Employee();
    employee2.id = 2L;
    employee2.name = "Jane Smith";
    employee2.department = "Marketing";
    employee2.homeAddress = "789 Home St";
    employee2.workAddress = "101 Work Ave";
    employee2.dateOfBirth = LocalDate.of(1990, 1, 1);
    employee2.grossSalary = 98000.0;
    employee2.netSalary = 72000.0;

    Assertions.assertThat(employee1)
        .usingRecursiveComparison()
        .ignoringFields("id")
        .ignoringFieldsMatchingRegexes(".Address", ".Salary")
        .isEqualTo(employee2);
}
```

这里，除了id字段之外，所有以Address和Salary结尾的字段都被使用ignoringFieldMatchingRegex()忽略。

## 5. 使用ignoringExpectedNullFields()

最后，我们将研究AssertJ API中的ignoreExpectedNullFields()方法。**此方法在比较期间忽略预期对象中为空的字段，当预期对象中只有某些字段很重要**，而其他字段未设置或不相关时，这特别有用。

假设我们要比较两个Employee对象，但是在预期的对象中，某些字段为空，应该被忽略：

```java
@Test
public void givenEmployeesWithNullExpectedFields_whenComparingIgnoringExpectedNullFields_thenEmployeesAreEqual() {
    Employee expectedEmployee = new Employee();
    expectedEmployee.id = null;
    expectedEmployee.name = "Alice Johnson";
    expectedEmployee.department = null;
    expectedEmployee.homeAddress = null;
    expectedEmployee.workAddress = null;
    expectedEmployee.dateOfBirth = LocalDate.of(1985, 5, 15);
    expectedEmployee.grossSalary = null;
    expectedEmployee.netSalary = null;

    Employee actualEmployee = new Employee();
    actualEmployee.id = 3L;
    actualEmployee.name = "Alice Johnson";
    actualEmployee.department = "HR";
    actualEmployee.homeAddress = "789 Home St";
    actualEmployee.workAddress = "123 Work Ave";
    actualEmployee.dateOfBirth = LocalDate.of(1985, 5, 15);
    actualEmployee.grossSalary = 90000.0;
    actualEmployee.netSalary = 65000.0;

    Assertions.assertThat(actualEmployee)
        .usingRecursiveComparison()
        .ignoringExpectedNullFields()
        .isEqualTo(expectedEmployee);
}
```

这里，根据expectedEmployee对象中的所有非空字段对expectedEmployee对象和actualEmployee对象进行比较。

## 6. 总结

在本教程中，我们了解了AssertJ提供的各种方法，用于在测试中进行对象比较时忽略某些字段。使用ignoringFields()、ignoringFieldsMatchingRegexes()和ignoringExpectedNullFields()等方法，我们可以使测试更加灵活且易于维护。