---
layout: post
title:  Quarkus中基于角色的访问控制
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

在本教程中，我们将讨论基于角色的访问控制(RBAC)以及如何使用[Quarkus](https://quarkus.io/)实现此功能。

RBAC是一种用于实现复杂安全系统的著名机制，Quarkus是一个现代云原生全栈Java框架，开箱即用地支持RBAC。

在开始之前，需要注意的是，角色可以以多种方式应用。在企业中，角色通常只是权限的集合，用于标识用户可以执行的一组特定操作。在[Jakarta](https://www.baeldung.com/java-enterprise-evolution)中，角色是允许执行资源操作(相当于权限)的标签。有多种实现RBAC系统的方法。

在本教程中，我们将使用分配给资源的权限来控制访问，并且角色将对权限列表进行分组。

## 2. RBAC

基于角色的访问控制是一种安全模型，它根据预定义的权限授予应用程序用户访问权限。系统管理员可以在访问尝试时将这些权限分配给特定资源并验证这些权限，为了帮助管理权限，他们创建角色来对权限进行分组：

![](/assets/images/2025/quarkus/javarbacquarkus01.png)

为了演示如何使用Quarkus实现RBAC系统，我们需要一些其他工具，例如JSON Web Tokens(JWT)、JPA和Quarkus Security模块。JWT可帮助我们实现一种简单且独立的方式来验证身份和授权，因此为了简单起见，我们将利用它作为示例。同样，JPA将帮助我们处理域逻辑和数据库之间的通信，而Quarkus将成为所有这些组件的粘合剂。

## 3. JWT

[JSON Web Tokens(JWT)](https://www.baeldung.com/java-json-web-tokens-jjwt)是一种在用户和服务器之间以紧凑、URL安全的JSON对象形式传输信息的安全方式。此令牌经过数字签名以供验证，通常用于基于Web的应用程序中的身份认证和安全数据交换。在身份验证期间，服务器会发出包含用户身份和声明的JWT，客户端将在后续请求中使用该JWT来访问受保护的资源：

![](/assets/images/2025/quarkus/javarbacquarkus02.png)

客户端通过提供一些凭据来请求令牌，然后授权服务器提供签名的令牌；稍后，当尝试访问资源时，客户端提供JWT令牌，资源服务器会验证该令牌并根据所需的权限进行验证。考虑到这些基础概念，让我们探索如何在Quarkus应用程序中集成RBAC和JWT。

## 4. 数据设计

为了简单起见，我们将创建一个基本的RBAC系统供本示例使用。为此，我们使用下表：

![](/assets/images/2025/quarkus/javarbacquarkus03.png)

这使我们能够表示用户、他们的角色以及组成每个角色的权限。[JPA](https://www.baeldung.com/learn-jpa-hibernate)数据库表将表示我们的域对象：

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(unique = true)
    private String username;

    @Column
    private String password;

    @Column(unique = true)
    private String email;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(name = "user_roles",
            joinColumns = @JoinColumn(name = "user_id"),
            inverseJoinColumns = @JoinColumn(name = "role_name"))
    private Set<Role> roles = new HashSet<>();

    // Getter and Setters
}
```

用户表保存登录凭证以及用户和角色之间的关系：

```java
@Entity
@Table(name = "roles")
public class Role {
    @Id
    private String name;

    @Roles
    @Convert(converter = PermissionConverter.class)
    private Set<Permission> permissions = new HashSet<>();

    // Getters and Setters
}
```

同样，为了简单起见，权限使用逗号分隔的值存储在一列中，为此，我们使用PermissionConverter。

## 5. JSON Web Token和Quarkus

在凭证方面，要使用JWT令牌并启用登录，我们需要以下依赖：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-jwt-build</artifactId>
    <version>3.9.4</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-jwt</artifactId>
    <version>3.9.4</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-test-security</artifactId>
    <scope>test</scope>
    <version>3.9.4</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-test-security-jwt</artifactId>
    <scope>test</scope>
    <version>3.9.4</version>
</dependency>
```

这些模块为我们提供了实现令牌生成、权限验证和测试实现的工具。现在，为了定义依赖和Quarkus版本，我们将使用[BOM Parent](https://mvnrepository.com/artifact/io.quarkus/quarkus-bom)，其中包含与框架兼容的特定版本。对于此示例，我们需要：

- [quarkus-smallrye-jwt-build](https://mvnrepository.com/artifact/io.quarkus/quarkus-smallrye-jwt-build)
- [quarkus-smallrye-jwt](https://mvnrepository.com/artifact/io.quarkus/quarkus-smallrye-jwt)
- [quarkus-test-security](https://mvnrepository.com/artifact/io.quarkus/quarkus-test-security)
- [quarkus-test-security-jwt](https://mvnrepository.com/artifact/io.quarkus/quarkus-test-security-jwt)

接下来，为了实现Token签名，我们需要RSA[公钥和私钥](https://www.baeldung.com/linux/ssh-setup-public-key-auth)。Quarkus有一种简单的配置方法，生成后，我们必须配置以下属性：

```properties
mp.jwt.verify.publickey.location=publicKey.pem
mp.jwt.verify.issuer=my-issuer
smallrye.jwt.sign.key.location=privateKey.pem
```

Quarkus默认查看/resources或提供的绝对路径，框架使用密钥签署声明并验证令牌。

## 6. 凭证

现在，要创建JWT令牌并设置其权限，我们需要验证用户的凭据，以下代码是我们如何执行此操作的示例：

```java
@Path("/secured")
public class SecureResourceController {
    // other methods...

    @POST
    @Path("/login")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @PermitAll
    public Response login(@Valid final LoginDto loginDto) {
        if (userService.checkUserCredentials(loginDto.username(), loginDto.password())) {
            User user = userService.findByUsername(loginDto.username());
            String token = userService.generateJwtToken(user);
            return Response.ok().entity(new TokenResponse("Bearer " + token,"3600")).build();
        } else {
            return Response.status(Response.Status.UNAUTHORIZED).entity(new Message("Invalid credentials")).build();
        }
    }
}
```

login端点验证用户凭据，并在成功的情况下发出令牌作为响应。另一个需要注意的重要事项是@PermitAll，它确保此端点是公开的并且不需要任何身份验证。但是，我们很快会更详细地讨论权限。

这里我们要特别注意的另一段重要代码是generateJwtToken方法，该方法创建并签署一个令牌。

```java
public String generateJwtToken(final User user) {
    Set<String> permissions = user.getRoles()
            .stream()
            .flatMap(role -> role.getPermissions().stream())
            .map(Permission::name)
            .collect(Collectors.toSet());

    return Jwt.issuer(issuer)
            .upn(user.getUsername())
            .groups(permissions)
            .expiresIn(3600)
            .claim(Claims.email_verified.name(), user.getEmail())
            .sign();
}
```

在这个方法中，我们检索每个角色提供的权限列表并将其注入到令牌中。颁发者还定义令牌、重要声明和存活时间，然后，最后，我们对令牌进行签名。一旦用户收到它，它将用于验证所有后续调用。令牌包含服务器对相应用户进行身份验证和授权所需的所有内容，用户只需将承载令牌发送到Authentication标头即可验证调用。

## 7. 权限

如前所述，Jakarta使用@RolesAllowed为资源分配权限。尽管它称它们为角色，但它们的工作方式类似于权限(根据我们之前定义的概念)，这意味着我们只需要用它来标注我们的端点即可保护它们，例如：

```java
@Path("/secured")
public class SecureResourceController {
    private final UserService userService;
    private final SecurityIdentity securityIdentity;

    // constructor

    @GET
    @Path("/resource")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @RolesAllowed({"VIEW_ADMIN_DETAILS"})
    public String get() {
        return "Hello world, here are some details about the admin!";
    }

    @GET
    @Path("/resource/user")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @RolesAllowed({"VIEW_USER_DETAILS"})
    public Message getUser() {
        return new Message("Hello "+securityIdentity.getPrincipal().getName()+"!");
    }

    //...
}
```

查看代码，我们可以看到向我们的端点添加权限控制是多么简单。在我们的例子中，`/secured/resource/user`现在需要VIEW_USER_DETAILS权限，并且`/secured/resource`需要VIEW_ADMIN_DETAILS。我们还可以观察到，可以分配一个权限列表，而不是只有一个。在这种情况下，Quarkus将需要@RolesAllowed中列出的至少一个权限。

另一个重要说明是，令牌包含有关当前登录用户(安全身份中的主体)的权限和信息。

## 8. 测试

Quarku 提供了许多工具，使我们的应用程序测试变得简单且易于实现。使用这些工具，我们可以配置JWT的创建和设置及其上下文，从而使测试意图清晰易懂。以下测试显示了这一点：

```java
@QuarkusTest
class SecureResourceControllerTest {
    @Test
    @TestSecurity(user = "user", roles = "VIEW_USER_DETAILS")
    @JwtSecurity(claims = {
            @Claim(key = "email", value = "user@test.io")
    })
    void givenSecureAdminApi_whenUserTriesToAccessAdminApi_thenShouldNotAllowRequest() {
        given()
                .contentType(ContentType.JSON)
                .get("/secured/resource")
                .then()
                .statusCode(403);
    }

    @Test
    @TestSecurity(user = "admin", roles = "VIEW_ADMIN_DETAILS")
    @JwtSecurity(claims = {
            @Claim(key = "email", value = "admin@test.io")
    })
    void givenSecureAdminApi_whenAdminTriesAccessAdminApi_thenShouldAllowRequest() {
        given()
                .contentType(ContentType.JSON)
                .get("/secured/resource")
                .then()
                .statusCode(200)
                .body(equalTo("Hello world, here are some details about the admin!"));
    }

    //...
}
```

@TestSecurity注解允许定义安全属性，而@JwtSecurity允许定义Token的声明。使用这两种工具，我们可以测试多种场景和用例。

到目前为止，我们看到的工具已经足以使用Quarkus实现强大的RBAC系统。但是，它还有更多选择。

## 9. Quarkus Security

Quarkus还提供了一个强大的安全系统，可以与我们的RBAC解决方案集成。让我们看看如何将此类功能与我们的RBAC实现相结合。首先，我们需要了解这些概念，因为Quarkus权限系统不适用于角色。但是，可以创建角色权限之间的映射：

```properties
quarkus.http.auth.policy.role-policy1.permissions.VIEW_ADMIN_DETAILS=VIEW_ADMIN_DETAILS
quarkus.http.auth.policy.role-policy1.permissions.VIEW_USER_DETAILS=VIEW_USER_DETAILS
quarkus.http.auth.policy.role-policy1.permissions.SEND_MESSAGE=SEND_MESSAGE
quarkus.http.auth.policy.role-policy1.permissions.CREATE_USER=CREATE_USER
quarkus.http.auth.policy.role-policy1.permissions.OPERATOR=OPERATOR
quarkus.http.auth.permission.roles1.paths=/permission-based/*
quarkus.http.auth.permission.roles1.policy=role-policy1
```

使用application.properties文件，我们定义一个角色策略，将角色映射到权限。映射的工作方式类似于quarkus.http.auth.policy.{policyName}.permissions.{roleName}={listOfPermissions}。在有关角色和权限的示例中，它们具有相同的名称并一一对应。但是，这可能不是强制性的，也可以将角色映射到权限列表。然后，映射完成后，我们使用配置的最后两行定义应用此策略的路径。

资源权限设置也会有点不同，例如：

```java
@Path("/permission-based")
public class PermissionBasedController {
    private final SecurityIdentity securityIdentity;

    public PermissionBasedController(SecurityIdentity securityIdentity) {
        this.securityIdentity = securityIdentity;
    }

    @GET
    @Path("/resource/version")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @PermissionsAllowed("VIEW_ADMIN_DETAILS")
    public String get() {
        return "2.0.0";
    }

    @GET
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @Path("/resource/message")
    @PermissionsAllowed(value = {"SEND_MESSAGE", "OPERATOR"}, inclusive = true)
    public Message message() {
        return new Message("Hello "+securityIdentity.getPrincipal().getName()+"!");
    }
}
```

设置类似，在我们的例子中，唯一的变化是@PermissionsAllowed注解而不是@RolesAllowed。**此外，权限还允许不同的行为，例如包含标志，权限匹配机制的行为从OR更改为AND**。我们使用与之前相同的设置来测试行为：

```java
@QuarkusTest
class PermissionBasedControllerTest {
    @Test
    @TestSecurity(user = "admin", roles = "VIEW_ADMIN_DETAILS")
    @JwtSecurity(claims = {
            @Claim(key = "email", value = "admin@test.io")
    })
    void givenSecureVersionApi_whenUserIsAuthenticated_thenShouldReturnVersion() {
        given()
                .contentType(ContentType.JSON)
                .get("/permission-based/resource/version")
                .then()
                .statusCode(200)
                .body(equalTo("2.0.0"));
    }

    @Test
    @TestSecurity(user = "user", roles = "SEND_MESSAGE")
    @JwtSecurity(claims = {
            @Claim(key = "email", value = "user@test.io")
    })
    void givenSecureMessageApi_whenUserOnlyHasOnePermission_thenShouldNotAllowRequest() {
        given()
                .contentType(ContentType.JSON)
                .get("/permission-based/resource/message")
                .then()
                .statusCode(403);
    }

    @Test
    @TestSecurity(user = "new-operator", roles = {"SEND_MESSAGE", "OPERATOR"})
    @JwtSecurity(claims = {
            @Claim(key = "email", value = "operator@test.io")
    })
    void givenSecureMessageApi_whenUserOnlyHasBothPermissions_thenShouldAllowRequest() {
        given()
                .contentType(ContentType.JSON)
                .get("/permission-based/resource/message")
                .then()
                .statusCode(200)
                .body("message", equalTo("Hello new-operator!"));
    }
}
```

Quarkus Security模块提供许多其他功能，但本文不会介绍它们。

## 10. 总结

在本文中，我们讨论了RBAC系统以及如何利用Quarkus框架来实现它。我们还看到了关于如何使用角色或权限的一些细微差别以及它们在此实现中的概念差异。最后，我们观察了Jakarta实现和Quarkus Security模块之间的差异，以及它们如何在这两种情况下帮助测试此类功能。