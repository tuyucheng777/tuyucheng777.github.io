---
layout: post
title:  使用Spring Security与CAS SSO
category: spring-security
copyright: spring-security
excerpt: Spring Data JPA
---

## 1. 概述

在本教程中，我们将研究Apereo CAS，并了解Spring Boot服务如何使用它进行身份验证。[CAS](https://apereo.github.io/cas/7.1.x/index.html)是一种[企业单点登录(SSO)](https://www.baeldung.com/cs/sso-guide)解决方案，也是开源的。

什么是SSO？当你使用相同的凭据登录YouTube、Gmail和地图时，这就是单点登录。我们将通过设置CAS服务器和Spring Boot应用来演示这一点。Spring Boot应用将使用CAS进行身份验证。

## 2. CAS服务器设置

### 2.1 CAS安装和依赖

服务器使用Maven(Gradle) War Overlay风格来简化设置和部署：

```shell
git clone https://github.com/apereo/cas-overlay-template.git cas-server
```

此命令将把cas-overlay-template克隆到cas-server目录中。

我们将介绍的一些方面包括JSON服务注册和JDBC数据库连接，因此，我们将把它们的模块添加到build.gradle文件的依赖部分：

```groovy
compile "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
compile "org.apereo.cas:cas-server-support-jdbc:${casServerVersion}"
```

让我们确保检查[casServer](https://github.com/apereo/cas/releases)的最新版本。

### 2.2 CAS服务器配置

在启动CAS服务器之前，我们需要添加一些基本配置。首先，创建一个cas-server/src/main/resources文件夹，然后在此文件夹中创建application.properties：

```properties
server.port=8443
spring.main.allow-bean-definition-overriding=true
server.ssl.key-store=classpath:/etc/cas/thekeystore
server.ssl.key-store-password=changeit
```

让我们继续创建上面配置中引用的密钥库文件。首先，我们需要在cas-server/src/main/resources中创建文件夹/etc/cas和/etc/cas/config。

然后，我们需要将目录更改为cas-server/src/main/resources/etc/cas并运行命令来生成keystore：

```shell
keytool -genkey -keyalg RSA -alias thekeystore -keystore thekeystore -storepass changeit -validity 360 -keysize 2048
```

为了避免出现SSL握手错误，我们应该使用localhost作为名字和姓氏的值，组织名称和单位也应使用相同的值。此外，我们需要将thekeystore导入到我们将用于运行客户端应用程序的JDK/JRE中：

```shell
keytool -importkeystore -srckeystore thekeystore -destkeystore $JAVA11_HOME/jre/lib/security/cacerts
```

源和目标密钥库的密码都是changeit。在Unix系统上，我们可能必须以管理员(sudo)权限运行此命令。导入后，我们应该重新启动正在运行的所有Java实例或重新启动系统。

我们使用JDK 11，因为CAS版本6.1.x需要它。此外，我们定义了指向其主目录的环境变量$JAVA11_HOME。现在我们可以启动CAS服务器：

```shell
./gradlew[.bat] run -Dorg.gradle.java.home=$JAVA11_HOME
```

**当应用程序启动时，我们会看到终端上打印“READY”**，并且服务器将在[https://localhost:8443](https://localhost:8443/)上可用。

### 2.3 CAS服务器用户配置

由于我们尚未配置任何用户，因此我们无法登录。CAS有不同的[管理配置](https://apereo.github.io/cas/7.1.x/configuration/Configuration-Server-Management.html)方法，包括独立模式。让我们创建一个配置文件夹cas-server/src/main/resources/etc/cas/config，在其中我们将创建一个属性文件cas.properties。现在，我们可以在属性文件中定义一个静态用户：

```properties
cas.authn.accept.users=casuser::Mellon
```

我们必须将配置文件夹的位置传达给CAS服务器以使设置生效，让我们更新task.gradle，以便我们可以从命令行将位置作为JVM参数传递：

```groovy
task run(group: "build", description: "Run theCASweb application in embedded container mode") {
    dependsOn 'build'
    doLast {
        def casRunArgs = new ArrayList<>(Arrays.asList(
                "-server -noverify -Xmx2048M -XX:+TieredCompilation -XX:TieredStopAtLevel=1".split(" ")))
        if (project.hasProperty('args')) {
            casRunArgs.addAll(project.args.split('\\s+'))
        }
        javaexec {
            main = "-jar"
            jvmArgs = casRunArgs
            args = ["build/libs/${casWebApplicationBinaryName}"]
            logger.info "Started ${commandLine}"
        }
    }
}
```

然后我们保存文件并运行：

```shell
./gradlew run
  -Dorg.gradle.java.home=$JAVA11_HOME
  -Pargs="-Dcas.standalone.configurationDirectory=/cas-server/src/main/resources/etc/cas/config"
```

请注意cas.standalone.configurationDirectory的值是绝对路径，我们现在可以转到https://localhost:8443并使用用户名casuser和密码Mellon登录。

## 3. CAS客户端设置

我们将使用[Spring Initializr](https://start.spring.io/)生成Spring Boot客户端应用程序，它将具有Web、Security、Freemarker和DevTools依赖。此外，我们还将在其pom.xml中添加[Spring Security CAS](https://mvnrepository.com/artifact/org.springframework.security/spring-security-cas)模块的依赖：

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-cas</artifactId>
    <versionId>5.3.0.RELEASE</versionId>
</dependency>
```

最后，让我们添加以下Spring Boot属性来配置应用程序：

```properties
server.port=8900
spring.freemarker.suffix=.ftl
```

## 4. CAS服务器服务注册

**客户端应用程序必须在任何身份验证之前向CAS服务器注册**，CAS服务器支持使用YAML、JSON、MongoDB和LDAP客户端注册表。

在本教程中，我们将使用JSON服务注册表方法。让我们再创建另一个文件夹cas-server/src/main/resources/etc/cas/services，这个文件夹将存放服务注册表JSON文件。

我们将创建一个包含客户端应用程序定义的JSON文件，文件的名称casSecuredApp-8900.json遵循以下模式serviceName-Id.json：

```json
{
    "@class" : "org.apereo.cas.services.RegexRegisteredService",
    "serviceId" : "http://localhost:8900/login/cas",
    "name" : "casSecuredApp",
    "id" : 8900,
    "logoutType" : "BACK_CHANNEL",
    "logoutUrl" : "http://localhost:8900/exit/cas"
}
```

serviceId属性为客户端应用程序定义一个正则表达式URL模式，该模式应与客户端应用程序的URL匹配。

id属性应具有唯一性，换句话说，不应有两个或多个具有相同id的服务注册到同一个CAS服务器，id重复会导致冲突和配置覆盖。

**我们还将注销类型配置为BACK_CHANNEL，将URL配置为[http://localhost:8900/exit/cas](http://localhost:8900/exit/cas)，以便我们稍后可以进行单点注销**。

在CAS服务器可以使用我们的JSON配置文件之前，我们必须在cas.properties中启用JSON注册表：

```properties
cas.serviceRegistry.initFromJson=true
cas.serviceRegistry.json.location=classpath:/etc/cas/services
```

## 5. CAS客户端单点登录配置

下一步是配置Spring Security以与CAS服务器配合使用，我们还应该检查[完整的交互流程](https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/html5/#cas-sequence)，称为CAS序列。

让我们将以下Bean配置添加到Spring Boot应用程序的CasSecuredApplication类中：

```java
@Bean
public CasAuthenticationFilter casAuthenticationFilter(AuthenticationManager authenticationManager, ServiceProperties serviceProperties) throws Exception {
    CasAuthenticationFilter filter = new CasAuthenticationFilter();
    filter.setAuthenticationManager(authenticationManager);
    filter.setServiceProperties(serviceProperties);
    return filter;
}

@Bean
public ServiceProperties serviceProperties() {
    logger.info("service properties");
    ServiceProperties serviceProperties = new ServiceProperties();
    serviceProperties.setService("http://cas-client:8900/login/cas");
    serviceProperties.setSendRenew(false);
    return serviceProperties;
}

@Bean
public TicketValidator ticketValidator() {
    return new Cas30ServiceTicketValidator("https://localhost:8443");
}

@Bean
public CasAuthenticationProvider casAuthenticationProvider(TicketValidator ticketValidator, ServiceProperties serviceProperties) {
    CasAuthenticationProvider provider = new CasAuthenticationProvider();
    provider.setServiceProperties(serviceProperties);
    provider.setTicketValidator(ticketValidator);
    provider.setUserDetailsService(
            s -> new User("test@test.com", "Mellon", true, true, true, true, AuthorityUtils.createAuthorityList("ROLE_ADMIN")));
    provider.setKey("CAS_PROVIDER_LOCALHOST_8900");
    return provider;
}
```

ServiceProperties Bean具有与casSecuredApp-8900.json中的serviceId相同的URL，这很重要，因为它向CAS服务器标识此客户端。

ServiceProperties的sendRenew属性设置为false，这意味着用户只需向服务器出示一次登录凭据。

AuthenticationEntryPoint Bean将处理身份验证异常，因此，它将把用户重定向到CAS服务器的登录URL进行身份验证。

总而言之，身份验证流程如下：

1. 用户尝试访问安全页面，从而触发身份验证异常
2. 异常触发AuthenticationEntryPoint；作为响应，AuthenticationEntryPoint将把用户带到CAS服务器登录页面-[https://localhost:8443/login](https://localhost:8443/login)
3. 身份验证成功后，服务器将使用票证重定向回客户端
4. CasAuthenticationFilter将获取重定向并调用CasAuthenticationProvider
5. CasAuthenticationProvider将使用TicketValidator来确认CAS服务器上提供的票证
6. 如果票证有效，用户将被重定向到所请求的安全URL

最后，让我们配置HttpSecurity以保护WebSecurityConfig中的一些路由。在此过程中，我们还将添加用于异常处理的身份验证入口点：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers( "/secured", "/login").authenticated()
        .and()
        .exceptionHandling().authenticationEntryPoint(authenticationEntryPoint())
        .and()
        .addFilterBefore(singleSignOutFilter, CasAuthenticationFilter.class)
}
```

## 6. CAS客户端单点注销配置

到目前为止，我们已经处理了单点登录；现在让我们考虑CAS单点注销(SLO)。

使用CAS管理用户身份验证的应用程序可以从两个地方注销用户：

- 客户端应用程序可以在本地注销用户-这不会影响用户在使用同一CAS服务器的其他应用程序中的登录状态。
- 客户端应用程序还可以从CAS服务器注销用户-这将导致用户从连接到同一CAS服务器的所有其他客户端应用程序中注销。

我们将首先在客户端应用程序上实现注销，然后将其扩展为CAS服务器上的单点注销。

为了使幕后发生的事情一目了然，我们将创建一个logout()方法来处理本地注销。成功后，它会将我们重定向到一个带有单点注销链接的页面：

```java
@GetMapping("/logout")
public String logout(
        HttpServletRequest request,
        HttpServletResponse response,
        SecurityContextLogoutHandler logoutHandler) {
    Authentication auth = SecurityContextHolder
            .getContext().getAuthentication();
    logoutHandler.logout(request, response, auth );
    new CookieClearingLogoutHandler(
            AbstractRememberMeServices.SPRING_SECURITY_REMEMBER_ME_COOKIE_KEY)
            .logout(request, response, auth);
    return "auth/logout";
}
```

在单点注销过程中，CAS服务器会先让用户的票证过期。然后向所有注册的客户端应用发送异步请求，每个收到此信号的客户端应用都会执行本地注销，从而达到一次注销的目的，从而导致所有客户端都注销。

话虽如此，让我们向客户端应用程序添加一些Bean配置，具体来说，在CasSecuredApplication中：

```java
@Bean
public SecurityContextLogoutHandler securityContextLogoutHandler() {
    return new SecurityContextLogoutHandler();
}

@Bean
public LogoutFilter logoutFilter() {
    LogoutFilter logoutFilter = new LogoutFilter("https://localhost:8443/logout",
            securityContextLogoutHandler());
    logoutFilter.setFilterProcessesUrl("/logout/cas");
    return logoutFilter;
}

@Bean
public SingleSignOutFilter singleSignOutFilter() {
    SingleSignOutFilter singleSignOutFilter = new SingleSignOutFilter();
    singleSignOutFilter.setLogoutCallbackPath("/exit/cas");
    singleSignOutFilter.setIgnoreInitConfiguration(true);
    return singleSignOutFilter;
}
```

logoutFilter将拦截对/logout/cas的请求并将应用程序重定向到CAS服务器，SingleSignOutFilter将拦截来自CAS服务器的请求并执行本地注销。

## 7. 将CAS服务器连接到数据库

我们可以配置CAS服务器以从MySQL数据库读取凭据，我们将使用在本地计算机中运行的MySQL服务器的测试数据库，让我们更新cas-server/src/main/resources/application.yml：

```yaml
cas:
    authn:
        accept:
            users:
        jdbc:
            query[0]:
                sql: SELECT * FROM users WHERE email = ?
                url: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
                dialect: org.hibernate.dialect.MySQLDialect
                user: root
                password: root
                ddlAuto: none
                driverClass: com.mysql.cj.jdbc.Driver
                fieldPassword: password
                passwordEncoder:
                    type: NONE
```

另外，在cas-secured-app cas-secured-app/src/main/resources/application.properties中进行相同的配置：

```properties
spring.jpa.generate-ddl=false
spring.datasource.url= jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

**我们将cas.authn.accept.users设置为空白，这将停用CAS服务器对静态用户存储库的使用**。

根据上面的SQL，用户的凭据存储在users表中，email列代表用户的主体(username)。

请务必检查[支持的数据库、可用驱动程序和方言列表](https://apereo.github.io/cas/7.1.x/installation/JDBC-Drivers.html)。我们还将密码编码器类型设置为NONE，其他[加密机制](https://apereo.github.io/cas/7.1.x/password_management/Password-Management-JDBC.html)及其特殊属性也可用。

注意CAS服务器数据库中的用户必须与客户端应用程序数据库中的用户相同。

让我们更新CasAuthenticationProvider以拥有与CAS服务器相同的用户名：

```java
@Bean
public CasUserDetailsService getUser(){
    return new CasUserDetailsService();
}

@Bean
public CasAuthenticationProvider casAuthenticationProvider(
        TicketValidator ticketValidator,
        ServiceProperties serviceProperties) {
    CasAuthenticationProvider provider = new CasAuthenticationProvider();
    provider.setServiceProperties(serviceProperties);
    provider.setTicketValidator(ticketValidator);
    provider.setUserDetailsService(getUser());
    provider.setKey("CAS_PROVIDER_LOCALHOST_8900");
    return provider;
}
```

CasAuthenticationProvider需要UserDetailsService来根据CAS票证加载用户详细信息，UserDetailsService负责从数据源(例如数据库)检索用户信息，在UserDetailsService实现的loadUserByUsername方法中，你可以自定义逻辑以根据提供的用户名加载用户详细信息。

```java
public class CasUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Get the user from the database.
        CasUser casUser = getUserFromDatabase(username);

        // Create a UserDetails object.
        UserDetails userDetails = new User(
                casUser.getEmail(),
                casUser.getPassword(),
                Collections.singletonList(new SimpleGrantedAuthority("ROLE_ADMIN")));

        return userDetails;
    }

    private CasUser getUserFromDatabase(String username) {
        return userRepository.findByEmail(username);
    }
}
```

loadUserByUsername方法是CasUserDetailsService类的一部分，此方法负责根据用户名加载用户的详细信息，你可以找到有关[使用数据库支持的UserDetailsService进行身份验证](https://www.baeldung.com/spring-security-authentication-with-a-database)的更多信息。

一旦CAS票证被验证并且用户详细信息被加载，CasAuthenticationProvider就会创建一个经过身份验证的Authentication对象，然后可用于应用程序中的授权和访问控制。

CasAuthenticationProvider不使用密码进行身份验证，尽管如此，其用户名必须与CAS服务器的用户名匹配，身份验证才能成功。CAS服务器要求MySQL服务器在localhost的3306端口上运行，用户名和密码应为root。

再次重新启动CAS服务器和Spring Boot应用程序，然后使用新的凭据进行身份验证。

## 8. 总结

我们已经了解了如何将CAS SSO与Spring Security结合使用以及所涉及的许多配置文件。CAS SSO还有许多其他方面可以配置，从主题和协议类型到身份验证策略。