---
layout: post
title:  将Retrofit与RxJava集成
category: libraries
copyright: libraries
excerpt: Retrofit
---

## 1. 概述

本文主要介绍如何使用[Retrofit](http://square.github.io/retrofit/)实现一个简单的[支持RxJava](https://github.com/ReactiveX/RxJava)的REST客户端。

我们将使用标准Retrofit方法构建一个与GitHub API交互的示例应用程序，然后使用RxJava对其进行增强，以利用响应式编程的优势。

## 2. 简单改造

我们先用Retrofit构建一个示例，我们将使用GitHub API获取任何仓库中贡献超过100个的所有贡献者的排序列表。

### 2.1 Maven依赖

要使用Retrofit启动项目，我们需要包含以下Maven工件：

```xml
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>retrofit</artifactId>
    <version>2.3.0</version>
</dependency>

<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>converter-gson</artifactId>
    <version>2.3.0</version>
</dependency>
```

要获取最新版本，请查看Maven Central仓库中的[retrofit](https://mvnrepository.com/artifact/com.squareup.retrofit2/retrofit)和[converter-gson](https://mvnrepository.com/artifact/com.squareup.retrofit2/converter-gson)。

### 2.2 API接口

让我们创建一个简单的接口：

```java
public interface GitHubBasicApi {

    @GET("users/{user}/repos")
    Call<List> listRepos(@Path("user") String user);

    @GET("repos/{user}/{repo}/contributors")
    Call<List> listRepoContributors(
            @Path("user") String user,
            @Path("repo") String repo);
}
```

listRepos()方法检索作为路径参数传递的给定用户的仓库列表。

listRepoContributors()方法检索给定用户和仓库的贡献者列表，均作为路径参数传递。

### 2.3 逻辑

让我们使用RetrofitCall对象和普通Java代码实现所需的逻辑：

```java
class GitHubBasicService {

    private GitHubBasicApi gitHubApi;

    GitHubBasicService() {
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://api.github.com/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();

        gitHubApi = retrofit.create(GitHubBasicApi.class);
    }

    List<String> getTopContributors(String userName) throws IOException {
        List<Repository> repos = gitHubApi
                .listRepos(userName)
                .execute()
                .body();

        repos = repos != null ? repos : Collections.emptyList();

        return repos.stream()
                .flatMap(repo -> getContributors(userName, repo))
                .sorted((a, b) -> b.getContributions() - a.getContributions())
                .map(Contributor::getName)
                .distinct()
                .sorted()
                .collect(Collectors.toList());
    }

    private Stream<Contributor> getContributors(String userName, Repository repo) {
        List<Contributor> contributors = null;
        try {
            contributors = gitHubApi
                    .listRepoContributors(userName, repo.getName())
                    .execute()
                    .body();
        } catch (IOException e) {
            e.printStackTrace();
        }

        contributors = contributors != null ? contributors : Collections.emptyList();

        return contributors.stream()
                .filter(c -> c.getContributions() > 100);
    }
}
```

## 3. 与RxJava集成

Retrofit允许我们使用自定义处理程序而不是普通的Call对象来接收调用结果，方法是使用RetrofitCall适配器，这样就可以在这里使用RxJava Observables和Flowables。

### 3.1 Maven依赖

要使用RxJava适配器，我们需要包含这个Maven工件：

```xml
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>adapter-rxjava</artifactId>
    <version>2.3.0</version>
</dependency>
```

如需最新版本，请检查Maven中央仓库中的[adapter-rxjava](https://mvnrepository.com/artifact/com.squareup.retrofit2/adapter-rxjava)。

### 3.2 注册RxJava调用适配器

让我们将RxJavaCallAdapter添加到构建器中：

```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
    .build();
```

### 3.3 API接口

此时，我们可以将接口方法的返回类型更改为使用Observable<...\>而不是Call<...\>。我们可以使用其他Rx类型，例如Observable、Flowable、Single、Maybe、Completable。

让我们修改API接口以使用Observable：

```java
public interface GitHubRxApi {

    @GET("users/{user}/repos")
    Observable<List<Repository>> listRepos(@Path("user") String user);

    @GET("repos/{user}/{repo}/contributors")
    Observable<List<Contributer>> listRepoContributors(
            @Path("user") String user,
            @Path("repo") String repo);
}
```

### 3.4 逻辑

让我们使用RxJava来实现它：

```java
class GitHubRxService {

    private GitHubRxApi gitHubApi;

    GitHubRxService() {
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://api.github.com/")
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();

        gitHubApi = retrofit.create(GitHubRxApi.class);
    }

    Observable<String> getTopContributors(String userName) {
        return gitHubApi.listRepos(userName)
                .flatMapIterable(x -> x)
                .flatMap(repo -> gitHubApi.listRepoContributors(userName, repo.getName()))
                .flatMapIterable(x -> x)
                .filter(c -> c.getContributions() > 100)
                .sorted((a, b) -> b.getContributions() - a.getContributions())
                .map(Contributor::getName)
                .distinct();
    }
}
```

## 4. 总结

对比使用RxJava前后的代码，我们发现有以下几点改进：

- 响应式：由于我们的数据现在以流的形式流动，它使我们能够以非阻塞背压进行异步流处理
- 清晰：得益于声明式
- 简洁：整个操作可以表示为一个操作链