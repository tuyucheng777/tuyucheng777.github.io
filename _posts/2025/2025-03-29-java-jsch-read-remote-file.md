---
layout: post
title:  Java JSch库逐行读取远程文件
category: libraries
copyright: libraries
excerpt: JSch
---

## 1. 概述

[Java安全通道(JSch)](https://www.baeldung.com/java-ssh-connection#jsch)库提供了用于将Java应用程序连接到远程服务器的API，从而实现了各种远程操作。其强大功能之一是能够直接从远程服务器读取文件，而无需将其下载到本地计算机。

在本教程中，我们将学习如何使用JSch连接到远程服务器并逐行读取特定文件。

## 2. Maven依赖

首先，让我们将JSch[依赖](https://mvnrepository.com/artifact/com.github.mwiede/jsch)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.github.mwiede</groupId>
    <artifactId>jsch</artifactId>
    <version>0.2.20</version>
</dependency>
```

该依赖提供用于建立与远程服务器的连接并打开SSH文件传输协议(SFTP)通道进行文件传输的类。

## 3. 连接远程服务器

让我们创建变量来存储到远程服务器的连接详细信息：

```java
private static final String HOST = "HOST_NAME";
private static final String USER = "USERNAME";
private static final String PRIVATE_KEY = "PRIVATE_KEY";
private static final int PORT = 22;
```

HOST可以是远程服务器的域名或IP地址，USER是用于向远程服务器进行身份验证的用户名，而PRIVATE_KEY表示[SSH](https://www.baeldung.com/cs/ssh-intro)私钥身份验证的路径，SSH端口默认为22。

接下来，让我们创建一个会话：

```java
JSch jsch = new JSch();
jsch.addIdentity(PRIVATE_KEY);
Session session = jsch.getSession(USER, HOST, PORT);
session.setConfig("StrictHostKeyChecking", "no");
session.connect();
```

在这里，我们创建JSch的一个实例，它是JSch功能的入口点。接下来，我们加载用于身份验证的密钥。然后，我们使用连接详细信息创建一个新的Session对象。为简单起见，我们禁用了严格的主机密钥检查。

## 4. 使用JSch读取远程文件

建立会话后，让我们为SFTP创建一个新的通道：

```java
ChannelSftp channelSftp = (ChannelSftp) session.openChannel("sftp");
channelSftp.connect();
```

在上面的代码中，我们创建了一个ChannelSftp实例，并通过调用它的connect()方法来建立连接。

现在我们已经打开了SFTP通道，我们可以[列出远程目录中的文件](https://www.baeldung.com/java-show-every-file-remote-server#using-the-jsch-library)，以轻松找到我们想要读取的文件。

我们定义一个字段来存储远程文件的路径：

```java
private static final String filePath = "REMOTE_DIR/examplefile.txt";
```

接下来让我们远程连接该文件并读取其内容：

```java
InputStream stream = channelSftp.get(filePath);
try (BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(stream))) {
    String line;
    while ((line = bufferedReader.readLine()) != null) {
        LOGGER.info(line);
    }
}
```

这里，我们创建了一个[BufferedReader](https://www.baeldung.com/java-buffered-reader)实例，它接收InputStream对象作为参数，以便有效地读取文件。**由于InputStream返回的是字节流，InputStreamReader可帮助将其解码为字符**。

最后，我们调用BufferedReader对象上的readLine()方法并使用while循环遍历所有行。

我们不需要明确关闭BufferedReader对象，因为它在[try-with-resources](https://www.baeldung.com/java-try-with-resources)块中使用。

## 5. 关闭连接

成功读取文件后，我们需要关闭SSH会话和SFTP通道：

```java
channelSftp.disconnect();
session.disconnect();
```

在上面的代码中，我们调用Session和ChannelSftp对象上的disconnect()方法来关闭连接并释放资源。

## 6. 总结

在本文中，我们将学习如何使用JSch库逐行读取远程文件，我们建立了与远程服务器的连接并创建了SFTP通道。然后，我们使用BufferedReader类逐行读取每个文件。