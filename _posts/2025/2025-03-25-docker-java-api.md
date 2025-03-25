---
layout: post
title:  Java版Docker指南
category: libraries
copyright: libraries
excerpt: Docker Java
---

## 1. 概述

在本文中，我们将介绍另一个成熟的平台特定API-[Docker的Java API客户端](https://github.com/docker-java/docker-java)。

通过整篇文章，我们介绍了如何连接正在运行的Docker守护程序以及API为Java开发人员提供了哪些重要的功能。

## 2. Maven依赖

首先，我们需要将主要依赖添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>com.github.docker-java</groupId>
    <artifactId>docker-java</artifactId>
    <version>3.0.14</version>
</dependency>
```

在撰写本文时，**API的最新版本是3.0.14**，每个版本都可以从[GitHub发布页面](https://github.com/docker-java/docker-java/releases)或[Maven仓库](https://mvnrepository.com/artifact/com.github.docker-java/docker-java)查看。

## 3. 使用Docker客户端

Docker Client是我们可以在Docker引擎/守护进程和我们的应用程序之间建立连接的地方。

默认情况下，Docker守护进程只能通过unix:///var/run/docker.sock文件访问。除非另有配置，否则我们**可以本地与在Unix套接字上监听的Docker引擎进行通信**。

这里，我们应用Docker ClientBuilder类来接收默认设置来创建连接：

```java
DockerClient dockerClient = DockerClientBuilder.getInstance().build();
```

类似地，我们可以分两步打开一个连接：

```java
DefaultDockerClientConfig.Builder config = DefaultDockerClientConfig.createDefaultConfigBuilder();
DockerClient dockerClient = DockerClientBuilder
    .getInstance(config)
    .build();
```

由于引擎可以依赖于其他特性，因此客户端也可以根据不同的条件进行配置。

例如，构建器接受服务器URL，也就是说，**如果引擎在端口2375上可用，我们可以更新连接值**：

```java
DockerClient dockerClient = DockerClientBuilder.getInstance("tcp://docker.tuyucheng.com:2375").build();
```

请注意，**我们需要根据连接类型在连接字符串前面添加unix://或tcp://**。

如果我们更进一步，我们可以使用DefaultDockerClientConfig类获得更高级的配置：

```java
DefaultDockerClientConfig config = DefaultDockerClientConfig.createDefaultConfigBuilder()
    .withRegistryEmail("info@tuyucheng.com")
    .withRegistryPassword("tuyucheng")
    .withRegistryUsername("tuyucheng")
    .withDockerCertPath("/home/tuyucheng/.docker/certs")
    .withDockerConfig("/home/tuyucheng/.docker/")
    .withDockerTlsVerify("1")
    .withDockerHost("tcp://docker.tuyucheng.com:2376").build();

DockerClient dockerClient = DockerClientBuilder.getInstance(config).build();
```

同样，我们可以使用Properties执行相同的方法：

```java
Properties properties = new Properties();
properties.setProperty("registry.email", "info@tuyucheng.com");
properties.setProperty("registry.password", "tuyucheng");
properties.setProperty("registry.username", "tuyucheng");
properties.setProperty("DOCKER_CERT_PATH", "/home/tuyucheng/.docker/certs");
properties.setProperty("DOCKER_CONFIG", "/home/tuyucheng/.docker/");
properties.setProperty("DOCKER_TLS_VERIFY", "1");
properties.setProperty("DOCKER_HOST", "tcp://docker.tuyucheng.com:2376");

DefaultDockerClientConfig config = DefaultDockerClientConfig.createDefaultConfigBuilder()
    .withProperties(properties).build();

DockerClient dockerClient = DockerClientBuilder.getInstance(config).build();
```

除非我们在源代码中配置引擎的设置，否则另一种选择是设置相应的环境变量，这样我们只能考虑项目中DockerClient的默认实例：

```shell
export DOCKER_CERT_PATH=/home/tuyucheng/.docker/certs
export DOCKER_CONFIG=/home/tuyucheng/.docker/
export DOCKER_TLS_VERIFY=1
export DOCKER_HOST=tcp://docker.tuyucheng.com:2376
```

## 4. 容器管理

API允许我们对容器管理进行多种选择，让我们逐一了解一下。

### 4.1 列出容器

现在我们已经建立了连接，我们可以列出Docker主机上所有正在运行的容器：

```java
List<Container> containers = dockerClient.listContainersCmd().exec();
```

如果显示正在运行的容器不符合我们的需要，我们可以利用提供的选项来查询容器。

在这种情况下，我们显示处于“exited”状态的容器：

```java
List<Container> containers = dockerClient.listContainersCmd()
    .withShowSize(true)
    .withShowAll(true)
    .withStatusFilter("exited").exec()
```

它相当于：

```shell
$ docker ps -a -s -f status=exited
# or 
$ docker container ls -a -s -f status=exited
```

### 4.2 创建容器

创建容器由createContainerCmd方法提供，我们可以**使用以“with”前缀开头的可用方法进行更复杂的声明**。

假设我们有一个docker create命令，定义一个主机相关的MongoDB容器，在端口27017上进行内部监听：

```shell
$ docker create --name mongo \
  --hostname=tuyucheng \
  -e MONGO_LATEST_VERSION=3.6 \
  -p 9999:27017 \
  -v /Users/tuyucheng/mongo/data/db:/data/db \
  mongo:3.6 --bind_ip_all
```

我们能够以编程方式引导同一个容器及其配置：

```java
CreateContainerResponse container = dockerClient.createContainerCmd("mongo:3.6")
    .withCmd("--bind_ip_all")
    .withName("mongo")
    .withHostName("tuyucheng")
    .withEnv("MONGO_LATEST_VERSION=3.6")
    .withPortBindings(PortBinding.parse("9999:27017"))
    .withBinds(Bind.parse("/Users/tuyucheng/mongo/data/db:/data/db")).exec();
```

### 4.3 启动、停止和终止容器

一旦我们创建了容器，我们就可以分别通过名称或id来启动、停止和终止它：

```java
dockerClient.startContainerCmd(container.getId()).exec();

dockerClient.stopContainerCmd(container.getId()).exec();

dockerClient.killContainerCmd(container.getId()).exec();
```

### 4.4 检查容器

inspectContainerCmd方法接收一个String参数，该参数表示容器的名称或id。使用此方法，我们可以直接观察容器的元数据：

```java
InspectContainerResponse container = dockerClient.inspectContainerCmd(container.getId()).exec();
```

### 4.5 快照容器

与docker commit命令类似，我们可以使用commitCmd方法创建一个新的镜像。

在我们的示例中，场景是，**我们之前运行一个alpine:3.6容器，其id为“3464bb547f88”，并在其上安装了git**。

现在，我们要从容器创建一个新的镜像快照：

```java
String snapshotId = dockerClient.commitCmd("3464bb547f88")
    .withAuthor("Tuyucheng <info@tuyucheng.com>")
    .withEnv("SNAPSHOT_YEAR=2018")
    .withMessage("add git support")
    .withCmd("git", "version")
    .withRepository("alpine")
    .withTag("3.6.git").exec();
```

由于我们用git捆绑的新镜像保留在主机上，因此我们可以在Docker主机上搜索它：

```shell
$ docker image ls alpine --format "table {{.Repository}} {{.Tag}}"
REPOSITORY TAG
alpine     3.6.git
```

## 5. 镜像管理

我们提供了一些适用的命令来管理镜像操作。

### 5.1 列出镜像

要列出Docker主机上所有可用的镜像(包括悬空镜像)，我们需要应用listImagesCmd方法：

```java
List<Image> images = dockerClient.listImagesCmd().exec();
```

如果我们的Docker Host上有两个镜像，**我们应该在运行时获取它们的Image对象**。我们要查找的镜像是：

```shell
$ docker image ls --format "table {{.Repository}} {{.Tag}}"
REPOSITORY TAG
alpine     3.6
mongo      3.6
```

接下来，为了查看中间镜像，我们需要明确请求它：

```java
List<Image> images = dockerClient.listImagesCmd()
    .withShowAll(true).exec();
```

如果只显示悬挂的镜像，则必须考虑使用withDanglingFilter方法：

```java
List<Image> images = dockerClient.listImagesCmd()
    .withDanglingFilter(true).exec();
```

### 5.2 构建镜像

让我们关注使用API构建镜像的方式，**buildImageCmd方法从Dockerfile构建Docker镜像**。在我们的项目中，我们已经有一个Dockerfile，它提供了一个安装了git的Alpine镜像：

```dockerfile
FROM alpine:3.6

RUN apk --update add git openssh && \
  rm -rf /var/lib/apt/lists/* && \
  rm /var/cache/apk/*

ENTRYPOINT ["git"]
CMD ["--help"]
```

新镜像的构建不会使用缓存，在开始构建过程之前，Docker引擎将尝试提取较新版本的alpine:3.6。**如果一切顺利，我们最终应该会看到具有给定名称的镜像alpine:git**：

```java
String imageId = dockerClient.buildImageCmd()
    .withDockerfile(new File("path/to/Dockerfile"))
    .withPull(true)
    .withNoCache(true)
    .withTag("alpine:git")
    .exec(new BuildImageResultCallback())
    .awaitImageId();
```

### 5.3 检查镜像

我们可以通过inspectImageCmd方法检查有关镜像的低级信息：

```java
InspectImageResponse image = dockerClient.inspectImageCmd("161714540c41").exec();
```

### 5.4 标记镜像

使用docker tag命令为镜像添加标签非常简单，因此API也不例外。我们也可以使用tagImageCmd方法实现相同的目的。**要使用git将id为161714540c41的Docker镜像标记到tuyucheng/alpine仓库中**：

```java
String imageId = "161714540c41";
String repository = "tuyucheng/alpine";
String tag = "git";

dockerClient.tagImageCmd(imageId, repository, tag).exec();
```

我们将列出新创建的镜像，它就在那里：

```shell
$ docker image ls --format "table {{.Repository}} {{.Tag}}"
REPOSITORY      TAG
baeldung/alpine git
```

### 5.5 推送镜像

在将镜像发送到注册中心服务之前，必须配置Docker客户端与该服务配合，因为使用注册中心需要提前进行身份验证。

由于我们假设客户端配置了Docker Hub，我们可以将tuyucheng/alpine镜像推送到tuyucheng DockerHub帐户：

```java
dockerClient.pushImageCmd("tuyucheng/alpine")
    .withTag("git")
    .exec(new PushImageResultCallback())
    .awaitCompletion(90, TimeUnit.SECONDS);
```

**我们必须遵守该过程的持续时间。在示例中，我们等待90秒**。

### 5.6 拉取镜像

要从注册中心服务下载镜像，我们使用pullImageCmd方法。此外，如果从私有注册表中提取镜像，客户端必须知道我们的凭证，否则该过程将失败。**与提取镜像相同，我们指定回调以及提取镜像的固定时间段**：

```java
dockerClient.pullImageCmd("tuyucheng/alpine")
    .withTag("git")
    .exec(new PullImageResultCallback())
    .awaitCompletion(30, TimeUnit.SECONDS);
```

拉取镜像后检查该镜像是否存在于Docker主机上：

```shell
$ docker images baeldung/alpine --format "table {{.Repository}} {{.Tag}}"
REPOSITORY      TAG
tuyucheng/alpine git
```

### 5.7 删除镜像

其余函数中另一个简单的函数是removeImageCmd方法，我们可以通过其短ID或长ID删除镜像：

```java
dockerClient.removeImageCmd("beaccc8687ae").exec();
```

### 5.8 在注册表中搜索

为了从Docker Hub搜索镜像，客户端附带了searchImagesCmd方法，该方法接收表示术语的字符串值。在这里，我们在Docker Hub中探索与名称包含“Java”相关的镜像：

```java
List<SearchItem> items = dockerClient.searchImagesCmd("Java").exec();
```

输出返回SearchItem对象列表中的前25个相关镜像。

## 6. 卷管理

如果Java项目需要与Docker进行卷交互，我们也应该考虑本节。简要地，我们看一下Docker Java API提供的卷的基本技术。

### 6.1 列出卷

所有可用的卷(包括命名和未命名的卷)都列出：

```java
ListVolumesResponse volumesResponse = dockerClient.listVolumesCmd().exec();
List<InspectVolumeResponse> volumes = volumesResponse.getVolumes();
```

### 6.2 检查卷

inspectVolumeCmd方法是显示卷详细信息的形式。我们通过指定其短ID来检查卷：

```java
InspectVolumeResponse volume = dockerClient.inspectVolumeCmd("0220b87330af5").exec();
```

### 6.3 创建卷

API提供两种不同的选项来创建卷，无参数createVolumeCmd方法创建一个卷，其名称由Docker指定：

```java
CreateVolumeResponse unnamedVolume = dockerClient.createVolumeCmd().exec();
```

与使用默认行为不同，名为withName的辅助方法允许我们为卷设置名称：

```java
CreateVolumeResponse namedVolume = dockerClient.createVolumeCmd().withName("myNamedVolume").exec();
```

### 6.4 删除卷

我们可以使用removeVolumeCmd方法直观地从Docker主机中删除卷。**需要注意的是，如果卷正在容器中使用，则无法删除它**。我们从卷列表中删除卷myNamedVolume：

```java
dockerClient.removeVolumeCmd("myNamedVolume").exec();
```

## 7. 网络管理

我们的最后一部分是关于使用API管理网络任务。

### 7.1 列出网络

我们可以使用以list开头的常规API方法之一来显示网络单元列表：

```java
List<Network> networks = dockerClient.listNetworksCmd().exec();
```

### 7.2 创建网络

docker network create命令的等效操作由createNetworkCmd方法执行，如果我们有第三方或自定义网络驱动程序，withDriver方法除了可以接收内置驱动程序外，还可以接收它们。在我们的例子中，让我们创建一个名为tuyucheng的桥接网络：

```java
CreateNetworkResponse networkResponse = dockerClient.createNetworkCmd()
    .withName("tuyucheng")
    .withDriver("bridge").exec();
```

此外，使用默认设置创建网络单元并不能解决问题，我们可以应用其他辅助方法来构建高级网络。因此，**要用自定义值覆盖默认子网**：

```java
CreateNetworkResponse networkResponse = dockerClient.createNetworkCmd()
    .withName("tuyucheng")
    .withIpam(new Ipam()
        .withConfig(new Config()
        .withSubnet("172.36.0.0/16")
        .withIpRange("172.36.5.0/24")))
    .withDriver("bridge").exec();
```

我们可以使用docker命令运行相同的命令：

```shell
$ docker network create \
  --subnet=172.36.0.0/16 \
  --ip-range=172.36.5.0/24 \
  tuyucheng
```

### 7.3 检查网络

API中还涵盖了显示网络的低级细节：

```java
Network network = dockerClient.inspectNetworkCmd().withNetworkId("tuyucheng").exec();
```

### 7.4 删除网络

我们可以使用removeNetworkCmd方法通过其名称或id安全地删除网络单元：

```java
dockerClient.removeNetworkCmd("tuyucheng").exec();
```

## 8. 总结

在本详尽教程中，我们探索了Java Docker API客户端的各种不同功能，以及部署和管理场景的几种实现方法。