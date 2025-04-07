---
layout: post
title:  JSF EL 2简介
category: webmodules
copyright: webmodules
excerpt: JSF
---

## 1. 简介

表达式语言(EL)是一种脚本语言，已被许多Java框架采用，例如使用[SpEL](https://www.baeldung.com/spring-expression-language)的Spring和使用JBoss EL的JBoss。

在本文中，我们将重点介绍JSF对此脚本语言的实现-Unified EL。

EL目前为3.0版本，这是一次重大升级，允许以独立模式使用处理引擎-例如在Java SE平台上，以前的版本依赖于Jakarta EE兼容的应用程序服务器或Web容器。本文讨论的是EL 2.2版本。

## 2. 立即评估和延期评估

JSF中EL的主要功能是连接JSF视图(通常是XHTML标签)和基于Java的后端，后端可以是用户创建的托管Bean，也可以是容器管理的对象(如HTTP会话)。

我们将研究EL 2.2，JSF中的EL有两种一般形式：立即语法EL和延迟语法EL。

### 2.1 立即语法EL

也称为JSP EL，这是一种脚本格式，是从Java Web应用程序开发的JSP时代保留下来的。

JSP EL表达式以美元符号($)开头，然后是左花括号({)，然后是实际表达式，最后以右花括号(})结尾：

```text
${ELBean.value > 0}
```

此语法：

1. 在页面的生命周期中仅评估一次(在开始时)，这意味着上面示例中的表达式读取的值必须在页面加载之前设置
2. 提供对Bean值的只读访问
3. 因此，需要遵守Java Bean命名约定

对于大多数用途来说，这种形式的EL用途不是十分广泛。

### 2.2 延迟执行EL

延迟执行EL是专为JSF设计的EL，它与JSP EL的主要语法区别在于它用“#”而不是“$”标记。

```text
#{ELBean.value > 0}
```

延迟EL：

1. 与JSF生命周期同步，这意味着延迟EL中的EL表达式在JSF页面渲染的不同时间点(开始和结束)进行评估
2. 提供对Bean值的读写访问，这允许使用EL在JSF backing-bean(或任何其他地方)中设置值
3. 允许程序员调用对象上的任意方法，并根据EL的版本将参数传递给这些方法

统一EL是统一延迟EL和JSP EL的规范，允许在同一页面中使用两种语法。

## 3. 统一EL

Unified EL允许两种通用的表达式，值表达式和方法表达式。

需要注意的是，以下部分将显示一些示例，这些示例都可以在应用程序中找到(请参阅最后的Github链接)，导航至：

```text
http://localhost:8080/jsf/el_intro.jsf
```

### 3.1 值表达式

值表达式允许我们读取或设置托管Bean属性，具体取决于它所放置的位置。

以下表达式将托管Bean属性读取到页面上：

```xml
Hello, #{ELBean.firstName}
```

但是，以下表达式允许我们在用户对象上设置一个值：

```xml
<h:inputText id="firstName" value="#{ELBean.firstName}" required="true"/>
```

变量必须遵循Java Bean命名约定才能进行此类处理，对于要提交的Bean的值，只需保存封装形式即可。

### 3.2 方法表达式

Unified EL提供方法表达式来在JSF页面内执行公共的非静态方法，这些方法可能有或没有返回值。

以下是一个简单的例子：

```html
<h:commandButton value="Save" action="#{ELBean.save}"/>
```

所引用的save()方法是在名为ELBean的支持Bean上定义的。

从EL 2.2开始，你还可以将参数传递给使用EL访问的方法，这可以让我们重写我们的示例：

```html
<h:inputText id="firstName" binding="#{firstName}" required="true"/>
<h:commandButton value="Save"
  action="#{ELBean.saveFirstName(firstName.value.toString().concat('(passed)'))}"/>
```

我们在这里所做的是为inputText组件创建一个页面范围的绑定表达式，并将value属性直接传递给方法表达式。

请注意，传递给方法的变量没有任何特殊符号、花括号或转义字符。

### 3.3 隐式EL对象

JSF EL引擎提供对多个容器管理对象的访问，其中一些是：

- #{Application}：也可用作#{servletContext}，这是代表Web应用程序实例的对象
- #{applicationScope}：Web应用程序范围内可访问的变量映射
- #{Cookie}：HTTP Cookie变量的映射
- #{facesContext}：FacesContext的当前实例
- #{flash}：JSF Flash作用域对象
- #{header}：当前请求中的HTTP标头映射
- #{initParam}：Web应用程序上下文初始化变量的映射
- #{param}：HTTP请求查询参数映射
- #{request}：HTTPServletRequest对象
- #{requestScope}：请求作用域的变量映射
- #{sessionScope}：会话作用域的变量映射
- #{session}：HTTPSession对象
- #{viewScope}：视图(页面)作用域的变量映射

以下简单示例通过访问header隐式对象列出了所有请求标头和值：

```xml
<c:forEach items="#{header}" var="header">
   <tr>
       <td>#{header.key}</td>
       <td>#{header.value}</td>
   </tr>
</c:forEach>
```

## 4. 可以在EL中做什么

EL用途广泛，可以用于Java代码、XHTML标签、Javascript，甚至用于JSF配置文件(如faces-config.xml文件)，让我们来看看一些具体的用例。

### 4.1 在页面标签中使用EL

EL可以出现在标准HTML标签中：

```html
<meta name="description" content="#{ELBean.pageDescription}"/>
```

### 4.2 在JavaScript中使用EL

当在Javascript或<script\>标签中遇到EL时将被解释：

```javascript
<script type="text/javascript"> var theVar = #{ELBean.firstName};</script>
```

这里将把一个支持Bean变量设置为Javascript变量。

### 4.3 使用运算符在EL中评估布尔逻辑

EL支持相当高级的比较运算符：

-   eq相等运算符，等同于“==”
-   lt小于运算符，等同于“<”
-   le小于或等于运算符，等同于“<=”
-   gt大于运算符，等同于“>”
-   ge大于或等于，等同于“>=”

### 4.4 在Backing Bean中评估EL

在辅助Bean代码中，可以使用JSF应用程序评估EL表达式，这为将JSF页面与辅助Bean连接起来开辟了无限可能。你可以检索隐式EL对象，或从辅助Bean轻松检索实际HTML页面组件或其值：

```java
FacesContext ctx = FacesContext.getCurrentInstance(); 
Application app = ctx.getApplication(); 
String firstName = app.evaluateExpressionGet(ctx, "#{firstName.value}", String.class); 
HtmlInputText firstNameTextBox = app.evaluateExpressionGet(ctx, "#{firstName}", HtmlInputText.class);
```

这使得开发人员在与JSF页面交互时具有很大的灵活性。

## 5. 在EL中不能做什么

EL< 3.0版本确实存在一些限制，以下部分将讨论其中的一些限制。

### 5.1 无重载

EL不支持使用重载，因此，在辅助Bean中使用以下方法：

```java
public void save(User theUser);
public void save(String username);
public void save(Integer uid);
```

JSF EL将无法正确评估以下表达式

```xml
<h:commandButton value="Save" action="#{ELBean.save(firstName.value)}"/>
```

JSF ELResolver将检查Bean的类定义，并选取java.lang.Class#getMethods返回的第一个方法(该方法返回类中可用的方法)。返回方法的顺序无法保证，这将不可避免地导致未定义的行为。

### 5.2 没有枚举或常量值

JSF EL < 3.0不支持在脚本中使用常量值或枚举，因此，具有以下任何一项：

```java
public static final String USER_ERROR_MESS = "No, you can’t do that";
enum Days { Sat, Sun, Mon, Tue, Wed, Thu, Fri };
```

意味着你将无法执行以下操作：

```html
<h:outputText id="message" value="#{ELBean.USER_ERROR_MESS}"/>
<h:commandButton id="saveButton" value="save" rendered="bean.offDay==Days.Sun"/>
```

### 5.3 没有内置空安全

JSF EL < 3.0不提供隐式空安全访问，有些人可能会觉得这对于现代脚本引擎来说很奇怪。

因此，如果下面表达式中的person为空，则整个表达式将失败，并出现典型的NPE。

```text
Hello Mr, #{ELBean.person.surname}"
```

## 6. 总结

我们研究了JSF EL的一些基础知识、优势和局限性。

这在很大程度上是一种多功能脚本语言，但仍有一些改进的空间；它也是将JSF视图与JSF模型和控制器绑定在一起的粘合剂。
