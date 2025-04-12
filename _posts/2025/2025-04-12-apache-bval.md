---
layout: post
title:  Apache BVal简介
category: apache
copyright: apache
excerpt: Apache BVal
---

## 1. 简介

在本文中，我们将了解**Apache BVal库对Java Bean Validation规范(JSR 349)的实现**。

## 2. Maven依赖

为了使用Apache BVal，我们首先需要将以下依赖添加到pom.xml文件：

```xml
<dependency>
    <groupId>org.apache.bval</groupId>
    <artifactId>bval-jsr</artifactId>
    <version>3.0.1</version>
</dependency>
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

自定义BVal约束可以在可选的bval-extras依赖中找到：

```xml
<dependency>
    <groupId>org.apache.bval</groupId>
    <artifactId>bval-extras</artifactId>
    <version>3.0.1</version>
</dependency>
```

可以从Maven Central下载[bval-jsr](https://mvnrepository.com/artifact/org.apache.bval/bval-jsr)、[bval-extras](https://mvnrepository.com/artifact/org.apache.bval/bval-extras)和[validation-api](https://mvnrepository.com/artifact/javax.validation/validation-api)的最新版本。

## 3. 应用约束

Apache BVal提供了jakarta.validation包中定义的所有约束的实现，为了将约束应用于Bean的属性，我们可以**在属性声明中添加约束注解**。

让我们创建一个具有四个属性的User类，然后应用@NotNull、@Size和@Min注解：

```java
public class User {
    
    @NotNull
    private String email;
    
    private String password;

    @Size(min=1, max=20)
    private String name;

    @Min(18)
    private int age;

    // standard constructor, getters, setters
}
```

## 4. 校验Bean

为了校验应用于User类的约束，我们需要获取一个ValidatorFactory实例和一个或多个Validator实例。

### 4.1 获取ValidatorFactory

Apache BVal文档建议获取此类的单个实例，因为工厂创建是一个繁琐的过程：

```java
ValidatorFactory validatorFactory = Validation.byProvider(ApacheValidationProvider.class)
    .configure().buildValidatorFactory();
```

### 4.2 获取校验器

接下来，我们需要从上面定义的validatorFactory中获取一个Validator实例：

```java
Validator validator = validatorFactory.getValidator();
```

**这是一个线程安全的实现**，因此我们可以安全地重用已经创建的实例。

Validator类提供了三种确定Bean有效性的方法：validate()、validateProperty()和validateValue()。

这些方法中的每一个都会返回一组ConstraintViolation对象，其中包含有关未遵守的约束的信息。

### 4.3 validate() API

validate()方法检查整个Bean的有效性，这意味着它校验对作为参数传递的对象属性应用的所有约束。

让我们创建一个JUnit测试，其中定义一个User对象并使用validate()方法来测试它的属性：

```java
@Test
public void givenUser_whenValidate_thenValidationViolations() {
    User user = new User("ana@yahoo.com", "pass", "nameTooLong_______________", 15);

    Set<ConstraintViolation<User>> violations = validator.validate(user);
    assertTrue("no violations", violations.size() > 0);
}
```

### 4.4 validateProperty() API

validateProperty()方法可用于校验Bean的单个属性。

让我们创建一个JUnit测试，其中我们将定义一个age属性小于所需最小值18的User对象，并验证校验此属性是否会导致一次违规：

```java
@Test
public void givenInvalidAge_whenValidateProperty_thenConstraintViolation() {
    User user = new User("ana@yahoo.com", "pass", "Ana", 12);

    Set<ConstraintViolation<User>> propertyViolations = validator.validateProperty(user, "age");
 
    assertEquals("size is not 1", 1, propertyViolations.size());
}
```

### 4.5 validateValue() API

在将某个值设置到Bean上之前，可以使用validateValue()方法**检查该值是否是Bean属性的有效值**。

让我们用User对象创建一个JUnit测试，然后校验值20是否是age属性的有效值：

```java
@Test
public void givenValidAge_whenValidateValue_thenNoConstraintViolation() {
    User user = new User("ana@yahoo.com", "pass", "Ana", 18);
    
    Set<ConstraintViolation<User>> valueViolations = validator.validateValue(User.class, "age", 20);
 
    assertEquals("size is not 0", 0, valueViolations.size());
}
```

### 4.6 关闭ValidatorFactory

**使用ValidatorFactory后，我们必须记得在最后关闭它**：

```java
if (validatorFactory != null) {
    validatorFactory.close();
}
```

## 5. 非JSR约束

**Apache BVal库还提供了一系列不属于JSR规范的约束**，并提供了额外的、更强大的校验功能。

bval-jsr包包含两个附加约束：@Email用于校验有效的电子邮件地址，@NotEmpty用于确保值不为空。

其余自定义BVal约束在可选包bval-extras中提供。

该包包含用于校验各种数字格式的约束，例如确保数字为有效的国际银行账号的@IBAN注解、用于校验有效标准账簿编号的@Isbn注解以及用于校验国际文章编号的@EAN13注解。

该库还提供注解以**确保各种类型的信用卡号的有效性**：@AmericanExpress，@Diners，@Discover，@Mastercard和@Visa。

你可以**使用@Domain和@InetAddress注解来确定值是否包含有效的域或Internet地址**。

最后，该包包含@Directory和@NotDirectory注解，用于校验File对象是否是目录。

让我们在User类上定义附加属性，并对它们应用一些非JSR注解：

```java
public class User {
    
    @NotNull
    @Email
    private String email;
    
    @NotEmpty
    private String password;

    @Size(min=1, max=20)
    private String name;

    @Min(18)
    private int age;
    
    @Visa
    private String cardNumber = "";
    
    @IBAN
    private String iban = "";
    
    @InetAddress
    private String website = "";

    @Directory
    private File mainDirectory = new File(".");

    // standard constructor, getters, setters
}
```

可以采用与JSR约束类似的方式测试这些约束：

```java
@Test
public void whenValidateNonJSR_thenCorrect() {
    User user = new User("ana@yahoo.com", "pass", "Ana", 20);_
    user.setCardNumber("1234");
    user.setIban("1234");
    user.setWebsite("10.0.2.50");
    user.setMainDirectory(new File("."));
    
    Set<ConstraintViolation<User>> violations = validator.validateProperty(user,"iban");
 
    assertEquals("size is not 1", 1, violations.size());
 
    violations = validator.validateProperty(user,"website");
 
    assertEquals("size is not 0", 0, violations.size());

    violations = validator.validateProperty(user, "mainDirectory");
 
    assertEquals("size is not 0", 0, violations.size());
}
```

虽然这些额外的注解对于潜在的校验需求很方便，但使用不属于JSR规范的注解的一个缺点是，如果以后有必要，你无法轻松切换到不同的JSR实现。

## 6. 自定义约束

为了定义我们自己的约束，我们首先需要按照标准语法创建一个注解。

让我们创建一个Password注解，它将定义用户密码必须满足的条件：

```java
@Constraint(validatedBy = { PasswordValidator.class })
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface Password {
    String message() default "Invalid password";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    int length() default 6;

    int nonAlpha() default 1;
}
```

密码值的实际校验是在实现ConstraintValidator接口的类中完成的-在我们的例子中是PasswordValidator类；该类重写了isValid()方法，并校验密码的长度是否小于length属性，以及密码中包含的非字母数字字符的数量是否少于nonAlpha属性中指定的数量：

```java
public class PasswordValidator implements ConstraintValidator<Password, String> {
    private int length;
    private int nonAlpha;

    @Override
    public void initialize(Password password) {
        this.length = password.length();
        this.nonAlpha = password.nonAlpha();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext ctx) {
        if (value.length() < length) {
            return false;
        }
        int nonAlphaNr = 0;
        for (int i = 0; i < value.length(); i++) {
            if (!Character.isLetterOrDigit(value.charAt(i))) {
                nonAlphaNr++;
            }
        }
        if (nonAlphaNr < nonAlpha) {
            return false;
        }
        return true;
    }
}
```

让我们将自定义约束应用于User类的password属性：

```java
@Password(length = 8)
private String password;
```

我们可以创建一个JUnit测试来校验无效的密码值是否会导致约束违反：

```java
@Test
public void givenValidPassword_whenValidatePassword_thenNoConstraintViolation() {
    User user = new User("ana@yahoo.com", "password", "Ana", 20);
    Set<ConstraintViolation<User>> violations = validator.validateProperty(user, "password");

    assertEquals(
            "message incorrect",
            "Invalid password",
            violations.iterator().next().getMessage());
}
```

现在让我们创建一个JUnit测试来校验有效的密码值：

```java
@Test
public void givenValidPassword_whenValidatePassword_thenNoConstraintViolation() {
    User user = new User("ana@yahoo.com", "password#", "Ana", 20);
		
    Set<ConstraintViolation<User>> violations = validator.validateProperty(user, "password");
    assertEquals("size is not 0", 0, violations.size());
}
```

## 7. 总结

在本文中，我们举例说明了Apache BVal Bean校验实现的使用。