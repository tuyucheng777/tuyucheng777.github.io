---
layout: post
title:  Ratpack与RxJava
category: webmodules
copyright: webmodules
excerpt: Ratpack
---

## 1. 简介

[RxJava](https://www.baeldung.com/rxjava-tutorial)是最流行的响应式编程库之一。

[Ratpack](https://www.baeldung.com/ratpack)是一组Java库，用于创建基于Netty的精简且强大的Web应用程序。

在本教程中，我们将讨论在Ratpack应用程序中结合RxJava来创建一个良好的响应式Web应用程序。

## 2. Maven依赖

现在，我们首先需要[ratpack-core](https://mvnrepository.com/artifact/io.ratpack/ratpack-core)和[ratpack-rx](https://mvnrepository.com/artifact/io.ratpack/ratpack-rx)依赖：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-rx</artifactId>
    <version>1.6.0</version>
</dependency>
```

顺便说一句，请注意，ratpack-rx为我们导入了rxjava依赖。

## 3. 初始设置

RxJava使用其插件系统支持集成第三方库，**因此，我们可以将不同的执行策略合并到RxJava的执行模型中**。 

Ratpack通过RxRatpack插入此执行模型，我们在启动时对其进行初始化：

```java
RxRatpack.initialise();
```

现在，需要注意的是，**该方法每次JVM运行只需要调用一次**。

结果是我们**能够将RxJava的Observable映射到RxRatpack的Promise类型**，反之亦然。

## 4. Observable到Promise

我们可以将RxJava中的Observable转换为Ratpack Promise。

**但是，这其中还是有一点不匹配的**。即Promise发出一个单一的值，而Observable可以发出一个值流。

RxRatpack通过提供两种不同的方法来处理这个问题：promiseSingle()和promise()。

因此，假设我们有一个名为MovieService的服务，它在getMovie()上发出一个Promise。我们将使用promiseSingle()，因为我们知道它只会发出一次：

```java
Handler movieHandler = (ctx) -> {
    MovieService movieSvc = ctx.get(MovieService.class);
    Observable<Movie> movieObs = movieSvc.getMovie();
    RxRatpack.promiseSingle(movieObs)
        .then(movie -> ctx.render(Jackson.json(movie)));
};
```

另一方面，如果getMovies()可以返回电影结果流，我们将使用promise()：

```java
Handler moviesHandler = (ctx) -> {
    MovieService movieSvc = ctx.get(MovieService.class);
    Observable<Movie> movieObs = movieSvc.getMovies();
    RxRatpack.promise(movieObs)
        .then(movie -> ctx.render(Jackson.json(movie)));
};
```

然后，我们可以像平常一样将这些处理程序添加到我们的Ratpack服务器：

```java
RatpackServer.start(def -> def.registryOf(rSpec -> rSpec.add(MovieService.class, new MovieServiceImpl()))
    .handlers(chain -> chain
        .get("movie", movieHandler)
        .get("movies", moviesHandler)));
```

## 5. Promise到Observable

相反，我们可以将Ratpack中的Promise类型映射回RxJava Observable。 

**RxRatpack再次有两种方法：observe()和observerEach()**。

在这种情况下，想象我们有一个电影服务，它返回Promise而不是Observable。

通过getMovie()，我们将使用observe()：

```java
Handler moviePromiseHandler = ctx -> {
    MoviePromiseService promiseSvc = ctx.get(MoviePromiseService.class);
    Promise<Movie> moviePromise = promiseSvc.getMovie();
    RxRatpack.observe(moviePromise)
        .subscribe(movie -> ctx.render(Jackson.json(movie)));
};
```

当我们得到一个列表时，比如使用getMovies()，我们会使用observerEach()：

```java
Handler moviesPromiseHandler = ctx -> {
    MoviePromiseService promiseSvc = ctx.get(MoviePromiseService.class);
    Promise<List<Movie>> moviePromises = promiseSvc.getMovies();
    RxRatpack.observeEach(moviePromises)
        .toList()
        .subscribe(movie -> ctx.render(Jackson.json(movie)));
};
```

然后，我们可以再次按预期添加处理程序：

```java
RatpackServer.start(def -> def.registryOf(regSpec -> regSpec
    .add(MoviePromiseService.class, new MoviePromiseServiceImpl()))
        .handlers(chain -> chain
            .get("movie", moviePromiseHandler)
            .get("movies", moviesPromiseHandler)));
```

## 6. 并行处理

**RxRatpack借助fork()和forkEach()方法支持并行性**。

它遵循我们已经看到的模式。

fork()接收单个Observable并**将其执行并行到不同的计算线程上**，然后，它会自动将数据绑定回原始执行。

另一方面，forkEach()对Observable的值流发出的每个元素执行相同的操作。

假设我们想将电影标题大写，而这是一项昂贵的操作。

简而言之，我们可以使用forkEach()将每个任务的执行卸载到线程池上：

```java
Observable<Movie> movieObs = movieSvc.getMovies();
Observable<String> upperCasedNames = movieObs.compose(RxRatpack::forkEach)
    .map(movie -> movie.getName().toUpperCase())
    .serialize();
```

## 7. 隐式错误处理

最后，隐式错误处理是RxJava集成中的关键功能之一。

默认情况下，RxJava Observable序列会将任何异常转发给执行上下文异常处理程序。因此，无需在Observable序列中定义错误处理程序。

因此，**我们可以配置Ratpack来处理RxJava引发的这些错误**。

例如，假设我们希望在HTTP响应中打印每个错误。

请注意，我们通过Observable抛出的异常会被我们的ServerErrorHandler捕获并处理：

```java
RatpackServer.start(def -> def.registryOf(regSpec -> regSpec
    .add(ServerErrorHandler.class, (ctx, throwable) -> {
            ctx.render("Error caught by handler : " + throwable.getMessage());
        }))
    .handlers(chain -> chain
        .get("error", ctx -> {
            Observable.<String> error(new Exception("Error from observable")).subscribe(s -> {});
        })));
```

**但请注意，任何订阅者级别的错误处理都优先**。如果我们的Observable想要自己处理错误，那当然可以，但由于它没有这样做，所以异常会渗透到Ratpack。

## 8. 总结

在本文中，我们讨论了如何使用Ratpack配置RxJava。

我们探索了RxJava中的Observables到Ratpack中的Promise类型的转换，反之亦然；我们还研究了集成支持的并行性和隐式错误处理功能。