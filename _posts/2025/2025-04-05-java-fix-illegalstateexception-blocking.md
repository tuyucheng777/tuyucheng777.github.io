---
layout: post
title:  如何解决“java.lang.IllegalStateException:block()/blockFirst()/blockLast() are blocking”
category: springreactive
copyright: springreactive
excerpt: Spring Reactive
---

## 1. 简介

在本文中，我们将看到开发人员在使用[Spring Webflux](https://www.baeldung.com/spring-webflux)时常犯的一个错误。**Spring Webflux是一个从头开始构建的非阻塞Web框架**，旨在利用多核、下一代处理器并处理大量并发连接。

由于它是一个非阻塞框架，线程不应该被阻塞，让我们更详细地探讨这一点。

## 2. Spring Webflux线程模型

为了更好地理解这个问题，我们需要了解[Spring Webflux的线程模型](https://www.baeldung.com/spring-webflux-concurrency)。

在Spring Webflux中，一小批工作线程处理传入的请求。这与Servlet模型形成对比，在Servlet模型中，每个请求都有一个专用线程。因此，框架可以保护这些请求接收线程上发生的事情。

基于这种理解，让我们深入探讨本文的重点。

## 3. 理解线程阻塞的IllegalStateException

让我们借助一个例子来了解在Spring Webflux中何时以及为何会出现错误“java.lang.IllegalStateException: block()/blockFirst()/blockLast() are blocking, which is not supported in thread”。

让我们以文件搜索API为例，此API从文件系统读取文件并在文件中搜索用户提供的文本。

### 3.1 文件服务

让我们首先定义一个FileService类，将文件的内容读取为字符串：

```java
@Service
public class FileService {
    @Value("${files.base.dir:/tmp/bael-7724}")
    private String filesBaseDir;

    public Mono<String> getFileContentAsString(String fileName) {
        return DataBufferUtils.read(Paths.get(filesBaseDir + "/" + fileName), DefaultDataBufferFactory.sharedInstance, DefaultDataBufferFactory.DEFAULT_INITIAL_CAPACITY)
                .map(dataBuffer -> dataBuffer.toString(StandardCharsets.UTF_8))
                .reduceWith(StringBuilder::new, StringBuilder::append)
                .map(StringBuilder::toString);
    }
}
```

值得注意的是，FileService以响应式方式(异步方式)从文件系统读取文件。

### 3.2 文件内容搜索服务

我们准备利用这个FileService来编写文件搜索服务：

```java
@Service
public class FileContentSearchService {
    @Autowired
    private FileService fileService;

    public Mono<Boolean> blockingSearch(String fileName, String searchTerm) {
        String fileContent = fileService
                .getFileContentAsString(fileName)
                .doOnNext(content -> ThreadLogger.log("1. BlockingSearch"))
                .block();

        boolean isSearchTermPresent = fileContent.contains(searchTerm);

        return Mono.just(isSearchTermPresent);
    }
}
```

文件搜索服务根据文件中是否找到搜索词返回一个布尔值。为此，我们调用FileService的getFileContentAsString()方法。由于我们异步获取结果(即作为Mono<String\>)，我们调用block()来获取字符串值。之后，我们检查fileContent是否包含searchTerm。最后，我们将结果包装并返回到[Mono](https://www.baeldung.com/java-reactor-flux-vs-mono)中。

### 3.3 文件控制器

最后，我们编写使用FileContentSearchService的blockingSearch()方法的FileController：

```java
@RestController
@RequestMapping("bael7724/v1/files")
public class FileController {
    // ...
    @GetMapping(value = "/{name}/blocking-search")
    Mono<Boolean> blockingSearch(@PathVariable("name") String fileName, @RequestParam String term) {
        return fileContentSearchService.blockingSearch(fileName, term);
    }
}
```

### 3.4 重现异常

我们可以观察到Controller调用了FileContentSearchService的方法，而后者又调用了block()方法。由于这是在请求接受线程上进行的，如果我们按照当前安排调用API，就会遇到我们想要的臭名昭著的异常：

```text
12:28:51.610 [reactor-http-epoll-2] ERROR o.s.b.a.w.r.e.AbstractErrorWebExceptionHandler - [ea98e542-1]  500 Server Error for HTTP GET "/bael7724/v1/files/a/blocking-search?term=a"
java.lang.IllegalStateException: block()/blockFirst()/blockLast() are blocking, which is not supported in thread reactor-http-epoll-2
    at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:86)
    Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Error has been observed at the following site(s):
    *__checkpoint ⇢ cn.tuyucheng.taketoday.filters.TraceWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ cn.tuyucheng.taketoday.filters.ExceptionalTraceFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ HTTP GET "/bael7724/v1/files/a/blocking-search?term=a" [ExceptionHandlingWebHandler]
Original Stack Trace:
	at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:86)
	at reactor.core.publisher.Mono.block(Mono.java:1712)
	at cn.tuyucheng.taketoday.bael7724.service.FileContentSearchService.blockingSearch(FileContentSearchService.java:20)
	at cn.tuyucheng.taketoday.bael7724.controller.FileController.blockingSearch(FileController.java:35)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
```

### 3.5 根本原因

此异常的根本原因是在接受请求的线程上调用block()，在上面的示例代码中，**block()方法在接受请求的线程池中的一个线程上被调用**。具体来说，是在标记为“仅非阻塞操作”的线程上，即实现Reactor的[NonBlocking](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/NonBlocking.html)标记接口的线程，例如由Schedulers.parallel()启动的线程。

## 4. 解决方案

现在让我们看看如何解决这个异常。

### 4.1 拥抱响应式

惯用的方法是使用响应式操作而不是调用block()，让我们更新代码以利用map()操作将String转换为Boolean：

```java
public Mono<Boolean> nonBlockingSearch(String fileName, String searchTerm) {
    return fileService.getFileContentAsString(fileName)
        .doOnNext(content -> ThreadLogger.log("1. NonBlockingSearch"))
        .map(content -> content.contains(searchTerm))
        .doOnNext(content -> ThreadLogger.log("2. NonBlockingSearch"));
}
```

这样我们就完全不需要调用block()了。运行上述方法时，我们注意到以下线程上下文：

```text
[1. NonBlockingSearch] ThreadName: Thread-4, Time: 2024-06-17T07:40:59.506215299Z
[2. NonBlockingSearch] ThreadName: Thread-4, Time: 2024-06-17T07:40:59.506361786Z
[1. In Controller] ThreadName: Thread-4, Time: 2024-06-17T07:40:59.506465805Z
[2. In Controller] ThreadName: Thread-4, Time: 2024-06-17T07:40:59.506543145Z
```

以上日志语句表明我们在接受请求的同一个线程池上执行了操作。

值得注意的是，**即使我们没有遇到异常，最好在不同的线程池上运行I/O操作(例如从文件读取)**。

### 4.2 有界弹性线程池上的阻塞

假设由于某种原因我们无法避免block()，那么我们该怎么做呢？我们得出结论，异常发生在我们在接受请求的线程池上调用block()时。因此，要调用block()，我们需要切换线程池，让我们看看如何做到这一点：

```java
public Mono<Boolean> workableBlockingSearch(String fileName, String searchTerm) {
    return Mono.just("")
            .doOnNext(s -> ThreadLogger.log("1. WorkableBlockingSearch"))
            .publishOn(Schedulers.boundedElastic())
            .doOnNext(s -> ThreadLogger.log("2. WorkableBlockingSearch"))
            .map(s -> fileService.getFileContentAsString(fileName)
                    .block()
                    .contains(searchTerm))
            .doOnNext(s -> ThreadLogger.log("3. WorkableBlockingSearch"));
}
```

为了切换线程池，Spring Webflux提供了两个运算符publishOn()和subscribeOn()。我们使用了publishOn()，它可以更改publishOn()之后的操作的线程，而不会影响订阅或上游操作。由于线程池现在已切换到boundedElastic，我们可以调用block()。

现在，如果我们运行workableBlockingSearch()方法，我们将获得以下线程上下文：

```text
[1. WorkableBlockingSearch] ThreadName: parallel-2, Time: 2024-06-17T07:40:59.440562518Z
[2. WorkableBlockingSearch] ThreadName: boundedElastic-1, Time: 2024-06-17T07:40:59.442161018Z
[3. WorkableBlockingSearch] ThreadName: boundedElastic-1, Time: 2024-06-17T07:40:59.442891230Z
[1. In Controller] ThreadName: boundedElastic-1, Time: 2024-06-17T07:40:59.443058091Z
[2. In Controller] ThreadName: boundedElastic-1, Time: 2024-06-17T07:40:59.443181770Z
```

我们可以看到，从数字2开始，操作确实发生在boundedElastic线程池上，因此我们没有得到IllegalStateException。

### 4.3 注意事项

让我们看一下使用该块的解决方案的一些注意事项。

调用block()时有很多可能出错的地方，让我们举一个例子，即使我们使用Scheduler来切换线程上下文，它也不会按照我们期望的方式运行：

```java
public Mono<Boolean> incorrectUseOfSchedulersSearch(String fileName, String searchTerm) {
    String fileContent = fileService.getFileContentAsString(fileName)
        .doOnNext(content -> ThreadLogger.log("1. IncorrectUseOfSchedulersSearch"))
        .publishOn(Schedulers.boundedElastic())
        .doOnNext(content -> ThreadLogger.log("2. IncorrectUseOfSchedulersSearch"))
        .block();

    boolean isSearchTermPresent = fileContent.contains(searchTerm);

    return Mono.just(isSearchTermPresent);
}
```

在上面的代码示例中，我们按照解决方案中的建议使用了publishOn()，但block()方法仍然导致异常。运行上述代码时，我们会收到以下日志：

```text
[1. IncorrectUseOfSchedulersSearch] ThreadName: Thread-4, Time: 2024-06-17T08:57:02.490298417Z
[2. IncorrectUseOfSchedulersSearch] ThreadName: boundedElastic-1, Time: 2024-06-17T08:57:02.491870410Z
14:27:02.495 [parallel-1] ERROR o.s.b.a.w.r.e.AbstractErrorWebExceptionHandler - [53e4bce1]  500 Server Error for HTTP GET "/bael7724/v1/files/robots.txt/incorrect-use-of-schedulers-search?term=r-"
java.lang.IllegalStateException: block()/blockFirst()/blockLast() are blocking, which is not supported in thread parallel-1
    at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:86)
    Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Error has been observed at the following site(s):
    *__checkpoint ⇢ cn.tuyucheng.taketoday.filters.TraceWebFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ cn.tuyucheng.taketoday.filters.ExceptionalTraceFilter [DefaultWebFilterChain]
    *__checkpoint ⇢ HTTP GET "/bael7724/v1/files/robots.txt/incorrect-use-of-schedulers-search?term=r-" [ExceptionHandlingWebHandler]
Original Stack Trace:
	at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:86)
	at reactor.core.publisher.Mono.block(Mono.java:1712)
	at cn.tuyucheng.taketoday.bael7724.service.FileContentSearchService.incorrectUseOfSchedulersSearch(FileContentSearchService.java:64)
	at cn.tuyucheng.taketoday.bael7724.controller.FileController.incorrectUseOfSchedulersSearch(FileController.java:48)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
```

这表明第二条日志语句确实在boundedElastic线程池上运行。但是，我们仍然遇到了异常。原因是block()仍在同一个接受请求的线程池上运行。

让我们看看另一个注意事项。即使我们正在切换线程池，我们也不能使用并行线程池，即Schedulers.parallel()。如前所述，某些线程池不允许在其线程上调用block()-并行线程池就是其中之一。

最后，我们在示例中仅使用了Schedulers.boundedElastic()。相反，我们也可以通过Schedulers.fromExecutorService()使用任何自定义线程池。

## 5. 总结

总之，为了有效解决Spring Webflux中使用block()等阻塞操作时出现IllegalStateException的问题，我们应该采用非阻塞、响应式方法。通过利用map()等响应式运算符，我们可以在同一个响应式线程池上执行操作，从而无需显式block()。如果无法避免block()，则使用publishOn()将执行上下文切换到boundedElastic调度程序或自定义线程池可以将这些操作与响应式请求接受线程池隔离，从而防止出现异常。

必须了解不支持阻塞调用的线程池，并确保应用正确的上下文切换以维持应用程序的稳定性和性能。