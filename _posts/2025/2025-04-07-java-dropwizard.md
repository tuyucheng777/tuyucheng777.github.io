---
layout: post
title:  Dropwizard简介
category: webmodules
copyright: webmodules
excerpt: Dropwizard
---

## 1. 概述

**Dropwizard是一个开源Java框架，用于快速开发高性能RESTful Web服务**。它收集了一些流行的库来创建轻量级包，使用的主要库是Jetty、Jersey、Jackson、JUnit和Guava。此外，它还使用自己的库[Metrics](https://www.baeldung.com/dropwizard-metrics)。

在本教程中，我们将学习如何配置和运行一个简单的Dropwizard应用程序。完成后，我们的应用程序将公开一个RESTful API，允许我们获取存储的品牌列表。

## 2. Maven依赖

首先，[dropwizard-core](https://mvnrepository.com/artifact/io.dropwizard/dropwizard-core)依赖是我们创建服务所需的全部内容，让我们将其添加到pom.xml中：

```xml
<dependency>
    <groupId>io.dropwizard</groupId>
    <artifactId>dropwizard-core</artifactId>
    <version>2.0.0</version>
</dependency>
```

## 3. 配置

现在，我们将创建每个Dropwizard应用程序运行所需的必要类。

Dropwizard应用程序将属性存储在YML文件中，因此，我们将在资源目录中创建presentation-config.yml文件：

```yaml
defaultSize: 5
```

我们可以通过创建一个扩展io.dropwizard.Configuration的类来访问该文件中的值：

```java
public class BasicConfiguration extends Configuration {
    @NotNull private final int defaultSize;

    @JsonCreator
    public BasicConfiguration(@JsonProperty("defaultSize") int defaultSize) {
        this.defaultSize = defaultSize;
    }

    public int getDefaultSize() {
        return defaultSize;
    }
}
```

**Dropwizard使用[Jackson](https://www.baeldung.com/jackson)将配置文件反序列化到我们的类中**，因此，我们使用了Jackson的注解。

接下来，让我们创建主应用程序类，它负责准备我们的服务以供使用：

```java
public class IntroductionApplication extends Application<BasicConfiguration> {

    public static void main(String[] args) throws Exception {
        new IntroductionApplication().run("server", "introduction-config.yml");
    }

    @Override
    public void run(BasicConfiguration basicConfiguration, Environment environment) {
        //register classes
    }

    @Override
    public void initialize(Bootstrap<BasicConfiguration> bootstrap) {
        bootstrap.setConfigurationSourceProvider(new ResourceConfigurationSourceProvider());
        super.initialize(bootstrap);
    }
}
```

首先，main方法负责运行应用程序；我们可以将参数传递给run方法，也可以自己填充。

**第一个参数可以是server或check**，check选项用于验证配置，而server选项用于运行应用程序；**第二个参数是配置文件的位置**。 

此外，initialize方法将配置提供程序设置为ResourceConfigurationSourceProvider，这允许应用程序在资源目录中找到给定的配置文件；不必重写此方法。

最后，run方法允许我们访问Environment和BaseConfiguration，我们将在本文后面使用它们。

## 4. 资源

首先，让我们为我们的品牌创建一个域类：

```java
public class Brand {
    private final Long id;
    private final String name;

    // all args constructor and getters
}
```

其次，让我们创建一个负责返回品牌的BrandRepository类：

```java
public class BrandRepository {
    private final List<Brand> brands;

    public BrandRepository(List<Brand> brands) {
        this.brands = ImmutableList.copyOf(brands);
    }

    public List<Brand> findAll(int size) {
        return brands.stream()
                .limit(size)
                .collect(Collectors.toList());
    }

    public Optional<Brand> findById(Long id) {
        return brands.stream()
                .filter(brand -> brand.getId().equals(id))
                .findFirst();
    }
}
```

此外，**我们能够使用[Guava](https://www.baeldung.com/guava-collections)的ImmutableList，因为它是Dropwizard本身的一部分**。

第三，我们将创建一个BrandResource类。**Dropwizard默认使用[JAX-RS](https://www.baeldung.com/jax-rs-spec-and-implementations)，并以Jersey作为实现**。因此，我们将使用此规范中的注解来公开我们的REST API端点：

```java
@Path("/brands")
@Produces(MediaType.APPLICATION_JSON)
public class BrandResource {
    private final int defaultSize;
    private final BrandRepository brandRepository;

    public BrandResource(int defaultSize, BrandRepository brandRepository) {
        this.defaultSize = defaultSize;
        this.brandRepository = brandRepository;
    }

    @GET
    public List<Brand> getBrands(@QueryParam("size") Optional<Integer> size) {
        return brandRepository.findAll(size.orElse(defaultSize));
    }

    @GET
    @Path("/{id}")
    public Brand getById(@PathParam("id") Long id) {
        return brandRepository
                .findById(id)
                .orElseThrow(RuntimeException::new);
    }
}
```

此外，我们将size定义为Optional，以便在未提供参数时使用配置中的defaultSize。

最后，我们将在IntroductionApplication类中注册BrandResource，为了做到这一点，让我们实现run方法：

```java
@Override
public void run(BasicConfiguration basicConfiguration, Environment environment) {
    int defaultSize = basicConfiguration.getDefaultSize();
    BrandRepository brandRepository = new BrandRepository(initBrands());
    BrandResource brandResource = new BrandResource(defaultSize, brandRepository);

    environment
            .jersey()
            .register(brandResource);
}
```

**所有创建的资源都应该在此方法中注册**。

## 5. 运行应用程序

在本节中，我们将学习如何从命令行运行应用程序。

首先，我们将配置项目以使用[maven-shade-plugin](https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-shade-plugin)构建JAR文件：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <configuration>
        <createDependencyReducedPom>true</createDependencyReducedPom>
        <filters>
            <filter>
                <artifact>*:*</artifact>
                <excludes>
                    <exclude>META-INF/*.SF</exclude>
                    <exclude>META-INF/*.DSA</exclude>
                    <exclude>META-INF/*.RSA</exclude>
                </excludes>
            </filter>
        </filters>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer
                      implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>cn.tuyucheng.taketoday.dropwizard.introduction.IntroductionApplication</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这是插件的建议配置，此外，我们在<mainClass\>元素中包含了主类的路径。

最后，我们将使用[Maven](https://www.baeldung.com/maven)构建应用程序。一旦我们有了JAR文件，我们就可以运行该应用程序：

```shell
java -jar target/dropwizard-1.0.0.jar
```

**不需要传递参数，因为我们已经将它们包含在IntroductionApplication类中**。

此后，控制台日志应以以下内容结束：

```text
INFO  [2020-01-08 18:55:06,527] org.eclipse.jetty.server.Server: Started @1672ms
```

现在，应用程序监听端口8080，我们可以通过http://localhost:8080/brands访问我们的品牌端点。

## 6. 更改默认端口

默认情况下，典型的Dropwizard应用程序使用两个端口：

- 主应用程序的端口8080
- 管理界面的端口8081

但是，我们可以使用配置文件夹或命令行参数更改默认端口。

### 6.1 通过配置文件更改端口

通过配置文件来改变默认端口，我们需要修改资源目录中的introduction-config.yml：

```yaml
server:
    applicationConnectors:
        -   type: http
            port: 9090
    adminConnectors:
        -   type: http
            port: 9091
```

此配置将应用程序设置为在端口9090上运行，并将管理界面设置为在端口9091上运行；我们可以通过将配置文件传递给run()方法来应用此配置：

```java
// ...
new IntroductionApplication().run("server", "introduction-config.yml");
// ...
```

另外，我们可以在启动应用程序时通过命令行应用配置文件：

```shell
$ java -jar target/dropwizard-1.0.0.jar server introduction-config.yml
```

在上面的命令中，我们使用server和introduction-config.yml参数启动应用程序。

### 6.2 通过命令行更改端口

或者，我们可以使用命令行上的JVM系统属性覆盖端口设置：

```shell
$ java -Ddw.server.applicationConnectors[0].port=9090 \  
  -Ddw.server.adminConnectors[0].port=9091 \ 
  -jar target/dropwizard-1.0.0.jar server
```

-Ddw前缀定义Dropwizard特定的系统属性，用于设置主应用程序端口和管理界面端口。如果指定了这些系统属性，则会覆盖配置文件设置。如果未定义系统属性，则使用配置文件中的值。

## 7. 健康检查

启动应用程序时，我们被告知该应用程序没有任何健康检查。幸运的是，**Dropwizard提供了一种简单的解决方案来为我们的应用程序添加健康检查**。

让我们首先添加一个扩展com.codahale.metrics.health.HealthCheck的简单类：

```java
public class ApplicationHealthCheck extends HealthCheck {
    @Override
    protected Result check() throws Exception {
        return Result.healthy();
    }
}
```

这个简单的方法将返回有关我们组件健康状况的信息，我们可以创建多个健康检查，其中一些可能在某些情况下失败。例如，如果与数据库的连接失败，我们将返回Result.unhealthy()。

最后，我们需要在IntroductionApplication类的run方法中注册我们的健康检查：

```java
environment
    .healthChecks()
    .register("application", new ApplicationHealthCheck());
```

运行应用程序后，我们可以在http://localhost:8081/healthcheck访问健康检查响应：

```json
{
    "application": {
        "healthy": true,
        "duration": 0
    },
    "deadlocks": {
        "healthy": true,
        "duration": 0
    }
}
```

可以看到，我们的健康检查已经注册到了application标签下。

## 8. 总结

在本文中，我们学习了如何使用Maven设置Dropwizard应用程序。

应用程序的基本设置非常简单快捷，此外，Dropwizard还包含运行高性能RESTful Web服务所需的所有库。
