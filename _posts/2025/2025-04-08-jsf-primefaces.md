---
layout: post
title:  Primefaces简介
category: webmodules
copyright: webmodules
excerpt: Primefaces
---

## 1. 简介

Primefaces是用于[Java Server Faces(JSF)应用程序](https://www.baeldung.com/spring-jsf)的开源UI组件套件。

在本教程中，我们将介绍Primefaces，并演示如何配置它以及使用它的一些主要功能。

## 2. 概述

### 2.1 Java Server Faces

**Java Server Faces是一个面向组件的框架，用于为Java Web应用程序构建用户界面**。JSF规范通过Java Community Process正式化，是一种标准化的显示技术。

### 2.2 Primefaces

Primefaces建立在JSF之上，通过提供可添加到任何项目的预构建UI组件来支持快速应用程序开发。

除了Primefaces，还有[Primefaces Extensions](http://primefaces-extensions.github.io/)项目，这个基于社区的开源项目除了标准组件外还提供了其他组件。

## 3. 应用程序设置

为了演示一些Primefaces组件，让我们使用Maven创建一个简单的Web应用程序。

### 3.1 Maven配置

Primefaces具有轻量级配置，只有一个jar，因此首先，让我们将依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.primefaces</groupId>
    <artifactId>primefaces</artifactId>
    <version>6.2</version>
</dependency>
```

最新版本可以在[这里](https://mvnrepository.com/artifact/org.primefaces)找到。

### 3.2 控制器–命名Bean

接下来，让我们为组件创建Bean类：

```java
@Named("helloPFBean")
public class HelloPFBean {
}
```

我们需要提供一个@Named注解来将我们的控制器绑定到视图组件。

### 3.3 视图

最后，让我们在.xhtml文件中声明Primefaces命名空间：

```xhtml
<html xmlns:p="http://primefaces.org/ui">
```

## 4. 示例组件

要呈现页面，请启动服务器并导航至：

```text
http://localhost:8080/jsf/pf_intro.xhtml
```

### 4.1 PanelGrid

我们将使用PanelGrid作为标准JSF panelGrid的扩展：

```xhtml
<p:panelGrid columns="2">
    <h:outputText value="#{helloPFBean.firstName}"/>
    <h:outputText value="#{helloPFBean.lastName}" />
</p:panelGrid>
```

在这里，我们定义了一个具有两列的panelGrid，并从JSF facelets设置outputText以显示来自命名Bean的数据。

每个outputText中声明的值对应于我们在@Named中声明的firstName和lastName属性：

```java
private String firstName;
private String lastName;
```

让我们看一下第一个简单的组件：

![](/assets/images/2025/webmodules/jsfprimefaces01.png)

### 4.2 SelectOneRadio

**我们可以使用selectOneRadio组件来提供单选按钮功能**：

```xhtml
<h:panelGrid columns="2">
    <p:outputLabel for="jsfCompSuite" value="Component Suite" />
    <p:selectOneRadio id="jsfCompSuite" value="#{helloPFBean.componentSuite}">
        <f:selectItem itemLabel="ICEfaces" itemValue="ICEfaces" />
        <f:selectItem itemLabel="RichFaces" itemValue="Richfaces" />
    </p:selectOneRadio>
</h:panelGrid>
```

我们需要在支持Bean中声明值变量来保存单选按钮的值：

```java
private String componentSuite;
```

此设置将生成一个2选项单选按钮，该按钮与底层String属性componentSuite绑定：

![](/assets/images/2025/webmodules/jsfprimefaces02.png)

### 4.3 DataTable

接下来，**让我们使用dataTable组件以表格布局显示数据**：

```xhtml
<p:dataTable var="technology" value="#{helloPFBean.technologies}">
    <p:column headerText="Name">
        <h:outputText value="#{technology.name}" />
    </p:column>

    <p:column headerText="Version">
        <h:outputText value="#{technology.currentVersion}" />
    </p:column>
</p:dataTable>
```

类似地，我们需要提供一个Bean属性来保存表格的数据：

```java
private List<Technology> technologies;
```

这里我们列出了各种技术及其版本号：

![](/assets/images/2025/webmodules/jsfprimefaces03.png)

### 4.4 使用Ajax的输入文本

我们还可以**使用p:ajax为我们的组件提供Ajax功能**。

例如，让我们使用这个组件来应用模糊事件：

```xhtml
<h:panelGrid columns="3">
    <h:outputText value="Blur event " />
    <p:inputText id="inputTextId" value="#{helloPFBean.inputText}}">
        <p:ajax event="blur" update="outputTextId"
                listener="#{helloPFBean.onBlurEvent}" />
    </p:inputText>
    <h:outputText id="outputTextId"
                  value="#{helloPFBean.outputText}" />
</h:panelGrid>
```

因此，我们需要在Bean中提供属性：

```java
private String inputText;
private String outputText;
```

此外，我们还需要在Bean中为AJAX模糊事件提供一个监听器方法：

```java
public void onBlurEvent() {
    outputText = inputText.toUpperCase();
}
```

这里，我们只是将文本转换为大写来演示该机制：

![](/assets/images/2025/webmodules/jsfprimefaces04.png)

### 4.5 Button

除此之外，我们还可以**使用p:commandButton作为标准h:commandButton组件的扩展**。

例如：

```xhtml
<p:commandButton value="Open Dialog" 
    icon="ui-icon-note" 
    onclick="PF('exDialog').show();">
</p:commandButton>
```

因此，通过此配置，我们有了用于打开对话框的按钮(使用onclick属性)：

![](/assets/images/2025/webmodules/jsfprimefaces05.png)

### 4.6 Dialog

此外，**为了提供对话框的功能，我们可以使用p:dialog组件**。

让我们也使用上一个示例中的按钮来单击打开对话框：

```xhtml
<p:dialog header="Example dialog" widgetVar="exDialog" minHeight="40">
    <h:outputText value="Hello Tuyucheng!" />
</p:dialog>
```

在这种情况下，我们有一个具有基本配置的对话框，可以使用上一节中描述的commandButton来触发：

![](/assets/images/2025/webmodules/jsfprimefaces06.png)

## 5. Primefaces移动版

**Primefaces Mobile(PFM)提供了一个UI工具包来为移动设备创建Primefaces应用程序**。

因此，PFM支持针对移动设备调整的响应式设计。

### 5.1 配置

首先，我们需要在faces-config.xml中启用移动导航支持：

```xhtml
<navigation-handler>
    org.primefaces.mobile.application.MobileNavigationHandler
</navigation-handler>
```

### 5.2 命名空间

然后，要使用PFM组件，我们需要在.xhtml文件中包含PFM命名空间：

```xhtml
xmlns:pm="http://primefaces.org/mobile"
```

除了标准的Primefaces jar之外，我们的配置中不需要任何其他库。

### 5.3 RenderKit

最后，**我们需要提供RenderKit，用于在移动环境中渲染组件**。

如果应用程序内只有一个PFM页面，我们可以在页面内定义一个RenderKit：

```xhtml
<f:view renderKitId="PRIMEFACES_MOBILE" />
```

使用完整的PFM应用程序，我们可以在faces-config.xml内的应用程序范围内定义我们的RenderKit：

```xml
<default-render-kit-id>PRIMEFACES_MOBILE</default-render-kit-id>
```

### 5.4 示例页面

现在，让我们制作示例PFM页面：

```xhtml
<pm:page id="enter">
    <pm:header>
        <p:outputLabel value="Introduction to PFM"></p:outputLabel>
    </pm:header>
    <pm:content>
        <h:form id="enterForm">
            <pm:field>
                <p:outputLabel
                        value="Enter Magic Word">
                </p:outputLabel>
                <p:inputText id="magicWord"
                             value="#{helloPFMBean.magicWord}">
                </p:inputText>
            </pm:field>
            <p:commandButton
                    value="Go!" action="#{helloPFMBean.go}">
            </p:commandButton>
        </h:form>
    </pm:content>
</pm:page>
```

从中可以看出，我们使用PFM中的page、header和content组件来构建一个带有页眉的简单表单：

![](/assets/images/2025/webmodules/jsfprimefaces07.png)

此外，我们使用了支持Bean来检查用户输入和导航：

```java
public String go() {
    if(this.magicWord != null
            && this.magicWord.toUpperCase().equals("BAELDUNG")) {
        return "pm:success";
    }

    return "pm:failure";
}
```

如果单词正确，我们将导航至下一页：

```xhtml
<pm:page id="success">
    <pm:content>
        <p:outputLabel value="Correct!">
        </p:outputLabel>
        <p:button value="Back"
                  outcome="pm:enter?transition=flow">
        </p:button>
    </pm:content>
</pm:page>
```

此配置产生以下布局：

![](/assets/images/2025/webmodules/jsfprimefaces08.png)

如果出现错误的单词，我们将导航至下一页：

```xml
<pm:page id="failure">
    <pm:content>
        <p:outputLabel value="That is not the magic word">
        </p:outputLabel>
        <p:button value="Back" outcome="pm:enter?transition=flow">
        </p:button>
    </pm:content>
</pm:page>
```

此配置将生成以下布局：

![](/assets/images/2025/webmodules/jsfprimefaces09.png)

**请注意，PFM在[6.2版中已弃用](https://www.primefaces.org/homepage/mobile/)，并将在6.3版中删除，以支持响应式标准套件**。

## 6. 总结

在本教程中，我们解释了使用Primefaces JSF组件套件的好处，并演示了如何在基于Maven的项目中配置和使用Primefaces。

此外，我们还演示了专门用于移动设备的UI套件Primefaces Mobile。