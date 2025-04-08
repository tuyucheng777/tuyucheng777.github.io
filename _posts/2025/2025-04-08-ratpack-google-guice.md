---
layout: post
title:  Ratpack Google Guice集成
category: webmodules
copyright: webmodules
excerpt: Ratpack
---

## 1. 概述

在我们之前的[文章](https://www.baeldung.com/ratpack)中，我们展示了如何使用Ratpack构建可扩展的应用程序。

在本教程中，我们将进一步讨论如何使用Google Guice和[Ratpack](https://ratpack.io/)作为依赖管理引擎。

## 2. 为什么选择Google Guice？

[Google Guice](https://github.com/google/guice)是Google根据Apache许可证发布的Java平台开源软件框架。

它是一个非常轻量的依赖管理模块，易于配置。此外，为了方便使用，它只允许构造函数级别的依赖注入。

关于Guice的更多详细信息请参见[此处](https://www.baeldung.com/guice)。

## 3. 使用Guice和Ratpack

### 3.1 Maven依赖

Ratpack对Guice依赖提供一流的支持，因此，我们不必手动为Guice添加任何外部依赖；它已随Ratpack预构建。有关Ratpack的Guice支持的更多详细信息，请参见[此处](https://ratpack.io/manual/current/guice.html)。

因此，我们只需要在pom.xml中添加以下核心Ratpack依赖：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.4.5</version>
</dependency>
```

你可以在[Maven Central](https://mvnrepository.com/artifact/io.ratpack/ratpack-core)上查看最新版本。

### 3.2 构建服务模块

完成Maven配置后，我们将构建一个服务，并在此示例中充分利用一些简单的依赖注入。

让我们创建一个服务接口和一个服务类：

```java
public interface DataPumpService {
    String generate();
}
```

这是将充当注入器的服务接口。现在，我们必须构建将实现该接口的服务类，并定义服务方法generate()：

```java
public class DataPumpServiceImpl implements DataPumpService {

    @Override
    public String generate() {
        return UUID.randomUUID().toString();
    }
}
```

这里要注意的一点是，由于我们使用的是Ratpack的Guice模块，**因此我们不需要使用Guice的@ImplementedBy或@Inject注解来手动注入服务类**。

### 3.3 依赖管理

有两种方法可以使用Google Guice执行依赖管理。

第一种是使用Guice的[AbstractModule](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/AbstractModule.html)，另一种是使用Guice的[实例绑定](https://github.com/google/guice/wiki/InstanceBindings)机制方法：

```java
public class DependencyModule extends AbstractModule {

    @Override
    public void configure() {
        bind(DataPumpService.class).to(DataPumpServiceImpl.class)
                .in(Scopes.SINGLETON);
    }
}
```

这里有几点需要注意：

- 通过扩展AbstractModule，我们覆盖了默认的configure()方法
- 我们将DataPumpServiceImpl类与DataPumpService接口进行映射，该接口是之前构建的服务层
- 以Singleton的方式注入了依赖

### 3.4 与现有应用程序集成

依赖管理配置已经准备好了，现在让我们来集成它：

```java
public class Application {

    public static void main(String[] args) throws Exception {
        RatpackServer
                .start(server -> server.registry(Guice
                                .registry(bindings -> bindings.module(DependencyModule.class)))
                        .handlers(chain -> chain.get("randomString", ctx -> {
                            DataPumpService dataPumpService = ctx.get(DataPumpService.class);
                            ctx.render(dataPumpService.generate().length());
                        })));
    }
}
```

在这里，使用registry()-我们绑定了扩展AbstractModule的DependencyModule类，Ratpack的Guice模块将在内部完成其余需要的操作，并在应用程序Context中注入服务。

由于它在应用程序上下文中可用，我们现在可以从应用程序中的任何位置获取服务实例。在这里，我们从当前上下文中获取了DataPumpService实例，并使用服务的generate()方法映射了/randomString URL。

**因此，每当访问/randomString URL时，它都会返回随机字符串片段**。

### 3.5 运行时实例绑定

如前所述，我们现在将使用Guice的实例绑定机制在运行时进行依赖管理。除了使用Guice的bindInstance()方法而不是AbstractModule来注入依赖外，它几乎与以前的技术相同：

```java
public class Application {

    public static void main(String[] args) throws Exception {
        RatpackServer.start(server -> server
                .registry(Guice.registry(bindings -> bindings
                        .bindInstance(DataPumpService.class, new DataPumpServiceImpl())))
                .handlers(chain -> chain.get("randomString", ctx -> {
                    DataPumpService dataPumpService = ctx.get(DataPumpService.class);
                    ctx.render(dataPumpService.generate());
                })));
    }
}
```

这里，通过使用bindInstance()，我们执行实例绑定，即将DataPumpService接口注入到DataPumpServiceImpl类。

通过这种方式，我们可以将服务实例注入到应用程序上下文中，就像我们在前面的例子中所做的那样。

虽然我们可以使用这两种技术中的任何一种进行依赖管理，但最好使用AbstractModule，因为它将依赖管理模块与应用程序代码完全分开。这样，代码将更加干净，将来也更易于维护。

### 3.6 工厂绑定

还有一种依赖管理方法，称为工厂绑定。它与Guice的实现没有直接关系，但也可以与Guice并行工作。

工厂类将客户端与实现分离，简单工厂使用静态方法来获取和设置接口的Mock实现。

我们可以使用已经创建的服务类来启用工厂绑定，我们只需要创建一个工厂类，就像DependencyModule(它扩展了Guice的AbstractModule类)一样，并通过静态方法绑定实例：

```java
public class ServiceFactory {

    private static DataPumpService instance;

    public static void setInstance(DataPumpService dataPumpService) {
        instance = dataPumpService;
    }

    public static DataPumpService getInstance() {
        if (instance == null) {
            return new DataPumpServiceImpl();
        }
        return instance;
    }
}
```

**在这里，我们在工厂类中静态注入服务接口**。因此，一次只能有一个该接口的实例可用于此工厂类。然后，我们创建了常规的Getter/Setter方法来设置和获取服务实例。

这里要注意的是，在Getter方法中我们进行了一次明确的检查以确保只存在服务的单个实例；如果它为空，那么我们只会创建实现服务类的实例并返回相同的实例。

此后，我们可以在应用程序链中使用该工厂实例：

```java
.get("factory", ctx -> ctx.render(ServiceFactory.getInstance().generate()))
```

## 4. 测试

我们将使用Ratpack的[MainClassApplicationUnderTest](https://ratpack.io/manual/current/api/ratpack/test/MainClassApplicationUnderTest.html)来测试我们的应用程序，借助Ratpack的内部JUnit测试框架，我们必须为其添加必要的依赖([ratpack-test](https://mvnrepository.com/artifact/io.ratpack/ratpack-test))。

这里需要注意的是，由于URL内容是动态的，我们在编写测试用例时无法预测它。因此，我们将在测试用例中匹配/randomString URL端点的内容长度：

```java
@RunWith(JUnit4.class)
public class ApplicationTest {

    MainClassApplicationUnderTest appUnderTest
            = new MainClassApplicationUnderTest(Application.class);

    @Test
    public void givenStaticUrl_getDynamicText() {
        assertEquals(21, appUnderTest.getHttpClient()
                .getText("/randomString").length());
    }

    @After
    public void shutdown() {
        appUnderTest.close();
    }
}
```

## 5. 总结

在这篇简短的文章中，我们展示了如何将Google Guice与Ratpack结合使用。