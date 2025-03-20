---
layout: post
title:  在Docker容器中构建和运行Micronaut应用程序
category: micronaut
copyright: micronaut
excerpt: Micronaut
---

## 1. 简介

**在本教程中，我们将探讨如何在[Docker](https://www.baeldung.com/ops/docker-guide)容器中构建和运行[Micronaut](https://www.baeldung.com/micronaut)应用程序**，我们将讨论从设置Micronaut项目到创建和运行Docker化应用程序的所有内容。

Micronaut是一个基于JVM的现代框架，旨在构建模块化、易于测试的微服务和无服务器应用程序，同时占用最少的内存。Docker允许开发人员将应用程序打包到容器中，这些容器结合了必要的代码、库和依赖项，以便在不同环境中统一运行，从而简化了部署过程。

**Micronaut和Docker一起使用时，可以形成强大的组合，以高效开发和部署可扩展且有弹性的应用程序**。

## 2. 先决条件

为了获得在Docker容器中构建和运行Micronaut应用程序的有效指南，我们需要提前做好以下准备：

- **Java开发工具包(JDK)**：建议使用JDK 21以获得更好的性能并访问新的语言功能。Micronaut也支持JDK 17，但我们需要确保相应地更改版本标志。在本教程中，我们将使用JDK 21，因为它是Micronaut的默认版本。
- **Micronaut**：Micronaut安装是可选的，因为我们也可以在Web上创建项目。在本教程中，我们将学习如何使用CLI和Micronaut网站来执行此操作。
- **Gradle或Maven**：我们将使用Gradle和Maven自动创建Docker镜像，利用Micronaut的内置包装器而无需额外安装。
- **Docker**：应该在我们的机器上安装并配置[Docker](https://www.baeldung.com/ops/docker-install-windows-linux-mac)以方便容器化。

我们将专注于使用这些工具来构建装有Micronaut应用程序的Docker容器，而不是这些环境的初始设置。**我们应该确保已准备好无缝跟进**。

## 3. 设置Micronaut应用程序

**要开始设置我们的Micronaut应用程序，我们首先要创建一个新的Micronaut项目**。我们可以通过Micronaut的CLI、Micronaut Launch网站或兼容的IDE(如IntelliJ IDEA)来完成此操作。

### 3.1 使用CLI

要安装Micronaut CLI，我们将遵循[Micronauts官方文档](https://micronaut.io/download/)，它使用[SDKMAN](https://www.baeldung.com/java-sdkman-intro)。安装后，我们可以使用此命令来创建项目：

```bash
mn create-app cn.tuyucheng.taketoday.micronautdocker
```

这会在cn.tuyucheng.taketoday.micronautdocker包下创建一个基本的Micronaut应用程序脚手架。**默认情况下，Micronaut将项目设置为使用Java 21和[Gradle](https://www.baeldung.com/gradle)作为构建工具**。

要使用Java 17或Maven，我们可以包含适当的标志：

```bash
mn create-app cn.tuyucheng.taketoday.micronautdocker --jdk 17 --build maven
```

我们可以在[Micronaut的CLI指南](https://docs.micronaut.io/latest/guide/#cli)中找到其他标志和设置选项。

### 3.2 使用Micronaut Launch

如果我们想跳过Micronaut的安装，**我们可以使用他们的[Micronaut Launch Website](https://micronaut.io/launch)创建一个新项目**。与CLI不同，该网站允许我们下载一个包含所有必要文件的文件夹，无需任何安装即可开始工作。

### 3.3 检查环境

一旦我们成功创建了项目，我们可以通过启动应用程序来检查一切是否正常。

以下是使用Gradle启动应用程序的命令：

```bash
./gradlew run
```

如果我们使用Maven，我们可以通过调用mn:run目标来启动应用程序：

```bash
./mvnw mn:run
```

我们可以通过在浏览器中导航到http://localhost:8080或使用[curl](https://www.baeldung.com/curl-rest)之类的工具来验证应用程序是否正在运行。

**此步骤确保所有内容均已正确配置，并且我们的Micronaut应用程序已正确设置**。验证后，我们就可以继续并开始对我们的应用程序进行Docker化。

## 4. Docker化应用程序

在我们的Micronaut应用程序设置完毕并正确运行后，下一步就是准备进行容器化。

要创建自动生成的Docker镜像，我们需要确保我们的构建工具具有一些配置以使其正常工作。Gradle和Maven都内置了对创建Docker镜像的支持，不过，如果我们想修改一些琐碎的Docker功能，我们需要进行一些设置。

**在本节中，我们将更改基础镜像和公开的端口、添加自定义指令并更改输出Docker镜像名称**。

### 4.1 使用Gradle进行Docker化

为了在Dockerfile中进行修改，我们需要捕获Gradle任务并添加自定义设置：

```kotlin
tasks.named<MicronautDockerfile>("dockerfile") {
    baseImage.set("eclipse-temurin:21-jre-alpine")
    exposedPorts.set(arrayListOf(9090))
    instruction("RUN echo 'Hello world!'")
}
```

在此代码中，我们使用baseImage.set()来更改基础镜像的名称，使用exposedPorts.set()来更改Docker将要公开的端口，并使用indication()来添加自定义echo指令。

**如果我们想更改输出的Docker镜像名称，我们将使用**：

```kotlin
tasks.named<DockerBuildImage>("dockerBuild") {
    images.set(listOf("micronaut_docker_gradle"))
}
```

这将捕获dockerBuild任务并将Docker镜像名称设置为micronaut_docker_gradle。**要了解有关从Gradle修改Dockerfile的更多信息，我们可以转到官方的[Micronaut Gradle Plugin Docker支持文档](https://micronaut-projects.github.io/micronaut-gradle-plugin/latest/#_docker_support)**。

### 4.2 使用Maven进行Docker化

由于Micronaut Maven插件没有内置创建Docker镜像的支持，**我们将使用Google Cloud Tools中的[Jib](https://www.baeldung.com/jib-dockerizing)**。Jib是一个流行的插件，用于直接从Maven构建优化的Docker镜像，而无需单独的Dockerfile。

**首先，我们需要在properties部分设置一些变量**：

```xml
<properties>
    <!-- ... -->
    <docker.baseImage>eclipse-temurin:21-jre-alpine</docker.baseImage>
    <docker.exposedPort>9090</docker.exposedPort>
    <docker.imageName>micronaut_docker_maven</docker.imageName>
</properties>
```

接下来，我们配置jib dockerBuild流程。**为此，我们将修改build标签中的plugins部分**：

```xml
<build>
    <plugins>
        <!-- ... -->
        <plugin>
            <groupId>com.google.cloud.tools</groupId>
            <artifactId>jib-maven-plugin</artifactId>
            <version>3.3.1</version>
            <configuration>
                <from>
                    <image>${docker.baseImage}</image>
                </from>
                <container>
                    <ports>
                        <port>${docker.exposedPort}</port>
                    </ports>
                    <entrypoint>
                        <shell>sh</shell>
                        <option>-c</option>
                        <arg>
                            java -cp /app/resources:/app/classes:/app/libs/* ${exec.mainClass}
                        </arg>
                    </entrypoint>
                </container>
                <to>
                    <image>${docker.imageName}</image>
                </to>
            </configuration>
        </plugin>
    </plugins>
</build>
```

正如演示的那样，jib不支持直接在Dockerfile中写入自定义指令，我们可以参考官方的[Micronaut Maven插件打包文档](https://micronaut-projects.github.io/micronaut-maven-plugin/latest/examples/package.html)。

## 5. 构建Docker镜像

应用程序打包并准备就绪后，我们将重点介绍如何构建、检查和运行我们的Docker镜像。

以下是使用Gradle构建镜像的命令：

```shell
./gradlew dockerBuild
```

如果我们使用Maven，我们可以通过调用jib:dockerBuild目标来启动应用程序：

```shell
./mvnw jib:dockerBuild
```

运行dockerBuild命令后，我们可以通过列出本地Docker镜像来确认镜像是否已创建：

```shell
docker images
```

运行这些命令后，我们应该看到名为micronaut_docker_gradle或micronaut_docker_maven的Docker镜像，**这证实我们的Docker镜像已准备好部署**。

## 6. 运行容器

Docker镜像构建完成后，下一步就是将其作为容器运行。

以下是使用Gradle启动容器的命令：

```shell
docker run -p 9090:9090 micronaut_docker_gradle:latest
```

如果使用Maven，我们可以通过运行为Maven创建的镜像来启动容器：

```shell
docker run -p 9090:9090 micronaut_docker_maven:latest
```

此命令将容器的端口9090映射到主机的端口9090，从而**允许我们通过http://localhost:9090本地访问该应用程序**。

如果容器启动成功，我们应该看到表明应用程序正在运行的日志。**要验证容器是否处于活动状态，我们应该运行**：

```shell
docker ps
```

这将显示正在运行的容器列表，包括它们的ID、状态和端口映射。我们将查找名称为micronaut_docker_gradle或 micronaut_docker_maven的任何条目以获取更多信息。

**如果我们需要进行一些调试**，我们可以使用以下命令：

```shell
docker logs <container_id>
```

运行容器可确保我们的Micronaut应用程序现已在Docker化环境中完全部署并运行。

## 7. 总结

在本文中，我们介绍了在Docker容器中构建和运行Micronaut应用程序的过程。我们首先使用CLI和Micronaut Launch设置Micronaut项目，然后探索如何配置Gradle和Maven以自动创建Docker镜像。我们还演示了如何自定义Docker镜像以满足特定要求，包括修改基础镜像、公开端口和命名约定。最后，我们构建并运行了Docker容器，确保应用程序可访问且完全可运行。

通过将Micronaut的灵活性与Docker的可移植性相结合，我们可以简化可扩展轻量级应用程序的开发和部署。这种集成不仅简化了工作流程，而且还确保了跨环境的一致性，使其成为现代微服务开发的宝贵方法。