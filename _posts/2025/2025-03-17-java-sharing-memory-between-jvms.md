---
layout: post
title:  在JVM之间共享内存
category: java
copyright: java
excerpt: Java Sun
---

## 1. 简介

在本教程中，我们将展示如何在同一台机器上运行的两个或多个JVM之间共享内存。此功能可实现非常快速的进程间通信，因为我们无需任何I/O操作即可移动数据块。

## 2. 共享内存如何工作？

在任何现代操作系统中运行的进程都会获得所谓的虚拟内存空间，**我们之所以称之为虚拟，是因为尽管它看起来像一个大而连续的私有可寻址内存空间，但实际上它是由遍布物理RAM的页面组成的**。在这里，页面只是操作系统的俚语，指的是连续内存块，其大小范围取决于所使用的特定CPU架构。对于x86-84，页面可以小到4KB，大到1GB。

在给定时间内，只有一小部分虚拟空间实际映射到物理页面。随着时间的推移，进程开始消耗更多内存来执行其任务，操作系统开始分配更多物理页面并将它们映射到虚拟空间。当对内存的需求超过物理可用内存时，操作系统将开始将当时未使用的页面换出到辅助存储，为请求腾出空间。

**共享内存块的行为与常规内存相同，但与常规内存不同的是，它不属于单个进程**。当某个进程更改此块中任何字节的内容时，任何其他有权访问同一共享内存的进程都会立即“看到”此更改。

以下是共享内存的常见用途列表：

- 调试器(有没有想过调试器如何检查另一个进程中的变量？)
- 进程间通信
- 进程之间共享的只读内容(例如：动态库代码)
- 各种黑客攻击

## 3. 共享内存和内存映射文件

**顾名思义，内存映射文件是一种常规文件，其内容直接映射到进程虚拟内存中的连续区域**。这意味着我们可以读取和/或更改其内容，而无需明确使用I/O操作。操作系统将检测对映射区域的任何写入，并安排后台I/O操作来保存修改后的数据。

由于无法保证此后台操作何时发生，因此操作系统还提供了一个系统调用来刷新任何待处理的更改。这对于数据库重做日志等用例很重要，但对于我们的进程间通信(简称IPC)场景则不需要。

内存映射文件通常由数据库服务器使用，以实现高吞吐量I/O操作，但我们也可以使用它们来引导基于共享内存的IPC机制。**基本思想是，所有需要共享数据的进程都映射同一个文件，这样，它们现在就拥有了一个共享内存区域**。

## 4. 在Java中创建内存映射文件

在Java中，我们使用[FileChannel](https://www.baeldung.com/java-filechannel)的map()方法将文件的一个区域映射到内存中，该方法返回一个[MappedByteBuffer](https://www.baeldung.com/java-mapped-byte-buffer)，使我们能够访问其内容：

```java
MappedByteBuffer createSharedMemory(String path, long size) {
    try (FileChannel fc = (FileChannel)Files.newByteChannel(new File(path).toPath(), EnumSet.of(
        StandardOpenOption.CREATE,
        StandardOpenOption.SPARSE,
        StandardOpenOption.WRITE,
        StandardOpenOption.READ))) {

        return fc.map(FileChannel.MapMode.READ_WRITE, 0, size);
    }
    catch( IOException ioe) {
        throw new RuntimeException(ioe);
    }
}
```

这里使用SPARSE选项非常重要，**只要底层操作系统和文件系统支持，我们就可以映射相当大的内存区域，而无需实际占用磁盘空间**。

现在，让我们创建一个简单的演示应用程序。Producer应用程序将分配一个足够大的共享内存，以容纳64KB的数据和一个SHA1哈希值(20字节)。接下来，它将启动一个循环，用随机数据填充缓冲区，然后用其SHA1哈希值填充。我们将连续重复此操作30秒，然后退出：

```java
// ... SHA1 digest initialization omitted

MappedByteBuffer shm = createSharedMemory("some_path.dat", 64 * 1024 + 20);
Random rnd = new Random();

long start = System.currentTimeMillis();
long iterations = 0;
int capacity = shm.capacity();
System.out.println("Starting producer iterations...");
while(System.currentTimeMillis() - start < 30000) {

    for (int i = 0; i < capacity - hashLen; i++) {
        byte value = (byte) (rnd.nextInt(256) & 0x00ff);
        digest.update(value);
        shm.put(i, value);
    }

    // Write hash at the end
    byte[] hash = digest.digest();
    shm.put(capacity - hashLen, hash);
    iterations++;
}

System.out.printf("%d iterations run\n", iterations);
```

为了测试我们确实可以共享内存，我们还将创建一个Consumer应用，它将读取缓冲区的内容，计算其哈希值，并将其与Producer生成的哈希值进行比较。我们将重复此过程30秒，在每次迭代中，还将计算缓冲区内容的哈希值并将其与缓冲区末尾的哈希值进行比较：

```java
// ... digest initialization omitted

MappedByteBuffer shm = createSharedMemory("some_path.dat", 64 * 1024 + 20);
long start = System.currentTimeMillis();
long iterations = 0;
int capacity = shm.capacity();
System.out.println("Starting consumer iterations...");

long matchCount = 0;
long mismatchCount = 0;
byte[] expectedHash = new byte[hashLen];

while (System.currentTimeMillis() - start < 30000) {

    for (int i = 0; i < capacity - 20; i++) {
        byte value = shm.get(i);
        digest.update(value);
    }

    byte[] hash = digest.digest();
    shm.get(capacity - hashLen, expectedHash);

    if (Arrays.equals(hash, expectedHash)) {
        matchCount++;
    } else {
        mismatchCount++;
    }
    iterations++;
}

System.out.printf("%d iterations run. matches=%d, mismatches=%d\n", iterations, matchCount, mismatchCount);
```

为了测试我们的内存共享方案，让我们同时启动两个程序。这是它们在3Ghz、四核Intel I7机器上运行时的输出：

```shell
# Producer output
Starting producer iterations...
11722 iterations run


# Consumer output
Starting consumer iterations...
18893 iterations run. matches=11714, mismatches=7179
```

我们可以看到，在很多情况下，消费者检测到的预期计算值是不同的。欢迎来到并发问题的奇妙世界！

## 5. 同步共享内存访问

**我们看到的问题的根本原因是我们需要同步对共享内存缓冲区的访问**，消费者必须等待生产者完成写入哈希后才能开始读取数据。另一方面，生产者也必须等待消费者完成使用数据后才能再次写入数据。

对于常规的多线程应用程序来说，解决这个问题并不是什么大问题。标准库提供了几个同步原语，使我们能够控制在给定时间内谁可以写入共享内存。

但是，我们的情况是多JVM场景，因此这些标准方法都不适用。那么，我们该怎么办呢？**简而言之，我们必须作弊**。我们可以求助于特定于操作系统的机制，例如信号量，但这会妨碍我们应用程序的可移植性。此外，这意味着使用JNI或JNA，这也使事情变得复杂。

进入[Unsafe](https://www.baeldung.com/java-unsafe)，尽管它的名字有点吓人，但这个标准库类提供了我们实现简单锁定机制所需的一切：compareAndSwapInt()方法。

此方法实现了一个原子测试和设置原语，它接收四个参数。**虽然文档中没有明确说明，但它不仅可以针对Java对象，还可以针对原始内存地址**。对于后者，我们在第一个参数中传递null，这使得它将offset参数视为虚拟内存地址。

当我们调用此方法时，它会首先检查目标地址处的值并将其与预期值进行比较。如果它们相等，则它会将位置的内容修改为新值并返回true表示成功。如果位置处的值与预期值不同，则不会发生任何事情，并且该方法返回false。

**更重要的是，即使在多核架构中，该原子操作也能保证工作，这对于同步多个执行线程至关重要**。

让我们创建一个SpinLock类，利用这个方法来实现一个简单的锁机制：

```java
//... package and imports omitted

public class SpinLock {
    private static final Unsafe unsafe;

    // ... unsafe initialization omitted
    private final long addr;

    public SpinLock(long addr) {
        this.addr = addr;
    }

    public boolean tryLock(long maxWait) {
        long deadline = System.currentTimeMillis() + maxWait;
        while (System.currentTimeMillis() < deadline ) {
            if (unsafe.compareAndSwapInt(null, addr, 0, 1)) {
                return true;
            }
        }
        return false;
    }

    public void unlock() {
        unsafe.putInt(addr, 0);
    }
}
```

该实现缺少关键功能，例如在释放锁之前检查它是否拥有锁，但这足以满足我们的目的。

好的，那么我们如何获取用于存储锁状态的内存地址呢？这必须是共享内存缓冲区内的地址，以便两个进程都可以使用它，但MappedByteBuffer类不会公开实际的内存地址。

检查map()返回的对象，我们可以看到它是一个DirectByteBuffer。**这个类有一个公共方法，称为address()，它返回的正是我们想要的。不幸的是，这个类是包私有的，所以我们不能使用简单的强制转换来访问这个方法**。

为了绕过这个限制，我们再稍微作弊一下，使用反射来调用这个方法：

```java
private static long getBufferAddress(MappedByteBuffer shm) {
    try {
        Class<?> cls = shm.getClass();
        Method maddr = cls.getMethod("address");
        maddr.setAccessible(true);
        Long addr = (Long) maddr.invoke(shm);
        if (addr == null) {
            throw new RuntimeException("Unable to retrieve buffer's address");
        }
        return addr;
    } catch (NoSuchMethodException | InvocationTargetException | IllegalAccessException ex) {
        throw new RuntimeException(ex);
    }
}
```

在这里，我们使用setAccessible()使address()方法可通过方法句柄调用。但是，请注意，从Java 17开始，除非我们明确使用运行时–add-opens标志，否则此技术将不起作用。

## 6. 为生产者和消费者添加同步

现在我们有了锁机制，让我们首先将其应用于生产者。出于本演示的目的，我们假设生产者将始终在消费者之前启动。我们需要它，以便我们可以初始化缓冲区，清除其内容，包括我们将与SpinLock一起使用的区域：

```java
public static void main(String[] args) throws Exception {

    // ... digest initialization omitted
    MappedByteBuffer shm = createSharedMemory("some_path.dat", 64 * 1024 + 20);

    // Cleanup lock area 
    shm.putInt(0, 0);

    long addr = getBufferAddress(shm);
    System.out.println("Starting producer iterations...");

    long start = System.currentTimeMillis();
    long iterations = 0;
    Random rnd = new Random();
    int capacity = shm.capacity();
    SpinLock lock = new SpinLock(addr);
    while(System.currentTimeMillis() - start < 30000) {

        if (!lock.tryLock(5000)) {
            throw new RuntimeException("Unable to acquire lock");
        }

        try {
            // Skip the first 4 bytes, as they're used by the lock
            for (int i = 4; i < capacity - hashLen; i++) {
                byte value = (byte) (rnd.nextInt(256) & 0x00ff);
                digest.update(value);
                shm.put(i, value);
            }

            // Write hash at the end
            byte[] hash = digest.digest();
            shm.put(capacity - hashLen, hash);
            iterations++;
        }
        finally {
            lock.unlock();
        }
    }
    System.out.printf("%d iterations run\n", iterations);
}
```

与非同步版本相比，只有细微的变化：

- 检索与MappedByteBuffer关联的内存地址 
- 使用此地址创建一个SpinLock实例，该锁使用int，因此它将占用缓冲区的4个初始字节
- 使用SpinLock实例来保护用随机数据及其哈希填充缓冲区的代码

现在，让我们将类似的变化应用到消费者端：

```java
private static void main(String[] args) throws Exception {

    // ... digest initialization omitted
    MappedByteBuffer shm = createSharedMemory("some_path.dat", 64 * 1024 + 20);
    long addr = getBufferAddress(shm);

    System.out.println("Starting consumer iterations...");

    Random rnd = new Random();
    long start = System.currentTimeMillis();
    long iterations = 0;
    int capacity = shm.capacity();

    long matchCount = 0;
    long mismatchCount = 0;
    byte[] expectedHash = new byte[hashLen];
    SpinLock lock = new SpinLock(addr);
    while (System.currentTimeMillis() - start < 30000) {

        if (!lock.tryLock(5000)) {
            throw new RuntimeException("Unable to acquire lock");
        }

        try {
            for (int i = 4; i < capacity - hashLen; i++) {
                byte value = shm.get(i);
                digest.update(value);
            }

            byte[] hash = digest.digest();
            shm.get(capacity - hashLen, expectedHash);

            if (Arrays.equals(hash, expectedHash)) {
                matchCount++;
            } else {
                mismatchCount++;
            }

            iterations++;
        } finally {
            lock.unlock();
        }
    }

    System.out.printf("%d iterations run. matches=%d, mismatches=%d\n", iterations, matchCount, mismatchCount);
}
```

经过这些更改，我们现在可以运行双方并将它们与之前的结果进行比较：

```text
# Producer output
Starting producer iterations...
8543 iterations run

# Consumer output
Starting consumer iterations...
8607 iterations run. matches=8607, mismatches=0
```

**正如预期的那样，报告的迭代次数与非同步版本相比会更低**。主要原因是我们大部分时间都花在代码的关键部分中持有锁，无论哪个程序持有锁，都会阻止另一方执行任何操作。

如果我们比较第一种情况报告的平均迭代次数，它将与我们这次得到的迭代次数总和大致相同，这表明锁机制本身增加的开销很小。

## 6. 总结

在本教程中，我们探索了如何在同一台机器上运行的两个JVM之间共享内存区域。我们可以使用此处介绍的技术作为高吞吐量、低延迟进程间通信库的基础。