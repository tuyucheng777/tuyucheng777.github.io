---
layout: post
title:  使用Spring Security防止用户名枚举攻击
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1.概述

在本教程中，我们将概括描述枚举攻击。更具体地说，我们将探索针对Web应用程序的用户名枚举攻击。而且，最重要的是，我们将探索通过Spring Security处理它们的选项。

## 2.解释枚举攻击

枚举在技术上意味着集合中所有项目的完整有序列表。尽管此定义仅限于数学，但其本质使其成为一种强大的黑客工具。枚举通常会暴露可用于开发的攻击媒介。在这种情况下，它通常被称为资源枚举。

顾名思义，资源枚举是一种从任何主机收集资源列表的方法。这些资源可以是任何有价值的东西，包括用户名、服务或页面。这些资源可以暴露主机中的潜在漏洞。

现在，可以有几种可能的方法，探索或什至未探索，来利用这些漏洞。

## 3.针对Web应用程序的流行枚举攻击

在Web应用程序中，最常使用的枚举攻击之一是用户名枚举攻击。这基本上使用Web应用程序的任何显式或隐式功能来收集有效的用户名。攻击者可能会使用流行的用户名选择来攻击Web应用程序。

现在，Web应用程序中的哪种功能可以揭示用户名是否有效？老实说，它可以尽可能多变。它可能是设计的功能，例如，注册页面让用户知道用户名已被占用。

或者，这可能与使用有效用户名的登录尝试与使用无效用户名的登录尝试所花费的时间大不相同这一事实一样隐含。

## 4.设置模拟用户名枚举攻击

我们将使用一个使用SpringBoot和Spring Security的简单用户Web应用程序来演示这些攻击媒介。此Web应用程序将具有支持演示的最小功能集。[前面的教程中详细讨论了如何设置这样的应用程序](https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)。

Web应用程序的常见功能通常会泄露可用于发起枚举攻击的信息。让我们通过它们。

### 4.1.用户注册

用户注册需要一个唯一的用户名，为简单起见，通常选择电子邮件地址。现在，如果我们选择一个已经存在的电子邮件，应用程序应该这样告诉我们：

![](/assets/images/2023/springsecurity/springsecurityenumerationattacks01.png)

再加上电子邮件列表并不难获得，这可能会导致用户名枚举攻击，以找出应用程序中的有效用户名。

### 4.2.用户登录

同样，当我们尝试登录应用程序时，它要求我们提供用户名和密码。现在，如果我们提供的用户名不存在，应用程序可能会将此信息返回给我们：

![](/assets/images/2023/springsecurity/springsecurityenumerationattacks02.png)

和以前一样，这很简单，可以用来进行用户名枚举攻击。

### 4.3.重设密码

重置密码通常用于将密码重置链接发送到用户的电子邮件。现在，这将再次要求我们提供用户名或电子邮件：

![](/assets/images/2023/springsecurity/springsecurityenumerationattacks03.png)

如果此用户名或电子邮件在应用程序中不存在，应用程序将这样通知，从而导致与我们之前看到的类似的漏洞。

## 5.防止用户名枚举攻击

可以有多种方法来防止用户名枚举攻击。其中许多我们可以通过简单调整功能来实现，例如网络应用程序上的用户消息。

此外，随着时间的推移，Spring Security已经足够成熟，可以支持处理许多此类攻击媒介。有开箱即用的功能和扩展点来创建自定义保护措施。我们将探索其中的一些技术。

让我们来看看可用于防止此类攻击的流行选项。请注意，并非所有这些解决方案都适合甚至可能适用于Web应用程序的每个部分。随着我们的进行，我们将更详细地讨论这个问题。

### 5.1.调整消息

首先，我们必须排除无意中提供比要求更多信息的所有可能性。这在注册中会很困难，但在登录和重置密码页面中却相当简单。

例如，我们可以轻松地将登录页面的消息抽象化：

![](/assets/images/2023/springsecurity/springsecurityenumerationattacks04.png)

我们可以对密码重置页面的消息进行类似的调整。

### 5.2.包括验证码

虽然调整消息在某些页面上效果很好，但有些页面(如注册)很难做到。在这种情况下，我们可以使用另一个名为[CAPTCHA](https://www.baeldung.com/cs/captcha-intro)的工具。

现在，在这一点上，值得注意的是，由于有大量的可能性要经过，任何枚举攻击很可能都是机器人攻击。因此，检测人类或机器人的存在可以帮助我们防止攻击。验证码是实现这一目标的一种流行方式。

有几种可能的方法可以在Web应用程序中实施或集成CAPTCHA服务。其中一项服务是[Google的reCAPTCHA，它可以很容易地集成到注册页面上](https://www.baeldung.com/spring-security-registration-captcha)。

### 5.3.速率限制

虽然CAPTCHA很好地达到了目的，但它确实增加了延迟，更重要的是，给合法用户带来了不便。这与登录等常用页面更相关。

一种有助于防止对登录等常用页面进行机器人攻击的技术是速率限制。速率限制是指防止在某个阈值之后对资源的连续尝试。

例如，我们可以在3次登录尝试失败后的一天内阻止来自特定IP的请求：

![](/assets/images/2023/springsecurity/springsecurityenumerationattacks05.png)

Spring Security使这变得特别方便。

我们首先为AuthenticationFailureBadCredentialsEvent和AuthenticationSuccessEvent定义侦听器。这些侦听器调用一项服务，该服务记录来自特定IP的失败尝试次数。一旦违反设定的阈值，后续请求将在UserDetailsService中被阻止。

[另一个教程中提供了有关此方法的详细讨论](https://www.baeldung.com/spring-security-block-brute-force-authentication-attempts)。

### 5.4.地理限制

此外，我们可以在注册期间按国家/地区捕获用户的位置。我们可以使用它来验证来自不同位置的登录尝试。如果我们检测到异常位置，可以采取适当的措施：

-有选择地启用验证码
-强制执行升级身份验证(作为多因素身份验证的一部分)
-要求用户安全地验证位置
-在连续请求时暂时阻止用户

同样，Spring Security通过其扩展点，可以在AuthenticationProvider中插入自定义位置验证服务。[在之前的教程中已经详细描述了它的一种特殊风格](https://www.baeldung.com/spring-security-restrict-authentication-by-geography)。

### 5.5.多重身份验证

最后，我们应该注意，基于密码的身份验证通常是第一步，并且在大多数情况下是唯一需要的步骤。但应用程序采用多因素身份验证机制以提高安全性的情况并不少见。对于在线银行等敏感应用程序尤其如此。

涉及多因素身份验证时，有许多可能的因素：

-知识因素：这是指用户知道什么，比如PIN
-拥有因素：这是指用户拥有的东西，例如令牌或智能手机
-固有因素：这是指用户固有的东西，比如指纹

Spring Security在这里也很方便，因为它允许我们插入自定义AuthenticationProvider。GoogleAuthenticator应用程序是实现额外拥有因素的流行选择。这允许用户在其智能手机的应用程序上生成一个临时令牌，并在任何应用程序中使用它进行身份验证。显然，这需要在应用程序中预先设置用户，无论是在注册期间还是之后。

[在Spring安全应用程序中集成GoogleAuthenticator已在之前的教程中进行了详细介绍](https://www.baeldung.com/spring-security-two-factor-authentication-with-soft-token)。

更重要的是，像多因素身份验证这样的解决方案只有在应用程序需要时才适用。因此，我们不应该主要使用它来防止枚举攻击。

### 5.6.处理时间延迟

在处理像登录这样的请求时，检查用户名是否存在通常是我们做的第一件事。如果用户名不存在，请求会立即返回错误。相反，具有有效用户名的请求将涉及许多进一步的步骤，例如密码匹配和角色验证。当然，对这两种情况做出回应的时间可能会有所不同。

现在，即使我们提取错误消息以隐藏用户名是否有效的事实，处理时间的显着差异也可能会提示攻击者。

此问题的可能解决方案是添加强制延迟以排除处理时间的差异。但是，由于这不是肯定会发生的问题，因此我们应该只在必要时才采用此解决方案。

## 6.总结

虽然我们介绍了很多关于用户名枚举攻击的技巧，但很自然地会问，什么时候使用什么？显然，对此没有唯一的答案，因为它主要取决于应用程序的类型及其要求。

一些事情，比如给用户的消息，必须尽可能少地泄露信息。此外，限制对登录等资源的连续失败尝试是明智的。

但是，只有在要求认为有必要时，我们才应采取任何额外措施。我们还应该理性地权衡它们与对可用性的威慑。

此外，重要的是要认识到我们可以对不同的资源应用这些措施的任意组合以有选择地保护它们。

## 七.总结

在本教程中，我们讨论了枚举攻击——尤其是用户名枚举攻击。我们通过使用Spring Security的简单SpringBoot应用程序的镜头看到了这一点。

我们研究了几种方法来逐步解决用户名枚举攻击的问题。

最后，我们讨论了这些措施在应用程序安全方面的适当性。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules)上获得。