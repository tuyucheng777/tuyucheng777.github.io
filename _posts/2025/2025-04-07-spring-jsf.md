---
layout: post
title:  JSF与Spring
category: webmodules
copyright: webmodules
excerpt: JSF
---

## 1. 概述

在本文中，我们将研究从JSF管理Bean和JSF页面中访问Spring中定义的Bean的方法，以便将业务逻辑的执行委托给Spring Bean。

本文假设读者对JSF和Spring有初步了解，本文基于JSF的[Mojarra实现](https://javaee.github.io/javaserverfaces/index-0.html)。

## 2. Spring

让我们在Spring中定义以下Bean，UserManagementDAO Bean将用户名添加到内存存储中，它由以下接口定义：

```java
public interface UserManagementDAO {
    boolean createUser(String newUserData);
}
```

使用以下Java配置来配置Bean的实现：

```java
public class SpringCoreConfig {
    @Bean
    public UserManagementDAO userManagementDAO() {
        return new UserManagementDAOImpl();
    }
}
```

或者使用以下XML配置：

```xml
<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor" />
<bean class="cn.tuyucheng.taketoday.dao.UserManagementDAOImpl" id="userManagementDAO"/>
```

我们在XML中定义Bean，并注册CommonAnnotationBeanPostProcessor以确保@PostConstruct注解被识别。

## 3. 配置

以下部分解释了实现Spring和JSF上下文集成的配置项。

### 3.1 不使用web.xml的Java配置

通过实现WebApplicationInitializer，我们能够以编程方式配置ServletContext，以下是MainWebAppInitializer类中的onStartup()实现：

```java
public void onStartup(ServletContext sc) throws ServletException {
    AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
    root.register(SpringCoreConfig.class);
    sc.addListener(new ContextLoaderListener(root));
}
```

AnnotationConfigWebApplicationContext引导Spring上下文并通过注册SpringCoreConfig类添加Bean。

类似地，在Mojarra实现中，有一个FacesInitializer类用于配置FacesServlet。要使用此配置，只需扩展FacesInitializer即可。MainWebAppInitializer的完整实现现在如下：

```java
public class MainWebAppInitializer extends FacesInitializer implements WebApplicationInitializer {
    public void onStartup(ServletContext sc) throws ServletException {
        AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
        root.register(SpringCoreConfig.class);
        sc.addListener(new ContextLoaderListener(root));
    }
}
```

### 3.2 使用web.xml

我们首先在应用程序的web.xml文件中配置ContextLoaderListener：

```xml
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

此监听器负责在Web应用程序启动时启动Spring应用程序上下文，默认情况下，此监听器将查找名为applicationContext.xml的Spring配置文件。

### 3.3 faces-config.xml

我们现在在face-config.xml文件中配置SpringBeanFacesELResolver：

```xml
<el-resolver>org.springframework.web.jsf.el.SpringBeanFacesELResolver</el-resolver>
```

EL解析器是JSF框架支持的可插拔组件，它允许我们在评估表达式语言(EL)表达式时自定义JSF运行时的行为，此EL解析器将允许JSF运行时通过JSF中定义的EL表达式访问Spring组件。

## 4. 在JSF中访问Spring Bean

此时，我们的JSF Web应用程序已准备好从JSF支持Bean或JSF页面访问我们的Spring Bean。

### 4.1 从Backing Bean JSF 2.0

现在可以从JSF支持bean访问Spring Bean，根据你运行的JSF版本，有两种可能的方法。使用JSF 2.0，你可以在JSF托管Bean上使用@ManagedProperty注解。

```java
@ManagedBean(name = "registration")
@RequestScoped
public class RegistrationBean implements Serializable {
    @ManagedProperty(value = "#{userManagementDAO}")
    transient private IUserManagementDAO theUserDao;

    private String userName;
    // getters and setters
}
```

请注意，使用@ManagedProperty时，Getter和Setter是必需的。

现在，为了从托管Bean断言Spring Bean的可访问性，我们将添加createNewUser()方法：

```java
public void createNewUser() {
    FacesContext context = FacesContext.getCurrentInstance();
    boolean operationStatus = userDao.createUser(userName);
    context.isValidationFailed();
    if (operationStatus) {
        operationMessage = "User " + userName + " created";
    }
}
```

该方法的要点是使用userDao Spring Bean，并访问其功能。

### 4.2 JSF 2.2中的Backing Bean

另一种方法(仅在JSF 2.2及更高版本中有效)是使用CDI的@Inject注解，这适用于JSF托管Bean(使用@ManagedBean注解)和CDI托管Bean(使用@Named注解)。

确实，使用CDI注解，这是注入Bean的唯一有效方法：

```java
@Named( "registration")
@RequestScoped
public class RegistrationBean implements Serializable {
    @Inject
    UserManagementDAO theUserDao;
}
```

使用这种方法，Getter和Setter就不再是必需的了。还请注意，EL表达式不存在。

### 4.3 从JSF视图

createNewUser()方法将从以下JSF页面触发：

```html
<h:form>
    <h:panelGrid id="theGrid" columns="3">
        <h:outputText value="Username"/>
        <h:inputText id="firstName" binding="#{userName}" required="true"
                     requiredMessage="#{msg['message.valueRequired']}" value="#{registration.userName}"/>
        <h:message for="firstName" style="color:red;"/>
        <h:commandButton value="#{msg['label.saveButton']}" action="#{registration.createNewUser}"
                         process="@this"/>
        <h:outputText value="#{registration.operationMessage}" style="color:green;"/>
    </h:panelGrid>
</h:form>
```

要呈现页面，请启动服务器并导航至：

```text
http://localhost:8080/jsf/index.jsf
```

我们还可以在JSF视图中使用EL来访问Spring Bean，要测试它，只需将之前引入的JSF页面中的第7行更改为：

```xml
<h:commandButton value="Save" action="#{registration.userDao.createUser(userName.value)}"/>
```

在这里，我们直接在Spring DAO上调用createUser方法，将userName的绑定值从JSF页面内部传递给该方法，从而完全绕过托管Bean。

## 5. 总结

我们研究了Spring和JSF上下文之间的基本集成，能够在JSF Bean和页面中访问Spring Bean。

值得注意的是，虽然JSF运行时提供了可插拔式架构，使得Spring框架能够提供集成组件，但是Spring框架的注解不能在JSF上下文中使用，反之亦然。

这意味着你无法在JSF托管Bean中使用@Autowired或@Component等注解，也无法在Spring托管Bean上使用@ManagedBean注解。但是，你可以在JSF 2.2+托管Bean和Spring Bean中使用@Inject注解(因为Spring支持JSR-330)。