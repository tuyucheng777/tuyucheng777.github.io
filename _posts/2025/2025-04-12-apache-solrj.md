---
layout: post
title:  使用Apache SolrJ在Java中使用Solr的指南
category: apache
copyright: apache
excerpt: Apache SolrJ
---

## 1. 概述

[Apache Solr](http://lucene.apache.org/solr/)是一个基于Lucene构建的开源搜索平台，Apache SolrJ是一个基于Java的Solr客户端，它提供了索引、查询和删除文档等主要搜索功能的接口。

在本文中，我们将探讨如何使用SolrJ与Apache Solr服务器交互。

## 2. 设置

为了在你的机器上安装Solr服务器，请参阅[Solr快速入门指南](http://lucene.apache.org/solr/quickstart.html)。

安装过程很简单——只需下载zip/tar包，解压内容，然后从命令行启动服务器即可。在本文中，我们将创建一个核心名为“bigboxstore”的Solr服务器：

```shell
bin/solr start
bin/solr create -c 'bigboxstore'
```

默认情况下，Solr监听8983端口以接收HTTP查询，你可以通过在浏览器中打开http://localhost:8983/solr/#/bigboxstore URL并观察Solr仪表板来验证它是否已成功启动。

## 3. Maven配置

现在我们已经启动并运行了Solr服务器，让我们直接进入SolrJ Java客户端。要在项目中使用SolrJ，你需要在pom.xml文件中声明以下Maven依赖：

```xml
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr-solrj</artifactId>
    <version>6.4.0</version>
</dependency>
```

始终可以找到由[Maven Central](https://mvnrepository.com/artifact/org.apache.solr/solr-solrj)托管的最新版本。

## 4. Apache SolrJ Java API

让我们通过连接到Solr服务器来启动SolrJ客户端：

```java
String urlString = "http://localhost:8983/solr/bigboxstore";
HttpSolrClient solr = new HttpSolrClient.Builder(urlString).build();
solr.setParser(new XMLResponseParser());
```

注意：**SolrJ使用二进制格式(而非XML)作为其默认响应格式**，为了与Solr兼容，需要像上文所示那样显式调用setParser()来解析XML格式。更多详细信息请参见[此处](https://cwiki.apache.org/confluence/display/solr/Using+SolrJ)。

### 4.1 索引文档

让我们使用SolrInputDocument定义要索引的数据，并使用add()方法将其添加到我们的索引中：

```java
SolrInputDocument document = new SolrInputDocument();
document.addField("id", "123456");
document.addField("name", "Kenmore Dishwasher");
document.addField("price", "599.99");
solr.add(document);
solr.commit();
```

**注意：任何修改Solr数据库的操作都需要在操作后跟commit()**。

### 4.2 使用Bean进行索引

**你还可以使用Bean索引Solr文档**，让我们定义一个ProductBean，其属性用@Field标注：

```java
public class ProductBean {

    String id;
    String name;
    String price;

    @Field("id")
    protected void setId(String id) {
        this.id = id;
    }

    @Field("name")
    protected void setName(String name) {
        this.name = name;
    }

    @Field("price")
    protected void setPrice(String price) {
        this.price = price;
    }

    // getters and constructor omitted for space
}
```

然后，让我们将Bean添加到我们的索引中：

```java
solrClient.addBean( new ProductBean("888", "Apple iPhone 6s", "299.99") );
solrClient.commit();
```

### 4.3 根据字段和ID查询索引文档

让我们通过使用SolrQuery查询我们的Solr服务器来验证文档是否已添加。

来自服务器的QueryResponse将包含与任何格式为field:value的查询匹配的SolrDocument对象列表，在此示例中，我们按价格查询：

```java
SolrQuery query = new SolrQuery();
query.set("q", "price:599.99");
QueryResponse response = solr.query(query);

SolrDocumentList docList = response.getResults();
assertEquals(docList.getNumFound(), 1);

for (SolrDocument doc : docList) {
     assertEquals((String) doc.getFieldValue("id"), "123456");
     assertEquals((Double) doc.getFieldValue("price"), (Double) 599.99);
}
```

一个更简单的选项是使用getById()通过Id进行查询，如果找到匹配项，它将仅返回一个文档：

```java
SolrDocument doc = solr.getById("123456");
assertEquals((String) doc.getFieldValue("name"), "Kenmore Dishwasher");
assertEquals((Double) doc.getFieldValue("price"), (Double) 599.99);
```

### 4.4 删除文档

当我们想从索引中删除文档时，我们可以使用deleteById()并验证它已被删除：

```java
solr.deleteById("123456");
solr.commit();
SolrQuery query = new SolrQuery();
query.set("q", "id:123456");
QueryResponse response = solr.query(query);
SolrDocumentList docList = response.getResults();
assertEquals(docList.getNumFound(), 0);
```

我们还可以选择deleteByQuery()，因此让我们尝试删除具有特定名称的任何文档：

```java
solr.deleteByQuery("name:Kenmore Dishwasher");
solr.commit();
SolrQuery query = new SolrQuery();
query.set("q", "id:123456");
QueryResponse response = solr.query(query);
SolrDocumentList docList = response.getResults();
assertEquals(docList.getNumFound(), 0);
```

## 5. 总结

在这篇简短的文章中，我们了解了如何使用SolrJ Java API与Apache Solr全文搜索引擎执行一些常见的交互。
