---
layout: post
title:  Spring REST API + OAuth2 + Angular(使用Spring Security OAuth遗留堆栈)
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将使用OAuth保护REST API并从简单的Angular客户端使用它。

我们要构建的应用程序将由4个独立的模块组成：

- 授权服务器
- 资源服务器
- UI隐式：使用隐式流的前端应用程序
- UI密码：使用密码流的前端应用程序

注意：本文使用的是[Spring OAuth旧项目](https://spring.io/projects/spring-authorization-server)，有关使用新Spring Security 5堆栈的版本，请参阅我们的文章[Spring REST API + OAuth2 + Angular](https://www.baeldung.com/rest-api-spring-oauth2-angular)。

## 2. 授权服务器

首先，让我们开始将授权服务器设置为一个简单的Spring Boot应用程序。

### 2.1 Maven配置

我们将设置以下一组依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>    
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
</dependency>  
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
</dependency>
```

请注意，我们使用spring-jdbc和MySQL，因为我们将使用JDBC支持的令牌存储实现。

### 2.2 @EnableAuthorizationServer

现在，让我们开始配置负责管理访问令牌的授权服务器：

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerOAuth2Config extends AuthorizationServerConfigurerAdapter {

    @Autowired
    @Qualifier("authenticationManagerBean")
    private AuthenticationManager authenticationManager;

    @Override
    public void configure(
            AuthorizationServerSecurityConfigurer oauthServer)
            throws Exception {
        oauthServer
                .tokenKeyAccess("permitAll()")
                .checkTokenAccess("isAuthenticated()");
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients)
            throws Exception {
        clients.jdbc(dataSource())
                .withClient("sampleClientId")
                .authorizedGrantTypes("implicit")
                .scopes("read")
                .autoApprove(true)
                .and()
                .withClient("clientIdPassword")
                .secret("secret")
                .authorizedGrantTypes(
                        "password","authorization_code", "refresh_token")
                .scopes("read");
    }

    @Override
    public void configure(
            AuthorizationServerEndpointsConfigurer endpoints)
            throws Exception {

        endpoints
                .tokenStore(tokenStore())
                .authenticationManager(authenticationManager);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JdbcTokenStore(dataSource());
    }
}
```

注意：

- 为了持久保存令牌，我们使用了JdbcTokenStore
- 我们注册了一个“implicit”授权类型的客户端
- 我们注册了另一个客户端并授予了“password”，“authorization_code”和“refresh_token”授权类型
- 为了使用“password”授权类型，我们需要接入并使用AuthenticationManager Bean

### 2.3 数据源配置

接下来，让我们配置数据源以供JdbcTokenStore使用：

```java
@Value("classpath:schema.sql")
private Resource schemaScript;

@Bean
public DataSourceInitializer dataSourceInitializer(DataSource dataSource) {
    DataSourceInitializer initializer = new DataSourceInitializer();
    initializer.setDataSource(dataSource);
    initializer.setDatabasePopulator(databasePopulator());
    return initializer;
}

private DatabasePopulator databasePopulator() {
    ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
    populator.addScript(schemaScript);
    return populator;
}

@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
    dataSource.setUrl(env.getProperty("jdbc.url"));
    dataSource.setUsername(env.getProperty("jdbc.user"));
    dataSource.setPassword(env.getProperty("jdbc.pass"));
    return dataSource;
}
```

请注意，由于我们使用JdbcTokenStore，因此我们需要初始化数据库模式，我们使用DataSourceInitializer-以及以下SQL模式：

```sql
drop table if exists oauth_client_details;
create table oauth_client_details (
                                      client_id VARCHAR(255) PRIMARY KEY,
                                      resource_ids VARCHAR(255),
                                      client_secret VARCHAR(255),
                                      scope VARCHAR(255),
                                      authorized_grant_types VARCHAR(255),
                                      web_server_redirect_uri VARCHAR(255),
                                      authorities VARCHAR(255),
                                      access_token_validity INTEGER,
                                      refresh_token_validity INTEGER,
                                      additional_information VARCHAR(4096),
                                      autoapprove VARCHAR(255)
);

drop table if exists oauth_client_token;
create table oauth_client_token (
                                    token_id VARCHAR(255),
                                    token LONG VARBINARY,
                                    authentication_id VARCHAR(255) PRIMARY KEY,
                                    user_name VARCHAR(255),
                                    client_id VARCHAR(255)
);

drop table if exists oauth_access_token;
create table oauth_access_token (
                                    token_id VARCHAR(255),
                                    token LONG VARBINARY,
                                    authentication_id VARCHAR(255) PRIMARY KEY,
                                    user_name VARCHAR(255),
                                    client_id VARCHAR(255),
                                    authentication LONG VARBINARY,
                                    refresh_token VARCHAR(255)
);

drop table if exists oauth_refresh_token;
create table oauth_refresh_token (
                                     token_id VARCHAR(255),
                                     token LONG VARBINARY,
                                     authentication LONG VARBINARY
);

drop table if exists oauth_code;
create table oauth_code (
                            code VARCHAR(255), authentication LONG VARBINARY
);

drop table if exists oauth_approvals;
create table oauth_approvals (
                                 userId VARCHAR(255),
                                 clientId VARCHAR(255),
                                 scope VARCHAR(255),
                                 status VARCHAR(10),
                                 expiresAt TIMESTAMP,
                                 lastModifiedAt TIMESTAMP
);

drop table if exists ClientDetails;
create table ClientDetails (
                               appId VARCHAR(255) PRIMARY KEY,
                               resourceIds VARCHAR(255),
                               appSecret VARCHAR(255),
                               scope VARCHAR(255),
                               grantTypes VARCHAR(255),
                               redirectUrl VARCHAR(255),
                               authorities VARCHAR(255),
                               access_token_validity INTEGER,
                               refresh_token_validity INTEGER,
                               additionalInformation VARCHAR(4096),
                               autoApproveScopes VARCHAR(255)
);
```

请注意，我们不一定需要显式的DatabasePopulator Bean-**我们可以简单地使用schema.sql，Spring Boot默认使用它**。

### 2.4 安全配置

最后，让我们保护授权服务器。

当客户端应用程序需要获取访问令牌时，它将在一个简单的表单登录驱动的身份验证过程之后进行获取：

```java
@Configuration
public class ServerSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("john").password("123").roles("USER");
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean()
            throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/login").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().permitAll();
    }
}
```

这里需要注意的是，**表单登录配置对于密码流来说不是必需的**-仅对于隐式流来说才是必需的，因此你可以根据你使用的OAuth2流跳过它。

## 3. 资源服务器

现在，让我们讨论资源服务器；这本质上是我们最终希望能够使用的REST API。

### 3.1 Maven配置

我们的资源服务器配置与之前的授权服务器应用程序配置相同。

### 3.2 令牌存储配置

接下来，我们将配置我们的TokenStore以访问授权服务器用于存储访问令牌的同一数据库：

```java
@Autowired
private Environment env;

@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
    dataSource.setUrl(env.getProperty("jdbc.url"));
    dataSource.setUsername(env.getProperty("jdbc.user"));
    dataSource.setPassword(env.getProperty("jdbc.pass"));
    return dataSource;
}

@Bean
public TokenStore tokenStore() {
    return new JdbcTokenStore(dataSource());
}
```

请注意，对于这个简单的实现，即使授权和资源服务器是独立的应用程序，我们也共享SQL支持的令牌存储。

原因当然是资源服务器需要能够检查授权服务器颁发的访问令牌的有效性。

### 3.3 远程令牌服务

我们可以使用RemoteTokeServices，而不是在资源服务器中使用TokenStore：

```java
@Primary
@Bean
public RemoteTokenServices tokenService() {
    RemoteTokenServices tokenService = new RemoteTokenServices();
    tokenService.setCheckTokenEndpointUrl("http://localhost:8080/spring-security-oauth-server/oauth/check_token");
    tokenService.setClientId("fooClientIdPassword");
    tokenService.setClientSecret("secret");
    return tokenService;
}
```

注意：

- 该RemoteTokenService将使用授权服务器上的CheckTokenEndPoint来验证AccessToken并从中获取Authentication对象。
- 可以在AuthorizationServerBaseURL + “/oauth/check_token”找到
- 授权服务器可以使用任何TokenStore类型[JdbcTokenStore、JwtTokenStore...\]–这不会影响RemoteTokenService或资源服务器。

### 3.4 示例控制器

接下来，让我们实现一个公开Foo资源的简单控制器：

```java
@Controller
public class FooController {

    @PreAuthorize("#oauth2.hasScope('read')")
    @RequestMapping(method = RequestMethod.GET, value = "/foos/{id}")
    @ResponseBody
    public Foo findById(@PathVariable long id) {
        return new Foo(Long.parseLong(randomNumeric(2)), randomAlphabetic(4));
    }
}
```

请注意客户端需要“read”范围来访问此资源。

我们还需要启用全局方法安全并配置MethodSecurityExpressionHandler：

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class OAuth2ResourceServerConfig extends GlobalMethodSecurityConfiguration {

    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        return new OAuth2MethodSecurityExpressionHandler();
    }
}
```

这是我们的基本Foo资源：

```java
public class Foo {
    private long id;
    private String name;
}
```

### 3.5 Web配置

最后，让我们为API设置一个非常基本的Web配置：

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "cn.tuyucheng.taketoday.web.controller" })
public class ResourceWebConfig implements WebMvcConfigurer {}
```

## 4. 前端-设置

我们现在来看一下客户端的简单前端Angular实现。

首先，我们将使用[Angular CLI](https://cli.angular.io/)来生成和管理我们的前端模块。

首先，我们将安装[node和npm](https://nodejs.org/en/download/)–因为Angular CLI是一个npm工具。

然后，我们需要使用[frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin)通过Maven构建我们的Angular项目：

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

最后，使用Angular CLI生成一个新模块：

```shell
ng new oauthApp
```

请注意，我们将有两个前端模块-一个用于密码流，另一个用于隐式流。

在以下部分中，我们将讨论每个模块的Angular应用程序逻辑。

## 5. 使用Angular的密码流

我们将在这里使用OAuth2密码流程-这就是为什么这只是一个概念证明，而不是一个可用于生产的应用程序。你会注意到客户端凭据已暴露给前端-这是我们将在以后的文章中讨论的问题。

我们的用例很简单：一旦用户提供他们的凭证，前端客户端就会使用它们从授权服务器获取访问令牌。

### 5.1 应用服务

让我们从位于app.service.ts的AppService开始，它包含服务器交互的逻辑：

- obtainedAccessToken()：根据用户凭证获取访问令牌
- saveToken()：使用ng2-cookies库将访问令牌保存在Cookie中
- getResource()：使用其ID从服务器获取Foo对象
- checkCredentials()：检查用户是否已登录
- logout()：删除访问令牌Cookie并注销用户

```typescript
export class Foo {
    constructor(
        public id: number,
        public name: string) { }
}

@Injectable()
export class AppService {
    constructor(
        private _router: Router, private _http: Http){}

    obtainAccessToken(loginData){
        let params = new URLSearchParams();
        params.append('username',loginData.username);
        params.append('password',loginData.password);
        params.append('grant_type','password');
        params.append('client_id','fooClientIdPassword');
        let headers =
            new Headers({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8',
                'Authorization': 'Basic '+btoa("fooClientIdPassword:secret")});
        let options = new RequestOptions({ headers: headers });

        this._http.post('http://localhost:8081/spring-security-oauth-server/oauth/token',
            params.toString(), options)
            .map(res => res.json())
            .subscribe(
                data => this.saveToken(data),
                err => alert('Invalid Credentials'));
    }

    saveToken(token){
        var expireDate = new Date().getTime() + (1000 * token.expires_in);
        Cookie.set("access_token", token.access_token, expireDate);
        this._router.navigate(['/']);
    }

    getResource(resourceUrl) : Observable<Foo>{
        var headers =
            new Headers({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8',
                'Authorization': 'Bearer '+Cookie.get('access_token')});
        var options = new RequestOptions({ headers: headers });
        return this._http.get(resourceUrl, options)
            .map((res:Response) => res.json())
            .catch((error:any) =>
                Observable.throw(error.json().error || 'Server error'));
    }

    checkCredentials(){
        if (!Cookie.check('access_token')){
            this._router.navigate(['/login']);
        }
    }

    logout() {
        Cookie.delete('access_token');
        this._router.navigate(['/login']);
    }
}
```

注意：

- 要获取访问令牌，我们向“/oauth/token”端点发送POST请求
- 我们使用客户端凭证和基本身份验证来访问此端点
- 然后我们将用户凭证连同客户端ID和授权类型参数一起发送到URL编码
- 获取访问令牌后，**我们将其存储在Cookie中**

Cookie存储在这里尤其重要，因为我们仅将Cookie用于存储目的，而不是直接用于驱动身份验证过程，**这有助于防止跨站点请求伪造(CSRF)类型的攻击和漏洞**。

### 5.2 登录组件

接下来，让我们看一下负责登录表单的LoginComponent：

```typescript
@Component({
    selector: 'login-form',
    providers: [AppService],
    template: `<h1>Login</h1>
    <input type="text" [(ngModel)]="loginData.username" />
    <input type="password"  [(ngModel)]="loginData.password"/>
    <button (click)="login()" type="submit">Login</button>`
})
export class LoginComponent {
    public loginData = {username: "", password: ""};

    constructor(private _service: AppService) {
    }

    login() {
        this._service.obtainAccessToken(this.loginData);
    }
}
```

### 5.3 主页组件

接下来，我们的HomeComponent负责显示和操作我们的主页：

```typescript
@Component({
    selector: 'home-header',
    providers: [AppService],
    template: `<span>Welcome !!</span>
    <a (click)="logout()" href="#">Logout</a>
    <foo-details></foo-details>`
})

export class HomeComponent {
    constructor(
        private _service:AppService){}

    ngOnInit(){
        this._service.checkCredentials();
    }

    logout() {
        this._service.logout();
    }
}
```

### 5.4 Foo组件

最后，我们的FooComponent显示我们的Foo详细信息：

```typescript
@Component({
    selector: 'foo-details',
    providers: [AppService],
    template: `<h1>Foo Details</h1>
    <label>ID</label> <span>{{foo.id}}</span>
    <label>Name</label> <span>{{foo.name}}</span>
    <button (click)="getFoo()" type="submit">New Foo</button>`
})

export class FooComponent {
    public foo = new Foo(1,'sample foo');
    private foosUrl = 'http://localhost:8082/spring-security-oauth-resource/foos/';

    constructor(private _service:AppService) {}

    getFoo(){
        this._service.getResource(this.foosUrl+this.foo.id)
            .subscribe(
                data => this.foo = data,
                error =>  this.foo.name = 'Error');
    }
}
```

### 5.5 应用组件

我们的简单AppComponent将充当根组件：

```typescript
@Component({
    selector: 'app-root',
    template: `<router-outlet></router-outlet>`
})

export class AppComponent {}
```

我们把所有的组件，服务和路由包装在AppModule中：

```typescript
@NgModule({
    declarations: [
        AppComponent,
        HomeComponent,
        LoginComponent,
        FooComponent
    ],
    imports: [
        BrowserModule,
        FormsModule,
        HttpModule,
        RouterModule.forRoot([
            { path: '', component: HomeComponent },
            { path: 'login', component: LoginComponent }])
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

## 6. 隐式流程

接下来，我们将重点关注隐式流模块。

### 6.1 应用服务

类似地，我们将从服务开始，但这次我们将使用库[angular-oauth2-oidc](https://github.com/manfredsteyer/angular-oauth2-oidc)，而不是自己获取访问令牌：

```typescript
@Injectable()
export class AppService {

    constructor(
        private _router: Router, private _http: Http, private oauthService: OAuthService){
        this.oauthService.loginUrl =
            'http://localhost:8081/spring-security-oauth-server/oauth/authorize';
        this.oauthService.redirectUri = 'http://localhost:8086/';
        this.oauthService.clientId = "sampleClientId";
        this.oauthService.scope = "read write foo bar";
        this.oauthService.setStorage(sessionStorage);
        this.oauthService.tryLogin({});
    }

    obtainAccessToken(){
        this.oauthService.initImplicitFlow();
    }

    getResource(resourceUrl) : Observable<Foo>{
        var headers =
            new Headers({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8',
                'Authorization': 'Bearer '+this.oauthService.getAccessToken()});
        var options = new RequestOptions({ headers: headers });
        return this._http.get(resourceUrl, options)
            .map((res:Response) => res.json())
            .catch((error:any) => Observable.throw(error.json().error || 'Server error'));
    }

    isLoggedIn(){
        if (this.oauthService.getAccessToken() === null){
            return false;
        }
        return true;
    }

    logout() {
        this.oauthService.logOut();
        location.reload();
    }
}
```

请注意，获取访问令牌后，每当我们在资源服务器中使用受保护的资源时，我们都会通过Authorization标头使用它。

### 6.2 主页组件

HomeComponent来处理我们的简单主页：

```typescript
@Component({
    selector: 'home-header',
    providers: [AppService],
    template: `
    <button *ngIf="!isLoggedIn" (click)="login()" type="submit">Login</button>
    <div *ngIf="isLoggedIn">
        <span>Welcome !!</span>
        <a (click)="logout()" href="#">Logout</a>
        <br/>
        <foo-details></foo-details>
    </div>`
})

export class HomeComponent {
    public isLoggedIn = false;

    constructor(
        private _service:AppService){}

    ngOnInit(){
        this.isLoggedIn = this._service.isLoggedIn();
    }

    login() {
        this._service.obtainAccessToken();
    }

    logout() {
        this._service.logout();
    }
}
```

### 6.3 Foo组件

FooComponent与密码流模块中的完全相同。

### 6.4 应用模块

最后，我们的AppModule：

```typescript
@NgModule({
    declarations: [
        AppComponent,
        HomeComponent,
        FooComponent
    ],
    imports: [
        BrowserModule,
        FormsModule,
        HttpModule,
        OAuthModule.forRoot(),
        RouterModule.forRoot([
            { path: '', component: HomeComponent }])
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

## 7. 运行前端

1. 要运行任何前端模块，我们需要先构建应用程序：

```shell
mvn clean install
```

2. 然后我们需要导航到我们的Angular应用程序目录：

```shell
cd src/main/resources
```

3. 最后，启动我们的应用程序：

```shell
npm start
```

服务器默认在4200端口启动，要更改任何模块的端口，请更改：

```text
"start": "ng serve"
```

在package.json中使其在端口8086上运行例如：

```text
"start": "ng serve --port 8086"
```

## 8. 总结

在本文中，我们学习了如何使用OAuth2授权我们的应用程序。