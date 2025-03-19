---
layout: post
title:  JMeter Beanshell使用指南
category: load
copyright: load
excerpt: JMeter
---

## 1. 简介

在本快速教程中，我们将创建一个使用[JMeter](https://www.baeldung.com/jmeter)中提供的大多数[BeanShell](https://beanshell.github.io/)功能的测试用例。最后，我们将学习如何使用必要的工具通过BeanShell脚本进行任何测试。

## 2. 设置JMeter和BeanShell

首先，我们需要下载JMeter。要运行它，我们需要将[下载](https://jmeter.apache.org/download_jmeter.cgi)的文件解压到任意位置，然后运行可执行文件(对于基于Unix的系统，运行jmeter.sh；对于Windows，运行jmeter.bat)。

在撰写本文时，最新的稳定版本是5.6.3，它已经捆绑了BeanShell 2.0b6二进制文件。**版本3目前正在[开发中](https://github.com/beanshell/beanshell?tab=readme-ov-file#development-road-map)，将支持较新的Java功能。在此之前，我们只能使用Java 4语法，尽管JMeter可以在最新的JVM上运行**。我们稍后会看到这会如何影响我们的脚本。

最重要的是，虽然BeanShell是一种功能丰富的脚本语言，但它主要用于在JMeter中实现测试步骤。我们将在我们的场景中实际看到这一点：向API发出POST请求，同时捕获统计数据，例如发送的总字节数和耗时。

**启动JMeter后，我们需要的唯一非BeanShell元素是[线程组](https://www.baeldung.com/jmeter#jmeter-script)**。我们通过右键单击“Test Plan”，然后选择“Add”，然后选择“Threads (Users)”，然后选择“Thread Group”来创建一个线程组。我们只需要指定所需的线程数和循环数：

![](/assets/images/2025/load/jmeterbeanshell01.png)

完成后，我们就可以为测试创建基于BeanShell的元素了。

## 3. 预处理器

我们将从[预处理器](https://jmeter.apache.org/usermanual/component_reference.html#preprocessors)开始，为我们的POST请求配置值。我们通过右键单击“Thread Group”，然后单击“Add”，然后单击“Pre Processors”，然后单击“BeanShell PreProcessor”来执行此操作。**在其内容中，让我们编写一个脚本来为请求生成随机值**：

```java
random = new Random();

key = "k"+random.nextInt();
value = random.nextInt();
```

**要声明变量，我们不需要明确包含它们的类型，我们可以使用JVM或JMeter的lib文件夹中可用的任何类型**。

让我们使用vars对象(脚本之间共享的特殊变量)来保存这些值以供以后使用，以及我们的API地址：

```java
vars.put("base-api", "http://localhost:8080/api");

vars.put("key", key);
vars.putObject("value", value);
```

我们为value使用putObject()，因为put()仅接收字符串值。我们的最终变量定义当前线程应打印摘要之前需要进行多少次测试迭代，我们稍后会用到它：

```java
vars.putObject("summary-iterations", 5)
```

## 4. 采样器

我们的[采样器](https://jmeter.apache.org/usermanual/component_reference.html#samplers)检索先前存储的值，并使用与JMeter捆绑在一起的[Apache HTTP框架](https://www.baeldung.com/httpclient-post-http-request)将它们发送到API。**最重要的是，我们需要为脚本中直接提到的任何不在[默认导入](http://www.beanshell.org/manual/bshmanual.html)列表中的类添加import语句**。

为了创建请求体，我们将使用[String.format()](https://www.baeldung.com/string/format)的较旧的new Object[]{...}语法，因为[可变参数](https://www.baeldung.com/java-varargs)是Java 5的功能：

```java
url = vars.get("base-api");
json = String.format(
    "{\"key\": \"%s\", \"value\": %s}", 
    new Object[]{ vars.get("key"), vars.get("value") }
);
```

现在，让我们执行请求：

```java
client = HttpClients.createDefault();
body = new StringEntity(json, ContentType.APPLICATION_JSON);

post = new HttpPost(url);
post.setEntity(body);

response = client.execute(post);
```

JMeter包含ResponseCode和ResponseMessage变量，我们将在后续阶段检索它们：

```java
ResponseCode = response.getStatusLine().getStatusCode();
ResponseMessage = EntityUtils.toString(response.getEntity());
```

**最后，由于我们不能使用try-with-resources块，我们关闭资源并为脚本选择一个返回值**：

```java
response.close();
client.close();

return json;
```

返回值可以是任意类型。稍后我们将返回请求主体，以便计算请求大小。

## 5. 后处理器

[后处理器](https://jmeter.apache.org/usermanual/component_reference.html#postprocessors)在采样器之后立即执行，我们将使用它来收集和汇总所需的信息。让我们创建一个函数来增加变量并保存结果：

```java
incrementVar(name, increment) {
    value = (Long) vars.getObject(name);
    if (value == null)
        value = 0l;

    value += increment;
    vars.putObject(name, value);
    log.info("{}: {}", name, value);
}
```

这里，我们还有一个记录器，无需额外配置，它将记录到JMeter控制台。**请注意，可见性修饰符、返回类型和参数类型不是必需的；它们是从上下文推断出来的**。

我们将增加每次线程迭代的耗时和发送/接收的字节数，prev变量允许我们从上一个脚本中获取该信息：

```java
incrementVar("elapsed-time-total", prev.getTime());
incrementVar("bytes-received-total", prev.getResponseMessage().getBytes().length);
incrementVar("bytes-sent-total", prev.getBytesAsLong());
```

## 6. 监听器

[监听器](https://jmeter.apache.org/usermanual/component_reference.html#listeners)在后处理器之后运行，我们将使用一个监听器将报告写入文件系统。首先，让我们编写一个辅助函数并设置一些变量：

```java
println(writer, message, arg1) {
    writer.println(String.format(message, new Object[] {arg1}));
}

thread = ctx.getThread();
threadGroup = ctx.getThreadGroup();

request = prev.getResponseDataAsString();
response = prev.getResponseMessage();
```

**ctx变量提供来自线程组和当前线程的信息**。接下来，让我们构建一个[FileWriter](https://www.baeldung.com/java-append-to-file)来在主目录中写入报告：

```java
fw = new FileWriter(new File(System.getProperty("user.home"), "jmeter-report.txt"), true);
writer = new PrintWriter(new BufferedWriter(fw));

println(writer, "* iteration: %s", vars.getIteration());
println(writer, "* elapsed time: %s ms.", prev.getTime());
println(writer, "* request: %s", request);
println(writer, "= %s bytes", prev.getBytesAsLong());
println(writer, "* response body: %s", response);
println(writer, "= %s bytes", response.getBytes().length);
```

由于此操作针对每个线程迭代运行，因此我们调用vars.getIteration()来跟踪迭代次数。最后，如果当前迭代是summary-iterations的倍数，我们将打印摘要：

```java
if (vars.getIteration() % vars.getObject("summary-iterations") == 0) {
    println(writer, "## summary for %s", thread.getThreadName());
    println(writer, "* total bytes sent: %s bytes", vars.get("bytes-sent-total"));
    println(writer, "* total bytes received: %s bytes", vars.get("bytes-received-total"));
    println(writer, "* total elapsed time: %s ms.", vars.get("elapsed-time-total"));
}
```

最后，让我们关闭writer：

```java
writer.close();
```

## 7. 总结

在本文中，我们探讨了如何有效地使用JMeter中的BeanShell向测试计划添加自定义脚本。我们介绍了预处理器、采样器、后处理器和监听器等重要组件，展示了如何操作请求数据、处理响应和记录指标。