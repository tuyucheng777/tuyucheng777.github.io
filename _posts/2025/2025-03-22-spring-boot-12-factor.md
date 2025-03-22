---
layout: post
title:  Spring Boot微服务中的十二要素方法
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将了解[十二因素应用程序方法](https://12factor.net/)。

我们还将了解如何借助Spring Boot开发微服务。在此过程中，我们将了解如何应用十二要素方法来开发此类微服务。

## 2. 什么是十二因素方法论？

**十二要素方法是一套十二种最佳实践，用于开发以服务形式运行的应用程序**，它最初是由Heroku于2011年为在其云平台上部署为服务的应用程序起草的。随着时间的推移，事实证明，它对于任何[软件即服务](https://en.wikipedia.org/wiki/Software_as_a_service)(SaaS)开发来说都足够通用。

那么，我们所说的软件即服务是什么意思呢？传统上，我们设计、开发、部署和维护软件解决方案，以从中获取商业价值。但是，我们不必参与这个过程就能获得相同的结果。例如，计算适用税是许多领域的通用功能。

现在，我们可以决定自己构建和管理这项服务，或者**订购商业服务，这些服务就是我们所说的软件即服务**。

虽然软件即服务不会对其开发的架构施加任何限制；但采用一些最佳实践非常有用。

如果我们将软件设计为模块化、可移植且可在现代云平台上扩展的，那么它就非常适合我们的服务产品，这就是十二要素方法论发挥作用的地方。我们将在本教程的后面部分看到它们的实际作用。

## 3. 使用Spring Boot进行微服务

[微服务](https://www.baeldung.com/spring-microservices-guide)是一种**将软件开发为松散耦合服务的架构风格**，这里的关键要求是服务应围绕业务领域边界进行组织，这通常是最难识别的部分。

此外，这里的服务对其数据拥有唯一的权限，并向其他服务公开操作。服务之间的通信通常通过HTTP等轻量级协议进行，这促进服务可以独立部署和扩展。

现在，微服务架构和软件即服务并不相互依赖。但是，不难理解，**在开发软件即服务时，利用微服务架构非常有益**。它有助于实现我们之前讨论的许多目标，例如模块化和可扩展性。

[Spring Boot](https://spring.io/projects/spring-boot)是一个基于Spring的应用程序框架，它省去了开发企业应用程序所需的大量样板代码。它为我们提供了一个[高度规范](https://www.baeldung.com/cs/opinionated-software-design)但又灵活的平台来开发微服务。在本教程中，我们将利用Spring Boot使用十二要素方法交付微服务。

## 4. 应用十二因素方法

现在让我们定义一个简单的应用程序，我们将尝试使用我们刚刚讨论的工具和实践来开发它。我们都喜欢看电影，但要记住我们已经看过的电影却是一项挑战。

现在，我们需要一个简单的服务来记录和查询我们看过的电影：

![](/assets/images/2025/springboot/springboot12factor01.png)

这是一个非常简单且标准的微服务，具有数据存储和REST端点。我们需要定义一个映射到持久层的模型：

```java
@Entity
public class Movie {
    @Id
    private Long id;
    private String title;
    private String year;
    private String rating;
    // getters and setters
}
```

我们已经定义了一个具有id和一些其他属性的JPA实体，现在让我们看看REST控制器是什么样子的：

```java
@RestController
public class MovieController {

    @Autowired
    private MovieRepository movieRepository;
    @GetMapping("/movies")
    public List<Movie> retrieveAllStudents() {
        return movieRepository.findAll();
    }

    @GetMapping("/movies/{id}")
    public Movie retrieveStudent(@PathVariable Long id) {
        return movieRepository.findById(id).get();
    }

    @PostMapping("/movies")
    public Long createStudent(@RequestBody Movie movie) {
        return movieRepository.save(movie).getId();
    }
}
```

这涵盖了我们简单服务的基础，我们将在以下小节中讨论如何实施十二因素方法，并介绍应用程序的其余部分。

### 4.1 代码库

十二要素应用的第一个最佳实践是在版本控制系统中跟踪它，[Git](https://git-scm.com/)是当今使用最广泛的版本控制系统，几乎无处不在。该原则规定，**应在单个代码仓库中跟踪应用程序，并且不得与任何其他应用程序共享该仓库**。

Spring Boot提供了许多方便的方式来引导应用程序，包括命令行工具和[Web界面](https://start.spring.io/)。一旦我们生成引导应用程序，我们就可以将其转换为Git仓库：

```shell
git init
```

此命令应从应用程序的根目录运行。此阶段的应用程序已包含一个.gitignore文件，该文件可有效限制生成的文件进行版本控制。因此，我们可以直接创建一个初始提交：

```shell
git add .
git commit -m "Adding the bootstrap of the application."
```

最后，如果愿意的话，我们可以添加一个远程并将我们的提交推送到远程(这不是严格的要求)：

```shell
git remote add origin https://github.com/<username>/12-factor-app.git
git push -u origin master
```

### 4.2 依赖

接下来，**十二要素应用程序应始终显式声明其所有依赖**，我们应该使用依赖声明清单来执行此操作。Java有多个依赖项管理工具，如Maven和Gradle，我们可以使用其中之一来实现此目标。

因此，我们的简单应用程序依赖于一些外部库，例如用于促进REST API和连接到数据库的库。让我们看看如何使用Maven声明式地定义它们。

Maven要求我们在XML文件中描述项目的依赖关系，通常称为[项目对象模型](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)(POM)：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

虽然这看起来很简单，但这些依赖项通常还有其他传递依赖，这在一定程度上使其变得复杂，但有助于我们实现目标。现在，我们的应用程序没有未明确描述的直接依赖项。

### 4.3 配置

应用程序通常具有许多配置，其中一些配置可能在部署之间有所不同，而其他配置则保持不变。

在我们的示例中，我们有一个持久数据库。我们需要要连接的数据库的地址和凭据，这很可能在部署之间发生变化。

**十二要素应用程序应将所有此类因部署而异的配置外部化**，此处的建议是对此类配置使用环境变量，这样可以完全分离配置和代码。

Spring提供了一个配置文件，我们可以在其中声明这样的配置并将其附加到环境变量：

```properties
spring.datasource.url=jdbc:mysql://${MYSQL_HOST}:${MYSQL_PORT}/movies
spring.datasource.username=${MYSQL_USER}
spring.datasource.password=${MYSQL_PASSWORD}
```

在这里，我们将数据库URL和凭据定义为配置，并映射了要从环境变量中选择的实际值。

在Windows上，我们可以在启动应用程序之前设置环境变量：

```shell
set MYSQL_HOST=localhost
set MYSQL_PORT=3306
set MYSQL_USER=movies
set MYSQL_PASSWORD=password
```

我们可以使用[Ansible](https://www.ansible.com/)或[Chef](https://www.chef.io/)等配置管理工具来自动化此过程。

### 4.4 支持服务

支持服务是应用程序运行所依赖的服务，例如数据库或消息代理。**十二要素应用程序应将所有此类支持服务视为附加资源**，这实际上意味着不需要任何代码更改即可交换兼容的支持服务，唯一的更改应该是配置。

在我们的应用程序中，我们使用[MySQL](https://www.mysql.com/)作为支持服务来提供持久层。

[Spring JPA](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)使代码与实际的数据库提供商完全无关，我们只需要定义一个提供所有标准操作的Repository：

```java
@Repository
public interface MovieRepository extends JpaRepository<Movie, Long> {
}
```

我们可以看到，这并不直接依赖于MySQL。Spring 会在类路径上检测MySQL驱动程序，并动态提供此接口的MySQL特定实现。此外，它还会直接从配置中提取其他详细信息。

因此，如果我们必须从MySQL更改为Oracle，我们要做的就是替换依赖项中的驱动程序并替换配置。

### 4.5 构建、发布和运行

**十二要素方法将代码库转换为运行应用程序的过程严格分为三个不同的阶段**：

- 构建阶段：在此阶段，我们获取代码库，执行静态和动态检查，然后生成可执行文件包(如JAR)。使用[Maven](https://maven.apache.org/)之类的工具，这非常简单：

    ```shell
    mvn clean compile test package
    ```

- 发布阶段：在这个阶段，我们将可执行文件包与正确的配置结合起来。在这里，我们可以使用[Packer](https://www.packer.io/)和[Ansible](https://www.ansible.com/)等配置程序来创建Docker镜像：

    ```shell
    packer build application.json
    ```

- 运行阶段：最后，这是我们在目标执行环境中运行应用程序的阶段。如果我们使用[Docker](https://www.docker.com/)作为容器来发布我们的应用程序，那么运行应用程序就足够简单了：

    ```shell
    docker run --name <container_id> -it <image_id>
    ```

最后，我们不必手动执行这些阶段，这就是[Jenkins](https://jenkins.io/)声明式管道的用武之地。

### 4.6 进程

**十二要素应用应在执行环境中作为无状态进程运行**，换句话说，它们无法在请求之间本地存储持久状态。它们可能会生成需要存储在一个或多个有状态支持服务中的持久数据。

在我们的示例中，我们暴露了多个端点，这些端点上的请求完全独立于之前的任何请求。例如，如果我们在内存中跟踪用户请求并使用该信息来处理未来的请求，则违反了十二要素应用。

因此，十二要素应用不会施加像粘性会话这样的限制，这使得此类应用具有高度的可移植性和可扩展性。在提供自动扩展的云执行环境中，这是应用程序非常理想的行为。

### 4.7 端口绑定

传统的Java Web应用程序是作为WAR或Web存档开发的，这通常是具有依赖项的Servlet集合，并且需要符合要求的容器运行时，例如Tomcat。相反，**十二要素应用程序不需要这样的运行时依赖项**，它是完全独立的，只需要像Java这样的执行运行时。

在我们的案例中，我们使用Spring Boot开发了一个应用程序。除了许多其他优点之外，Spring Boot还为我们提供了一个默认的嵌入式应用程序服务器。因此，我们之前使用Maven生成的JAR完全能够在任何环境中执行，只要具有兼容的Java运行时即可：

```shell
java -jar application.jar
```

在这里，我们的简单应用程序通过HTTP绑定将其端点公开到特定端口(如8080)。按照上面的方式启动应用程序后，应该可以访问导出的服务(如HTTP)。

应用程序可以通过绑定多个端口来导出多种服务，如FTP或[WebSocket](https://www.baeldung.com/websockets-spring)。

### 4.8 并发

Java提供线程作为处理应用程序中并发的经典模型。线程类似于轻量级进程，代表程序中的多条执行路径。线程功能强大，但在帮助应用程序扩展方面存在局限性。

**十二要素方法建议应用程序依靠进程进行扩展**，这实际上意味着应用程序应该设计为在多个进程之间分配工作负载。但是，各个进程可以自由地在内部利用像Thread这样的并发模型。

Java应用程序在启动时会获得一个与底层JVM绑定的进程，我们实际上需要的是一种启动应用程序的多个实例并在它们之间进行智能负载分配的方法。由于我们已经将应用程序打包为[Docker](https://www.baeldung.com/docker-java-api)容器，因此[Kubernetes](https://www.baeldung.com/kubernetes)是此类编排的自然选择。

### 4.9 可处理性

**应用程序进程可能会因故意或意外事件而关闭，无论哪种情况，十二要素应用程序都应该能够妥善处理**。换句话说，应用程序进程应该是完全一次性的，不会产生任何不良副作用。此外，进程应该快速启动。

例如，在我们的应用程序中，其中一个端点是为电影创建新的数据库记录。现在，处理此类请求的应用程序可能会意外崩溃。然而，这不应该影响应用程序的状态。当客户端再次发送相同的请求时，它不应该导致重复的记录。

总之，应用程序应该公开[幂等](https://www.baeldung.com/cs/idempotent-operations)服务，这是用于云部署的服务的另一个非常理想的属性。这提供了随时停止、移动或启动新服务的灵活性，而无需考虑任何其他事项。

### 4.10 开发/生产平价

应用程序通常在本地机器上开发，在其他环境中测试，最后部署到生产环境中。通常这些环境是不同的，例如，开发团队在Windows机器上工作，而生产部署则在Linux机器上进行。

**十二要素方法建议尽可能缩小开发和生产环境之间的差距**，这些差距可能是由于较长的开发周期、涉及的团队不同或使用的技术堆栈不同造成的。

现在，Spring Boot和Docker等技术在很大程度上自动弥补了这一差距，无论我们在哪里运行容器化应用程序，它都应具有相同的行为。我们还必须使用相同的支持服务(例如数据库)。

此外，我们应该拥有正确的流程，例如持续集成和交付，以进一步缩小这一差距。

### 4.11 日志

日志是应用程序在其生命周期内生成的重要数据，它们为应用程序的运行提供了宝贵的见解。通常，应用程序可以生成多个级别的日志，具有不同的详细信息，并以多种不同的格式输出。

然而，十二要素应用程序将自己与日志生成及其处理分离开来。**对于这样的应用程序，日志只不过是按时间顺序排列的事件流**，它只是将这些事件写入执行环境的标准输出。此类流的捕获、存储、管理和归档应由执行环境处理。

有相当多的工具可用于此目的。首先，我们可以使用[SLF4J](https://www.baeldung.com/slf4j-with-log4j2-logback)在应用程序中抽象地处理日志记录。此外，我们可以使用[Fluentd](https://www.fluentd.org/)之类的工具从应用程序和支持服务收集日志流。

我们可以将其输入到[Elasticsearch](https://www.elastic.co/)中进行存储和索引。最后，我们可以在[Kibana](https://www.elastic.co/products/kibana)中生成有意义的仪表板以供可视化。

### 4.12 管理进程

我们经常需要对应用程序状态执行一些一次性任务或例行程序，例如，修复不良记录。现在，我们可以通过多种方式实现这一点。由于我们可能不经常需要它，我们可以编写一个小脚本来独立于另一个环境运行它。

现在，**十二要素方法强烈建议将此类管理脚本与应用程序代码库放在一起**。这样做时，应遵循与我们应用于主应用程序代码库相同的原则。还建议使用执行环境的内置REPL工具在生产服务器上运行此类脚本。

在我们的示例中，我们如何用迄今为止已观看的电影来填充我们的应用程序？虽然我们可以使用我们的端点，但这似乎不切实际。我们需要一个脚本来执行一次性加载，我们可以编写一个小的Java函数来从文件中读取电影列表并将它们批量保存到数据库中。

此外，我们可以使用[与Java运行时集成的Groovy](https://www.baeldung.com/groovy-java-applications)来启动此类进程。

## 5. 实际应用

现在，我们已经了解了十二要素方法论所建议的所有因素。**开发一个应用程序成为十二要素应用程序当然有其好处，特别是当我们希望将它们部署为云上的服务时**。但是，像所有其他指南、框架、模式一样，我们必须问，这是灵丹妙药吗？

老实说，软件设计和开发中没有一种方法可以声称是万灵药，十二因素方法也不例外。虽然**其中一些因素非常直观**，而且我们很可能已经在使用它们，但其他因素可能不适用于我们。我们必须根据我们的目标来评估这些因素，然后做出明智的选择。

值得注意的是，所有这些因素都**有助于我们开发模块化、独立、可移植、可扩展和可观察的应用程序**。根据应用程序的不同，我们可能能够通过其他更好的方式实现它们。也没有必要同时采用所有因素，即使只采用其中的一些因素也可能使我们比以前更好。

最后，这些因素非常简单而优雅。在我们要求应用程序具有更高的吞吐量和更低的延迟并且几乎没有停机和故障的时代，它们变得更加重要。**采用这些因素让我们从一开始就有一个正确的开端**，结合微服务架构和应用程序的容器化，它们似乎恰到好处。

## 6. 总结

在本教程中，我们介绍了十二要素方法的概念，我们讨论了如何利用Spring Boot的微服务架构来有效地交付它们。此外，我们详细探讨了每个因素以及如何将它们应用于我们的应用程序。我们还探索了几种工具，以便以有效的方式成功应用这些单个因素。