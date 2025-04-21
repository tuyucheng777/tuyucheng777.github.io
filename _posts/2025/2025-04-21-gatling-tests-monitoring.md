---
layout: post
title:  Gatling测试监控
category: load
copyright: load
excerpt: Gatling
---

## 1. 概述

Gatling是一款成熟高效的性能测试工具，我们可以用它来对REST应用程序进行负载测试。**但是，我们能从Gatling直接看到的结果只有断言是否得到满足，以及服务器在压力测试期间是否崩溃**。

我们希望获得的信息远不止这些，**在性能测试中，我们希望部署JVM监控，以便了解其是否以最佳状态运行和运行**。

在本文中，我们将设置一些工具来帮助我们在执行Gatling模拟时监控应用程序。设置将在容器级别进行，并且在演示期间我们将使用[Docker Compose](https://www.baeldung.com/ops/docker-compose)进行本地执行，**全面测试和监控性能方面所需的工具包括**：

- 一个公开指标的REST应用程序，[Spring Boot Actuator](https://www.baeldung.com/spring-boot-actuators)可用于获取本教程中所需的指标，无需额外操作。
- [Prometheus](https://www.baeldung.com/spring-boot-prometheus)将成为从REST应用程序收集指标并将其存储为时间序列数据的工具。
- [InfluxDB](https://www.baeldung.com/spring-boot-self-hosted-monitoring#1-influxdb)是一个时序数据库，用于从Gatling收集指标。
- [Grafana](https://www.baeldung.com/spring-boot-self-hosted-monitoring#1-grafana)可以将结果以漂亮的图表形式可视化，让我们可以轻松地与前面提到的数据源集成，我们还可以保存Grafana仪表板并重复使用它们。

## 2. 设置监控工具

**为了演示如何实现良好的监控，我们将使用容器和Docker Compose来启动和编排所有工具**。对于每个工具，我们需要一个包含创建容器指令的[Dockerfile](https://www.baeldung.com/ops/dockerfile-image-generate)。然后，我们将通过Docker Compose启动所有服务，这将使服务之间的执行和通信更加容易。

### 2.1 REST API

让我们从用于运行性能测试的REST API开始，我们将使用一个包含两个端点的简单[Spring Boot MVC应用程序](https://www.baeldung.com/spring-mvc)。由于我们的重点是监控性能测试，因此两个端点都是虚拟的，只需返回一个简单的OK响应即可。

**我们之所以有两个端点，是因为一个端点会立即回复200，而第二个端点会有一些随机延迟，持续一到两秒，然后才返回成功响应**：

```java
@RestController
public class PerformanceTestsController {
    @GetMapping("/api/fast-response")
    public ResponseEntity<String> getFastResponse() {
        return ResponseEntity.ok("was that fast enough?");
    }

    @GetMapping("/api/slow-response")
    public ResponseEntity<String> getSlowResponse() throws InterruptedException {
        int min = 1000;
        int max = 2000;
        TimeUnit.MILLISECONDS.sleep(ThreadLocalRandom.current()
                .nextInt(min, max));

        return ResponseEntity.ok("this took a while");
    }
}
```

然后，我们需要 通过创建Dockerfile来[容器化](https://www.baeldung.com/dockerizing-spring-boot-application)该服务：

```dockerfile
FROM openjdk:17-jdk-slim

COPY target/gatling-java.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

EXPOSE 8080
```

首先，我们选择所需的基础镜像，在本例中是Java 17镜像。然后，我们复制启动服务器所需的构件，我们只需要[创建Spring Boot应用程序JAR文件](https://www.baeldung.com/java-create-jar)，并将其设置为容器的ENTRYPOINT。最后，我们指定需要导出的端口(8080)作为容器的访问点。

### 2.2 Prometheus

**要创建Prometheus容器，我们需要做的就是找到想要使用的Prometheus版本的基础镜像。然后，配置需要抓取指标的目标**。

让我们在configuration.yml文件中定义目标的配置：

```yaml
global:
    scrape_interval:     15s
    evaluation_interval: 15s

scrape_configs:
    - job_name: 'prometheus'
      static_configs:
          - targets: ['localhost:9090']
    - job_name: 'grafana'
      scrape_interval: 5s
      metrics_path: /metrics
      static_configs:
          - targets: ['grafana:3000']
    - job_name: 'service_metrics'
      scrape_interval: 5s
      metrics_path: /private/metrics
      static_configs:
          - targets: ['service:8080']
```

首先，我们将抓取和评估间隔设置为15秒，以便获取更频繁、更准确的数据。请注意，在实际环境中，此频率可能会造成不必要的噪音，默认值通常为30秒。

在scrape_configs中，我们添加了三个作业：

- prometheus将从本地主机(localhost)的Prometheus实例抓取数据，以监控Prometheus在性能测试执行期间是否运行正常。我们之所以包含此功能，是因为运行状况不佳的监控工具可能会导致结果不准确。
- grafana指向grafana服务，我们只需设置主机和路径，主机为grafana:3000，并且仅在通过Docker Compose运行服务时有效，Grafana服务的默认路径为/metrics。
- service_metrics将从我们的Spring Boot应用程序中读取指标，与grafana类似，我们将Docker Compose执行的主机设置为service:8080，并通过Spring Boot Actuator配置定义路径。

有了configuration.yml文件，我们就需要定义Dockerfile：

```dockerfile
FROM prom/prometheus:v2.48.1

COPY config/prometheus-docker.yml /etc/prometheus/prometheus.yml

EXPOSE 9090:9090
```

首先，选择要使用的Prometheus镜像版本。然后，复制并覆盖Prometheus期望在/etc/prometheus/prometheus.yml中找到的配置。最后，指定需要导出为容器访问点的端口(9090)。

### 2.3 Gatling

**为了针对我们创建的两个端点运行性能测试，我们需要创建一个[Gatling模拟](https://www.baeldung.com/gatling-load-testing-rest-endpoint)，以产生我们想要的负载和持续时间，以测试我们的服务**。为了更好地演示，我们将有两个Gatling模拟，一个用于第一个端点，一个用于第二个端点：

```java
public class SlowEndpointSimulation extends Simulation {
    public SlowEndpointSimulation() {
        ChainBuilder getSlowEndpointChainBuilder = SimulationUtils.simpleGetRequest("request_slow_endpoint", "/api/slow-response", 200);
        PopulationBuilder slowResponsesPopulationBuilder = SimulationUtils.buildScenario("getSlowResponses", getSlowEndpointChainBuilder, 120, 30, 300);

        setUp(slowResponsesPopulationBuilder)
                .assertions(
                        details("request_slow_endpoint").successfulRequests().percent().gt(95.00),
                        details("request_slow_endpoint").responseTime().max().lte(10000)
                );
    }
}
```

SlowEndpointSimulation针对/api/slow-response路径设置了一个名为request_slow_endpoint的模拟，负载峰值为每秒120个请求，持续300秒，我们断言95%的响应应该成功，并且响应时间应小于10秒。

类似地，我们定义第二个端点的模拟：

```java
public class FastEndpointSimulation extends Simulation {
    public FastEndpointSimulation() {
        ChainBuilder getFastEndpointChainBuilder
                = SimulationUtils.simpleGetRequest("request_fast_endpoint", "/api/fast-response", 200);
        PopulationBuilder fastResponsesPopulationBuilder
                = SimulationUtils.buildScenario("getFastResponses", getFastEndpointChainBuilder, 200, 30, 180);

        setUp(fastResponsesPopulationBuilder)
                .assertions(
                        details("request_fast_endpoint").successfulRequests().percent().gt(95.00),
                        details("request_fast_endpoint").responseTime().max().lte(10000)
                );
    }
}
```

FastEndpointSimulation类的功能与SlowEndpointSimulation相同，只是名称不同，为request_fast_endpoint，并且针对的是/api/fast-response路径。为了更好地对应用程序施加压力，我们将该类中的事务数设置为每秒200次，持续180秒。

**为了让Gatling导出可从Grafana监控和存储的指标，我们需要配置Gatling并将其公开给Graphite。数据将指向InfluxDB实例，该实例将存储这些数据。最后，Grafana将从InfluxDB(作为其数据源)读取Gatling公开的指标，这样我们就可以创建漂亮的图表来可视化它们**。

为了能够将指标公开给Graphite，我们需要在gatling.conf文件中添加配置：

```conf
data {
    writers = [console, file, graphite]
    console {
    }
    file {
    }
    leak {
    }
    graphite {
        light = false
        host = "localhost"
        port = 2003
        protocol = "tcp"
        rootPathPrefix = "gatling"
        bufferSize = 8192
        writePeriod = 1
    }
}
```

我们添加了额外的Graphite写入器并设置了所需的配置，主机名为localhost，因为InfluxDB将在Docker Compose中运行，并且可以在本地主机上访问，端口号应与InfluxDb公开的端口号2003匹配。最后，我们将rootPathPrefix设置为gatling，这将是我们公开的指标的前缀；其余属性仅用于更好的调整。

Gatling的执行不会成为Docker Compose的一部分，通过Docker Compose启动所有服务后，我们可以使用控制台运行任何我们想要的Gatling场景，模拟该服务的真实外部客户端。

### 2.4 InfluxDB

**设置InfluxDB不像其他工具那么简单，首先，我们需要一个目标版本的基础镜像，然后是一个configuration.conf文件，以及一个ENTRYPOINT文件，其中包含一些额外的容器配置，用于存储Gatling导出的数据**。

我们需要的配置与Graphite有关，我们需要在influxdb.conf文件中启用Graphite端点，以便Gatling可以访问它并发布其执行数据：

```conf
[[graphite]]
    enabled = true
    database = "graphite"
    retention-policy = ""
    bind-address = ":2003"
    protocol = "tcp"
    consistency-level = "one"
    batch-size = 5000
    batch-pending = 10
    batch-timeout = "1s"
    separator = "."
    udp-read-buffer = 0
```

首先，我们设置enabled=true，并添加一些与数据库相关的属性，例如保留策略、一致性级别等，以及一些连接属性，例如协议、绑定地址等。**这里重要的属性是我们设置为绑定地址的端口，这样我们就可以在Graphite配置中的Gatling中使用它**。

下一步是创建实例化influx数据库并创建数据库的entrypoint.sh文件，使用我们创建的配置文件：

```shell
#!/usr/bin/env sh

if [ ! -f "/var/lib/influxdb/.init" ]; then
    exec influxd -config /etc/influxdb/influxdb.conf $@ &

    until wget -q "http://localhost:8086/ping" 2> /dev/null; do
        sleep 1
    done

    influx -host=localhost -port=8086 -execute="CREATE USER ${INFLUX_USER} WITH PASSWORD '${INFLUX_PASSWORD}' WITH ALL PRIVILEGES"
    influx -host=localhost -port=8086 -execute="CREATE DATABASE ${INFLUX_DB}"

    touch "/var/lib/influxdb/.init"

    kill -s TERM %1
fi

exec influxd $@
```

首先，我们执行influxd并指向我们创建的配置文件。然后，在until循环中等待服务可用。最后，我们启动数据库。

创建所有先决条件后，创建Dockerfile应该很简单：

```dockerfile
FROM influxdb:1.3.1-alpine

WORKDIR /app
COPY entrypoint.sh ./
RUN chmod u+x entrypoint.sh
COPY influxdb.conf /etc/influxdb/influxdb.conf

ENTRYPOINT ["/app/entrypoint.sh"]
```

首先，我们使用我们选择的版本的基础镜像，然后将文件复制到容器并授予必要的访问权限，最后，我们将ENTRYPOINT设置为我们创建的entrypoint.sh文件。

### 2.5 Grafana

在容器中启动Grafana服务非常简单，我们只需使用Grafana提供的镜像即可。在本例中，我们还会在镜像中包含一些配置文件和仪表盘，这可以在启动服务后手动完成，但为了获得更好的体验，并将配置和仪表盘存储在VCS中，以便用于历史记录和备份，我们将预先创建这些文件。

首先，我们在yml文件中设置数据源：

```yaml
datasources:
    - name: Prometheus-docker
      type: prometheus
      isDefault: false
      access: proxy
      url: http://prometheus:9090
      basicAuth: false
      jsonData:
          graphiteVersion: "1.1"
          tlsAuth: false
          tlsAuthWithCACert: false
      version: 1
      editable: true
    - name: InfluxDB
      type: influxdb
      uid: P951FEA4DE68E13C5
      isDefault: false
      access: proxy
      url: http://influxdb:8086
      basicAuth: false
      jsonData:
          dbName: "graphite"
      version: 1
      editable: true
```

第一个数据源是Prometheus，第二个是InfluxDB。重要的配置是url，确保我们正确指向Docker Compose中运行的两个服务。

然后，我们创建一个yml文件来定义Grafana可以从中检索我们存储的仪表板的路径：

```yaml
providers:
    - name: 'dashboards'
      folder: ''
      type: file
      disableDeletion: false
      allowUiUpdates: true
      editable: true
      options:
          path: /etc/grafana/provisioning/dashboards
          foldersFromFilesStructure: true
```

在这里，我们需要正确设置路径。然后，我们将其包含在Dockerfile中，以存储仪表板的JSON文件：

```dockerfile
FROM grafana/grafana:10.2.2

COPY provisioning/ /etc/grafana/provisioning/
COPY dashboards/ /etc/grafana/provisioning/dashboards

EXPOSE 3000:3000
```

在Dockerfile中，我们设置了想要使用的Grafana版本的基础镜像。然后，我们将配置文件从provisioning复制到/etc/grafana/provisioning/，Grafana会在其中查找配置，并将dashboards文件夹中的仪表板JSON文件复制到我们在上一步中设置的路径/etc/grafana/provisioning/dashboards。最后，我们指定了想要为服务公开的端口3000。

请注意，为了演示，我们在dashboards文件夹中创建了两个仪表板文件-application-metrics.json和gatling-metrics.json，但它们太长，无法包含在本文中。

### 2.6 Docker Compose

创建完所有服务后，我们将所有内容整合到Docker Compose中：

```yaml
services:
    influxdb:
        build: influxDb
        ports:
            - '8086:8086'
            - '2003:2003'
        environment:
            - INFLUX_USER=admin
            - INFLUX_PASSWORD=admin
            - INFLUX_DB=influx

    prometheus:
        build: prometheus
        depends_on:
            - service
        ports:
            - "9090:9090"

    grafana:
        build: grafana
        ports:
            - "3000:3000"

    service:
        build: .
        ports:
            - "8080:8080"
```

Docker Compose将启动四项服务-influxdb、prometheus、grafana和service。请注意，此处使用的名称将作为这些服务内部通信的主机。

## 3. 监控Gatling测试

**现在我们已经设置好了工具，可以通过Docker Compose启动所有监控服务和REST API服务器。然后，我们可以运行在Gatling中创建的性能测试，并访问Grafana来监控性能**。

### 3.1 执行测试

第一步是通过Docker Compose从终端运行docker-compose up–build来启动所有服务，当它们全部启动并正常运行后，我们就可以使用终端和Maven执行Gatling模拟了。

例如，对于第二个模拟，我们需要执行mvn gatling:test-Dgatling.simulationClass=org.baeldung.FastEndpointSimulation：

![](/assets/images/2025/load/gatlingtestsmonitoring01.png)

正如我们在结果中看到的，我们的“模拟在179秒内完成”并且所有断言都得到满足。

### 3.2 使用Grafana仪表板进行监控

现在，让我们访问Grafana来检查我们收集到的监控数据。我们可以通过浏览器访问http://localhost:3000/，并在用户名和密码字段中输入“admin”进行登录。现在应该有两个现有的仪表板，分别对应我们通过Dockerfile引入的两个仪表板JSON文件。

**第一个仪表板为我们提供了应用程序指标的可视化**：

![](/assets/images/2025/load/gatlingtestsmonitoring02.png)

在“Transaction per Second (TPS)”和“Response delays”视图中，我们可以看到Gatling产生了预期的负载，红色圆圈标注的指标表示慢速端点性能测试，测试耗时300秒，TPS为120，响应延迟约为1.5秒。

紫色突出显示的指标用于快速端点性能测试，此模拟持续了180秒，TPS为200，响应延迟要小得多，因为此端点立即回复了成功响应。

**第二个仪表板是关于Gatling指标的**，它显示了客户端对响应码、延迟等的看法：

![](/assets/images/2025/load/gatlingtestsmonitoring03.png)

在此仪表板中，左侧显示Gatling收集的快速端点测试模拟指标，右侧显示慢速端点测试模拟指标。我们可以验证产生的流量是否符合预期，以及响应状态和延迟是否与我们在应用程序指标仪表板中看到的内容完全一致。

### 3.3 读取结果

现在一切就绪，让我们开始调优JVM。**通过读取指标，我们可以识别可能存在的bug，并尝试以不同的方式调优JVM来提升性能**。

现有仪表板包含JVM线程指标的视图，例如，我们可以查看每个端点的线程使用情况。然后，我们可以进行观察，例如，测试特定端点时的线程数是否高于预期，如果高于预期，则可能意味着我们在该特定端点的实现中过度使用了线程。

**监控JVM的另一个重要方面是垃圾回收指标**，我们应该始终包含有关GC频率、持续时间和其他因素的仪表板。然后，我们可以尝试不同的垃圾回收和配置，重新运行性能测试，并找到适合我们API的最佳解决方案。

## 4. 总结

在本文中，我们研究了如何在使用Gatling执行性能测试时设置合适的监控。我们设置了监控REST API和Gatling所需的工具，并演示了一些测试执行过程。最后，我们使用Grafana仪表板读取结果，并提到了运行性能测试时需要关注的一些指标，例如与JVM线程和垃圾回收相关的指标。