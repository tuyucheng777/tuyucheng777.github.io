---
layout: post
title:  通过Java程序创建并运行Apache JMeter测试脚本
category: load
copyright: load
excerpt: JMeter
---

## 1. 简介

[Apache JMeter](https://jmeter.apache.org/)是一款基于Java的开源应用程序，旨在分析和衡量Web应用程序的性能。它允许测试人员模拟服务器、网络或对象上的重负载，以分析不同负载下的整体性能。JMeter提供了一个易于使用的GUI，用于定义、执行和查看各种负载测试的报告。

**尽管JMeter提供了用于创建和执行测试脚本的用户友好的GUI，但在某些情况下利用Java编程实现自动化可能会很有用，特别是在持续集成和部署管道中**。

在本教程中，我们将探讨如何使用Java以编程方式创建和执行Apache Meter测试脚本，并附带一个实际示例来说明所涉及的步骤。

## 2. 设置环境

在开始编码之前，让我们确保已经设置了所需的环境。要安装JMeter，我们可以从[JMeter网站](https://jmeter.apache.org/download_jmeter.cgi)下载它。JMeter与[Java 8](https://www.baeldung.com/java-8-new-features)或更高版本兼容。或者，在MacOS上，我们可以使用以下命令使用Homebrew安装JMeter：

```bash
brew install jmeter
```

我们还需要配置Java项目以包含JMeter属性，对于Maven项目，请将以下依赖项添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.jmeter</groupId>
    <artifactId>ApacheJMeter_core</artifactId>
    <version>5.6.3</version>
</dependency>
<dependency>
    <groupId>org.apache.jmeter</groupId>
    <artifactId>ApacheJMeter_http</artifactId>
    <version>5.6.3</version>
</dependency>
```

这些依赖项包括[Apache JMeter的核心功能](https://mvnrepository.com/artifact/org.apache.jmeter/ApacheJMeter_core)和[HTTP组件](https://mvnrepository.com/artifact/org.apache.jmeter/ApacheJMeter_http)，它们为创建和运行JMeter测试计划、发送[HTTP请求](https://www.baeldung.com/java-http-request)以及在JMeter测试计划中处理HTTP响应提供了必要的类和实用程序。

## 3. 创建测试脚本和环境变量文件

我们将生成一个简单的测试脚本来模拟对指定URL或应用程序的HTTP GET请求，此脚本的主要目的是对指定的目标应用程序(在本例中为[https://www.google.com](https://www.google.com))执行负载测试。

它将通过分派多个并发请求来模拟所提供URL上的真实负载，使我们能够评估应用程序在不同负载级别下的性能。在下面的脚本中，我们首先检查JMETER_HOME环境变量以找到JMeter安装目录。

验证完成后，我们初始化JMeter引擎StandardJMeterEngine并创建测试计划TestPlan。下面是实现此目的的Java代码：

```java
@Test
void givenJMeterScript_whenUsingCode_thenExecuteViaJavaProgram() throws IOException {
    String jmeterHome = System.getenv("JMETER_HOME");
    if (jmeterHome == null) {
        throw new RuntimeException("JMETER_HOME environment variable is not set.");
    }

    String file = Objects.requireNonNull(JMeterLiveTest.class.getClassLoader().getResource("jmeter.properties")).getFile();
    JMeterUtils.setJMeterHome(jmeterHome);
    JMeterUtils.loadJMeterProperties(file);
    JMeterUtils.initLocale();

    StandardJMeterEngine jmeter = new StandardJMeterEngine();

    HTTPSamplerProxy httpSampler = getHttpSamplerProxy();

    LoopController loopController = getLoopController();

    ThreadGroup threadGroup = getThreadGroup(loopController);

    TestPlan testPlan = getTestPlan(threadGroup);

    HashTree testPlanTree = new HashTree();
    HashTree threadGroupHashTree = testPlanTree.add(testPlan, threadGroup);
    threadGroupHashTree.add(httpSampler);

    SaveService.saveTree(testPlanTree, Files.newOutputStream(Paths.get("script.jmx")));
    Summariser summer = null;
    String summariserName = JMeterUtils.getPropDefault("summariser.name", "summary");
    if (summariserName.length() > 0) {
        summer = new Summariser(summariserName);
    }

    String logFile = "output-logs.jtl";
    ResultCollector logger = new ResultCollector(summer);
    logger.setFilename(logFile);
    testPlanTree.add(testPlanTree.getArray()[0], logger);

    jmeter.configure(testPlanTree);
    jmeter.run();

    System.out.println("Test completed. See output-logs.jtl file for results");
    System.out.println("JMeter .jmx script is available at script.jmx");
}
```

在下面的getLoopController()方法中，循环控制器决定了迭代或循环的次数。在这种情况下，我们将循环控制器配置为仅执行一次测试计划：

```java
private static LoopController getLoopController() {
    LoopController loopController = new LoopController();
    loopController.setLoops(1);
    loopController.setFirst(true);
    loopController.setProperty(TestElement.TEST_CLASS, LoopController.class.getName());
    loopController.setProperty(TestElement.GUI_CLASS, LoopControlPanel.class.getName());
    loopController.initialize();
    return loopController;
}
```

方法getThreadGroup()定义一个名为“Sample Thread Group”的[线程组](https://www.baeldung.com/jmeter-run-multiple-thread-groups)，指定用于模拟加速期的虚拟用户/线程数。该组中的每个线程代表一个向目标URL发出请求的虚拟用户，“Sample Thread Group”包含10个线程(虚拟用户)，并以5秒为间隔加速：

```java
private static ThreadGroup getThreadGroup(LoopController loopController) {
    ThreadGroup threadGroup = new ThreadGroup();
    threadGroup.setName("Sample Thread Group");
    threadGroup.setNumThreads(10);
    threadGroup.setRampUp(5);
    threadGroup.setSamplerController(loopController);
    threadGroup.setProperty(TestElement.TEST_CLASS, ThreadGroup.class.getName());
    threadGroup.setProperty(TestElement.GUI_CLASS, ThreadGroupGui.class.getName());
    return threadGroup;
}
```

随后，在getHttpSamplerProxy()方法中，我们创建一个HTTP采样器HTTPSamplerProxy，用于将HTTP GET请求分派到目标URL(https://www.google.com)。我们使用域、路径和请求方法配置采样器：

```java
private static HTTPSamplerProxy getHttpSamplerProxy() {
    HTTPSamplerProxy httpSampler = new HTTPSamplerProxy();
    httpSampler.setDomain("www.google.com");
    httpSampler.setPort(80);
    httpSampler.setPath("/");
    httpSampler.setMethod("GET");
    httpSampler.setProperty(TestElement.TEST_CLASS, HTTPSamplerProxy.class.getName());
    httpSampler.setProperty(TestElement.GUI_CLASS, HttpTestSampleGui.class.getName());
    return httpSampler;
}
```

我们通过将线程组和HTTP采样器添加到HashTree来构建测试计划：

```java
private static TestPlan getTestPlan(ThreadGroup threadGroup) {
    TestPlan testPlan = new TestPlan("Sample Test Plan");
    testPlan.setProperty(TestElement.TEST_CLASS, TestPlan.class.getName());
    testPlan.setProperty(TestElement.GUI_CLASS, TestPlanGui.class.getName());
    testPlan.addThreadGroup(threadGroup);
    return testPlan;
}
```

最后，我们使用测试计划树配置JMeter引擎，并调用运行方法执行测试。

**此脚本概述了一个负载配置文件，该配置文件模拟了10个并发用户在5秒内逐渐增加的情况，每个用户都会向目标URL发起一个HTTP GET请求。由于无需为多次迭代进行额外配置，因此负载保持一致**。

jmeter.properties文件是Apache JMeter使用的配置文件，用于定义测试执行的各种设置和参数。在此文件中，以下属性指定完成测试执行后保存测试结果的格式：

```properties
jmeter.save.saveservice.output_format=xml
```

## 4. 理解输出文件

我们将测试计划保存为script.jmx，采用.jmx文件格式，该格式与JMeter GUI中的加载和执行兼容。此外，我们配置了一个Summarizer来收集和汇总测试结果。此外，还建立了一个结果收集器，将结果存储在名为output-logs.jtl的.jtl文件中。

### 4.1 理解.jtl文件

output-logs.jtl文件的内容如下：

```text
<?xml version="1.0" encoding="UTF-8"?>
<testResults version="1.2">
<httpSample t="354" it="0" lt="340" ct="37" ts="1711874302012" s="true" lb="" rc="200" rm="OK"  tn="Sample Thread Group 1-1" dt="text" by="22388" sby="111" ng="1" na="1">
    <java.net.URL>http://www.google.com/</java.net.URL>
</httpSample>
<httpSample t="351" it="0" lt="317" ct="21" ts="1711874302466" s="true" lb="" rc="200" rm="OK"  tn="Sample Thread Group 1-2" dt="text" by="22343" sby="111" ng="1" na="1">
    <java.net.URL>http://www.google.com/</java.net.URL>
</httpSample>
<httpSample t="410" it="0" lt="366" ct="14" ts="1711874303024" s="true" lb="" rc="200" rm="OK"  tn="Sample Thread Group 1-3" dt="text" by="22398" sby="111" ng="1" na="1">
    <java.net.URL>http://www.google.com/</java.net.URL>
</httpSample>
<httpSample t="363" it="0" lt="344" ct="19" ts="1711874303483" s="true" lb="" rc="200" rm="OK"  tn="Sample Thread Group 1-4" dt="text" by="22367" sby="111" ng="1" na="1">
    <java.net.URL>http://www.google.com/</java.net.URL>
</httpSample>
</testResults>
```

输出包含<httpSample\>元素，该元素表示测试期间发出的HTTP请求，并包含与该请求相关的各种属性和元数据。让我们分解一下这些属性：

- t：样本执行所需的总时间(以毫秒为单位)
- it：空闲时间，即等待响应的时间
- lt：延迟，即请求到达服务器并返回所需的时间，不包括空闲时间
- ct：连接时间，即建立与服务器的连接所需的时间
- ts：样本执行的时间戳，以纪元以来的毫秒数表示
- s：表示样本是否成功(true)或失败(false)
- lb：与样本相关的标签，通常是采样器名称
- rc：服务器返回的HTTP响应码
- rm：与响应码关联的响应消息
- tn：执行样本的线程的名称
- dt：样本返回的数据类型(例如文本、二进制)
- by：响应体中接收的字节数
- sby：请求中发送的字节数
- ng/na：线程组中的活动线程数/所有线程组中的活动线程数

这些数据对于分析测试系统性能、识别瓶颈以及优化应用程序的性能至关重要。

### 4.2 理解.jmx文件

.jmx文件表示XML格式的JMeter测试计划配置，script.jmx文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0" jmeter="5.4.1">
    <org.apache.jorphan.collections.HashTree>
        <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="Sample Test Plan"/>
        <org.apache.jorphan.collections.HashTree>
            <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Sample Thread Group">
                <intProp name="ThreadGroup.num_threads">10</intProp>
                <intProp name="ThreadGroup.ramp_time">5</intProp>
                <elementProp name="ThreadGroup.main_controller" elementType="LoopController"                  guiclass="LoopControlPanel" testclass="LoopController">
                    <boolProp name="LoopController.continue_forever">false</boolProp>
                    <intProp name="LoopController.loops">1</intProp>
                </elementProp>
            </ThreadGroup>
            <org.apache.jorphan.collections.HashTree>
                <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy">
                    <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
                        <collectionProp name="Arguments.arguments"/>
                    </elementProp>
                    <stringProp name="HTTPSampler.domain">www.google.com</stringProp>
                    <intProp name="HTTPSampler.port">80</intProp>
                    <stringProp name="HTTPSampler.path">/</stringProp>
                    <stringProp name="HTTPSampler.method">GET</stringProp>
                </HTTPSamplerProxy>
                <org.apache.jorphan.collections.HashTree/>
            </org.apache.jorphan.collections.HashTree>
        </org.apache.jorphan.collections.HashTree>
    </org.apache.jorphan.collections.HashTree>
</jmeterTestPlan>
```

测试计划在<jmeterTestPlan\>元素中定义，在测试计划中，<ThreadGroup\>元素描述了将执行测试场景的虚拟用户或[线程](https://www.baeldung.com/java-thread-parameters)组，这包括线程数和启动时间等详细信息。

线程组由控制器协调，以<LoopController\>元素为例，它管理执行流程，确定循环次数以及执行是否应无限期地继续。

**要发送的实际HTTP请求由<HTTPSampleProxy\>元素表示，每个元素都详细说明了域、端口、路径和HTTP方法等方面。这些元素共同构成了在JMeter框架内编排性能测试的综合蓝图**，它可以模拟用户与Web服务器的交互并随后分析服务器响应。

## 5. 总结

在本文中，我们演示了如何使用JMeter Java API以编程方式创建和执行Apache JMeter测试脚本，使用这种方法，开发人员可以自动化性能测试并将其无缝集成到他们的开发工作流程中。

它可以高效地测试Web应用程序，帮助在开发周期早期识别和解决性能瓶颈。