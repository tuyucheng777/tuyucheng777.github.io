---
layout: post
title:  在Java中的HttpServletRequest中设置参数
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

使用[Servlet API](https://www.baeldung.com/intro-to-servlets)开发Java中的Web应用程序时，HttpServletRequest对象在处理传入的HTTP请求中起着关键作用，它提供对请求的各个方面(例如参数、标头和属性)的访问。

请求参数始终由HTTP客户端提供。但是，有些情况下，我们可能需要在应用程序处理HttpServletRequest对象之前以编程方式在HttpServletRequest对象中设置参数。

需要注意的是，**HttpServletRequest缺少用于添加新参数或更改参数值的Setter方法**。在本文中，我们将探讨如何通过扩展原始HttpServletRequest的功能来实现这一点。

## 2. Maven依赖

除了标准[Java Servlet](https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api) API之外：

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.1.0</version>
    <scope>provided</scope>
</dependency>
```

我们还将在我们的用例中使用[commons-text](https://mvnrepository.com/artifact/org.apache.commons/commons-text)库，通过转义HTML实体来清理请求参数：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-text</artifactId>
    <version>1.10.0</version>
</dependency>
```

## 3. 必需的Servlet组件

在深入研究实际示例之前，让我们快速了解一下我们将要使用的某些基础Servlet组件。

### 3.1 HttpServletRequest

HttpServletRequest类是客户端和Servlet之间通信的主要方式，**它封装传入的HTTP请求**，提供对参数、标头和其他与请求相关的信息的访问。

### 3.2 HttpServletRequestWrapper

HttpServletRequestWrapper通过充当现有HttpServletRequest对象的[装饰器](https://www.baeldung.com/java-decorator-pattern)来扩展HttpServletRequest的功能，这使我们能够根据特定需求附加其他职责。

### 3.3 Filter

[Filter](https://www.baeldung.com/intercepting-filter-pattern-in-java)会在请求和响应遍历Servlet容器时捕获并处理它们，这些**过滤器旨在在Servlet执行之前调用**，使其能够更改传入请求和传出响应。

## 4. 参数清理

在HttpServletRequest中以编程方式设置参数的应用之一是清理请求参数，从而有效缓解[跨站点脚本(XSS)](https://www.baeldung.com/cs/cross-site-scripting-xss-explained)漏洞。**此过程涉及从用户输入中消除或编码潜在有害字符，从而增强Web应用程序的安全性**。

### 4.1 示例

现在，让我们详细探索这个过程。首先，我们必须设置一个Servlet过滤器来拦截请求。过滤器提供了一种在请求和响应到达目标Servlet或JSP之前对其进行修改的方法。

下面是一个Servlet过滤器的具体示例，它拦截对特定URL模式的所有请求，确保过滤器链通过原始HttpServletRequest对象返回SanitizeParametersRequestWrapper：

```java
@WebFilter(urlPatterns = {"/sanitize/with-sanitize.jsp"})
public class SanitizeParametersFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpSerlvetRequest httpReq = (HttpSerlvetRequest) request;
        chain.doFilter(new SanitizeParametersRequestWrapper(httpReq), response);
    }
}
```

SanitizeParameterRequestWrapper类扩展了HttpServletRequestWrapper，这在参数清理过程中起着至关重要的作用，该类旨在清理来自原始HttpServletRequest的请求参数，并仅将清理后的参数公开给调用JSP：

```java
public class SanitizeParametersRequestWrapper extends HttpServletRequestWrapper {

    private final Map<String, String[]> sanitizedMap;

    public SanitizeParametersRequestWrapper(HttpServletRequest request) {
        super(request);
        sanitizedMap = Collections.unmodifiableMap(
                request.getParameterMap().entrySet().stream()
                        .collect(Collectors.toMap(
                                Map.Entry::getKey,
                                entry -> Arrays.stream(entry.getValue())
                                        .map(StringEscapeUtils::escapeHtml4)
                                        .toArray(String[]::new)
                        )));
    }

    @Override
    public Map<String, String[]> getParameterMap() {
        return sanitizedMap;
    }

    @Override
    public String[] getParameterValues(String name) {
        return Optional.ofNullable(getParameterMap().get(name))
                .map(values -> Arrays.copyOf(values, values.length))
                .orElse(null);
    }

    @Override
    public String getParameter(String name) {
        return Optional.ofNullable(getParameterValues(name))
                .map(values -> values[0])
                .orElse(null);
    }
}
```

在构造函数中，我们迭代每个请求参数并使用StringEscapeUtils.escapeHtml4来清理值，处理后的参数收集在新的sanitizedMap中。

我们重写getParameter方法，以便从sanitizedMap中返回相应的参数，而不是原始请求参数。尽管getParameter方法是主要使用重点，但重写getParameterMap和getParameterValues也至关重要。**我们确保所有参数检索方法的行为一致，并在整个过程中保持安全标准**。

根据规范，getParameterMap方法保证不可修改Map，从而防止更改内部值。**因此，遵守此约定并确保重写也返回不可修改的Map非常重要**。同样，重写的getParameterValues方法返回克隆数组而不是其内部值。

现在，让我们创建一个JSP来演示，它只是在屏幕上呈现输入请求参数的值：

```html
The text below comes from request parameter "input":<br/>
<%=request.getParameter("input")%>
```

### 4.2 结果

现在，让我们在未激活清理过滤器的情况下运行JSP，我们将向请求参数注入一个脚本标记作为反射型XSS：

```text
http://localhost:8080/sanitize/without-sanitize.jsp?input=<script>alert('Hello');</script>
```

我们将看到参数中嵌入的JavaScript正在被浏览器执行：

![](/assets/images/2025/webmodules/javaservletrequestsetparameter01.png)

接下来，我们将运行净化后的版本：

```text
http://localhost:8080/sanitize/with-sanitize.jsp?input=<script>alert('Hello');</script>
```

在这种情况下，过滤器将捕获请求并将SanitizeParameterRequestWrapper传递给调用JSP。因此，弹出窗口将不再出现；相反，我们将观察到HTML实体被转义并明显呈现在屏幕上：

```html
The text below comes from request parameter "input":
<script>alert('Hello');</script>
```

## 5. 第三方资源访问

让我们考虑另一种情况，第三方模块接收请求参数locale来更改模块显示的语言。请注意，我们无法直接修改第三方模块的源代码。

在我们的示例中，我们将locale参数设置为默认系统区域设置。但是，它也可以从其他来源获取，例如HTTP会话。

### 5.1 示例

第三方模块是一个JSP页面：

```html
<%
    String localeStr = request.getParameter("locale");
    Locale currentLocale = (localeStr != null ? new Locale(localeStr) : null);
%>
The language you have selected: <%=currentLocale != null ? currentLocale.getDisplayLanguage(currentLocale) : " None"%>
```

如果提供了locale设置参数，模块将显示语言名称。

我们将为目标请求参数提供默认的系统语言Locale.getDefault().getLanguage()，为此，我们在SetParameterRequestWrapper中设置locale参数，该参数装饰原始HttpServletRequest：

```java
@WebServlet(name = "LanguageServlet", urlPatterns = "/setparam/lang")
public class LanguageServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        SetParameterRequestWrapper requestWrapper = new SetParameterRequestWrapper(request);
        requestWrapper.setParameter("locale", Locale.getDefault().getLanguage());
        request.getRequestDispatcher("/setparam/3rd_party_module.jsp").forward(requestWrapper, response);
    }
}
```

在创建SetParameterRequestWrapper时，我们将采用与上一节类似的方法。此外，我们将实现setParameter方法，以将新参数添加到现有参数Map中：

```java
public class SetParameterRequestWrapper extends HttpServletRequestWrapper {

    private final Map<String, String[]> paramMap;

    public SetParameterRequestWrapper(HttpServletRequest request) {
        super(request);
        paramMap = new HashMap<>(request.getParameterMap());
    }

    @Override
    public Map<String, String[]> getParameterMap() {
        return Collections.unmodifiableMap(paramMap);
    }

    public void setParameter(String name, String value) {
        paramMap.put(name, new String[] {value});
    }

    // getParameter() and getParameterValues() are the same as SanitizeParametersRequestWrapper
}
```

getParameter方法检索存储在paramMap中的参数而不是原始请求。

### 5.2 结果

LanguageServlet通过SetParameterRequestWrapper将locale参数传递给第三方模块，当我们在英语语言服务器上访问第三方模块时，我们将看到以下内容：

```text
The language you have selected: English
```

## 6. 总结

在本文中，我们了解了一些重要的Servlet组件，这些组件可用于Java Web应用程序开发中处理客户端请求，包括HttpServletRequest、HttpServletRequestWrapper和Filter。

通过具体的示例，**我们演示了如何扩展HttpServletRequest的功能以通过编程方式设置和修改请求参数**。

无论是通过清理用户输入来增强安全性还是与第三方资源集成，这些技术都能帮助开发人员应对各种场景。