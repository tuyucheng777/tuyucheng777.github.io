---
layout: post
title:  使用Spring Security检测泄露的密码
category: spring-security
copyright: spring-security
excerpt: Spring Data JPA
---

## 1. 概述

在构建处理敏感数据的Web应用程序时，确保用户密码的安全非常重要。**密码安全的一个重要方面是检查密码是否被泄露，通常是由于其存在于数据泄露中**。

[Spring Security](https://www.baeldung.com/security-spring) 6.3引入了一项新功能，使我们能够轻松检查密码是否已被泄露。

在本教程中，我们将探索Spring Security中新的CompromisedPasswordChecker API以及如何将其集成到我们的Spring Boot应用程序中。

## 2. 了解泄露的密码

泄露的密码是在数据泄露中暴露的密码，使其容易受到未经授权的访问。攻击者经常使用这些泄露的密码进行[凭证填充](https://www.baeldung.com/cs/security-credential-stuffing-password-spraying#credential-stuffing)和[密码填充攻击](https://www.baeldung.com/cs/security-credential-stuffing-password-spraying#password-spraying)，在多个网站上使用泄露的用户名密码对或针对多个帐户使用常用密码。

为了降低这种风险，在创建账户之前检查用户密码是否被泄露至关重要。

**还要注意的是，以前有效的密码可能会随着时间的推移而泄露，**因此我们始终建议不仅在创建帐户时检查密码是否泄露，而且在登录过程中或允许用户更改密码的任何过程中也检查密码是否泄露。如果由于检测到密码泄露而导致登录尝试失败，我们可以提示用户重置密码。

## 3. CompromisedPasswordChecker API

Spring Security提供了一个简单的CompromisedPasswordChecker接口，用于检查密码是否已被泄露：

```java
public interface CompromisedPasswordChecker {
    CompromisedPasswordDecision check(String password);
}
```

该接口公开一个check()方法，该方法以密码作为输入并返回CompromisedPasswordDecision的实例，指示密码是否被泄露。

check()方法需要纯文本密码，因此我们必须在使用[PasswordEncoder](https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)加密密码之前调用该方法。

### 3.1 配置CompromisedPasswordChecker Bean

为了在我们的应用程序中启用泄露密码检查，我们需要声明CompromisedPasswordChecker类型的Bean：

```java
@Bean
public CompromisedPasswordChecker compromisedPasswordChecker() {
    return new HaveIBeenPwnedRestApiPasswordChecker();
}
```

**HaveIBeenPwnedRestApiPasswordChecker是Spring Security提供的CompromisedPasswordChecker的默认实现**。

**此默认实现与流行的[Have I Been Pwned API](https://haveibeenpwned.com/API/v3#PwnedPasswords)集成，该API维护着一个包含因数据泄露而泄露的密码的庞大数据库**。

调用此默认实现的check()方法时，它会安全地对提供的密码进行哈希处理，并将哈希的前5个字符发送到Have I Been Pwned API，API会响应与此前缀匹配的哈希后缀列表。然后，该方法将密码的完整哈希与此列表进行比较，并确定密码是否被破解。整个检查过程无需通过网络发送明文密码即可完成。

### 3.2 自定义CompromisedPasswordChecker Bean

如果我们的应用程序使用代理服务器进行出站HTTP请求，我们可以使用自定义RestClient配置HaveIBeenPwnedRestApiPasswordChecker：

```java
@Bean
public CompromisedPasswordChecker customCompromisedPasswordChecker() {
    RestClient customRestClient = RestClient.builder()
            .baseUrl("https://api.proxy.com/password-check")
            .defaultHeader("X-API-KEY", "api-key")
            .build();

    HaveIBeenPwnedRestApiPasswordChecker compromisedPasswordChecker = new HaveIBeenPwnedRestApiPasswordChecker();
    compromisedPasswordChecker.setRestClient(customRestClient);
    return compromisedPasswordChecker;
}
```

现在，当我们在应用程序中调用CompromisedPasswordChecker Bean的check()方法时，它会将API请求与自定义HTTP标头一起发送到我们定义的基本URL。

## 4. 处理泄露的密码

现在我们已经配置了CompromisedPasswordChecker Bean，让我们看看如何在服务层中使用它来验证密码。让我们以新用户注册的常见用例为例：

```java
@Autowired
private CompromisedPasswordChecker compromisedPasswordChecker;

String password = userCreationRequest.getPassword();
CompromisedPasswordDecision decision = compromisedPasswordChecker.check(password);
if (decision.isCompromised()) {
    throw new CompromisedPasswordException("The provided password is compromised and cannot be used.");
}
```

这里，我们只需使用客户端提供的明文密码调用check()方法并检查返回的CompromisedPasswordDecision。**如果isCompromised()方法返回true，我们将抛出CompromisedPasswordException以中止注册过程**。

## 5. 处理CompromisedPasswordException

当我们的服务层抛出CompromisedPasswordException时，我们希望妥善处理它并向客户端提供反馈。

一种方法是在@RestControllerAdvice类中定义一个全局异常处理程序：

```java
@ExceptionHandler(CompromisedPasswordException.class)
public ProblemDetail handle(CompromisedPasswordException exception) {
    return ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, exception.getMessage());
}
```

当此处理程序方法捕获到CompromisedPasswordException时，它会返回[ProblemDetail](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html)类的一个实例，该实例构造一个符合RFC 9457规范的错误响应：

```json
{
    "type": "about:blank",
    "title": "Bad Request",
    "status": 400,
    "detail": "The provided password is compromised and cannot be used.",
    "instance": "/api/v1/users"
}
```

## 6. 自定义CompromisedPasswordChecker实现

虽然HaveIBeenPwnedRestApiPasswordChecker实现是一个很好的解决方案，但在某些情况下，我们可能希望与其他提供商集成，甚至实现我们自己的受损密码检查逻辑。

我们可以通过实现CompromisedPasswordChecker接口来实现这一点：

```java
public class PasswordCheckerSimulator implements CompromisedPasswordChecker {
    public static final String FAILURE_KEYWORD = "compromised";

    @Override
    public CompromisedPasswordDecision check(String password) {
        boolean isPasswordCompromised = false;
        if (password.contains(FAILURE_KEYWORD)) {
            isPasswordCompromised = true;
        }
        return new CompromisedPasswordDecision(isPasswordCompromised);
    }
}
```

如果密码中包含“compromised”一词，我们的示例实现会认为该密码已被泄露。虽然在实际场景中用处不大，但它展示了插入我们自己的自定义逻辑是多么简单。

在我们的测试用例中，使用这种模拟实现而不是对外部API进行HTTP调用通常是一种很好的做法。要在测试中使用我们的自定义实现，我们可以将其定义为[@TestConfiguration](https://www.baeldung.com/spring-boot-testing#test-configuration-withtestconfiguration)类中的Bean：

```java
@TestConfiguration
public class TestSecurityConfiguration {
    @Bean
    public CompromisedPasswordChecker compromisedPasswordChecker() {
        return new PasswordCheckerSimulator();
    }
}
```

在我们的测试类中，我们想要使用这个自定义实现，我们将用@Import(TestSecurityConfiguration.class)对其进行标注。

此外，为了避免在运行测试时出现[BeanDefinitionOverrideException](https://www.baeldung.com/spring-boot-Bean-definition-override-exception)[@ConditionalOnMissingBean]，我们将使用(https://www.baeldung.com/spring-boot-custom-auto-configuration#2-Bean-conditions)注解来标注我们的主要CompromisedPasswordChecker Bean。

最后，为了验证我们自定义实现的行为，我们将编写一个测试用例：

```java
@Test
void whenPasswordCompromised_thenExceptionThrown() {
    String emailId = RandomString.make() + "@taketoday.it";
    String password = PasswordCheckerSimulator.FAILURE_KEYWORD + RandomString.make();
    String requestBody = String.format("""
            {
                "emailId"  : "%s",
                "password" : "%s"
            }
            """, emailId, password);

    String apiPath = "/users";
    mockMvc.perform(post(apiPath).contentType(MediaType.APPLICATION_JSON).content(requestBody))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.status").value(HttpStatus.BAD_REQUEST.value()))
            .andExpect(jsonPath("$.detail").value("The provided password is compromised and cannot be used."));
}
```

## 7. 创建自定义@NotCompromised注解

如前所述，**我们不仅应该在用户注册期间检查密码是否被泄露，还应该在所有允许用户更改密码或使用密码进行身份验证的API(例如登录API)中检查密码是否被泄露**。

虽然我们可以在服务层对每个流程执行此检查，但**使用[自定义校验注解](https://www.baeldung.com/spring-mvc-custom-validator)可以提供一种更具声明式和可重用性的方法**。

首先，让我们定义一个自定义的@NotCompromised注解：

```java
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = CompromisedPasswordValidator.class)
public @interface NotCompromised {
    String message() default "The provided password is compromised and cannot be used.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

接下来我们实现ConstraintValidator接口：

```java
public class CompromisedPasswordValidator implements ConstraintValidator<NotCompromised, String> {
    @Autowired
    private CompromisedPasswordChecker compromisedPasswordChecker;

    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        CompromisedPasswordDecision decision = compromisedPasswordChecker.check(password);
        return !decision.isCompromised();
    }
}
```

我们自动注入CompromisedPasswordChecker类的实例并使用它来检查客户端的密码是否被泄露。

我们现在可以在请求主体的密码字段上使用自定义的@NotCompromised注解并验证它们的值：

```java
@NotCompromised
private String password;
```

```java
@Autowired
private Validator validator;

UserCreationRequestDto request = new UserCreationRequestDto();
request.setEmailId(RandomString.make() + "@taketoday.it");
request.setPassword(PasswordCheckerSimulator.FAILURE_KEYWORD + RandomString.make());

Set<ConstraintViolation<UserCreationRequestDto>> violations = validator.validate(request);

assertThat(violations).isNotEmpty();
assertThat(violations)
    .extracting(ConstraintViolation::getMessage)
    .contains("The provided password is compromised and cannot be used.");
```

## 8. 总结

在本文中，我们探讨了如何使用Spring Security的CompromisedPasswordChecker API来检测和阻止泄露密码的使用，从而增强应用程序的安全性。

我们讨论了如何配置默认的HaveIBeenPwnedRestApiPasswordChecker实现。我们还讨论了如何针对特定环境对其进行自定义，甚至实现我们自己的自定义泄露密码检查逻辑。

总之，检查泄露的密码可以为我们用户的帐户增加一层额外的保护，以防潜在的安全攻击。