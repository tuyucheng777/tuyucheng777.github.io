---
layout: post
title:  Swagger Parser指南
category: libraries
copyright: libraries
excerpt: Swagger
---

## 1. 简介

[Swagger](https://swagger.io/)是一组用于设计、描述和记录RESTful API的工具。

在本教程中，**我们将探讨如何使用Java解析OpenAPI文档文件并提取其各种组件**。

## 2. 什么是Swagger？

Swagger本质上是一套用于开发和描述REST API的开源规则、规范和工具。然而，随着新标准和规范的发展，这些规范现在更名为[OpenAPI规范](https://www.openapis.org/)(OAS)。

**OpenAPI规范标准化了如何创建API设计文档**，它创建了一个RESTful接口，我们可以使用它来轻松开发和使用API。API规范有效地映射了与其相关的所有资源和操作。

OpenAPI文档是一种自包含或复合资源，它定义API及其各种元素。该文档可以采用[JSON或YAML](https://www.baeldung.com/yaml-json-differeneces)格式表示。

OpenAPI规范的最新版本是[OAS 3.1](https://spec.openapis.org/oas/v3.1.0)，它允许我们指定HTTP资源、动词、响应代码、数据模型、媒体类型、安全方案和其他API组件。我们可以使用OpenAPI定义来生成文档、代码生成和许多其他用例。

另一方面，**Swagger已发展成为开发API的最广泛使用的开源工具集之一。它基本上提供了一套完整的工具集来设计、构建和记录API**。

为了验证OpenAPI文档，我们使用[Swagger Validator](https://validator.swagger.io/)工具。此外，[Swagger Editor](https://editor.swagger.io/)提供了一个基于GUI的编辑器，可帮助我们在运行时编辑和可视化API文档。

我们可以轻松地将生成的OpenAPI文档与第三方工具一起使用，例如[导入到Postman](https://www.baeldung.com/swagger-apis-in-postman)中。

## 3. 使用Swagger解析器

**Swagger Parser是Swagger工具之一，可以帮助我们解析文档并提取其各个组件**。接下来我们来看看如何用Java实现解析器：

### 3.1 依赖

在开始之前，让我们将Swagger Parser的[Maven依赖](https://mvnrepository.com/artifact/io.swagger.parser.v3/swagger-parser)添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>io.swagger.parser.v3</groupId>
    <artifactId>swagger-parser</artifactId>
    <version>2.1.13</version>
</dependency>
```

接下来，让我们深入了解如何解析文档。

### 3.2 OpenAPI文档示例

在开始之前，我们需要一些可以解析的示例OpenAPI文档，让我们使用以下名为sample.yml的示例OpenAPI YAML文档：

```yaml
openapi: 3.0.0
info:
    title: User APIs
    version: '1.0'
servers:
    -   url: https://jsonplaceholder.typicode.com
        description: Json Place Holder Service
paths:
    /users/{id}:
        parameters:
            -   schema:
                    type: integer
                name: id
                in: path
                required: true
        get:
            summary: Fetch user by ID
            tags:
                - User
            responses:
                '200':
                    description: OK
                    content:
                        application/json:
                            schema:
                                type: object
                                properties:
                                    id:
                                        type: integer
                                    name:
                                        type: string
            operationId: get-users-user_id
            description: Retrieve a specific user by ID
```

上述YAML是一个非常简单的OpenAPI规范，它定义了一个通过ID获取用户详细信息的API。

类似地，我们还有一个名为sample.json的等效JSON文档文件：

```json
{
    "openapi": "3.0.0",
    "info": {
        "title": "User APIs",
        "version": "1.0"
    },
    "servers": [
        {
            "url": "https://jsonplaceholder.typicode.com",
            "description": "Json Place Holder Service"
        }
    ],
    "paths": {
        "/users/{id}": {
            "parameters": [
                {
                    "schema": {
                        "type": "integer"
                    },
                    "name": "id",
                    "in": "path",
                    "required": true
                }
            ],
            "get": {
                "summary": "Fetch user by ID",
                "tags": [
                    "User"
                ],
                "responses": {
                    "200": {
                        "description": "OK",
                        "content": {
                            "application/json": {
                                "schema": {
                                    "type": "object",
                                    "properties": {
                                        "id": {
                                            "type": "integer"
                                        },
                                        "name": {
                                            "type": "string"
                                        }
                                    }
                                }
                            }
                        }
                    }
                },
                "operationId": "get-users-user_id",
                "description": "Retrieve a specific user by ID"
            }
        }
    }
}
```

我们将使用这些OpenAPI文档文件作为我们所有的编码示例。

现在让我们看看如何解析该文档。

### 3.3 解析OpenAPI YAML文档

首先，**我们使用OpenAPIParser().readLocation()方法读取并解析YAML或JSON文件，此方法接收三个参数**：

- String：我们要读取的文件的URL
- List<AuthorizationValue\>：如果要读取的文档受到保护，则要传递的授权标头列表
- ParserOptions：额外解析选项，用于自定义解析时的行为

首先，让我们检查一下从URL读取OpenAPI文档的代码片段：

```java
SwaggerParseResult result = new OpenAPIParser().readLocation("sample.yml", null, null);
```

readLocation()方法返回包含解析结果的SwaggerParserResult实例。

其次，我们将使用返回的SwaggerParserResult实例来获取解析的详细信息：

```java
OpenAPI openAPI = result.getOpenAPI();
```

SwaggerParserResult.get()方法返回OpenAPI类的实例，返回的OpenAPI类实例基本上是OpenAPI文档的POJO版本。

最后，我们现在可以使用获取的OpenAPI实例中的各种Getter方法来获取OpenAPI文档的各个组件：

```java
SpecVersion version = openAPI.getSpecVersion();

Info info = openAPI.getInfo();

List<Server> servers = openAPI.getServers();

Paths paths = openAPI.getPaths();
```

### 3.4 解析JSON文档

类似地，我们也可以解析等效的JSON文档文件，让我们通过将其文件名作为URL传递来解析sample.json文件：

```java
SwaggerParseResult result = new OpenAPIParser().readLocation("sample.json", null, null);
```

此外，**我们甚至可以使用OpenAPIParser().readContents(String swaggerString, List<AuthorizationValue\> auth, ParseOptions options)方法从字符串解析OpenAPI规范文档**。

另外，我们可以通过调用SwaggerParserResult.getMessages()方法来获取解析过程中的任何验证错误和警告，此方法返回包含错误消息的字符串列表：

```java
List<String> messages = result.getMessages();
```

## 4. 总结

在本文中，我们研究了OpenAPI规范和Swagger的基础知识。

我们了解了如何使用Java解析OpenAPI文档文件，我们实现了解析YAML和JSON规范文件的代码。