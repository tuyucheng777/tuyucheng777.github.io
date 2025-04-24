---
layout: post
title:  Java版Stripe API简介
category: saas
copyright: saas
excerpt: Stripe
---

## 1. 概述

**[Stripe](https://stripe.com/)是一种基于云的服务，使企业和个人能够通过互联网接收付款**，并提供客户端库(JavaScript和原生移动)和服务器端库(Java、Ruby、Node.js等)。

Stripe提供了一个抽象层，降低了收款的复杂性。因此，**我们不需要直接处理信用卡信息，而是处理一个象征着授权扣款的令牌**。

在本教程中，我们将创建一个示例Spring Boot项目，该项目允许用户输入信用卡，然后使用[Java的Stripe API](https://stripe.com/docs/api?lang=java)从该卡中扣除一定金额。

## 2. 依赖

为了在项目中使用[Stripe API Java](https://github.com/stripe/stripe-java)，我们在pom.xml中添加相应的依赖：

```xml
<dependency>
    <groupId>com.stripe</groupId>
    <artifactId>stripe-java</artifactId>
    <version>4.2.0</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/com.stripe/stripe-java)中找到它的最新版本。

对于我们的示例项目，我们将利用spring-boot-starter-parent：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
</parent>
```

我们还将使用[Lombok](https://www.baeldung.com/intro-to-project-lombok)来减少样板代码，并使用[Thymeleaf](https://www.baeldung.com/thymeleaf-in-spring-mvc)作为提供动态网页的模板引擎。

由于我们使用spring-boot-starter-parent来管理这些库的版本，因此我们不必在pom.xml中包含它们的版本：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

请注意，**如果你使用的是NetBeans，则可能需要明确使用版本1.16.16的Lombok**，因为Spring Boot 1.5.2提供的Lombok版本中的错误导致NetBeans生成大量错误。

## 3. API密钥

在我们与Stripe通信并执行信用卡收费之前，我们需要[注册一个Stripe帐户](https://dashboard.stripe.com/register)并获取机密/公共Stripe API密钥。

确认账户后，我们将登录并访问[Stripe仪表板](https://dashboard.stripe.com/dashboard)，然后在左侧菜单中选择“API keys”：

![](/assets/images/2025/saas/javastripeapi01.png)

将会有两对密钥/公钥-[一对用于test，一对用于live](https://stripe.com/docs/dashboard#livemode-and-testing)，让我们将此选项卡保持打开状态，以便稍后使用这些密钥。

## 4. 一般流程

信用卡收费将通过5个简单的步骤完成，涉及前端(在浏览器中运行)、后端(我们的Spring Boot应用程序)和Stripe：

1. 用户进入结帐页面并点击“Pay with Card”
2. 向用户显示Stripe Checkout覆盖对话框，其中填写信用卡详细信息
3. 用户确认“Pay <amount\>”，这将：
   - 将信用卡发送至Stripe
   - 在响应中获取一个令牌，该令牌将附加到现有表单中
   - 将该表单连同金额、公共API密钥、电子邮件和令牌一起提交到我们的后端
4. 我们的后端使用令牌、金额和机密API密钥联系Stripe
5. 后端检查Stripe响应并向用户提供操作的反馈

![](/assets/images/2025/saas/javastripeapi02.png)

我们将在以下章节中更详细地介绍每个步骤。

## 5. 结帐表单

[Stripe Checkout](https://stripe.com/checkout)是一个[可定制、支持移动设备且可本地化的组件](https://stripe.com/docs/checkout)，它能渲染表单来输入信用卡信息，通过引入和配置“checkout.js”，它负责：

- “Pay with Card”按钮效果图

  ![](/assets/images/2025/saas/javastripeapi03.png)
- 支付覆盖对话框渲染(点击“Pay with Card”后触发)

  ![](/assets/images/2025/saas/javastripeapi04.png)
- 信用卡验证
- “记住我”功能(将卡与手机号码关联)
- 将信用卡信息发送到Stripe，并用随附表单中的令牌替换它(点击“Pay <amount\>”后触发)

**如果我们需要对结帐表单进行比Stripe Checkout提供的更多的控制，那么我们可以使用[Stripe Elements](https://stripe.com/docs/stripe-js#examples**)。

接下来，我们将分析准备表单的控制器，然后分析表单本身。

### 5.1 控制器

让我们首先创建一个控制器来准备结帐表单所需的必要信息的模型。

首先，**我们需要从Stripe仪表盘复制公钥的测试版本**，并将其用于将STRIPE_PUBLIC_KEY定义为环境变量。然后，我们将此值用于stripePublicKey字段。

我们还在这里手动设置货币和金额(以美分表示)，仅仅是为了演示目的，但在实际应用中，我们可能会设置可用于获取实际值的产品/销售ID。

然后，我们将发送到包含结帐表单的结帐视图：

```java
@Controller
public class CheckoutController {

    @Value("${STRIPE_PUBLIC_KEY}")
    private String stripePublicKey;

    @RequestMapping("/checkout")
    public String checkout(Model model) {
        model.addAttribute("amount", 50 * 100); // in cents
        model.addAttribute("stripePublicKey", stripePublicKey);
        model.addAttribute("currency", ChargeRequest.Currency.EUR);
        return "checkout";
    }
}
```

关于Stripe API密钥，你可以将它们定义为每个应用程序(测试与实时)的环境变量。

**与任何密码或敏感信息一样，最好将密钥放在版本控制系统之外**。

### 5.2 表单

通过添加一个带有脚本的表单，并正确配置数据属性，包含“Pay with Card”按钮和结帐对话框：

```html
<form action='/charge' method='POST' id='checkout-form'>
    <input type='hidden' th:value='${amount}' name='amount' />
    <label>Price:<span th:text='${amount/100}' /></label>
    <!-- NOTE: data-key/data-amount/data-currency will be rendered by Thymeleaf -->
    <script
            src='https://checkout.stripe.com/checkout.js'
            class='stripe-button'
            th:attr='data-key=${stripePublicKey}, 
         data-amount=${amount}, 
         data-currency=${currency}'
            data-name='Tuyucheng'
            data-description='Spring course checkout'
            data-image
                    ='https://www.tuyucheng.com/wp-content/themes/tuyucheng/favicon/android-chrome-192x192.png'
            data-locale='auto'
            data-zip-code='false'>
    </script>
</form>
```

“checkout.js”脚本在提交之前自动触发对Stripe的请求，然后将Stripe令牌和Stripe用户电子邮件附加为隐藏字段“stripeToken”和“stripeEmail”。

这些将与其他表单字段一起提交到我们的后端，脚本数据属性不会被提交。

我们使用Thymeleaf来呈现属性“data-key”、“data-amount”和“data-currency”。

金额(“data-amount”)仅用于显示(与“data-currency”一起使用)，其单位是所用货币的美分，因此我们将其除以100来显示。

用户请求付款后，Stripe公钥会传递给Stripe，**此处请勿使用私钥，因为它会被发送到浏览器**。

## 6. 收费操作

对于服务器端处理，我们需要定义结帐表单使用的POST请求处理程序，让我们看一下收费操作所需的类。

### 6.1 ChargeRequest实体

让我们定义ChargeRequest POJO，我们将在收费操作期间将其用作业务实体：

```java
@Data
public class ChargeRequest {

    public enum Currency {
        EUR, USD;
    }
    private String description;
    private int amount;
    private Currency currency;
    private String stripeEmail;
    private String stripeToken;
}
```

### 6.2 服务

让我们编写一个StripeService类来将实际的收费操作传达给Stripe：

```java
@Service
public class StripeService {

    @Value("${STRIPE_SECRET_KEY}")
    private String secretKey;

    @PostConstruct
    public void init() {
        Stripe.apiKey = secretKey;
    }
    public Charge charge(ChargeRequest chargeRequest)
            throws AuthenticationException, InvalidRequestException,
            APIConnectionException, CardException, APIException {
        Map<String, Object> chargeParams = new HashMap<>();
        chargeParams.put("amount", chargeRequest.getAmount());
        chargeParams.put("currency", chargeRequest.getCurrency());
        chargeParams.put("description", chargeRequest.getDescription());
        chargeParams.put("source", chargeRequest.getStripeToken());
        return Charge.create(chargeParams);
    }
}
```

如CheckoutController中所示，**secretKey字段由我们从Stripe仪表板复制的STRIPE_SECRET_KEY环境变量填充**。

一旦服务初始化完毕，此密钥将用于所有后续的Stripe操作。

Stripe库返回的对象代表[收费操作](https://stripe.com/docs/api/charges/object?lang=java)，并包含操作ID等有用数据。

### 6.3 控制器

**最后，让我们编写一个控制器，用于接收结帐表单发出的POST请求，并通过StripeService将费用提交给Stripe**。

请注意，“ChargeRequest”参数会自动使用表单中包含的请求参数“amount”、“stripeEmail”和“stripeToken”进行初始化：

```java
@Controller
public class ChargeController {

    @Autowired
    private StripeService paymentsService;

    @PostMapping("/charge")
    public String charge(ChargeRequest chargeRequest, Model model)
            throws StripeException {
        chargeRequest.setDescription("Example charge");
        chargeRequest.setCurrency(Currency.EUR);
        Charge charge = paymentsService.charge(chargeRequest);
        model.addAttribute("id", charge.getId());
        model.addAttribute("status", charge.getStatus());
        model.addAttribute("chargeId", charge.getId());
        model.addAttribute("balance_transaction", charge.getBalanceTransaction());
        return "result";
    }

    @ExceptionHandler(StripeException.class)
    public String handleError(Model model, StripeException ex) {
        model.addAttribute("error", ex.getMessage());
        return "result";
    }
}
```

成功后，我们将状态、操作ID、费用ID和余额交易ID添加到模型中，以便稍后向用户显示(第7节)，这样做是为了说明[费用对象](https://stripe.com/docs/api/charges/object)的一些内容。

我们的ExceptionHandler将处理在收费操作期间抛出的StripeException类型的异常。

如果我们需要更细粒度的错误处理，我们可以为StripeException的子类添加单独的处理程序，例如CardException、RateLimitException或AuthenticationException。

“result”视图呈现收费操作的结果。

## 7. 显示结果

用于显示结果的HTML是一个基本的Thymeleaf模板，用于显示收费操作的结果，无论收费操作是否成功，ChargeController都会将结果发送到此处：

```html
<!DOCTYPE html>
<html xmlns='http://www.w3.org/1999/xhtml' xmlns:th='http://www.thymeleaf.org'>
<head>
    <title>Result</title>
</head>
<body>
<h3 th:if='${error}' th:text='${error}' style='color: red;'></h3>
<div th:unless='${error}'>
    <h3 style='color: green;'>Success!</h3>
    <div>Id.: <span th:text='${id}' /></div>
    <div>Status: <span th:text='${status}' /></div>
    <div>Charge id.: <span th:text='${chargeId}' /></div>
    <div>Balance transaction id.: <span th:text='${balance_transaction}' /></div>
</div>
<a href='/checkout.html'>Checkout again</a>
</body>
</html>
```

成功后，用户将看到一些有关收费操作的详细信息：

![](/assets/images/2025/saas/javastripeapi05.png)

出现错误时，Stripe将返回错误消息并向用户显示：

![](/assets/images/2025/saas/javastripeapi06.png)

## 8. 总结

在本教程中，我们演示了如何使用Stripe Java API进行信用卡扣款，将来，我们可以复用这些服务器端代码来开发原生移动应用。

为了测试整个付款流程，我们不需要使用真正的信用卡(即使在测试模式下)，我们可以依靠[Stripe测试卡](https://stripe.com/docs/testing#cards)。

收费操作是Stripe Java API提供的众多功能之一，[官方API参考](https://stripe.com/docs/api)可以指导我们完成所有操作。