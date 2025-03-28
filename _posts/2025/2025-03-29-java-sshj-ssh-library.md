---
layout: post
title:  SSHJ简介
category: libraries
copyright: libraries
excerpt: SSHJ
---

## 1. 概述

**[SSHJ](https://github.com/hierynomus/sshj)是一个开源Java库，它使用[SSH](https://www.baeldung.com/cs/ssh-intro)协议与远程服务器进行安全通信**。

在本文中，我们将介绍SSHJ库的基本功能。

## 2. 依赖

要使用SSHJ库，我们必须向项目添加以下依赖：

```xml
<dependency>
    <groupId>com.hierynomus</groupId>
    <artifactId>sshj</artifactId>
    <version>0.38.0</version>
</dependency>
```

可以在Maven Central找到最新版本的[SSHJ库](https://mvnrepository.com/artifact/com.hierynomus/sshj)。

## 3. SSHJ库

SSHJ库有助于通过SSH建立与远程服务器的安全连接。

**使用SSHJ库，我们可以使用SCP或SFTP协议处理文件上传和下载。此外，我们还可以使用它进行本地端口转发和远程端口转发**。

## 4. 连接SSH客户端

SSH客户端可以使用密码或公钥身份验证连接到服务器，SSHJ库使我们能够使用任一方法登录。

### 4.1 密码验证

我们可以使用SSHJ库通过SSH端口连接到服务器。SSH连接需要指定主机名、端口、用户名和密码。

**SSH客户端使用authPassword()方法通过密码验证连接到服务器**：

```java
String host = // ...
int port = // ...
String username = // ...
String password = // ...

SSHClient client = new SSHClient();
client.addHostKeyVerifier(new PromiscuousVerifier());
client.connect(host, port);
client.authPassword(username, password);
```

正如我们在上面的代码中看到的，我们使用密码验证将客户端连接到主机。

### 4.2 公钥认证

我们也可以使用公钥连接到服务器。要使用公钥连接，我们需要在服务器上的known_hosts文件中有一个文件条目，或者我们可以在客户端计算机上为远程服务器[生成一个公钥](https://www.baeldung.com/linux/ssh-setup-public-key-auth)，并将公钥复制到服务器上的授权SSH密钥中。

**SSH客户端使用authPublickey()方法通过公钥认证连接到服务器**：

```java
String host = // ... 
String username = // ... 
File privateKeyFile = // ... 
int port = // ...
```

```java
SSHClient client = new SSHClient();
KeyProvider privateKey = client.loadKeys(privateKeyFile.getPath());
client.addHostKeyVerifier(new PromiscuousVerifier());
client.connect(host, port);
client.authPublickey(username, privateKey);
```

我们可以为客户端[生成一个公钥](https://www.baeldung.com/linux/ssh-setup-public-key-auth)，并在要连接的服务器上更新它。在其余的例子中，我们将使用第一种方法登录，即使用用户名和密码。

## 5. 通过SSH执行命令

**我们可以通过SSHJ库在连接到服务器的sshClient启动的会话上使用exec()方法执行命令**：

```java
SSHClient client = new SSHClient();
Session session = sshClient.startSession();
Command cmd = session.exec("ls -lsa");
BufferedReader reader = new BufferedReader(new InputStreamReader(cmd.getInputStream()));
String line;
while ((line = reader.readLine()) != null) {
    System.out.println(line);
}
cmd.join(5, TimeUnit.SECONDS);
session.close();
```

正如我们在上面的代码中看到的，我们为sshClient启动了一个会话。然后，我们执行[ls -lsa](https://www.baeldung.com/linux/ls-files-by-type)命令，该命令列出目录中的所有文件。然后我们使用[BufferedReader](https://www.baeldung.com/java-buffered-reader)读取执行命令的输出。

类似地，其他命令也可以在这里执行。

## 6. 通过SCP上传/下载文件

我们可以通过[SCP](https://www.baeldung.com/cs/transfer-files-protocols#2-scp)上传文件。**对于上传，我们使用SCPFileTransfer对象上的upload()方法**：

```java
String filePath = // ... 
```

```java
SSHClient ssh = new SSHClient();
ssh.useCompression();
ssh.newSCPFileTransfer()
    .upload(new FileSystemFile(filePath), "/upload/");
```

这里我们将一个文件传输到服务器上的upload目录。

**useCompression()方法将[zlib](https://www.baeldung.com/cs/zlib-vs-gzip-vs-zip)压缩添加到首选算法中，这可以显著提高速度，但不能保证它会成功协商**。如果客户端已连接，则重新协商完成；否则，它只会返回。我们也可以在连接客户端之前使用useCompression()。

**对于SCP文件下载，我们使用SCPFileTransfer对象上的download()方法**：

```java
String downloadPath = // ...
String fileName = // ...
```

```java
SSHClient ssh = new SSHClient();
ssh.useCompression();
ssh.newSCPFileTransfer()
    .download("/upload/" + fileName, downloadPath);
```

这里我们将文件从服务器上的upload目录下载到客户端上的downloadPath位置。

**上述上传和下载方法在内部运行scp命令，使用SSH连接将文件从本地机器复制到远程服务器，反之亦然**。

## 7. 通过SFTP上传/下载文件

我们可以通过[SFTP](https://www.baeldung.com/cs/transfer-files-protocols#3-sftp)上传文件。**对于上传，我们使用SFTPClient对象上的put()方法**：

```java
String filePath = // ...
SSHClient ssh = new SSHClient();
SFTPClient sftp = ssh.newSFTPClient();
sftp.put(new FileSystemFile(filePath), "/upload/");
```

这里我们将文件从客户端的用户主目录传输到服务器上的upload目录。

**对于SFTP下载，我们使用SFTPClient对象上的get()方法**：

```java
String downloadPath = // ...
String fileName = // ...
```

```java
SSHClient ssh = new SSHClient();
SFTPClient sftp = ssh.newSFTPClient();
sftp.get("/upload/" + fileName, downloadPath);
sftp.close();
```

这里我们将文件从服务器上的upload目录下载到客户端上的downloadPath位置。

## 8. 本地端口转发

[本地端口转发](https://www.baeldung.com/linux/ssh-setup-public-key-auth)用于访问远程服务器上的服务，就好像服务在客户端上运行一样：

```java
SSHClient ssh = new SSHClient();
Parameters params = new Parameters(ssh.getRemoteHostname(), 8081, "google.com", 80);
ServerSocket ss = new ServerSocket();
ss.setReuseAddress(true);
ss.bind(new InetSocketAddress(params.getLocalHost(), params.getLocalPort()));
ssh.newLocalPortForwarder(params, ss)
    .listen();
```

在这里，**我们将服务器的80端口转发到客户端机器的8081端口，这样我们就可以从客户端机器上的8081端口访问托管在服务器80端口上的网站或服务**。

## 9. 远程端口转发

使用[远程端口转发](https://www.baeldung.com/linux/ssh-setup-public-key-auth)，我们可以将客户端机器上运行的服务公开到远程服务器网络：

```java
SSHClient ssh = new SSHClient();
ssh.getConnection()
    .getKeepAlive()
    .setKeepAliveInterval(5);
ssh.getRemotePortForwarder()
    .bind(new Forward(8083), new SocketForwardingConnectListener(new InetSocketAddress("google.com", 80)));
ssh.getTransport()
    .join();
```

这里，**我们将客户端8083端口上运行的服务转发到远程服务器的80端口。实际上，客户端机器8083端口上运行的服务暴露在远程服务器的80端口上**。

对于本地和远程端口转发，我们需要确保有适当的防火墙设置。

## 10. 检查连接断开

我们需要检查连接断开情况以监控服务器连接状态和健康状况，SSHJ提供了使用keep alive检查连接断开的选项：

```java
String hostName = // ...
String userName = // ...
String password = // ...
```

```java
DefaultConfig defaultConfig = new DefaultConfig();
defaultConfig.setKeepAliveProvider(KeepAliveProvider.KEEP_ALIVE);
SSHClient ssh = new SSHClient(defaultConfig);

ssh.addHostKeyVerifier(new PromiscuousVerifier());
ssh.connect(hostName, 22);
ssh.getConnection()
    .getKeepAlive()
    .setKeepAliveInterval(5);
ssh.authPassword(userName, password);

Session session = ssh.startSession();
session.allocateDefaultPTY();
new CountDownLatch(1).await();
session.allocateDefaultPTY();

session.close();
ssh.disconnect();

```

在上面的代码中，我们可以看到**配置KeepAliveProvider.KEEP_ALIVE为SSHJ库启用了[Keep Alive](https://www.baeldung.com/linux/ssh-keep-alive)模式**。

我们使用setKeepAliveInterval()来设置来自客户端的两个keep-alive消息之间的间隔。

## 11. 总结

在本文中，我们回顾了SSHJ库的基本用法和实现，我们弄清楚了如何使用SCP和SFTP模式上传或下载文件。此外，我们还了解了如何使用密码或公钥身份验证连接SSH客户端。远程和本地端口转发也可以通过SSHJ库实现。总的来说，SSHJ库为Java中的SSH客户端完成了大部分工作。