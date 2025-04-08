---
layout: post
title:  使用Loki在Spring Boot中记录日志
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

Grafana Labs开发了[Loki](https://www.baeldung.com/ops/grafana-loki)，这是一个受[Prometheus](https://prometheus.io/)启发的开源日志聚合系统。其目的是存储和索引日志数据，方便高效查询和分析各种应用程序和系统生成的日志。

在本文中，我们将使用Grafana Loki为Spring Boot应用程序设置日志记录。Loki将收集和汇总应用程序日志，Grafana将显示它们。

## 2. 运行Loki和Grafana服务

我们首先启动Loki和Grafana服务，以便收集和观察日志。[Docker](https://www.baeldung.com/spring-boot-docker-start-with-profile)容器将帮助我们更轻松地配置和运行它们。

首先，让我们在[docker-compose](https://www.baeldung.com/spring-boot-postgresql-docker)文件中组合Loki和Grafana服务：

```yaml
version: "3"
networks:
    loki:
services:
    loki:
        image: grafana/loki:2.9.0
        ports:
            - "3100:3100"
        command: -config.file=/etc/loki/local-config.yaml
        networks:
            - loki
    grafana:
        environment:
            - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
            - GF_AUTH_ANONYMOUS_ENABLED=true
            - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
        entrypoint:
            - sh
            - -euc
            - |
                mkdir -p /etc/grafana/provisioning/datasources
                cat <<EOF > /etc/grafana/provisioning/datasources/ds.yaml
                apiVersion: 1
                datasources:
                - name: Loki
                  type: loki
                  access: proxy
                  orgId: 1
                  url: http://loki:3100
                  basicAuth: false
                  isDefault: true
                  version: 1
                  editable: false
                EOF
                /run.sh
        image: grafana/grafana:latest
        ports:
            - "3000:3000"
        networks:
            - loki
```

接下来，我们需要使用docker-compose命令启动服务：

```shell
docker-compose up
```

最后，让我们确认两个服务是否已启动：

```shell
docker ps

211c526ea384        grafana/loki:2.9.0       "/usr/bin/loki -conf…"   4 days ago          Up 56 seconds       0.0.0.0:3100->3100/tcp   surajmishra_loki_1
a1b3b4a4995f        grafana/grafana:latest   "sh -euc 'mkdir -p /…"   4 days ago          Up 56 seconds       0.0.0.0:3000->3000/tcp   surajmishra_grafana_1
```

## 3. 使用Spring Boot配置Loki

启动Grafana和Loki服务后，我们需要配置应用程序以向其发送日志。**我们将使用[loki-logback-appender](https://mvnrepository.com/artifact/com.github.loki4j/loki-logback-appender)，它将负责将日志发送到Loki聚合器以存储和索引日志**。

首先，我们需要在pom.xml文件中添加loki-logback-appender：

```xml
<dependency>
    <groupId>com.github.loki4j</groupId>
    <artifactId>loki-logback-appender</artifactId>
    <version>1.4.1</version>
</dependency>
```

其次，我们需要在src/main/resources文件夹下创建一个logback-spring.xml文件，此文件将控制Spring Boot应用程序的日志记录行为，例如日志的格式、Loki服务的端点等：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="LOKI" class="com.github.loki4j.logback.Loki4jAppender">
        <http>
            <url>http://localhost:3100/loki/api/v1/push</url>
        </http>
        <format>
            <label>
                <pattern>app=${name},host=${HOSTNAME},level=%level</pattern>
                <readMarkers>true</readMarkers>
            </label>
            <message>
                <pattern>
                    {
                    "level":"%level",
                    "class":"%logger{36}",
                    "thread":"%thread",
                    "message": "%message",
                    "requestId": "%X{X-Request-ID}"
                    }
                </pattern>
            </message>
        </format>
    </appender>

    <root level="INFO">
        <appender-ref ref="LOKI" />
    </root>
</configuration>
```

完成设置后，让我们编写一个以INFO级别记录数据的简单服务：

```java
@Service
class DemoService{

    private final Logger LOG = LoggerFactory.getLogger(DemoService.class);

    public void log(){
        LOG.info("DemoService.log invoked");
    }
}
```

## 4. 测试验证

让我们通过启动Grafana和Loki容器进行实时测试，然后执行Service方法将日志推送到Loki。之后，我们将使用[HTTP API](https://grafana.com/docs/loki/latest/reference/api/#query-loki-over-a-range-of-time)查询Loki以确认日志是否确实被推送。有关启动Grafana和Loki容器，请参阅前面的部分。

首先，让我们执行DemoService.log()方法，该方法将调用Logger.info()。这将使用loki-logback-appender发送一条消息，Loki将收集该消息：

```java
DemoService service = new DemoService();
service.log();
```

其次，我们将创建一个请求，用于调用Loki HTTP API提供的REST端点。此GET API接收表示query、开始时间start和结束时间end的查询参数，我们将这些参数添加为请求对象的一部分：

```java
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);

String query = "{level=\"INFO\"} |= `DemoService.log invoked`";

// Get time in UTC
LocalDateTime currentDateTime = LocalDateTime.now(ZoneOffset.UTC);
String current_time_utc = currentDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss'Z'"));

LocalDateTime tenMinsAgo = currentDateTime.minusMinutes(10);
String start_time_utc = tenMinsAgo.format(DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss'Z'"));

URI uri = UriComponentsBuilder.fromUriString(baseUrl)
    .queryParam("query", query)
    .queryParam("start", start_time_utc)
    .queryParam("end", current_time_utc)
    .build()
    .toUri();
```

接下来，我们使用请求对象执行REST请求：

```java
RestTemplate restTemplate = new RestTemplate();
ResponseEntity<String> response = restTemplate.exchange(uri, HttpMethod.GET, new HttpEntity<>(headers), String.class);
```

现在我们需要处理响应并提取我们感兴趣的日志消息，我们将使用[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)来读取JSON响应并提取日志消息：

```java
ObjectMapper objectMapper = new ObjectMapper();
List<String> messages = new ArrayList<>();
String responseBody = response.getBody();
JsonNode jsonNode = objectMapper.readTree(responseBody);
JsonNode result = jsonNode.get("data")
    .get("result")
    .get(0)
    .get("values");

result.iterator()
    .forEachRemaining(e -> {
        Iterator<JsonNode> elements = e.elements();
        elements.forEachRemaining(f -> messages.add(f.toString()));
    });
```

最后，让我们断言在响应中收到的消息包含由DemoService记录的消息：

```java
assertThat(messages).anyMatch(e -> e.contains(expected));
```

## 5. 日志聚合和可视化

由于使用loki-logback-appender进行了配置，我们的服务日志被推送到Loki服务。我们可以通过在浏览器中访问http://localhost:3000(部署Grafana服务的位置)来查看它。

**要查看已在Loki中存储和索引的日志，我们需要使用Grafana。Grafana数据源为Loki提供了可配置的连接参数，我们需要在其中输入Loki端点、身份验证机制等**。

首先，让我们配置已推送日志的Loki端点：

![](/assets/images/2025/springboot/springbootlokigrafanalogging01.png)

成功配置数据源后，让我们继续探索用于查询日志的数据页面：

![](/assets/images/2025/springboot/springbootlokigrafanalogging02.png)

我们可以编写[查询](https://grafana.com/docs/loki/latest/query/)以将应用程序日志提取到Grafana中进行可视化。在我们的演示服务中，我们正在推送INFO日志，因此我们需要将其添加到我们的过滤器并运行查询：

![](/assets/images/2025/springboot/springbootlokigrafanalogging03.png)

一旦我们运行查询，我们将看到所有与我们的搜索相匹配的INFO日志：

![](/assets/images/2025/springboot/springbootlokigrafanalogging04.png)

## 6. 总结

在本文中，我们使用Grafana Loki为Spring Boot应用程序设置了日志记录。我们还通过单元测试和可视化验证了我们的设置，使用记录INFO日志的简单逻辑并在Grafana中设置了Loki数据源。