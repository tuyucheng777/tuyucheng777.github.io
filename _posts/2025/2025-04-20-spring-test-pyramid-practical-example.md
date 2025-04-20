---
layout: post
title:  测试金字塔在基于Spring的微服务中的实际应用
category: springboot
copyright: springboot
excerpt: 测试金字塔
---

## 1. 概述

在本教程中，我们将介绍流行的软件测试模型，称为测试金字塔。

我们将了解它在微服务领域中的具体应用，在此过程中，我们将开发一个示例应用程序并进行相关测试以符合此模型。此外，我们将尝试理解使用模型的优势和局限性。

## 2. 初衷

在我们开始了解任何特定模型(例如测试金字塔)之前，必须了解为什么我们需要一个模型。

软件测试的需求与生俱来，或许与软件开发的历史一样悠久。软件测试经历了漫长的发展历程，从手动到自动化，甚至更进一步。然而，目标始终如一-**交付符合规范的软件**。

### 2.1 测试类型

实践中存在多种不同类型的测试，每个测试都侧重于特定的目标。遗憾的是，人们对这些测试的词汇甚至理解都存在很大差异。

让我们回顾一下一些流行的、可能明确的：

- **单元测试**：单元测试是**针对小单元代码的测试，最好是独立的**。其目标是验证最小可测试代码段的行为，而无需考虑代码库的其余部分。这自然意味着任何依赖项都需要用Mock、存根或类似的结构替换。
- **集成测试**：虽然单元测试侧重于代码片段的内部，但事实上，很多复杂性都存在于代码之外。代码单元需要协同工作，并且通常需要与外部服务(例如数据库、消息代理或Web服务)协作，**集成测试是针对应用程序与外部依赖项集成时的行为进行的测试**。
- **UI测试**：我们开发的软件通常通过界面来使用，用户可以与界面交互，应用程序通常具有Web界面。然而，API接口正变得越来越流行。**UI测试针对这些界面的行为，这些界面通常本质上是高度交互的**。现在，这些测试可以以端到端的方式进行，也可以单独测试用户界面。

### 2.2 手动测试与自动测试

自软件测试诞生以来，手动测试一直是软件测试的主流，即使在今天，手动测试仍然被广泛应用。然而，手动测试的局限性并不难理解，**为了使测试有效，测试必须全面且频繁地运行**。

在敏捷开发方法和云原生微服务架构中，这一点尤为重要。但是，测试自动化的需求早已被人们意识到。

回想一下我们之前讨论过的不同类型的测试，你会发现，随着从单元测试到集成测试和UI测试的转变，它们的复杂性和范围都会增加。出于同样的原因，**单元测试的自动化更容易实现**，并且也带来了很多好处。但随着我们进一步深入，自动化测试变得越来越困难，而且好处也越来越少。

除某些方面外，目前大多数软件行为的自动化测试都是可能的。然而，必须合理权衡自动化带来的收益与所需投入的努力。

## 3. 什么是测试金字塔？

现在我们已经对测试类型和工具有了足够的了解，是时候了解测试金字塔到底是什么了。

我们已经了解了应该编写不同类型的测试，但是，我们应该如何决定每种类型应该编写多少个测试？有哪些好处或陷阱需要注意？这些都是测试金字塔等测试自动化模型所要解决的一些问题。

[Mike Cohn](https://www.mountaingoatsoftware.com/blog)在他的著作[《敏捷成功》](https://www.pearson.com/us/higher-education/program/Cohn-Succeeding-with-Agile-Software-Development-Using-Scrum/PGM201415.html)中提出了一种名为“测试金字塔”的结构，**它以可视化的方式展示了我们应该在不同粒度级别编写的测试数量**。

其理念是，在最精细的层面上，它应该达到最高，随着测试范围的扩大，它应该开始下降。这呈现出典型的金字塔形状，因此得名：

![](/assets/images/2025/springboot/springtestpyramidpracticalexample01.png)

虽然这个概念非常简单优雅，但有效地运用它往往是一个挑战。重要的是要理解，我们不能拘泥于模型的形状和它提到的测试类型，关键点在于：

- 我们必须编写具有不同粒度级别的测试
- 随着测试范围变得越来越粗略，我们必须编写更少的测试

## 4. 测试自动化工具

所有主流编程语言中都提供了多种工具来编写不同类型的测试，我们将介绍Java世界中一些常用的工具。

### 4.1 单元测试

- 测试框架：Java中最流行的选择是[JUnit](https://www.baeldung.com/junit)，它的下一代版本是[JUnit5](https://www.baeldung.com/junit-5)。该领域的其他热门选择包括[TestNG](https://www.baeldung.com/testng)，它与JUnit5相比提供了一些差异化的功能。不过，对于大多数应用程序来说，这两个都是合适的选择。
- [Mock](https://www.baeldung.com/mockito-series)：正如我们之前所见，在执行单元测试时，我们肯定希望扣除大部分(如果不是全部)依赖项。为此，我们需要一种机制，用Mock或存根之类的测试替身来替换依赖项。Mockito是一个优秀的框架，可以为Java中的真实对象提供Mock。

### 4.2 集成测试

- 测试框架：集成测试的范围比单元测试更广，但入口点通常是相同的代码，只是抽象程度更高。因此，适用于单元测试的测试框架也适用于集成测试。
- Mock：集成测试的目标是通过真实的集成测试应用程序的行为，然而，我们可能不想使用真实的数据库或消息代理进行测试，许多数据库和类似的服务都提供了[可嵌入的版本](https://www.baeldung.com/java-in-memory-databases)来编写集成测试。

### 4.3 UI测试

- 测试框架：UI测试的复杂性取决于处理软件UI元素的客户端，例如，网页的行为可能因设备、浏览器甚至操作系统而异。Selenium是使用Web应用程序模拟浏览器行为的常用选择，然而，对于[REST API](https://www.baeldung.com/java-selenium-with-junit-and-testng)，像[REST-Assured](https://www.baeldung.com/rest-assured-tutorial)这样的框架是更好的选择。
- [Mock](https://angular.io/)和[React](https://reactjs.org/)：借助Angular等JavaScript框架，用户界面的交互性越来越强，并且更倾向于客户端渲染。使用[Jasmine](https://jasmine.github.io/)和[Mocha](https://mochajs.org/)等测试框架单独测试这些UI元素更为合理。显然，我们应该结合端到端测试来做到这一点。

## 5. 在实践中采用原则

让我们开发一个小应用程序来演示我们目前讨论过的原则，我们将开发一个小型微服务，并了解如何编写符合测试金字塔的测试。

**[微服务架构](https://www.baeldung.com/spring-microservices-guide)有助于将应用程序构建为围绕领域边界的松散耦合服务的集合**，[Spring Boot](https://www.baeldung.com/spring-boot)提供了一个优秀的平台，可以快速启动一个包含用户界面和数据库等依赖的微服务。

我们将利用这些来展示测试金字塔的实际应用。

### 5.1 应用程序架构

我们将开发一个基本应用程序，允许存储和查询我们看过的电影：

![](/assets/images/2025/springboot/springtestpyramidpracticalexample02.png)

我们可以看到，它有一个简单的REST控制器，公开3个端点：

```java
@RestController
public class MovieController {

    @Autowired
    private MovieService movieService;

    @GetMapping("/movies")
    public List<Movie> retrieveAllMovies() {
        return movieService.retrieveAllMovies();
    }

    @GetMapping("/movies/{id}")
    public Movie retrieveMovies(@PathVariable Long id) {
        return movieService.retrieveMovies(id);
    }

    @PostMapping("/movies")
    public Long createMovie(@RequestBody Movie movie) {
        return movieService.createMovie(movie);
    }
}
```

除了处理数据编组和解组之外，控制器仅路由到适当的服务：

```java
@Service
public class MovieService {

    @Autowired
    private MovieRepository movieRepository;

    public List<Movie> retrieveAllMovies() {
        return movieRepository.findAll();
    }

    public Movie retrieveMovies(@PathVariable Long id) {
        Movie movie = movieRepository.findById(id)
                .get();
        Movie response = new Movie();
        response.setTitle(movie.getTitle()
                .toLowerCase());
        return response;
    }

    public Long createMovie(@RequestBody Movie movie) {
        return movieRepository.save(movie)
                .getId();
    }
}
```

此外，我们有一个映射到持久层的JPA Repository：

```java
@Repository
public interface MovieRepository extends JpaRepository<Movie, Long> {
}
```

最后，我们的简单域实体用于保存和传递电影数据：

```java
@Entity
public class Movie {
    @Id
    private Long id;
    private String title;
    private String year;
    private String rating;

    // Standard setters and getters
}
```

有了这个简单的应用程序，我们现在就可以探索不同粒度和数量的测试了。

### 5.2 单元测试

首先，我们将了解如何为我们的应用程序编写一个简单的单元测试。从这个应用程序中可以明显看出，**大多数逻辑都集中在服务层**。这要求我们对其进行更广泛、更频繁的测试-非常适合单元测试：

```java
public class MovieServiceUnitTests {

    @InjectMocks
    private MovieService movieService;

    @Mock
    private MovieRepository movieRepository;

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void givenMovieServiceWhenQueriedWithAnIdThenGetExpectedMovie() {
        Movie movie = new Movie(100L, "Hello World!");
        Mockito.when(movieRepository.findById(100L))
                .thenReturn(Optional.ofNullable(movie));

        Movie result = movieService.retrieveMovies(100L);

        Assert.assertEquals(movie.getTitle().toLowerCase(), result.getTitle());
    }
}
```

这里，我们使用JUnit作为测试框架，并使用Mockito来Mock依赖项。我们的服务出于一些奇怪的需求，需要返回小写的电影名称，而这正是我们打算在这里测试的。这类行为有很多，我们应该用这样的单元测试来全面覆盖。

### 5.3 集成测试

在我们的单元测试中，我们Mock了Repository，这是我们对持久层的依赖。虽然我们已经彻底测试了服务层的行为，但在它连接到数据库时仍然可能遇到问题。这时，集成测试就派上用场了：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MovieControllerIntegrationTests {

    @Autowired
    private MovieController movieController;

    @Test
    public void givenMovieControllerWhenQueriedWithAnIdThenGetExpectedMovie() {
        Movie movie = new Movie(100L, "Hello World!");
        movieController.createMovie(movie);

        Movie result = movieController.retrieveMovies(100L);

        Assert.assertEquals(movie.getTitle().toLowerCase(), result.getTitle());
    }
}
```

注意这里一些有趣的区别，现在，我们没有Mock任何依赖项。但是，**根据情况，我们可能仍然需要Mock一些依赖项**。此外，我们使用SpringRunner运行这些测试。

这实际上意味着我们将拥有一个Spring应用程序上下文和实时数据库来运行此测试，导致运行速度会变慢，因此，我们在这里尽量选择较少的场景进行测试。

### 5.4 UI测试

最后，我们的应用程序需要使用REST端点，这些端点可能有一些细微的差别需要测试。由于这是我们应用程序的用户界面，因此我们将重点关注它，以进行UI测试。现在让我们使用REST-Assured来测试应用程序：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class MovieApplicationE2eTests {

    @Autowired
    private MovieController movieController;

    @LocalServerPort
    private int port;

    @Test
    public void givenMovieApplicationWhenQueriedWithAnIdThenGetExpectedMovie() {
        Movie movie = new Movie(100L, "Hello World!");
        movieController.createMovie(movie);

        when().get(String.format("http://localhost:%s/movies/100", port))
                .then()
                .statusCode(is(200))
                .body(containsString("Hello World!".toLowerCase()));
    }
}
```

如我们所见，**这些测试是在正在运行的应用程序上运行的，并通过可用的端点访问它**。我们专注于测试与HTTP相关的典型场景，例如响应码。由于显而易见的原因，这些测试的运行速度最慢。

因此，我们必须非常谨慎地选择测试场景，我们应该只关注那些在之前的更细致的测试中未能涵盖的复杂情况。

## 6. 微服务测试金字塔

现在我们已经了解了如何编写不同粒度的测试并合理地构建它们，但是，关键目标是通过更精细、更快速的测试来捕捉大部分应用程序的复杂性。

**虽然在[单体应用程序](https://www.baeldung.com/cs/microservices-vs-monolithic-architectures)中解决这个问题可以给我们所需的金字塔结构，但对于其他架构来说这可能不是必需的**。

众所周知，微服务架构将一个应用程序拆分成一组松散耦合的应用程序。这样一来，它就将应用程序固有的一些复杂性外部化了。

现在，这些复杂性体现在服务之间的通信中。单元测试并不总是能够捕捉到它们，我们必须编写更多的集成测试。

虽然这可能意味着我们偏离了经典的金字塔模型，但这并不意味着我们也偏离了原则。记住，**我们仍然在通过尽可能精细的测试来捕捉大部分的复杂性**。只要我们清楚这一点，一个可能与完美金字塔不符的模型仍然有价值。

这里需要理解的重要一点是，模型只有在能够提供价值时才有用。通常，价值取决于具体环境，在本例中，具体环境就是我们为应用程序选择的架构。因此，虽然使用模型作为指导很有帮助，但我们应该关注其基本原则，并最终选择在我们的架构环境中有意义的模型。

## 7. 与CI集成

当我们将自动化测试集成到持续集成流水线中时，其强大功能和优势才能得到充分体现，[Jenkins](https://jenkins.io/)是声明式定义构建和[部署流水线](https://www.baeldung.com/jenkins-pipelines)的热门选择。

**我们可以集成任何已在Jenkins流水线中自动化的测试**，但是，我们必须明白，这会增加流水线的执行时间。持续集成的主要目标之一是快速反馈，如果我们开始添加导致流水线运行速度变慢的测试，这可能会产生冲突。

关键在于，**在预期运行频率更高的管道中添加快速测试，例如单元测试**。例如，在每次提交时触发的管道中添加UI测试，可能并不会给我们带来好处。但这只是一个指导原则，最终还是取决于我们处理的应用程序的类型和复杂性。

## 8. 总结

在本文中，我们了解了软件测试的基础知识，我们了解了不同的测试类型以及使用可用工具之一实现自动化测试的重要性。

此外，我们理解了测试金字塔的含义，我们使用基于Spring Boot构建的微服务实现了它。

最后，我们讨论了测试金字塔的相关性，特别是在微服务等架构的背景下。