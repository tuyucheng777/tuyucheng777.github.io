---
layout: post
title:  JIRA REST API集成
category: saas
copyright: saas
excerpt: JIRA
---

## 1. 简介

在本文中，我们将快速了解如何使用其REST API与JIRA集成。

## 2. Maven依赖

所需的工件可以在Atlassian的公共Maven仓库中找到：

```xml
<repository>
    <id>atlassian-public</id>
    <url>https://packages.atlassian.com/maven/repository/public</url>
</repository>
```

将仓库添加到pom.xml后，我们需要添加以下依赖：

```xml
<dependency>
    <groupId>com.atlassian.jira</groupId>
    <artifactId>jira-rest-java-client-core</artifactId>
    <version>4.0.0</version>
</dependency>
<dependency>
    <groupId>com.atlassian.fugue</groupId>
    <artifactId>fugue</artifactId>
    <version>2.6.1</version>
</dependency>
```

你可以参考Maven Central来获取[core](https://mvnrepository.com/artifact/com.atlassian.jira/jira-rest-java-client-core)和[fugue](https://mvnrepository.com/artifact/com.atlassian.fugue/fugue)依赖的最新版本。

## 3. 创建Jira客户端

首先，让我们看一下连接到Jira实例所需的一些基本信息：

- username：任何有效Jira用户的用户名
- password：该用户的密码
- jiraUrl：托管Jira实例的URL

一旦我们有了这些详细信息，我们就可以实例化我们的Jira客户端：

```java
MyJiraClient myJiraClient = new MyJiraClient(
    "user.name", 
    "password", 
    "http://jira.company.com");
```

该类的构造函数：

```java
public MyJiraClient(String username, String password, String jiraUrl) {
    this.username = username;
    this.password = password;
    this.jiraUrl = jiraUrl;
    this.restClient = getJiraRestClient();
}
```

getJiraRestClient()利用提供的所有信息并返回JiraRestClient的实例，这是我们与Jira REST API通信的主要接口：

```java
private JiraRestClient getJiraRestClient() {
    return new AsynchronousJiraRestClientFactory()
        .createWithBasicHttpAuthentication(getJiraUri(), this.username, this.password);
}
```

这里我们使用基本身份验证与API进行通信，不过，我们也支持更复杂的身份验证机制，例如OAuth。

getUri()方法只是将jiraUrl转换为java.net.URI的实例：

```java
private URI getJiraUri() {
    return URI.create(this.jiraUrl);
}
```

至此，我们完成了自定义Jira客户端的基础架构创建。现在，我们可以看看与API交互的各种方式。

### 3.1 创建新问题

我们先创建一个新问题，本文中的其他所有示例都将使用这个新创建的问题：

```java
public String createIssue(String projectKey, Long issueType, String issueSummary) {
    IssueRestClient issueClient = restClient.getIssueClient();
    IssueInput newIssue = new IssueInputBuilder(projectKey, issueType, issueSummary).build();
    return issueClient.createIssue(newIssue).claim().getKey();
}
```

projectKey是定义你项目的唯一键，它只是附加到我们所有问题的前缀。下一个参数issueType也取决于项目，它标识问题的类型，例如“Task”或“Story”，issueSummary是我们问题的标题。

问题会以IssueInput的实例形式传递给REST API，除了我们之前提到的输入之外，受理人、报告人、受影响的版本以及其他元数据也可以作为IssueInput传递。

### 3.2 更新问题描述

Jira中的每个问题都由一个唯一的字符串标识，例如“MYKEY-123”，我们需要此问题键来与REST API交互并更新问题描述：

```java
public void updateIssueDescription(String issueKey, String newDescription) {
    IssueInput input = new IssueInputBuilder()
        .setDescription(newDescription)
        .build();
    restClient.getIssueClient()
        .updateIssue(issueKey, input)
        .claim();
}
```

一旦描述更新，我们就不要再读回更新后的描述：

```java
public Issue getIssue(String issueKey) {
    return restClient.getIssueClient()
        .getIssue(issueKey) 
        .claim();
}
```

Issue实例表示由issueKey标识的问题，我们可以使用此实例来读取此问题的描述：

```java
Issue issue = myJiraClient.getIssue(issueKey);
System.out.println(issue.getDescription());
```

这会将问题的描述打印到控制台。

### 3.3 对问题进行投票

一旦我们获得了Issue的实例，我们就可以使用它来执行更新/编辑操作，让我们为这个问题投票：

```java
public void voteForAnIssue(Issue issue) {
    restClient.getIssueClient()
        .vote(issue.getVotesUri())
        .claim();
}
```

这将代表使用其凭证的用户向问题添加投票。可以通过检查投票数来验证：

```java
public int getTotalVotesCount(String issueKey) {
    BasicVotes votes = getIssue(issueKey).getVotes();
    return votes == null ? 0 : votes.getVotes();
}
```

这里要注意的一点是，我们再次在这里获取一个新的Issue实例，因为我们希望获得更新后的投票数。

### 3.4 添加评论

我们可以使用同一个Issue实例来代表用户添加评论，就像添加投票一样，添加评论也非常简单：

```java
public void addComment(Issue issue, String commentBody) {
    restClient.getIssueClient()
        .addComment(issue.getCommentsUri(), Comment.valueOf(commentBody));
}
```

我们使用了Comment类提供的工厂方法valueOf()来创建Comment的实例，此外，还有许多其他工厂方法可用于高级用例，例如控制Comment的可见性。

让我们获取该Issue的一个新实例并读取所有Comments：

```java
public List<Comment> getAllComments(String issueKey) {
    return StreamSupport.stream(getIssue(issueKey).getComments().spliterator(), false)
        .collect(Collectors.toList());
}
```

### 3.5 删除问题

删除问题也相当简单，我们只需要标识该问题的键：

```java
public void deleteIssue(String issueKey, boolean deleteSubtasks) {
    restClient.getIssueClient()
        .deleteIssue(issueKey, deleteSubtasks)
        .claim();
}
```

## 4. 总结

在这篇快速文章中，我们创建了一个简单的Java客户端，它与Jira REST API集成并执行一些基本操作。