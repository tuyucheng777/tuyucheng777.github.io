---
layout: post
title:  Gatling、JMeter和The Grinder：负载测试工具比较
category: load
copyright: load
excerpt: JMeter、Gatling、The Grinder
---

## 1. 简介

选择合适的工具来完成这项工作可能很困难。在本教程中，我们将通过比较三种Web应用程序负载测试工具(Apache JMeter、Gatling和The Grinder)与简单的REST API来简化这一过程。

## 2. 负载测试工具

首先，让我们快速回顾一下每个背景。

### 2.1 Gatling

[Gatling](https://gatling.io/)是一款负载测试工具，可在Scala中创建测试脚本。**Gatling的记录器会生成Scala测试脚本，这是Gatling的一项重要功能**。查看我们的[Gatling简介](https://www.baeldung.com/introduction-to-gatling)教程了解更多信息。

### 2.2 JMeter

[JMeter](https://jmeter.apache.org/)是Apache的一款负载测试工具，它提供了一个很好的GUI，我们可以使用它进行配置。**一个称为逻辑控制器的独特功能为在GUI中设置测试提供了极大的灵活性**。

访问我们的[JMeter简介教程](https://www.baeldung.com/jmeter)来获取屏幕截图和更多解释。

### 2.3 The Grinder

我们的最后一款工具[The Grinder](http://grinder.sourceforge.net/)提供了比其他两款更基于编程的脚本引擎，并使用Jython。不过，The Grinder 3也具有录制脚本的功能。

Grinder 与其他两款工具的不同之处还在于它允许使用控制台和代理进程，此功能提供了代理进程的能力，因此负载测试可以扩展到多个服务器。**它被特别宣传为一款专为开发人员构建的负载测试工具，用于查找死锁和速度减慢**。

## 3. 测试用例设置

接下来，为了进行测试，我们需要一个API。我们的API功能包括：

- 添加/更新奖励记录
- 查看单次/全部奖励记录
- 将交易链接到客户奖励记录
- 查看客户奖励记录的交易

**我们的场景**：

一家商店正在全国范围内进行促销，新老客户都需要客户奖励账户才能获得优惠。奖励API根据客户ID检查客户奖励账户，如果不存在奖励账户，则添加该账户，然后链接到交易。

此后，我们查询交易。

### 3.1 我们的REST API

让我们通过查看一些方法存根来快速了解一下API：

```java
@PostMapping(path="/rewards/add")
public @ResponseBody RewardsAccount addRewardsAcount(@RequestBody RewardsAccount body)

@GetMapping(path="/rewards/find/{customerId}")
public @ResponseBody Optional<RewardsAccount> findCustomer(@PathVariable Integer customerId)

@PostMapping(path="/transactions/add")
public @ResponseBody Transaction addTransaction(@RequestBody Transaction transaction)

@GetMapping(path="/transactions/findAll/{rewardId}")
public @ResponseBody Iterable<Transaction> findTransactions(@PathVariable Integer rewardId)
```

请注意一些关系，例如通过奖励ID查询交易和通过客户ID获取奖励账户，这些关系强制一些逻辑和一些响应解析以创建我们的测试场景。

被测应用程序还使用H2内存数据库进行持久保存。

**幸运的是，我们的工具都可以很好地处理这个问题，有些甚至比其他的更好**。

### 3.2 我们的测试计划

接下来，我们需要测试脚本。

为了进行公平的比较，我们将对每个工具执行相同的自动化步骤：

1. 生成随机客户账户ID
2. 发布交易
3. 解析响应中的随机客户ID和交易ID
4. 使用客户ID查询客户奖励账户ID
5. 解析响应中的奖励账户ID
6. 如果不存在奖励账户ID，则通过POST请求添加一个
7. 使用交易ID发布具有更新奖励ID的相同初始交易
8. 根据奖励账户ID查询所有交易

让我们仔细看看每个工具的第4步，并且，一定要查看[所有三个已完成脚本的示例](https://github.com/eugenp/tutorials/tree/master/testing-modules/load-testing-comparison/src/main/resources/scripts)。

### 3.3 Gatling

**对于Gatling来说，熟悉Scala对开发人员来说是一个福音，因为Gatling API非常强大并且包含很多功能**。

Gatling的API采用了构建器DSL方法，正如我们在其第4步中看到的那样：

```scala
.exec(http("get_reward")
    .get("/rewards/find/${custId}")
    .check(jsonPath("$.id").saveAs("rwdId")))
```

特别值得注意的是，当我们需要读取和验证HTTP响应时，Gatling支持JSON Path。在这里，我们将获取奖励ID并将其保存到Gatling的内部状态中。

此外，Gatling的表达式语言使得动态请求体字符串更加容易：

```scala
.body(StringBody(
    """{ 
        "customerRewardsId":"${rwdId}",
        "customerId":"${custId}",
        "transactionDate":"${txtDate}" 
    }""")).asJson)
```

最后是本次比较的配置。1000次运行设置为整个场景的重复，atOnceUsers方法设置线程/用户：

```scala
val scn = scenario("RewardsScenario")
    .repeat(1000) {
    ...
    }
    setUp(
        scn.inject(atOnceUsers(100))
    ).protocols(httpProtocol)
```

**[整个Scala脚本](https://github.com/tuyucheng777/taketoday-tutorial4j/tree/master/software.test/load-testing-comparison/src/main/resources/scripts)可以在我们的Github仓库中查看**。

### 3.4 JMeter

**JMeter在GUI配置后生成一个XML文件**，该文件包含具有设置属性及其值的JMeter特定对象，例如：

```xml
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Add Transaction" enabled="true">
<JSONPostProcessor guiclass="JSONPostProcessorGui" testclass="JSONPostProcessor" testname="Transaction Id Extractor" enabled="true">
```

查看testname属性，我们可以标记它们，因为我们识别它们与上面的逻辑步骤相匹配。添加子项、变量和依赖项步骤的能力为JMeter提供了脚本提供的灵活性。此外，我们甚至还为变量设置了作用域！

我们在JMeter中对运行和用户的配置使用ThreadGroups：

```xml
<stringProp name="ThreadGroup.num_threads">100</stringProp>
```

[查看整个jmx文件作为参考](https://github.com/tuyucheng777/taketoday-tutorial4j/tree/master/software.test/load-testing-comparison/src/main/resources/scripts)。虽然可能，但使用功能齐全的GUI将测试以XML格式编写为.jmx文件是没有意义的。

### 3.5 The Grinder

如果没有Scala和GUI的函数式编程，我们为The Grinder编写的Jython脚本看起来非常简单。添加一些系统Java类，我们的代码行数就会少很多。

```python
customerId = str(random.nextInt());
result = request1.POST("http://localhost:8080/transactions/add",
    "{"'"customerRewardsId"'":null,"'"customerId"'":"+ customerId + ","'"transactionDate"'":null}")
txnId = parseJsonString(result.getText(), "id")
```

然而，测试设置代码行数的减少与需要更多字符串维护代码(例如解析JSON字符串)相抵消。此外，[HTTPRequest](http://grinder.sourceforge.net/g3/script-javadoc/net/grinder/plugin/http/HTTPRequest.html) API的功能也很少。

使用Grinder，我们在外部属性文件中定义线程、进程和运行值：

```properties
grinder.threads = 100
grinder.processes = 1
grinder.runs = 1000
```

**我们的The Grinder的完整Jython脚本将[如此所示](https://github.com/tuyucheng777/taketoday-tutorial4j/tree/master/software.test/load-testing-comparison/src/main/resources/scripts)**。

## 4. 测试运行

### 4.1 测试执行

**这三种工具都建议使用命令行进行大负载测试**。

为了运行测试，我们将使用[Gatling开源版本3.4.0](https://gatling.io/open-source/)作为独立工具、[JMeter 5.3](https://jmeter.apache.org/download_jmeter.cgi)和[The Grinder版本3](https://sourceforge.net/projects/grinder/)。

Gatling仅要求我们设置JAVA_HOME和GATLING_HOME，要执行Gatling，我们使用：

```bash
./gatling.sh
```

在GATLING_HOME/bin目录中。

JMeter需要一个参数来禁用测试的GUI，如启动配置GUI时提示的那样：

```bash
./jmeter.sh -n -t TestPlan.jmx -l log.jtl
```

与Gatling一样，Grinder要求我们设置JAVA_HOME和GRINDERPATH。但是，它还需要几个属性：

```bash
export CLASSPATH=/home/lore/Documents/grinder-3/lib/grinder.jar:$CLASSPATH
export GRINDERPROPERTIES=/home/lore/Documents/grinder-3/examples/grinder.properties
```

如上所述，我们提供了一个grinder.properties文件，用于进行其他配置，例如线程、运行、进程和控制台主机。

最后，我们使用以下命令启动控制台和代理：

```bash
java -classpath $CLASSPATH net.grinder.Console
```

```shell
java -classpath $CLASSPATH net.grinder.Grinder $GRINDERPROPERTIES
```

### 4.2 测试结果

每个测试运行了1000次，有100个用户/线程，让我们来看看其中的一些亮点：

|        |   成功的请求   | 错误 | 总测试时间(秒)| 平均响应时间(毫秒) |  平均吞吐量   |
| :----: |:---------:| :--: | :--------------: |:----------:|:--------:|
| Gatling | 500000个请求 |  0   |      218秒       |     42     | 2283请求/秒 |
|JMeter| 499997个请求 |  0   |      237秒       |     46     | 2101请求/秒 |
| The Grinder | 499997个请求 |  0   |      221秒       |     43     | 2280请求/秒 |

**结果显示，这3种工具的速度相似，但根据平均吞吐量，Gatling略胜于其他2种工具**。 

每个工具还在更友好的用户界面中提供附加信息。

**Gatling将在运行结束时生成一份HTML报告，其中包含整个运行以及每个请求的多个图表和统计数据**。以下是测试结果报告的片段：

![](/assets/images/2025/load/gatlingjmetergrindercomparison01.png)

**使用JMeter时，我们可以在测试运行后打开GUI，并根据保存结果的日志文件生成HTML报告**：

![](/assets/images/2025/load/gatlingjmetergrindercomparison02.png)

JMeter HTML报告还包含每个请求的统计信息细目分类。

**最后，The Grinder控制台记录每个代理和运行的统计数据**：

![](/assets/images/2025/load/gatlingjmetergrindercomparison03.png)

The Grinder的速度虽然很快，但是却需要额外的开发时间并且输出数据的多样性较低。

## 5. 比较

现在是时候全面了解每个负载测试工具了。

|          | Gatling  |JMeter| The Grinder |
|:--------:| :-----: | :-----: |:-----------:|
|  项目与社区   |    9    |    9    |      6      |
|    性能    |    9    |    8    |      9      |
| 脚本能力/API |    7    |    9    |      8      |
|   用户界面   |    9    |    8    |      6      |
|    报告    |    9    |    7    |      6      |
|   一体化    |    7    |    9    |      7      |
|    概括    | **8.3** | **8.3** |      **7**      |

**Gatling**：

- 可靠、精致的负载测试工具，可使用Scala脚本输出漂亮的报告
- 产品的开源和企业支持级别

**JMeter**：

- 强大的API(通过GUI)用于测试脚本开发，无需编码
- Apache基金会支持以及与Maven的良好集成

**The Grinder**：

- 为使用Jython的开发人员提供的快速性能负载测试工具
- 跨服务器可扩展性为大型测试提供了更多潜力

简而言之，如果需要速度和可扩展性，那么就使用The Grinder。

如果美观的交互式图表有助于展现性能提升并支持变革，那么请使用Gatling。

JMeter是用于复杂业务逻辑或具有多种消息类型的集成层的工具，作为Apache软件基金会的一部分，JMeter提供了成熟的产品和庞大的社区。

## 6. 总结

综上所述，我们发现这些工具在某些领域功能相似，而在其他领域则大放异彩。适合工作的正确工具是软件开发中行之有效的口语智慧。