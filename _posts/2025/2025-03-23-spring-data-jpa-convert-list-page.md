---
layout: post
title:  使用Spring Data JPA将List转换为Page
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在本教程中，我们将了解如何使用[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)将List<Object\>转换为Page<Object\>。**在Spring Data JPA应用程序中，通常以[可分页](https://www.baeldung.com/spring-data-jpa-pagination-sorting)的方式从数据库检索数据。但是，在某些情况下，我们需要将实体列表转换为Page对象以在可分页端点中使用。例如，我们可能希望从外部API检索数据或在内存中处理数据**。

我们将设置一个简单的示例来帮助我们可视化数据流和转换。我们将把系统分为RestController、Service和[Repository](https://www.baeldung.com/spring-data-repositories)层，并了解如何使用Spring Data JPA提供的分页抽象将从数据库检索到的大量数据List<Object\>转换为更小、更有条理的Page。最后，我们将编写一些测试来观察分页的实际效果。

## 2. Spring Data JPA中的关键分页抽象

让我们简要了解一下Spring Data JPA提供的用于生成分页数据的关键抽象。

### 2.1 Page

Page是Spring Data为实现分页而提供的关键接口之一，它提供了一种以分页格式表示和管理数据库查询返回的大型结果集的方法。

然后**我们可以使用Page对象向用户显示所需数量的记录以及导航至后续页面的链接**。

Page封装了页面内容等详细信息，以及涉及分页详细信息的元数据，例如页码和页面大小、是否有下一页或上一页、剩余多少个元素以及页面和元素的总数。

### 2.2 Pageable

Pageable是分页信息的抽象接口，实现此接口的具体类是PageRequest。它表示分页元数据，例如当前页码、每页元素数和排序条件。它是Spring Data JPA中的一个接口，**它提供了一种方便的方法来为查询指定分页信息**，或者在我们的例子中，将分页信息与内容捆绑在一起，以从List<Object\>创建Page<Object\>。

### 2.3 PageImpl

最后，还有PageImpl类，**它提供了Page接口的便捷实现，可用于表示查询结果页面，包括分页元数据**。它通常与Spring Data的Repository接口和分页机制结合使用，以可分页的方式检索和操作数据。

现在我们已经对所涉及的组件有了基本的了解，让我们建立一个简单的示例。

## 3. 示例设置

让我们考虑一个简单的客户信息微服务示例，它有一个REST端点，可以根据请求参数获取分页的客户数据。我们将首先在[POM](https://www.baeldung.com/maven)中设置所需的依赖。所需的依赖是[spring-boot-starter-data-jpa](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)和[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)，我们还将添加[spring-boot-starter-test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)以用于测试目的：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependendencies>
```

接下来让我们设置[REST控制器](https://www.baeldung.com/spring-controller-vs-restcontroller)。

### 3.1 CustomerController

首先，让我们添加带有请求参数的适当方法来驱动服务层的逻辑：

```java
@GetMapping("/api/customers")
public ResponseEntity<Page<Customer>> getCustomers(@RequestParam(defaultValue = "0") int page, @RequestParam(defaultValue = "10") int size) {
    Page<Customer> customerPage = customerService.getCustomers(page, size);
    HttpHeaders headers = new HttpHeaders();
    headers.add("X-Page-Number", String.valueOf(customerPage.getNumber()));
    headers.add("X-Page-Size", String.valueOf(customerPage.getSize()));

    return ResponseEntity.ok()
            .headers(headers)
            .body(customerPage);
}
```

**在这里，我们可以看到getCustomers方法需要Page<Customer\>类型的ResponseEntity**。

### 3.2 CustomerService

接下来，让我们设置与Repository交互的服务类，将数据转换为所需的Page，并将其返回给Controller：

```java
public Page<Customer> getCustomers(int page, int size) {

    List<Customer> allCustomers = customerRepository.findAll();
    //... logic to convert the List<Customer> to Page<Customer>
    //... return Page<Customer>
}
```

这里我们省略了细节，只关注服务调用JPA Repository并获取可能很大的客户数据集作为List<Customer\>的事实。接下来，让我们详细了解如何使用JPA提供的API将此列表转换为Page<Customer\>。

## 4. 将List<Customer\>转换为Page<Customer\>

现在，让我们详细说明CustomerService如何将从CustomerRepository收到的List<Customer\>转换为Page对象。本质上，**从数据库检索所有客户的列表后，我们要使用PageRequest工厂方法创建一个Pageable对象**：

```java
private Pageable createPageRequestUsing(int page, int size) {
    return PageRequest.of(page, size);
}
```

请注意，这些page和size参数是作为请求参数从我们的CustomerRestController传递给CustomerService的参数。

然后，我们将把大的Customer列表拆分成一个子列表。我们需要知道开始和结束索引，基于此我们可以创建一个子列表。这可以使用Pageable对象的getOffset()和getPageSize()方法计算：

```java
int start = (int) pageRequest.getOffset();
```

接下来我们获取结束索引：

```java
int end = Math.min((start + pageRequest.getPageSize()), allCustomers.size());
```

**该子列表将构成我们的Page对象的内容**：

```java
 List<Customer> pageContent = allCustomers.subList(start, end);
```

最后，我们将创建一个PageImpl实例，**它将封装pageContent和pageRequest以及List<Customer\>的总大小**：

```java
new PageImpl<>(pageContent, pageRequest, allCustomers.size());
```

让我们把所有的部分放在一起：

```java
public Page<Customer> getCustomers(int page, int size) {
    Pageable pageRequest = createPageRequestUsing(page, size);

    List<Customer> allCustomers = customerRepository.findAll();
    int start = (int) pageRequest.getOffset();
    int end = Math.min((start + pageRequest.getPageSize()), allCustomers.size());

    List<Customer> pageContent = allCustomers.subList(start, end);
    return new PageImpl<>(pageContent, pageRequest, allCustomers.size());
}
```

## 5. 将Page<Customer\>转换为List<Customer\>

分页是处理大量结果集时需要考虑的重要功能，通过将数据分解为称为页面的较小部分，它可以显著增强数据组织。

Spring数据提供了一种支持分页的便捷方法，要在查询方法中添加分页，我们需要更改签名以接收Pageable对象作为参数并返回Page<T\>而不是List<T\>。

通常，返回的Page对象表示结果的一部分。它包含有关元素列表、所有元素的数量以及页面数量的信息。

例如，让我们看看如何获取给定页面的客户列表：

```java
public List<Customer> getCustomerListFromPage(int page, int size) {
    Pageable pageRequest = createPageRequestUsing(page, size);
    Page<Customer> allCustomers = customerRepository.findAll(pageRequest);

    return allCustomers.hasContent() ? allCustomers.getContent() : Collections.emptyList();
}
```

我们可以看到，我们使用了相同的createPageRequestUsing()工厂方法来创建Pageable对象。然后，我们调用getContent()方法将返回的页面作为客户列表获取。

请注意，我们在调用getContent()之前使用了hasContent()方法来检查页面是否有内容。

## 6. 测试服务

让我们编写一个快速测试来查看List<Customer\>是否拆分为Page<Customer\>，以及页面大小和页码是否正确。我们将Mock customerRepository.findAll()方法以返回大小为20的Customer列表。

在设置中，我们只需在调用findAll()时提供此列表：

```java
@BeforeEach
void setup() {
    when(customerRepository.findAll()).thenReturn(ALL_CUSTOMERS);
}
```

在这里，我们构建一个参数化测试并对内容、内容大小、总元素和总页数进行断言：

```java
@ParameterizedTest
@MethodSource("testIO")
void givenAListOfCustomers_whenGetCustomers_thenReturnsDesiredDataAlongWithPagingInformation(int page, int size, List<String> expectedNames, long expectedTotalElements, long expectedTotalPages) {
    Page<Customer> customers = customerService.getCustomers(page, size);
    List<String> names = customers.getContent()
            .stream()
            .map(Customer::getName)
            .collect(Collectors.toList());

    assertEquals(expectedNames.size(), names.size());
    assertEquals(expectedNames, names);
    assertEquals(expectedTotalElements, customers.getTotalElements());
    assertEquals(expectedTotalPages, customers.getTotalPages())
}
```

最后，本次参数化测试的测试数据输入和输出为：

```java
private static Collection<Object[]> testIO() {
    return Arrays.asList(
            new Object[][] {
                    { 0, 5, PAGE_1_CONTENTS, 20L, 4L },
                    { 1, 5, PAGE_2_CONTENTS, 20L, 4L },
                    { 2, 5, PAGE_3_CONTENTS, 20L, 4L },
                    { 3, 5, PAGE_4_CONTENTS, 20L, 4L },
                    { 4, 5, EMPTY_PAGE, 20L, 4L } }
    );
}
```

每个测试使用不同的页面大小对(0、1、2、3、4)运行服务方法，每个页面预计包含5个元素。我们预计页面总数为4，因为原始列表的总大小为20。最后，每个页面预计包含5个元素。

现在，我们将添加将Page<Customer\>转换为List<Customer\>的测试用例。首先，让我们测试Page对象不为空的场景：

```java
@Test
void givenAPageOfCustomers_whenGetCustomerList_thenReturnsList() {
    Page<Customer> pagedResponse = new PageImpl<Customer>(ALL_CUSTOMERS.subList(0, 5));
    when(customerRepository.findAll(any(Pageable.class))).thenReturn(pagedResponse);

    List<Customer> customers = customerService.getCustomerListFromPage(0, 5);
    List<String> customerNames = customers.stream()
            .map(Customer::getName)
            .collect(Collectors.toList());

    assertEquals(PAGE_1_CONTENTS.size(), customers.size());
    assertEquals(PAGE_1_CONTENTS, customerNames);
}
```

这里，我们Mock了customerRepository.findAll(pageRequest)方法的调用，以返回一个包含部分ALL_CUSTOMERS列表的Page对象。因此，返回的客户列表与给定Page对象中包装的客户列表相同。

接下来，让我们看看如果customerRepository.findAll(pageRequest)返回一个空页面会发生什么：

```java
@Test
void givenAnEmptyPageOfCustomers_whenGetCustomerList_thenReturnsEmptyList() {
    Page<Customer> emptyPage = Page.empty();
    when(customerRepository.findAll(any(Pageable.class))).thenReturn(emptyPage);
    List<Customer> customers = customerService.getCustomerListFromPage(0, 5);

    assertThat(customers).isEmpty();
}
```

毫不奇怪，返回的列表是空的。

## 7. 测试控制器

最后，我们还要测试一下Controller，以确保我们能以JSON格式获取ResponseEntity<Page<Customer>\>。我们将使用MockMVC向GET端点发送请求，并期望分页响应具有预期的参数：

```java
@Test
void givenTotalCustomers20_whenGetRequestWithPageAndSize_thenPagedReponseIsReturnedFromDesiredPageAndSize() throws Exception {
    MvcResult result = mockMvc.perform(get("/api/customers?page=1&size=5"))
            .andExpect(status().isOk())
            .andReturn();

    MockHttpServletResponse response = result.getResponse();

    JSONObject jsonObject = new JSONObject(response.getContentAsString());
    assertThat(jsonObject.get("totalPages")).isEqualTo(4);
    assertThat(jsonObject.get("totalElements")).isEqualTo(20);
    assertThat(jsonObject.get("number")).isEqualTo(1);
    assertThat(jsonObject.get("size")).isEqualTo(5);
    assertThat(jsonObject.get("content")).isNotNull();
}
```

本质上，我们使用MockMvc实例来模拟对/api/customers端点的HTTP GET请求，我们提供查询参数page = 1和size = 5，然后我们期望得到成功的响应和包含页面元数据和内容的正文。

最后，让我们快速了解一下将List<Customer\>转换为Page<Customer\>如何有利于API设计和使用。

## 8. 使用Page<Customer\>优于List<Customer\>

在我们的示例中，选择返回Page<Customer\>而不是整个List<Customer\>作为API响应，根据用例的不同，可能会带来一些好处。**其中一个好处是优化网络流量和处理**。本质上，如果底层数据源返回大量客户列表，则将其转换为Page<Customer\>允许客户端仅请求特定页面的结果，而不是整个列表，这可以简化客户端的处理并减少网络负载。

**此外，通过返回Page<Customer\>，API提供了一种标准化的响应格式，客户端可以轻松理解和使用**。Page<Customer\>对象包含所请求页面的客户列表以及元数据(例如总页数和每页项目数)。

**最后，将对象列表转换为页面为API设计提供了灵活性**。例如，API可以允许客户端按不同字段对结果进行排序。我们还可以根据条件筛选结果，甚至为每个客户返回字段子集。

## 9. 总结

在本教程中，我们使用Spring Data JPA来处理List<Object\>和Page<Object\>之间的转换。我们使用了Spring Data JPA提供的API，包括Page、Pageable和PageImpl类。最后，我们简要介绍了使用Page<Object\>而不是List<Object\>的一些好处。

总之，在REST端点中将List<Object\>转换为Page<Object\>提供了一种更高效、标准化和灵活的方式来处理大型数据集并在API中实现分页。