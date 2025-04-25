---
layout: post
title:  CDI拦截器与Spring AspectJ
category: libraries
copyright: libraries
excerpt: CDI
---

## 1. 简介

拦截器模式通常用于在应用程序中添加新的、横切的功能或逻辑，并且在大量库中得到了坚实的支持。

在本文中，我们将介绍并对比其中两个主要库：CDI拦截器和Spring AspectJ。

## 2. CDI拦截器项目设置

Jakarta EE官方支持CDI，但一些实现也支持在Java SE环境中使用CDI，[Weld](http://weld.cdi-spec.org/)可以被视为Java SE支持的CDI实现的一个例子。

为了使用CDI，我们需要在POM中导入Weld库：

```xml
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se-core</artifactId>
    <version>3.1.6.Final</version>
</dependency>
```

最新的Weld库可以在[Maven仓库](https://mvnrepository.com/artifact/org.jboss.weld.se/weld-se-core)中找到。

现在让我们创建一个简单的拦截器。

## 3. CDI拦截器介绍

为了指定我们需要拦截的类，让我们创建拦截器绑定：

```java
@InterceptorBinding
@Target( { METHOD, TYPE } )
@Retention( RUNTIME )
public @interface Audited {
}
```

定义拦截器绑定后，我们需要定义实际的拦截器实现：

```java
@Audited
@Interceptor
public class AuditedInterceptor {
    public static boolean calledBefore = false;
    public static boolean calledAfter = false;

    @AroundInvoke
    public Object auditMethod(InvocationContext ctx) throws Exception {
        calledBefore = true;
        Object result = ctx.proceed();
        calledAfter = true;
        return result;
    }
}
```

每个@AroundInvoke方法都接收一个javax.interceptor.InvocationContext参数，返回一个java.lang.Object，并且可以抛出Exception。

因此，当我们使用新的@Audit接口标注一个方法时，auditMethod将首先被调用，然后目标方法才会继续执行。

## 4. 应用CDI拦截器

让我们将创建的拦截器应用到一些业务逻辑上：

```java
public class SuperService {
    @Audited
    public String deliverService(String uid) {
        return uid;
    }
}
```

我们创建了这个简单的服务，并用@Audited注解标注了我们想要拦截的方法。

要启用CDI拦截器，需要在META-INF目录中的beans.xml文件中指定完整的类名：

```xml
<beans xmlns="http://java.sun.com/xml/ns/javaee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/beans_1_2.xsd">
    <interceptors>
        <class>cn.tuyucheng.taketoday.interceptor.AuditedInterceptor</class>
    </interceptors>
</beans>
```

为了验证拦截器确实有效，我们现在运行以下测试：

```java
public class TestInterceptor {
    Weld weld;
    WeldContainer container;

    @Before
    public void init() {
        weld = new Weld();
        container = weld.initialize();
    }

    @After
    public void shutdown() {
        weld.shutdown();
    }

    @Test
    public void givenTheService_whenMethodAndInterceptorExecuted_thenOK() {
        SuperService superService = container.select(SuperService.class).get();
        String code = "123456";
        superService.deliverService(code);

        Assert.assertTrue(AuditedInterceptor.calledBefore);
        Assert.assertTrue(AuditedInterceptor.calledAfter);
    }
}
```

在这个快速测试中，我们首先从容器中获取Bean SuperService，然后在其上调用业务方法deliverService，并通过验证其状态变量来检查拦截器AuditedInterceptor是否确实被调用。

我们还有@Before和@After注解方法，分别用于初始化和关闭Weld容器。

## 5. CDI注意事项

我们可以指出CDI拦截器具有以下优点：

- 这是Jakarta EE规范的标准功能
- 一些CDI实现库可以在Java SE中使用
- 当我们的项目对第三方库有严重限制时可以使用

CDI拦截器的缺点如下：

- 具有业务逻辑的类和拦截器之间的紧密耦合
- 很难看出项目中哪些类被拦截了
- 缺乏灵活的机制将拦截器应用于一组方法

## 6. Spring AspectJ

Spring也支持使用AspectJ语法实现类似的拦截器功能。

首先，我们需要向POM添加以下Spring和AspectJ依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.2.1</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.20.1</version>
</dependency>
```

可以在Maven仓库中找到[spring-context](https://mvnrepository.com/artifact/org.springframework/spring-context)、[aspectjweaver](https://mvnrepository.com/artifact/org.aspectj/aspectjweaver)的最新版本。

我们现在可以使用AspectJ注解语法创建一个简单的切面：

```java
@Aspect
public class SpringTestAspect {
    @Autowired
    private List accumulator;

    @Around("execution(* cn.tuyucheng.taketoday.spring.service.SpringSuperService.*(..))")
    public Object auditMethod(ProceedingJoinPoint jp) throws Throwable {
        String methodName = jp.getSignature().getName();
        accumulator.add("Call to " + methodName);
        Object obj = jp.proceed();
        accumulator.add("Method called successfully: " + methodName);
        return obj;
    }
}
```

我们创建了一个适用于SpringSuperService类的所有方法的切面-为简单起见，它看起来像这样：

```java
public class SpringSuperService {
    public String getInfoFromService(String code) {
        return code;
    }
}
```

## 7. Spring AspectJ切面应用

为了验证该切面确实适用于该服务，让我们编写以下单元测试：

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = { AppConfig.class })
public class TestSpringInterceptor {
    @Autowired
    SpringSuperService springSuperService;

    @Autowired
    private List accumulator;

    @Test
    public void givenService_whenServiceAndAspectExecuted_thenOk() {
        String code = "123456";
        String result = springSuperService.getInfoFromService(code);

        Assert.assertThat(accumulator.size(), is(2));
        Assert.assertThat(accumulator.get(0), is("Call to getInfoFromService"));
        Assert.assertThat(accumulator.get(1), is("Method called successfully: getInfoFromService"));
    }
}
```

在这个测试中，我们注入我们的服务，调用方法并检查结果。

配置如下：

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
    @Bean
    public SpringSuperService springSuperService() {
        return new SpringSuperService();
    }

    @Bean
    public SpringTestAspect springTestAspect() {
        return new SpringTestAspect();
    }

    @Bean
    public List getAccumulator() {
        return new ArrayList();
    }
}
```

@EnableAspectJAutoProxy注解中的一个重要方面是，它支持处理标有AspectJ的@Aspect注解的组件，类似于Spring的XML元素中的功能。

## 8. Spring AspectJ注意事项

让我们指出使用Spring AspectJ的一些优点：

- 拦截器与业务逻辑分离
- 拦截器可以从依赖注入中受益
- 拦截器本身包含所有配置信息
- 添加新的拦截器不需要扩充现有代码
- 拦截器具有灵活的机制来选择要拦截的方法
- 无需Jakarta EE即可使用

当然也有一些缺点：

- 你需要了解AspectJ语法才能开发拦截器
- AspectJ拦截器的学习曲线比CDI拦截器的学习曲线要高

## 9. CDI拦截器与Spring AspectJ

如果你当前的项目使用Spring，那么考虑Spring AspectJ是一个不错的选择。

如果你使用的是功能齐全的应用程序服务器，或者你的项目不使用Spring(或其他框架，例如Google Guice)并且严格使用Jakarta EE，那么除了选择CDI拦截器之外别无选择。

## 10. 总结

本文介绍了拦截器模式的两种实现：CDI拦截器和Spring AspectJ，并分别介绍了它们各自的优缺点。