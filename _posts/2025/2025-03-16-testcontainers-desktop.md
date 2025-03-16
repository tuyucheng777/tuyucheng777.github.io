---
layout: post
title:  Testcontainers Desktop
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将探索Testcontainers Desktop应用程序，这是一个运行[Testcontainers](https://www.baeldung.com/spring-boot-testcontainers-integration-test)的简单但功能强大的工具。我们将学习如何使用它来配置我们的[Docker环境](https://www.baeldung.com/ops/docker-guide)，管理容器生命周期，并深入了解我们的开发和测试模式。

## 2. Testcontainers Desktop

**Testcontainers Desktop提供了一个极简的UI，旨在简化Testcontainer的配置和调试**。我们可以从[官方网站](https://testcontainers.com/desktop/)免费下载Testcontainers Desktop。要开始使用它，我们将通过创建一个帐户或通过Google、GitHub或Docker等第三方进行注册。

就这样！一旦我们安装了应用程序并登录，我们就可以开始在开发工作流程中使用Testcontainers Desktop：

![](/assets/images/2025/springboot/testcontainersdesktop01.png)

我们应该在任务栏中看到Testcontainers徽标，如果我们右键单击它，我们将看到我们今天将探索的一些主要功能：

- 使用Testcontainers Cloud
- 冻结容器关闭
- 定义固定端口
- 与容器交互
- 查看Testcontainers仪表板
- 执行高级自定义

## 3. Testcontainers执行模式

Testcontainers Desktop为开发人员提供了两种运行测试的主要方式：本地或云端。值得注意的是，本地执行是默认行为。

### 3.1 本地执行

**本地执行利用了我们的本地Docker环境**。例如，让我们运行一个使用Testcontainers启动MongoDB Docker容器的JUnit测试：

```java
@Testcontainers
@SpringBootTest
class DynamicPropertiesLiveTest {

    @Container
    static MongoDBContainer mongoDBContainer = new MongoDBContainer(DockerImageName.parse("mongo:4.0.10"));

    @DynamicPropertySource
    static void setProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.mongodb.uri", mongoDBContainer::getReplicaSetUrl);
    }
   
    @Test
    void whenRequestingHobbits_thenReturnFrodoAndSam() {
        // ...
    }
}
```

如果我们本地还没有Docker镜像，我们将在日志中看到Docker正在拉取它。之后，MongoDB容器启动：

```text
org.testcontainers.dockerclient.DockerClientProviderStrategy - Found Docker environment with local Npipe socket (npipe:////./pipe/docker_engine)
org.testcontainers.DockerClientFactory - Docker host IP address is localhost
org.testcontainers.DockerClientFactory - Connected to docker:
    Server Version: 4.8.3
    API Version: 1.41
    Operating System: fedora
    Total Memory: 7871 MB
org.testcontainers.DockerClientFactory - Checking the system...
org.testcontainers.DockerClientFactory - ✔︎ Docker server version should be at least 1.6.0
tc.mongo:4.0.10 - Pulling docker image: mongo:4.0.10. Please be patient; this may take some time but only needs to be done once.
tc.mongo:4.0.10 - Starting to pull image
tc.mongo:4.0.10 - Pulling image layers:  1 pending,  1 downloaded,  0 extracted, (0 bytes/? MB)
tc.mongo:4.0.10 - Pulling image layers:  0 pending,  2 downloaded,  0 extracted, (0 bytes/0 bytes)
[ ... ]
tc.mongo:4.0.10 - Pull complete. 14 layers, pulled in 17s (downloaded 129 MB at 7 MB/s)
tc.mongo:4.0.10 - Creating container for image: mongo:4.0.10
tc.mongo:4.0.10 - Container mongo:4.0.10 is starting: 3d74c3a...
tc.mongo:4.0.10 - Container mongo:4.0.10 started in PT21.0624015S
```

此外，我们可以通过在终端中运行“docker ps”命令来手动检查容器是否创建。

### 3.2 Testcontainers云执行

**Testcontainers Cloud提供了一个可扩展的平台，用于在云环境中运行测试**。如果我们不想在本地运行容器，或者我们无法访问正在运行的Docker环境，那么这是理想的选择。

**TestContainer Cloud是Testcontainers的付费功能，但我们每月可以免费使用最多300分钟**。

从小UI中，让我们切换到“Run with Testcontainers Cloud”：

![](/assets/images/2025/springboot/testcontainersdesktop02.png)

让我们使用此选项重新运行测试，并再次观察日志：

```text
org.testcontainers.dockerclient.DockerClientProviderStrategy - Found Docker environment with Testcontainers Host with tc.host=tcp://127.0.0.1:65497
org.testcontainers.DockerClientFactory - Docker host IP address is 127.0.0.1
org.testcontainers.DockerClientFactory - Connected to docker:
    Server Version: 78+testcontainerscloud (via Testcontainers Desktop 1.7.0)
    API Version: 1.43
    Operating System: Ubuntu 20.04 LTS
    Total Memory: 7407 MB
org.testcontainers.DockerClientFactory - Checking the system...
org.testcontainers.DockerClientFactory - ✔︎ Docker server version should be at least 1.6.0
tc.mongo:4.0.10 - Pulling docker image: mongo:4.0.10. Please be patient; this may take some time but only needs to be done once.
tc.mongo:4.0.10 - Starting to pull image
tc.mongo:4.0.10 - Pulling image layers:  0 pending,  0 downloaded,  0 extracted, (0 bytes/0 bytes)
tc.mongo:4.0.10 - Pulling image layers: 12 pending,  1 downloaded,  0 extracted, (0 bytes/? MB)
[ ... ]
```

正如预期的那样，使用了不同的环境，Docker镜像不再下载到本地。不用说，如果我们运行命令“docker ps”，我们将看不到任何在本地运行的容器。

## 4. 调试测试容器

Testcontainers Desktop通过防止容器关闭、定义固定端口、自定义配置以满足我们的需求以及直接与容器交互等功能，实现了流畅的调试体验。

### 4.1 冻结容器关闭

**我们可以使用桌面应用程序手动控制容器的生命周期**。例如，我们可以使用选项“Freeze container shutdown”来允许正在运行的容器在启动它的测试终止后继续运行：

![](/assets/images/2025/springboot/testcontainersdesktop03.png)

如果我们启用此功能并重新运行测试，我们将收到确认容器已被冻结的通知。

接下来，我们将识别本地机器上与Docker容器的公开端口相对应的端口。MongoDB通常在端口27017上运行。让我们打开一个终端并运行命令“docker ps”来查看此映射：

![](/assets/images/2025/springboot/testcontainersdesktop04.png)

我们可以看到，Docker将容器的端口27017映射到我们机器上的端口64215。因此，我们可以使用此端口通过我们首选的MongoDB客户端应用程序连接到数据库。

[Studio3T](https://studio3t.com/)是MongoDB的图形用户界面，方便进行数据库管理、查询和可视化。让我们使用它来配置和测试与Mongo Testcontainer的连接：

![](/assets/images/2025/springboot/testcontainersdesktop05.png)

我们的测试从“test”数据库向“characters”集合中插入了一些记录，让我们运行一个简单的查询来列出集合中的所有记录：

![](/assets/images/2025/springboot/testcontainersdesktop06.png)

我们可以看到，所有记录都存在于数据库中。

### 4.2 定义固定端口

通常，Testcontainers会在随机端口上启动。**但是，如果我们经常需要找到公开的端口以进行调试，则可以定义固定端口**。为此，我们首先需要导航到Services > Open config location，这将打开一个文件夹，其中包含一些较受欢迎的Testconatiners模块的配置示例：

![](/assets/images/2025/springboot/testcontainersdesktop07.png)

让我们继续我们的用例并检查“mongodb.toml.example”文件。首先，我们将其重命名为“mongodb.toml”，删除“.example”扩展名。

现在，让我们检查一下文件的内容。注释逐步解释了如何自定义此文件以允许Testcontainers Desktop正确代理服务的端口。让我们关注“ports”变量，我们可以使用它来定义本地端口和容器端口之间的映射：

```text
# `local-port` configures on which port on your machine the service is exposed.
# `container-port` indicates which port inside the container to proxy.
ports = [
  {local-port = 27017, container-port = 27017},
]
```

因此，只需重命名文件并启用此配置，我们就能够使用固定端口27017连接到MongoDB数据库。

换句话说，**我们不再需要在每次重新运行测试时手动检查端口映射，而是可以依赖Mongo的默认端口**。

### 4.3 与容器交互

有时，即使连接到数据库也不够，例如当我们需要更详细的调试时。在这种情况下，我们可以直接访问Docker容器本身。例如，我们可以打开连接到容器的终端并与其交互。

为此，我们导航到Containers，然后选择要调试的容器。之后，系统将提示我们三个操作供选择：“Open terminal”、“Tail logs”或“Terminate”：

![](/assets/images/2025/springboot/testcontainersdesktop08.png)

“Open terminal”操作允许我们访问连接到容器的终端。例如，我们可以使用此终端启动MongoDB shell并查询数据，而无需在本地系统上安装任何MongoDB客户端应用程序。

让我们首先打开一个终端(Containers > mongo:4.0.10 > Open terminal)：

![](/assets/images/2025/springboot/testcontainersdesktop09.png)

从现在开始，指令取决于我们使用的容器和我们想要调试的用例。在我们的例子中，让我们执行以下命令：

- “mongo”：打开MongoDB shell提示符
- “show dbs”：列出服务器上现有的数据库
- “use test”：切换到“test”数据库，即我们的应用程序创建的数据库
-  “db.getCollection(“characters”).find({“race”:”hobbit”})”：查询“characters”集合并按“race”属性进行过滤

![](/assets/images/2025/springboot/testcontainersdesktop10.png)

正如预期的那样，我们可以看到使用MongoDB shell执行的命令。最后一个查询db.getCollection(...)从“test”数据库的“characters”集合中检索记录列表。

## 5. Testcontainers仪表板

**Testcontainers Desktop提供了一个用户友好的仪表板，其中包含我们使用的Testcontainers的摘要**。我们可以通过从菜单中选择“Open Dashboard...”选项来访问此网页：

![](/assets/images/2025/springboot/testcontainersdesktop11.png)

仪表板提供了所用测试容器和图像的概览，以及指向资源和帐户设置的有用链接。在页面底部，我们可以看到最近的活动和用于执行的环境。

**此协作工具可汇总桌面和CI环境中的测试数据，提供对开发和测试模式的洞察**。仪表板上的小部件可帮助解答有关测试一致性、发布影响、热门容器镜像和过时依赖项的问题。

## 6. 总结

在本文中，我们发现了Testcontainers Desktop应用程序的各种功能，这些功能可帮助我们运行和调试Testcontainers。我们探索了冻结容器关闭、使用固定端口以及访问连接到容器的终端。此外，我们还研究了Testcontainers Dashboard，这是一种可增强测试活动可见性和洞察力的工具。