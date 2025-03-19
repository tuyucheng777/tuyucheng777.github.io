---
layout: post
title:  如何Mock Amazon S3进行集成测试
category: mock
copyright: mock
excerpt: S3Mock
---

## 1. 简介

在本文中，我们将学习如何Mock [Amazon S3](https://www.baeldung.com/java-aws-s3)(简单存储服务)来运行Java应用程序的集成测试。

为了演示其工作原理，我们将创建一个使用AWS SDK与S3交互的CRUD服务。然后，我们将使用Mock的S3服务为每个操作编写集成测试。

## 2. S3概述

Amazon Simple Storage Service(S3)是AWS提供的一种高度可扩展且安全的云存储服务，**它使用对象存储模型，允许用户在网络上的任何位置存储和检索数据**。

该服务可通过REST风格的API访问，AWS为Java应用程序提供了SDK来执行创建、列出和删除S3存储桶和对象等操作。

接下来，我们开始使用AWS SDK为S3创建Java CRUD服务，并实现创建、读取、更新和删除操作。

## 3. 演示S3 CRUD Java服务

在开始使用S3之前，我们需要在项目中添加对AWS SDK的依赖：

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.20.52</version>
</dependency>
```

要查看最新版本，可以检查[中央仓库](https://mvnrepository.com/artifact/software.amazon.awssdk/s3)。

接下来，我们创建S3CrudService类，并以software.amazon.awssdk.services.s3.S3Client作为依赖项：

```java
class S3CrudService {
    private final S3Client s3Client;

    public S3CrudService(S3Client s3Client) {
        this.s3Client = s3Client;
    }

    // ...
}
```

现在我们已经创建了Service，**让我们使用AWS SDK提供的S3Client API来实现createBucket()、createObject()、getObject()和deleteObject()操作**：

```java
void createBucket(String bucketName) {
    // build bucketRequest
    s3Client.createBucket(bucketRequest);
}

void createObject(String bucketName, File inMemoryObject) {
    // build putObjectRequest
    s3Client.putObject(request, RequestBody.fromByteBuffer(inMemoryObject.getContent()));
}

Optional<byte[]> getObject(String bucketName, String objectKey) {
    try {
        // build getObjectRequest
        ResponseBytes<GetObjectResponse> responseResponseBytes = s3Client.getObjectAsBytes(getObjectRequest);
        return Optional.of(responseResponseBytes.asByteArray());
    } catch (S3Exception e) {
        return Optional.empty();
    }
}

boolean deleteObject(String bucketName, String objectKey) {
    try {
        // build deleteObjectRequest
        s3Client.deleteObject(deleteObjectRequest);
        return true;
    } catch (S3Exception e) {
        return false;
    }
}
```

现在我们已经创建了S3操作，让我们学习如何使用Mock S3服务来实现集成测试。

## 4. 使用S3Mock库进行集成测试

在本教程中，**我们选择使用Adobe开源的[S3Mock](https://github.com/adobe/S3Mock)库**。S3Mock是一个轻量级服务器，可实现Amazon S3 API最常用的操作。对于支持的S3操作，我们可以查看S3Mock仓库的README文件。

库开发人员建议隔离运行S3Mock服务，最好使用提供的Docker容器。

按照建议，**让我们使用[Docker和Testcontainers](https://www.baeldung.com/docker-test-containers)运行S3Mock服务进行集成测试**。

### 4.1 依赖

接下来，让我们添加必要的依赖项以便将S3Mock与Testcontainers一起运行：

```xml
<dependency>
    <groupId>com.adobe.testing</groupId>
    <artifactId>s3mock</artifactId>
    <version>3.3.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.adobe.testing</groupId>
    <artifactId>s3mock-testcontainers</artifactId>
    <version>3.3.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.4</version>
    <scope>test</scope>
</dependency>
```

我们可以检查Maven Central上的[s3mock](https://mvnrepository.com/artifact/com.adobe.testing/s3mock)，[s3mock-testcontainers](https://mvnrepository.com/artifact/com.adobe.testing/s3mock-testcontainers)，[junit-jupiter](https://mvnrepository.com/artifact/org.testcontainers/junit-jupiter)链接来查看最新版本。

### 4.2 设置

**作为先决条件，我们必须有一个正在运行的Docker环境以确保可以启动测试容器**。

当我们在集成测试类上使用@TestConainers和@Container注解时，S3MockContainer的最新Docker镜像将从注册表中提取并在本地Docker环境中启动：

```java
@Testcontainers
class S3CrudServiceIntegrationTest {
    @Container
    private  S3MockContainer s3Mock = new S3MockContainer("latest");
}
```

在运行集成测试之前，让我们在@BeforeEach生命周期方法中创建一个S3Client实例：

```java
@BeforeEach
void setUp() {
    var endpoint = s3Mock.getHttpsEndpoint();
    var serviceConfig = S3Configuration.builder()
            .pathStyleAccessEnabled(true)
            .build();
    var httpClient = UrlConnectionHttpClient.builder()
            .buildWithDefaults(AttributeMap.builder()
                    .put(TRUST_ALL_CERTIFICATES, Boolean.TRUE)
                    .build());
    s3Client = S3Client.builder()
            .endpointOverride(URI.create(endpoint))
            .serviceConfiguration(serviceConfig)
            .httpClient(httpClient)
            .build();
}
```

在setup()方法中，我们使用S3Client接口提供的构建器初始化了S3Client的实例。在此初始化过程中，我们为以下参数指定了配置：

- **endOverwrite**：此参数配置用于定义S3 Mock服务的地址
- **pathStyleAccessEnabled**：我们在服务配置中将此参数设置为true
- **TRUST_ALL_CERTIFICATES**：另外，我们配置了一个httpClient实例，其中所有证书均受信任，通过将TRUST_ALL_CERTIFICATES设置为true来指示

### 4.3 为S3CrudService编写集成测试

当我们完成基础设施设置后，让我们为S3CrudService操作编写一些集成测试。

首先，让我们创建一个存储桶并验证其是否创建成功：

```java
var s3CrudService = new S3CrudService(s3Client);
s3CrudService.createBucket(TEST_BUCKET_NAME);

var createdBucketName = s3Client.listBuckets().buckets().get(0).name();
assertThat(TEST_BUCKET_NAME).isEqualTo(createdBucketName);
```

成功创建存储桶后，让我们在S3中上传一个新对象。

为此，首先，我们使用FileGenerator生成一个字节数组，然后createObject()方法将其作为对象保存在已创建的存储桶中：

```java
var fileToSave = FileGenerator.generateFiles(1, 100).get(0);
s3CrudService.createObject(TEST_BUCKET_NAME, fileToSave);
```

接下来，让我们使用已保存文件的文件名调用getObject()方法来确认该对象是否确实保存在S3中：

```java
var savedFileContent = s3CrudService.getObject(TEST_BUCKET_NAME, fileToSave.getName());
assertThat(Arrays.equals(fileToSave.getContent().array(), savedFileContent)).isTrue();
```

最后，让我们测试一下deleteObject()是否也能按预期工作。首先，我们使用存储桶名称和目标文件名调用deleteObject()方法。随后，我们再次调用getObject()并检查结果是否为空：

```java
s3CrudService.deleteObject(TEST_BUCKET_NAME,fileToSave.getName());

var deletedFileContent = s3CrudService.getObject(TEST_BUCKET_NAME, fileToSave.getName());
assertThat(deletedFileContent).isEmpty();
```

## 5. 总结

在本教程中，**我们学习了如何使用S3Mock库Mock真实的S3服务来编写依赖于AWS S3服务的集成测试**。

为了演示这一点，我们首先实现了一个基本的CRUD服务，用于从S3创建、读取和删除对象。然后，我们使用S3Mock库实现了集成测试。