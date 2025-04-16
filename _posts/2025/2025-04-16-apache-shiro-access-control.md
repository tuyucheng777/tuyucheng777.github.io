---
layout: post
title:  使用Apache Shiro进行基于权限的访问控制
category: security
copyright: security
excerpt: Apache Shiro
---

## 1. 简介

在本教程中，我们将研究如何使用[Apache Shiro](https://www.baeldung.com/apache-shiro) Java安全框架实现细粒度的基于权限的访问控制。

## 2. 设置

我们将使用与Shiro介绍相同的设置-也就是说，我们只将[shiro-core](https://mvnrepository.com/artifact/org.apache.shiro/shiro-core)模块添加到依赖中：

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.1</version>
</dependency>
```

此外，出于测试目的，我们将使用一个简单的INI Realm，将以下shiro.ini文件放在类路径的根目录下：

```properties
[users]
jane.admin = password, admin
john.editor = password2, editor
zoe.author = password3, author
 
[roles]
admin = *
editor = articles:*
author = articles:create, articles:edit
```

然后，我们将使用上述Realm初始化Shiro：

```java
IniRealm iniRealm = new IniRealm("classpath:shiro.ini");
SecurityManager securityManager = new DefaultSecurityManager(iniRealm);
SecurityUtils.setSecurityManager(securityManager);
```

## 3. 角色和权限

通常，当我们谈论身份验证和授权时，我们关注用户和角色的概念。

具体而言，**角色是应用程序或服务的跨领域用户类别**。因此，所有具有特定角色的用户都将有权访问某些资源和操作，但可能对应用程序或服务的其他部分具有受限访问权限。

角色集通常是预先设计的，很少会因新的业务需求而改变。但是，角色也可以动态定义-例如由管理员定义。

使用Shiro，我们有几种方法来测试用户是否具有特定角色，最直接的方法是使用hasRole方法：

```java
Subject subject = SecurityUtils.getSubject();
if (subject.hasRole("admin")) {       
    logger.info("Welcome Admin");              
}
```

### 3.1 权限

但是，如果我们通过测试用户是否具有特定角色来检查授权，则会出现问题。实际上，**我们正在硬编码角色和权限之间的关系**；换句话说，当我们想要授予或撤销对资源的访问权限时，我们必须更改源代码。当然，这也意味着重新构建和重新部署。

我们可以做得更好；这就是为什么我们现在要介绍权限的概念。**权限表示软件可以做什么，我们可以授权或拒绝哪些操作，而不是谁可以执行这些操作**。例如，“编辑当前用户的个人资料”、“批准文档”或“创建新文章”。

Shiro对权限做了很少的假设，在最简单的情况下，权限是纯字符串：

```java
Subject subject = SecurityUtils.getSubject();
if (subject.isPermitted("articles:create")) {
    //Create a new article
}
```

请注意，**在Shiro中权限的使用完全是可选的**。

### 3.2 将权限与用户关联

Shiro具有将权限与角色或个人用户关联的灵活模型，但是，典型的Realm(包括我们在本教程中使用的简单INI Realm)仅将权限与角色关联。

因此，由Principal标识的用户具有多个角色，并且每个角色具有多个Permission。

例如，我们可以在INI文件中看到，用户zoe.author具有author角色，并赋予他们articles:create和articles:edit权限：

```properties
[users]
zoe.author = password3, author
#Other users...

[roles]
author = articles:create, articles:edit
#Other roles...
```

类似地，可以配置其他Realm类型(例如内置的JDBC Realm)以将权限与角色关联。

## 4. 通配符权限

**Shiro中权限的默认实现是通配符权限**，可以灵活地表示多种权限方案。

在Shiro中我们用字符串来表示通配符权限，权限字符串由一个或多个用冒号分隔的组件组成，例如：

```plaintext
articles:edit:1
```

字符串每个部分的含义取决于应用程序，因为Shiro不强制执行任何规则。但是，在上面的例子中，我们可以很清楚地将字符串解释为一个层次结构：

1. 我们公开的资源类别(articles)
2. 对此类资源的操作(edit)
3. 我们要允许或拒绝操作的特定资源的ID

这种资源:操作:id的三层结构是Shiro应用程序中的常见模式，因为它既简单又有效地表示许多不同的场景。

因此，我们可以重新审视前面的例子来遵循这个方案：

```java
Subject subject = SecurityUtils.getSubject();
if (subject.isPermitted("articles:edit:123")) {
    //Edit article with id 123
}
```

请注意，**通配符权限字符串中的组件数不必是三个，尽管通常情况是三个组件**。

### 4.1 权限隐含和实例级粒度

当我们将通配符权限与Shiro权限的另一个特性-隐含结合起来时，通配符权限就会大放异彩。

**当我们测试角色时，我们测试的是确切的成员资格**：Subject要么具有特定角色，要么不具有。换句话说，Shiro测试角色是否相等。

另一方面，**当我们测试权限时，我们会测试隐含**：Subject的权限是否隐含了我们正在测试的权限？

隐含的具体含义取决于权限的实现，实际上，对于通配符权限，隐含是指部分字符串匹配，顾名思义，存在通配符的可能性。

因此，假设我们为author角色分配以下权限：

```properties
[roles]
author = articles:*
```

然后，具有author角色的每个人都可以对文章进行所有可能的操作：

```java
Subject subject = SecurityUtils.getSubject();
if (subject.isPermitted("articles:create")) {
    //Create a new article
}
```

也就是说，字符串articles:将与任何第一个部分为articles的通配符权限匹配。

通过这种方案，我们既可以分配非常具体的权限(对具有给定ID的特定资源执行特定操作)，也可以分配广泛的权限，例如编辑任何文章或对任何文章执行任何操作。

当然，出于性能原因，由于隐含不是简单的相等比较，**我们应该始终针对最具体的权限进行测试**：

```java
if (subject.isPermitted("articles:edit:1")) { //Better than "articles:*"
    //Edit article
}
```

## 5. 自定义权限实现

让我们简单谈谈权限自定义，尽管通配符权限涵盖了广泛的场景，但我们可能希望用为我们的应用程序定制的解决方案来替换它们。

假设我们需要对路径的权限进行建模，使得对某个路径的权限意味着对所有子路径的权限。实际上，我们可以使用通配符权限来完成这项任务，但我们先忽略这一点。

那么，我们需要什么？

1. Permission实现
2. 告诉Shiro

让我们看看如何实现这两点。

### 5.1 写入权限实现

**Permission实现是一个具有单一方法的类-意味着**：

```java
public class PathPermission implements Permission {

    private final Path path;

    public PathPermission(Path path) {
        this.path = path;
    }

    @Override
    public boolean implies(Permission p) {
        if(p instanceof PathPermission) {
            return ((PathPermission) p).path.startsWith(path);
        }
        return false;
    }
}
```

如果这隐含了其他权限对象，则该方法返回true，否则返回false。

### 5.2 告诉Shiro我们的实现

然后，有多种方法可以将Permission实现集成到Shiro中，但最直接的方法是将自定义PermissionResolver注入到我们的Realm中：

```java
IniRealm realm = new IniRealm();
Ini ini = Ini.fromResourcePath(Main.class.getResource("/com/.../shiro.ini").getPath());
realm.setIni(ini);
realm.setPermissionResolver(new PathPermissionResolver());
realm.init();

SecurityManager securityManager = new DefaultSecurityManager(realm);
```

**PermissionResolver负责将权限的字符串表示形式转换为实际的Permission对象**：

```java
public class PathPermissionResolver implements PermissionResolver {
    @Override
    public Permission resolvePermission(String permissionString) {
        return new PathPermission(Paths.get(permissionString));
    }
}
```

我们必须使用基于路径的权限来修改我们之前的shiro.ini：

```properties
[roles]
admin = /
editor = /articles
author = /articles/drafts
```

然后，我们将能够检查路径的权限：

```java
if(currentUser.isPermitted("/articles/drafts/new-article")) {
    log.info("You can access articles");
}
```

请注意，我们在这里以编程方式配置一个简单的Realm，在典型的应用程序中，我们将使用shiro.ini文件或其他方式(如Spring)来配置Shiro和Realm；实际的shiro.ini文件可能包含：

```properties
[main]
permissionResolver = cn.tuyucheng.taketoday.shiro.permissions.custom.PathPermissionResolver
dataSource = org.apache.shiro.jndi.JndiObjectFactory
dataSource.resourceName = java://app/jdbc/myDataSource

jdbcRealm = org.apache.shiro.realm.jdbc.JdbcRealm
jdbcRealm.dataSource = $dataSource
jdbcRealm.permissionResolver = $permissionResolver
```

## 6. 总结

在本文中，我们回顾了Apache Shiro如何实现基于权限的访问控制。