---
layout: post
title:  Java EE 7中的转换器、监听器和校验器
category: webmodules
copyright: webmodules
excerpt: JSF
---

## 1. 概述

JEE 7提供了一些有用的功能，例如验证用户输入、将值转换为适当的Java数据类型。

在本教程中，我们将重点介绍转换器、监听器和校验器提供的功能。

## 2. 转换器

转换器允许我们将字符串输入值转换为Java数据类型，预定义的转换器位于javax.faces.convert包中，它们与任何Java数据类型甚至标准类(如Date)兼容。

要定义整数转换器，首先我们在用作JSF表单后端的托管Bean中创建属性：

```java
private Integer age;
	 
// getters and setters
```

然后我们使用f:converter标签在表单中创建组件：

```xhtml
<h:outputLabel value="Age:"/>
<h:inputText id="Age" value="#{convListVal.age}">
    <f:converter converterId="javax.faces.Integer" />
</h:inputText>
<h:message for="Age" />
```

以类似的方式，我们创建其他数字转换器，如Double转换器：

```java
private Double average;
```

然后我们在视图中创建适当的JSF组件，请注意，我们使用变量average，然后根据名称约定使用Getter和Setter将其映射到字段：

```xhtml
<h:outputLabel value="Average:"/>
<h:inputText id="Average" value="#{convListVal.average}">
    <f:converter converterId="javax.faces.Double" />
</h:inputText>
<h:message for="Average" />
```

**如果我们想向用户提供反馈，我们需要包含一个h:message标签，以供控件用作错误消息的占位符**。

一个有用的转换器是DateTime转换器，因为它允许我们验证日期、时间并格式化这些值。

首先，与前面的转换器一样，我们使用Getter和Setter声明我们的字段：

```java
private Date myDate;
// getters and setters
```

然后我们在视图中创建组件，这里我们需要使用模式输入日期，如果不使用模式，我们会收到错误，并给出输入正确模式的示例：

```xhtml
<h:outputLabel value="Date:"/>
<h:inputText id="MyDate" value="#{convListVal.myDate}">
    <f:convertDateTime pattern="dd/MM/yyyy" />
</h:inputText>
<h:message for="MyDate" />
<h:outputText value="#{convListVal.myDate}">
    <f:convertDateTime dateStyle="full" locale="en"/>
</h:outputText>
```

在我们的例子中，我们可以转换输入日期并发送帖子数据，格式化为h:outputText中的完整日期。

## 3. 监听器

监听器允许我们监视组件中的变化。

与之前一样，我们在托管Bean中定义属性：

```java
private String name;
```

然后我们在视图中定义我们的监听器：

```xhtml
<h:outputLabel value="Name:"/>
<h:inputText id="name" size="30" value="#{convListVal.name}">
    <f:valueChangeListener type="cn.tuyucheng.taketoday.convListVal.MyListener" />
</h:inputText>
```

我们通过添加f:valueChangeListener来设置我们的h:inputText标签，并且，在监听器标签内，我们需要指定一个类，该类将用于在触发监听器时执行任务。

```java
public class MyListener implements ValueChangeListener {
    private static final Logger LOG = Logger.getLogger(MyListener.class.getName());	
        
    @Override
    public void processValueChange(ValueChangeEvent event) throws AbortProcessingException {
        if (event.getNewValue() != null) {
            LOG.log(Level.INFO, "\tNew Value:{0}", event.getNewValue());
        }
    }
}
```

监听器类必须实现ValueChangeListener接口并重写processValueChange()方法来执行监听器任务，以写入日志消息。

## 4. 校验器

我们使用校验器来验证JSF组件数据，并使用提供的一组标准类来验证用户输入。

在这里，我们定义了一个标准校验器，使我们能够检查文本字段中用户输入的长度。

首先，我们在托管Bean中创建字段：

```java
private String surname;
```

然后在视图中创建我们的组件：

```xhtml
<h:outputLabel value="surname" for="surname"/>
<h:panelGroup>
    <h:inputText id="surname" value="#{convListVal.surname}">
        <f:validateLength minimum="5" maximum="10"/>
    </h:inputText>
    <h:message for="surname" errorStyle="color:red"  />
</h:panelGroup>
```

在h:inputText标签内，我们放置了校验器，用于验证输入的长度。请记住，JSF中预定义了各种标准校验器，我们可以按照此处介绍的方式使用它们。

## 5. 测试

为了测试这个JSF应用程序，我们将使用Arquillian对Drone、Graphene和Selenium Web Driver执行功能测试。

首先，我们使用ShrinkWrap部署我们的应用程序：

```java
@Deployment(testable = false)
public static WebArchive createDeployment() {
    return (ShrinkWrap.create(
        WebArchive.class, "jee7.war").
        addClasses(ConvListVal.class, MyListener.class)).
        addAsWebResource(new File(WEBAPP_SRC, "ConvListVal.xhtml")).
        addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
}
```

然后我们测试每个组件的错误消息，以验证我们的应用程序是否正常运行：

```java
@Test
@RunAsClient
public void givenAge_whenAgeInvalid_thenErrorMessage() throws Exception {
    browser.get(deploymentUrl.toExternalForm() + "ConvListVal.jsf");
    ageInput.sendKeys("stringage");
    guardHttp(sendButton).click();
    assertTrue("Show Age error message", browser.findElements(By.id("myForm:ageError")).size() > 0);
}
```

每个组件都进行了类似的测试。

## 6. 总结

在本教程中，我们创建了JEE 7提供的转换器、监听器和校验器的实现。
