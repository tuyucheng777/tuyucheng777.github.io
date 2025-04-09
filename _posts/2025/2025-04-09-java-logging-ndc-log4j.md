---
layout: post
title:  具有嵌套诊断上下文(NDC)的Java日志记录
category: log
copyright: log
excerpt: Log4j
---

## 1. 概述

嵌套诊断上下文(NDC)是一种帮助区分来自不同来源的交叉日志消息的机制，NDC通过提供向每个日志条目添加独特上下文信息的功能来实现这一点。

在本文中，我们将探讨NDC的使用及其在各种Java日志框架中的使用/支持。

## 2. 诊断上下文

在典型的多线程应用程序(如Web应用程序或REST API)中，每个客户端请求都由不同的线程处理，此类应用程序生成的日志将混合所有客户端请求和源，这使得很难从日志中找出任何业务意义或进行调试。

嵌套诊断上下文(NDC)管理每个线程的上下文信息堆栈，NDC中的数据可用于代码中的每个日志请求，并且可以配置为与每条日志消息一起记录-即使在数据不在范围内的地方也是如此。每条日志消息中的上下文信息有助于根据其来源和上下文区分日志。

映射[诊断上下文(MDC)](https://www.baeldung.com/mdc-in-log4j-2-logback)也以映射的形式管理每个线程的信息。

## 3. 示例应用程序中的NDC堆栈

为了演示NDC堆栈的用法，我们以向投资账户汇款的REST API为例。

所需输入的信息在Investment类中表示：

```java
public class Investment {
    private String transactionId;
    private String owner;
    private Long amount;

    public Investment (String transactionId, String owner, Long amount) {
        this.transactionId = transactionId;
        this.owner = owner;
        this.amount = amount;
    }

    // standard getters and setters
}
```

使用InvestmentService执行向投资账户的转账，这些类的完整源代码可以在[Github项目](https://github.com/eugenp/tutorials/tree/master/logging-modules/log-mdc)中找到。

在示例应用程序中，数据transactionId和owner被放置在处理给定请求的线程中的NDC堆栈中，该数据在该线程中的每条日志消息中可用。这样，可以跟踪每个唯一的事务，并且可以识别每条日志消息的相关上下文。

## 4. Log4j中的NDC

Log4j提供了一个名为NDC的类，它提供了静态方法来管理NDC堆栈中的数据；基本用法：

- 进入上下文时，使用NDC.push()在当前线程中添加上下文数据
- 离开上下文时，使用NDC.pop()取出上下文数据
- 退出线程时，调用NDC.remove()删除线程的诊断上下文并确保释放内存(从Log4j 1.3开始，不再需要)

在示例应用程序中，让我们使用NDC在代码中的相关位置添加/删除上下文数据：

```java
import org.apache.log4j.NDC;

@RestController
public class Log4JController {
    @Autowired
    @Qualifier("Log4JInvestmentService")
    private InvestmentService log4jBusinessService;

    @RequestMapping(
            value = "/ndc/log4j",
            method = RequestMethod.POST)
    public ResponseEntity<Investment> postPayment(
            @RequestBody Investment investment) {

        NDC.push("tx.id=" + investment.getTransactionId());
        NDC.push("tx.owner=" + investment.getOwner());

        log4jBusinessService.transfer(investment.getAmount());

        NDC.pop();
        NDC.pop();

        NDC.remove();

        return new ResponseEntity<Investment>(investment, HttpStatus.OK);
    }
}
```

可以通过使用log4j.properties中附加程序使用的ConversionPattern中的%x选项将NDC的内容显示在日志消息中：

```text
log4j.appender.consoleAppender.layout.ConversionPattern 
  = %-4r [%t] %5p %c{1} - %m - [%x]%n
```

让我们将REST API部署到Tomcat，示例请求：

```shell
POST /logging-service/ndc/log4j
{
  "transactionId": "4",
  "owner": "Marc",
  "amount": 2000
}
```

我们可以在日志输出中看到诊断上下文信息：

```text
48569 [http-nio-8080-exec-3]  INFO Log4JInvestmentService 
  - Preparing to transfer 2000$. 
  - [tx.id=4 tx.owner=Marc]
49231 [http-nio-8080-exec-4]  INFO Log4JInvestmentService 
  - Preparing to transfer 1500$. 
  - [tx.id=6 tx.owner=Samantha]
49334 [http-nio-8080-exec-3]  INFO Log4JInvestmentService 
  - Has transfer of 2000$ completed successfully ? true. 
  - [tx.id=4 tx.owner=Marc] 
50023 [http-nio-8080-exec-4]  INFO Log4JInvestmentService 
  - Has transfer of 1500$ completed successfully ? true. 
  - [tx.id=6 tx.owner=Samantha]
...
```

## 5. Log4j 2中的NDC

Log4j 2中的NDC被称为线程上下文堆栈：

```java
import org.apache.logging.log4j.ThreadContext;

@RestController
public class Log4J2Controller {
    @Autowired
    @Qualifier("Log4J2InvestmentService")
    private InvestmentService log4j2BusinessService;

    @RequestMapping(
            value = "/ndc/log4j2",
            method = RequestMethod.POST)
    public ResponseEntity<Investment> postPayment(
            @RequestBody Investment investment) {

        ThreadContext.push("tx.id=" + investment.getTransactionId());
        ThreadContext.push("tx.owner=" + investment.getOwner());

        log4j2BusinessService.transfer(investment.getAmount());

        ThreadContext.pop();
        ThreadContext.pop();

        ThreadContext.clearAll();

        return new ResponseEntity<Investment>(investment, HttpStatus.OK);
    }
}
```

与Log4j一样，让我们在Log4j 2配置文件log4j2.xml中使用%x选项：

```xml
<Configuration status="INFO">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout
              pattern="%-4r [%t] %5p %c{1} - %m -%x%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="cn.tuyucheng.taketoday.log4j2" level="TRACE" />
            <AsyncRoot level="DEBUG">
            <AppenderRef ref="stdout" />
        </AsyncRoot>
    </Loggers>
</Configuration>
```

日志输出：

```text
204724 [http-nio-8080-exec-1]  INFO Log4J2InvestmentService 
  - Preparing to transfer 1500$. 
  - [tx.id=6, tx.owner=Samantha]
205455 [http-nio-8080-exec-2]  INFO Log4J2InvestmentService 
  - Preparing to transfer 2000$. 
  - [tx.id=4, tx.owner=Marc]
205525 [http-nio-8080-exec-1]  INFO Log4J2InvestmentService 
  - Has transfer of 1500$ completed successfully ? false. 
  - [tx.id=6, tx.owner=Samantha]
206064 [http-nio-8080-exec-2]  INFO Log4J2InvestmentService 
  - Has transfer of 2000$ completed successfully ? true. 
  - [tx.id=4, tx.owner=Marc]
...
```

## 6. 日志门面(JBoss日志)中的NDC

像SLF4J这样的日志门面提供与各种日志框架的集成，SLF4J不支持NDC(但包含在slf4j-ext模块中)。JBoss Logging是一个日志桥梁，与SLF4J一样；JBoss Logging支持NDC。

默认情况下，JBoss Logging将按照以下优先顺序在ClassLoader中搜索可用的后端/提供程序：JBoss LogManager、Log4j 2、Log4j、SLF4J和JDK Logging。

JBoss LogManager作为日志提供程序通常在WildFly应用服务器中使用，在我们的例子中，JBoss日志桥将按优先级选择下一个(即Log4j 2)作为日志提供程序。

让我们首先在pom.xml中添加所需的依赖：

```xml
<dependency>
    <groupId>org.jboss.logging</groupId>
    <artifactId>jboss-logging</artifactId>
    <version>3.3.0.Final</version>
</dependency>
```

你可以在[此处](https://mvnrepository.com/artifact/org.jboss.logging/jboss-logging)查看依赖的最新版本。

让我们向NDC堆栈添加上下文信息：

```java
import org.jboss.logging.NDC;

@RestController
public class JBossLoggingController {
    @Autowired
    @Qualifier("JBossLoggingInvestmentService")
    private InvestmentService jbossLoggingBusinessService;

    @RequestMapping(
            value = "/ndc/jboss-logging",
            method = RequestMethod.POST)
    public ResponseEntity<Investment> postPayment(
            @RequestBody Investment investment) {

        NDC.push("tx.id=" + investment.getTransactionId());
        NDC.push("tx.owner=" + investment.getOwner());

        jbossLoggingBusinessService.transfer(investment.getAmount());

        NDC.pop();
        NDC.pop();

        NDC.clear();

        return
                new ResponseEntity<Investment>(investment, HttpStatus.OK);
    }
}
```

日志输出：

```text
17045 [http-nio-8080-exec-1]  INFO JBossLoggingInvestmentService 
  - Preparing to transfer 1,500$. 
  - [tx.id=6, tx.owner=Samantha]
17725 [http-nio-8080-exec-1]  INFO JBossLoggingInvestmentService 
  - Has transfer of 1,500$ completed successfully ? true. 
  - [tx.id=6, tx.owner=Samantha]
18257 [http-nio-8080-exec-2]  INFO JBossLoggingInvestmentService 
  - Preparing to transfer 2,000$. 
  - [tx.id=4, tx.owner=Marc]
18904 [http-nio-8080-exec-2]  INFO JBossLoggingInvestmentService 
  - Has transfer of 2,000$ completed successfully ? true. 
  - [tx.id=4, tx.owner=Marc]
...
```

## 7. 总结

我们介绍了诊断上下文如何以有意义的方式帮助关联日志-从业务角度以及调试目的来看，这是一种丰富日志记录的宝贵技术，尤其是在多线程应用程序中。