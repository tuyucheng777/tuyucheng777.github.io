---
layout: post
title:  检查Apache Kafka服务器是否正在运行
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

使用Apache Kafka的客户端应用程序通常分为两类，即生产者和消费者。**生产者和消费者都需要底层Kafka服务器启动并运行，然后它们才能开始生产和消费工作**。

在本文中，我们将学习一些确定Kafka服务器是否正在运行的策略。

## 2. 使用Zookeeper命令

查找是否存在活动代理的最快方法之一是使用Zookeeper的dump命令，**dump命令是可用于管理Zookeeper服务器的[4LW](https://zookeeper.apache.org/doc/r3.4.10/zookeeperAdmin.html#sc_zkCommands)命令之一**。

让我们继续使用[nc](https://www.baeldung.com/linux/netcat-command)命令通过在2181端口监听的Zookeeper服务器发送dump命令：

```shell
$ echo dump | nc localhost 2181 | grep -i broker | xargs
/brokers/ids/0
```

执行该命令后，我们会看到在Zookeeper服务器上注册的临时代理ID列表。如果不存在临时ID，则没有任何代理节点正在运行。

此外，需要注意的是，dump命令需要在配置中明确允许，通常在zookeeper.properties或zoo.cfg配置文件中可用：

```properties
lw.commands.whitelist=dump
```

或者，我们也可以使用Zookeeper API来查找[活跃代理的列表](https://www.baeldung.com/ops/kafka-list-active-brokers-in-cluster#zookeeper-apis)。

## 3. 使用Apache Kafka的AdminClient

**如果我们的生产者或消费者是Java应用程序，那么我们可以使用Apache Kafka的[AdminClient](https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html)类来检查Kafka服务器是否启动**。

让我们定义KafkaAdminClient类来包装AdminClient类的实例，以便我们可以快速测试我们的代码：

```java
public class KafkaAdminClient {
    private final AdminClient client;

    public KafkaAdminClient(String bootstrap) {
        Properties props = new Properties();
        props.put("bootstrap.servers", bootstrap);
        props.put("request.timeout.ms", 3000);
        props.put("connections.max.idle.ms", 5000);

        this.client = AdminClient.create(props);
    }
}
```

接下来，让我们在KafkaAdminClient类中定义verifyConnection()方法来验证客户端是否可以连接正在运行的代理服务器：

```java
public boolean verifyConnection() throws ExecutionException, InterruptedException {
    Collection<Node> nodes = this.client.describeCluster()
        .nodes()
        .get();
    return nodes != null && nodes.size() > 0;
}
```

最后，让我们通过连接正在运行的Kafka集群来测试我们的代码：

```java
@Test
void givenKafkaIsRunning_whenCheckedForConnection_thenConnectionIsVerified() throws Exception {
    boolean alive = kafkaAdminClient.verifyConnection();
    assertThat(alive).isTrue();
}
```

## 4. 使用kcat实用程序

我们可以使用[kcat](https://manpages.ubuntu.com/manpages/focal/man1/kafkacat.1.html)(以前称为kafkacat)命令来检查是否有正在运行的Kafka代理节点。为此，让我们**使用-L选项显示现有主题的元数据**：

```shell
$ kcat -b localhost:9092 -t demo-topic -L
Metadata for demo-topic (from broker -1: localhost:9092/bootstrap):
 1 brokers:
  broker 0 at 192.168.1.53:9092 (controller)
 1 topics:
  topic "demo-topic" with 1 partitions:
    partition 0, leader 0, replicas: 0, isrs: 0
```

接下来，让我们在代理节点关闭时执行相同的命令：

```shell
$ kcat -b localhost:9092 -t demo-topic -L -m 1
%3|1660579562.937|FAIL|rdkafka#producer-1| [thrd:localhost:9092/bootstrap]: localhost:9092/bootstrap: Connect to ipv4#127.0.0.1:9092 failed: Connection refused (after 1ms in state CONNECT)
% ERROR: Failed to acquire metadata: Local: Broker transport failure (Are the brokers reachable? Also try increasing the metadata timeout with -m <timeout>?)
```

在这种情况下，我们收到“Connection refused”错误，因为没有正在运行的代理节点。此外，我们必须注意，我们能够通过使用-m选项将请求超时限制为1秒来快速失败。

## 5. 使用UI工具

**对于不需要自动检查的实验性POC项目，我们可以依赖[Offset Explorer](https://www.kafkatool.com/)等UI工具**。但是，如果我们想验证企业级Kafka客户端的代理节点状态，则不建议使用此方法。

让我们使用Offset Explorer使用Zookeeper主机和端口详细信息连接到Kafka集群：

![](/assets/images/2025/kafka/apachekafkacheckserverisrunning01.png)

我们可以在左侧窗格中看到正在运行的代理列表，就这样，我们只需单击一下按钮即可获得它。

## 6. 总结

在本教程中，我们探索了一些使用Zookeeper命令、Apache的AdminClient和kcat实用程序的命令行方法，然后采用基于UI的方法来确定Kafka服务器是否启动。