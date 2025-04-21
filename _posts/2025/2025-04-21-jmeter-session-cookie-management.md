---
layout: post
title:  Apache JMeter中的Session/Cookie管理
category: load
copyright: load
excerpt: Apache JMeter
---

## 1. 简介

在本教程中，我们将学习[JMeter](https://www.baeldung.com/jmeter)如何管理会话和Cookie，以及如何设置测试计划，以便登录应用程序、访问受保护的资源以及注销。在此过程中，我们将使用HTTP Cookie管理器、CSV数据集配置和响应断言来确保我们的测试模拟真实的用户行为。

## 2. Web应用程序中的会话管理

在Web应用程序中，会话和cookie管理有多种形式，对于身份验证和用户体验至关重要。

假设一个应用程序使用[表单登录](https://www.baeldung.com/spring-security-login)，这种身份验证方式允许我们发送一个身份验证请求，成功后会返回一个包含[JSESSIONID](https://www.baeldung.com/spring-security-session)的Cookie，我们可以将其添加到后续的请求中，这样就无需在每个请求中都包含凭据了。

### 2.1 JMeter如何处理会话和Cookie

**与浏览器不同，JMeter需要显式配置才能在请求之间维护状态**。我们需要的具体组件是[HTTP Cookie管理器](https://jmeter.apache.org/usermanual/component_reference.html#HTTP_Cookie_Manager)和[HTTP头管理器](https://jmeter.apache.org/usermanual/component_reference.html#HTTP_Header_Manager)。

## 3. 配置JMeter测试计划

我们将使用这些组件来创建一个测试计划，以登录应用程序、访问受保护的资源并注销。

最后，我们将得到这样的结构：

![](/assets/images/2025/load/jmetersessioncookiemanagement01.png)

### 3.1 创建线程组

**首先，我们需要一个线程组，它控制我们在测试中使用多少个用户和迭代次数**。

让我们右键单击Test Plan，然后Add > Threads (Users) > Thread Group并修改一些属性：

![](/assets/images/2025/load/jmetersessioncookiemanagement02.png)

- Stop Thread：由于我们需要登录才能访问应用程序，因此我们希望测试在第一次出现错误时停止。如果登录因任何原因失败，**此选项有助于减少生成的请求数**。
- Number of Threads：**此参数用于测试应用程序是否能够同时处理多个会话**，由于我们需要每个用户的有效凭证，因此我们不能超过数据库中用户数的限制，稍后我们将配置该限制。
- Loop Count：使用任何大于1的数字有助于验证我们的应用程序是否处理回访用户。
- Same user on each iteration：**此选项控制每个用户是否在测试计划的多次迭代中保持其身份**，如果未选中，则在下一次迭代开始时不会保留Cookie。由于我们的计划以登录开始并以注销结束，因此此选项的值无关紧要。

我们可以将其他选项保留为默认值，因为它们与会话管理无关。

### 3.2 使用CSV文件加载用户

由于我们正在与多个用户一起运行测试，因此最有效和最安全的加载方式是将它们包含在CSV文件中。

让我们创建一个名为users.csv的文件并包含3个用户，使用第一行作为标头：

```csv
username,password
alex_foo,password123
jane_bar,password213
john_baz,password321
```

然后，我们右键单击线程组，选择Add > Config Element > CSV Data Set Config：

![](/assets/images/2025/load/jmetersessioncookiemanagement03.png)

**在此组件中，我们唯一不会保留默认值的属性是Filename，我们将使用它来选择CSV文件**，其他值得注意的选项包括：

- Variable Names：**这将覆盖使用文件第一行的列名作为变量名的默认行为**，与“Ignore first line”结合使用时，如果我们想要使用不同的变量名，这将非常有用。
- Recycle on EOF：每个新线程都会读取文件中的下一行，**如果此选项为false，则当读取完所有行后，值将返回“EOF”，供下一个用户读取**。否则，它将循环回到文件开头。

这里定义的变量几乎可以在任何地方使用，我们可以为稍后将使用的请求和断言元素输入值。

### 3.3 添加HTTP Cookie管理器

最后，我们将添加另一个Config Element，然后选择HTTP Cookie Manager，**此组件启用JMeter中的会话管理**：

![](/assets/images/2025/load/jmetersessioncookiemanagement04.png)

- Clear cookies each iteration：这与“Same user on each iteration”选项具有相同的效果，因此当我们选中下一个选项时不可用。
- Use Thread Group configuration to control cookie clearing：顾名思义，这考虑了我们之前定义的“Same user on each iteration”。

## 4. 创建登录请求

我们的第一个请求是登录表单，让我们右键单击线程组，然后选择Add > Sampler > HTTP Request：

![](/assets/images/2025/load/jmetersessioncookiemanagement05.png)

让我们检查一下与我们的场景最相关的配置：

- Protocol：只有当我们的应用程序使用[HTTPS](https://www.baeldung.com/spring-channel-security-https)时，我们才需要更改它。
- Server Name or IP：应指向我们的服务器地址。
- Method：我们的应用程序使用HTTP POST方法登录页面。
- Path：/login，**此路径是我们的登录端点**。
- Follow Redirects：只有当登录重定向到我们想要进行测试断言的页面时，才应该勾选此项。否则，为了提高性能，我们应该不勾选它。

最后，Body Data是一个查询字符串，用于定义用户凭据。**由于我们使用的是CSV文件组件，因此可以在此处引用当前游标的变量**。此外，我们使用Spring Security登录表单的默认参数名称：

```text
username=${username}&password=${password}
```

### 4.1 包含HTTP标头管理器

我们需要将登录请求的Content-Type标头定义为application/x-www-form-urlencoded，为此，我们将右键单击登录请求，然后选择Add > Config Element > HTTP Header Manager，并将其作为名称/值对添加：

![](/assets/images/2025/load/jmetersessioncookiemanagement06.png)

**如果我们不包括它，我们的登录将会失败，因为服务器不知道如何处理请求**。

### 4.2 登录后重定向的断言请求

当登录请求成功时，它会返回302代码，重定向到应用程序中的[登录页面](https://www.baeldung.com/spring-security-login#3-the-landing-page-on-success)。为了断言这一点，我们可以右键单击我们的登录请求，然后选择Add > Assertions > Response Assertion：

![](/assets/images/2025/load/jmetersessioncookiemanagement07.png)

这里，我们结合使用了“Response Code”和“Equals”选项，并在“Patterns to Test”字段中输入302代码，其他选项保留默认值。

### 4.3 断言没有登录错误

如果登录失败，响应中会包含指向错误页面的Location头，让我们添加另一个Response Assertion：

![](/assets/images/2025/load/jmetersessioncookiemanagement08.png)

- Response Headers：**这定义了我们要检查某个响应头的值**，我们不需要设置响应头的名称。
- Contains + Not：**标头值不应包含“Patterns to Test”字段中指定的字符串**。
- Patterns to Test：Location标头包含单词“error”并在此处匹配。
- Custom failure message：我们还可以在此处引用当前username变量，生成更具描述性的JMeter错误消息。

## 5. 访问受保护的资源

**有了cookie管理器，所有后续请求都会自动包含会话cookie，从而授予对受保护资源的访问权限**：

![](/assets/images/2025/load/jmetersessioncookiemanagement09.png)

然后，我们将检查响应是否与预期值匹配。**在本例中，我们指定的安全端点Path返回当前登录用户的名称**，其他选项我们将保留默认值。

因此，让我们将username变量包含在新的Response Assertion中的Patterns to Test字段中：

![](/assets/images/2025/load/jmetersessioncookiemanagement10.png)

- Text Response：我们将以纯文本形式检查回复
- Matches：我们想要精确匹配

## 6. 创建注销请求

在迭代结束时，我们将添加指向注销端点的注销请求：

![](/assets/images/2025/load/jmetersessioncookiemanagement11.png)

**并使用与登录断言类似的逻辑来检查我们是否成功注销，其中我们检查其中一个标头是否包含“logout”**：

![](/assets/images/2025/load/jmetersessioncookiemanagement12.png)

现在，我们有一个经过身份验证的用户与应用程序同时交互的完整测试周期。

## 7. 探索请求

为了直观地了解底层发生的情况，我们右键单击“Thread Group”，然后选择Add > Listener > View Results Tree。**运行测试并检查登录请求后，单击Response data > Response headers即可看到JSESSIONID**：

![](/assets/images/2025/load/jmetersessioncookiemanagement13.png)

**我们还可以看到，当单击Request > Request Body时，cookie在我们的请求中被发送回受保护的资源**：

![](/assets/images/2025/load/jmetersessioncookiemanagement14.png)

最后，我们可以在Response Data > Response Body下验证预期的响应：

![](/assets/images/2025/load/jmetersessioncookiemanagement15.png)

## 8. 总结

在本文中，我们探讨了JMeter如何处理会话和Cookie管理，从而使我们能够模拟真实的身份验证场景。我们配置了一个测试计划，用于登录、访问受保护的资源以及注销，以确保我们的应用程序正确维护和使用户会话失效。