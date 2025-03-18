---
layout: post
title:  Mockito Answer API：根据参数返回值
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 简介

Mock是一种测试技术，它用具有预定义行为的对象替换真实组件。这使开发人员能够隔离和测试特定组件，而无需依靠依赖项。**Mock是具有预定义方法调用答案的对象，这些答案也具有执行预期**。

在本教程中，我们将了解如何在Mockito提供的Answer API的帮助下，使用[Mockito](https://www.baeldung.com/mockito-series)根据不同的角色测试基于用户角色的身份验证服务。

## 2. Maven依赖

在阅读本文之前，添加[Mockito依赖](https://mvnrepository.com/artifact/org.mockito/mockito-core)是必不可少的。

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.14.2</version>
    <scope>test</scope>
</dependency>
```

让我们添加[JUnit 5](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)依赖，因为我们在单元测试的某些部分需要它。

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.11.3</version>
    <scope>test</scope>
</dependency>
```

## 3. Answer API介绍

**Answer API允许我们通过指定Mock方法在调用时应返回的内容来自定义Mock方法的行为**，当我们希望Mock根据输入参数提供不同的响应时，这将非常有用。让我们探索更多主题，以更清楚地理解概念。

Mockito中的Answer API会拦截Mock对象上的方法调用，并将这些调用重定向到自定义行为。此过程涉及内部步骤，允许Mockito Mock复杂的行为而无需修改底层代码。让我们详细探索这一点，从Mock创建到方法调用拦截。

### 3.1 使用thenAnswer()创建Mock并存根

Mockito中的Mock创建从mock()方法开始，该方法使用CGLib或反射API生成目标类的代理，然后在内部注册此代理，使Mockito能够管理其生命周期。

创建Mock后，我们继续进行方法存根，定义方法的特定行为。Mockito拦截此调用，识别目标方法和参数，并使用thenAnswer()方法设置自定义响应。thenAnswer()方法接收接口实现，允许我们指定自定义行为：

```java
// Mocking an OrderService class
OrderService orderService = mock(OrderService.class);

// Stubbing processOrder method
when(orderService.processOrder(anyString())).thenAnswer(invocation -> {
    String orderId = invocation.getArgument(0);
    return "Order " + orderId + " processed successfully";
});

// Using the stubbed method
String result = orderService.processOrder("12345");
System.out.println(result);  // Output: Order 12345 processed successfully
```

此处，processOrder()被存根以返回带有订单ID的消息。调用时，Mockito会拦截并应用自定义Answer逻辑。

### 3.2 方法调用拦截

了解Mockito的Answer API的工作原理对于在测试中设置灵活的行为至关重要，让我们分解一下在测试执行期间在Mock上调用方法时发生的内部过程：

- 当在Mock上调用某个方法时，**该调用会通过代理实例重定向到Mockito的内部处理机制**。
- Mockito检查该方法是否已注册行为，它使用方法签名来查找适当的Answer实现。
- 如果找到Answer实现，则传递给方法的参数、方法签名以及对Mock对象的引用的信息将存储在InvocationOnMock类的实例中。
- 使用InvocationOnMock，我们可以通过getArgument(int index)[访问方法参数](https://www.baeldung.com/mockito-argumentcaptor)，从而动态控制方法的行为。

此内部过程使Answer API能够根据上下文动态响应。考虑一个内容管理系统，其中用户权限因角色而异。我们可以使用Answer API动态Mock授权，具体取决于用户的角色和请求的操作。让我们看看我们将如何在以下部分中实现这一点。

## 4. 创建User和Action模型

由于我们将使用内容管理系统作为示例，因此我们将有四个角色：管理员、编辑者、查看者和访客，这些角色将作为不同CRUD操作的基本授权。管理员可以执行所有操作，编辑者可以创建、读取和更新，查看者只能读取内容，访客无权执行任何操作。让我们首先创建一个用户类：

```java
public class CmsUser {
    private String username;
    private String role;

    public CmsUser(String username, String role) {
        this.username = username;
        this.role = role;
    }

    public String getRole() {
        return this.role;
    }
}
```

现在，让我们定义一个枚举来使用ActionEnum类捕获可能的CRUD操作：

```java
public enum ActionEnum {
    CREATE, READ, UPDATE, DELETE;
}
```

定义好ActionEnum后，我们就可以开始Service层了。首先定义AuthorizationService接口，此接口将包含一个方法来检查用户是否可以执行特定的CRUD操作：

```java
public interface AuthorizationService {
    boolean authorize(CmsUser user, ActionEnum actionEnum);
}
```

此方法将返回给定的CmsUser是否有资格执行给定的CRUD操作。现在我们完成了此操作，我们可以继续查看Answer API的实际实现。

## 5. 创建AuthorizationService测试

我们首先创建AuthorizationService接口的Mock版本：

```java
@Mock
private AuthorizationService authorizationService;
```

现在，让我们**创建一个setup方法来初始化Mock并为AuthorizationService中的authorize()方法定义一个默认行为**，允许它根据角色Mock不同的用户权限：

```java
@Before
public void setup() {
    MockitoAnnotations.initMocks(this);
    when(this.authorizationService.authorize(any(CmsUser.class), any(ActionEnum.class)))
            .thenAnswer(invocation -> {
                CmsUser user = invocation.getArgument(0);
                ActionEnum action = invocation.getArgument(1);
                return switch (user.getRole()) {
                    case "ADMIN" -> true;
                    case "EDITOR" -> action != ActionEnum.DELETE;
                    case "VIEWER" -> action == ActionEnum.READ;
                    default -> false;
                };
            });
}
```

在此设置方法中，我们初始化测试类的Mock，以便在每次测试运行之前准备好所有Mock依赖项以供使用。接下来，我们使用when(this.authorizationService.authorize(...)).thenAnswer(...)定义authorizationServiceMock中authorize()方法的行为。此设置指定了一个自定义答案：每当使用任何CmsUser和任何ActionEnum调用authorize()方法时，它都会根据用户的角色做出响应。

为了验证正确性，我们可以从代码库中运行givenRoles_whenInvokingAuthorizationService_thenReturnExpectedResults()。

## 6. 验证我们的实现

现在我们已经完成所有事情，让我们创建测试方法来验证我们的实现。

```java
@Test
public void givenRoles_whenInvokingAuthorizationService_thenReturnExpectedResults() {
    CmsUser adminUser = createCmsUser("Admin User", "ADMIN");
    CmsUser guestUser = createCmsUser("Guest User", "GUEST");
    CmsUser editorUser = createCmsUser("Editor User", "EDITOR");
    CmsUser viewerUser = createCmsUser("Viewer User", "VIEWER");

    verifyAdminUserAccess(adminUser);
    verifyEditorUserAccess(editorUser);
    verifyViewerUserAccess(viewerUser);
    verifyGuestUserAccess(guestUser);
}
```

为了保持文章的简洁性，我们只关注其中一种验证方法。我们将介绍验证管理员用户访问权限的实现，可以参考代码仓库来了解其他用户角色的实现。

我们首先为不同的角色创建CmsUser实例。然后，我们调用verifyAdminUserAccess()方法，将adminUser实例作为参数传递。在verifyAdminUserAccess()方法中，我们遍历所有ActionEnum值并断言管理员用户可以访问所有值，这验证了授权服务是否正确地授予管理员用户对所有操作的完全访问权限。实现其他用户角色验证方法遵循类似的模式，如果我们需要进一步了解，可以在代码仓库中探索这些方法。

## 7. 总结

在本文中，**我们研究了如何使用Mockito的Answer API在Mock测试中动态实现基于角色的授权逻辑**。通过为用户设置基于角色的访问权限，我们展示了如何根据特定参数的属性返回不同的响应。这种方法可以增强代码覆盖率并最大限度地减少不可预见的故障几率，从而使我们的测试更加可靠和有效。