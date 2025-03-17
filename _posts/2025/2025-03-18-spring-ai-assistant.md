---
layout: post
title:  使用Spring AI实现AI助手
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

在本教程中，我们将讨论[Spring AI](https://www.baeldung.com/spring-ai)的概念，它可以帮助使用LLM(如ChatGPT、Ollama、Mistreal等)创建AI助手。

越来越多的企业采用人工智能助手来增强现有业务功能的用户体验：

- 回答用户查询
- 根据用户输入执行交易
- 总结长句子和文件

虽然这些只是LLM的一些基本功能，但它们的潜力远不止这些功能。

## 2. Spring AI功能

Spring AI框架提供了一系列令人兴奋的特性来实现AI驱动的功能：

- 可以与底层LLM服务和向量DB无缝集成的接口
- 使用RAG和函数调用API生成上下文感知响应并执行操作
- 结构化输出转换器将LLM响应转换为POJO或机器可读格式(如JSON)
- 通过Advisor API提供的拦截器丰富提示并应用防护措施
- 通过维护对话状态来增强用户参与度

我们也可以将其形象化：

![](/assets/images/2025/springai/springaiassistant01.png)

为了说明其中一些功能，我们将在传统订单管理系统(OMS)中构建一个聊天机器人：

OMS的典型功能包括：

- 创建订单
- 获取用户订单

## 3. 先决条件

首先，我们需要一个OpenAI订阅才能使用其LLM服务。然后在Spring Boot应用程序中，我们将添加Spring AI库的Maven依赖。我们已经在[其他文章](https://www.baeldung.com/tag/spring-ai)中详细介绍了先决条件，因此我们不会进一步详细阐述这个主题。

此外，我们将使用内存中的[HSQLDB](https://www.baeldung.com/spring-boot-hsqldb)数据库来快速入门。让我们创建必要的表并在其中插入一些数据：

```sql
CREATE TABLE User_Order (
                            order_id BIGINT NOT NULL PRIMARY KEY,
                            user_id VARCHAR(20) NOT NULL,
                            quantity INT
);

INSERT INTO User_Order (order_id, user_id, quantity) VALUES (1, 'Jenny', 2);
INSERT INTO User_Order (order_id, user_id, quantity) VALUES (2, 'Mary', 5);
INSERT INTO User_Order (order_id, user_id, quantity) VALUES (3, 'Alex', 1);
INSERT INTO User_Order (order_id, user_id, quantity) VALUES (4, 'John', 3);
INSERT INTO User_Order (order_id, user_id, quantity) VALUES (5, 'Sophia', 4);
--and so on..
```

我们将在Spring application.properties文件中使用一些与初始化HSQLDB和OpenAI客户端相关的标准配置属性：

```properties
spring.datasource.driver-class-name=org.hsqldb.jdbc.JDBCDriver
spring.datasource.url=jdbc:hsqldb:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=none

spring.ai.openai.chat.options.model=gpt-4o-mini
spring.ai.openai.api-key=xxxxxxx
```

**选择适合用例的正确模型是一个复杂的迭代过程，需要大量的反复试验**。不过，对于本文的简单演示应用程序，经济高效的[GPT-4o mini](https://openai.com/index/gpt-4o-mini-advancing-cost-efficient-intelligence/)模型就足够了。

## 4. 函数调用API

此功能是LLM应用程序中流行的[代理概念](https://www.baeldung.com/cs/artificial-intelligence-agents)的支柱之一，**它使应用程序能够执行一系列复杂的精确任务并独立做出决策**。

例如，在旧版订单管理应用中构建聊天机器人可以大有帮助。聊天机器人可以帮助用户提出订单请求、检索订单历史记录，并通过自然语言执行更多操作。

这些是由一个或多个应用程序函数驱动的专业技能，我们在发送给LLM的提示中定义算法，并附带支持函数模式。**LLM收到此模式并确定执行所请求技能的正确函数，随后，它将决定发送给应用程序**。

最后，应用程序执行该函数并向LLM发送回更多信息：

![](/assets/images/2025/springai/springaiassistant02.png)

首先，让我们看一下遗留应用程序的主要组件：

![](/assets/images/2025/springai/springaiassistant03.png)

**OrderManagementService类有两个重要功能：创建订单和获取用户订单历史记录**，这两个功能都使用OrderRepository Bean与数据库集成：

```java
@Service
public class OrderManagementService {
    @Autowired
    private OrderRepository orderRepository;

    public Long createOrder(OrderInfo orderInfo) {
        return orderRepository.save(orderInfo).getOrderID();
    }

    public Optional<List<OrderInfo>> getAllUserOrders(String userID) {
        return orderRepository.findByUserID(userID);
    }
}
```

现在，让我们看看Spring AI如何帮助在遗留应用程序中实现聊天机器人：

![](/assets/images/2025/springai/springaiassistant04.png)

在类图中，OmAiAssistantConfiguration类是一个Spring配置Bean，它注册了函数回调Bean createOrderFn和getUserOrderFn：

```java
@Configuration
public class OmAiAssistantConfiguration {
    @Bean
    @Description("Create an order. The Order ID is identified with orderID. "
            + The order quantity is identified by orderQuantity."
            + "The user is identified by userID. "
            + "The order quantity should be a positive whole number."
            + "If any of the parameters like user id and the order quantity is missing"
            + "then ask the user to provide the missing information.")
    public Function<CreateOrderRequest, Long> createOrderFn(OrderManagementService orderManagementService) {
        return createOrderRequest -> orderManagementService.createOrder(createOrderRequest.orderInfo());
    }
    
    @Bean
    @Description("get all the orders of an user. The user ID is identified with userID.")
    public Function<GetOrderRequest, List<OrderInfo>> getUserOrdersFn(OrderManagementService orderManagementService) {
        return getOrderRequest -> orderManagementService.getAllUserOrders(getOrderRequest.userID()).get();
    }
}
```

**@Description注解有助于生成函数模式。随后，应用程序将此模式作为提示的一部分发送给LLM**。由于函数使用OrderManagementService类的现有方法，因此它提高了可重用性。

此外，CreateOrderRequest和GetOrderRequests是记录类，可帮助Spring AI为下游服务调用生成POJO：

```java
record GetOrderRequest(String userID) {}

record CreateOrderRequest(OrderInfo orderInfo) {}
```

最后，让我们看一下将用户查询发送到LLM服务的新OrderManagementAIAssistant类：

```java
@Service
public class OrderManagementAIAssistant {
    @Autowired
    private ChatModel chatClient;

    public ChatResponse callChatClient(Set<String> functionNames, String promptString) {
        Prompt prompt  = new Prompt(promptString, OpenAiChatOptions
                .builder()
                .withFunctions(functionNames)
                .build()
        );
        return chatClient.call(prompt);
    }
}
```

callChatClient()方法帮助注册Prompt对象中的函数。稍后，它调用ChatModel#call()方法获取来自LLM服务的响应。

## 5. 函数调用场景

对于用户向AI助手提出的查询或指令，我们将介绍几个基本场景：

- LLM决定并确定要执行的一个或多个函数
- LLM抱怨执行该函数的信息不完整
- LLM有条件地执行语句

到目前为止，我们已经讨论了这些概念；因此，接下来，我们将使用此功能来构建聊天机器人。

### 5.1 执行回调函数一次或多次

现在，让我们研究一下当我们使用由用户查询和函数模式组成的提示来调用LLM时它的行为。

我们将从创建订单的示例开始：

```java
void whenOrderInfoProvided_thenSaveInDB(String promptString) {
    ChatResponse response = this.orderManagementAIAssistant
        .callChatClient(Set.of("createOrderFn"), promptString);
    String resultContent = response.getResult().getOutput().getText();
    logger.info("The response from theLLMservice: {}", resultContent);
}
```

令人惊讶的是，通过自然语言，我们得到了预期的结果：

|                            Prompt                            |                                                                                           LLM Response                                                                                            |                         Observation                          |
| :----------------------------------------------------------: |:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:| :----------------------------------------------------------: |
| Create an order with quantity 20 for user id Jenny, and randomly generate a positive whole number for the order ID |                                   The order has been successfully created with the following details: – Order ID: 123456 – User ID: Jenny – Order Quantity: 20                                    | The program creates the order with the information provided in the prompt. |
| Create two orders. The first order is for user id Sophia with quantity 30. The second order is for user id Mary with quantity 40. Randomly generate positive whole numbers for the order IDs. | The orders have been successfully created:1. Order for Sophia: Order ID 1 with a quantity of 30. 2. Order for Mary: Order ID 2 with a quantity of 40.If you need anything else, feel free to ask! | The program creates two orders. The LLM was smart to request to execute the function twice. |


接下来，让我们看看LLM是否可以理解请求获取用户订单详细信息的提示：

```java
void whenUserIDProvided_thenFetchUserOrders(String promptString) {
    ChatResponse response = this.orderManagementAIAssistant
        .callChatClient(Set.of("getUserOrdersFn"), promptString);
    String resultContent = response.getResult().getOutput().getText();
    logger.info("The response from theLLMservice: {}", resultContent);
}
```

结果与上一个案例类似，LLM成功识别要执行的已注册函数。随后，Spring Boot应用程序调用请求的函数并返回一个或多个用户的订单详情。

### 5.2 有条件地执行回调函数

让我们考虑一下只有满足某些条件时才会创建订单的程序：

```java
void whenUserIDProvided_thenCreateOrderIfUserHasLessThanTwoOrders(String promptString) {
    ChatResponse response = this.orderManagementAIAssistant
        .callChatClient(Set.of("getUserOrdersFn", "createOrderFn"), promptString);
    String resultContent = response.getResult()
        .getOutput()
        .getText();
    logger.info("The response from theLLMservice: {}", resultContent);
}
```

我们已经在ChatModel Bean中注册了getUserOrderFn和CreateOrderFn函数。此外，LLM必须确定哪个函数将为请求的操作产生最佳结果。

与以前一样，结果令人满意：

|Prompt   |  Output | Observation  |
|---|---|---|
| Create an order for user id Alex with quantity 25. Don’t create any order if the user has more than 2 orders. While creating the order, randomly generate positive whole numbers for the order ID  | User Alex already has more than 2 orders (specifically, 4 orders). Therefore, I will not create a new order.  | The LLM can conditionally invoke downstream functions based on the output of the previous functions.  |
| Create an order for user id David with quantity 25. Don’t create any order if the user has more than 2 orders. While creating the order, randomly generate positive whole numbers for the order ID  | An order has been successfully created for user ID “David” with an order quantity of 25, and the order ID is 42.  | This time getUserOrderFn returned less than 2 orders for David. Hence, the LLM decided to request executing createOrderFn.  |
|  Create an order with a quantity of 20 | Please provide your user ID to create the order.  | The LLM identified at the beginning that the user ID was missing and aborted further processing.  |


## 6. Spring AI Advisors API

在前面的部分中，我们讨论了应用程序的功能方面。但是，所有功能都存在一些共同的问题，例如：

- 防止用户输入敏感信息
- 记录和审核用户查询
- 保持对话状态
- 丰富提示

幸运的是，Advisors API可以帮助持续解决这些问题，我们已经在一篇[文章](https://www.baeldung.com/spring-ai-advisors)中详细解释了这一点。

## 7. Spring AI结构化输出API和Spring AI RAG

LLM主要以自然语言生成对用户查询的响应消息，然而，下游服务主要理解机器可读格式的消息，如POJO、JSON等。这时，Spring AI生成结构化输出的能力就发挥了重要作用。

就像[Advisors API](https://www.baeldung.com/spring-artificial-intelligence-structure-output)一样，我们已经在其中一篇文章中详细解释了这一点。因此，我们不会对此进行更多讨论，而是转到下一个主题。

有时，应用程序必须查询向量DB以对存储的数据执行语义搜索，以检索其他信息。之后，从向量DB获取的结果将用于提示中，为LLM提供上下文信息。这称为[RAG技术](https://www.baeldung.com/spring-ai-redis-rag-app)，也可以使用Spring AI实现。

## 8. 总结

在本文中，我们探讨了有助于创建AI助手的关键Spring AI功能。Spring AI正在不断发展，具有许多开箱即用的功能。但是，无论编程框架如何，正确选择底层LLM服务和向量DB都至关重要。此外，实现这些服务的最佳配置可能很棘手，需要付出很多努力。尽管如此，这对于应用程序的广泛采用仍然很重要。