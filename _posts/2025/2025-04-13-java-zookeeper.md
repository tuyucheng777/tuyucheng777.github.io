---
layout: post
title:  Java和Zookeeper入门
category: apache
copyright: apache
excerpt: Apache Zookeeper
---

## 1. 概述

**[Apache ZooKeeper](https://zookeeper.apache.org/)是一种分布式协调服务**，可简化分布式应用程序的开发。Apache Hadoop、[HBase](https://cwiki.apache.org/confluence/display/ZOOKEEPER/PoweredBy)等项目使用它来实现不同的用例，例如领导者选举、配置管理、节点协调、服务器租约管理等。

**ZooKeeper集群内的节点将其数据存储在共享的分层命名空间中**，该命名空间类似于标准文件系统或树形数据结构。

在本文中，我们将探讨如何使用Apache Zookeeper的Java API来存储、更新和删除ZooKeeper中存储的信息。

## 2. 设置

Apache ZooKeeper Java库的最新版本可以在[这里](https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper)找到：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
</dependency>
```

## 3. ZooKeeper数据模型-ZNode

ZooKeeper具有分层命名空间，非常类似于分布式文件系统，其中存储协调数据，如状态信息、协调信息、位置信息等，这些信息存储在不同的节点上。

**ZooKeeper树中的每个节点都称为ZNode**。

每个ZNode都会维护任何数据或ACL变更的版本号和时间戳，此外，这允许ZooKeeper验证缓存并协调更新。

## 4. 部署

### 4.1 安装

最新的ZooKeeper版本可以从[这里](https://www.apache.org/dyn/closer.cgi/zookeeper/)下载，下载之前，需要确保满足[这里](https://zookeeper.apache.org/doc/r3.3.5/zookeeperAdmin.html#sc_systemReq)描述的系统要求。

### 4.2 独立模式

本文将以独立模式运行ZooKeeper，因为它所需的配置极少，请按照[此处](https://zookeeper.apache.org/doc/r3.3.5/zookeeperStarted.html#sc_InstallingSingleMode)文档中描述的步骤操作。

注意：在独立模式下，没有复制，因此如果ZooKeeper进程失败，服务将会中断。

## 5. ZooKeeper CLI示例

我们现在将使用ZooKeeper命令行界面(CLI)与ZooKeeper交互：

```shell
bin/zkCli.sh -server 127.0.0.1:2181
```

上述命令在本地启动了一个独立实例，现在我们来看看如何在ZooKeeper中创建ZNode并存储信息：

```text
[zk: localhost:2181(CONNECTED) 0] create /MyFirstZNode ZNodeVal
Created /FirstZnode
```

我们在ZooKeeper分层命名空间的根目录创建了一个ZNode “MyFirstZNode”，并将“ZNodeVal”写入其中。

由于我们没有传递任何标志，因此创建的ZNode将是持久的。

现在让我们发出一个“get”命令来获取与ZNode相关的数据和元数据：

```text
[zk: localhost:2181(CONNECTED) 1] get /FirstZnode

“Myfirstzookeeper-app”
cZxid = 0x7f
ctime = Sun Feb 18 16:15:47 IST 2018
mZxid = 0x7f
mtime = Sun Feb 18 16:15:47 IST 2018
pZxid = 0x7f
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 22
numChildren = 0
```

我们可以使用set操作来更新现有ZNode的数据。

例如：

```shell
set /MyFirstZNode ZNodeValUpdated
```

这会将MyFirstZNode上的数据从ZNodeVal更新为ZNodeValUpdated。

## 6. ZooKeeper Java API示例

现在让我们看一下Zookeeper Java API并创建一个节点、更新该节点并检索一些数据。

### 6.1 Java包

ZooKeeper Java绑定主要由两个Java包组成：

1. org.apache.zookeeper：定义ZooKeeper客户端库的主类以及ZooKeeper事件类型和状态的许多静态定义
2. org.apache.zookeeper.data：定义与ZNode相关的特征，例如访问控制列表(ACL)、ID、统计信息等

服务器实现中也使用了ZooKeeper Java API，例如org.apache.zookeeper.server、org.apache.zookeeper.server.quorum和org.apache.zookeeper.server.upgrade。

但是，它们超出了本文的讨论范围。

### 6.2 连接到ZooKeeper实例

现在让我们创建ZKConnection类，用于连接和断开已经运行的ZooKeeper：

```java
public class ZKConnection {
    private ZooKeeper zoo;
    CountDownLatch connectionLatch = new CountDownLatch(1);

    // ...

    public ZooKeeper connect(String host) throws IOException, InterruptedException {
        zoo = new ZooKeeper(host, 2000, new Watcher() {
            public void process(WatchedEvent we) {
                if (we.getState() == KeeperState.SyncConnected) {
                    connectionLatch.countDown();
                }
            }
        });

        connectionLatch.await();
        return zoo;
    }

    public void close() throws InterruptedException {
        zoo.close();
    }
}
```

要使用ZooKeeper服务，应用程序**必须首先实例化ZooKeeper类的对象，该类是ZooKeeper客户端库的主类**。

在connect方法中，我们实例化了一个ZooKeeper类的实例。此外，我们还注册了一个回调方法来处理来自ZooKeeper的WatchedEvent事件，以确认连接已成功连接，并使用CountDownLatch的countDown完成connect方法的执行。

一旦与服务器建立连接，客户端就会被分配一个会话ID。为了保持会话有效，客户端应该定期向服务器发送心跳。

只要会话ID仍然有效，客户端应用程序就可以调用ZooKeeper API。

### 6.3 客户端操作

我们现在将创建一个ZKManager接口，它公开不同的操作，例如创建ZNode并保存一些数据、获取和更新ZNode数据：

```java
public interface ZKManager {
    void create(String path, byte[] data) throws KeeperException, InterruptedException;
    Object getZNodeData(String path, boolean watchFlag);
    void update(String path, byte[] data) throws KeeperException, InterruptedException;
}
```

现在我们看一下上述接口的实现：

```java
public class ZKManagerImpl implements ZKManager {
    private static ZooKeeper zkeeper;
    private static ZKConnection zkConnection;

    public ZKManagerImpl() {
        initialize();
    }

    private void initialize() {
        zkConnection = new ZKConnection();
        zkeeper = zkConnection.connect("localhost");
    }

    public void closeConnection() {
        zkConnection.close();
    }

    public void create(String path, byte[] data) throws KeeperException, InterruptedException {
        zkeeper.create(
                path,
                data,
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.PERSISTENT);
    }

    public Object getZNodeData(String path, boolean watchFlag) throws KeeperException, InterruptedException {
        byte[] b = null;
        b = zkeeper.getData(path, null, null);
        return new String(b, "UTF-8");
    }

    public void update(String path, byte[] data) throws KeeperException, InterruptedException {
        int version = zkeeper.exists(path, true).getVersion();
        zkeeper.setData(path, data, version);
    }
}
```

在上面的代码中，连接和断开连接的调用被委托给了之前创建的ZKConnection类，我们的create方法用于根据字节数组数据在给定路径创建一个ZNode。仅出于演示目的，我们保持ACL完全开放。

一旦创建，ZNode就会持久存在，并且在客户端断开连接时不会被删除。

在getZNodeData方法中，从ZooKeeper获取ZNode数据的逻辑非常简单。最后，使用update方法，我们检查给定路径上是否存在ZNode，如果存在则获取它。

除此之外，为了更新数据，我们首先检查ZNode是否存在并获取当前版本。然后，我们调用setData方法，并以ZNode的路径、数据和当前版本作为参数。只有当传递的版本与最新版本匹配时，ZooKeeper才会更新数据。

## 7. 总结

在开发分布式应用程序时，Apache ZooKeeper作为分布式协调服务发挥着至关重要的作用。具体来说，它适用于存储共享配置、选举主节点等用例。

ZooKeeper还提供了一个优雅的基于Java的API，可用于客户端应用程序代码，实现与ZooKeeper ZNodes的无缝通信。