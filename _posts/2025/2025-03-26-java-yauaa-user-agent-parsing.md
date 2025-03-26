---
layout: post
title:  使用Yauaa进行用户代理解析
category: libraries
copyright: libraries
excerpt: Yauaa
---

## 1. 概述

在构建Web应用程序时，我们经常需要有关访问我们应用程序的设备和浏览器的信息，以便我们可以提供优化的用户体验。

[用户代理](https://developer.mozilla.org/en-US/docs/Glossary/User_agent)是一个字符串，用于标识向我们的服务器发出请求的客户端。它包含有关客户端设备、操作系统、浏览器等的信息，它通过User-Agent请求标头发送到服务器。

然而，由于用户代理字符串的结构复杂且性质多样，解析它们可能具有挑战性。在Java生态系统中，[Yuaa](https://github.com/nielsbasjes/yauaa)库有助于简化此过程。

**在本教程中，我们将探讨如何在Spring Boot应用程序中利用Yauaa来解析用户代理字符串并实现特定于设备的路由**。

## 2. 设置项目

在深入实现之前，我们需要包含SDK依赖并正确配置我们的应用程序。

### 2.1 依赖

让我们首先将[yauaa依赖](https://mvnrepository.com/artifact/nl.basjes.parse.useragent/yauaa/latest)添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>nl.basjes.parse.useragent</groupId>
    <artifactId>yauaa</artifactId>
    <version>7.28.1</version>
</dependency>
```

此依赖为我们提供了解析和分析来自应用程序的用户代理请求标头所需的类。

### 2.2 定义UserAgentAnalyzer配置Bean

现在我们已经添加了正确的依赖，让我们定义我们的UserAgentAnalyzer Bean：

```java
private static final int CACHE_SIZE = 1000;

@Bean
public UserAgentAnalyzer userAgentAnalyzer() {
    return UserAgentAnalyzer
            .newBuilder()
            .withCache(CACHE_SIZE)
            // ... other settings
            .build();
}
```

UserAgentAnalyzer类是解析用户代理字符串的主要入口点。

默认情况下，UserAgentAnalyzer使用大小为10000的内存缓存，但我们可以根据需要使用withCache()方法对其进行更新，就像我们在示例中所做的那样。**缓存可避免重复解析相同的用户代理字符串，从而帮助我们提高性能**。

## 3. 探索UserAgent字段

一旦我们定义了UserAgentAnalyzer Bean，我们就可以使用它来解析用户代理字符串并获取有关客户端的信息。

为了演示，我们接收用于编写本教程的设备的用户代理：

```text
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.6 Safari/605.1.15
```

现在，我们将使用UserAgentAnalyzer Bean来解析上述内容并提取所有可用字段：

```java
UserAgent userAgent = userAgentAnalyzer.parse(USER_AGENT_STRING);
userAgent
    .getAvailableFieldNamesSorted()
    .forEach(fieldName -> {
        log.info("{}: {}", fieldName, userAgent.getValue(fieldName));
    });
```

我们来看一下执行上述代码时生成的日志：

```text
cn.tuyucheng.taketoday.Application : DeviceClass: Desktop
cn.tuyucheng.taketoday.Application : DeviceName: Apple Macintosh
cn.tuyucheng.taketoday.Application : DeviceBrand: Apple
cn.tuyucheng.taketoday.Application : DeviceCpu: Intel
cn.tuyucheng.taketoday.Application : DeviceCpuBits: 64
cn.tuyucheng.taketoday.Application : OperatingSystemClass: Desktop
cn.tuyucheng.taketoday.Application : OperatingSystemName: Mac OS
cn.tuyucheng.taketoday.Application : OperatingSystemVersion: >=10.15.7
cn.tuyucheng.taketoday.Application : OperatingSystemVersionMajor: >=10.15
cn.tuyucheng.taketoday.Application : OperatingSystemNameVersion: Mac OS >=10.15.7
cn.tuyucheng.taketoday.Application : OperatingSystemNameVersionMajor: Mac OS >=10.15
cn.tuyucheng.taketoday.Application : LayoutEngineClass: Browser
cn.tuyucheng.taketoday.Application : LayoutEngineName: AppleWebKit
cn.tuyucheng.taketoday.Application : LayoutEngineVersion: 605.1.15
cn.tuyucheng.taketoday.Application : LayoutEngineVersionMajor: 605
cn.tuyucheng.taketoday.Application : LayoutEngineNameVersion: AppleWebKit 605.1.15
cn.tuyucheng.taketoday.Application : LayoutEngineNameVersionMajor: AppleWebKit 605
cn.tuyucheng.taketoday.Application : AgentClass: Browser
cn.tuyucheng.taketoday.Application : AgentName: Safari
cn.tuyucheng.taketoday.Application : AgentVersion: 17.6
cn.tuyucheng.taketoday.Application : AgentVersionMajor: 17
cn.tuyucheng.taketoday.Application : AgentNameVersion: Safari 17.6
cn.tuyucheng.taketoday.Application : AgentNameVersionMajor: Safari 17
cn.tuyucheng.taketoday.Application : AgentInformationEmail: Unknown
cn.tuyucheng.taketoday.Application : WebviewAppName: Unknown
cn.tuyucheng.taketoday.Application : WebviewAppVersion: ??
cn.tuyucheng.taketoday.Application : WebviewAppVersionMajor: ??
cn.tuyucheng.taketoday.Application : WebviewAppNameVersion: Unknown
cn.tuyucheng.taketoday.Application : WebviewAppNameVersionMajor: Unknown
cn.tuyucheng.taketoday.Application : NetworkType: Unknown
```

**我们可以看到，UserAgent类提供了有关设备、操作系统、浏览器等的宝贵信息**，我们可以使用这些字段并根据业务需求做出明智的决策。

还要注意的是，对于给定的用户代理字符串，并非所有字段都可用，这从日志语句中的Unknown和??值可以看出。如果我们不想显示这些不可用的值，我们可以改用UserAgent类的getCleanedAvailableFieldNamesSorted()方法。

## 4. 实现基于设备的路由

现在我们已经了解了UserAgent类中可用的各种字段，**让我们通过在我们的应用程序中实现基于设备的路由来利用这些知识**。

为了演示，**我们假设需要将应用程序提供给移动设备，同时限制对非移动设备的访问**，我们可以通过检查用户代理字符串的DeviceClass字段来实现这一点。

首先，让我们定义一个受支持的设备类别列表，我们将其视为移动设备：

```java
private static final List SUPPORTED_MOBILE_DEVICE_CLASSES = List.of("Mobile", "Tablet", "Phone");
```

接下来，让我们创建控制器方法：

```java
@GetMapping("/mobile/home")
public ModelAndView homePage(@RequestHeader(HttpHeaders.USER_AGENT) String userAgentString) {
    UserAgent userAgent = userAgentAnalyzer.parse(userAgentString);
    String deviceClass = userAgent.getValue(UserAgent.DEVICE_CLASS);
    boolean isMobileDevice = SUPPORTED_MOBILE_DEVICE_CLASSES.contains(deviceClass);

    if (isMobileDevice) {
        return new ModelAndView("/mobile-home");
    }
    return new ModelAndView("error/open-in-mobile", HttpStatus.FORBIDDEN);
}
```

在我们的方法中，我们使用[@RequestHeader](https://www.baeldung.com/spring-rest-http-headers)获取用户代理字符串的值，然后使用UserAgentAnalyzer Bean对其进行解析。然后，我们从解析的UserAgent实例中提取DEVICE_CLASS字段，并检查它是否与任何受支持的移动设备类匹配。

如果我们确定传入请求来自移动设备，我们将返回/mobile-home视图。否则，我们将返回HTTP状态为Forbidden的错误视图，表示该资源只能从移动设备访问。

## 5. 测试基于设备的路由

现在我们已经实现了基于设备的路由，让我们使用[MockMvc](https://www.baeldung.com/integration-testing-in-spring)对其进行测试以确保它按预期工作：

```java
private static final String SAFARI_MAC_OS_USER_AGENT = "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Safari/605.1.15";

mockMvc.perform(get("/mobile/home")
    .header("User-Agent", SAFARI_MAC_OS_USER_AGENT))
    .andExpect(view().name("error/open-in-mobile"))
    .andExpect(status().isForbidden());
```

我们使用适用于MacOS的Safari用户代理字符串模拟请求并调用我们的/home端点，我们验证了我们的错误视图已返回到客户端，HTTP状态为403 forbidden。

类似地，现在让我们使用移动用户代理调用我们的端点：

```java
private static final String SAFARI_IOS_USER_AGENT = "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1";

mockMvc.perform(get("/mobile/home")
    .header("User-Agent", SAFARI_IOS_USER_AGENT))
    .andExpect(view().name("/mobile-home"))
    .andExpect(status().isOk());
```

在这里，我们使用IOS的Safari用户代理字符串，并断言请求已成功完成并向客户端返回正确的视图。

## 6. 减少内存占用并加快执行速度

默认情况下，Yuaa会解析用户代理字符串中的所有可用字段。但是，如果我们只需要应用程序中的子集字段，则可以在UserAgentAnalyzer Bean中指定它们：

```java
UserAgentAnalyzer
    .newBuilder()
    .withField(UserAgent.DEVICE_CLASS)
    // ... other settings
    .build();
```

在这里，我们配置UserAgentAnalyzer以仅解析我们在基于设备的路由实现中使用的DEVICE_CLASS字段。

如果需要解析多个字段，我们可以将多个withField()方法链接在一起。**强烈推荐这种方法，因为它有助于减少内存使用并提高性能**。

## 7. 总结

在本文中，我们介绍了使用Yauaa来解析和分析用户代理字符串。

我们完成了必要的配置，查看了我们可以访问的不同字段，并在我们的应用程序中实现了基于设备的路由。