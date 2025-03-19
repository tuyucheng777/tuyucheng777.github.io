---
layout: post
title:  分析JMeter结果
category: load
copyright: load
excerpt: JMeter
---

## 1. 概述

如果我们想提高服务性能，[Apache JMeter](https://www.baeldung.com/jmeter)测试执行会得出一些结果，我们需要分析这些结果。这些结果可以是服务方面的日志和指标，也可以是测试执行方面的JMeter结果，包括响应错误码率、吞吐量等。

JMeter提供了多种读取测试结果的方法，每种方法在不同情况下都可能有用。有许多内置选项，但如果我们想要更高级的报告，JMeter社区为此提供了许多免费插件。

**在本教程中，我们将重点分析JMeter结果。我们将研究最流行的可用选项，然后深入研究一些更常用的选项，仅使用JMeter增强工具**。

## 2. 分析JMeter结果

运行性能测试后，我们希望能够观察结果并理解它们以进行改进。

**性能测试的结果可以从两个不同的来源观察到**。首先是被测服务本身，从这个来源，我们可以收集诸如JVM内存使用情况、活动线程以及DB事务或下游REST请求的执行时间等指标。

**第二点是客户端发出请求的点，在本例中，它是我们用来执行性能测试的工具**。从这一点开始，我们可以收集有关[总响应时间](https://www.baeldung.com/java-jmeter-latency-vs-load-time)、失败请求率、发送的请求量、吞吐量等指标。

JMeter通过多种方式和不同的工具提供所需的所有信息，我们可以将它们分为三类：

- JMeter报告，包括一个特殊的[HTML报告仪表板工具](https://www.baeldung.com/jmeter-dashboard-report)
- 插件
- 托管结果服务

## 3. 使用托管服务分析JMeter结果

JMeter附带：

- GraphiteBackendListenerClient允许我们将指标发送到Graphite后端
- InfluxDBBackendListenerClient允许我们将指标发送到InfluxDB后端

**这些后端监听器客户端将结果实时存储在外部数据库中**，这使我们能够保留数据的历史记录，并使用历史记录来比较不同时间、不同版本的服务的多次执行情况等。

**我们可以使用托管结果服务(例如BlazeMeter Sense、Grafana或Taurus)从测试结果中获取报告和仪表板，并使用它们来理解结果**。这些托管服务可以是云的，也可以是自托管的。

### 3.1 使用托管服务的风险

**使用托管服务有好处，比如可以使用一些专门为创建仪表板而创建的工具**。例如，Grafana有非常丰富的仪表板，可以可视化结果。

**另一方面，这种方法会增加复杂性、风险，有时还会增加成本**。如果我们只使用开源、免费的工具，我们就无法保证服务永远存在或永远免费。

例如，BlazeMeter Sense最初是免费云服务，但后来停止了。现在，我们必须设置自己的本地BlazeMeter Sense服务器，才能将其与JMeter结果一起使用。

此外，自托管服务增加了复杂性，因为我们需要在较低层面上了解它们的设置方式，并且它们还会增加机器托管它们的成本。

此外，在大多数情况下，例如使用Grafana或Taurus来可视化JMeter结果，我们需要自行托管多个服务。这是因为我们生成的数据需要存储在某个地方，以便这些服务可以检索它们。对于自托管的Grafana，我们必须：

- 自托管数据库
- 在我们的JMeter测试中设置一个后端监听器来连接到我们的数据库
- 设置Grafana服务器并将其连接到与源相同的数据库
- 使用从JMeter导出到我们自托管数据库的指标，在Grafana UI中创建我们想要的仪表板

## 4. 使用JMeter插件分析JMeter结果

**Apache JMeter是一款非常流行的开源工具，这使得它还得到了社区创建的插件的良好支持，这些插件可以满足实际工程师和测试人员的需求**。对于JMeter结果，插件列表很长。一些比较流行的插件是：

- [JMeterPluginsCMD](https://jmeter-plugins.org/wiki/JMeterPluginsCMD/)是一个小型命令行实用程序，用于从JTL文件生成图表，它的行为就像所有图表上的右键单击上下文菜单一样。
- [图表生成器监听器](https://jmeter-plugins.org/wiki/GraphsGeneratorListener/)在测试结束时生成以下图表：随时间变化的活动线程数、随时间变化的响应时间、每秒的事务量、每秒的服务器点击量、每秒的响应码响应、随时间变化的延迟字节、随时间变化的吞吐量、响应时间与线程事务、吞吐量与线程、响应时间分布、响应时间百分位数。
- [JWeter](https://bitbucket.org/pjtr/jweter/src/master/)是另一个用于分析和可视化JMeter结果文件的流行工具。
- [JMeter结果分析插件](https://github.com/afranken/jmeter-analysis-maven-plugin)是一个Maven插件，可以解析JMeter结果XML文件并生成带有图表的详细报告。
- [UbikLoadPack可观测性插件](https://github.com/ubikingenierie/ulp-observability-plugin)允许我们从我们最喜欢的浏览器监控非GUI/CLI性能测试。

## 5. 使用JMeter报告监听器分析JMeter结果

**我们可以使用JMeter提供的不同类型的报告监听器来收集结果，这些监听器在测试计划执行期间收集数据，然后以各种格式呈现，例如纯文本、聚合数据、表格、图形等**。

我们可以为不同的范围设置监听器，比如为不同的采样器设置不同的监听器，为特定的线程组设置监听器，甚至为整个测试计划的执行设置监听器：

![](/assets/images/2025/load/jmeteranalyzeresults01.png)

在这种情况下，我们只有一个采样器，因此特定于采样器的监听器与线程组监听器的功能相同，但对于更多采样器则不是这种情况。测试计划范围的监听器会收集所有线程组和所有采样器的数据。

为了演示，我们将使用Spring Boot应用程序，并测试创建UUID并在响应体中返回它的端点。不同线程组的采样器都将指向此端点：

![](/assets/images/2025/load/jmeteranalyzeresults02.png)

在图片中，我们看到采样器“uuid-generator-endpoint-test”指向在localhost上运行的Spring Boot服务器，端口8080，路径为/api/uuid。

在我们的测试计划中添加监听器的方法是右键单击要添加监听器的项目 -> Add -> Listener：

![](/assets/images/2025/load/jmeteranalyzeresults03.png)

在这里，我们选择向“60 operations per sec, for 1m – 60 users”采样器添加一些监听器，按照前面描述的步骤进行操作。我们添加了一个View Results Tree和Table、一个Response Time Graph等，稍后我们将看到。

### 5.1 查看结果树和表监听器

**使用查看结果监听器，我们可以收集有关JMeter发出的请求和响应的指标以及有关完整事务的一些信息，如延迟、发送字节、执行时间等**。

View Results Tree列出了所有提出的请求：

![](/assets/images/2025/load/jmeteranalyzeresults04.png)

**在大多数JMeter结果监听器中，如果我们愿意，可以选择仅按错误或仅按成功过滤请求**。在我们的监听器中，我们收集所有成功和错误。**如果我们想将结果存储起来以供历史记录或稍后查看，我们还可以选择在文件中查看结果**。我们不会在演示中使用它。

以类似的方式，“View Results in Table”以表格的形式提供相同的信息：

![](/assets/images/2025/load/jmeteranalyzeresults05.png)

这样，我们可以按列排序，并且可以更轻松地浏览数据。

### 5.2 聚合报告和图表监听器

**顾名思义，聚合报告和聚合图表监听器收集与查看结果监听器相同的数据，然后对其进行聚合。然后，这些监听器以统计值的形式提供结果，例如平均值、中位数、99%百分位数、吞吐量等**。

汇总报告以表格的形式提供这些值，其中每行代表一个采样器(在我们的示例中，我们只有一个采样器)：

![](/assets/images/2025/load/jmeteranalyzeresults06.png)

聚合图与聚合报告的功能相同，但最重要的是，它提供一些值的图表，例如最小值、最大值、99％线等：

![](/assets/images/2025/load/jmeteranalyzeresults07.png)

从该图可以看出，监听器的默认JMeter图表并不是可视化结果和提供最低配置的最佳工具。这就是为什么JMeter提供仪表板报告GUI(我们稍后会看到)以及与Grafana等更合适的工具集成的原因。

### 5.3 响应时间图

**响应时间图监听器导出有关采样时间的统计信息**。例如，在REST应用程序中，JMeter接收响应需要多长时间？

Response Time Graph有两种模式，首先，我们在“Settings”模式中进行设置：

![](/assets/images/2025/load/jmeteranalyzeresults08.png)

在这里，我们可以看到一些关于字体、轴等的信息，但我们还可以获得一个配置，它实际上可以帮助我们使图表更具描述性，这就是Interval。我们可以更改此值以获得或多或少详细的图表，如果我们设置50毫秒的间隔，那么我们的图表中就会出现更多垂直线。我们应该记住“Apply Interval”，然后“Display Graph”：

![](/assets/images/2025/load/jmeteranalyzeresults09.png)

在此图中，我们可以看到50毫秒间隔内的一些平均响应时间，它包含更多信息，但结果并不那么直观。如果我们将其更改为一秒，那么我们就能获得此执行的响应时间的非常好的可视化效果：

![](/assets/images/2025/load/jmeteranalyzeresults10.png)

### 5.4 图形结果监听器

最后一个可能派上用场的JMeter结果监听器是Graph Results监听器。**Graph Results监听器收集所有数据，汇总它们，并以图形形式呈现它们，并随时间变化显示值**。这与Aggregate Report非常相似，因为元素相同：平均值、中位数、偏差和吞吐量。这里的区别在于，我们可以看到所有这些随时间变化的情况：

![](/assets/images/2025/load/jmeteranalyzeresults11.png)

## 6. 使用JMeter仪表板报告GUI分析JMeter结果

正如我们目前所见，JMeter结果图并不是可视化我们想要的指标的最佳方式。**这就是为什么JMeter提供了一个GUI，其中包含在测试执行期间创建的报告的仪表板**。但这些仪表板不是默认创建的，我们需要遵循以下流程来创建它们：

1. 在测试计划中添加一个Simple Data Writer监听器，将结果写入文件：
   ![](/assets/images/2025/load/jmeteranalyzeresults12.png)
2. 创建一个属性文件，用于配置timestamp_format、threads等内容
3. 使用工具中的“Generate HTML Report”创建包含HTML报告的输出文件夹：
   ![](/assets/images/2025/load/jmeteranalyzeresults13.png)
4. 在弹出窗口中设置JTL和属性文件以及用于输出的文件夹：
   ![](/assets/images/2025/load/jmeteranalyzeresults14.png)
5. 在浏览器中打开输出文件夹中的index.html文件，以访问报告

然后，我们可以查看我们创建的HTML页面，从摘要页面index.html开始：

![](/assets/images/2025/load/jmeteranalyzeresults15.png)

此摘要页面包含一个请求摘要饼图和两个表格，其中一个表格包含Application Performance Index，另一个表格包含其他统计信息。

从左侧菜单，我们可以导航到更多图表：

![](/assets/images/2025/load/jmeteranalyzeresults16.png)

在此列表中，我们可以从“Over Time”、“Throughput”或“Response Times”图表中进行选择，并显示“Active Threads Over Time”、“Time Vs Threads”等图表。

## 7. 总结

在本文中，我们介绍了分析Apache JMeter结果的选项。我们研究了自托管工具选项，并列出了一些更受欢迎的插件。最后，我们深入了解了JMeter增强选项，演示了JMeter的不同报告和JMeter提供的仪表板报告GUI的用法。