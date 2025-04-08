---
layout: post
title:  Spring Webflux和Spring Data Reactive中的分页
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 简介

在本文中，我们将探讨分页对于检索信息的意义，将Spring Data Reactive分页与Spring Data进行比较，并通过示例演示如何实现分页。

## 2. 分页的意义

在处理返回大量资源的端点时，分页是一个必不可少的概念。它通过将数据分解为更小、更易于管理的块(称为“页面”)来实现高效的数据检索和呈现。

假设有一个显示产品详细信息的UI页面，其中可能显示10到10000条记录。假设UI旨在从后端获取并显示整个目录，在这种情况下，它将消耗额外的后端资源并导致用户等待很长时间。

**实现分页系统可以显著提升用户体验，与一次性获取整组记录相比，先检索几条记录，然后在请求时提供加载下一组记录的选项会更有效**。

使用分页，后端可以返回包含较小子集(例如10条记录)的初始响应，并使用偏移量或下一页链接检索后续页面。这种方法将获取和显示记录的负载分散到多个页面，从而改善整体应用程序体验。

## 3. Spring Data 与Spring Data Reactive中的分页

[Spring Data](https://www.baeldung.com/spring-data)是Spring框架生态系统中的一个项目，旨在简化和增强Java应用程序中的数据访问。Spring Data提供了一组通用的抽象和功能，通过减少样板代码和推广最佳实践来简化开发过程。

如[Spring Data分页示例](https://www.baeldung.com/spring-data-jpa-pagination-sorting)中所述，[PageRequest](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html)对象接收page、size和sort参数，可用于配置和请求不同的页面。Spring Data提供[PagingAndSortingRepository](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)，它提供使用分页和排序抽象检索实体的方法。Repository方法接收[Pageable](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Pageable.html)和[Sort](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.html)对象，可用于配置返回的[Page](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Page.html)信息。此Page对象包含[totalElements](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Page.html#getTotalElements())和[totalPages](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Page.html#getTotalPages())属性，这些属性通过内部执行其他查询来填充，此信息可用于请求后续页面的信息。

相反，Spring Data Reactive并不完全支持分页，原因在于Spring Reactive支持异步非阻塞，它必须等待(或阻塞)直到返回特定页面大小的所有数据，这不是很高效。但是，**Spring Data Reactive仍然支持[Pageable](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Pageable.html)，我们可以使用[PageRequest](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html)对象对其进行配置以检索特定的数据块，并添加显式查询以获取记录总数**。

**当使用包含有关页面记录的元数据的Spring Data时，我们可以获得响应的[Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)，而不是Page**。

## 4. 基本应用

### 4.1 Spring WebFlux和Spring Data Reactive中分页的实现

在本文中，我们将使用一个简单的Spring R2DBC应用程序，该应用程序通过GET /products分页公开产品信息。

让我们考虑一个简单的Product模型：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Table
public class Product {

    @Id
    @Getter
    private UUID id;

    @NotNull
    @Size(max = 255, message = "The property 'name' must be less than or equal to 255 characters.")
    private String name;

    @NotNull
    private double price;
}
```

我们可以通过传递[Pageable](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Pageable.html)对象从ProductRepository中获取产品列表，该对象包含[Page](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/AbstractPageRequest.html#getPageNumber())和[Size](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/AbstractPageRequest.html#getPageSize())等配置：

```java
@Repository
public interface ProductRepository extends ReactiveSortingRepository<Product, UUID> {
    Flux<Product> findAllBy(Pageable pageable);   
}
```

此查询以Flux而非Page的形式响应结果集，因此需要单独查询记录总数以填充Page响应。

让我们添加一个带有[PageRequest](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html)对象的控制器，该对象还运行一个附加查询来获取记录总数，这是因为我们的Repository不会返回[Page](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Page.html)信息，而是返回Flux<Product\>：

```java
@GetMapping("/products")
public Mono<Page<Product>> findAllProducts(Pageable pageable) {
    return this.productRepository.findAllBy(pageable)
        .collectList()
        .zipWith(this.productRepository.count())
        .map(p -> new PageImpl<>(p.getT1(), pageable, p.getT2()));
}
```

最后，我们必须将查询结果集和最初收到的Pageable对象都发送给[PageImpl](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageImpl.html)，该类具有计算Page信息的辅助方法，其中包括有关页面的元数据，以便获取下一组记录。

现在，当我们尝试访问端点时，我们应该收到包含页面元数据的产品列表：

```json
{
    "content": [
        {
            "id": "cdc0c4e6-d4f6-406d-980c-b8c1f5d6d106",
            "name": "product_A",
            "price": 1
        },
        {
            "id": "699bc017-33e8-4feb-aee0-813b044db9fa",
            "name": "product_B",
            "price": 2
        },
        {
            "id": "8b8530dc-892b-475d-bcc0-ec46ba8767bc",
            "name": "product_C",
            "price": 3
        },
        {
            "id": "7a74499f-dafc-43fa-81e0-f4988af28c3e",
            "name": "product_D",
            "price": 4
        }
    ],
    "pageable": {
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "pageNumber": 0,
        "pageSize": 20,
        "offset": 0,
        "paged": true,
        "unpaged": false
    },
    "last": true,
    "totalElements": 4,
    "totalPages": 1,
    "first": true,
    "numberOfElements": 4,
    "size": 20,
    "number": 0,
    "sort": {
        "sorted": false,
        "unsorted": true,
        "empty": true
    },
    "empty": false
}
```

与Spring Data类似，我们使用某些查询参数导航到不同的页面，并通过扩展[WebMvcConfigurationSupport](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurationSupport.html)来配置默认属性。

让我们将默认页面大小从20更改为100，并通过重写addArgumentResolvers方法将默认页面设置为0：

```java
@Configuration
public class CustomWebMvcConfigurationSupport extends WebMvcConfigurationSupport {

    @Bean
    public PageRequest defaultPageRequest() {
        return PageRequest.of(0, 100);
    }

    @Override
    protected void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        SortHandlerMethodArgumentResolver argumentResolver = new SortHandlerMethodArgumentResolver();
        argumentResolver.setSortParameter("sort");
        PageableHandlerMethodArgumentResolver resolver = new PageableHandlerMethodArgumentResolver(argumentResolver);
        resolver.setFallbackPageable(defaultPageRequest());
        resolver.setPageParameterName("page");
        resolver.setSizeParameterName("size");
        argumentResolvers.add(resolver);
    }
}
```

现在，我们可以从第0页开始发出请求，最大大小为100条记录：

```shell
$ curl --location 'http://localhost:8080/products?page=0&size=50&sort=price,DESC'
```

如果不指定页面和大小参数，默认页面索引为0，每页100条记录。但请求将页面大小设置为50：

```json
{
    "content": [
        // ....
    ],
    "pageable": {
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "pageNumber": 0,
        "pageSize": 50,
        "offset": 0,
        "paged": true,
        "unpaged": false
    },
    "last": true,
    "totalElements": 4,
    "totalPages": 1,
    "first": true,
    "numberOfElements": 4,
    "size": 50,
    "number": 0,
    "sort": {
        "sorted": false,
        "unsorted": true,
        "empty": true
    },
    "empty": false
}
```

## 5. 总结

在本文中，我们了解了Spring Data Reactive分页的独特性，我们还实现了一个返回带有分页的产品列表的端点。