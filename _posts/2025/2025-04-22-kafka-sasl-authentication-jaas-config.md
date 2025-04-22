---
layout: post
title:  使用JAAS配置在Kafka中实现SASL身份验证
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

身份验证是设计任何消息系统(例如Kafka)的基本方面，我们可以使用基于用户的凭证、SSL证书或基于令牌等方法来实现身份验证。

在本教程中，我们将学习如何在Kafka服务中实现一种名为“**简单身份验证和套接字层**(SASL)”的身份验证机制，我们还将使用[Spring Kafka](https://www.baeldung.com/spring-kafka)提供的机制实现客户端身份验证。

## 2. Kafka身份验证简介

[Kafka](https://www.baeldung.com/spring-kafka)支持各种身份验证和授权机制来保护网络通信安全，它支持[SSL](https://www.baeldung.com/spring-boot-kafka-ssl)、SASL或委托令牌。身份验证可以发生在客户端与Broker之间、Broker与[Zookeeper](https://www.baeldung.com/java-zookeeper)之间，或者Broker之间。

我们可以根据系统要求和其他基础设施因素采用任何相关方法，[SSL](https://www.baeldung.com/java-ssl)身份验证使用X.509证书对客户端和代理进行身份验证，并提供单向或双向身份验证。

**[SASL](https://www.baeldung.com/java-sasl)身份验证是一个支持不同身份验证机制的安全框架**：

- **SASL/GSSAPI**：SASL/GSSAPI(通用安全服务应用程序接口)是一种标准API，它通过标准API抽象化安全机制，并可轻松与现有的Kerberos服务集成。SASL/GSSAPI身份验证使用密钥分发中心通过网络提供身份验证，通常用于现有基础设施(例如Active Directory或Kerberos服务器)可用的场合。
- **SASL/PLAIN**：SASL/PLAIN使用基于用户的凭证进行身份验证，它主要用于非生产环境，因为它在网络上不安全。
- **SASL/SCRAM**：使用SASL/SCRAM身份验证，通过对密码进行哈希处理并添加盐值来创建带盐质询-响应，从而提供比纯文本机制更高的安全性。SCRAM支持不同的哈希算法，例如SHA-256、SHA-512或SHA-1(安全性较低)。
- **SASL/OAUTHBEARER**：SASL/OAUTHBEARER使用OAUTH 2.0承载令牌进行身份验证，当我们拥有现有的身份提供商(如Keycloak或OKTA)时很有用。

我们还可以将SASL与SSL身份验证结合起来，提供传输层加密。

对于Kafka中的授权，我们可以使用内置的**基于ACL或OAUTH/OIDC(OpenID Connect)或自定义授权器**。

在本教程中，我们将重点介绍GSSAPI身份验证实现，因为它被广泛使用并且保持其简单性。

## 3. 使用SASL/GSSAPI身份验证实现Kafka服务

假设我们需要在Docker环境中构建一个支持GSSAPI身份验证的Kafka服务，为此，我们可以利用[Kerberos](https://www.baeldung.com/spring-security-kerberos-integration#kerberos-benefits)运行时提供票证授予票证(TGT)服务并充当身份验证服务器。

### 3.1 设置Kerberos

为了在Docker环境中实现Kerberos服务，我们需要自定义[Kerberos](https://web.mit.edu/kerberos/)设置。

首先，让我们包含一个krb5.conf文件来配置领域TUYUCHENG.COM的一些配置：

```conf
[libdefaults]
    default_realm = TUYUCHENG.COM
    dns_lookup_realm = false
    dns_lookup_kdc = false
    forwardable = true
    rdns = true

[realms]
    TUYUCHENG.COM = {
        kdc = kdc
        admin_server = kdc
    }
```

领域是Kafka服务的逻辑名称或域名。

我们需要编写一个脚本，使用kdb5_util初始化Kerberos数据库，然后使用Kadmin.local命令为Kafka、Zookeeper和客户端应用程序创建主体及其关联的密钥表文件。最后，我们将使用krb5kdc和kadmind命令启动Kerberos服务。

然后，让我们实现脚本kdc_setup.sh来添加主体，创建keytab文件，并运行Kerberos服务：

```shell
kadmin.local -q "addprinc -randkey kafka/localhost@TUYUCHENG.COM"
kadmin.local -q "addprinc -randkey zookeeper/zookeeper.sasl_default@TUYUCHENG.COM"
kadmin.local -q "addprinc -randkey client@TUYUCHENG.COM"

kadmin.local -q "ktadd -k /etc/krb5kdc/keytabs/kafka.keytab kafka/localhost@TUYUCHENG.COM"
kadmin.local -q "ktadd -k /etc/krb5kdc/keytabs/zookeeper.keytab zookeeper/zookeeper.sasl_default@TUYUCHENG.COM"
kadmin.local -q "ktadd -k /etc/krb5kdc/keytabs/client.keytab client@TUYUCHENG.COM"

krb5kdc
kadmind -nofork
```

**任何主体的格式通常为<service-name\>/<host/domain\>@REALM**，hostname部分是可选的，REALM通常大写。

我们还应该注意，**主体应该正确设置，否则，身份验证将由于服务名称或完全限定域名不匹配而失败**。

最后，我们来实现一个Dockerfile来准备Kerberos环境：

```dockerfile
FROM debian:bullseye

RUN apt-get update && \
    apt-get install -y krb5-kdc krb5-admin-server krb5-user && \
    rm -rf /var/lib/apt/lists/*
COPY config/krb5.conf /etc/krb5.conf
COPY setup_kdc.sh /setup_kdc.sh

RUN chmod +x /setup_kdc.sh
EXPOSE 88 749

CMD ["/setup_kdc.sh"]
```

上述Dockerfile使用之前创建的krb5.conf和setup_kdc.sh文件来初始化并运行Kerberos服务。

我们还将添加一个kadm5.acl文件来授予管理员主体完全权限：

```shell
*/admin@TUYUCHENG.COM *
```

### 3.2 Kafka和Zookeeper的配置

为了在Kafka中配置GSSAPI身份验证，我们将使用[JAAS](https://www.baeldung.com/java-authentication-authorization-service)(Java身份验证和授权服务)来指定Kafka或客户端如何向Kerberos的密钥分发中心(KDC)进行身份验证。

我们将在Kafka服务器、Zookeeper的单独文件中创建JAAS相关配置。

首先，我们将实现zookeeper_jaas.conf文件并设置之前创建的zookeeper.keytab文件和principal参数：

```conf
Server {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    keyTab="/etc/kafka/keytabs/zookeeper.keytab"
    principal="zookeeper/zookeeper.sasl_default@TUYUCHENG.COM";
};
```

该主体必须与Zookeeper的Kerberos主体相同，通过将useKeyTab设置为true，我们强制身份验证使用keytab文件。

然后，我们在kafka_server_jaas.conf文件中配置Kafka服务器和客户端JAAS相关属性：

```conf
KafkaServer {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    keyTab="/etc/kafka/keytabs/kafka.keytab"
    principal="kafka/localhost@TUYUCHENG.COM"
    serviceName="kafka";
};

Client {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    keyTab="/etc/kafka/keytabs/client.keytab"
    principal="client@TUYUCHENG.COM"
    serviceName="kafka";
};
```

### 3.3 将Kafka与Zookeeper和Kerberos集成

借助Docker服务可以轻松集成Kafka、Zookeeper和自定义Kerberos服务。

首先，我们将使用之前的Dockerfile实现自定义Kerberos服务：

```yaml
services:
    kdc:
        build:
            context: .
            dockerfile: Dockerfile
        volumes:
            - ./config:/etc/krb5kdc
            - ./keytabs:/etc/krb5kdc/keytabs
            - ./config/krb5.conf:/etc/krb5.conf
        ports:
            - "88:88/udp"
```

上述服务将在内部和主机环境的典型UDP 88端口上可用。

然后，让我们使用confluentinc:cp-zookeeper基础镜像设置Zookeeper服务：

```yaml
zookeeper:
    image: confluentinc/cp-zookeeper:latest
    container_name: zookeeper
    environment:
        ZOOKEEPER_CLIENT_PORT: 2181
        ZOOKEEPER_TICK_TIME: 2000
        KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/zookeeper_jaas.conf"
    volumes:
        - ./config/zookeeper_jaas.conf:/etc/kafka/zookeeper_jaas.conf
        - ./keytabs:/etc/kafka/keytabs
        - ./config/krb5.conf:/etc/krb5.conf
    ports:
        - "2181:2181"
```

上述Zookeeper服务也使用zookeeper_jaas.conf配置了GSSAPI身份验证。

最后，我们将使用与GSSAPI相关的环境属性设置Kafka服务：

```yaml
kafka:
    image: confluentinc/cp-kafka:latest
    container_name: kafka
    environment:
        KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
        KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: GSSAPI
        KAFKA_SASL_ENABLED_MECHANISMS: GSSAPI
        KAFKA_LISTENERS: SASL_PLAINTEXT://:9092
        KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://localhost:9092
        KAFKA_INTER_BROKER_LISTENER_NAME: SASL_PLAINTEXT
        KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf"
        KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    volumes:
        - ./config/kafka_server_jaas.conf:/etc/kafka/kafka_server_jaas.conf
        - ./keytabs:/etc/kafka/keytabs
        - ./config/krb5.conf:/etc/krb5.conf
    depends_on:
        - zookeeper
        - kdc
    ports:
        - 9092:9092
```

在上面的Kafka服务中，我们在Kafka代理服务器和客户端上都启用了GSSAPI身份验证。Kafka服务将使用之前创建的kafka_server_jaas.conf文件进行GSSAPI配置，例如主体文件和密钥表文件。

我们应该注意，**KAFKA_ADVERTISED_LISTENERS属性是Kafka客户端将监听的端点**。

现在，我们将使用[docker compose](https://www.baeldung.com/ops/docker-compose)命令运行整个Docker设置：

```shell
$ docker compose up --build
```

```text
kafka      | [2025-02-03 18:09:10,147] INFO Successfully authenticated client: authenticationID=kafka/localhost@TUYUCHENG.COM; authorizationID=kafka/localhost@TUYUCHENG.COM. (org.apache.kafka.common.security.authenticator.SaslServerCallbackHandler)
kafka      | [2025-02-03 18:09:10,148] INFO [RequestSendThread controllerId=1001] Controller 1001 connected to localhost:9092 (id: 1001 rack: null) for sending state change requests (kafka.controller.RequestSendThread)
```

从以上日志中，我们确认Kafka、Zookeeper、Kerberos服务均已集成，没有错误。

## 4. 使用Spring实现Kafka客户端

我们将使用[Spring Kafka](https://www.baeldung.com/spring-kafka#consuming-messages)来实现Kafka监听器应用程序。

### 4.1 Maven依赖

首先，我们将包含[spring-kafka](https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka)依赖：

```shell
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>3.1.2</version>
</dependency>
```

### 4.2 实现Kafka监听器

我们将使用Spring Kafka的[KafkaListener](https://www.baeldung.com/kafka-create-listener-consumer-api)和ConsumerRecord类来实现监听器。

让我们用@KafkaListener注解实现Kafka监听器并添加所需的主题：

```java
@KafkaListener(topics = test-topic)
public void receive(ConsumerRecord<String, String> consumerRecord) {
    log.info("Received payload: '{}'", consumerRecord.toString());
    messages.add(consumerRecord.value());
}
```

另外，我们将在application-sasl.yml文件中配置与Spring监听器相关的配置：

```yaml
spring:
    kafka:
        bootstrap-servers: localhost:9092
        consumer:
            group-id: test
            auto-offset-reset: earliest
            key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

现在，让我们运行Spring应用程序并验证设置：

```text
kafka | [2025-02-01 03:08:01,532] INFO [SocketServer listenerType=ZK_BROKER, nodeId=1001] Failed authentication with /172.21.0.1 (channelId=172.21.0.4:9092-172.21.0.1:59840-16) (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
```

以上日志确认客户端应用程序无法按预期向Kafka服务器进行身份验证。

为了解决这个问题，我们还需要在应用程序中包含Spring Kafka JAAS配置。

## 5. 使用JAAS配置Kafka客户端

我们将使用spring.kafka.properties配置来提供SASL/GSSAPI设置。

现在，我们将包括一些与客户端主体、密钥表文件和sasl.mechanism作为GSSAPI相关的附加配置：

```yaml
spring:
    kafka:
        bootstrap-servers: localhost:9092
        properties:
            sasl.mechanism: GSSAPI
            sasl.jaas.config: >
                com.sun.security.auth.module.Krb5LoginModule required
                useKeyTab=true
                storeKey=true
                keyTab="./src/test/resources/sasl/keytabs/client.keytab"
                principal="client@TUYUCHENG.COM"
                serviceName="kafka";
```

我们应该注意，**上述serviceName配置应该与Kafka主体的serviceName完全匹配**。 

我们再次验证一下Kafka消费者应用程序。

## 6. 在应用程序中测试Kafka监听器

为了快速验证监听器，我们将使用Kafka提供的实用程序kafka-console-producer.sh向主题发送消息。

运行以下命令向主题发送消息：

```shell
$ kafka-console-producer.sh --broker-list localhost:9092 \
  --topic test-topic \
  --producer-property security.protocol=SASL_PLAINTEXT \
  --producer-property sasl.mechanism=GSSAPI \
  --producer-property sasl.kerberos.service.name=kafka \
  --producer-property sasl.jaas.config="com.sun.security.auth.module.Krb5LoginModule required 
    useKeyTab=true keyTab=\"/<path>/client.keytab\" 
    storeKey=true principal=\"client@TUYUCHENG.COM\";"
> hello
```

在上面的命令中，我们通过client.keytab文件传递类似的与身份验证相关的配置，如security.protocol、sasl.mechanism和sasl.jaas.config。

现在，让我们验证一下收到的消息的监听器日志：

```text
08:52:13.663 INFO  c.t.t.s.KafkaConsumer - Received payload: 'ConsumerRecord(topic = test-topic, .... key = null, value = hello)'
```

我们应该注意，任何投入生产的应用程序中可能还需要一些配置，例如配置SSL证书或DNS。

## 7. 总结

在本文中，我们学习了如何在docker环境中设置Kafka服务并使用自定义Kerberos设置在docker环境中启用SASL/GSSAPI身份验证。

我们还实现了客户端监听器应用程序，并使用JAAS配置配置了GSSAPI身份验证。最后，我们通过在监听器中发送和接收消息来测试整个设置。