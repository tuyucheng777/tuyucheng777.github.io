---
layout: post
title:  使用Spring Cloud Feign发布form-url-encoded的数据
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Feign
---

## 1. 概述

在本教程中，我们将学习如何使用[Feign Client](https://www.baeldung.com/intro-to-feign)在请求正文中使用form-url-encoded数据发出POST API请求。

## 2. POST form-url-encoded数据的方法

我们可以使用两种不同的方法来发送POST form-url-encoded数据，首先，我们需要创建一个自定义编码器，并将其配置到Feign客户端：
```java
class FormFeignEncoderConfig {
    @Bean
    public Encoder encoder(ObjectFactory<HttpMessageConverters> converters) {
        return new SpringFormEncoder(new SpringEncoder(converters));
    }
}
```

我们将在Feign客户端配置中使用这个自定义类：
```java
@FeignClient(name = "form-client", url = "http://localhost:8085/api",
  configuration = FormFeignEncoderConfig.class)
public interface FormClient {
    // request methods
}
```

现在，我们完成了Feign和Bean的配置，现在让我们看看请求方法。

### 2.1 使用POJO

我们将创建一个Java POJO类，并将所有表单参数作为成员：
```java
public class FormData {
    int id;
    String name;
    // constructors, getters and setters
}
```

我们将在POST请求中将该对象作为请求正文传递。
```java
@PostMapping(value = "/form", consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
void postFormData(@RequestBody FormData data);
```

让我们验证一下我们的代码；请求主体应该具有id和name作为form-url-encoded数据：
```java
@Test
public void givenFormData_whenPostFormDataCalled_thenReturnSuccess() {
    FormData formData = new FormData(1, "tuyucheng");
    stubFor(WireMock.post(urlEqualTo("/api/form"))
            .willReturn(aResponse().withStatus(HttpStatus.OK.value())));

    formClient.postFormData(formData);
    wireMockServer.verify(postRequestedFor(urlPathEqualTo("/api/form"))
            .withHeader("Content-Type", equalTo("application/x-www-form-urlencoded; charset=UTF-8"))
            .withRequestBody(equalTo("name=tuyucheng&id=1")));
}
```

### 2.2 使用Map

我们还可以使用[Map](https://www.baeldung.com/java-hashmap)而不是POJO类来在POST请求体中发送form-url-encoded数据。
```java
@PostMapping(value = "/form/map", consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
void postFormMapData(Map<String, ?> data);
```

**注意Map的值应该是‘?’**。

让我们验证一下我们的代码：
```java
@Test
public void givenFormMap_whenPostFormMapDataCalled_thenReturnSuccess() {
    Map<String, String> mapData = new HashMap<>();
    mapData.put("name", "tuyucheng");
    mapData.put("id", "1");
    stubFor(WireMock.post(urlEqualTo("/api/form/map"))
        .willReturn(aResponse().withStatus(HttpStatus.OK.value())));

    formClient.postFormMapData(mapData);
    wireMockServer.verify(postRequestedFor(urlPathEqualTo("/api/form/map"))
        .withHeader("Content-Type", equalTo("application/x-www-form-urlencoded; charset=UTF-8"))
        .withRequestBody(equalTo("name=tuyucheng&id=1")));
}
```

## 3. 总结

在本文中，我们了解了如何使用请求正文中的form-url-encoded数据发出POST API请求。