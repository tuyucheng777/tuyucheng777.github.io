---
layout: post
title:  JNDI – 什么是java:comp/env？
category: java
copyright: java
excerpt: Java JNDI
---

## 1. 概述

Java命名和目录接口([JNDI](https://www.baeldung.com/jndi))是一种应用程序编程接口(API)，它为基于Java的应用程序提供命名和目录服务。我们可以使用此接口绑定对象/资源、查找或查询对象以及检测同一对象上的更改。

在本教程中，**我们将特别探讨在JNDI命名中使用“java:comp/env”标准前缀的背景**。

## 2. 什么是JNDI？

简单来说，命名服务是一个将名称与对象关联起来的接口，随后借助名称来查找这些对象。**因此，命名服务维护一组将名称与对象映射的绑定**。

JNDI API使应用程序组件和客户端能够查找分布式资源、服务和EJB。

## 3. 访问命名上下文

[Context](https://www.baeldung.com/jndi#2-context-interface)接口提供对命名环境的访问，使用此对象，我们可以将名称绑定到对象、重命名对象以及列出绑定。让我们看看如何获取上下文：

```xml
JndiTemplate jndiTemplate = new JndiTemplate();
context = (InitialContext) jndiTemplate.getContext();
```

一旦我们有了上下文，我们就可以绑定对象：

```java
context.bind("java:comp/env/jdbc/datasource", ds);
```

然后，我们可以检索上下文中存在的对象：

```java
context.lookup("java:comp/env/jdbc/datasource");
```

## 4. 什么是java:comp/env？

如前面的示例所示，我们用标准前缀“java:comp/env”绑定名称。在我们的例子中，它是“java:comp/env/jdbc/datasource”，而不仅仅是“jdbc/datasource”。这个前缀是强制性的吗？我们可以完全避免它吗？让我们讨论一下。

### 4.1 目录服务

JNDI，顾名思义，是一种命名和目录服务。因此，**[目录服务](https://docs.oracle.com/javase/jndi/tutorial/getStarted/concepts/directory.html)将名称与对象关联起来，并允许这些对象具有属性**。因此，我们不仅可以通过名称查找对象，还可以获取对象的属性或根据其属性查找对象。一个实际的例子是电话目录服务，其中订户的姓名不仅映射到他的电话号码，还映射到他的地址。

目录通常按层次结构排列对象，在大多数情况下，目录对象存储在树结构中。因此，第一个元素/节点可能包含组对象，而组对象又可能包含特定对象。

例如，在“java:comp/env”中，“comp”元素是第一个节点，在下一级，它包含“env”元素。从这里，我们可以根据我们的约定存储或访问资源。例如，“jdbc/datasource”共享数据源对象。

### 4.2 分解

让我们分解一下之前的命名示例：“java:comp/env/jdbc/datasource”。

- **java**就像JDBC连接字符串中的“jdbc:”，它充当一种协议。
- **comp**是所有JNDI上下文的根，它绑定到为组件相关绑定保留的子树，名称“comp”是component的缩写，根上下文中没有其他绑定。
- **env**绑定到为组件的环境相关绑定保留的子树，“env”是environment的缩写。
- **jdbc**是jdbc资源的子上下文，还有[其他类型](https://docs.oracle.com/cd/E19747-01/819-0079/dgjndi.html)，或连接工厂的子上下文。
- **datasource**是我们的JDBC资源的名称。

这里，**除了最后一个元素外，其他所有父元素都是标准名称，因此不能改变**。

**此外，根上下文是为策略的未来扩展而保留的**。具体来说，这适用于命名不与组件本身绑定但与其他类型实体(如用户或部门)绑定的资源。例如，未来的策略可能允许我们使用诸如“java:user/Anne”和“java:org/finance”之类的名称来命名用户和组织/部门。

## 5. 相对路径与绝对路径

如果我们想使用更短版本的JNDI查找，我们可以借助子上下文来实现。这样，我们不需要提及命名的完整路径(绝对路径)，而是子上下文的相对路径。

我们可以从InitialContext对象派生出一个子上下文，这样我们就不必为检索的每个资源重复“java：comp/env”：

```java
Context subContext = (Context) context.lookup("java:comp/env");
DataSource ds = (DataSource) subContext.lookup("jdbc/datasource");
```

正如我们在这里看到的，我们创建了一个子上下文，它保存了“java：comp/env”内的值，然后使用这个子上下文(相对路径)来查找其中的特定命名。

## 6. 总结

在本文中，我们快速了解了什么是JNDI及其用例。然后，我们了解了如何将JNDI名称绑定到上下文并查找该名称。

随后，我们看到了标准前缀“java:comp/env”的拆分，以及在JNDI命名中使用此前缀的原因。我们还注意到，未来的政策可能会扩展根上下文“comp”和子上下文“env”。