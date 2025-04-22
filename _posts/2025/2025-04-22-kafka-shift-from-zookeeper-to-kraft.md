---
layout: post
title:  Kafka从ZooKeeper转向Kraft
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

Kafka的架构最近已从ZooKeeper转向基于仲裁的控制器，该控制器使用一种名为Kafka Raft(缩写为Kraft)的新共识协议。

在本教程中，我们将探讨Kafka做出此决定的原因，以及这一变化如何简化其架构并使其更加强大。

## 2. ZooKeeper简介

**[ZooKeeper](https://zookeeper.apache.org/)是一项支持高可靠性分布式协调的服务**，它最初由雅虎开发，旨在简化大数据集群上运行的流程。它最初是Hadoop的一个子项目，后来于2008年成为独立的Apache基金会项目，被广泛用于服务大型分布式系统中的多种用例。

### 2.1 ZooKeeper架构

ZooKeeper将**数据存储在分层命名空间中**，类似于标准文件系统。命名空间由[称为znode的数据寄存器](https://www.baeldung.com/java-zookeeper)组成，名称是由斜杠分隔的路径元素序列。

命名空间中的每个节点都由一个路径标识：

![](/assets/images/2025/kafka/kafkashiftfromzookeepertokraft01.png)

ZooKeeper命名空间中可以有三种类型的znode：

- 第一种是持久节点，这是默认类型，并且会一直保留在ZooKeeper中，直到被删除。
- 第二种是临时节点，如果创建该znode的会话断开连接，该znode就会被删除。此外，临时znode不能有子节点。
- 第三种是顺序节点，我们可以使用它来创建像ID这样的顺序号。

ZooKeeper凭借其简单的架构，提供了一个具有快速处理和可扩展性的可靠系统。它旨在**通过一组称为“集群”的服务器进行复制**，每个服务器都会在内存中维护状态镜像，并在持久存储中维护状态转换日志和快照：

![](/assets/images/2025/kafka/kafkashiftfromzookeepertokraft02.png)

ZooKeeper客户端只能连接到一台服务器，但如果该服务器不可用，则可以故障转移到另一台服务器。读取请求由每台服务器数据库的本地副本处理，写入请求由协议处理。这涉及将所有此类请求转发到领导服务器，该服务器使用[ZooKeeper原子广播(ZAB)协议](https://zookeeper.apache.org/doc/r3.4.13/zookeeperInternals.html#sc_atomicBroadcast)进行协调。

基本上，原子消息传递是ZooKeeper的核心，它使所有服务器保持同步。它确保消息的可靠传递，并确保消息按完整且因果顺序传递。消息传递系统基本上在服务器之间建立点对点的FIFO通道，利用TCP进行通信。

### 2.2 ZooKeeper使用

**ZooKeeper为来自客户端的所有更新提供顺序一致性和原子性**，此外，它不允许并发写入。此外，无论客户端连接到哪个服务器，它始终看到的服务视图相同。总而言之，ZooKeeper为高性能、高可用性和严格有序的访问提供了卓越的保障。

ZooKeeper还实现了极高的吞吐量和极低的延迟，这些特性使其非常**适合解决大型分布式系统中的诸多协调问题**，例如命名服务、配置管理、数据同步、领导者选举、消息队列和通知系统等用例。

## 3. Kafka中的ZooKeeper

**[Kafka](https://kafka.apache.org/)是一个分布式事件存储和流处理平台**，它最初由LinkedIn开发，并于2011年由Apache软件基金会开源。Kafka提供了一个高吞吐量、低延迟的实时数据处理平台，被广泛用于流分析和数据集成等高性能用例。

### 3.1 Kafka架构

Kafka是一个分布式系统，[由服务器和客户端组成](https://www.baeldung.com/spring-kafka)，它们使用基于TCP的二进制协议进行通信。它旨在以由一个或多个服务器(也称为代理)组成的集群形式运行，代理还充当事件的存储层。

Kafka将事件持久化地组织到主题中，主题可以有0个、1个或多个生产者和消费者。主题还会进行分区，并**分布在不同的Broker上**，以实现高可扩展性。此外，每个主题还可以在集群内进行复制：

![](/assets/images/2025/kafka/kafkashiftfromzookeepertokraft03.png)

在Kafka集群中，其中一个Broker充当控制器。控制器负责管理分区和副本的状态，并执行诸如重新分配分区之类的管理任务。在任何时间点，集群中只能有一个控制器。

客户端使应用程序能够以并行、大规模和容错的方式读取、写入和处理事件流。生产者是将事件发布到Kafka的客户端应用程序，与此同时，消费者是从Kafka订阅这些事件的应用程序。

### 3.2 ZooKeeper的作用

Kafka的设计特性使其具备高可用性和容错能力，但是，作为一个分布式系统，Kafka需要一种机制来协调所有活跃Broker之间的多个决策。它还需要维护集群及其配置的一致性视图，Kafka长期以来一直使用ZooKeeper来实现这一点。

基本上，直到最近对Kraft进行更改之前，**ZooKeeper一直作为Kafka的元数据管理工具来完成几个关键功能**：

![](/assets/images/2025/kafka/kafkashiftfromzookeepertokraft04.png)

- **控制器选举**：控制器选举主要依赖于ZooKeeper，为了选举控制器，每个Broker都会尝试在ZooKeeper中创建一个临时节点，第一个创建此临时节点的Broker将承担控制器角色，并被分配一个控制器epoch。
- **集群成员资格**：ZooKeeper在管理集群中Broker成员资格方面发挥着重要作用，当Broker连接到ZooKeeper实例时，会在组znode下创建一个临时znode，如果Broker发生故障，此临时znode将被删除。
- **主题配置**：Kafka在ZooKeeper中为每个主题维护一组配置，这些配置可以是针对每个主题的，也可以是全局的。它还存储了诸如现有主题列表、每个主题的分区数量以及副本位置等详细信息。
- **访问控制列表(ACL)**：Kafka还维护ZooKeeper中所有主题的ACL，这有助于决定哪些用户或哪些内容有权在每个主题上进行读写，它还保存了消费者组列表以及每个消费者组的成员等信息。
- **配额**：Kafka Broker可以控制客户端可以使用的Broker资源，这些资源以配额的形式存储在ZooKeeper中。配额有两种类型：由字节速率阈值定义的网络带宽配额和由CPU利用率阈值定义的请求速率配额。

### 3.3 ZooKeeper的问题

正如我们所见，ZooKeeper长期以来在Kafka架构中扮演着重要的角色，并且取得了显著的成功。那么，他们为什么决定改变它呢？简而言之，ZooKeeper为Kafka增加了一个额外的管理层，即使像ZooKeeper这样简单而强大的分布式系统管理仍然是一项复杂的任务。

Kafka并非唯一一个需要机制来协调其成员间任务的分布式系统，还有其他一些系统，例如MongoDB、Cassandra和Elasticsearch，它们都以各自的方式解决了这个问题。然而，它们并不依赖ZooKeeper等外部工具进行元数据管理。基本上，**它们依靠内部机制来实现这一目的**。

除了其他优势之外，它还使部署和运维管理更加简单。想象一下，如果我们只需要管理一个分布式系统而不是两个，那该有多好，此外，由于元数据处理效率更高，**可扩展性也得到了提升**。将元数据存储在Kafka内部而不是ZooKeeper中，使管理更加轻松，并提供更好的保障。

## 4. Kafka Raft(Kraft)协议

受Kafka与ZooKeeper复杂性的启发，有人提交了Kafka改进提案(KIP)，旨在**用自管理元数据仲裁机制取代ZooKeeper**。基础[KIP 500](https://cwiki.apache.org/confluence/display/KAFKA/KIP-500%3A+Replace+ZooKeeper+with+a+Self-Managed+Metadata+Quorum)定义了愿景，随后又陆续发布了多个KIP来完善细节，自管理模式最初作为Kafka 2.8的抢先体验版发布。

自管理模式将元数据管理的责任整合到Kafka内部，此模式利用Kafka中新的仲裁控制器服务，**仲裁控制器使用事件源存储模型**。此外，它使用Kafka Raft(Kraft)作为共识协议，以确保元数据在仲裁中准确复制。

**Kraft本质上是Raft共识协议基于事件的变体**，它也与ZAB协议类似，但显著的区别在于它采用事件驱动的架构。仲裁控制器使用事件日志来存储状态，并定期将其压缩为快照，以防止其无限增长：

![](/assets/images/2025/kafka/kafkashiftfromzookeepertokraft05.png)

其中一个仲裁控制器充当领导者，并在Kafka的元数据主题中创建事件。仲裁中的其他控制器通过响应这些事件来跟随领导者控制器，当其中一个代理因分区而发生故障时，它可以在重新加入后从日志中补齐丢失的事件，这缩短了不可用窗口。

与基于ZooKeeper的控制器不同，仲裁控制器无需从ZooKeeper加载状态。当领导节点发生变更时，新的活动控制器已在内存中拥有所有已提交的元数据记录。此外，仲裁控制器还使用相同的事件驱动机制来跟踪整个集群的所有元数据。

## 5. 简化且更好的Kafka

改用基于仲裁的控制器预计将为Kafka社区带来显著的缓解，首先，系统管理员将更容易监控、管理和支持Kafka，开发人员只需处理整个系统的单一安全模型。此外，我们提供了一个轻量级的单进程部署，方便用户快速上手Kafka。

新的元数据管理也显著提升了Kafka控制平面的性能，首先，它使控制器能够更快地进行故障转移，基于ZooKeeper的元数据管理一直是集群范围内分区限制的瓶颈，新的Quorum控制器旨在处理每个集群中数量更多的分区。

自Kafka 2.8版本起，自管理(Kraft)模式与ZooKeeper一同可用，该功能在3.0版本中作为预览功能发布，经过多项改进，Kraft模式已于3.3.1版本正式发布，可用于生产环境并可能会在3.4版本中弃用ZooKeeper。

## 6. 总结

在本文中，我们讨论了ZooKeeper的细节及其在Kafka中的作用。此外，我们还探讨了该架构的复杂性，以及Kafka选择用基于Quorum的控制器取代ZooKeeper的原因。

最后，我们介绍了这一变化在简化架构和更好的可扩展性方面给Kafka带来的好处。