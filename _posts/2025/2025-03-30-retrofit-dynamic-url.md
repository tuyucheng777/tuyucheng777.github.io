---
layout: post
title:  Retrofit 2 – 动态URL
category: libraries
copyright: libraries
excerpt: Retrofit
---

## 1. 概述

在这个简短的教程中，我们将学习如何在[Retrofit 2](https://www.baeldung.com/retrofit)中创建动态URL。

## 2. @Url注解

在某些情况下，我们需要在运行时在应用程序中使用动态URL。[Retrofit](https://mvnrepository.com/artifact/com.squareup.retrofit2/retrofit)库的第2个版本引入了@Url注解，允许我们为端点传递完整的URL：

```java
@GET
Call<ResponseBody> reposList(@Url String url);
```

该注解基于[OkHttp](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)库中的[HttpUrl](https://square.github.io/okhttp/5.x/okhttp/okhttp3/-http-url/index.html)类，使用<a href= ""\>像页面上的链接一样解析URL地址。使用@Url参数时，我们不需要在@GET注解中指定地址。

@Url参数替换了服务实现中的baseUrl：

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .addConverterFactory(GsonConverterFactory.create()).build();
```

重要的是，如果我们要使用@Url注解，则必须将其设置为服务方法中的第一个参数。

## 3. 路径参数

如果我们知道基本URL的某些部分是不变的，但不知道它的扩展名或将使用的参数数量，则可以**使用@Path注解和encoded标志**：

```java
@GET("{fullUrl}")
Call<List<Contributor>> contributorsList(@Path(value = "fullUrl", encoded = true) String fullUrl);
```

这样，所有的“/”就不会被替换为%2F，就像我们没有使用编码参数一样。但是，传递的地址中的所有字符“?”仍然会被替换为%3F。

## 4. 总结

Retrofit库允许我们在应用程序运行时仅使用@Url注解轻松提供动态URL。