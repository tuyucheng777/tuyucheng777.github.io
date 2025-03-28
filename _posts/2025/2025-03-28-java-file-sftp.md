---
layout: post
title:  使用Java通过SFTP传输文件
category: libraries
copyright: libraries
excerpt: JSch、SSHJ、Apache Commons VFS
---

## 1. 概述

在本教程中，**我们将讨论如何在Java中使用SFTP从远程服务器上传和下载文件**。

我们将使用三个不同的库：JSch、SSHJ和Apache Commons VFS。

## 2. 使用JSch

首先，让我们看看如何使用JSch库从远程服务器上传和下载文件。

### 2.1 Maven配置

我们需要将jsch依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.github.mwiede</groupId>
    <artifactId>jsch</artifactId>
    <version>0.2.16</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/com.github.mwiede/jsch)上找到jsch的最新版本。

### 2.2 设置JSch

现在我们将设置JSch。

JSch允许我们使用密码验证或公钥验证来访问远程服务器，在此示例中，**我们将使用密码验证**：

```java
private ChannelSftp setupJsch() throws JSchException {
    JSch jsch = new JSch();
    jsch.setKnownHosts("/Users/john/.ssh/known_hosts");
    Session jschSession = jsch.getSession(username, remoteHost);
    jschSession.setPassword(password);
    jschSession.connect();
    return (ChannelSftp) jschSession.openChannel("sftp");
}
```

在上面的例子中，remoteHost代表远程服务器的名称或IP地址(即example.com)，我们可以将测试中使用的变量定义为：

```java
private String remoteHost = "HOST_NAME_HERE";
private String username = "USERNAME_HERE";
private String password = "PASSWORD_HERE";
```

我们还可以使用以下命令生成known_hosts文件：

```shell
ssh-keyscan -H -t rsa REMOTE_HOSTNAME >> known_hosts
```

### 2.3 使用JSch上传文件

**要将文件上传到远程服务器，我们将使用方法ChannelSftp.put()**：

```java
@Test
public void whenUploadFileUsingJsch_thenSuccess() throws JSchException, SftpException {
    ChannelSftp channelSftp = setupJsch();
    channelSftp.connect();
 
    String localFile = "src/main/resources/sample.txt";
    String remoteDir = "remote_sftp_test/";
 
    channelSftp.put(localFile, remoteDir + "jschFile.txt");
 
    channelSftp.exit();
}
```

在这个例子中，该方法的第一个参数表示要传输的本地文件src/main/resources/sample.txt，而remoteDir是远程服务器上目标目录的路径。

### 2.4 使用JSch下载文件

**我们还可以使用ChannelSftp.get()从远程服务器下载文件**：

```java
@Test
public void whenDownloadFileUsingJsch_thenSuccess() throws JSchException, SftpException {
    ChannelSftp channelSftp = setupJsch();
    channelSftp.connect();
 
    String remoteFile = "welcome.txt";
    String localDir = "src/main/resources/";
 
    channelSftp.get(remoteFile, localDir + "jschFile.txt");
 
    channelSftp.exit();
}
```

remoteFile为需要下载的文件路径，localDir表示目标本地目录的路径。

## 3. 使用SSHJ

接下来，我们将使用SSHJ库从远程服务器上传和下载文件。

### 3.1 Maven配置

首先，我们将依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.hierynomus</groupId>
    <artifactId>sshj</artifactId>
    <version>0.38.0</version>
</dependency>
```

sshj的最新版本可以在[Maven Central](https://mvnrepository.com/artifact/com.github.mwiede/jsch)上找到。

### 3.2 设置SSHJ

**然后我们将设置SSHClient**。

SSHJ也允许我们使用密码或公钥验证来访问远程服务器。

我们将在示例中使用密码验证：

```java
private SSHClient setupSshj() throws IOException {
    SSHClient client = new SSHClient();
    client.addHostKeyVerifier(new PromiscuousVerifier());
    client.connect(remoteHost);
    client.useCompression();
    client.authPassword(username, password);
    return client;
}
```

我们还设置了客户端使用Zlib压缩，以提高传输性能。如果我们需要传输大文件，这将非常有用。

### 3.3 使用SSHJ上传文件

与JSch类似，**我们将使用SFTPClient.put()方法将文件上传到远程服务器**：

```java
@Test
public void whenUploadFileUsingSshj_thenSuccess() throws IOException {
    SSHClient sshClient = setupSshj();
    SFTPClient sftpClient = sshClient.newSFTPClient();
 
    sftpClient.put(localFile, remoteDir + "sshjFile.txt");
 
    sftpClient.close();
    sshClient.disconnect();
}
```

这里我们定义两个新变量：

```java
private String localFile = "src/main/resources/input.txt";
private String remoteDir = "remote_sftp_test/";
```

### 3.4 使用SSHJ下载文件

从远程服务器下载文件也是如此；我们将使用SFTPClient.get()：

```java
@Test
public void whenDownloadFileUsingSshj_thenSuccess() throws IOException {
    SSHClient sshClient = setupSshj();
    SFTPClient sftpClient = sshClient.newSFTPClient();
 
    sftpClient.get(remoteFile, localDir + "sshjFile.txt");
 
    sftpClient.close();
    sshClient.disconnect();
}
```

我们将添加上面使用的两个变量：

```java
private String remoteFile = "welcome.txt";
private String localDir = "src/main/resources/";
```

## 4. 使用Apache Commons VFS

最后，我们将使用Apache Commons VFS将文件传输到远程服务器。

事实上，**Apache Commons VFS内部使用了JSch库**。

### 4.1 Maven配置

我们需要将commons-vfs2依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-vfs2</artifactId>
    <version>2.9.0</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/org.apache.commons/commons-vfs2)上找到commons-vfs2的最新版本。

### 4.2 使用Apache Commons VFS上传文件

Apache Commons VFS有点不同。

**我们将使用FileSystemManager从目标文件创建FileObject，然后使用FileObject来传输我们的文件**。

在此示例中，我们将使用方法FileObject.copyFrom()上传文件：

```java
@Test
public void whenUploadFileUsingApacheVfs_thenSuccess() throws IOException {
    FileSystemManager manager = VFS.getManager();

    FileObject local = manager.resolveFile(
            System.getProperty("user.dir") + "/" + localFile);
    FileObject remote = manager.resolveFile(
            "sftp://" + username + ":" + password + "@" + remoteHost + "/" + remoteDir + "vfsFile.txt");

    remote.copyFrom(local, Selectors.SELECT_SELF);

    local.close();
    remote.close();
}
```

注意本地文件路径应该是绝对路径，远程文件路径应该以sftp://username:password@remoteHost开头。

### 4.3 使用Apache Commons VFS下载文件

从远程服务器下载文件非常相似；**我们还将使用FileObject.copyFrom()从remoteFile复制localFile**：

```java
@Test
public void whenDownloadFileUsingApacheVfs_thenSuccess() throws IOException {
    FileSystemManager manager = VFS.getManager();

    FileObject local = manager.resolveFile(
            System.getProperty("user.dir") + "/" + localDir + "vfsFile.txt");
    FileObject remote = manager.resolveFile(
            "sftp://" + username + ":" + password + "@" + remoteHost + "/" + remoteFile);

    local.copyFrom(remote, Selectors.SELECT_SELF);

    local.close();
    remote.close();
}
```

## 5. 总结

在本文中，我们学习了如何使用Java从远程SFTP服务器上传和下载文件。为此，我们使用了多个库：JSch、SSHJ和Apache Commons VFS。