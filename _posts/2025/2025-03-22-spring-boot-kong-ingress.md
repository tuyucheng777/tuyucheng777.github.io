---
layout: post
title:  Kong Ingress Controller与Spring Boot
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

[Kubernetes(K8s)](https://www.baeldung.com/ops/kubernetes)是一种自动化软件开发和部署的编排器，是当今流行的API托管选择，可在本地或云服务(例如Google Cloud Kubernetes Service(GKS)或Amazon Elastic Kubernetes Service(EKS))上运行。另一方面，[Spring](https://www.baeldung.com/spring-boot-minikube)已成为最流行的Java框架之一。

在本教程中，我们将演示如何使用Kong Ingress Controller(KIC)设置受保护的环境以在Kubernetes上部署我们的Spring Boot应用程序。我们还将通过为我们的应用程序实现一个简单的速率限制器(无需任何编码)来演示KIC的高级用法。

## 2. 增强安全性和访问控制

现代应用程序部署(尤其是API)面临许多挑战，例如：隐私法(例如GPDR)、安全问题(DDOS)和使用情况跟踪(例如API配额和速率限制)。在这种情况下，现代应用程序和API需要额外的保护级别来应对所有这些挑战，例如防火墙、反向代理、速率限制器和相关服务。尽管K8s环境可以保护我们的应用程序免受许多此类威胁，但我们仍需要采取一些措施来确保应用程序的安全，其中一项措施是部署入口控制器并设置其对应用程序的访问规则。

入口是一个对象，它通过向已部署的应用程序公开HTTP/HTTPS路由并对其强制执行访问规则来管理对K8s集群及其上部署的应用程序的外部访问。为了公开应用程序以允许外部访问，我们需要定义入口规则并使用入口控制器，这是一种专门的反向代理和负载均衡器。通常，入口控制器由第三方公司提供，功能各不相同，例如本文中使用的[Kong Ingress Controller](https://docs.konghq.com/kubernetes-ingress-controller/latest/)。

## 3. 设置环境

为了演示如何在Spring Boot应用程序中使用Kong Ingress Controller(KIC)，我们需要访问K8s集群，这样我们就可以使用完整的Kubernetes、本地安装或云提供的，或者使用[Minikube](https://www.baeldung.com/spring-boot-minikube)开发我们的示例应用程序。启动K8s环境后，我们需要在[集群](https://docs.konghq.com/kubernetes-ingress-controller/2.7.x/guides/getting-started/)上部署Kong Ingress Controller。Kong公开了一个外部IP，我们需要使用该IP来访问我们的应用程序，因此最好使用该地址创建一个环境变量：

```shell
export PROXY_IP=$(minikube service -n kong kong-proxy --url | head -1)
```

就这样！Kong Ingress Controller已安装，我们可以通过访问PROXY_IP来测试它是否正在运行：

```shell
curl -i $PROXY_IP
```

响应应该是404错误，没错，因为我们还没有部署任何应用程序，所以应该说没有与这些值匹配的路由。现在是时候创建一个示例应用程序了，但在此之前，如果[Docker](https://www.google.com/search?client=safari&rls=en&q=baeldung+docker&ie=UTF-8&oe=UTF-8)不可用，我们可能需要安装它。为了将我们的应用程序部署到K8s，我们需要一种创建容器镜像的方法，我们可以使用Docker来实现。

## 4. 创建示例Spring Boot应用程序

现在我们需要一个Spring Boot应用程序并将其部署到K8s集群。要生成一个至少有一个公开的Web资源的简单HTTP服务器应用程序，我们可以这样做：

```shell
curl https://start.spring.io/starter.tgz -d dependencies=webflux,actuator -d type=maven-project | tar -xzvf -
```

一个重要的事情是选择默认的Java版本。如果我们需要使用旧版本，则需要一个javaVersion属性：

```shell
curl https://start.spring.io/starter.tgz -d dependencies=webflux,actuator -d type=maven-project -d javaVersion=11 | tar -xzvf -
```

在这个示例应用程序中，我们选择了Webflux，它使用[Spring WebFlux和Netty](https://www.baeldung.com/spring-webflux)生成一个响应式Web应用程序，但还添加了另一个重要的依赖项。[Actuator](https://www.baeldung.com/spring-boot-actuators)是Spring应用程序的监控工具，已经公开了一些Web资源，这正是我们用Kong进行测试所需要的。这样，我们的应用程序已经公开了一些我们可以使用的Web资源。让我们来构建它：

```shell
./mvnw install
```

生成的jar是可执行的，因此我们可以通过运行它来测试应用程序：

```shell
java -jar target/*.jar
```

为了测试该应用程序，我们需要打开另一个终端并输入此命令：

```shell
curl -i http://localhost:8080/actuator/health
```

响应必须是应用程序的健康状态，由Actuator提供：

```shell
HTTP/1.1 200 OK
Content-Type: application/vnd.spring-boot.actuator.v3+json
Content-Length: 15

{"status":"UP"}
```

## 5. 从应用程序生成容器镜像

将应用程序部署到Kubernetes集群的过程涉及创建容器镜像并将其部署到集群可访问的仓库。在现实生活中，我们会将镜像推送到DockerHub或我们自己的私有容器镜像注册表。但是，由于我们使用的是Minikube，因此我们只需将Docker客户端环境变量指向Minikube的Docker：

```shell
$(minikube docker-env)
```

我们可以构建应用程序镜像：

```shell
./mvnw spring-boot:build-image
```

## 6. 部署应用程序

现在是时候在我们的K8s集群上部署应用程序了，我们需要创建一些K8s对象来部署和测试我们的应用程序，所有必需的文件都可以在演示的仓库中找到：

- 具有容器规范的应用程序的部署对象
- 为我们的Pod分配集群IP地址的服务定义
- 使用Kong的代理IP地址与我们的路由一起使用的入口规则

部署对象仅创建运行我们的镜像所需的pod，这是创建它的YAML文件：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    creationTimestamp: null
    labels:
        app: demo
    name: demo
spec:
    replicas: 1
    selector:
        matchLabels:
            app: demo
    strategy: {}
    template:
        metadata:
            creationTimestamp: null
            labels:
                app: demo
        spec:
            containers:
                - image: docker.io/library/demo:0.0.1-SNAPSHOT
                  name: demo
                  resources: {}
                  imagePullPolicy: Never

status: {}
```

我们指向Minikube内部创建的镜像，获取其全名。请注意，必须将imagePullPolicy属性指定为Never，因为我们没有使用镜像注册服务器，所以我们不希望K8s尝试下载镜像，而是使用其内部Docker存档中已有的镜像。我们可以使用kubectl部署它：

```shell
kubectl apply -f serviceDeployment.yaml
```

如果部署成功的话我们可以看到如下信息：

```text
deployment.apps/demo created
```

为了给我们的应用程序一个统一的IP地址，我们需要创建一个服务，为其分配一个内部集群范围的IP地址，这是创建它的YAML文件：

```yaml
apiVersion: v1
kind: Service
metadata:
    creationTimestamp: null
    labels:
        app: demo
    name: demo
spec:
    ports:
        - name: 8080-8080
          port: 8080
          protocol: TCP
          targetPort: 8080
    selector:
        app: demo
    type: ClusterIP
status:
    loadBalancer: {}
```

现在我们也可以使用kubectl来部署它：

```shell
kubectl apply -f clusterIp.yaml
```

请注意，我们选择指向我们部署的应用程序的标签demo。为了实现外部访问(在K8s集群之外)，我们需要创建一个入口规则，在我们的例子中，我们将其指向路径/actuator/health和端口8080：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: demo
spec:
    ingressClassName: kong
    rules:
        - http:
              paths:
                  - path: /actuator/health
                    pathType: ImplementationSpecific
                    backend:
                        service:
                            name: demo
                            port:
                                number: 8080
```

最后，我们使用kubectl进行部署：

```shell
kubectl apply -f ingress-rule.yaml
```

现在我们可以使用Kong的代理IP地址进行外部访问：

```shell
$ curl -i $PROXY_IP/actuator/health
HTTP/1.1 200 OK
Content-Type: application/vnd.spring-boot.actuator.v3+json
Content-Length: 49
Connection: keep-alive
X-Kong-Upstream-Latency: 325
X-Kong-Proxy-Latency: 1
Via: kong/3.0.0
```

## 7. 演示速率限制器

我们设法在Kubernetes上部署了一个Spring Boot应用程序，并使用Kong Ingress Controller提供对它的访问。但KIC的功能远不止这些：身份验证、负载均衡、监控、速率限制和其他功能。为了展示Kong的真正威力，我们将为我们的应用程序实现一个简单的速率限制器，将访问限制为每分钟仅5个请求。为此，我们需要在K8s集群中创建一个名为KongClusterPlugin的对象。YAML文件执行此操作：

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongClusterPlugin
metadata:
    name: global-rate-limit
    annotations:
        kubernetes.io/ingress.class: kong
    labels:
        global: true
config:
    minute: 5
    policy: local
plugin: rate-limiting
```

插件配置允许我们为应用程序指定额外的访问规则，并且我们将访问限制为每分钟5个请求。让我们应用此配置并测试结果：

```shell
kubectl apply -f rate-limiter.yaml
```

为了测试它，我们可以在一分钟内重复我们之前使用的CURL命令超过5次，我们就会收到429错误：

```shell
curl -i $PROXY_IP/actuator/health
HTTP/1.1 429 Too Many Requests
Date: Sun, 06 Nov 2022 19:33:36 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
RateLimit-Reset: 24
Retry-After: 24
X-RateLimit-Remaining-Minute: 0
X-RateLimit-Limit-Minute: 5
RateLimit-Remaining: 0
RateLimit-Limit: 5
Content-Length: 41
X-Kong-Response-Latency: 0
Server: kong/3.0.0

{
    "message":"API rate limit exceeded"
}
```

我们可以看到响应HTTP标头告知客户端有关速率限制的信息。

## 8. 清理资源

为了清理演示，我们需要按后进先出的顺序删除所有对象：

```shell
kubectl delete -f rate-limiter.yaml
kubectl delete -f ingress-rule.yaml
kubectl delete -f clusterIp.yaml
kubectl delete -f serviceDeployment.yaml
```

并停止Minikube集群：

```shell
minikube stop
```

## 9. 总结

在本文中，我们演示了如何使用Kong Ingress Controller来管理对部署在K8s集群上的Spring Boot应用程序的访问。