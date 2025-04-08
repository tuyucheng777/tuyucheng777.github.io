---
layout: post
title:  Jersey中的@FormDataParam和@FormParam
category: webmodules
copyright: webmodules
excerpt: Jersey
---

##  1. 简介

[Jersey](https://www.baeldung.com/jersey-rest-api-with-spring)是一个功能齐全的开源框架，用于使用Java开发Web服务和客户端。**通过使用Jersey，我们可以创建支持全套HTTP功能的强大Web应用程序**。

在本文中，我们将介绍Jersey的两个特定功能：@FormDataParam和@FormParam注解。虽然这两个注解本质上相似，但它们有显著差异，我们将在下面看到。

## 2. 背景

客户端和服务器之间交换数据的方式有很多种，最流行的两种方式是XML和JSON，但它们并不是唯一的选择。

**表单编码数据是在客户端和服务器之间发送数据的另一种格式化方式，特别是当客户端是网页时**。这是因为HTML可以轻松定义具有各种输入(文本、复选框、下拉菜单等)的表单，并将该数据发送回远程服务器。

这就是@FormDataParam与@FormParam注解的作用所在，**它们都用于在Jersey服务器应用程序中处理表单数据，但有一些重要的区别**。

现在我们知道了这两种注解的背景，让我们来看看如何使用它们以及它们的一些区别。

## 3. 使用@FormParam

简而言之，只要API需要URL编码的数据，我们就使用@FormParam注解。让我们看一个只有文本字段的简单HTML表单：

```html
<form method="post" action="/example1">
    <input name="first_name" type="text">
    <input name="last_name" type="text">
    <input name="age" type="text">
    <input type="submit">
</form>
```

此表单有3个字段：first_name、last_name和age，它还使用HTTP POST方法将表单数据发送到位于/example1的URL。

为了使用Jersey处理此表单，我们需要在服务器应用程序中定义以下端点：

```java
@POST
@Path("/example1")
public String example1(
        @FormParam("first_name") String firstName,
        @FormParam("last_name") String lastName,
        @FormParam("age") String age)
{
    // process form data
}
```

**请注意，我们对HTML表单中的每个字段都使用了注解**。无论表单有多少个字段或字段类型，其工作原理都相同。通过使用@FormParam注解，Jersey将每个字段的值绑定到相应的方法参数中。

实际上，表单编码数据与[MIME类型](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)application/x-www-form-urlencoded相关。在底层，Web浏览器和我们的服务之间发送的数据如下所示：

```text
first_name=john&last_name=smith&age=42
```

这种方法的好处是数据易于处理，因为它只涉及文本而不涉及二进制数据。此外，这种方法可以使用HTTP GET动词，因为字符串可以作为URL中的查询参数传递。**但是，通过URL传递表单数据时我们应该小心，因为它可能包含敏感数据**。

## 4. 使用@FormDataParam

**相比之下，@FormDataParam注解更加灵活，可以处理文本和二进制数据的任意组合**。实际上，这对于使用HTML表单进行文件上传等操作非常有用。

让我们看一个使用@FormDataParam注解的示例表单。首先，我们必须先将jersey-media-multipart模块从[Maven Central](https://mvnrepository.com/artifact/org.glassfish.jersey.media/jersey-media-multipart)导入到项目中：

```xml
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-multipart</artifactId>
    <version>3.1.3</version>
</dependency>
```

现在，让我们向上面创建的HTML表单添加一个文件上传字段：

```html
<form method="post" action="/example2" enctype="multipart/form-data >
    <input name="first_name" type="text">
    <input name="last_name" type="text">
    <input name="age" type="text">
    <input name="photo" type="file">
    <input type="submit">
</form>
```

请注意，我们现在在表单元素中指定了multipart/form-data的编码类型。这是必要的，以便Web浏览器能够以我们的服务期望的格式发送数据。

接下来，在我们的应用程序中，我们将创建以下端点来处理来自此表单的数据：

```java
@POST
@Path("/example2")
public String example2(
        @FormDataParam("first_name") String firstName,
        @FormDataParam("last_name") String lastName,
        @FormDataParam("age") String age,
        @FormDataParam("photo") InputStream photo)
{
    // handle form data
}
```

请注意，每个表单字段现在都使用了@FormDataParam注解，我们还使用InputStream作为处理二进制数据的参数类型。

与上一个示例(其中每个字段及其值都组合成一个字符串)不同，此示例将字段组合成多个部分，每个部分由唯一的标记分隔，允许Jersey轻松识别每个参数并将每个部分绑定到相应的方法参数中。

例如，从此表单发送到我们的服务的原始消息可能如下所示：

```text
------WebKitFormBoundarytCyB57mkvJedAHFx
Content-Disposition: form-data; name="first_name"

John
------WebKitFormBoundarytCyB57mkvJedAHFx
Content-Disposition: form-data; name="last_name"

Smith
------WebKitFormBoundarytCyB57mkvJedAHFx
Content-Disposition: form-data; name="age"

42
------WebKitFormBoundarytCyB57mkvJedAHFx
Content-Disposition: form-data; name="photo"; filename="john-smith-profile.jpeg"
Content-Type: image/jpeg


------WebKitFormBoundarytCyB57mkvJedAHFx--
```

**这种方法的主要优点是我们可以同时发送文本和二进制数据**；但是，与使用传统的表单编码数据相比，这种方法也存在一些缺点。

**首先，我们只能对这种类型使用HTTP POST动词**，这是因为二进制数据无法编码并随URL字符串传递。其次，有效负载大小要大得多。如我们在上例中看到的，定义分隔符和标头所需的额外文本比普通表单编码数据的大小要大得多。

## 5. 总结

在本文中，我们研究了Jersey库中的两个不同注解：@FormParam和@FormDataParam。这两个注解都处理应用程序中的表单数据，但正如我们所见，它们的用途截然不同。

@FormParam更适合发送表单编码数据，它可以使用GET或POST动词，但只能包含文本字段。另一方面，@FormDataParam注解处理多部分数据，它可以包括文本和二进制数据，但仅适用于POST动词。