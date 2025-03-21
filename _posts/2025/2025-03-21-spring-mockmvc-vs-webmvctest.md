---
layout: post
title:  使用MockMvc和SpringBootTest与使用WebMvcTest
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

让我们深入研究[Spring Boot测试](https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)的世界！在本教程中，我们将深入研究@SpringBootTest和@WebMvcTest注解。我们将探讨何时以及为何使用它们以及它们如何协同工作以测试我们的Spring Boot应用程序。此外，我们将揭示MockMvc的内部工作原理以及它如何在集成测试中与这两个注解交互。

## 2. 什么是@WebMvcTest和@SpringBootTest

@WebMvcTest注解用于创建MVC(或更具体地说是控制器)相关测试，它还可以配置为针对特定控制器进行测试，它主要加载并简化Web层的测试。

@SpringBootTest注解用于通过加载完整的应用程序上下文(如使用@Component和@Service标注的类、数据库连接等)来创建测试环境，它查找主类(具有@SpringBootApplication注解)并使用它来启动应用程序上下文。

这两个注解都是在Spring Boot 1.4中引入的。

## 3. 项目设置

在本教程中，我们将创建两个类，即SortingController和SortingService。SortingController接收带有整数列表的请求，并使用具有业务逻辑的辅助类SortingService对列表进行排序。

我们将使用构造函数注入来获取SortingService依赖项，如下所示：

```java
@RestController
public class SortingController {
    private final SortingService sortingService;

    public SortingController(SortingService sortingService){
        this.sortingService=sortingService;
    }    // ...
}
```

让我们声明一个GET方法来检查服务器是否正在运行，这也有助于我们在测试期间探索注解的工作：

```java
@GetMapping
public ResponseEntity<String> helloWorld(){
    return ResponseEntity.ok("Hello, World!");
}
```

接下来，我们还将有一个POST方法，该方法将数组作为JSON主体，并在响应中返回已排序的数组。测试此类方法将有助于我们了解MockMvc的使用：

```java
@PostMapping
public ResponseEntity<List<Integer>> sort(@RequestBody List<Integer> arr){
    return ResponseEntity.ok(sortingService.sortArray(arr));
}
```

## 4. 比较@SpringBootTest和@WebMvcTest 

@WebMvcTest注解位于org.springframework.boot.test.autoconfigure.web.servlet包中，而@SpringBootTest位于org.springframework.boot.test.context中。假设我们计划测试我们的应用程序，Spring Boot默认会将必要的依赖添加到我们的项目中。在类级别，我们可以一次使用其中任何一个。

### 4.1 使用MockMvc

在@SpringBootTest上下文中，MockMvc将自动从控制器调用实际的服务实现，服务层Bean将在应用程序上下文中可用。要在测试中使用MockMvc，我们需要添加@AutoConfigureMockMvc注解。此注解创建MockMvc的一个实例，将其注入mockMvc变量，并使其准备好进行测试，而无需手动配置：

```java
@AutoConfigureMockMvc
@SpringBootTest
class SortingControllerIntegrationTest {
    @Autowired
    private MockMvc mockMvc;
}
```

在@WebMvcTest中，MockMvc会伴随服务层的[@MockBean](https://www.baeldung.com/java-spring-mockbeans)来Mock服务层响应，而无需调用真正的服务。而且，服务层Bean不包含在应用程序上下文中，它由@AutoConfigureMockMvc默认提供：

```java
@WebMvcTest
class SortingControllerUnitTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private SortingService sortingService;
}
```

**注意**：当使用带有webEnvironment=RANDOM_PORT的@SpringBootTest时，请谨慎使用MockMvc，因为MockMvc会尝试确保处理Web请求所需的一切都已准备就绪，并且当webEnvironment=RANDOM_PORT尝试启动Servlet容器时，不会启动任何Servlet容器(处理传入的HTTP请求并生成响应)。如果结合使用，它们会相互矛盾。

### 4.2 什么是自动配置？

在@WebMvcTest中，Spring Boot会自动配置MockMvc实例、DispatcherServlet、HandlerMapping、HandlerAdapter和ViewResolvers。它还会扫描@Controller、@ControllerAdvice、@JsonComponent、Converter、GenericConverter、Filter、WebMvcConfigurer和HandlerMethodArgumentResolver组件。总体来说，它会自动配置与Web层相关的组件。

@SpringBootTest加载@SpringBootApplication(SpringBootConfiguration + EnableAutoConfiguration + ComponentScan)所做的一切，即一个成熟的应用程序上下文。它甚至加载application.properties文件和与Profile相关的信息，它还允许像使用@Autowired一样注入Bean。

### 4.3 轻量级还是重量级

**我们可以说@SpringBootTest是重量级的，因为它默认主要配置为集成测试，除非我们想使用任何Mock**。它还具有应用程序上下文中的所有Bean，这也是它与其他测试相比速度较慢的原因。

**另一方面，@WebMvcTest更加独立，只关注MVC层，它非常适合单元测试**。我们也可以针对一个或多个控制器，它在应用程序上下文中具有有限数量的Bean。此外，在运行时，我们可以观察到测试用例完成的相同时间差异(即使用@WebMvcTest的运行时间更短)。

### 4.4 测试期间的Web环境

当我们启动一个真正的应用程序时，我们通常点击“http://localhost:8080”来访问我们的应用程序。为了在测试期间模拟相同的场景，我们使用webEnvironment。并使用它为我们的测试用例定义一个端口(类似于URL中的8080)。@SpringBootTest可以介入模拟的webEnvironment(WebEnvironment.MOCK)或真实的webEnvironment(WebEnvironment.RANDOM_PORT)，而@WebMvcTest仅提供模拟的测试环境。

以下是@SpringBootTest与WebEnvironment的代码示例：

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SortingControllerWithWebEnvironmentIntegrationTest {
    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private ObjectMapper objectMapper;
}
```

现在让我们实际使用它们来编写测试用例，以下是GET方法的测试用例： 

```java
@Test
void whenHelloWorldMethodIsCalled_thenReturnSuccessString() {
    ResponseEntity<String> response = restTemplate.getForEntity("http://localhost:" + port + "/", String.class);
    Assertions.assertEquals(HttpStatus.OK, response.getStatusCode());
    Assertions.assertEquals("Hello, World!", response.getBody());
}
```

以下是检查POST方法正确性的测试用例：

```java
@Test
void whenSortMethodIsCalled_thenReturnSortedArray() throws Exception {
    List<Integer> input = Arrays.asList(5, 3, 8, 1, 9, 2);
    List<Integer> sorted = Arrays.asList(1, 2, 3, 5, 8, 9);

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);

    ResponseEntity<List> response = restTemplate.postForEntity("http://localhost:" + port + "/",
            new HttpEntity<>(objectMapper.writeValueAsString(input), headers),
            List.class);

    Assertions.assertEquals(HttpStatus.OK, response.getStatusCode());
    Assertions.assertEquals(sorted, response.getBody());
}
```

### 4.5 依赖

@WebMvcTest不会自动检测控制器所需的依赖项，因此我们必须对其进行Mock。而@SpringBootTest可以自动执行此操作。

这里我们可以看到我们使用了@MockBean因为我们从控制器内部调用服务：

```java
@WebMvcTest
class SortingControllerUnitTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private SortingService sortingService;
}
```

现在让我们看一个使用MockMvc和一个Mock Bean的测试示例：

```java
@Test
void whenSortMethodIsCalled_thenReturnSortedArray() throws Exception {
    List<Integer> input = Arrays.asList(5, 3, 8, 1, 9, 2);
    List<Integer> sorted = Arrays.asList(1, 2, 3, 5, 8, 9);

    when(sortingService.sortArray(input)).thenReturn(sorted);
    mockMvc.perform(post("/").contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(input)))
            .andExpect(status().isOk())
            .andExpect(content().json(objectMapper.writeValueAsString(sorted)));
}
```

这里我们使用when().thenReturn()来Mock服务类中的sortArray()函数，不这样做会导致NullPointerException。

### 4.6 自定义

**@SpringBootTest在大多数情况下不是自定义的好选择，但@WebMvcTest可以自定义为仅与有限的控制器类一起使用**。在下面的例子中，我特别提到了SortingController类。因此，只有一个控制器及其依赖项在应用程序中注册：

```java
@WebMvcTest(SortingController.class)
class SortingControllerUnitTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private SortingService sortingService;

    @Autowired
    private ObjectMapper objectMapper;
}
```

## 5. 总结

@SpringBootTest和@WebMvcTest各有不同的用途。@WebMvcTest专为MVC相关测试而设计，专注于Web层，并为特定控制器提供简单的测试。另一方面，@SpringBootTest通过加载完整的应用程序上下文(包括@Components、DB连接和@Service)来创建测试环境，使其适合集成和系统测试，类似于生产环境。

在使用MockMvc时，@SpringBootTest从控制器内部调用实际的服务实现，而@WebMvcTest伴随@MockBean用于Mock服务层响应而无需调用实际服务。