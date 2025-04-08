---
layout: post
title:  JSF表达式语言3.0指南
category: webmodules
copyright: webmodules
excerpt: JSF
---

## 1. 概述

在本文中，我们将介绍表达语言3.0版本(EL 3.0)的最新功能、改进和兼容性问题。

这是撰写本文时的最新版本，并附带较新的Java EE应用程序服务器(JBoss EAP 7和Glassfish 4就是实现对其支持的很好的例子)。

本文仅关注EL 3.0的开发-要了解有关表达语言的更多信息，请先阅读[EL版本2.2](https://www.baeldung.com/intro-to-jsf-expression-language)文章。

## 2. 先决条件

本文中显示的示例也已针对Tomcat 8进行了测试，要使用EL 3.0，你必须添加以下依赖：

```xml
<dependency>
   <groupId>jakarta.el</groupId>
   <artifactId>jakarta.el-api</artifactId>
   <version>3.0.3</version>
</dependency>
```

可以随时通过此[链接](https://mvnrepository.com/artifact/jakarta.el/jakarta.el-api)检查Maven仓库中的最新依赖。

## 3. Lambda表达式

最新的EL迭代为Lambda表达式提供了非常强大的支持，Lambda表达式是在Java SE 8中引入的，但EL对它的支持是在Java EE 7中提供的。

这里的实现功能齐全，在EL的使用和评估中具有很大的灵活性(以及一些隐含的风险)。

### 3.1 Lambda EL值表达式

此功能的基本用法允许我们将Lambda表达式指定为EL值表达式中的值类型：

```html
<h:outputText id="valueOutput" 
  value="#{(x->x*x*x);(ELBean.pageCounter)}"/>
```

由此扩展，你可以命名EL中的Lambda函数以便在复合语句中重复使用，就像在Java SE中的Lambda表达式中一样。复合Lambda表达式可以用分号(;)分隔：

```html
<h:outputText id="valueOutput" 
  value="#{cube=(x->x*x*x);cube(ELBean.pageCounter)}"/>
```

此代码片段将函数分配给多维数据集标识符，然后可立即重复使用。

### 3.2 将Lambda表达式传递给支持Bean

让我们更进一步：我们可以通过将逻辑封装在EL表达式中(作为Lambda)并将其传递给JSF支持Bean来获得很大的灵活性：

```html
<h:outputText id="valueOutput" 
  value="#{ELBean.multiplyValue(x->x*x*x)}"/>
```

现在我们可以将整个Lambda表达式作为jakarta.el.LambdaExpression的实例进行处理：

```java
public String multiplyValue(LambdaExpression expr){
    return (String) expr.invoke(FacesContext.getCurrentInstance().getELContext(), pageCounter);
}
```

这是一个引人注目的功能，它可以：

- 一种封装逻辑的简洁方法，提供非常灵活的函数式编程范例，上面的支持Bean逻辑可以根据从不同来源提取的值进行条件判断。
- 在可能尚未准备好升级的JDK 8之前的代码库中引入Lambda支持的一种简单方法。
- 使用新的Streams/Collections API的强大工具。

## 4. 集合API增强功能

早期版本的EL对集合API的支持有些欠缺，EL 3.0在对Java集合的支持方面引入了重大的API改进，并且与Lambda表达式一样，EL 3.0在Java EE 7中提供了JDK 8 Stream支持。

### 4.1 动态集合定义

在3.0中的新功能，我们现在可以在EL中动态定义临时数据结构：

- List：

```html
<h:dataTable var="listItem" value="#{['1','2','3']}">
    <h:column id="nameCol">
        <h:outputText id="name" value="#{listItem}"/>
    </h:column>
</h:dataTable>
```

- Set：

```html
<h:dataTable var="setResult" value="#{{'1','2','3'}}">
 ....
</h:dataTable>
```

注意：与普通Java Set一样，元素的顺序在列出时是不可预测的

- Map：

```html
<h:dataTable var="mapResult" value="#{{'one':'1','two':'2','three':'3'}}">
```

提示：教科书中定义动态Map时的一个常见错误是使用双引号(")而不是单引号作为Map键-这将导致EL编译错误。

### 4.2 高级集合操作

EL 3.0支持高级查询语义，它结合了Lambda表达式、新的Stream API和类似SQL的操作(如join和group)的强大功能。我们不会在本文中介绍这些内容，因为这些是高级主题。让我们看一个示例来展示它的强大功能：

```html
<h:dataTable var="streamResult" value="#{['1','2','3'].stream().filter(x-> x>1).toList()}">
    <h:column id="nameCol">
        <h:outputText id="name" value="#{streamResult}"/>
    </h:column>
</h:dataTable>

```

上表将使用传递的Lambda表达式过滤备用列表

```html
 <h:outputLabel id="avgLabel" for="avg" value="Average of integer list value"/>
 <h:outputText id="avg" value="#{['1','2','3'].stream().average().get()}"/>
```

输出文本avg将计算列表中数字的平均值，通过新的[Optional API](https://www.baeldung.com/java-8-new-features)(对以前版本的另一项改进)，这两个操作都是空安全的。

请记住，支持此功能不需要JDK 8，只需要JavaEE 7/EL 3.0，这意味着你可以在EL中执行大多数JDK 8 Stream操作，但不能在后备Bean Java代码中执行。

提示：你可以使用JSTL <c:set/\>标签将数据结构声明为页面级变量，并在整个JSF页面中操作该变量：

```html
 <c:set var='pageLevelNumberList' value="#{[1,2,3]}"/>
```

现在，你可以在整个页面中引用“#{pageLevelNumberList}”，就像它是真正的JSF组件或Bean一样，这允许在整个页面中进行大量重用。

```html
<h:outputText id="avg" 
    value="#{pageLevelNumberList.stream().average().get()}"/>
```

## 5. 静态字段和方法

以前版本的EL不支持静态字段、方法或枚举访问，现在情况已经发生了变化。

首先，我们必须手动将包含常量的类导入到EL上下文中，最好尽早完成，这里我们在JSF托管Bean的@PostConstruct初始化程序中执行此操作(ServletContextListener也是一个可行的候选者)：

```java
@PostConstruct
public void init() {
    FacesContext.getCurrentInstance()
            .getApplication().addELContextListener(new ELContextListener() {
                @Override
                public void contextCreated(ELContextEvent evt) {
                    evt.getELContext().getImportHandler()
                            .importClass("cn.tuyucheng.taketoday.el.controllers.ELSampleBean");
                }
            });
}
```

然后我们在所需的类中定义一个字符串常量字段(或者如果你选择，定义一个枚举)：

```java
public static final String constantField = "THIS_IS_NOT_CHANGING_ANYTIME_SOON";
```

之后我们现在可以在EL中访问该变量：

```html
<h:outputLabel id="staticLabel" for="staticFieldOutput" value="Constant field access: "/>
<h:outputText id="staticFieldOutput" value="#{ELSampleBean.constantField}"/>
```

根据EL 3.0规范，java.lang.\*之外的任何类都需要手动导入，如下所示。只有完成此操作后，类中定义的常量才可在EL中使用。理想情况下，导入是作为JSF运行时初始化的一部分完成的。

这里需要注意几点：

- 语法要求字段和方法是public、static(对于方法则是final的)

- EL 3.0规范的初始草案和发布版本之间的语法发生了变化，因此，在某些教科书中，你可能仍会发现类似以下内容：

  
  ```java
  T(YourClass).yourStaticVariableOrMethod
  ```

  这在实践中行不通(在实施周期的后期才决定进行设计变更以简化语法)

- 发布的最终语法仍然存在错误-运行这些语法的最新版本非常重要

## 6. 总结

我们研究了最新EL实现中的一些亮点，包含很多重大改进，为API带来了Lambda和Stream灵活性等新功能。

鉴于现在在EL中拥有的灵活性，记住JSF框架的设计目标之一非常重要：使用MVC模式彻底分离关注点。

因此，值得注意的是，API的最新改进可能会让我们在JSF中面临反模式，因为EL现在有能力执行真正的业务逻辑-比以前更强大。因此，在实际实施过程中牢记这一点很重要，以确保职责清晰分离。