---
layout: post
title:  在Java中检查目录的写入权限
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在Java中，与[文件系统](https://www.baeldung.com/linux/filesystems)交互以进行读取、写入或其他操作是一项常见任务，管理文件和目录通常涉及检查其权限。

在本教程中，我们将探讨在Java中检查目录写入权限的各种方法。

## 2. 测试设置

我们需要两个具有读写权限的目录来测试我们的逻辑，我们可以将此设置作为JUnit中[@BeforeEach](https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall)生命周期方法的一部分进行：

```java
@BeforeEach
void create() throws IOException {
    Files.createDirectory(Paths.get(writeDirPath));
    Set<PosixFilePermission> permissions = new HashSet<>();
    permissions.add(PosixFilePermission.OWNER_WRITE);
    Files.setPosixFilePermissions(Paths.get(writeDirPath), permissions);

    Files.createDirectory(Paths.get(readDirPath));
    permissions = new HashSet<>();
    permissions.add(PosixFilePermission.OWNER_READ);
    Files.setPosixFilePermissions(Paths.get(readDirPath), permissions);
}
```

类似地，为了清理，我们可以作为@AfterEach生命周期方法的一部分删除这些目录：

```java
@AfterEach
void destroy() throws IOException {
    Files.delete(Paths.get(readDirPath));
    Files.delete(Paths.get(writeDirPath));
}
```

## 3. 使用Java IO

[Java IO](https://www.baeldung.com/java-io)是Java编程语言的核心部分，它提供了一个框架，用于从各种来源(例如文件、网络连接、内存缓冲区等)读取和写入数据。为了处理文件，它包括File类，该类提供开箱即用的方法，例如canRead()、canWrite()和canExecute()，这些方法有助于检查文件的权限：

```java
boolean usingJavaIO(String path){
    File dir = new File(path);
    return dir.exists() && dir.canWrite();
}
```

在上面的逻辑中，我们使用canWrite()来检查目录的写权限。

我们可以写一个简单的测试来验证结果：

```java
@Test
void givenDirectory_whenUsingJavaIO_thenReturnsPermission(){
    CheckWritePermission checkWritePermission = new CheckWritePermission();
    assertTrue(checkWritePermission.usingJavaIO(writeDirPath));
    assertFalse(checkWritePermission.usingJavaIO(readDirPath));
}
```

## 4. 使用Java NIO

[Java NIO](https://www.baeldung.com/java-io-vs-nio)(新I/O)是标准Java IO API的替代方案，提供非阻塞IO操作。Java NIO旨在提高Java应用程序中IO操作的性能和可扩展性。

要使用NIO检查文件权限，我们可以使用java.nio.file包中的Files类。Files类提供了各种辅助方法，例如isReadable()、isWritable()和isExecutable()，这些方法有助于检查文件权限：

```java
boolean usingJavaNIOWithFilesPackage(String path){
    Path dirPath = Paths.get(path);
    return Files.isDirectory(dirPath) && Files.isWritable(dirPath);
}
```

在上面的逻辑中，我们使用isWritable()方法来检查目录的写权限。

我们可以写一个简单的测试来验证结果：

```java
@Test
void givenDirectory_whenUsingJavaNIOWithFilesPackage_thenReturnsPermission(){
    CheckWritePermission checkWritePermission = new CheckWritePermission();
    assertTrue(checkWritePermission.usingJavaNIOWithFilesPackage(writeDirPath));
    assertFalse(checkWritePermission.usingJavaNIOWithFilesPackage(readDirPath));
}
```

### 4.1 POSIX权限

POSIX文件权限是Unix和类Unix操作系统中常见的权限系统，它基于3种用户的3种访问权限：所有者、组和其他用户。Files类提供了getPosixFilePermissions()方法，该方法返回一组表示权限的PosixFilePermission枚举值：

```java
boolean usingJavaNIOWithPOSIX(String path) throws IOException {
    Path dirPath = Paths.get(path);
    Set<PosixFilePermission> permissions = Files.getPosixFilePermissions(dirPath);
    return permissions.contains(PosixFilePermission.OWNER_WRITE);
}
```

在上面的逻辑中，我们返回目录的POSIX权限并检查是否包含写权限。

我们可以写一个简单的测试来验证结果：

```java
@Test
void givenDirectory_whenUsingJavaNIOWithPOSIX_thenReturnsPermission() throws IOException {
    CheckWritePermission checkWritePermission = new CheckWritePermission();
    assertTrue(checkWritePermission.usingJavaNIOWithPOSIX(writeDirPath));
    assertFalse(checkWritePermission.usingJavaNIOWithPOSIX(readDirPath));
}
```

## 5. 总结

在本文中，我们探讨了3种不同的方法来在Java中检查目录的写入权限。Java IO提供了简单的方法来检查权限，而Java NIO在检查权限并以POSIX格式获取权限方面提供了更大的灵活性。