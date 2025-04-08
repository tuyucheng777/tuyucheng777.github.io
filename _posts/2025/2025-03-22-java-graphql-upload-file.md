---
layout: post
title:  在Java中使用GraphQL上传文件
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

**[GraphQL](https://www.baeldung.com/graphql)改变了开发人员与API交互的方式，为传统[REST](https://www.baeldung.com/cs/rest-architecture)方法提供了一种简化且强大的替代方案**。

但是，由于GraphQL处理二进制数据的性质，在Java中使用GraphQL处理文件上传(特别是在Spring Boot应用程序中)需要进行一些设置。在本教程中，我们将介绍如何[在Spring Boot应用程序中使用GraphQL](https://www.baeldung.com/spring-graphql)设置文件上传。

## 2. 使用GraphQL与HTTP上传文件

**在使用Spring Boot开发GraphQL API领域中，遵守最佳实践通常涉及利用标准[HTTP](https://www.baeldung.com/java-9-http-client)请求来处理文件上传**。

通过专用HTTP端点管理文件上传，然后通过URL或ID等标识符将这些上传链接到GraphQL变更，开发人员可以有效地最大限度地降低通常与将文件上传直接嵌入GraphQL查询相关的复杂性和处理开销。这种方法不仅简化了上传过程，还有助于避免与文件大小限制和序列化需求相关的潜在问题，从而有助于实现更精简和可扩展的应用程序结构。

尽管如此，某些情况下还是需要将文件上传直接合并到GraphQL查询中。在这种情况下，将文件上传功能集成到GraphQL API中需要一种量身定制的策略，以仔细平衡用户体验和应用程序性能。因此，我们需要定义一个专门的标量类型来处理上传。此外，此方法涉及部署特定机制来验证输入并将上传的文件映射到GraphQL操作中的正确变量。此外，上传文件需要请求主体的multipart/form-data内容类型，因此我们需要实现自定义HttpHandler。

## 3. GraphQL中的文件上传实现

本节概述了使用[Spring Boot](https://www.baeldung.com/spring-boot)在GraphQL API中集成文件上传功能的全面方法，通过一系列步骤，我们将探索创建和配置旨在直接通过GraphQL查询处理文件上传的基本组件。

在本指南中，我们将利用专门的[Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-graphql)在Spring Boot应用程序中启用GraphQL支持：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
    <version>3.3.0</version>
</dependency>
```

### 3.1 自定义Upload标量类型

首先，我们在GraphQL模式中定义自定义标量类型Upload。引入Upload标量类型扩展了GraphQL处理二进制文件数据的能力，使API能够接收文件上传。**自定义标量充当客户端文件上传请求与服务器处理逻辑之间的桥梁，确保以类型安全且结构化的方式处理文件上传**。

我们在src/main/resources/file-upload/graphql/upload.graphqls文件中定义它：

```graphql
scalar Upload

type Mutation {
    uploadFile(file: Upload!, description: String!): String
}

type Query {
    getFile: String
}
```

在上面的定义中，我们还有description参数来说明如何随文件传递附加数据。

### 3.2 UploadCoercing实现

**在GraphQL中，强制转换是指将值从一种类型转换为另一种类型的过程**，这在处理自定义标量类型(如我们的Upload类型)时尤其重要。在这种情况下，我们需要定义如何解析(从输入转换)和序列化(转换为输出)与此类型关联的值。

UploadCoercing实现对于以符合GraphQL API中文件上传的操作要求的方式管理这些转换至关重要。

让我们定义UploadCoercing类来正确处理Upload类型：

```java
public class UploadCoercing implements Coercing<MultipartFile, Void> {
    @Override
    public Void serialize(Object dataFetcherResult) {
        throw new CoercingSerializeException("Upload is an input-only type and cannot be serialized");
    }

    @Override
    public MultipartFile parseValue(Object input) {
        if (input instanceof MultipartFile) {
            return (MultipartFile) input;
        }
        throw new CoercingParseValueException("Expected type MultipartFile but was " + input.getClass().getName());
    }

    @Override
    public MultipartFile parseLiteral(Object input) {
        throw new CoercingParseLiteralException("Upload is an input-only type and cannot be parsed from literals");
    }
}
```

我们可以看到，这涉及将输入值(来自查询或突变)转换为我们的应用程序可以理解和使用的Java类型。对于Upload标量，这意味着从客户端获取文件输入并确保它在我们的服务器端代码中正确表示为MultipartFile。

### 3.3 MultipartGraphQlHttpHandler：处理多部分请求

GraphQL在其标准规范中旨在处理JSON格式的请求，这种格式非常适合典型的[CRUD](https://baeldung.com/spring-boot-react-crud)操作，但在处理文件上传时却不尽如人意，因为文件上传本质上是二进制数据，不易用JSON表示。multipart/form-data内容类型是通过HTTP提交表单和上传文件的标准，但处理这些请求需要以不同于标准GraphQL请求的方式解析请求正文。

**默认情况下，GraphQL服务器不直接理解或处理多部分请求，这通常会导致此类请求出现404 Not Found响应**。因此，我们需要实现一个处理程序来弥补这一缺陷，确保我们的应用程序能够正确处理multipart/form-data内容类型。

让我们实现这个类：

```java
public ServerResponse handleMultipartRequest(ServerRequest serverRequest) throws ServletException {
    HttpServletRequest httpServletRequest = serverRequest.servletRequest();

    Map<String, Object> inputQuery = Optional.ofNullable(this.<Map<String, Object>>deserializePart(httpServletRequest, "operations", MAP_PARAMETERIZED_TYPE_REF.getType())).orElse(new HashMap<>());

    final Map<String, Object> queryVariables = getFromMapOrEmpty(inputQuery, "variables");
    final Map<String, Object> extensions = getFromMapOrEmpty(inputQuery, "extensions");

    Map<String, MultipartFile> fileParams = readMultipartFiles(httpServletRequest);

    Map<String, List<String>> fileMappings = Optional.ofNullable(this.<Map<String, List<String>>>deserializePart(httpServletRequest, "map", LIST_PARAMETERIZED_TYPE_REF.getType())).orElse(new HashMap<>());

    fileMappings.forEach((String fileKey, List<String> objectPaths) -> {
        MultipartFile file = fileParams.get(fileKey);
        if (file != null) {
            objectPaths.forEach((String objectPath) -> MultipartVariableMapper.mapVariable(objectPath, queryVariables, file));
        }
    });

    String query = (String) inputQuery.get("query");
    String opName = (String) inputQuery.get("operationName");

    Map<String, Object> body = new HashMap<>();
    body.put("query", query);
    body.put("operationName", StringUtils.hasText(opName) ? opName : "");
    body.put("variables", queryVariables);
    body.put("extensions", extensions);

    WebGraphQlRequest graphQlRequest = new WebGraphQlRequest(serverRequest.uri(), serverRequest.headers().asHttpHeaders(), body, this.idGenerator.generateId().toString(), LocaleContextHolder.getLocale());

    if (logger.isDebugEnabled()) {
        logger.debug("Executing: " + graphQlRequest);
    }

    Mono<ServerResponse> responseMono = this.graphQlHandler.handleRequest(graphQlRequest).map(response -> {
        if (logger.isDebugEnabled()) {
            logger.debug("Execution complete");
        }
        ServerResponse.BodyBuilder builder = ServerResponse.ok();
        builder.headers(headers -> headers.putAll(response.getResponseHeaders()));
        builder.contentType(selectResponseMediaType(serverRequest));
        return builder.body(response.toMap());
    });

    return ServerResponse.async(responseMono);
}
```

MultipartGraphQlHttpHandler类中的handleMultipartRequest方法处理multipart/form-data请求。首先，我们从服务器请求对象中提取HTTP请求，该请求允许访问请求中包含的多部分文件和其他表单数据。然后，我们尝试反序列化请求的“operations”部分，其中包含GraphQL查询或变异，以及“map”部分，它指定如何将文件映射到GraphQL操作中的变量。

在反序列化这些部分之后，该方法继续从请求中读取实际的文件上传，使用“map”中定义的映射将每个上传的文件与GraphQL操作中的正确变量关联起来。

### 3.4 实现文件上传DataFetcher

由于我们有用于上传文件的uploadFile变异，因此我们需要实现特定逻辑来从客户端接收文件和其他元数据并保存文件。

**在GraphQL中，模式中的每个字段都链接到DataFetcher，该组件负责检索与该字段关联的数据**。

虽然某些字段可能需要专门的DataFetcher实现才能从数据库或其他持久存储系统获取数据，但许多字段只是从内存对象中提取数据。这种提取通常依赖于字段名称并利用标准Java对象模式来访问所需的数据。

让我们实现DataFetcher接口的实现：

```java
@Component
public class FileUploadDataFetcher implements DataFetcher<String> {
    private final FileStorageService fileStorageService;

    public FileUploadDataFetcher(FileStorageService fileStorageService) {
        this.fileStorageService = fileStorageService;
    }

    @Override
    public String get(DataFetchingEnvironment environment) {
        MultipartFile file = environment.getArgument("file");
        String description = environment.getArgument("description");
        String storedFilePath = fileStorageService.store(file, description);
        return String.format("File stored at: %s, Description: %s", storedFilePath, description);
    }
}
```

当GraphQL框架调用此数据获取器的get方法时，它会从突变的参数中检索文件和可选描述。然后，它会调用FileStorageService来存储文件，并传递文件及其描述。

## 4. Spring Boot配置GraphQL上传支持

使用Spring Boot将文件上传集成到GraphQL API是一个多方面的过程，需要配置几个关键组件。

让我们根据我们的实现来定义配置：

```java
@Configuration
public class MultipartGraphQlWebMvcAutoconfiguration {

    private final FileUploadDataFetcher fileUploadDataFetcher;

    public MultipartGraphQlWebMvcAutoconfiguration(FileUploadDataFetcher fileUploadDataFetcher) {
        this.fileUploadDataFetcher = fileUploadDataFetcher;
    }

    @Bean
    public RuntimeWiringConfigurer runtimeWiringConfigurer() {
        return (builder) -> builder
                .type(newTypeWiring("Mutation").dataFetcher("uploadFile", fileUploadDataFetcher))
                .scalar(GraphQLScalarType.newScalar()
                        .name("Upload")
                        .coercing(new UploadCoercing())
                        .build());
    }

    @Bean
    @Order(1)
    public RouterFunction<ServerResponse> graphQlMultipartRouterFunction(
            GraphQlProperties properties,
            WebGraphQlHandler webGraphQlHandler,
            ObjectMapper objectMapper
    ) {
        String path = properties.getPath();
        RouterFunctions.Builder builder = RouterFunctions.route();
        MultipartGraphQlHttpHandler graphqlMultipartHandler = new MultipartGraphQlHttpHandler(webGraphQlHandler, new MappingJackson2HttpMessageConverter(objectMapper));
        builder = builder.POST(path, RequestPredicates.contentType(MULTIPART_FORM_DATA)
                .and(RequestPredicates.accept(SUPPORTED_MEDIA_TYPES.toArray(new MediaType[]{}))), graphqlMultipartHandler::handleMultipartRequest);
        return builder.build();
    }
}
```

**RuntimeWiringConfigurer在此设置中起着关键作用，使我们能够将GraphQL模式的操作(例如变更和查询)与相应的数据获取器链接起来**。此链接对于uploadFile变更至关重要，我们应用FileUploadDataFetcher来处理文件上传过程。

**此外，RuntimeWiringConfigurer有助于在GraphQL模式中定义和集成自定义Upload标量类型**。此标量类型与UploadCoercing相关联，使GraphQL API能够理解并正确处理文件数据，确保文件在上传过程中正确序列化和反序列化。

为了处理传入请求，特别是那些携带文件上传所需的multipart/form-data内容类型的请求，我们使用RouterFunction Bean定义。此函数擅长拦截这些特定类型的请求，使我们能够通过MultipartGraphQlHttpHandler处理它们。此处理程序是解析多部分请求、提取文件并将它们映射到GraphQL操作中的适当变量的关键，从而促进文件上传突变的执行。我们还使用@Order(1)注解应用正确的顺序。

## 5. 使用Postman测试文件上传

通过[Postman](https://www.baeldung.com/postman-testing-collections)测试GraphQL API中的文件上传功能需要采用非标准方法，因为内置的GraphQL有效负载格式不直接支持multipart/form-data请求，而这对于上传文件至关重要。相反，我们必须手动构建一个多部分请求，模仿客户端上传文件以及GraphQL变异的方式。

在Body选项卡中，应将选择设置为form-data。需要三个键值对：operations、map和具有根据map值命名的键名的文件变量。

对于operations键，其值应为封装GraphQL查询和变量的JSON对象，其中文件部分以null表示，作为占位符。此部分的类型仍为Text。

```json
{"query": "mutation UploadFile($file: Upload!, $description: String!) { uploadFile(file: $file, description: $description) }","variables": {"file": null,"description": "Sample file description"}}
```

接下来，map键需要一个值，该值是另一个JSON对象。这次，将文件变量映射到包含文件的表单字段。如果我们将文件附加到键0，则map会将此键与GraphQL变量中的文件变量明确关联，确保服务器正确解释表单数据的哪一部分包含该文件。此值也具有Text类型。

```json
{"0": ["variables.file"]}
```

最后，我们添加一个文件本身，其键与map对象中的引用相匹配。在我们的例子中，我们使用0作为此值的键。与之前的文本值不同，此部分的类型为File。

执行请求后，我们应该得到一个JSON响应：

```json
{
    "data": {
        "uploadFile": "File stored at: File uploaded successfully: C:\\Development\\TutorialsTuyucheng\\tutorials\\uploads\\2023-06-21_14-22.bmp with description: Sample file description, Description: Sample file description"
    }
}
```

## 6. 总结

在本文中，我们探讨了如何使用Spring Boot向GraphQL API添加文件上传功能。我们首先引入了一个名为Upload的自定义标量类型，它处理GraphQL突变中的文件数据。

然后，我们实现了MultipartGraphQlHttpHandler类来管理multipart/form-data请求，这是通过GraphQL突变上传文件所必需的。**与使用JSON的标准GraphQL请求不同，文件上传需要多部分请求来处理二进制文件数据**。

**FileUploadDataFetcher类处理uploadFile突变**，它提取并存储上传的文件，并向客户端发送有关文件上传状态的明确响应。

通常，使用纯HTTP请求进行文件上传并通过GraphQL查询传递结果ID会更有效。但是，有时直接使用GraphQL进行文件上传也是必要的。