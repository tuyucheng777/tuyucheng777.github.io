---
layout: post
title:  使用Java列出远程服务器上的所有文件
category: libraries
copyright: libraries
excerpt: JSch、SSHJ、Apache MinaSSHD
---

## 1. 概述

与远程服务器交互是现代软件开发和系统管理中的常见任务，**使用SSH客户端与远程服务器进行编程交互允许部署应用程序、管理配置、传输文件等**。[JSch](https://www.baeldung.com/java-ssh-connection)、[Apache MinaSSHD](https://www.baeldung.com/java-ssh-connection#apache-mina-sshd)和[SSHJ](https://www.baeldung.com/java-sshj-ssh-library)库是Java中流行的SSH客户端。

在本教程中，我们将学习如何使用JSch、Apache Mina SSHD和SSHJ库与远程服务器交互。此外，我们将了解如何使用私钥建立[与远程服务器的连接](https://www.baeldung.com/java-ssh-connection)，并列出服务器中特定目录中的所有文件夹。

## 2. 使用JSch库

JSch(Java安全通道)库提供用于与[SSH](https://www.baeldung.com/cs/ssh-intro)服务器建立连接的类，它是SSH2的Java实现。

首先，让我们将[JSch依赖](https://mvnrepository.com/artifact/com.github.mwiede/jsch)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.github.mwiede</groupId>
    <artifactId>jsch</artifactId>
    <version>0.2.18</version>
</dependency>
```

接下来，让我们定义连接详细信息以建立与远程服务器的连接：

```java
private static final String HOST = "HOST_NAME";
private static final String USER = "USERNAME";
private static final String PRIVATE_KEY = "PRIVATE_KEY";
private static final int PORT = 22;
private static final String REMOTE_DIR = "REMOTE_DIR";
```

在这里，我们定义主机、用户和身份验证密钥的路径。此外，我们还定义端口和我们打算列出其文件夹的远程目录。

接下来，让我们创建一个JSch对象并添加PRIVATE_KEY用于身份验证：

```java
JSch jsch = new JSch();
jsch.addIdentity(PRIVATE_KEY);
```

然后，让我们创建一个会话并连接到远程服务器：

```java
Session session = jsch.getSession(USER, HOST, PORT);
session.setConfig("StrictHostKeyChecking", "no");
session.connect();
```

**Session对象允许我们创建一个新的SSH会话**。为简单起见，我们禁用严格主机密钥检查。

此外，让我们通过建立的SSH连接打开一个SFTP通道：

```java
ChannelSftp channelSftp = (ChannelSftp) session.openChannel("sftp");
channelSftp.connect();
```

这里，我们在已建立的会话上创建了一个安全的文件传输协议，**ChannelSftp对象允许我们从远程服务器上传、下载、列出等**。

### 2.1 详细文件列表

现在我们已经打开了SFTP通道，让我们检索指定远程目录中的文件列表：

```java
Vector<ChannelSftp.LsEntry> files = channelSftp.ls(REMOTE_DIR);
for (ChannelSftp.LsEntry entry : files) {
    LOGGER.info(entry.getLongname());
}
```

在上面的代码中，我们调用ChannelSftp对象上的ls()方法，该方法返回ChannelSftp.LsEntry对象的[Vector](https://www.baeldung.com/java-vector-guide)，每个对象代表一个文件或目录。然后，我们循环遍历文件和目录列表并记录每个文件或目录的长名称。**getLongname()方法包含其他详细信息，例如权限、所有者、组和大小**。

### 2.2 仅文件名

如果我们只对文件名感兴趣，我们可以调用ChannelSftp.LsEntry对象上的getFilename()方法：

```java
Vector<ChannelSftp.LsEntry> files = channelSftp.ls(REMOTE_DIR);
for (ChannelSftp.LsEntry entry : files) {
    LOGGER.info(entry.getFilename());
}
```

值得注意的是，操作成功后我们必须关闭SSH会话和SFTP通道：

```java
channelSftp.disconnect();
session.disconnect();
```

本质上，关闭连接有助于释放资源。

## 3. 使用Apache Mina SSHD库

Apache Mina SSHD库旨在支持为客户端和服务器端提供SSH协议的Java应用程序。

我们可以执行多个SSH操作，如文件传输、部署等。要使用该库，让我们将[sshd-core](https://mvnrepository.com/artifact/org.apache.sshd/sshd-core)和[sshd-sftp](https://mvnrepository.com/artifact/org.apache.sshd/sshd-sftp)依赖项添加到pom.xml：

```xml
<dependency>
    <groupId>org.apache.sshd</groupId>
    <artifactId>sshd-core</artifactId>
    <version>2.13.1</version>
</dependency>
<dependency>
    <groupId>org.apache.sshd</groupId>
    <artifactId>sshd-sftp</artifactId>
    <version>2.13.1</version>
</dependency>
```

让我们维护上一节中使用的连接详细信息。首先，让我们启动SSH客户端：

```java
try (SshClient client = SshClient.setUpDefaultClient()) {
    client.start();
    client.setServerKeyVerifier(AcceptAllServerKeyVerifier.INSTANCE);
    // ...  
}
```

接下来，让我们连接到SSH服务器：

```java
try (ClientSession session = client.connect(USER, HOST, PORT).verify(10000).getSession()) {
    FileKeyPairProvider fileKeyPairProvider = new FileKeyPairProvider(Paths.get(privateKey));
    Iterable<KeyPair> keyPairs = fileKeyPairProvider.loadKeys(null);
    for (KeyPair keyPair : keyPairs) {
        session.addPublicKeyIdentity(keyPair);
    }

    session.auth().verify(10000);
}
```

在上面的代码中，我们使用身份验证凭据创建客户端会话。此外，我们使用FileKeyPairProvider对象加载私钥，**由于私钥不需要密码，因此我们将null传递给loadKeys()方法**。

### 3.1 详细文件列表

为了列出远程服务器上的文件夹，让我们创建一个SftpClientFactory对象来通过已建立的SSH会话打开SFTP通道：

```java
SftpClientFactory factory = SftpClientFactory.instance();
```

接下来，让我们读取远程目录并获取目录条目的Iterable对象：

```java
try (SftpClient sftp = factory.createSftpClient(session)) {
    Iterable<SftpClient.DirEntry> entriesIterable = sftp.readDir(REMOTE_DIR);
    List<SftpClient.DirEntry> entries = StreamSupport.stream(entriesIterable.spliterator(), false)
        .collect(Collectors.toList());
    for (SftpClient.DirEntry entry : entries) {
        LOGGER.info(entry.getLongFilename());
    }
}
```

在这里，我们读取远程目录并获取目录条目的[Iterable](https://www.baeldung.com/java-iterator)，并将其转换为[List](https://www.baeldung.com/java-convert-iterator-to-list)，然后我们将长文件名记录到控制台。由于我们使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)块，因此我们不需要明确关闭会话。

### 3.2 仅文件名

但是，为了仅获取文件名，我们可以对目录条目使用getFilename()方法，而不是getLongFileName()：

```java
for (SftpClient.DirEntry entry : entries) {
    LOGGER.info(entry.getFilename());
}
```

getFilename()消除了其他文件信息并仅记录文件名。

## 4. 使用SSHJ库

SSHJ库也是一个Java库，它提供用于连接远程服务器并与之交互的类。要使用该库，让我们将其依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.hierynomus</groupId>
    <artifactId>sshj</artifactId>
    <version>0.38.0</version>
</dependency>
```

另外，让我们保留前面部分使用的连接详细信息。

让我们创建一个SSHClient对象来建立与远程服务器的连接：

```java
try (SSHClient sshClient = new SSHClient()) {
    sshClient.addHostKeyVerifier(new PromiscuousVerifier());
    sshClient.connect(HOST);
    sshClient.authPublickey(USER, PRIVATE_KEY);
    // ...
}
```

然后，我们在已建立的SSH会话上建立一个SFTP通道：

```java
try (SFTPClient sftpClient = sshClient.newSFTPClient()) {
    List<RemoteResourceInfo> files = sftpClient.ls(REMOTE_DIR);
    for (RemoteResourceInfo file : files) {
        LOGGER.info("Filename: " + file.getName());
    }
}
```

在上面的代码中，我们调用ls()，它接收sftpClient上的远程目录并将其存储为RemoteResourceInfo类型。然后我们循环遍历文件条目并将文件名记录到控制台。

最后，我们可以使用getAttributes()方法获取有关文件的更多详细信息：

```java
LOGGER.info("Permissions: " + file.getAttributes().getPermissions());
LOGGER.info("Last Modification Time: " + file.getAttributes().getMtime());
```

在这里，我们通过调用getAttributes()方法上的getPermissions()和getMtime()方法进一步记录文件权限和修改时间。

## 5. 总结

在本文中，我们学习了如何使用JSch、Apache SSHD Mina和SSHJ库与远程服务器交互。此外，我们还了解了如何建立安全连接、使用私钥进行身份验证以及执行基本的文件操作。