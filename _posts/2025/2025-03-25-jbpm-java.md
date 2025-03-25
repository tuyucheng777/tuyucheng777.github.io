---
layout: post
title:  Java版jBPM指南
category: libraries
copyright: libraries
excerpt: jBPM
---

## 1. 简介

在本教程中，我们将讨论业务流程管理(BPM)系统及其在Java中作为[jBPM](https://docs.jboss.org/jbpm/release/7.20.0.Final/jbpm-docs/html_single/)系统的实现。

## 2. 业务流程管理系统

我们可以将[业务流程管理](https://en.wikipedia.org/wiki/Business_process_management)定义为范围从开发扩展到公司各个方面的领域之一。

BPM提供了对公司职能流程的可见性，这使我们能够通过使用迭代改进来找到流程图所描绘的最佳流程。改进后的流程可提高利润并降低成本。

BPM定义了它自己的目标、生命周期、实践和所有参与者之间的通用语言，即业务流程。

## 3. jBPM系统

**jBPM是Java中BPM系统的实现，它允许我们创建业务流程、执行它并监控其生命周期**。jBPM的核心是一个用Java编写的工作流引擎，它为我们提供了一个使用最新的业务流程建模符号(BPMN) 2.0规范来创建和执行流程的工具。

jBPM主要关注可执行的业务流程，这些流程有足够的细节，因此它们可以在工作流引擎上执行。

下面是我们的BPMN流程模型的执行顺序的图形流程图示例，以帮助我们理解：

![](/assets/images/2025/libraries/jbpmjava01.png)

1.  我们使用初始上下文开始执行流程，用绿色起始节点表示
2.  首先，任务1将执行
3.  完成任务1后，我们将继续执行任务2
4.  遇到红色结束节点时执行停止

## 4. jBPM项目的IDE插件

让我们看看如何在Eclipse和IntelliJ IDEA中安装插件来创建jBPM项目和BPMN 2.0流程。

### 4.1 Eclipse插件

我们需要安装一个插件来创建jBPM项目，让我们按照以下步骤操作：

1.  在“Help”部分，单击“Install New Software”
2.  添加[Drools和jBPM更新站点](https://docs.jbpm.org/7.64.0.Final/jbpm-docs/html_single/#jbpmreleasenotes)
3.  接受许可协议条款并完成插件安装
4.  重新启动Eclipse

Eclipse重新启动后，我们需要转到Windows -> Preferences -> Drools -> Drools Flow Nodes：

![](/assets/images/2025/libraries/jbpmjava02.png)

选择所有选项后，我们可以单击“Apply and Close”。现在，我们准备创建我们的第一个jBPM项目。

### 4.2 IntelliJ IDEA插件

IntelliJ IDEA默认安装了jBPM插件，但它只存在于Ultimate而不是Community版本中。

我们只需要通过单击Configure -> Settings -> Plugins -> Installed -> JBoss jBPM来启用它：

![](/assets/images/2025/libraries/jbpmjava03.png)

目前，这个IDE没有BPMN 2.0流程设计器，但我们可以从任何其他设计器导入.bpmn文件并运行它们。

## 4.3 使用JDK 21运行

JDK 21中已删除java.lang.Compiler，使用JDK 21运行，我们需要添加[mvel2](https://mvnrepository.com/artifact/org.mvel/mvel2)依赖(表达式语言库)，如下所示：

```xml
<dependency>
    <groupId>org.mvel</groupId>
    <artifactId>mvel2</artifactId>
    <version>2.5.2.Final</version>
</dependency>
```

## 5. Hello World示例

让我们亲自动手创建一个简单的Hello World项目。

### 5.1 创建一个jBPM项目

要在Eclipse中创建一个新的jBPM项目，我们将转到File -> New -> Other -> jBPM Project(Maven)。在提供我们项目的名称后，我们可以点击完成。Eclipse将为我们完成所有艰苦的工作，并将下载所需的Maven依赖来为我们创建示例jBPM项目。

要在IntelliJ IDEA中创建相同的文件，我们可以转到File -> New -> Project -> JBoss Drools，IDE将下载所有需要的依赖并将它们放在项目的lib文件夹中。

### 5.2 创建Hello World流程模型

让我们创建一个在控制台中打印“Hello World”的小型BPM流程模型。

为此，我们需要在src/main/resources下创建一个新的BPMN文件：

![](/assets/images/2025/libraries/jbpmjava04.png)

文件扩展名为.bpmn，它在BPMN设计器中打开：

![](/assets/images/2025/libraries/jbpmjava05.png)

设计器的左侧面板列出了我们之前在设置Eclipse插件时选择的节点，我们将使用这些节点来创建我们的流程模型。中间面板是工作区，我们将在其中创建流程模型。右侧是属性选项卡，我们可以在这里设置流程或节点的属性。

在这个HelloWorld模型中，我们将使用：

-   启动事件：启动流程实例所必需的
-   脚本任务：启用Java片段
-   结束事件：结束流程实例所必需的

如前所述，IntelliJ IDEA没有BPMN设计器，但我们可以导入在Eclipse或Web设计器中设计的.bpmn文件。

### 5.3 声明并创建知识库(kbase)

**所有BPMN文件都作为流程加载到kbase中，我们需要将各自的流程ID传递给jBPM引擎以便执行它们**。

我们将使用我们的kbase和BPMN文件包声明在resources/META-INF下创建kmodule.xml：

```xml
<kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">
    <kbase name="kbase" packages="cn.tuyucheng.taketoday.bpmn.process" />
</kmodule>
```

声明完成后，我们可以使用KieContainer加载kbase：

```java
KieServices kService = KieServices.Factory.get();
KieContainer kContainer = kService.getKieClasspathContainer();
KieBase kbase = kContainer.getKieBase(kbaseId);
```

### 5.4 创建jBPM运行时管理器

我们将使用org.jbpm.test包中的JBPMHelper来构建示例运行时环境。

我们需要两件事来创建环境：首先，创建EntityManagerFactory的数据源，其次，我们的kbase。

JBPMHelper有启动内存中的H2服务器和设置数据源的方法，使用相同的方法，我们可以创建EntityManagerFactory：

```java
JBPMHelper.startH2Server();
JBPMHelper.setupDataSource();
EntityManagerFactory emf = Persistence.createEntityManagerFactory(persistenceUnit);
```

一旦我们准备好了一切，我们就可以创建我们的RuntimeEnvironment：

```java
RuntimeEnvironmentBuilder runtimeEnvironmentBuilder = RuntimeEnvironmentBuilder.Factory.get().newDefaultBuilder();
RuntimeEnvironment runtimeEnvironment = runtimeEnvironmentBuilder.entityManagerFactory(emf).knowledgeBase(kbase).get();
```

使用RuntimeEnvironment，我们可以创建我们的jBPM运行时管理器：

```java
RuntimeManager runtimeManager = RuntimeManagerFactory.Factory.get()
  .newSingletonRuntimeManager(runtimeEnvironment);
```

### 5.5 执行流程实例

最后，我们将使用RuntimeManager来获取RuntimeEngine：

```java
RuntimeEngine engine = manager.getRuntimeEngine(initialContext);
```

使用RuntimeEngine，我们将创建一个知识会话并启动该流程：

```java
KieSession ksession = engine.getKieSession();
ksession.startProcess(processId);
```

该流程将启动并在IDE控制台上打印Hello World。

## 6. 总结

在本文中，我们介绍了BPM系统，使用它的Java实现-[jBPM](https://www.jbpm.org/)。