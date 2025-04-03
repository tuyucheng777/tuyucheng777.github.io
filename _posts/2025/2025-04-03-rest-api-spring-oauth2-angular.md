## 1. 概述

在本教程中，**我们将使用OAuth2保护REST API并从简单的Angular客户端使用它**。

我们要构建的应用程序将由3个独立的模块组成：

- 授权服务器
- 资源服务器
- UI授权码：使用授权码流程的前端应用程序

**我们将使用Spring Security 5中的OAuth堆栈**，如果你想使用Spring Security OAuth旧堆栈，请查看之前的文章：[Spring REST API + OAuth2 + Angular(使用Spring Security OAuth旧堆栈)](https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)。

## 2. OAuth2授权服务器(AS)

简单来说，**授权服务器是一个颁发授权令牌的应用程序**。

以前，Spring Security OAuth堆栈提供了将授权服务器设置为Spring应用程序的可能性，但该项目已被弃用，主要是因为OAuth是一个开放标准，拥有许多知名提供商，例如Okta、Keycloak和ForgeRock等。

其中，我们将使用[Keycloak](https://www.baeldung.com/spring-boot-keycloak)。它是由RedHat管理的开源身份和访问管理服务器，由JBoss用Java开发。它不仅支持OAuth2，还支持其他标准协议，例如OpenID Connect和SAML。

在本教程中，**我们将[在Spring Boot应用程序中设置嵌入式Keycloak服务器](https://www.baeldung.com/keycloak-embedded-in-a-spring-boot-application)**。

## 3. 资源服务器(RS)

现在让我们讨论资源服务器；**这本质上是REST API，我们最终希望能够使用它**。

### 3.1 Maven配置

我们的资源服务器的pom与以前的授权服务器pom非常相似，除了Keycloak部分之外，**还带有额外的[spring-boot-starter-oauth2-resource-server](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-oauth2-resource-server)依赖**：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

### 3.2 安全配置

由于我们使用的是Spring Boot，**因此我们可以使用Boot属性定义所需的最小配置**。

我们将在application.yml文件中执行此操作：

```yaml
server:
    port: 8081
    servlet:
        context-path: /resource-server

spring:
    security:
        oauth2:
            resourceserver:
                jwt:
                    issuer-uri: http://localhost:8083/auth/realms/tuyucheng
                    jwk-set-uri: http://localhost:8083/auth/realms/tuyucheng/protocol/openid-connect/certs
```

在这里，我们指定将使用JWT令牌进行授权。

**jwk-set-uri属性指向包含公钥的URI，以便我们的资源服务器可以验证令牌的完整性**。

issuer-uri属性代表一种额外的安全措施，用于验证令牌的颁发者(即授权服务器)。但是，添加此属性还要求授权服务器必须在我们启动资源服务器应用程序之前运行。

接下来，让我们为API设置安全配置来保护端点：

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.cors()
                .and()
                .authorizeRequests()
                .antMatchers(HttpMethod.GET, "/user/info", "/api/foos/**")
                .hasAuthority("SCOPE_read")
                .antMatchers(HttpMethod.POST, "/api/foos")
                .hasAuthority("SCOPE_write")
                .anyRequest()
                .authenticated()
                .and()
                .oauth2ResourceServer()
                .jwt();
        return http.build();
    }
}
```

我们可以看到，对于GET方法，我们仅允许具有读取范围的请求。对于POST方法，请求者除了读取之外还需要具有写入权限。但是，对于任何其他端点，请求应该只针对任何用户进行身份验证。

此外，oauth2ResourceServer()方法指定这是一个资源服务器，具有jwt()格式的令牌。

这里要注意的另一点是使用cors()方法允许在请求中使用Access-Control标头，这一点尤其重要，因为我们正在处理Angular客户端，并且我们的请求将来自另一个原始URL。

### 3.3 模型和Repository

接下来，让我们为模型Foo定义一个javax.persistence.Entity：

```java
@Entity
public class Foo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // constructor, getters and setters
}
```

然后我们需要一个Foo的Repository，我们将使用Spring的PagingAndSortingRepository：

```java
public interface IFooRepository extends PagingAndSortingRepository<Foo, Long> {
}
```

### 3.4 服务与实现

之后，我们将为我们的API定义并实现一个简单的服务：

```java
public interface IFooService {
    Optional<Foo> findById(Long id);

    Foo save(Foo foo);

    Iterable<Foo> findAll();
}

@Service
public class FooServiceImpl implements IFooService {

    private IFooRepository fooRepository;

    public FooServiceImpl(IFooRepository fooRepository) {
        this.fooRepository = fooRepository;
    }

    @Override
    public Optional<Foo> findById(Long id) {
        return fooRepository.findById(id);
    }

    @Override
    public Foo save(Foo foo) {
        return fooRepository.save(foo);
    }

    @Override
    public Iterable<Foo> findAll() {
        return fooRepository.findAll();
    }
}
```

### 3.5 示例控制器

现在让我们实现一个通过DTO公开我们的Foo资源的简单控制器：

```java
@RestController
@RequestMapping(value = "/api/foos")
public class FooController {

    private IFooService fooService;

    public FooController(IFooService fooService) {
        this.fooService = fooService;
    }

    @CrossOrigin(origins = "http://localhost:8089")
    @GetMapping(value = "/{id}")
    public FooDto findOne(@PathVariable Long id) {
        Foo entity = fooService.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
        return convertToDto(entity);
    }

    @GetMapping
    public Collection<FooDto> findAll() {
        Iterable<Foo> foos = this.fooService.findAll();
        List<FooDto> fooDtos = new ArrayList<>();
        foos.forEach(p -> fooDtos.add(convertToDto(p)));
        return fooDtos;
    }

    protected FooDto convertToDto(Foo entity) {
        FooDto dto = new FooDto(entity.getId(), entity.getName());

        return dto;
    }
}
```

**注意上面的@CrossOrigin的使用；这是我们需要允许在指定URL上运行的Angular App的CORS的控制器级配置**。

这是FooDto：

```java
public class FooDto {
    private long id;
    private String name;
}
```

## 4. 前端—设置

我们现在将研究客户端的简单前端Angular实现，它将访问我们的REST API。

我们首先使用[Angular CLI](https://cli.angular.io/)来生成和管理我们的前端模块。

**首先，我们安装[node和npm](https://nodejs.org/en/download/)**，因为Angular CLI是一个npm工具。

然后我们需要使用[frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin)使用Maven构建我们的Angular项目：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.github.eirslett</groupId>
            <artifactId>frontend-maven-plugin</artifactId>
            <version>1.3</version>
            <configuration>
                <nodeVersion>v6.10.2</nodeVersion>
                <npmVersion>3.10.10</npmVersion>
                <workingDirectory>src/main/resources</workingDirectory>
            </configuration>
            <executions>
                <execution>
                    <id>install node and npm</id>
                    <goals>
                        <goal>install-node-and-npm</goal>
                    </goals>
                </execution>
                <execution>
                    <id>npm install</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                </execution>
                <execution>
                    <id>npm run build</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <configuration>
                        <arguments>run build</arguments>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

最后，**使用Angular CLI生成一个新模块**：

```shell
ng new oauthApp
```

在下一节中，我们将讨论Angular应用程序逻辑。

## 5. 使用Angular的授权码流程

我们将在这里使用OAuth2授权码流程。

我们的用例：客户端应用向授权服务器请求代码，并显示登录页面。**一旦用户提供其有效凭证并提交，授权服务器就会向我们提供代码**，然后前端客户端使用它来获取访问令牌。

### 5.1 主页组件

让我们从主要组件HomeComponent开始，所有操作都从这里开始：

```typescript
@Component({
    selector: 'home-header',
    providers: [AppService],
    template: `<div class="container" >
    <button *ngIf="!isLoggedIn" class="btn btn-primary" (click)="login()" type="submit">
        Login</button>
    <div *ngIf="isLoggedIn" class="content">
        <span>Welcome !!</span>
        <a class="btn btn-default pull-right"(click)="logout()" href="#">Logout</a>
        <br/>
        <foo-details></foo-details>
    </div>
  </div>`
})

export class HomeComponent {
    public isLoggedIn = false;

    constructor(private _service: AppService) { }

    ngOnInit() {
        this.isLoggedIn = this._service.checkCredentials();
        let i = window.location.href.indexOf('code');
        if(!this.isLoggedIn && i != -1) {
            this._service.retrieveToken(window.location.href.substring(i + 5));
        }
    }

    login() {
        window.location.href = 'http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/auth?
        response_type=code&scope=openid%20write%20read&client_id=' + 
        this._service.clientId + '&redirect_uri='+ this._service.redirectUri;
    }

    logout() {
        this._service.logout();
    }
}
```

一开始，当用户没有登录时，只显示登录按钮。单击此按钮后，用户将导航到AS的授权URL，并在其中输入用户名和密码。成功登录后，用户将使用授权码重定向回来，然后我们使用此代码检索访问令牌。

### 5.2 应用服务

现在让我们看看位于app.service.ts的AppService，它包含服务器交互的逻辑：

- retrieveToken()：使用授权码获取访问令牌
- saveToken()：使用ng2-cookies库将访问令牌保存在cookie中
- getResource()：使用其ID从服务器获取Foo对象
- checkCredentials()：检查用户是否已登录
- logout()：删除访问令牌cookie并注销用户

```typescript
export class Foo {
    constructor(public id: number, public name: string) { }
}

@Injectable()
export class AppService {
    public clientId = 'newClient';
    public redirectUri = 'http://localhost:8089/';

    constructor(private _http: HttpClient) { }

    retrieveToken(code) {
        let params = new URLSearchParams();
        params.append('grant_type','authorization_code');
        params.append('client_id', this.clientId);
        params.append('redirect_uri', this.redirectUri);
        params.append('code',code);

        let headers =
            new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});

        this._http.post('http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token',
            params.toString(), { headers: headers })
            .subscribe(
                data => this.saveToken(data),
                err => alert('Invalid Credentials'));
    }

    saveToken(token) {
        var expireDate = new Date().getTime() + (1000 * token.expires_in);
        Cookie.set("access_token", token.access_token, expireDate);
        console.log('Obtained Access token');
        window.location.href = 'http://localhost:8089';
    }

    getResource(resourceUrl) : Observable<any> {
        var headers = new HttpHeaders({
            'Content-type': 'application/x-www-form-urlencoded; charset=utf-8',
            'Authorization': 'Bearer '+Cookie.get('access_token')});
        return this._http.get(resourceUrl, { headers: headers })
            .catch((error:any) => Observable.throw(error.json().error || 'Server error'));
    }

    checkCredentials() {
        return Cookie.check('access_token');
    }

    logout() {
        Cookie.delete('access_token');
        window.location.reload();
    }
}
```

在retrieveToken方法中，我们使用客户端凭据和Basic Auth向/openid-connect/token端点发送POST以获取访问令牌，参数以URL编码格式发送。获取访问令牌后，我们将其存储在Cookie中。

Cookie存储在这里尤其重要，因为我们仅将Cookie用于存储目的，而不是直接用于驱动身份验证过程，**这有助于防止跨站点请求伪造(CSRF)攻击和漏洞**。

### 5.3 Foo组件

最后，FooComponent显示我们的Foo详细信息：

```typescript
@Component({
    selector: 'foo-details',
    providers: [AppService],
    template: `<div class="container">
    <h1 class="col-sm-12">Foo Details</h1>
    <div class="col-sm-12">
        <label class="col-sm-3">ID</label> <span>{{foo.id}}</span>
    </div>
    <div class="col-sm-12">
        <label class="col-sm-3">Name</label> <span>{{foo.name}}</span>
    </div>
    <div class="col-sm-12">
        <button class="btn btn-primary" (click)="getFoo()" type="submit">New Foo</button>        
    </div>
  </div>`
})

export class FooComponent {
    public foo = new Foo(1,'sample foo');
    private foosUrl = 'http://localhost:8081/resource-server/api/foos/';

    constructor(private _service:AppService) {}

    getFoo() {
        this._service.getResource(this.foosUrl+this.foo.id)
            .subscribe(
                data => this.foo = data,
                error =>  this.foo.name = 'Error');
    }
}
```

### 5.4 应用组件

我们的简单AppComponent将充当根组件：

```typescript
@Component({
    selector: 'app-root',
    template: `<nav class="navbar navbar-default">
    <div class="container-fluid">
        <div class="navbar-header">
            <a class="navbar-brand" href="/">Spring Security Oauth - Authorization Code</a>
        </div>
    </div>
  </nav>
  <router-outlet></router-outlet>`
})

export class AppComponent { }
```

我们把所有的组件，服务和路由包装在AppModule中：

```typescript
@NgModule({
    declarations: [
        AppComponent,
        HomeComponent,
        FooComponent
    ],
    imports: [
        BrowserModule,
        HttpClientModule,
        RouterModule.forRoot([
            { path: '', component: HomeComponent, pathMatch: 'full' }], {onSameUrlNavigation: 'reload'})
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

## 6. 运行前端

1. 要运行任何前端模块，我们需要先构建应用程序：

```shell
mvn clean install
```

2. 然后我们需要导航到Angular应用程序目录：

```shell
cd src/main/resources
```

3. 最后，启动我们的应用程序：

```shell
npm start
```

服务器将默认在端口4200上启动；要更改任何模块的端口，请更改：

```text
"start": "ng serve"
```

在package.json中；例如，为了使其在端口8089上运行，添加：

```text
"start": "ng serve --port 8089"
```

## 7. 总结

在本文中，我们学习了如何使用OAuth2授权我们的应用程序。