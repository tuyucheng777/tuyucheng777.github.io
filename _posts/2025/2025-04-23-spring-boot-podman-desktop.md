---
layout: post
title:  使用Podman Desktop对Spring Boot应用程序进行容器化
category: springboot
copyright: springboot
excerpt: Podman
---

## 1. 概述

在本教程中，我们将学习如何使用Podman Desktop对Spring Boot应用程序进行容器化，[Podman](https://www.baeldung.com/ops/podman-intro)是一个容器化工具，它允许我们无需守护进程即可管理容器。

**Podman Desktop是一个具有图形用户界面的桌面应用程序，用于使用Podman管理容器**。

为了演示它的用法，我们将创建一个简单的Spring Boot应用程序，构建一个容器镜像，并使用Podman Desktop运行一个容器。

## 2. 安装Podman Desktop

首先，我们需要在本地机器上[安装Podman Desktop](https://podman-desktop.io/docs/installation)才能开始使用，它适用于Windows、macOS和Linux操作系统。下载安装程序后，我们可以按照安装说明在我们的机器上设置Podman Desktop。

以下是设置Podman Desktop的几个重要步骤：

- Podman应该安装在机器上，如果没有安装，Podman Desktop会提示并为我们安装。
- Podman准备就绪后，系统将提示我们启动Podman机器，我们可以选择默认设置或根据需要自定义设置，这是运行容器之前所必需的。
- 此外，对于Windows，我们需要先[启用/安装WSL2(适用于Linux的Windows子系统)](https://learn.microsoft.com/en-us/windows/wsl/install)，然后才能运行Podman。

**在安装过程结束时，我们应该有一个正在运行的Podman机器，可以使用Podman Desktop应用程序进行管理，我们可以在Dashboard部分验证这一点**：

![](/assets/images/2025/springboot/springbootpodmandesktop01.png)

## 3. 创建Spring Boot应用程序

让我们创建一个小型Spring Boot应用程序，该应用程序将有一个[REST控制器](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration#controller)，当我们访问/hello端点时，它会返回“Hello,World!”消息。

我们将使用Maven构建项目并创建一个[jar文件](https://www.baeldung.com/java-create-jar)，然后，我们将创建一个Containerfile(在Docker上下文中也称为Dockerfile)，我们将使用此文件使用Podman Desktop为我们的应用程序构建容器镜像。

### 3.1 设置项目

首先，我们将[Spring Boot Starter Web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)依赖添加到我们的项目中：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.3.2</version>
    </dependency>
</dependencies>
```

此依赖提供了创建Spring Boot Web应用程序所需的库。

### 3.2 控制器

接下来，让我们创建REST控制器：
```java
@RestController
public class HelloWorldController {
    @GetMapping("/hello")
    public String helloWorld() {
        return "Hello, World!";
    }
}
```

这里我们使用@RestController注解将类标记为控制器，并使用@GetMapping注解将方法映射到/hello端点，当我们访问此端点时，它会返回“Hello,World!”消息。

### 3.3 构建项目

我们可以通过在终端中运行以下命令来使用Maven构建项目：
```shell
mvn clean package
```

此命令编译项目、运行测试并在target目录中创建jar文件。

## 4. Containerfile

现在我们已经准备好了Spring Boot应用程序，让我们创建一个Containerfile来为我们的应用程序构建镜像，我们将在项目的根目录中创建此文件：
```Dockerfile
FROM openjdk:17-alpine
WORKDIR /app
COPY target/spring-boot-podman-desktop-1.0.0.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

在此文件中：

- 首先，我们使用openjdk:17-alpine镜像作为基础镜像
- 接下来，我们将工作目录设置为/app
- 然后，我们将Maven生成的jar文件复制到/app目录
- 我们公开端口8080，这是Spring Boot应用程序的默认端口
- 最后，我们使用CMD命令指定我们要在容器启动时使用java-jar命令运行jar文件

## 5. 使用Podman Desktop构建镜像

设置Containerfile后，让我们使用Podman Desktop应用程序来创建镜像。

**首先，我们转到“Images”部分并单击“Build”按钮**：

![](/assets/images/2025/springboot/springbootpodmandesktop02.png)

接下来，我们将填写镜像的详细信息：

- 设置镜像的名称
- 选择Containerfile位置
- 使用项目目录作为上下文目录
- 我们还可以选择镜像的平台，这里使用默认值

以下是参数和值的示例：

![](/assets/images/2025/springboot/springbootpodmandesktop03.png)

填写完详细信息后，我们可以点击Build按钮来构建镜像，构建完成后，我们可以在Images部分找到该镜像。

## 6. 运行容器

现在我们已经准备好了镜像，我们可以使用该镜像运行容器。可以通过单击“Images”部分中hello-world-demo镜像旁边的“Run”按钮来执行此操作：

![](/assets/images/2025/springboot/springbootpodmandesktop04.png)

### 6.1 启动容器

接下来，我们将填写容器的详细信息，我们在Containerfile中设置的属性将被预先填充，我们可以根据需要自定义它们：

![](/assets/images/2025/springboot/springbootpodmandesktop05.png)

此时，端口映射和命令已经填好，如果需要，我们可以设置其他属性，例如环境变量、卷等，我们还可以设置容器的名称。

填写详细信息后，我们可以单击“Start Container”按钮来启动容器，这将打开“Container Details”部分并显示容器日志：

![](/assets/images/2025/springboot/springbootpodmandesktop06.png)

### 6.2 测试应用程序

容器启动后，我们可以通过打开浏览器并导航到http://localhost:8080/hello来访问该应用程序，我们将在页面上看到“Hello,World!”消息：

![](/assets/images/2025/springboot/springbootpodmandesktop07.png)

### 6.3 停止容器

要停止容器，我们单击上面Container Details部分中的Stop按钮。

或者，我们可以转到容器列表并单击该容器的Stop按钮：

![](/assets/images/2025/springboot/springbootpodmandesktop08.png)

## 7. 总结

在本文中，我们学习了如何使用Podman Desktop将Spring Boot应用程序容器化，我们创建了一个带有API端点的简单Spring Boot应用程序，并为其创建了一个Containerfile。然后，我们使用Podman Desktop应用程序构建了一个镜像，并使用该镜像运行了一个容器。最后，我们在容器启动后测试了我们的端点是否正常工作。