---
layout: post
title:  使用Java进行SSH连接
category: libraries
copyright: libraries
excerpt: JSch、Apache MINA SSHD
---

## 1. 简介

[SSH](https://www.baeldung.com/cs/ssh-intro)是一种网络协议，允许一台计算机通过不安全的网络[安全地连接到另一台计算机](https://www.baeldung.com/linux/secure-shell-ssh)。在本教程中，我们将展示如何**使用JSch和Apache MINA SSHD库通过Java建立与远程SSH服务器的连接**。

在我们的示例中，我们将首先打开SSH连接，然后执行一个命令，读取输出并将其写入控制台，最后关闭SSH连接。我们将尽可能简化示例代码。

## 2. JSch

[JSch](http://www.jcraft.com/jsch/)是SSH2的Java实现，它允许我们连接到SSH服务器并使用端口转发、X11转发和文件传输。此外，它采用BSD风格许可证，并为我们提供了一种使用Java建立SSH连接的简便方法。

首先，让我们将[JSch Maven依赖](https://mvnrepository.com/artifact/com.jcraft/jsch)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>com.jcraft</groupId>
    <artifactId>jsch</artifactId>
    <version>0.1.55</version>
</dependency>
```

### 2.1 实现

**要使用JSch建立SSH连接，我们需要用户名、密码、主机URL和SSH端口**。默认SSH端口是22，但我们可能会将服务器配置为使用其他端口进行SSH连接：

```java
public static void listFolderStructure(String username, String password, String host, int port, String command) throws Exception {
    Session session = null;
    ChannelExec channel = null;
    
    try {
        session = new JSch().getSession(username, host, port);
        session.setPassword(password);
        session.setConfig("StrictHostKeyChecking", "no");
        session.connect();
        
        channel = (ChannelExec) session.openChannel("exec");
        channel.setCommand(command);
        ByteArrayOutputStream responseStream = new ByteArrayOutputStream();
        channel.setOutputStream(responseStream);
        channel.connect();
        
        while (channel.isConnected()) {
            Thread.sleep(100);
        }
        
        String responseString = new String(responseStream.toByteArray());
        System.out.println(responseString);
    } finally {
        if (session != null) {
            session.disconnect();
        }
        if (channel != null) {
            channel.disconnect();
        }
    }
}
```

正如我们在代码中看到的，我们首先创建一个客户端会话并将其配置为连接到我们的SSH服务器。然后，我们创建一个用于与SSH服务器通信的客户端通道，在其中我们提供了一个通道类型-在本例中为exec，这意味着我们将向服务器传递shell命令。

此外，我们还应该为将要写入服务器响应的通道设置输出流。使用channel.connect()方法建立连接后，将传递命令，并将收到的响应写入控制台。

让我们看看如何使用JSch提供的不同配置参数：

- StrictHostKeyChecking：它表示应用程序是否将检查是否可以在已知主机中找到主机公钥。此外，可用的参数值为ask、yes和no，其中ask是默认值。如果我们将此属性设置为yes，JSch将永远不会自动将主机密钥添加到known_hosts文件中，并且它将拒绝连接到主机密钥已更改的主机，这会强制用户手动添加所有新主机。如果我们将其设置为no，JSch将自动将新主机密钥添加到已知主机列表中
- compression.s2c：指定是否对从服务器到客户端应用程序的数据流使用压缩，可用值为zlib和none，其中第二个是默认值
- compression.c2s：指定是否对客户端-服务器方向的数据流使用压缩，可用值为zlib和none，其中第二个为默认值

**与服务器的通信结束后关闭会话和SFTP通道非常重要，以避免内存泄漏**。

## 3. Apache MINA SSHD

[Apache MINA SSHD](https://mina.apache.org/sshd-project/)为基于Java的应用程序提供SSH支持，该库基于Apache MINA，这是一个可扩展且高性能的异步IO库。

让我们添加[Apache Mina SSHD](https://mvnrepository.com/artifact/org.apache.sshd/sshd-core) Maven依赖：

```xml
<dependency>
    <groupId>org.apache.sshd</groupId>
    <artifactId>sshd-core</artifactId>
    <version>2.5.1</version>
</dependency>
```

### 3.1 实现

让我们看一下使用Apache MINA SSHD连接SSH服务器的代码示例：

```java
public static void listFolderStructure(String username, String password,
                                       String host, int port, long defaultTimeoutSeconds, String command) throws IOException {

    SshClient client = SshClient.setUpDefaultClient();
    client.start();

    try (ClientSession session = client.connect(username, host, port)
            .verify(defaultTimeoutSeconds, TimeUnit.SECONDS).getSession()) {
        session.addPasswordIdentity(password);
        session.auth().verify(defaultTimeoutSeconds, TimeUnit.SECONDS);

        try (ByteArrayOutputStream responseStream = new ByteArrayOutputStream();
             ClientChannel channel = session.createChannel(Channel.CHANNEL_SHELL)) {
            channel.setOut(responseStream);
            try {
                channel.open().verify(defaultTimeoutSeconds, TimeUnit.SECONDS);
                try (OutputStream pipedIn = channel.getInvertedIn()) {
                    pipedIn.write(command.getBytes());
                    pipedIn.flush();
                }

                channel.waitFor(EnumSet.of(ClientChannelEvent.CLOSED),
                        TimeUnit.SECONDS.toMillis(defaultTimeoutSeconds));
                String responseString = new String(responseStream.toByteArray());
                System.out.println(responseString);
            } finally {
                channel.close(false);
            }
        }
    } finally {
        client.stop();
    }
}
```

使用Apache MINA SSHD时，我们的事件顺序与JSch非常相似。首先，我们使用SshClient类实例建立与SSH服务器的连接。如果我们使用SshClient.setupDefaultClient()对其进行初始化，我们将能够使用具有适合大多数用例的默认配置的实例。这包括密码、压缩、MAC、密钥交换和签名。

之后，我们将创建ClientChannel并将ByteArrayOutputStream附加到它，以便将其用作响应流。如我们所见，SSHD要求为每个操作定义超时。它还允许我们使用Channel.waitFor()方法定义在命令传递后等待服务器响应的时间。

值得注意的是，**SSHD会将完整的控制台输出写入响应流。JSch只会对命令执行结果执行此操作**。

有关Apache Mina SSHD的完整文档可在该[项目的官方GitHub仓库](https://github.com/apache/mina-sshd/tree/master/docs)中找到。

## 4. 总结

本文介绍了如何使用两个可用的Java库(JSch和Apache Mina SSHD)与Java建立SSH连接，我们还展示了如何将命令传递到远程服务器并获取执行结果。