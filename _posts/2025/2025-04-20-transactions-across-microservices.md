---
layout: post
title:  跨微服务事务指南
category: designpattern
copyright: designpattern
excerpt: 事务
---

## 1. 简介

在本文中，我们将讨论跨微服务实现事务的选项。

我们还将研究分布式微服务场景中事务的一些替代方案。

## 2. 避免跨微服务事务

分布式事务是一个非常复杂的过程，涉及许多可能失败的移动部件。此外，如果这些部件运行在不同的机器上，甚至不同的数据中心，提交事务的过程可能会变得非常漫长且不可靠。

这会严重影响用户体验和整体系统带宽，因此，**解决分布式事务问题的最佳方法之一就是完全避免它们**。

### 2.1 需要事务的架构示例

通常，微服务的设计目标是独立且可独立使用，它应该能够解决一些原子业务任务。

**如果我们可以将我们的系统分成这样的微服务，那么很有可能我们根本不需要在它们之间实现事务**。

例如，让我们考虑一个用户之间的广播消息系统。

user微服务将关注用户配置文件(创建新用户、编辑配置文件数据等)，并具有以下底层域类：

```java
@Entity
public class User implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    
    @Basic
    private String name;
    
    @Basic
    private String surname;
    
    @Basic
    private Instant lastMessageTime;
}
```

message微服务主要负责广播，它封装了实体Message及其相关的所有内容：

```java
@Entity
public class Message implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @Basic
    private long userId;

    @Basic
    private String contents;

    @Basic
    private Instant messageTimestamp;
}
```

每个微服务都有自己的数据库，请注意，我们不会从实体Message中引用实体User，因为message微服务无法访问用户类，我们仅通过id来引用用户。

现在，User实体包含lastMessageTime字段，因为我们想在个人资料中显示有关最后用户活动时间的信息。

但是，为了向用户添加新消息并更新lastMessageTime，我们现在必须跨微服务实现事务。

### 2.2 无事务的替代方法

我们可以改变我们的微服务架构并从User实体中删除字段lastMessageTime。

然后，我们可以通过向message微服务发出单独的请求并找到该用户所有消息的最大messageTimestamp值来在用户个人资料中显示此时间。

可能，如果message微服务负载过高甚至瘫痪，我们将无法在用户的个人资料中显示其最后一条消息的时间。

**但这比仅仅因为user微服务没有及时响应而无法提交分布式事务来保存消息更容易接受**。

当然，当我们必须[跨多个微服务](https://www.baeldung.com/cs/microservices-cross-cutting-concerns)实现业务流程时，情况会更加复杂，并且我们不希望这些微服务之间存在不一致。

## 3. 两阶段提交协议

[两阶段提交协议](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)(或2PC)是一种跨不同软件组件(多个数据库、消息队列等)实现事务的机制。

### 3.1 2PC的架构

分布式事务中的重要参与者之一是事务协调器，分布式事务包含两个步骤：

- 准备阶段：在此阶段，事务的所有参与者都准备提交，并通知协调者他们已准备好完成事务
- 提交或回滚阶段：在此阶段，事务协调器向所有参与者发出提交或回滚命令

**2PC的问题在于，与单个微服务的运行时间相比，它相当慢**。

**协调微服务之间的事务，即使它们位于同一个网络上，也会真正降低系统速度**，因此这种方法通常不用于高负载场景。

### 3.2 XA标准

[XA标准](https://en.wikipedia.org/wiki/X/Open_XA)是用于在支持资源之间执行2PC分布式事务的规范，任何兼容JTA的应用服务器(JBoss、GlassFish等)都开箱即用地支持该标准。

参与分布式事务的资源可以是两个不同微服务的两个数据库。

**然而，要利用此机制，必须将资源部署到单个JTA平台，这对于微服务架构来说并不总是可行的**。

### 3.3 REST-AT标准草案

另一个提议的标准是[REST-AT](https://github.com/jbosstm/documentation/tree/master/rts/docs)，它由RedHat进行了一些开发，但仍未完成草案阶段。不过，WildFly应用服务器开箱即用地支持它。

该标准允许使用应用程序服务器作为事务协调器，并使用特定的REST API来创建和加入分布式事务。

希望参与两阶段事务的RESTful Web服务也必须支持特定的REST API。

不幸的是，为了将分布式事务桥接到微服务的本地资源，我们仍然必须将这些资源部署到单个JTA平台，或者解决自己编写此桥接器这一非平凡任务。

## 4. 最终一致性和补偿

**到目前为止，处理跨微服务一致性的最可行模型之一是[最终一致性](https://www.baeldung.com/cs/eventual-consistency-vs-strong-eventual-consistency-vs-strong-consistency)**。

该模型并不强制跨微服务实施分布式ACID事务，相反，它建议使用一些机制来确保系统在未来某个时间点达到最终一致性。

### 4.1 最终一致性的案例

例如，假设我们需要解决以下任务：

- 注册用户资料
- 进行一些自动背景检查，确保用户确实可以访问系统

第二项任务是确保该用户不会因为某种原因被禁止访问我们的服务器。

但这可能需要一些时间，我们希望将其提取到一个单独的微服务中，让用户等待这么长时间才知道注册成功是不合理的。

**解决这个问题的一种方法是采用包含补偿的消息驱动方法**，让我们考虑以下架构：

- 负责注册用户资料的user微服务
- 负责进行背景调查的validation微服务
- 支持持久队列的消息传递平台

消息平台可以确保微服务发送的消息被持久化，如果接收者当前不可用，消息将在稍后被传递。

### 4.2 快乐场景

在这种架构下，理想的情况是：

- user微服务注册一个用户，并将有关用户的信息保存在本地数据库中
- user微服务会标记该用户，这可能表示该用户尚未经过验证，无法访问完整的系统功能
- 向用户发送注册确认信息，并警告用户系统并非所有功能均可立即访问
- user微服务向validation微服务发送消息以对用户进行背景检查
- validation微服务运行背景检查，并向user微服务发送检查结果消息
  - 如果结果为true，则user微服务将解除对用户的阻止
  - 如果结果为false，则user微服务删除该用户帐户

完成所有这些步骤后，系统应该处于一致的状态。然而，在一段时间内，用户实体似乎处于不完整的状态。

**最后一步，当user微服务删除无效账户时，就是补偿阶段**。

### 4.3 故障场景

现在让我们考虑一些失败的情况：

- 如果无法访问validation微服务，则消息传递平台及其持久队列功能可确保validation微服务稍后收到此消息
- 假设消息传递平台出现故障，则user微服务会尝试在稍后的某个时间再次发送消息，例如，通过对所有尚未验证的用户进行计划的批处理
- 如果validation微服务收到消息，验证用户，但由于消息传递平台故障而无法发回响应，则validation微服务也会在稍后重试发送消息
- 如果其中一条消息丢失，或者发生其他故障，user微服务将通过预定的批处理找到所有未验证的用户，并再次发送验证请求

即使某些消息被发出多次，也不会影响微服务数据库中数据的一致性。

**通过仔细考虑所有可能的故障场景，我们可以确保系统满足最终一致性的条件。同时，我们无需处理成本高昂的分布式事务**。

但我们必须意识到，确保最终一致性是一项复杂的任务，它没有一个适用于所有情况的单一解决方案。

## 5. 总结

在本文中，我们讨论了一些跨微服务实现事务的机制。并且，我们也首先探索了一些替代这种事务方式的方法。