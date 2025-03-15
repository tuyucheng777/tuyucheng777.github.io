---
layout: post
title:  Java中使用Testcontainers和MongoDB
category: test-lib
copyright: test-lib
excerpt: Testcontainers
---

## 1. 概述

**测试容器帮助我们在运行测试之前启动容器，并通过在代码中定义它们来停止它们**。

在本教程中，我们将了解如何使用[MongoDB](https://www.baeldung.com/spring-data-mongodb-tutorial)配置[Testcontainers](https://www.baeldung.com/docker-test-containers)。接下来，我们将了解如何为我们的测试创建基础集成。最后，我们将学习如何使用库进行数据访问层和MongoDB应用程序集成测试。

## 2. 配置

为了在我们的测试中使用带有MongoDB的Testcontainers，我们需要在具有测试范围的pom.xml文件中添加以下依赖项：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.18.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.18.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mongodb</artifactId>
    <version>1.18.3</version>
    <scope>test</scope>
</dependency>
```

**第一个是[核心依赖项](https://mvnrepository.com/artifact/org.testcontainers/testcontainers)，它提供Testcontainers的主要功能，例如启动和停止容器。[下一个](https://mvnrepository.com/artifact/org.testcontainers/junit-jupiter)依赖项是JUnit 5扩展，[最后一个](https://mvnrepository.com/artifact/org.testcontainers/mongodb)依赖是MongoDB模块**。

记住，需要在我们的机器上安装[Docker](https://www.baeldung.com/ops/docker-guide)来运行MongoDB容器。

### 2.1 创建模型

让我们首先使用@Document注解创建与Product表对应的实体：

```java
@Document(collection = "Product")
public class Product {

    @Id
    private String id;

    private String name;

    private String description;

    private double price;

    // standard constructor, getters, setters
}
```

### 2.2 创建Repository

然后，我们将创建从MongoRepository扩展的ProductRepository类：

```java
@Repository
public interface ProductRepository extends MongoRepository<Product, String> {

    Optional<Product> findByName(String name);
}
```

### 2.3 创建REST控制器

最后，让我们通过创建一个控制器来与Repository交互，从而公开REST API ：

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductRepository productRepository;

    public ProductController(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @PostMapping
    public String createProduct(@RequestBody Product product) {
        return productRepository.save(product)
                .getId();
    }

    @GetMapping("/{id}")
    public Product getProduct(@PathVariable String id) {
        return productRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Product not found"));
    }
}
```

## 3. Testcontainers MongoDB集成基础

我们将创建一个抽象基类，该基类扩展到需要在运行测试之前和之后启动和停止MongoDB容器的所有类：

```java
@Testcontainers
@SpringBootTest(classes = MongoDbTestContainersApplication.class)
public abstract class AbstractBaseIntegrationTest {

    @Container
    static MongoDBContainer mongoDBContainer = new MongoDBContainer("mongo:7.0").withExposedPorts(27017);

    @DynamicPropertySource
    static void containersProperties(DynamicPropertyRegistry registry) {
        mongoDBContainer.start();
        registry.add("spring.data.mongodb.host", mongoDBContainer::getHost);
        registry.add("spring.data.mongodb.port", mongoDBContainer::getFirstMappedPort);
    }
}
```

**我们添加了@Testcontainers注解以在我们的测试中启用Testcontainers支持，并添加了@SpringBootTest注解以启动Spring Boot应用程序上下文**。

我们还定义了一个MongoDB容器字段，该字段使用mongo:7.0 Docker镜像启动MongoDB容器并公开端口27017，@Container注解在运行测试之前启动MongoDB容器。

### 3.1 数据访问层集成测试

数据访问层集成测试我们的应用程序与数据库之间的交互，我们将为MongoDB数据库创建一个简单的数据访问层并为其编写集成测试。

让我们创建扩展AbstractBaseIntegrationTest类的数据访问集成测试类：

```java
public class ProductDataLayerAccessIntegrationTest extends AbstractBaseIntegrationTest {

    @Autowired
    private ProductRepository productRepository;

    // ..
    
}
```

现在，我们可以为数据访问层编写集成测试：

```java
@Test
public void givenProductRepository_whenSaveAndRetrieveProduct_thenOK() {
    Product product = new Product("Milk", "1L Milk", 10);

    Product createdProduct = productRepository.save(product);
    Optional<Product> optionalProduct = productRepository.findById(createdProduct.getId());

    assertThat(optionalProduct.isPresent()).isTrue();

    Product retrievedProduct = optionalProduct.get();
    assertThat(retrievedProduct.getId()).isEqualTo(product.getId());
}

@Test
public void givenProductRepository_whenFindByName_thenOK() {
    Product product = new Product("Apple", "Fruit", 10);

    Product createdProduct = productRepository.save(product);
    Optional<Product> optionalProduct = productRepository.findByName(createdProduct.getName());

    assertThat(optionalProduct.isPresent()).isTrue();

    Product retrievedProduct = optionalProduct.get();
    assertThat(retrievedProduct.getId()).isEqualTo(product.getId());
}
```

我们创建了两个场景：第一个场景保存并检索产品，第二个场景按名称查找产品。两个测试都与Testcontainers创建的MongoDB数据库进行交互。

### 3.2 应用程序集成测试

应用程序集成测试用于测试不同应用程序组件之间的交互，我们将创建一个使用我们之前创建的数据访问层的简单应用程序，并为其编写集成测试。

让我们创建扩展AbstractBaseIntegrationTest类的应用程序集成测试类：

```java
@AutoConfigureMockMvc
public class ProductIntegrationTest extends AbstractBaseIntegrationTest {

    @Autowired
    private MockMvc mvc;
    private ObjectMapper objectMapper = new ObjectMapper();

    // ..
}
```

**我们需要@AutoConfigureMockMvc注解来在我们的测试中启用MockMvc支持，并需要MockMvc字段对我们的应用程序执行HTTP请求**。

现在，我们可以为我们的应用程序编写集成测试：

```java
@Test
public void givenProduct_whenSave_thenGetProduct() throws Exception {
    MvcResult mvcResult = mvc.perform(post("/products").contentType("application/json")
        .content(objectMapper.writeValueAsString(new Product("Banana", "Fruit", 10))))
        .andExpect(status().isOk())
        .andReturn();

    String productId = mvcResult.getResponse()
        .getContentAsString();

    mvc.perform(get("/products/" + productId))
        .andExpect(status().isOk());
}
```

我们开发了一个测试来保存产品，然后使用HTTP检索它。此过程涉及将数据存储在MongoDB数据库中，该数据库由Testcontainers初始化。

## 4. 总结

在本文中，我们学习了如何使用MongoDB配置Testcontainers，并为数据访问层和使用MongoDB的应用程序编写集成测试。

我们首先使用MongoDB配置库来进行设置，接下来，我们为测试创建了一个基础集成测试。

最后，我们编写了使用Testcontainers提供的MongoDB数据库的数据访问和应用程序集成测试。