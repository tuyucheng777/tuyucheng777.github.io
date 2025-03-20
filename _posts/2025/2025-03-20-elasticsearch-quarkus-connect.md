---
layout: post
title:  使用Quarkus连接ElasticSearch
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

[Quarkus](https://www.baeldung.com/quarkus-io)是一个现代框架，它使构建高性能应用程序变得轻松有趣。在本教程中，我们将探索如何将Quarkus与著名的全文搜索引擎和NoSQL数据存储[ElasticSearch](https://www.baeldung.com/elasticsearch-java)集成。

## 2. 依赖和配置

一旦我们在localhost上运行了[ElasticSearch实例](https://www.baeldung.com/elasticsearch-java#setup)，就可以将依赖添加到我们的Quarkus应用程序中：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-elasticsearch-rest-client</artifactId>
    <version>${quarkus.version}</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-elasticsearch-java-client</artifactId>
    <version>${quarkus.version}</version>
</dependency>
```

我们添加了[quarkus-elasticsearch-rest-client依赖](https://mvnrepository.com/artifact/io.quarkus/quarkus-elasticsearch-rest-client)，它为我们带来了一个低级ElasticSearch REST客户端。除此之外，我们还添加了[quarkus-elasticsearch-java-client依赖](https://mvnrepository.com/artifact/io.quarkus/quarkus-elasticsearch-java-client)，它使我们能够使用ElasticSearch Java客户端。**在我们的应用程序中，我们可以选择最适合我们需求的选项**。

下一步，让我们将ElasticSearch主机添加到我们的application.properties文件中：

```properties
quarkus.elasticsearch.hosts=localhost:9200
```

现在我们准备开始在Quarkus应用程序中使用ElasticSearch，**所有必要的Bean将由ElasticSearchRestClientProducer和ElasticSearchJavaClientProducer在后台自动创建**。

## 3. ElasticSearch低级REST客户端

我们可以使用[ElasticSearch低级REST客户端](https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client/8.14.3/org/elasticsearch/client/RestClient.html)将我们的应用程序与ElasticSearch集成，这使我们能够完全控制序列化和反序列化过程，并允许我们使用JSON为ElasticSearch构建查询。

让我们创建一个我们想要在应用程序中索引的模型：

```java
public class StoreItem {
    private String id;
    private String name;
    private Long price;
    //getters and setters
}
```

在我们的模型中，我们有一个用于文档ID的字段。此外，我们还添加了一些字段以方便搜索。

现在，让我们添加方法来索引StoreItem：

```java
private void indexUsingRestClient() throws IOException, InterruptedException {
    iosPhone = new StoreItem();
    iosPhone.setId(UUID.randomUUID().toString());
    iosPhone.setPrice(1000L);
    iosPhone.setName("IOS smartphone");

    Request restRequest = new Request(
            "PUT",
            "/store-items/_doc/" + iosPhone.getId());
    restRequest.setJsonEntity(JsonObject.mapFrom(iosPhone).toString());
    restClient.performRequest(restRequest);
}
```

这里我们创建了一个具有随机ID和特定名称的StoreItem，然后我们对/store-items/_doc/{id}路径执行了PUT请求以索引我们的文档。现在我们将创建一个方法来验证我们的文档是如何被索引的，并通过各种字段搜索它们：

```java
@Test
void givenRestClient_whenSearchInStoreItemsByName_thenExpectedDocumentsFound() throws Exception {
    indexUsingRestClient();

    Request request = new Request(
            "GET",
            "/store-items/_search");

    JsonObject termJson = new JsonObject().put("name", "IOS smartphone");
    JsonObject matchJson = new JsonObject().put("match", termJson);
    JsonObject queryJson = new JsonObject().put("query", matchJson);
    request.setJsonEntity(queryJson.encode());

    Response response = restClient.performRequest(request);
    String responseBody = EntityUtils.toString(response.getEntity());

    JsonObject json = new JsonObject(responseBody);
    JsonArray hits = json.getJsonObject("hits").getJsonArray("hits");
    List<StoreItem> results = new ArrayList<>(hits.size());

    for (int i = 0; i < hits.size(); i++) {
        JsonObject hit = hits.getJsonObject(i);
        StoreItem fruit = hit.getJsonObject("_source").mapTo(StoreItem.class);
        results.add(fruit);
    }

    assertThat(results)
            .hasSize(1)
            .containsExactlyInAnyOrder(iosPhone);
}
```

我们使用indexUsingRestClient()方法对StoreItem进行索引。然后，我们构建JSON查询并执行搜索请求。我们对每个搜索结果进行反序列化，并验证它是否包含我们的StoreItem。

**至此，我们使用低级REST客户端在Quarkus应用程序中实现了与ElasticSearch的基本集成。如我们所见，我们需要自己处理所有序列化和反序列化过程**。

## 4. ElasticSearch Java客户端

[ElasticSearch Java Client](https://www.baeldung.com/elasticsearch-java#elasticsearch-java-client)是一个更高级的客户端，我们可以使用它的DSL语法来更优雅地创建ElasticSearch查询。

让我们创建一个方法来使用此客户端索引我们的StoreItem：

```java
private void indexUsingElasticsearchClient() throws IOException, InterruptedException {
    androidPhone = new StoreItem();
    androidPhone.setId(UUID.randomUUID().toString());
    androidPhone.setPrice(500L);
    androidPhone.setName("Android smartphone");

    IndexRequest<StoreItem> request = IndexRequest.of(
            b -> b.index("store-items")
                    .id(androidPhone.getId())
                    .document(androidPhone));

    elasticsearchClient.index(request);
}
```

我们构建了另一个StoreItem并创建了[IndexRequest](https://artifacts.elastic.co/javadoc/co/elastic/clients/elasticsearch-java/8.14.3/co/elastic/clients/elasticsearch/core/IndexRequest.html),然后我们调用ElasticSearch Java Client的index()方法来执行请求。

现在让我们搜索已保存的元素：

```java
@Test
void givenElasticsearchClient_whenSearchInStoreItemsByName_thenExpectedDocumentsFound() throws Exception {
    indexUsingElasticsearchClient();
    Query query = QueryBuilders.match()
            .field("name")
            .query(FieldValue.of("Android smartphone"))
            .build()
            ._toQuery();

    SearchRequest request = SearchRequest.of(
            b -> b.index("store-items")
                    .query(query)
    );
    SearchResponse<StoreItem> searchResponse = elasticsearchClient
            .search(request, StoreItem.class);

    HitsMetadata<StoreItem> hits = searchResponse.hits();
    List<StoreItem> results = hits.hits().stream()
            .map(Hit::source)
            .collect(Collectors.toList());

    assertThat(results)
            .hasSize(1)
            .containsExactlyInAnyOrder(androidPhone);
}
```

我们使用indexUsingElasticSearchClient()方法对新文档进行了索引。然后，我们构建了[SearchRequest](https://artifacts.elastic.co/javadoc/co/elastic/clients/elasticsearch-java/8.14.3/co/elastic/clients/elasticsearch/core/SearchRequest.html)，使用ElasticSearch Java客户端执行它，并将所有匹配项收集到StoreItem实例列表中。**我们使用DSL语法创建查询，因此我们不必担心序列化和反序列化**。

## 5. 总结

我们可以看到，Quarkus提供了将我们的应用程序与ElasticSearch集成的出色功能。首先，我们需要做的就是添加扩展依赖并提供几行配置。如果我们想要更多的控制权，我们总是可以自己定义一个客户端Bean。此外，在本文中，我们探讨了如何在Quarkus中使用Elasticsearch的低级REST客户端和高级Java客户端。