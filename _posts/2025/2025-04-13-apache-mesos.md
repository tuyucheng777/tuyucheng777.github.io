---
layout: post
title:  Apache Mesos指南
category: apache
copyright: apache
excerpt: Apache Mesos
---

## 1. 概述

我们通常将各种应用程序部署在同一台机器集群上，例如，现在常见的情况是，在同一个集群中同时部署[Apache Spark](https://www.baeldung.com/apache-spark)或[Apache Flink](https://www.baeldung.com/apache-flink)等分布式处理引擎和[Apache Cassandra](https://www.baeldung.com/cassandra-with-java)等分布式数据库。

**Apache Mesos是一个允许此类应用程序之间有效共享资源的平台**。

在本文中，我们将首先讨论部署在同一集群上的应用程序内部资源分配的一些问题。之后，我们将了解Apache Mesos如何在应用程序之间提供更好的资源利用率。

## 2. 共享集群

许多应用程序需要共享集群，总的来说，有两种常见的方法：

- 对集群进行静态分区并在每个分区上运行一个应用程序
- 为应用程序分配一组机器

虽然这些方法允许应用程序彼此独立运行，但并不能实现较高的资源利用率。

例如，假设一个应用程序**只运行很短一段时间，之后会进入非活动状态**。由于我们已为该应用程序分配了静态机器或分区，因此在非活动期间，我们**拥有未使用的资源**。

我们可以通过将非活动期间的空闲资源重新分配给其他应用程序来优化资源利用率。

Apache Mesos有助于应用程序之间的动态资源分配。

## 3. Apache Mesos

在我们上面讨论的两种集群共享方法中，应用程序只能感知到它们正在运行的特定分区或机器的资源。但是，Apache Mesos为应用程序提供了集群中所有资源的抽象视图。

我们稍后会看到，Mesos充当了机器和应用程序之间的接口，**它为应用程序提供集群中所有机器的可用资源**。它会**频繁更新这些信息，以包含已达到完成状态的应用程序所释放的资源**，这使得应用程序能够做出最佳决策，确定在哪台机器上执行哪个任务。

为了理解Mesos的工作原理，让我们看一下它的[架构](http://mesos.apache.org/documentation/latest/architecture/)：

![](/assets/images/2025/apache/apachemesos01.png)

此图是Mesos官方文档的一部分([来源](https://mesos.apache.org/assets/img/documentation/architecture3.jpg))，其中，Hadoop和MPI是共享集群的两个应用程序。

我们将在接下来的几节中讨论这里显示的每个组件。

### 3.1 Mesos主节点

Master是此设置中的核心组件，用于存储集群中资源的当前状态。此外，它还通过传递有关资源和任务等信息，充当代理和应用程序之间的协调器。

由于主节点的任何故障都会导致资源和任务状态丢失，因此我们将其部署为高可用性配置。如上图所示，Mesos部署了备用主节点守护进程和一个领导者守护进程，这些守护进程依靠Zookeeper在发生故障时恢复状态。

### 3.2 Mesos代理

Mesos集群必须在每台机器上运行代理，这些代理会**定期向主节点报告其资源**，并**接收应用程序已安排运行的任务**。即使计划任务完成或丢失，此循环也会重复。

我们将在以下部分中了解应用程序如何在这些代理上安排和执行任务。

### 3.3 Mesos框架

Mesos允许应用程序实现一个抽象组件，该组件与主节点交互，**获取集群中的可用资源**，并基于这些资源做出调度决策；这些组件被称为框架。

Mesos框架由两个子组件组成：

- Scheduler：使应用程序能够根据所有代理上的可用资源来调度任务
- Executor：在所有代理上运行，并包含在该代理上执行任何计划任务所需的所有信息

整个过程如下流程所示：

![](/assets/images/2025/apache/apachemesos02.png)

首先，代理向主节点报告其资源。此时，主节点将这些资源提供给所有注册的调度器，这个过程称为资源提供，我们将在下一节详细讨论。

然后，调度器会选择最佳代理，并通过主节点在其上执行各种任务。一旦执行器完成分配的任务，代理就会将其资源重新发布到主节点，主节点会为集群中的所有框架重复此资源共享过程。

Mesos允许应用程序使用各种编程语言实现自定义的调度器和执行器，Java实现的调度器必须实现Scheduler接口：

```java
public class HelloWorldScheduler implements Scheduler {

    @Override
    public void registered(SchedulerDriver schedulerDriver, Protos.FrameworkID frameworkID,
                           Protos.MasterInfo masterInfo) {
    }

    @Override
    public void reregistered(SchedulerDriver schedulerDriver, Protos.MasterInfo masterInfo) {
    }

    @Override
    public void resourceOffers(SchedulerDriver schedulerDriver, List<Offer> list) {
    }

    @Override
    public void offerRescinded(SchedulerDriver schedulerDriver, OfferID offerID) {
    }

    @Override
    public void statusUpdate(SchedulerDriver schedulerDriver, Protos.TaskStatus taskStatus) {
    }

    @Override
    public void frameworkMessage(SchedulerDriver schedulerDriver, Protos.ExecutorID executorID,
                                 Protos.SlaveID slaveID, byte[] bytes) {
    }

    @Override
    public void disconnected(SchedulerDriver schedulerDriver) {
    }

    @Override
    public void slaveLost(SchedulerDriver schedulerDriver, Protos.SlaveID slaveID) {
    }

    @Override
    public void executorLost(SchedulerDriver schedulerDriver, Protos.ExecutorID executorID,
                             Protos.SlaveID slaveID, int i) {
    }

    @Override
    public void error(SchedulerDriver schedulerDriver, String s) {
    }
}
```

可以看出，**它主要由各种回调方法组成，特别是用于与主机通信**。

类似地，执行器的实现必须实现Executor接口：

```java
public class HelloWorldExecutor implements Executor {
    @Override
    public void registered(ExecutorDriver driver, Protos.ExecutorInfo executorInfo,
                           Protos.FrameworkInfo frameworkInfo, Protos.SlaveInfo slaveInfo) {
    }

    @Override
    public void reregistered(ExecutorDriver driver, Protos.SlaveInfo slaveInfo) {
    }

    @Override
    public void disconnected(ExecutorDriver driver) {
    }

    @Override
    public void launchTask(ExecutorDriver driver, Protos.TaskInfo task) {
    }

    @Override
    public void killTask(ExecutorDriver driver, Protos.TaskID taskId) {
    }

    @Override
    public void frameworkMessage(ExecutorDriver driver, byte[] data) {
    }

    @Override
    public void shutdown(ExecutorDriver driver) {
    }
}
```

我们将在后面的部分看到调度程序和执行程序的操作版本。

## 4. 资源管理

### 4.1 资源提供

正如我们之前所讨论的，代理会将其资源信息发布给主服务器。反过来，主服务器将这些资源提供给集群中运行的框架，这个过程称为资源提供。

资源提供由两部分组成-资源和属性。

资源用于发布代理机器的内存、CPU、磁盘等硬件信息。

每个代理有五种预定义资源：

- cpu
- gpus
- mem
- disk
- ports

这些资源的值可以定义为以下三种类型之一：

- Scalar：用于使用浮点数表示数值信息，允许小数值，例如1.5G内存
- Range：用于表示标量值的范围–例如端口范围
- Set：用于表示多个文本值

默认情况下，Mesos代理会尝试从机器检测这些资源。

但是，在某些情况下，我们可以在代理上配置自定义资源，此类自定义资源的值也应属于上述任何一种类型。

例如，我们可以使用这些资源来启动我们的代理：

```text
--resources='cpus:24;gpus:2;mem:24576;disk:409600;ports:[21000-24000,30000-34000];bugs(debug_role):{a,b,c}'
```

可以看出，我们已经为代理配置了一些预定义资源和一个名为bug的set类型的自定义资源。

除了资源之外，代理还可以向主服务器发布键值属性，这些属性作为代理的附加元数据，帮助框架进行调度决策。

一个有用的例子是将代理添加到不同的机架或区域，然后在同一个机架或区域上安排各种任务以实现数据局部性：

```text
--attributes='rack:abc;zone:west;os:centos5;level:10;keys:[1000-1500]'
```

与资源类似，属性的值可以是标量、范围或文本类型。

### 4.2 资源角色

许多现代操作系统都支持多用户，同样，Mesos也支持同一集群中的多个用户，这些用户被称为角色，我们可以将每个角色视为集群中的资源使用者。

由此，Mesos代理可以基于不同的分配策略对不同角色下的资源进行划分。此外，框架可以在集群内订阅这些角色，并对不同角色下的资源进行细粒度的控制。

例如，**假设一个集群托管着为组织中不同用户提供服务的应用程序，通过将资源划分为角色，每个应用程序都可以彼此独立地工作**。

此外，框架可以使用这些角色来实现数据局部性。

例如，假设集群中有两个应用程序，分别为生产者和消费者。生产者将数据写入持久卷，消费者随后可以读取该卷，我们可以通过与生产者共享该卷来优化消费者应用程序。

由于Mesos允许多个应用程序订阅同一个角色，我们可以将持久卷与资源角色关联起来。此外，生产者和消费者的框架都将订阅相同的资源角色，因此，消费者应用程序现在可以在与生产者应用程序相同的卷上启动数据读取任务。

### 4.3 资源预留

现在可能会出现一个问题：Mesos如何将集群资源分配给不同的角色，Mesos通过预留来分配资源。

预订类型有两种：

- 静态预留
- 动态预留

静态预留类似于我们前面讨论过的代理启动时的资源分配：

```text
 --resources="cpus:4;mem:2048;cpus(tuyucheng):8;mem(tuyucheng):4096"
```

这里唯一的区别是，现在Mesos代理为**名为tuyucheng的角色预留了8个CPU和4096m内存**。

与静态预留不同，动态预留允许我们重新分配角色内的资源，Mesos允许框架和集群运维人员通过框架消息(作为资源提供的响应)或通过[HTTP端点](http://mesos.apache.org/documentation/latest/reservation/#examples)动态更改资源分配。

Mesos将所有未指定任何角色的资源分配给一个名为(\*)的默认角色，Master将这些资源提供给所有框架，无论它们是否订阅了该角色。

### 4.4 资源权重和配额

通常，Mesos主节点使用公平策略提供资源，它使用加权主导资源公平性(wDRF)来识别缺少资源的角色。然后，主节点会向订阅了这些角色的框架提供更多资源。

尽管在应用程序之间公平共享资源是Mesos的一个重要特性，但这并非总是必要的。假设一个集群托管着一些资源占用较低的应用程序，以及一些资源需求较高的应用程序，在这样的部署中，我们希望根据应用程序的性质来分配资源。

**Mesos允许框架通过订阅角色并增加该角色的权重来请求更多资源，因此，如果有两个角色，一个权重为1，另一个权重为2，Mesos将为第二个角色分配两倍的公平份额资源**。

与资源类似，我们可以通过[HTTP端点](http://mesos.apache.org/documentation/latest/weights/#operator-http-endpoint)配置权重。

除了确保为具有权重的角色公平分配资源之外，Mesos还确保**为角色分配最少的资源**。

Mesos允许我们为资源角色添加配额，配额指定了角色保证获得的最小资源量。

## 5. 实现框架

正如我们在上一节中讨论过的，Mesos允许应用程序使用自己选择的语言提供框架实现。在Java中，框架是通过主类(作为框架进程的入口点)以及前面讨论过的Scheduler和Executor的实现来实现的。

### 5.1 框架主类

在实现调度程序和执行器之前，我们首先要实现框架的入口点：

- 向主服务器注册
- 向代理提供执行器运行时信息
- 启动调度程序

我们首先为Mesos添加[Maven依赖](https://mvnrepository.com/artifact/org.apache.mesos/mesos)：

```xml
<dependency>
    <groupId>org.apache.mesos</groupId>
    <artifactId>mesos</artifactId>
    <version>1.11.0</version>
</dependency>
```

接下来，我们将为框架实现HelloWorldMain，我们要做的第一件事就是在Mesos代理上启动执行器进程：

```java
public static void main(String[] args) {
    String path = System.getProperty("user.dir") + "/target/libraries2-1.0.0-SNAPSHOT.jar";

    CommandInfo.URI uri = CommandInfo.URI.newBuilder().setValue(path).setExtract(false).build();

    String helloWorldCommand = "java -cp libraries2-1.0.0-SNAPSHOT.jar cn.tuyucheng.taketoday.mesos.executors.HelloWorldExecutor";
    CommandInfo commandInfoHelloWorld = CommandInfo.newBuilder()
            .setValue(helloWorldCommand)
            .addUris(uri)
            .build();

    ExecutorInfo executorHelloWorld = ExecutorInfo.newBuilder()
            .setExecutorId(Protos.ExecutorID.newBuilder()
                    .setValue("HelloWorldExecutor"))
            .setCommand(commandInfoHelloWorld)
            .setName("Hello World (Java)")
            .setSource("java")
            .build();
}
```

在这里，我们首先配置了执行器二进制文件的位置，Mesos代理会在框架注册后下载此二进制文件。接下来，代理会运行给定的命令来启动执行器进程。

接下来，我们将初始化框架并启动调度程序：

```java
FrameworkInfo.Builder frameworkBuilder = FrameworkInfo.newBuilder()
    .setFailoverTimeout(120000)
    .setUser("")
    .setName("Hello World Framework (Java)");
 
frameworkBuilder.setPrincipal("test-framework-java");
 
MesosSchedulerDriver driver = new MesosSchedulerDriver(new HelloWorldScheduler(), frameworkBuilder.build(), args[0]);
```

最后，**我们将启动MesosSchedulerDriver，它会向Master节点注册自身。为了成功注册，我们必须将Master节点的IP地址作为程序参数args[0\]传递给主类**：

```java
int status = driver.run() == Protos.Status.DRIVER_STOPPED ? 0 : 1;

driver.stop();

System.exit(status);
```

在上面显示的类中，CommandInfo、ExecutorInfo和FrameworkInfo都是主服务器和框架之间的[protobuf消息](https://www.baeldung.com/google-protocol-buffer)的Java表示。

### 5.2 实现调度器

自Mesos 1.0起，**我们可以从任何Java应用程序调用[HTTP端点](http://mesos.apache.org/documentation/latest/scheduler-http-api/)来向Mesos主节点发送和接收消息**，这些消息包括框架注册、资源提供和拒绝等。

**对于Mesos 0.28或更早版本，我们需要实现Scheduler接口**。

在大多数情况下，我们只关注Scheduler的resourceOffers方法，让我们看看调度程序如何接收资源并基于这些资源初始化任务。

首先，我们来看看调度程序如何为任务分配资源：

```java
@Override
public void resourceOffers(SchedulerDriver schedulerDriver, List<Offer> list) {
    for (Offer offer : list) {
        List<TaskInfo> tasks = new ArrayList<TaskInfo>();
        Protos.TaskID taskId = Protos.TaskID.newBuilder()
            .setValue(Integer.toString(launchedTasks++)).build();

        System.out.println("Launching printHelloWorld " + taskId.getValue() + " Hello World Java");

        Protos.Resource.Builder cpus = Protos.Resource.newBuilder()
            .setName("cpus")
            .setType(Protos.Value.Type.SCALAR)
            .setScalar(Protos.Value.Scalar.newBuilder()
                .setValue(1));

        Protos.Resource.Builder mem = Protos.Resource.newBuilder()
            .setName("mem")
            .setType(Protos.Value.Type.SCALAR)
            .setScalar(Protos.Value.Scalar.newBuilder()
                .setValue(128));
```

这里，我们为任务分配了1个CPU和128M内存。接下来，我们将使用SchedulerDriver在代理上启动该任务：

```java
        TaskInfo printHelloWorld = TaskInfo.newBuilder()
            .setName("printHelloWorld " + taskId.getValue())
            .setTaskId(taskId)
            .setSlaveId(offer.getSlaveId())
            .addResources(cpus)
            .addResources(mem)
            .setExecutor(ExecutorInfo.newBuilder(helloWorldExecutor))
            .build();

        List<OfferID> offerIDS = new ArrayList<>();
        offerIDS.add(offer.getId());

        tasks.add(printHelloWorld);

        schedulerDriver.launchTasks(offerIDS, tasks);
    }
}
```

另外，调度程序也经常需要拒绝资源请求。例如，如果调度程序由于资源不足而无法在代理上启动任务，则它必须立即拒绝该请求：

```java
schedulerDriver.declineOffer(offer.getId());
```

### 5.3 实现执行器

正如我们之前讨论的，框架的执行器组件负责在Mesos代理上执行应用程序任务。

在Mesos 1.0中，我们使用HTTP端点来实现调度器(Scheduler)。同样，我们也可以使用[HTTP端点](http://mesos.apache.org/documentation/latest/executor-http-api/)来实现执行器(Executor)。

在前面的部分中，我们讨论了框架如何配置代理来启动执行器进程：

```shell
java -cp libraries2-1.0.0-SNAPSHOT.jar cn.tuyucheng.taketoday.mesos.executors.HelloWorldExecutor
```

值得注意的是，**此命令将HelloWorldExecutor视为主类**，我们将实现此主方法来**初始化MesosExecutorDriver**，该驱动程序连接到Mesos代理来接收任务并共享其他信息(例如任务状态)：

```java
public class HelloWorldExecutor implements Executor {
    public static void main(String[] args) {
        MesosExecutorDriver driver = new MesosExecutorDriver(new HelloWorldExecutor());
        System.exit(driver.run() == Protos.Status.DRIVER_STOPPED ? 0 : 1);
    }
}
```

现在要做的最后一件事是从框架接收任务并在代理上启动它们，启动任何任务的信息都包含在HelloWorldExecutor中：

```java
public void launchTask(ExecutorDriver driver, TaskInfo task) {
    Protos.TaskStatus status = Protos.TaskStatus.newBuilder()
            .setTaskId(task.getTaskId())
            .setState(Protos.TaskState.TASK_RUNNING)
            .build();
    driver.sendStatusUpdate(status);

    System.out.println("Execute Task!!!");

    status = Protos.TaskStatus.newBuilder()
            .setTaskId(task.getTaskId())
            .setState(Protos.TaskState.TASK_FINISHED)
            .build();
    driver.sendStatusUpdate(status);
}
```

当然，这只是一个简单的实现，但它解释了执行器如何在每个阶段与主服务器共享任务状态，然后在发送完成状态之前执行任务。

在某些情况下，执行器还可以将数据发送回调度器：

```java
String myStatus = "Hello Framework";
driver.sendFrameworkMessage(myStatus.getBytes());
```

## 6. 总结

在本文中，我们简要讨论了在同一集群中运行的应用程序之间的资源共享，我们还讨论了Apache Mesos如何通过对CPU和内存等集群资源的抽象视图来帮助应用程序实现最大利用率。

之后，我们讨论了基于各种公平策略和角色在应用程序之间动态分配资源，Mesos允许应用程序根据集群中Mesos代理提供的资源来做出调度决策。

最后，我们看到了Mesos框架的Java实现。