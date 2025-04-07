---
layout: post
title:  在Java Servlet中处理Cookie和Session
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在本教程中，我们将介绍如何**使用Servlet在Java中处理cookie和会话**。

此外，我们将简要描述什么是cookie，并探讨它的一些示例用例。

## 2. Cookie基础知识

简单地说，**Cookie是服务器在与客户端通信时使用的存储在客户端的一小段数据**。

它们用于在发送后续请求时识别客户端，还可用于将一些数据从一个Servlet传递到另一个Servlet。

欲了解更多详情，请参阅[本文](https://pl.wikipedia.org/wiki/HTTP_cookie)。

### 2.1 创建Cookie

Cookie类在jakarta.servlet.http包中定义。

为了将其发送到客户端，我们需要创建一个并将其添加到响应中：

```java
Cookie uiColorCookie = new Cookie("color", "red");
response.addCookie(uiColorCookie);
```

不过，它的API更加广泛–让我们来探索一下。

### 2.2 设置Cookie有效期

我们可以设置最大期限(使用方法maxAge(int))，它定义了给定cookie的有效期为多少秒：

```java
uiColorCookie.setMaxAge(60*60);
```

我们将最大使用期限设置为1小时，超过此期限后，客户端(浏览器)在发送请求时将无法使用该cookie，并且该cookie也应从浏览器缓存中删除。

### 2.3 设置Cookie域

Cookie API中另一个有用的方法是setDomain(String)。

这允许我们指定客户端应将其传送到的域名，这还取决于我们是否明确指定域名。

让我们设置cookie的域：

```java
uiColorCookie.setDomain("example.com");
```

该cookie将被传递给example.com及其子域发出的每个请求。

**如果我们没有明确指定域，它将被设置为创建cookie的域名**。

例如，如果我们从example.com创建一个cookie并将域名留空，那么它将被传送到www.example.com(不带子域)。

除了域名，我们还可以指定路径，接下来我们来看一下。

### 2.4 设置Cookie路径

该路径指定了cookie将被传送到的位置。

**如果我们明确指定路径，则Cookie将被传递到给定的URL及其所有子目录**：

```java
uiColorCookie.setPath("/welcomeUser");
```

隐式地，它将被设置为创建cookie的URL及其所有子目录。

现在让我们关注如何在Servlet中检索它们的值。

### 2.5 在Servlet中读取Cookie

Cookie由客户端添加到请求中，客户端检查其参数并决定是否可以将其传递到当前URL。

我们可以通过对传递给Servlet的请求(HttpServletRequest)调用getCookies()来获取所有cookie。

我们可以遍历这个数组并搜索我们需要的数组，例如通过比较它们的名称：

```java
public Optional<String> readCookie(String key) {
    return Arrays.stream(request.getCookies())
        .filter(c -> key.equals(c.getName()))
        .map(Cookie::getValue)
        .findAny();
}
```

### 2.6 删除Cookie

**要从浏览器中删除cookie，我们必须在响应中添加一个同名的新cookie，将maxAge值设置为0**：

```java
Cookie userNameCookieRemove = new Cookie("userName", "");
userNameCookieRemove.setMaxAge(0);
response.addCookie(userNameCookieRemove);
```

删除cookie的一个示例用例是用户注销操作-我们可能需要删除为活跃用户会话存储的一些数据。

现在我们知道了如何在Servlet中处理cookie。

接下来，我们将介绍我们经常从Servlet访问的另一个重要对象-Session对象。

## 3. HttpSession对象

HttpSession是跨不同请求存储用户相关数据的另一种选择，会话是保存上下文数据的服务器端存储。

不同会话对象之间不共享数据(客户端只能从其会话访问数据)，它也包含键值对，但与cookie相比，会话可以包含对象作为值，存储实现机制依赖于服务器。

会话通过cookie或请求参数与客户端匹配，更多信息可在[此处](https://en.wikipedia.org/wiki/Session_(computer_science))找到。

### 3.1 获取会话

我们可以直接从请求中获取HttpSession：

```java
HttpSession session = request.getSession();
```

如果会话不存在，上述代码将创建一个新会话，我们可以通过调用以下方法实现相同的目的：

```java
request.getSession(true)
```

如果我们只想获取现有会话而不是创建新会话，我们需要使用：

```java
request.getSession(false)
```

如果我们第一次访问JSP页面，则默认会创建一个新会话，我们可以通过将session属性设置为false来禁用此行为：

```html
<%@ page contentType="text/html;charset=UTF-8" session="false" %>
```

在大多数情况下，Web服务器使用Cookie进行会话管理。创建会话对象时，服务器会创建一个带有JSESSIONID键和值的Cookie，用于标识会话。

### 3.2 会话属性

会话对象提供了一系列方法来访问(创建、读取、修改、删除)为给定用户会话创建的属性：

- setAttribute(String, Object)使用键和新值创建或替换会话属性
- getAttribute(String)读取具有给定名称(键)的属性值
- removeAttribute(String)删除具有给定名称的属性

我们还可以通过调用getAttributeNames()轻松检查已经存在的会话属性。

正如我们之前提到的，我们可以从请求中检索会话对象。当我们已经拥有它时，我们可以快速执行上述方法。

我们可以创建一个属性：

```java
HttpSession session = request.getSession();
session.setAttribute("attributeKey", "Sample Value");
```

可以通过属性的键(名称)获取属性值：

```java
session.getAttribute("attributeKey");
```

当我们不再需要某个属性时，可以将其删除：

```java
session.removeAttribute("attributeKey");
```

用户会话的一个常见用例是当用户从我们的网站注销时，使它存储的整个数据无效，会话对象为此提供了一个解决方案：

```java
session.invalidate();
```

此方法从Web服务器中删除整个会话，因此我们无法再从中访问属性。

HttpSession对象有更多的方法，但我们提到的方法是最常见的。

## 4. 总结

在本文中，我们介绍了两种允许我们在对服务器的后续请求之间存储用户数据的机制-Cookie和Session。

请记住，HTTP协议是无状态的，因此必须在请求之间保持状态。