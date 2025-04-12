---
layout: post
title:  Apache Velocity简介
category: apache
copyright: apache
excerpt: Apache Velocity
---

## 1. 概述

[Velocity](http://velocity.apache.org/)是一个基于Java的模板引擎。

它是一个开源的Web框架，旨在用作MVC架构中的视图组件，它为一些现有技术(如JSP)提供了替代方案。

Velocity可用于生成XML文件、SQL、PostScript和大多数其他基于文本的格式。

在本文中，我们将探讨如何使用它来创建动态网页。

## 2. Velocity的工作原理

**Velocity的核心类是VelocityEngine**。

它使用数据模型和Velocity模板来协调读取、解析和生成内容的整个过程。

简而言之，对于任何典型的Velocity应用，我们需要遵循以下步骤：

- 初始化Velocity引擎
- 读取模板
- 将数据模型放入上下文对象中
- 将模板与上下文数据合并并渲染视图

让我们按照以下简单步骤来看一个例子：

```java
VelocityEngine velocityEngine = new VelocityEngine();
velocityEngine.init();
   
Template t = velocityEngine.getTemplate("index.vm");
    
VelocityContext context = new VelocityContext();
context.put("name", "World");
    
StringWriter writer = new StringWriter();
t.merge( context, writer );
```

## 3. Maven依赖

要使用Velocity，我们需要向Maven项目添加以下依赖：

```xml
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity</artifactId>
    <version>1.7</version>
    </dependency>
<dependency>
     <groupId>org.apache.velocity</groupId>
     <artifactId>velocity-tools</artifactId>
     <version>2.0</version>
</dependency>
```

这两个依赖的最新版本可以在这里找到：[velocity](https://mvnrepository.com/artifact/org.apache.velocity/velocity)和[speed-tools](https://mvnrepository.com/artifact/org.apache.velocity/velocity-tools)。

## 4. Velocity模板语言

Velocity模板语言(VTL)通过使用VTL引用，提供了将动态内容合并到网页中的最简单、最干净的方法。

Velocity模板中的VTL引用以$开头，用于获取与该引用关联的值；VTL还提供了一组指令，可用于操作Java代码的输出，这些指令以#开头。

### 4.1 引用

Velocity中有三种类型的引用：变量、属性和方法。接下来，我们来仔细看看它们。

- **变量**：使用#set指令或从Java对象字段返回的值在页面内定义，例如#set($message="Hello World")
- **属性**：如果属性在外部不可见，则引用对象内部的字段或属性的Getter方法。例如，"\$customer.name"指的是customer的name或getName() Getter方法。类似地，"$anArray.length"表示anArray数组的大小。
- **方法**：指Java对象的方法，例如，\$customer.getName()会调用customer.getName()方法并获取结果。同样，当我们想要获取List的大小时，可以使用"$aList.size()"

每个引用的最终值在渲染到最终输出时都会转换为字符串，稍后我们会看到更多相关示例。

### 4.2 指令

VTL提供了一组丰富的指令：

- **set**：可用于设置引用的值；该值可以分配给变量或属性引用：

  ```text
  #set ($message = "Hello World")
  #set ($customer.name = "Brian Mcdonald")
  ```

- **conditionals**：#if、#elseif和#else指令提供了一种根据条件检查生成内容的方法：

  ```html
  #if($employee.designation == "Manager")
      <h3> Manager </h3>
  #elseif($employee.designation == "Senior Developer")
      <h3> Senior Software Engineer </h3>
  #else
      <h3> Trainee </h3>
  #end
  ```

- **loops**：#foreach指令允许循环遍历对象集合：
  
  ```html
  <ul>
      #foreach($product in $productList)
          <li> $product </li>
      #end
  </ul>
  ```

- **include**：#include元素提供了将文件导入模板的功能：

  ```text
  #include("one.gif","two.txt","three.html"...)
  ```

- **parse**：#parse语句允许模板设计者导入另一个包含VTL的本地文件；然后Velocity将解析内容并呈现它：

  
  ```text
  #parse (Template)
  ```

- **evaluate**：#evaluate指令可用于动态评估VTL；这允许模板在渲染时评估字符串，例如使模板国际化：

  ```text
  #set($firstName = "David")
  #set($lastName = "Johnson")
  
  #set($dynamicsource = "$firstName$lastName")
  
  #evaluate($dynamicsource)
  ```

- **break**：#break指令停止当前执行范围的任何进一步渲染(即#foreach、#parse)

- **stop**：#stop指令停止模板的任何进一步渲染和执行

- **velocimacros**：#macro指令允许模板设计者定义VTL的重复段：

  ```html
  #macro(tablerows)
      <tr>
          <td>
          </td>
      </tr>
  #end
  ```

  现在我们可以将此宏放在模板中的任何位置，如#tablerows()：

  ```html
  #macro(tablerows $color $productList)
      #foreach($product in $productList)
          <tr>
              <td bgcolor=$color>$product.name</td>
          </tr>
      #end
  #end
  ```

### 4.3 其他功能

- **数学**：一些内置数学函数，可以在模板中使用：
  
  ```text
  #set($percent = $number / 100)
  #set($remainder = $dividend % $divisor)
  ```

- **范围运算符**：可与#set和#foreach结合使用：
  
  ```text
  #set($array = [0..10])
  
  #foreach($elem in $arr)
      $elem
  #end
  ```

## 5. Velocity Servlet

Velocity Engine的主要工作是根据模板生成内容。

引擎本身不包含任何与Web相关的功能，要实现Web应用程序，我们需要使用Servlet或基于Servlet的框架。

Velocity提供了一个开箱即用的实现VelocityViewServlet，它是Velocity-tools子项目的一部分。

为了利用VelocityViewServlet提供的内置功能，我们可以从VelocityViewServlet扩展我们的Servlet并覆盖handleRequest()方法：

```java
public class ProductServlet extends VelocityViewServlet {

    ProductService service = new ProductService();

    @Override
    public Template handleRequest(
                HttpServletRequest request,
                HttpServletResponse response,
                Context context) throws Exception {
    
        List<Product> products = service.getProducts();
        context.put("products", products);
    
        return getTemplate("index.vm");
    }
}
```

## 6. 配置

### 6.1 Web配置

现在让我们看看如何在web.xml中配置VelocityViewServlet。

我们需要指定可选的初始化参数，包括velocity.properties和toolbox.xml：

```xml
<web-app>
    <display-name>apache-velocity</display-name>
      //...
       
    <servlet>
        <servlet-name>velocity</servlet-name>
        <servlet-class>org.apache.velocity.tools.view.VelocityViewServlet</servlet-class>

        <init-param>
            <param-name>org.apache.velocity.properties</param-name>
            <param-value>/WEB-INF/velocity.properties</param-value>
        </init-param>
    </servlet>
        //...
</web-app>
```

我们还需要为这个Servlet指定映射，这个Servlet负责处理所有Velocity模板(\*.vm)的请求：

```xml
<servlet-mapping>
    <servlet-name>velocityLayout</servlet-name>
    <url-pattern>*.vm</url-pattern>
</servlet-mapping>
```

### 6.2 资源加载器

Velocity提供了灵活的资源加载器系统，允许同时运行一个或多个资源加载器：

- FileResourceLoader
- JarResourceLoader
- ClassPathResourceLoader
- URLResourceLoader
- DataSourceResourceLoader
- WebappResourceLoader

这些资源加载器在velocity.properties中配置：

```properties
resource.loader=webapp
webapp.resource.loader.class=org.apache.velocity.tools.view.WebappResourceLoader
webapp.resource.loader.path=
webapp.resource.loader.cache=true
```

## 7. Velocity模板

Velocity模板是编写所有视图生成逻辑的地方，我们使用Velocity模板语言(VTL)编写以下页面：

```html
<html>
    ...
    <body>
        <center>
        ...
        <h2>$products.size() Products on Sale!</h2>
        <br/>
            We are proud to offer these fine products
            at these amazing prices.
        ...
        #set( $count = 1 )
        <table class="gridtable">
            <tr>
                <th>Serial #</th>
                <th>Product Name</th>
                <th>Price</th>
            </tr>
            #foreach( $product in $products )
            <tr>
                <td>$count)</td>
                <td>$product.getName()</td>
                <td>$product.getPrice()</td>
            </tr>
            #set( $count = $count + 1 )
            #end
        </table>
        <br/>
        </center>
    </body>
</html>
```

## 8. 管理页面布局

Velocity为基于Velocity Tool的应用程序提供了简单的布局控制和可定制的错误屏幕。

VelocityLayoutServlet封装了渲染指定布局的功能，VelocityLayoutServlet是VelocityViewServlet的扩展。

### 8.1 Web配置

让我们看看如何配置VelocityLayoutServlet，我们定义一个Servlet来拦截Velocity模板页面的请求，并在Velocity.properties文件中定义布局相关的属性：

```xml
<web-app>
    // ...
    <servlet>
        <servlet-name>velocityLayout</servlet-name>
        <servlet-class>org.apache.velocity.tools.view.VelocityLayoutServlet</servlet-class>

        <init-param>
            <param-name>org.apache.velocity.properties</param-name>
            <param-value>/WEB-INF/velocity.properties</param-value>
        </init-param>
    </servlet>
    // ...
    <servlet-mapping>
        <servlet-name>velocityLayout</servlet-name>
        <url-pattern>*.vm</url-pattern>
    </servlet-mapping>
    // ...
</web-app>
```

### 8.2 布局模板

布局模板定义了Velocity页面的典型结构，默认情况下，VelocityLayoutServlet会在布局文件夹下搜索Default.vm文件，你可以通过覆盖以下几个属性来更改此位置：

```properties
tools.view.servlet.layout.directory = layout/
tools.view.servlet.layout.default.template = Default.vm
```

布局文件由页眉模板、页脚模板和Velocity变量$screen_content组成，用于呈现请求的Velocity页面的内容：

```html
<html>
    <head>
        <title>Velocity</title>
    </head>
    <body>
        <div>
            #parse("/fragments/header.vm")
        </div>
        <div>
            <!-- View index.vm is inserted here -->
            $screen_content
        </div>
        <div>
            #parse("/fragments/footer.vm")
        </div>
    </body>
</html>
```

### 8.3 请求屏幕中的布局规范

我们可以将特定屏幕的布局定义为页面开头的Velocity变量，为此，请在页面中添加以下代码：

```text
#set($layout = "MyOtherLayout.vm")
```

### 8.4 请求参数中的布局规范

我们可以在查询字符串中添加一个请求参数layout=MyOtherLayout.vm，VLS将找到它并在该布局中呈现屏幕，而不是搜索默认布局。

### 8.5 错误屏幕

我们可以使用VelocityLayoutServlet实现自定义的错误页面，VelocityLayoutServlet提供了两个变量\$error_cause和\$stack_trace来展示异常的详细信息。

我们在velocity.properties文件中配置错误页面：

```properties
tools.view.servlet.error.template = Error.vm
```

## 9. 总结

在本文中，我们了解了Velocity如何成为渲染动态网页的实用工具，此外，我们还了解了Velocity提供的Servlet的不同使用方法。