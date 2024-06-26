---
layout: post
title:  Javax BigDecimal验证
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

在教程[Java Bean Validation基础](https://www.baeldung.com/javax-validation)中，我们看到了如何将基本的javax验证应用于各种类型，在本教程中，我们将重点介绍如何将javax验证与BigDecimal结合使用。

## 2. 验证BigDecimal实例

不幸的是，**对于BigDecimal，我们不能使用经典的@Min或@Max javax注解，因为它们只接受long值**。

幸运的是，我们有一组专门的注解来处理它们：

-   @DecimalMin
-   @Digits
-   @DecimalMax

BigDecimal精度高，是[金融计算](https://www.baeldung.com/java-bigdecimal-biginteger)的首选。

让我们看看我们的Invoice类，它有一个BigDecimal类型的字段：

```java
public class Invoice {

    @DecimalMin(value = "0.0", inclusive = false)
    @Digits(integer=3, fraction=2)
    private BigDecimal price;
    private String description;

    public Invoice(BigDecimal price, String description) {
        this.price = price;
        this.description = description;
    }
}
```

### 2.1 @DecimalMin

**带注解的元素必须是一个数字，其值大于或等于指定的最小值**。@DecimalMin有一个inclusive属性，表示指定的最小值是包含还是不包含。

### 2.2 @DecimalMax

@DecimalMax是@DecimalMin的对应物。带注解的元素必须是一个数字，其值小于或等于指定的最大值。@DecimalMax有一个inclusive属性，表示指定的最大值是包含还是不包含。

另外，@Min和@Max只接受long值。在@DecimalMin和@DecimalMax中，我们可以指定字符串格式的值，可以是任何数字类型。

### 2.3 @Digit

在很多情况下，我们需要验证十进制数的整数部分和小数部分的位数。

**@Digit注解有两个属性，integer和fraction，用于指定数字的整数部分和小数部分允许的位数**。

根据[官方文档](https://docs.oracle.com/javaee/7/api/javax/validation/constraints/Digits.html)，integral允许我们**指定该数字可接受的最大整数位数**。

类似地，fraction属性允许我们**指定该数字可接受的最大小数位数**。

### 2.4 测试用例

让我们看看这些注解的实际效果。

首先，我们将添加一个测试，根据我们的验证创建一个具有无效price的Invoice，并检查验证是否会失败：

```java
public class InvoiceUnitTest {

    private static Validator validator;

    @BeforeClass
    public static void setupValidatorInstance() {
        validator = Validation.buildDefaultValidatorFactory().getValidator();
    }

    @Test
    public void whenMoreThanThreeIntegerDigits_thenShouldGiveConstraintViolations() {
        Invoice invoice = new Invoice(new BigDecimal("1021.21"), "Book purchased");
        Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
        assertThat(violations).hasSize(1);
        assertThat(violations)
                .extracting("message")
                .containsOnly("numeric value out of bounds (<3 digits>.<2 digits> expected)");
    }
}
```

现在让我们用正确的price检查验证：

```java
@Test
public void whenLessThanThreeIntegerDigits_thenShouldNotGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("10.21"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).isEmpty();
}
```

以类似的方式，让我们看看小数部分的验证是如何工作的：

```java
@Test
public void whenTwoFractionDigits_thenShouldNotGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("99.99"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).isEmpty();
}

@Test
public void whenMoreThanTwoFractionDigits_thenShouldGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("99.999"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).hasSize(1);
    assertThat(violations)
        .extracting("message")
        .containsOnly("numeric value out of bounds (<3 digits>.<2 digits> expected)");
}
```

等于0.00的price应该违反我们的约束：

```java
@Test
public void whenPriceIsZero_thenShouldGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("0.00"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).hasSize(1);
    assertThat(violations)
        .extracting("message")
        .containsOnly("must be greater than 0.0");
}
```

最后，让我们看看price大于零的情况：

```java
@Test
public void whenPriceIsGreaterThanZero_thenShouldNotGiveConstraintViolations() {
    Invoice invoice = new Invoice(new BigDecimal("100.50"), "Book purchased");
    Set<ConstraintViolation<Invoice>> violations = validator.validate(invoice);
    assertThat(violations).isEmpty();
}
```

## 3. 总结

在本文中，我们了解了如何对BigDecimal使用javax验证。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-validation-2)上获得。