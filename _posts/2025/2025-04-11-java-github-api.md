---
layout: post
title:  GitHub Java API
category: libraries
copyright: libraries
excerpt: GitHub Java API
---

## 1. 简介

在本文中，我们将了解[GitHub API Java](https://hub4j.github.io/github-api/)库，它为我们提供了[GitHib API](https://docs.github.com/en/rest?apiVersion=2022-11-28)的面向对象表示，使我们能够轻松地从Java应用程序与其进行交互。

## 2. 依赖

**要使用GitHub API Java库，我们需要在构建中包含[最新版本](https://mvnrepository.com/artifact/org.kohsuke/github-api)，目前为[1.327](https://mvnrepository.com/artifact/org.kohsuke/github-api/1.327)**。

如果我们使用Maven，我们可以在pom.xml文件中包含这个依赖：

```xml
<dependency>
    <groupId>org.kohsuke</groupId>
    <artifactId>github-api</artifactId>
    <version>1.327</version>
</dependency>
```

## 3. 客户端创建

**为了使用该库，我们首先需要创建一个GitHub客户端实例，这是与GitHub API交互的主要入口点**。

创建此实例的最简单方法是匿名连接：

```java
GitHub gitHub = GitHub.connectAnonymously();
```

这样我们就可以在没有任何身份验证凭据的情况下访问API，但是，我们能用这种方式实现的功能仅限于那些支持这种方式的功能。

或者，我们可以使用凭证进行连接：

```java
GitHub gitHub = GitHub.connect();
```

这样做会尝试通过几种不同的方式确定要使用的凭据：

- 如果设置了环境属性GITHUB_OAUTH，那么这将用作个人访问令牌。
- 否则，如果设置了环境属性GITHUB_JWT，这将用作JWT令牌。
- 否则，如果同时设置了GITHUB_LOGIN和GITHUB_PASSWORD，那么它们将直接用作凭据。
- 否则，我们将尝试使用用户主目录中的.github属性文件来提供等效属性，在这种情况下，名称将为小写，并且不带GITHUB_前缀。

如果这些值都不可用，则创建客户端将失败。

我们还可以选择通过手动提供凭证来创建客户端：

```java
GitHub github = new GitHubBuilder().withOAuthToken("my_personal_token").build();
GitHub github = new GitHubBuilder().withJwtToken("my_jwt_token").build();
GitHub github = new GitHubBuilder().withPassword("my_user", "my_password").build();
```

如果我们选择使用其中一个，那么我们就可以从任何我们希望的地方加载适当的凭证。

另请注意，该库仅在需要时使用这些凭据。这意味着，如果提供的凭据无效，我们只会在以需要身份验证的方式与API交互时发现。如果我们使用任何以匿名方式工作的API方法，那么这些方法将继续有效。

## 4. 我和其他用户

一旦我们创建了客户端，我们就可以开始与GitHub API交互。

**如果我们有一个正确验证的客户端，那么我们可以使用它来查询我们已验证的用户-即我自己**：

```java
GHMyself myself = gitHub.getMyself();
```

然后我们可以使用该对象来查询当前用户：

```java
assertEquals("someone", myself.getLogin());
assertEquals("someone@example.com", myself.getEmail());
assertEquals(50, myself.getFollows().size());
```

**我们还可以查询其他用户的详细信息**：

```java
GHUser user = gitHub.getUser("tuyucheng");
assertEquals("tuyucheng", user.getLogin());
assertEquals(2, user.getFollows().size());
```

这将返回一个GHUser，它是GHMyself的超类。因此，我们可以对GHMyself对象(代表当前经过身份验证的用户)执行某些操作，而不能对任何其他用户执行这些操作。这些包括：

- 管理公钥
- 管理电子邮件地址
- 管理组织会员资格

但是，任何需要GHUser的东西也可以接收GHMyself。

## 5. 仓库

我们还可以与仓库以及用户合作，这将包括查询用户拥有的仓库列表的能力，以及访问仓库的内容甚至对其进行更改。

请注意，GitHub中的所有仓库都只属于一个用户，因此我们需要知道用户名以及仓库名称才能正确访问它们。

### 5.1 列出仓库

**如果我们有一个GHUser对象(或GHMyself)，那么我们可以使用它来获取此用户拥有的仓库，这可以通过使用listRepositories()实现**：

```java
PagedIterable<GHRepository> repositories = user.listRepositories();
```

由于这些仓库的数量可能非常多，因此这为我们提供了一个PagedIterable。默认情况下，页面大小为30，但如果需要，我们可以在列出仓库时指定这一点：

```java
PagedIterable<GHRepository> repositories = user.listRepositories(50);
```

这为我们提供了多种访问实际仓库的方法，我们可以将其转换为包含整个仓库集合的Array、List或Set类型：

```java
Array<GHRepository> repositoriesArray = repositories.toArray();
List<GHRepository> repositoriesList = repositories.toList();
Set<GHRepository> repositoriesSet = repositories.toSet();
```

但是，这些操作会预先获取所有内容，如果数量非常大，则成本会很高。例如，[Microsoft用户](https://github.com/orgs/microsoft/repositories)目前有6688个仓库，按每页30个仓库计算，需要223次API调用才能收集完整列表。

或者，我们可以获取仓库上的迭代器，这样，它只会根据需要进行API调用，从而让我们更高效地访问集合：

```java
Iterator<GHRepository> repositoriesSet = repositories.toIterator();
```

更简单的是，PagedIterable本身是一个Iterable，可以在任何适用的地方直接使用-例如，在[增强for循环](https://www.baeldung.com/java-for-each-loop#for-each-loop)中：

```java
Set<String> names = new HashSet<>();
for (GHRepository ghRepository : user.listRepositories()) {
    names.add(ghRepository.getName());
}
```

这只是迭代每个返回的仓库并提取它们的名称，因为我们使用的是PagedIterable，所以它的行为与普通的Iterable类似，但只会在必要时在后台进行API调用。

### 5.2 直接访问仓库

**除了列出仓库之外，我们还可以通过名称直接访问它们**，如果我们有相应的GHUser，那么我们只需要仓库名称：

```java
GHRepository repository = user.getRepository("tutorials");
```

或者，我们可以直接从客户端访问仓库，在这种情况下，我们需要由用户名和仓库名称组合成单个字符串的全名：

```java
GHRepository repository = gitHub.getRepository("tuyucheng/tutorials");
```

这样，我们就得到了完全相同的GHRepository对象，就像我们通过GHUser对象导航一样。

### 5.3 使用仓库

**一旦我们获得了GHRepository对象，我们就可以开始直接与它交互**。

在最简单的层面上，这使我们能够检索仓库详细信息，例如其名称、所有者、创建日期等。

```java
String name = repository.getName();
String fullName = repository.getFullName();
GHUser owner = repository.getOwner();
Date created = repository.getCreatedAt();
```

但是，我们也可以查询仓库的内容，我们可以将其视为Git仓库来实现这一点-允许访问分支、标签、提交等：

```java
String defaultBranch = repository.getDefaultBranch();
GHBranch branch = repository.getBranch(defaultBranch);
String branchHash = branch.getSHA1();

GHCommit commit = repository.getCommit(branchHash);
System.out.println(commit.getCommitShortInfo().getMessage());

```

或者，如果我们知道全名，那么我们就可以访问文件的全部内容：

```java
String defaultBranch = repository.getDefaultBranch();
GHContent file = repository.getFileContent("pom.xml", defaultBranch);

String fileContents = IOUtils.toString(file.read(), Charsets.UTF_8);
```

如果它们存在，我们还可以访问某些特殊文件-具体来说，就是README文件和许可证：

```java
GHContent readme = repository.getReadme();
GHContent license = repository.getLicenseContent();
```

这些操作与直接访问文件完全相同，但不需要知道文件名。具体来说，GitHub支持这些概念的一系列不同文件名，这将返回正确的文件名。

### 5.4 操作仓库

**除了读取仓库内容，我们还可以对其进行更改**。

这可以包括更新仓库本身的配置，允许我们执行诸如更改描述、主页、可见性等操作：

```java
repository.setDescription("A new description");
repository.setVisibility(GHRepository.Visibility.PRIVATE);
```

我们还可以将其他仓库fork到我们自己的帐户中：

```java
repository.createFork().name("my_fork").create();
```

我们还可以创建分支、标签、PR和其他内容：

```java
repository.createRef("new-branch", oldBranch.getSHA1());
repository.createTag("new-tag", "This is a tag", branch.getSHA1(), "commit");
repository.createPullRequest("new-pr", "from-branch", "to-branch", "Description of the pull request");
```

如果需要的话，我们甚至可以创建完整的提交，尽管这更加复杂。

## 6. 总结

在本文中，我们简要介绍了GitHub API Java库，使用这个库可以实现更多功能。