---
layout: post
title:  Structurizr简介
category: libraries
copyright: libraries
excerpt: Structurizr
---

## 1. 简介

本文介绍了Structurizr，它是一种**为基于[C4模型](https://www.structurizr.com/help/c4)的架构定义和可视化提供编程方法的工具**。

Structurizr打破了UML等架构图编辑器的传统拖放方法，并允许我们使用我们最熟悉的工具Java来描述我们的架构工件。

## 2. 入门

首先，让我们将[structurizr-core](https://search.maven.org/search?q=a:structurizr-core)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.structurizr</groupId>
    <artifactId>structurizr-core</artifactId>
    <version>1.0.0</version>
</dependency>
```

## 3. 系统

让我们开始建模一个示例架构，假设我们正在构建一个具有欺诈检测功能的支付终端，供商家用于清算付款。

首先，我们需要创建一个Workspace和一个Model：

```java
Workspace workspace = new Workspace("Payment Gateway", "Payment Gateway");
Model model = workspace.getModel();
```

我们还在该模型中定义了一个用户和两个软件系统：

```java
Person user = model.addPerson("Merchant", "Merchant");
SoftwareSystem paymentTerminal = model.addSoftwareSystem("Payment Terminal", "Payment Terminal");
user.uses(paymentTerminal, "Makes payment");
SoftwareSystem fraudDetector = model.addSoftwareSystem("Fraud Detector", "Fraud Detector");
paymentTerminal.uses(fraudDetector, "Obtains fraud score");
```

现在我们的系统已经定义好了，我们可以创建一个视图：

```java
ViewSet viewSet = workspace.getViews();

SystemContextView contextView = viewSet.createSystemContextView(paymentTerminal, "context", "Payment Gateway Diagram");
contextView.addAllSoftwareSystems();
contextView.addAllPeople();
```

这里我们创建了一个包含所有软件系统和人员的视图，现在需要渲染视图。

## 4. 通过PlantUML查看

在上一节中，我们创建了一个简单的支付网关的视图。

下一步是创建一个人性化的图表，对于已经使用[PlantUML](http://plantuml.com/)的组织来说，最简单的解决方案可能是指示Structurizr执行PlantUML导出：

```java
StringWriter stringWriter = new StringWriter();
PlantUMLWriter plantUMLWriter = new PlantUMLWriter();
plantUMLWriter.write(workspace, stringWriter);
System.out.println(stringWriter.toString());
```

此处，生成的标记会打印到屏幕上，但也可以很容易地发送到文件。以这种方式呈现数据会生成下图：

![](/assets/images/2025/libraries/structurizr01.png)

## 5. 通过Structurizr网站查看

还有另一种渲染图表的选项，可以通过客户端API将架构视图发送到Structurizr网站，然后将使用其丰富的UI生成图表。

让我们创建一个API客户端：

```java
StructurizrClient client = new StructurizrClient("key", "secret");
```

key和secret参数是从其网站上的工作区仪表板获取的，然后可以通过以下方式引用工作区：

```java
client.putWorkspace(1337, workspace);
```

当然，我们需要在网站上注册并创建一个工作区。单个工作区的基础账户是免费的，同时，也有商业计划可供选择。

## 6. 容器

让我们通过添加一些容器来扩展我们的软件系统。在C4模型中，容器可以是Web应用程序、移动应用程序、桌面应用程序、数据库和文件系统：几乎任何包含代码和/或数据的东西。

首先，我们为支付终端创建一些容器：

```java
Container f5 = paymentTerminal.addContainer("Payment Load Balancer", "Payment Load Balancer", "F5");
Container jvm1 = paymentTerminal.addContainer("JVM-1", "JVM-1", "Java Virtual Machine");
Container jvm2 = paymentTerminal.addContainer("JVM-2", "JVM-2", "Java Virtual Machine");
Container jvm3 = paymentTerminal.addContainer("JVM-3", "JVM-3", "Java Virtual Machine");
Container oracle = paymentTerminal.addContainer("oracleDB", "Oracle Database", "RDBMS");
```

接下来，我们定义这些新创建的元素之间的关系：

```java
f5.uses(jvm1, "route");
f5.uses(jvm2, "route");
f5.uses(jvm3, "route");

jvm1.uses(oracle, "storage");
jvm2.uses(oracle, "storage");
jvm3.uses(oracle, "storage");
```

最后，创建一个可以提供给渲染器的容器视图：

```java
ContainerView view = workspace.getViews()
    .createContainerView(paymentTerminal, "F5", "Container View");
view.addAllContainers();
```

通过PlantUML呈现结果图会产生：

![](/assets/images/2025/libraries/structurizr02.png)

## 7. 组件

C4模型中的下一级细节由组件视图提供，创建组件视图与我们之前所做的类似。

首先，我们在容器中创建一些组件：

```java
Component jaxrs = jvm1.addComponent("jaxrs-jersey", "restful webservice implementation", "rest");
Component gemfire = jvm1.addComponent("gemfire", "Clustered Cache Gemfire", "cache");
Component hibernate = jvm1.addComponent("hibernate", "Data Access Layer", "jpa");
```

接下来，让我们添加一些关系：

```java
jaxrs.uses(gemfire, "");
gemfire.uses(hibernate, "");
```

最后，让我们创建视图：

```java
ComponentView componentView = workspace.getViews()
    .createComponentView(jvm1, JVM_COMPOSITION, "JVM Components");

componentView.addAllComponents();
```

通过PlantUML呈现的结果图如下：

![](/assets/images/2025/libraries/structurizr03.png)

## 8. 组件提取

对于使用Spring框架的现有代码库，Structurizr提供了一种自动提取Spring注解组件并将它们添加到架构工件的方法。

要使用此功能，我们需要添加另一个依赖：

```xml
<dependency>
    <groupId>com.structurizr</groupId>
    <artifactId>structurizr-spring</artifactId>
    <version>1.0.0-RC5</version>
</dependency>
```

接下来，我们需要创建一个配置有一个或多个解析策略的ComponentFinder。解析策略会影响哪些组件将添加到模型、依赖树遍历的深度等。

**我们甚至可以插入自定义解析策略**：

```java
ComponentFinder componentFinder = new ComponentFinder(
    jvm, "cn.tuyucheng.taketoday.structurizr",
    new SpringComponentFinderStrategy(
        new ReferencedTypesSupportingTypesStrategy()
    ),
    new SourceCodeComponentFinderStrategy(new File("/path/to/base"), 150));
```

最后，我们启动查找器：

```java
componentFinder.findComponents();
```

上面的代码扫描包cn.tuyucheng.taketoday.structurizr以查找Spring标注的Bean，并将它们作为组件添加到容器JVM中。不用说，我们可以自由地实现我们自己的扫描器、JAX-RS注解资源甚至Google Guice绑定器。

下面是示例项目中一个简单图表的示例：

![](/assets/images/2025/libraries/structurizr04.png)

## 9. 总结

本快速教程涵盖了Structurizr Java项目的基础知识。