---
layout: post
title:  使用Spring MVC测试OAuth安全API(使用Spring Security OAuth遗留堆栈)
category: springsecurity
copyright: springsecurity
excerpt: Spring Security OAuth
---

## 1. 概述

在本文中，我们将展示**如何使用Spring MVC测试支持测试使用OAuth保护的API**。

注意：本文使用的是[Spring OAuth遗留项目](https://spring.io/projects/spring-authorization-server)。

## 2. 授权和资源服务器

有关如何设置授权和资源服务器的教程，请查看之前的文章：[Spring REST API + OAuth2 + AngularJS](https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)。

我们的授权服务器使用JdbcTokenStore并定义了一个id为“fooClientIdPassword”、密码为“secret”的客户端，并支持password授予类型。

资源服务器将/employee URL限制为ADMIN角色。

从Spring Boot版本1.5.0开始，安全适配器优先级高于OAuth资源适配器，因此为了反转顺序，我们必须使用@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)标注WebSecurityConfigurerAdapter类。

否则，Spring将尝试根据Spring Security规则而不是Spring OAuth规则访问请求的URL，并且在使用令牌身份验证时我们会收到403错误。

## 3. 定义示例API

首先，让我们创建一个名为Employee的简单POJO，它具有两个我们将通过API进行操作的属性：

```java
public class Employee {
    private String email;
    private String name;

    // standard constructor, getters, setters
}
```

接下来，让我们定义一个具有两个请求映射的控制器，用于获取和保存Employee对象到列表中：

```java
@Controller
public class EmployeeController {

    private List<Employee> employees = new ArrayList<>();

    @GetMapping("/employee")
    @ResponseBody
    public Optional<Employee> getEmployee(@RequestParam String email) {
        return employees.stream()
                .filter(x -> x.getEmail().equals(email)).findAny();
    }

    @PostMapping("/employee")
    @ResponseStatus(HttpStatus.CREATED)
    public void postMessage(@RequestBody Employee employee) {
        employees.add(employee);
    }
}
```

请记住，**为了使其工作，我们需要一个额外的JDK 8 Jackson模块**。否则，Optional类将无法正确序列化/反序列化。可以从Maven Central下载最新版本的[jackson-datatype-jdk8](https://mvnrepository.com/search?q=jackson-datatype-jdk8)。

## 4. 测试API

### 4.1 设置测试类

为了测试我们的API，我们将创建一个用@SpringBootTest标注的测试类，该测试类使用AuthorizationServerApplication类来读取应用程序配置。

为了使用Spring MVC测试支持测试安全API，我们需要注入WebApplicationContext和Spring Security FilterChain Bean。在运行测试之前，我们将使用这些来获取MockMvc实例：

```java
@RunWith(SpringRunner.class)
@WebAppConfiguration
@SpringBootTest(classes = AuthorizationServerApplication.class)
public class OAuthMvcTest {

    @Autowired
    private WebApplicationContext wac;

    @Autowired
    private FilterChainProxy springSecurityFilterChain;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).addFilter(springSecurityFilterChain).build();
    }
}
```

### 4.2 获取访问令牌

简而言之，**使用OAuth2保护的API期望收到带有Bearer<access_token\>值的授权标头**。

为了发送所需的Authorization标头，我们首先需要通过向/oauth/token端点发出POST请求来获取有效的访问令牌。此端点需要HTTP Basic身份验证，其中包含OAuth客户端的id和secret，以及指定client_id、grant_type、username和password的参数列表。

使用Spring MVC测试支持，可以将参数包装在MultiValueMap中，并使用httpBasic方法发送客户端身份验证。

**让我们创建一个方法，发送一个POST请求来获取令牌并从JSON响应中读取access_token值**：

```java
private String obtainAccessToken(String username, String password) throws Exception {
    MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
    params.add("grant_type", "password");
    params.add("client_id", "fooClientIdPassword");
    params.add("username", username);
    params.add("password", password);

    ResultActions result
            = mockMvc.perform(post("/oauth/token")
                    .params(params)
                    .with(httpBasic("fooClientIdPassword","secret"))
                    .accept("application/json;charset=UTF-8"))
            .andExpect(status().isOk())
            .andExpect(content().contentType("application/json;charset=UTF-8"));

    String resultString = result.andReturn().getResponse().getContentAsString();

    JacksonJsonParser jsonParser = new JacksonJsonParser();
    return jsonParser.parseMap(resultString).get("access_token").toString();
}
```

### 4.3 测试GET和POST请求

可以使用header("Authorization", "Bearer "+ accessToken)方法将访问令牌添加到请求中。

让我们尝试在没有Authorization标头的情况下访问我们的一个安全映射，并验证我们是否收到unauthorized状态码：

```java
@Test
public void givenNoToken_whenGetSecureRequest_thenUnauthorized() throws Exception {
    mockMvc.perform(get("/employee")
                    .param("email", EMAIL))
            .andExpect(status().isUnauthorized());
}
```

我们已指定只有具有ADMIN角色的用户才能访问/employee URL。让我们创建一个测试，在其中获取具有USER角色的用户的访问令牌，并验证我们是否收到forbidden状态码：

```java
@Test
public void givenInvalidRole_whenGetSecureRequest_thenForbidden() throws Exception {
    String accessToken = obtainAccessToken("user1", "pass");
    mockMvc.perform(get("/employee")
                    .header("Authorization", "Bearer " + accessToken)
                    .param("email", "jim@yahoo.com"))
            .andExpect(status().isForbidden());
}
```

接下来，让我们使用有效的访问令牌测试我们的API，通过发送POST请求来创建Employee对象，然后发送GET请求来读取创建的对象：

```java
@Test
public void givenToken_whenPostGetSecureRequest_thenOk() throws Exception {
    String accessToken = obtainAccessToken("admin", "nimda");

    String employeeString = "{\"email\":\"jim@yahoo.com\",\"name\":\"Jim\"}";
        
    mockMvc.perform(post("/employee")
        .header("Authorization", "Bearer " + accessToken)
        .contentType(application/json;charset=UTF-8)
        .content(employeeString)
        .accept(application/json;charset=UTF-8))
        .andExpect(status().isCreated());

    mockMvc.perform(get("/employee")
        .param("email", "jim@yahoo.com")
        .header("Authorization", "Bearer " + accessToken)
        .accept("application/json;charset=UTF-8"))
        .andExpect(status().isOk())
        .andExpect(content().contentType(application/json;charset=UTF-8))
        .andExpect(jsonPath("$.name", is("Jim")));
}
```

## 5. 总结

在此快速教程中，我们演示了如何使用Spring MVC测试支持测试OAuth安全的API。