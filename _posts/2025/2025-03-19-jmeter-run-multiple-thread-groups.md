---
layout: post
title:  在JMeter中运行多个线程组
category: load
copyright: load
excerpt: JMeter
---

## 1. 概述

使用JMeter，我们可以对场景进行分组并以不同的方式运行它们以模拟真实世界的流量。

在本教程中，我们将学习如何以及何时使用多个线程组复制真实场景，以及如何使用简单的测试计划按顺序或并行运行它们。

## 2. 创建多个线程组

线程组是JMeter的一个元素，用于控制执行测试的线程数。

JMeter测试计划中的每个线程组模拟特定的真实应用场景。

大多数基于服务器的应用程序通常具有多个场景，因此创建一个与每个用例映射的单独线程组可以使我们在测试期间更灵活地正确分配此负载。

运行多个线程组有两种方式：顺序或并行。

## 3. 按顺序运行线程组

这主要在我们希望一个接一个地执行应用场景时有用，尤其是当各个场景之间存在依赖关系时。

### 3.1 用例

假设我们有一个电子商务应用程序，用户可以在其中浏览产品、将特定产品(他们喜欢的)添加到购物车中、最后启动结帐并下达最终订单。

对于此类应用，当我们想要模拟用户旅程时，我们希望脚本遵循特定的顺序。例如，我们的脚本可能首先执行浏览产品，然后将产品添加到购物车，最后下订单。

### 3.2 配置

从测试计划中，你可以通过选中复选框Run Thread Groups consecutively (i.e. one at a time)来实现此行为： 

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups01.png)

## 4. 并行运行线程组

这主要在各种场景之间没有依赖关系的情况下有用。

测试操作同时执行，模拟被测系统上的混合负载。

### 4.1 用例

举例来说，想象一个网站，其中的新闻被分为科技新闻、市场新闻、体育新闻等类别。

该网站的首页将始终显示各个不同类别的最新头条新闻。

对于这样的应用程序，我们仍然可以创建多个线程组，以便在不同的页面上实现不同的用户负载分布。

但是，由于这些线程组是互相排斥的，因此可以同时执行它们。

### 4.2 配置

JMeter中的测试计划默认配置为并行运行多个线程组，因此我们不需要选中Run Thread Groups consecutively。

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups02.png)

## 5. 测试用例设置

要试用测试计划，我们需要一个API。我们可以使用[JSON Placeholder](https://jsonplaceholder.typicode.com/)网站提供的API，该网站提供了模拟API供我们进行实验。

我们将在测试计划中使用两种场景

+ 场景1：阅读某篇帖子
+ 场景2：创建新帖子

由于大多数最终用户对阅读帖子而不是撰写新帖子感兴趣，因此我们希望将它们作为两个独立线程组的一部分。

## 6. 将线程组添加到测试计划

### 6.1 创建基本测试计划

我们将[运行JMeter](https://www.baeldung.com/jmeter#run-jmeter)来开始。

默认情况下，JMeter会创建一个名为Test Plan的默认测试计划。让我们将此名称更新为“My Test Plan”：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups03.png)

### 6.2 添加多个线程组

要创建线程组，我们右键单击Test Plan并选择Add -> Threads (Users) -> Thread Group：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups04.png)

现在我们将创建两个线程组，从GET请求线程组开始：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups05.png)

该线程组将用于阅读特定的帖子。

我们在这里指定了一些关键参数：

- Name：GET Request Thread Group(我们要给这个线程组起的名字)
- Number of Threads：5(我们将模拟作为负载一部分的虚拟用户数量)
- Ramp up Period：10(启动并运行配置的线程数所需的时间)
- Loop Count：1(JMeter应执行特定场景的次数)

接下来，我们将创建POST请求线程组：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups06.png)

该线程组将用于创建新帖子。

这里我们指定：

- Name：POST Request Thread Group(我们要给这个线程组起的名字)
- Number of Threads：5(我们将模拟作为负载一部分的虚拟用户数量)。
- Ramp up Period：10(为特定线程组启动并运行配置的线程数所需的时间)
- Loop Count：1(JMeter应执行单个线程组中定义的特定场景的次数)

### 6.3 添加请求

现在，对于每个线程组，我们将添加一个新的HTTP请求。

要创建请求，我们右键单击Test Group并选择Add -> Sampler -> HTTP Request：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups07.png)

现在我们在GET请求线程组下创建一个请求：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups08.png)

这里我们指定：

- Name：Read Post(我们想要赋予此HTTP请求的名称)
- Comments：Reading a particular post with ID =1
- Server Name or IP：my-json-server.typicode.com 
- HTTP Request Type：GET(HTTP请求方法)
- Path：/typicode/demo/posts
- Send Parameters with the request：这里我们使用了1个参数，即id(这是检索具有特定id的帖子所需的)

现在我们在POST请求线程组下创建另一个请求：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups09.png)

这里我们指定：

- Name：Create Post(我们想要赋予此 HTTP 请求的名称)
- Comments：Create a new post with ID =p1 (by publishing it to the server)
- Server Name or IP：my-json-server.typicode.com 
- Path：/typicode/demo/posts
- Send Parameters with the request：这里我们使用了两个参数，即id和title(这些是创建新帖子所需的属性)

### 6.4 添加摘要报告

JMeter使我们能够以多种格式查看结果。

为了查看执行结果，我们将在表监听器中添加一个查看结果功能。

要创建请求，我们右键单击“est Plan”，然后选择Add -> Listener -> View Results in Table：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups10.png)

### 6.5 运行测试(并行)

现在我们按下工具栏上的Run按钮(Ctrl + R)来启动JMeter性能测试。

测试结果实时显示：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups11.png)

这表明，根据配置的线程数，读取帖子和创建帖子将一个接一个地(并行)运行。

此测试结果是并行运行多个线程组的结果，这是测试计划的默认设置(未选中复选框)：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups12.png)

### 6.6 运行测试(串行)

现在我们从测试计划中选中Run Thread Groups consecutively(i.e. one at a time)复选框：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups13.png)

现在我们再次按下工具栏上的Run按钮(Ctrl + R)来启动JMeter性能测试。

测试结果实时显示：

![](/assets/images/2025/load/jmeterrunmultiplethreadgroups14.png)

这表明映射到“Read Post”的所有线程都首先执行，然后执行“Create Post”线程。

## 7. 总结

在本教程中，我们了解了如何创建多个线程组并使用它们来模拟真实的应用程序用户负载。

我们还学习了如何顺序或并行配置多个线程组的场景。