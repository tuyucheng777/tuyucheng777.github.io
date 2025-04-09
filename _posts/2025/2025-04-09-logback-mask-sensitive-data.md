---
layout: post
title:  使用Logback屏蔽日志中的敏感数据
category: log
copyright: log
excerpt: Logback
---

## 1. 概述

在新的GDPR时代，在众多问题中，我们必须特别注意记录个人的敏感数据。由于记录的数据量很大，因此在记录时隐藏用户的敏感信息非常重要。

在本教程中，我们将了解如何使用Logback屏蔽日志中的敏感数据。虽然这种方法是日志文件的最后一道防线，但它并不被视为问题的最终解决方案。

## 2. Logback

[Logback](https://www.baeldung.com/logback)是Java社区中使用最广泛的日志框架之一，它取代了其前身Log4j。Logback提供了更快的实现、更多的配置选项以及存档旧日志文件的更大灵活性。

敏感数据是指任何需要保护以防止未经授权访问的信息，这可以包括从个人身份信息(PII)(例如社会安全号码)到银行信息、登录凭据、地址、电子邮件等任何内容。

我们将在用户登录我们的应用程序日志时屏蔽属于用户的敏感数据。

## 3. 屏蔽数据

假设我们在Web请求的上下文中记录用户详细信息，我们需要屏蔽与用户相关的敏感数据，假设我们的应用程序收到我们记录的以下请求或响应：

```json
{
    "user_id": "87656",
    "ssn": "786445563",
    "address": "22 Street",
    "city": "Chicago",
    "Country": "U.S.",
    "ip_address":"192.168.1.1",
    "email_id":"spring@tuyucheng.com"
}
```

在这里，我们可以看到我们有敏感数据，如ssn、address、ip_address和email_id。因此，我们必须在记录时屏蔽这些数据。

我们将通过为Logback生成的所有日志条目配置屏蔽规则来集中屏蔽日志，**为此，我们必须实现自定义ch.qos.logback.classic.PatternLayout**。

### 3.1 PatternLayout

配置背后的想法是**使用自定义布局扩展我们需要的每个Logback附加器**，在我们的例子中，我们将编写一个MaskingPatternLayout类作为PatternLayout的实现，每个屏蔽模式代表与一种敏感数据匹配的正则表达式。

让我们构建MaskingPatternLayout类：

```java
public class MaskingPatternLayout extends PatternLayout {

    private Pattern multilinePattern;
    private List<String> maskPatterns = new ArrayList<>();

    public void addMaskPattern(String maskPattern) {
        maskPatterns.add(maskPattern);
        multilinePattern = Pattern.compile(maskPatterns.stream().collect(Collectors.joining("|")), Pattern.MULTILINE);
    }

    @Override
    public String doLayout(ILoggingEvent event) {
        return maskMessage(super.doLayout(event));
    }

    private String maskMessage(String message) {
        if (multilinePattern == null) {
            return message;
        }
        StringBuilder sb = new StringBuilder(message);
        Matcher matcher = multilinePattern.matcher(sb);
        while (matcher.find()) {
            IntStream.rangeClosed(1, matcher.groupCount()).forEach(group -> {
                if (matcher.group(group) != null) {
                    IntStream.range(matcher.start(group), matcher.end(group)).forEach(i -> sb.setCharAt(i, '*'));
                }
            });
        }
        return sb.toString();
    }
}
```

PatternLayout.doLayout()的实现负责在应用程序的每个日志消息中屏蔽与配置的模式之一匹配的数据。

logback.xml中的maskPatterns列表构造了一个多行模式，不幸的是，Logback引擎不支持构造函数注入。如果它以属性列表的形式出现，则每个配置条目都会调用addMaskPattern。因此，每次向列表中添加新的正则表达式时，我们都必须编译该模式。

### 3.2 配置

一般来说，我们可以**使用正则表达式模式来屏蔽敏感的用户详细信息**。

例如，对于SSN，我们可以使用如下正则表达式：

```text
\"SSN\"\s*:\s*\"(.*)\"
```

对于地址，我们可以使用：

```text
\"address\"\s*:\s*\"(.*?)\" 
```

此外，对于IP地址数据模式(192.169.0.1)，我们可以使用正则表达式：

```text
(\d+\.\d+\.\d+\.\d+)
```

最后，对于电子邮件，我们可以写：

```text
([\w.-]+@[\w.-]+\.\w+)
```

现在，我们将在logback.xml文件内的maskPattern标签中添加这些正则表达式模式：

```xml
<configuration>
    <appender name="mask" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="cn.tuyucheng.taketoday.logback.MaskingPatternLayout">
                <maskPattern>\"SSN\"\s*:\s*\"(.*?)\"</maskPattern> <!-- SSN JSON pattern -->
                <maskPattern>\"address\"\s*:\s*\"(.*?)\"</maskPattern> <!-- Address JSON pattern -->
                <maskPattern>(\d+\.\d+\.\d+\.\d+)</maskPattern> <!-- Ip address IPv4 pattern -->
                <maskPattern>([\w.-]+@[\w.-]+\.\w+)</maskPattern> <!-- Email pattern -->
                <pattern>%-5p [%d{ISO8601,UTC}] [%thread] %c: %m%n%rootException</pattern>
            </layout>
        </encoder>
    </appender>
</ configuration>
```

### 3.3 执行

现在，我们将为上述示例创建JSON并使用logger.info()记录详细信息：

```java
Map<String, String> user = new HashMap<String, String>();
user.put("user_id", "87656");
user.put("SSN", "786445563");
user.put("address", "22 Street");
user.put("city", "Chicago");
user.put("Country", "U.S.");
user.put("ip_address", "192.168.1.1");
user.put("email_id", "spring-boot.3@tuyucheng.cs.com");
JSONObject userDetails = new JSONObject(user);

logger.info("User JSON: {}", userDetails);
```

执行完之后我们可以看到输出：

```text
INFO  [2021-06-01 16:04:12,059] [main] cn.tuyucheng.taketoday.logback.MaskingPatternLayoutExample: User JSON: 
{"email_id":"*******************","address":"*********","user_id":"87656","city":"Chicago","Country":"U.S.", "ip_address":"***********","SSN":"*********"}
```

在这里，我们可以看到我们的记录器中的用户JSON已被屏蔽：

```json
{
    "user_id":"87656",
    "ssn":"*********",
    "address":"*********",
    "city":"Chicago",
    "Country":"U.S.",
    "ip_address":"*********",
    "email_id":"*****************"
}
```

通过这种方法，我们只能屏蔽日志文件中那些我们已经在logback.xml中的maskPattern中定义了正则表达式的数据。

## 4. 总结

在本教程中，我们介绍了如何使用PatternLayout功能通过Logback屏蔽应用程序日志中的敏感数据，以及如何在logback.xml中添加正则表达式模式来屏蔽特定数据。