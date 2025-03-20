---
layout: post
title:  调试Quarkus应用程序
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

在本教程中，我们将学习和比较用于调试Quarkus应用程序的不同选项。它们都有一个共同点：我们需要一个调试客户端(通常是IDE)，该客户端连接到正在运行的JVM，并允许我们设置断点并读取堆栈跟踪和变量。但是，启动应用程序(JVM)的选项有所不同。

## 2. 调试Java应用程序

Quarkus是基于Java的，因此我们可以使用常见方法来调试Java应用程序。我们可以在IDE中运行main方法，IDE的调试器直接连接到已启动的JVM。或者我们可以构建JAR文件(例如，使用Maven)并使用以下命令运行它：

```bash
java \
  -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 \
  -jar target/quarkus-app/quarkus-run.jar
```

我们还可以使用此命令使用Docker或Podman容器内运行应用程序，然后我们需要将端口5005发布到主机。

在应用程序运行时，我们可以通过IDE连接到端口5005。在IntelliJ中，我们可以使用Run > Edit Configurations > Remote JVM Debug来执行此操作。

## 3. Quarkus Dev模式

调试Quarkus应用程序的最佳方法是以开发模式启动它，这不仅会以调试模式启动JVM，监听端口5005，还会激活Dev UI、[Dev Services](https://quarkus.io/guides/dev-services)、Live Reload和[Continuous Testing](https://quarkus.io/guides/continuous-testing)等功能。我们可以在[Quarkus文档](https://quarkus.io/guides/dev-mode-differences)中找到有关这些功能的详细信息。

### 3.1 从命令行启动

要从命令行启动应用程序，我们可以使用以下命令：

```bash
# Maven
mvn quarkus:dev
# Quarkus CLI
quarkus dev
```

我们可以选择为所有这些命令提供更多参数：

- -Ddebug=<port\>指定5005以外的其他端口
- -Dsuspend等待调试器连接后再运行应用程序代码

然后，我们按照前面描述的相同方式使用IDE连接到JVM。

### 3.2 从IDE启动

IDE可以帮助我们一步启动Quarkus应用程序并连接调试器，安装Quarkus插件([Quarkus](https://plugins.jetbrains.com/plugin/20306-quarkus)、[Quarkus Run Configs](https://plugins.jetbrains.com/plugin/14242-quarkus-run-configs)和[Quarkus Tools](https://plugins.jetbrains.com/plugin/13234-quarkus-tools))后，IntelliJ具有专门针对Quarkus的运行配置。

![](/assets/images/2025/quarkus/quarkusdebugapps01.png)

如果没有，Quarkus允许我们创建一个自定义的main方法，我们可以从调试模式下的任何Java IDE直接运行该方法：

```java
@QuarkusMain
public class MyQuarkusApplication {
    public static void main(String[] args) {
        Quarkus.run(args);
    }
}
```

### 3.3 Quarkus远程开发

我们还可以在容器或Kubernetes环境(如[OpenShift)](https://quarkus.io/guides/deploying-to-openshift)中部署和运行Quarkus应用程序。为此，我们需要将openshift扩展添加到我们的应用程序中，例如使用Maven：

```bash
# Maven
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-openshift"
# Quarkus CLI
quarkus ext add io.quarkus:quarkus-openshift
```

然后，我们可以使用以下命令在OpenShift集群中创建一个项目，并将应用程序作为所谓的Mutable JAR部署到OpenShift：

```bash
# login to OpenShift
oc login ...
# create a project (connect to a project)
oc new-project quarkus-remote
# deploy the application to the OpenShift project, incl. a Route
mvn clean package -DskipTests \
  -Dquarkus.kubernetes.deploy=true \
  -Dquarkus.package.jar.type=mutable-jar  \
  -Dquarkus.live-reload.password=changeit \
  -Dquarkus.container-image.build=true \
  -Dquarkus.kubernetes-client.trust-certs=true \
  -Dquarkus.kubernetes.deployment-target=openshift \
  -Dquarkus.openshift.route.expose=true \
  -Dquarkus.openshift.env.vars.quarkus-launch-devmode=true
  -Dquarkus.openshift.env.vars.java-enable-debug=true
# with the Quarkus CLI, use "quarkus build -D..."
```

这也会激活实时编码，然后我们可以在OpenShift项目中找到一个具有单个pod、一个服务和一个路由的部署：

![](/assets/images/2025/quarkus/quarkusdebugapps02.png)

对于具有实时重加载和调试的远程开发，我们路由并运行以下命令：

```bash
# Maven
mvn quarkus:remote-dev \
  -Dquarkus.package.jar.type=mutable-jar \
  -Dquarkus.live-reload.password=changeit \
  -Dquarkus.live-reload.url=http://YOUR_APP_ROUTE_URL
# with the Quarkus CLI, use "quarkus "
```

然后，我们按照前面描述的相同方式使用IDE连接到JVM。

## 4. 总结

在本文中，我们了解了在本地和远程机器上以调试模式运行Quarkus应用程序的可能性，在容器中开发和调试可能有助于解决环境问题。