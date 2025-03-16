---
layout: post
title:  java.lang.Process API指南
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 简介

在本教程中，我们将深入了解Process API。

为了更深入地了解如何使用Process执行Shell命令，我们可以参考[之前的教程](https://www.baeldung.com/run-shell-command-in-java)。

进程指的是一个正在执行的应用程序，Process类提供与这些进程交互的方法，包括提取输出、执行输入、监视生命周期、检查退出状态以及销毁(终止)进程。

## 2. 使用Process类编译和运行Java程序

首先，让我们看一个借助Process API编译并运行另一个Java程序的示例：

```java
@Test
public void whenExecutedFromAnotherProgram_thenSourceProgramOutput3() throws IOException {
    Process process = Runtime.getRuntime().exec("javac -cp src src\\main\\java\\cn\\tuyucheng\\taketoday\\java9\\process\\OutputStreamExample.java");
    process = Runtime.getRuntime().exec("java -cp src/main/java cn.tuyucheng.taketoday.java9.process.OutputStreamExample");
    BufferedReader output = new BufferedReader(new InputStreamReader(process.getInputStream()));
    int value = Integer.parseInt(output.readLine());

    assertEquals(3, value);
}
```

因此，在现有Java代码中执行Java代码的应用实际上是无限的。

## 3. 创建进程

一般来说，我们的Java应用程序可以调用在我们的计算机系统内运行的任何应用程序，但受到操作系统的限制。

因此我们可以执行应用程序。那么，让我们看看利用Process API可以运行哪些不同的用例。

简而言之，ProcessBuilder类允许我们在应用程序中创建子进程。

让我们看一个打开基于Windows的记事本应用程序的演示：

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
```

## 4. 销毁进程

Process类还为我们提供了销毁子进程或进程的方法，**但是，应用程序如何被销毁与平台相关**。

接下来，让我们用实际的例子来举例说明不同的用例。

### 4.1 通过引用销毁进程

假设我们正在使用Windows操作系统，并且想要生成记事本应用程序并销毁它。

和以前一样，我们可以使用ProcessBuilder类和start()方法创建记事本应用程序的实例。然后，我们可以在Process对象上调用destroy()方法。

### 4.2 通过ID销毁进程

通常，我们可以终止操作系统中正在运行的进程，而这些进程可能不是由我们的应用程序创建的。**执行此操作时要小心谨慎，因为在不知情的情况下破坏关键进程可能会破坏操作系统的稳定性**。

首先，我们需要通过检查任务管理器并找出pid来找出当前正在运行的进程的进程ID。

让我们看一个例子：

```java
long pid = /* PID to kill */;
Optional<ProcessHandle> optionalProcessHandle = ProcessHandle.of(pid);
optionalProcessHandle.ifPresent(processHandle -> processHandle.destroy());
```

### 4.3 强制销毁进程

当我们执行destroy()方法时，它会终止子进程，就像我们在文章前面看到的那样。**在destroy()不起作用的情况下，我们可以选择destroyForcibly()**。

**值得注意的是，我们应该始终先使用destroy()方法**。随后，我们可以使用isAlive()方法对子进程进行快速检查。

如果返回true则执行destroyForcibly()：

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
process.destroy();
if (process.isAlive()) {
    process.destroyForcibly();
}
```

## 5. 等待进程完成

我们还有两个重载方法，通过它们我们可以确保等待一个进程完成。

### 5.1 waitfor() 

**简而言之，当此方法执行时，它会将当前执行进程线程置于阻塞等待状态，直到子进程终止**。

那么，让我们看看它的实际效果：

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
assertThat(process.waitFor() >= 0);
```

可以看到，当前线程等待子进程线程结束，一旦子进程结束，当前线程才会继续执行。

### 5.2 waitfor(long timeOut, TimeUnit time)

通常，执行此方法会使当前执行进程线程处于阻塞等待状态，直到子进程终止或超时。

那么，让我们在实践中看看这一点：

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
assertFalse(process.waitFor(1, TimeUnit.SECONDS));
```

从上面的例子我们可以看出，当前线程要继续执行，它将继续等待子进程线程结束或指定的时间间隔已经过去。

当执行此方法时，如果子进程已退出，则返回布尔值true；如果在子进程退出之前等待时间已经过去，则返回布尔值false。

## 6. exitValue() 

当运行此方法时，当前线程将不会等待子进程终止或销毁，但是，如果子进程未终止，它将抛出IllegalThreadStateException。

**另一种方法是，如果子进程已成功终止，则将导致进程的退出值**，它可以是任何可能的正整数。

那么，让我们看一个当子进程成功终止时exitValue()方法返回正整数的例子：

```java
@Test
public void givenSubProcess_whenCurrentThreadWillNotWaitIndefinitelyforSubProcessToEnd_thenProcessExitValueReturnsGrt0() throws IOException {
    ProcessBuilder builder = new ProcessBuilder("notepad.exe");
    Process process = builder.start();
    assertThat(process.exitValue() >= 0);
}
```

## 7. isAlive() 

通常，当我们想要执行业务处理时，无论进程是否处于活动状态都是主观的，我们可以执行快速检查以查找进程是否处于活动状态并返回布尔值。

让我们看一个简单的例子：

```java
ProcessBuilder builder = new ProcessBuilder("notepad.exe");
Process process = builder.start();
Thread.sleep(10000);
process.destroy();
assertTrue(process.isAlive());
```

## 8. 处理进程流

默认情况下，创建的子进程没有自己的终端或控制台，其所有标准I/O(即stdin、stdout、stderr)操作都将发送到父进程。因此，父进程可以使用这些流向子进程提供输入并从子进程获取输出。

因此，这给了我们极大的灵活性，因为它使我们能够控制子进程的输入/输出。

### 8.1 getErrorStream() 

有趣的是，我们可以获取子进程产生的错误并在此基础上执行业务处理。

之后，我们可以根据自己的需求执行具体的业务处理检查。

我们来看一个例子：

```java
@Test
public void givenSubProcess_whenEncounterError_thenErrorStreamNotNull() throws IOException {
    Process process = Runtime.getRuntime().exec("javac -cp src src\\main\\java\\cn\\tuyucheng\\taketoday\\java9\\process\\ProcessCompilationError.java");
    BufferedReader error = new BufferedReader(new InputStreamReader(process.getErrorStream()));
    String errorString = error.readLine();
    assertNotNull(errorString);
}
```

### 8.2 getInputStream() 

我们还可以获取子进程生成的输出并在父进程中使用它，从而允许在进程之间共享信息：

```java
@Test
public void givenSourceProgram_whenReadingInputStream_thenFirstLineEquals3() throws IOException {
    Process process = Runtime.getRuntime().exec("javac -cp src src\\main\\java\\cn\\tuyucheng\\taketoday\\java9\\process\\OutputStreamExample.java");
    process = Runtime.getRuntime().exec("java -cp  src/main/java cn.tuyucheng.taketoday.java9.process.OutputStreamExample");
    BufferedReader output = new BufferedReader(new InputStreamReader(process.getInputStream()));
    int value = Integer.parseInt(output.readLine());

    assertEquals(3, value);
}
```

### 8.3 getOutputStream() 

我们可以从父进程向子进程发送输入：

```java
Writer w = new OutputStreamWriter(process.getOutputStream(), "UTF-8");
w.write("send to childn");
```

### 8.4 过滤进程流

这是与选择性运行的进程进行交互的一个非常有效的用例。

Process类为我们提供了根据某个谓词有选择地过滤正在运行的进程的功能。

之后我们就可以针对这个选择性进程集进行业务操作了：

```java
@Test
public void givenRunningProcesses_whenFilterOnProcessIdRange_thenGetSelectedProcessPid() {
    assertThat(((int) ProcessHandle.allProcesses()
        .filter(ph -> (ph.pid() > 10000 && ph.pid() < 50000))
        .count()) > 0);
}
```

## 9. Java 9改进

Java 9引入了获取有关当前进程和衍生进程信息的新选项和方法，让我们深入了解并详细探索每个功能。

### 9.1 当前Java进程信息

我们现在可以通过java.lang.ProcessHandle.Info API获取有关该进程的大量信息：

- 用于启动进程的命令
- 命令的参数
- 进程启动的时间
- 该应用和创建该应用的用户所花费的总时间

例如，我们可以这样做：

```java
private static void infoOfCurrentProcess() {
    ProcessHandle processHandle = ProcessHandle.current();
    ProcessHandle.Info processInfo = processHandle.info();

    log.info("PID: " + processHandle.pid());
    log.info("Arguments: " + processInfo.arguments());
    log.info("Command: " + processInfo.command());
    log.info("Instant: " + processInfo.startInstant());
    log.info("Total CPU duration: " + processInfo.totalCpuDuration());
    log.info("User: " + processInfo.user());
}
```

需要注意的是，java.lang.ProcessHandle.Info是另一个接口java.lang.ProcessHandle中定义的公共接口，JDK提供程序(Oracle JDK、Open JDK、Zulu或其他)应以返回进程相关信息的方式实现这些接口。

输出取决于操作系统和Java版本，以下是输出示例：

```text
16:31:24.784 [main] INFO  c.t.t.j.process.ProcessAPIEnhancements - PID: 22640
16:31:24.790 [main] INFO  c.t.t.j.process.ProcessAPIEnhancements - Arguments: Optional[[Ljava.lang.String;@2a17b7b6]
16:31:24.791 [main] INFO  c.t.t.j.process.ProcessAPIEnhancements - Command: Optional[/Library/Java/JavaVirtualMachines/jdk-13.0.1.jdk/Contents/Home/bin/java]
16:31:24.795 [main] INFO  c.t.t.j.process.ProcessAPIEnhancements - Instant: Optional[2021-08-31T14:31:23.870Z]
16:31:24.795 [main] INFO  c.t.t.j.process.ProcessAPIEnhancements - Total CPU duration: Optional[PT0.818115S]
16:31:24.796 [main] INFO  c.t.t.j.process.ProcessAPIEnhancements - User: Optional[username]
```

### 9.2 生成进程信息

还可以获取新生成的进程的进程信息，在这种情况下，生成进程并获取java.lang.Process实例后，我们对其调用toHandle()方法以获取java.lang.ProcessHandle实例。

其余细节与上面部分相同：

```java
String javaCmd = ProcessUtils.getJavaCmd().getAbsolutePath();
ProcessBuilder processBuilder = new ProcessBuilder(javaCmd, "-version");
Process process = processBuilder.inheritIO().start();
ProcessHandle processHandle = process.toHandle();
```

### 9.3 枚举系统中的活动进程

我们可以列出系统中当前进程可见的所有进程，返回的列表是所调用API的快照，因此某些进程可能在拍摄快照后终止，或者添加了新进程。

为了做到这一点，我们可以使用java.lang.ProcessHandle接口中提供的静态方法allProcesses()，它返回一个ProcessHandle流：

```java
private static void infoOfLiveProcesses() {
    Stream<ProcessHandle> liveProcesses = ProcessHandle.allProcesses();
    liveProcesses.filter(ProcessHandle::isAlive)
        .forEach(ph -> {
            log.info("PID: " + ph.pid());
            log.info("Instance: " + ph.info().startInstant());
            log.info("User: " + ph.info().user());
        });
}
```

### 9.4 枚举子进程

有两种方法可以实现此目的：

- 获取当前进程的直接子进程
- 获取当前进程的所有后代

前者通过使用方法children()实现，后者通过使用方法descendants()实现：

```java
private static void infoOfChildProcess() throws IOException {
    int childProcessCount = 5;
    for (int i = 0; i < childProcessCount; i++) {
        String javaCmd = ProcessUtils.getJavaCmd()
            .getAbsolutePath();
        ProcessBuilder processBuilder = new ProcessBuilder(javaCmd, "-version");
        processBuilder.inheritIO().start();
    }

    Stream<ProcessHandle> children = ProcessHandle.current()
        .children();
    children.filter(ProcessHandle::isAlive)
        .forEach(ph -> log.info("PID: {}, Cmd: {}", ph.pid(), ph.info()
            .command()));
    Stream<ProcessHandle> descendants = ProcessHandle.current()
        .descendants();
    descendants.filter(ProcessHandle::isAlive)
        .forEach(ph -> log.info("PID: {}, Cmd: {}", ph.pid(), ph.info()
            .command()));
}
```

### 9.5 进程终止时触发相关操作

我们可能想在进程终止时运行某些操作，这可以通过使用java.lang.ProcessHandle接口中的onExit()方法来实现。**该方法返回一个[CompletableFuture](https://www.baeldung.com/java-completablefuture)，它提供了在CompletableFuture完成时触发依赖操作的能力**。

这里，CompletableFuture表示该进程已完成，但进程是否成功完成并不重要，我们调用CompletableFuture上的get()方法来等待其完成：

```java
private static void infoOfExitCallback() throws IOException, InterruptedException, ExecutionException {
    String javaCmd = ProcessUtils.getJavaCmd()
        .getAbsolutePath();
    ProcessBuilder processBuilder = new ProcessBuilder(javaCmd, "-version");
    Process process = processBuilder.inheritIO()
        .start();
    ProcessHandle processHandle = process.toHandle();

    log.info("PID: {} has started", processHandle.pid());
    CompletableFuture onProcessExit = processHandle.onExit();
    onProcessExit.get();
    log.info("Alive: " + processHandle.isAlive());
    onProcessExit.thenAccept(ph -> log.info("PID: {} has stopped", ph.pid()));
}
```

onExit()方法也可在java.lang.Process接口中使用。

## 10. 总结

在本文中，我们介绍了Java中Process API的大部分重要特性，在此过程中，我们还讨论了Java 9中引入的新改进。