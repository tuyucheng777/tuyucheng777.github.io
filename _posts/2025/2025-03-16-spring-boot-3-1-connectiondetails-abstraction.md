---
layout: post
title: Spring Boot ConnectionDetails抽象
category: springboot
copyright: springboot
excerpt: Spring Boot 3
---

## 1. 概述

在本教程中，我们将了解Spring Boot 3.1中引入的[ConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/service/connection/ConnectionDetails.html)接口，用于外部化连接属性。Spring Boot提供开箱即用的抽象，可与关系型数据库、NoSQL数据库、消息传递服务等远程服务集成。

传统上，application.properties文件用于存储远程服务的连接详细信息。因此，很难将这些属性外部化到外部服务(如[AWS Secret Manager](https://aws.amazon.com/secrets-manager/)、[Hashicorp Vault](https://www.vaultproject.io/)等)。

为了解决这一问题，Spring Boot引入了ConnectionDetails，此接口为空，作用类似于标签。**Spring提供了此接口的子接口，例如[JdbcConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/jdbc/JdbcConnectionDetails.html)、[CassandraConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/cassandra/CassandraConnectionDetails.html)、[KafkaConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/kafka/KafkaConnectionDetails.html)等，它们可以在Spring配置类中实现并指定为Bean**。此后，Spring依靠这些配置Bean来动态检索连接属性，而不是静态的application.properties文件。

我们首先介绍一个用例，然后介绍它的实现。

## 2. 用例描述

假设有一家跨国银行，名为马尔格迪银行。它运营着许多在Spring Boot上运行的应用程序，这些应用程序连接到各种远程服务。目前，这些远程服务的连接详细信息存储在application.properties文件中。

在最近的一次审查之后，银行合规机构对这些财产的安全性提出了一些担忧，他们针对这些问题提出了一些要求：

- **加密所有秘密**
- **定期轮换机密**
- **电子邮件中不交换秘密**

## 3. 提出的解决方案和设计

马尔格迪银行的应用程序所有者针对上述担忧进行了头脑风暴，最终想出了一个解决方案，他们建议将所有机密转移到[Hashicorp Vault](https://www.vaultproject.io/)中。

因此，所有Spring Boot应用程序都必须从Vault中读取机密，以下是建议的高级设计：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction01.png)

现在，Spring Boot应用程序必须使用密钥调用Vault服务来检索密钥。然后，使用检索到的密钥，它可以调用远程服务来获取连接对象以进行进一步的操作。

因此，应用程序将依赖Vault来安全地存储机密。Vault将根据组织的政策定期轮换机密。如果应用程序缓存了机密，它们还必须重新加载它们。

## 4. 使用ConnectionDetails实现

**使用ConnectionDetails接口，Spring Boot应用程序可以自行发现连接详细信息，而无需任何手动干预**。话虽如此，需要注意的是ConnectionDetails优先于application.properties文件。但是，仍有一些非连接属性(如JDBC连接池大小)仍可通过application.properties文件进行配置。

在下一节中，我们将利用[Spring Boot Docker Compose功能](https://www.baeldung.com/ops/docker-compose-support-spring-boot)看到各种ConnectionDetails实现类的实际应用。

### 4.1 外部化JDBC连接详细信息

这里我们以Spring Boot应用与Postgres数据库集成为例，先从类图开始：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction02.png)

在上面的类图中，[JdbcConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/jdbc/JdbcConnectionDetails.html)接口来自Spring Boot框架，PostgresConnectionDetails类实现接口的方法以从Vault获取详细信息：

```java
public class PostgresConnectionDetails implements JdbcConnectionDetails {
    @Override
    public String getUsername() {
        return VaultAdapter.getSecret("postgres_user_key");
    }

    @Override
    public String getPassword() {
        return VaultAdapter.getSecret("postgres_secret_key");
    }

    @Override
    public String getJdbcUrl() {
        return VaultAdapter.getSecret("postgres_jdbc_url");
    }
}
```

如下所示，JdbcConnectionDetailsConfiguration是应用程序中的配置类：

```java
@Configuration(proxyBeanMethods = false)
public class JdbcConnectionDetailsConfiguration {
    @Bean
    @Primary
    public JdbcConnectionDetails getPostgresConnection() {
        return new PostgresConnectionDetails();
    }
}
```

有趣的是，Spring Boot在应用程序启动过程中会自动发现它并获取JdbcConnectionDetails Bean。如前所述，该Bean包含从Vault检索Postgres数据库连接详细信息的逻辑。

由于我们使用Docker Compose启动Postgres数据库容器，Spring Boot会自动创建一个ConnectionDetails Bean，其中包含必要的连接详细信息。因此，我们使用@Primary注解使JdbcConectionDetails Bean优先于它。

让我们看看它是如何工作的：

```java
@Test
public void givenSecretVault_whenIntegrateWithPostgres_thenConnectionSuccessful() {
    String sql = "select current_date;";
    Date date = jdbcTemplate.queryForObject(sql, Date.class);
    assertEquals(LocalDate.now().toString(), date.toString());
}
```

正如预期的那样，应用程序连接到数据库并成功获取结果。

### 4.2 外部化RabbitMQ连接详细信息

与JdbcConnectionDetails类似，**Spring Boot提供了[RabbitConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/amqp/RabbitConnectionDetails.html)接口来与[RabbitMQ Server](https://www.baeldung.com/rabbitmq)集成**，让我们看看如何使用此接口来外部化用于连接RabbitMQ Server的Spring Boot属性：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction03.png)

首先，根据契约，让我们实现[RabbitConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/amqp/RabbitConnectionDetails.html)接口以从Vault中获取连接属性：

```java
public class RabbitMQConnectionDetails implements RabbitConnectionDetails {
    @Override
    public String getUsername() {
        return VaultAdapter.getSecret("rabbitmq_username");
    }

    @Override
    public String getPassword() {
        return VaultAdapter.getSecret("rabbitmq_password");
    }

    @Override
    public String getVirtualHost() {
        return "/";
    }

    @Override
    public List<Address> getAddresses() {
        return List.of(this.getFirstAddress());
    }

    @Override
    public Address getFirstAddress() {
        return new Address(VaultAdapter.getSecret("rabbitmq_host"), Integer.valueOf(VaultAdapter.getSecret("rabbitmq_port")));
    }
}
```

接下来，我们将在RabbitMQConnectionDetailsConfiguration类中定义上述Bean RabbitMQConnectionDetails：

```java
@Configuration(proxyBeanMethods = false)
public class RabbitMQConnectionDetailsConfiguration {
    @Primary
    @Bean
    public RabbitConnectionDetails getRabbitmqConnection() {
        return new RabbitMQConnectionDetails();
    }
}
```

最后，让我们看看是否有效：

```java
@Test
public void givenSecretVault_whenPublishMessageToRabbitmq_thenSuccess() {
    final String MSG = "this is a test message";
    this.rabbitTemplate.convertAndSend(queueName, MSG);
    assertEquals(MSG, this.rabbitTemplate.receiveAndConvert(queueName));
}
```

上述方法将消息发送到RabbitMQ中的队列，然后读取该消息。通过引用RabbitMQConnectionDetails Bean中的连接详细信息，Spring Boot会自动配置RabbitTemplate对象。我们将RabbitTemplate对象注入到测试类中，然后在上述测试方法中使用它。

### 4.3 外部化Redis连接详细信息

现在让我们继续讨论[Redis](https://www.baeldung.com/spring-data-redis-tutorial)上的Spring ConnectionDetails抽象。首先，我们从类图开始：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction04.png)

**让我们看一下RedisCacheConnectionDetails，它通过实现[RedisConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/data/redis/RedisConnectionDetails.html)将Redis的连接属性外部化**：

```java
public class RedisCacheConnectionDetails implements RedisConnectionDetails {
    @Override
    public String getPassword() {
        return VaultAdapter.getSecret("redis_password");
    }

    @Override
    public Standalone getStandalone() {
        return new Standalone() {
            @Override
            public String getHost() {
                return VaultAdapter.getSecret("redis_host");
            }

            @Override
            public int getPort() {
                return Integer.valueOf(VaultAdapter.getSecret("redis_port"));
            }
        };
    }
}
```

如下所示，配置类RedisConnectionDetailsConfiguration返回RedisConnectionDetails Bean：

```java
@Configuration(proxyBeanMethods = false)
@Profile("redis")
public class RedisConnectionDetailsConfiguration {
    @Bean
    @Primary
    public RedisConnectionDetails getRedisCacheConnection() {
        return new RedisCacheConnectionDetails();
    }
}
```

最后我们看看是否可以与Redis集成：

```java
@Test
public void giveSecretVault_whenStoreInRedisCache_thenSuccess() {
    redisTemplate.opsForValue().set("City", "New York");
    assertEquals("New York", redisTemplate.opsForValue().get("City"));
}
```

首先，Spring框架成功将RedisTemplate注入到测试类中。然后使用它向缓存中添加键值对，最后，我们再检索该值。

### 4.4 外部化MongoDB连接详细信息

和以前一样，让我们从通常的类图开始：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction05.png)

我们看一下上面的[MongoDBConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/mongo/MongoConnectionDetails.html)类的实现：

```java
public class MongoDBConnectionDetails implements MongoConnectionDetails {
    @Override
    public ConnectionString getConnectionString() {
        return new ConnectionString(VaultAdapter.getSecret("mongo_connection_string"));
    }
}
```

就像类图一样，我们已经实现了MongoConnectionDetails接口的方法getConnectionString()，方法getConnectionString()从Vault中检索连接字符串。

现在我们可以看看MongoDBConnectionDetailsConfiguration类如何创建MongoConnectionDetails Bean：

```java
@Configuration(proxyBeanMethods = false)
public class MongoDBConnectionDetailsConfiguration {
    @Bean
    @Primary
    public MongoConnectionDetails getMongoConnectionDetails() {
        return new MongoDBConnectionDetails();
    }
}
```

让我们看看我们的努力是否能够成功与[MongoDB服务器](https://www.baeldung.com/spring-data-mongodb-tutorial)集成：

```java
@Test
public void givenSecretVault_whenExecuteQueryOnMongoDB_ReturnResult() {
    mongoTemplate.insert("{\"msg\":\"My First Entry in MongoDB\"}", "myDemoCollection");
    String result = mongoTemplate.find(new Query(), String.class, "myDemoCollection").get(0);

    JSONObject jsonObject = new JSONObject(result);
    result = jsonObject.get("msg").toString();

    assertEquals("My First Entry in MongoDB", result);
}
```

因此，如上所示，该方法将数据插入MongoDB，然后成功检索数据。这是可能的，因为Spring Boot在MongoDBConnectionDetailsConfiguration中定义的MongoConnectionDetails Bean的帮助下创建了[mongoTemplate](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html) Bean。

### 4.5 外部化R2dbc连接详细信息

**Spring Boot还借助[R2dbcConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/r2dbc/R2dbcConnectionDetails.html)[为反应式关系数据库连接编程](https://www.baeldung.com/r2dbc)提供了ConnectionDetails抽象**，让我们看一下以下类图以外部化连接详细信息：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction06.png)

首先，让我们实现R2dbcPostgresConnectionDetails：

```java
public class R2dbcPostgresConnectionDetails implements R2dbcConnectionDetails {
    @Override
    public ConnectionFactoryOptions getConnectionFactoryOptions() {
        ConnectionFactoryOptions options = ConnectionFactoryOptions.builder()
                .option(ConnectionFactoryOptions.DRIVER, "postgresql")
                .option(ConnectionFactoryOptions.HOST, VaultAdapter.getSecret("r2dbc_postgres_host"))
                .option(ConnectionFactoryOptions.PORT, Integer.valueOf(VaultAdapter.getSecret("r2dbc_postgres_port")))
                .option(ConnectionFactoryOptions.USER, VaultAdapter.getSecret("r2dbc_postgres_user"))
                .option(ConnectionFactoryOptions.PASSWORD, VaultAdapter.getSecret("r2dbc_postgres_secret"))
                .option(ConnectionFactoryOptions.DATABASE, VaultAdapter.getSecret("r2dbc_postgres_database"))
                .build();

        return options;
    }
}
```

就像前面的部分一样，这里我们也使用VaultAdapter来检索连接详细信息。

现在，让我们实现R2dbcPostgresConnectionDetailsConfiguration类以将R2dbcPostgresConnectionDetails作为Spring Bean返回：

```java
@Configuration(proxyBeanMethods = false)
public class R2dbcPostgresConnectionDetailsConfiguration {
    @Bean
    @Primary
    public R2dbcConnectionDetails getR2dbcPostgresConnectionDetails() {
        return new R2dbcPostgresConnectionDetails();
    }
}
```

由于上述Bean，Spring Boot框架自动配置了R2dbcEntityTemplate。最后，它可以自动装配并以反应方式用于运行查询：

```java
@Test
public void givenSecretVault_whenQueryPostgresReactive_thenSuccess() {
    String sql = "select * from information_schema.tables";

    List<String> result = r2dbcEntityTemplate.getDatabaseClient().sql(sql).fetch().all()
        .map(r -> "hello " + r.get("table_name").toString()).collectList().block();
    logger.info("count ------" + result.size());
}
```

### 4.6 外部化Elasticsearch连接详细信息

**为了外部化[Elasticsearch](https://www.baeldung.com/spring-data-elasticsearch-tutorial)服务的连接详细信息，Spring Boot提供了接口[ElasticsearchConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/elasticsearch/ElasticsearchConnectionDetails.html)**。让我们首先看一下以下类图：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction07.png)

与之前一样，我们采用相同的模式来检索连接详细信息。现在，我们可以继续实现，从CustomElasticsearchConnectionDetails类开始：

```java
public class CustomElasticsearchConnectionDetails implements ElasticsearchConnectionDetails {
    @Override
    public List<Node> getNodes() {
        Node node1 = new Node(
                VaultAdapter.getSecret("elastic_host"),
                Integer.valueOf(VaultAdapter.getSecret("elastic_port1")),
                Node.Protocol.HTTP
        );
        Node node2 = new Node(
                VaultAdapter.getSecret("elastic_host"),
                Integer.valueOf(VaultAdapter.getSecret("elastic_port2")),
                Node.Protocol.HTTP
        );
        return List.of(node1, node2);
    }

    @Override
    public String getUsername() {
        return VaultAdapter.getSecret("elastic_user");
    }

    @Override
    public String getPassword() {
        return VaultAdapter.getSecret("elastic_secret");
    }
}
```

该类使用VaultAdapter设置连接详细信息。

让我们看一下Spring Boot用于发现ElasticSearchConnectionDetails Bean的配置类：

```java
@Configuration(proxyBeanMethods = false)
@Profile("elastic")
public class CustomElasticsearchConnectionDetailsConfiguration {
    @Bean
    @Primary
    public ElasticsearchConnectionDetails getCustomElasticConnectionDetails() {
        return new CustomElasticsearchConnectionDetails();
    }
}
```

最后，是时候检查它是如何工作的了：

```java
@Test
public void givenSecretVault_whenCreateIndexInElastic_thenSuccess() {
    Boolean result = elasticsearchTemplate.indexOps(Person.class).create();
    logger.info("index created:" + result);
    assertTrue(result);
}
```

有趣的是，Spring Boot会自动将具有正确连接详细信息的elasticsearchTemplate配置到测试类中。然后，它用于在Elasticsearch中创建索引。

### 4.7 外部化Cassandra连接详细信息

与往常一样，以下是所提出的实现的类图：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction08.png)

根据Spring Boot，我们必须实现[CassandraConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/cassandra/CassandraConnectionDetails.html)接口的方法，如上所示。让我们看看CustomCassandraConnectionDetails类的实现：

```java
public class CustomCassandraConnectionDetails implements CassandraConnectionDetails {
    @Override
    public List<Node> getContactPoints() {
        Node node = new Node(
                VaultAdapter.getSecret("cassandra_host"),
                Integer.parseInt(VaultAdapter.getSecret("cassandra_port"))
        );
        return List.of(node);
    }

    @Override
    public String getUsername() {
        return VaultAdapter.getSecret("cassandra_user");
    }

    @Override
    public String getPassword() {
        return VaultAdapter.getSecret("cassandra_secret");
    }

    @Override
    public String getLocalDatacenter() {
        return "datacenter-1";
    }
}
```

基本上，我们从Vault中检索大多数敏感的连接详细信息。

现在，我们可以看一下负责创建CustomCassandraConnectionDetails Bean的配置类：

```java
@Configuration(proxyBeanMethods = false)
public class CustomCassandraConnectionDetailsConfiguration {
    @Bean
    @Primary
    public CassandraConnectionDetails getCustomCassandraConnectionDetails() {
        return new CustomCassandraConnectionDetails();
    }
}
```

最后，让我们看看[Spring Boot](https://www.baeldung.com/spring-data-cassandra-tutorial)是否能够自动配置[CassandraTemplate](https://www.baeldung.com/spring-data-cassandratemplate-cqltemplate)：

```java
@Test
public void givenSecretVaultVault_whenRunQuery_thenSuccess() {
    Boolean result = cassandraTemplate.getCqlOperations()
        .execute("CREATE KEYSPACE IF NOT EXISTS spring_cassandra"
        + " WITH replication = {'class':'SimpleStrategy', 'replication_factor':3}");
    logger.info("the result -" + result);
    assertTrue(result);
}
```

使用CassandraTemplate，上述方法成功在Cassandra数据库中创建了一个键空间。

### 4.8 外部化Neo4j连接详细信息

Spring Boot为流行的图形数据库[Neo4j](https://www.baeldung.com/spring-data-neo4j-intro)提供了ConnectionDetails抽象：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction09.png)

接下来，让我们实现[CustomNeo4jConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/neo4j/Neo4jConnectionDetails.html)：

```java
public class CustomNeo4jConnectionDetails implements Neo4jConnectionDetails {
    @Override
    public URI getUri() {
        try {
            return new URI(VaultAdapter.getSecret("neo4j_uri"));
        } catch (URISyntaxException e) {
            throw new RuntimeException(e);
        }
    }
    @Override
    public AuthToken getAuthToken() {
        return AuthTokens.basic("neo4j", VaultAdapter.getSecret("neo4j_secret"));
    }
}
```

再次，这里我们也使用VaultAdapter从Vault读取连接详细信息。

现在，让我们实现CustomNeo4jConnectionDetailsConfiguration：

```java
@Configuration(proxyBeanMethods = false)
public class CustomNeo4jConnectionDetailsConfiguration {
    @Bean
    @Primary
    public Neo4jConnectionDetails getNeo4jConnectionDetails() {
        return new CustomNeo4jConnectionDetails();
    }
}
```

Spring Boot框架使用上述配置类加载Neo4jConnectionDetails Bean。

最后，看看以下方法是否成功连接到Neo4j数据库：

```java
@Test
public void giveSecretVault_whenRunQuery_thenSuccess() {
    Person person = new Person();
    person.setName("James");
    person.setZipcode("751003");

    Person data = neo4jTemplate.save(person);
    assertEquals("James", data.getName());
}
```

值得注意的是，[Neo4jTemplate](https://docs.spring.io/spring-data/neo4j/docs/current/api/org/springframework/data/neo4j/core/Neo4jTemplate.html)自动注入到测试类并将数据保存到数据库中。

### 4.9 外部化Kafka连接详细信息

Kafka是一种流行且功能极其强大的消息传递代理，Spring Boot也为其提供了集成库。**[KafkaConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/kafka/KafkaConnectionDetails.html)是Spring的最新功能，支持外部化连接属性**。因此，让我们看看如何在以下类图的帮助下使用它：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction10.png)

上述设计与迄今为止讨论过的设计非常相似。因此，我们将直接跳到它的实现，从CustomKafkaConnectionDetails类开始：

```java
public class CustomKafkaConnectionDetails implements KafkaConnectionDetails {
    @Override
    public List<String> getBootstrapServers() {
        return List.of(VaultAdapter.getSecret("kafka_servers"));
    }
}
```

对于Kafka单节点服务器的最基本设置，上述类只需重写方法getBootstrapServers()即可从Vault中读取属性。对于更复杂的多节点设置，还有其他方法可以重写。

我们现在可以看看CustomKafkaConnectionDetailsConfiguration类：

```java
@Configuration(proxyBeanMethods = false)
public class CustomKafkaConnectionDetailsConfiguration {
    @Bean
    public KafkaConnectionDetails getKafkaConnectionDetails() {
        return new CustomKafkaConnectionDetails();
    }
}
```

上述方法返回KafkaConnectionDetails Bean。最后，Spring使用它来将KafkaTemplate注入到以下方法中：

```java
@Test
public void givenSecretVault_whenPublishMsgToKafkaQueue_thenSuccess() {
    assertDoesNotThrow(kafkaTemplate::getDefaultTopic);
}
```

### 4.10 外部化Couchbase连接详细信息

**Spring Boot还提供了接口[CouchbaseConnectionDetails](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/couchbase/CouchbaseAutoConfiguration.html)，用于外部化[Couchbase](https://www.baeldung.com/spring-data-couchbase)数据库的连接属性**。我们来看看下面的类图：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction11.png)

我们首先通过重写其方法来实现CouchbaseConnectionDetails接口来获取用户、密码和连接字符串：

```java
public class CustomCouchBaseConnectionDetails implements CouchbaseConnectionDetails {
    @Override
    public String getConnectionString() {
        return VaultAdapter.getSecret("couch_connection_string");
    }

    @Override
    public String getUsername() {
        return VaultAdapter.getSecret("couch_user");
    }

    @Override
    public String getPassword() {
        return VaultAdapter.getSecret("couch_secret");
    }
}
```

然后，我们将在CustomCouchBaseConnectionDetails类中创建上述自定义Bean：

```java
@Configuration(proxyBeanMethods = false)
@Profile("couch")
public class CustomCouchBaseConnectionDetailsConfiguration {
    @Bean
    public CouchbaseConnectionDetails getCouchBaseConnectionDetails() {
        return new CustomCouchBaseConnectionDetails();
    }
}
```

Spring Boot在应用程序启动时加载上述配置类。

现在，我们可以检查以下方法是否能够成功连接到Couchbase服务器：

```java
@Test
public void givenSecretVault_whenConnectWithCouch_thenSuccess() {
    assertDoesNotThrow(cluster.ping()::version);
}
```

[Cluster](https://docs.couchbase.com/sdk-api/couchbase-java-client/com/couchbase/client/java/Cluster.html)类在方法中自动注入，然后用于与数据库集成。

### 4.11 外部化Zipkin连接详细信息

最后，在本节中，**我们将讨论ZipkinConnectionDetails接口，用于外部化连接到流行的分布式跟踪系统[Zipkin Server](https://www.baeldung.com/tracing-services-with-zipkin)的属性**。让我们从以下类图开始：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction12.png)

使用类图中的上述设计，我们首先实现CustomZipkinConnectionDetails：

```java
public class CustomZipkinConnectionDetails implements ZipkinConnectionDetails {
    @Override
    public String getSpanEndpoint() {
        return VaultAdapter.getSecret("zipkin_span_endpoint");
    }
}
```

方法getSpanEndpoint()使用VaultAdapter从Vault获取Zipkin API端点。

接下来，我们将实现CustomZipkinConnectionDetailsConfiguration类：

```java
@Configuration(proxyBeanMethods = false)
@Profile("zipkin")
public class CustomZipkinConnectionDetailsConfiguration {
    @Bean
    @Primary
    public ZipkinConnectionDetails getZipkinConnectionDetails() {
        return new CustomZipkinConnectionDetails();
    }
}
```

我们可以看到，它返回了ZipkinConnectionDetails Bean。在应用程序启动期间，Spring Boot会发现该Bean，以便[Zipkin库](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#actuator.micrometer-tracing.getting-started)可以将跟踪信息推送到Zipkin。

让我们首先运行该应用程序：

```shell
mvn spring-boot:run -P connection-details
-Dspring-boot.run.arguments="--spring.config.location=./target/classes/connectiondetails/application-zipkin.properties"
```

在运行应用程序之前，我们必须在本地工作站上运行Zipkin。

然后，我们将运行以下命令来访问ZipkinDemoController中定义的控制器端点：

```shell
 curl http://localhost:8080/zipkin/test
```

最后，我们可以在Zipkin前端检查跟踪：

![](/assets/images/2025/springboot/springboot31connectiondetailsabstraction13.png)

## 5. 总结

在本文中，我们了解了Spring Boot 3.1中的ConnectionDetails接口。我们了解了它如何帮助将敏感的连接详细信息外部化到应用程序使用的远程服务，值得注意的是，一些与连接无关的信息仍然是从application.properties文件中读取的。