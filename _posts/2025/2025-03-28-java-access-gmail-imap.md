---
layout: post
title:  使用IMAP从Gmail访问电子邮件
category: libraries
copyright: libraries
excerpt: Jakarta Mail
---

## 1. 概述

Gmail等网络邮件应用依靠[互联网消息应用程序协议(IMAP)](https://www.baeldung.com/cs/pop3-vs-imap-email-protocols#how-does-imap-work)等协议来从电子邮件服务器检索和处理[电子邮件](https://www.baeldung.com/java-email)。

在本教程中，我们将探索如何使用IMAP通过Java与Gmail服务器进行交互。此外，我们还将执行诸如阅读电子邮件、计算未读电子邮件、在文件夹之间移动电子邮件、将电子邮件标记为已读以及删除电子邮件等操作。此外，我们还将了解如何设置Google应用专用密码以进行身份验证。

## 2. 什么是IMAP？

IMAP是一种帮助电子邮件客户端检索存储在远程服务器上的电子邮件以供进一步操作的技术，与将电子邮件下载到客户端并将其从电子邮件服务器中删除的[POP3](https://www.baeldung.com/cs/pop3-vs-imap-email-protocols#how-does-pop3-work)不同，**IMAP将电子邮件保留在服务器上并允许多个客户端访问同一电子邮件服务器**。

此外，IMAP在从远程服务器访问电子邮件时保持开放连接，它允许在多个设备/机器之间实现更好的同步。

**IMAP默认在端口143上运行未加密连接，在端口993上运行SSL/TLS加密连接**。当连接加密时，它被称为IMAPS。

大多数网络邮件服务(包括Gmail)都支持IMAP和POP3。

## 3. 项目设置

首先，让我们将[jarkarta.mail-api](https://mvnrepository.com/artifact/jakarta.mail/jakarta.mail-api)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>jakarta.mail</groupId>
    <artifactId>jakarta.mail-api</artifactId>
    <version>2.1.3</version>
</dependency>
```

此依赖提供类来建立与电子邮件服务器的连接并执行不同形式的操作，如打开电子邮件、删除电子邮件和在不同文件夹之间移动电子邮件。

要连接到Gmail服务器，我们需要创建一个应用密码。让我们导航到[Google帐户](https://myaccount.google.com/)设置页面，并在侧栏中选择安全选项。然后，如果尚未启用双因素身份验证(2FA)，让我们启用它。

接下来，让我们在搜索栏中搜索“app password”，并为我们的应用程序创建一个新的应用专用密码。

## 4. 连接到Gmail

现在我们有了用于身份验证的应用密码，让我们创建一种方法来建立与Gmail服务器的连接：

```java
static Store establishConnection() throws MessagingException {
    Properties props = System.getProperties();
    props.setProperty("mail.store.protocol", "imaps");

    Session session = Session.getDefaultInstance(props, null);

    Store store = session.getStore("imaps");
    store.connect("imap.googlemail.com", "GMAIL", "APP PASSWORD");
    return store;
}
```

在上面的代码中，我们创建了一个返回Store对象的连接方法。我们创建一个[Properties](https://www.baeldung.com/java-properties)对象来设置邮件会话的配置参数，此外，我们为Store对象指定身份验证凭据，以便成功建立与我们电子邮件的连接。

## 5. 基本操作

通过IMAP成功连接到Gmail服务器后，我们可以执行阅读电子邮件、列出所有电子邮件、在文件夹之间移动电子邮件、删除电子邮件、将电子邮件标记为已读等基本操作。

### 5.1 统计总邮件数和未读邮件数

让我们列出收件箱和垃圾邮件文件夹中所有电子邮件和未读电子邮件的总数：

```java
static void emailCount(Store store) throws MessagingException {
    Folder inbox = store.getFolder("inbox");
    Folder spam = store.getFolder("[Gmail]/Spam");
    inbox.open(Folder.READ_ONLY);
    LOGGER.info("No of Messages : " + inbox.getMessageCount());
    LOGGER.info("No of Unread Messages : " + inbox.getUnreadMessageCount());
    LOGGER.info("No of Messages in spam : " + spam.getMessageCount());
    LOGGER.info("No of Unread Messages in spam : " + spam.getUnreadMessageCount());
    inbox.close(true);
}
```

上述方法接收Store对象作为参数，以建立与电子邮件服务器的连接。此外，我们定义一个Folder对象，指示收件箱和垃圾邮件文件夹。然后，我们在文件夹对象上调用getMessageCount()和getUnreadMessageCount()以获取总电子邮件数量和未读电子邮件数量。

**“\[Gmail\]”前缀表示Gmail层次结构中的特殊文件夹，并且必须用于Gmail专用文件夹**。除垃圾邮件文件夹外，其他Gmail专用文件夹包括\[Gmail\]/All Mail、\[Gmail\]/Bin和\[Gmail\]/Draft。

我们可以指定任何文件夹来获取其电子邮件计数。但是，如果该文件夹不存在，则会引发错误。

### 5.2 阅读电子邮件

此外，让我们阅读收件箱文件夹中的第一封电子邮件：

```java
static void readEmails(Store store) throws MessagingException, IOException {
    Folder inbox = store.getFolder("inbox");
    inbox.open(Folder.READ_ONLY);
    Message[] messages = inbox.getMessages();
    if (messages.length > 0) {
        Message message = messages[0];
        LOGGER.info("Subject: " + message.getSubject());
        LOGGER.info("From: " + Arrays.toString(message.getFrom()));
        LOGGER.info("Text: " + message.getContent());
    }
    inbox.close(true);
}
```

在上面的代码中，我们检索收件箱文件夹中的所有电子邮件并将它们存储在Message类型的数组中。然后，我们将电子邮件的主题、发件人地址和内容记录到控制台。

最后，操作成功后我们关闭收件箱文件夹。

### 5.3 搜索电子邮件

此外，我们可以通过创建SearchTerm实例并将其传递给search()方法来执行搜索操作：

```java
static void searchEmails(Store store, String from) throws MessagingException {
    Folder inbox = store.getFolder("inbox");
    inbox.open(Folder.READ_ONLY);
    SearchTerm senderTerm = new FromStringTerm(from);
    Message[] messages = inbox.search(senderTerm);
    Message[] getFirstFiveEmails = Arrays.copyOfRange(messages, 0, 5);
    for (Message message : getFirstFiveEmails) {
        LOGGER.info("Subject: " + message.getSubject());
        LOGGER.info("From: " + Arrays.toString(message.getFrom()));
    }
    inbox.close(true);
}
```

在这里，我们创建一个接收Store对象和搜索条件作为参数的方法。然后，我们打开收件箱并对其调用search()方法。在对Folder对象调用search()方法之前，我们将搜索查询传递给SearchTerm对象。

值得注意的是，搜索将仅限于指定的文件夹。

### 5.4 跨文件夹移动电子邮件

另外，我们可以在不同的文件夹之间移动电子邮件。如果指定的文件夹不可用，则会创建一个新文件夹：

```java
static void moveToFolder(Store store, Message message, String folderName) throws MessagingException {
    Folder destinationFolder = store.getFolder(folderName);
    if (!destinationFolder.exists()) {
        destinationFolder.create(Folder.HOLDS_MESSAGES);
    }
    Message[] messagesToMove = new Message[] { message };
    message.getFolder().copyMessages(messagesToMove, destinationFolder);
    message.setFlag(Flags.Flag.DELETED, true);
}
```

在这里，我们指定目标文件夹并在当前电子邮件文件夹上调用copyMessages()方法。copyMessages()方法接收messagesToMove和destinationFolder作为参数，将电子邮件复制到新文件夹后，我们将其从原始文件夹中删除。

### 5.5 将未读电子邮件标记为已读

此外，我们可以在指定的文件夹中将未读电子邮件标记为已读：

```java
static void markLatestUnreadAsRead(Store store) throws MessagingException {
    Folder inbox = store.getFolder("inbox");
    inbox.open(Folder.READ_WRITE);

    Message[] messages = inbox.search(new FlagTerm(new Flags(Flags.Flag.SEEN), false));
    if (messages.length > 0) {
        Message latestUnreadMessage = messages[messages.length - 1];
        latestUnreadMessage.setFlag(Flags.Flag.SEEN, true);
    }
    inbox.close(true);
}
```

**打开收件箱文件夹后，我们启用读写权限，因为将邮件标记为已读会改变其状态**。然后，我们搜索收件箱并列出所有未读邮件。最后，我们将最新的未读邮件标记为已读。

### 5.6 删除电子邮件

虽然我们可以将电子邮件移至“垃圾箱”文件夹，但我们也可以在连接仍处于打开状态时删除电子邮件：

```java
static void deleteEmail(Store store) throws MessagingException {
    Folder inbox = store.getFolder("inbox");
    inbox.open(Folder.READ_WRITE);
    Message[] messages = inbox.getMessages();

    if (messages.length >= 7) {
        Message seventhLatestMessage = messages[messages.length - 7];

        seventhLatestMessage.setFlag(Flags.Flag.DELETED, true);
        LOGGER.info("Delete the seventh message: " + seventhLatestMessage.getSubject());
    } else {
        LOGGER.info("There are less than seven messages in the inbox.");
    }
    inbox.close(true);
}
```

在上面的代码中，获取收件箱中的所有电子邮件后，我们选择数组中的第7封电子邮件并对其调用setFlag()方法。此操作将电子邮件标记为删除。

## 6. 总结

在本文中，我们介绍了使用Java的IMAP的基础知识，重点介绍了Gmail集成。此外，我们还探讨了基本的电子邮件操作，例如阅读电子邮件、在文件夹之间移动电子邮件、删除电子邮件以及将未读电子邮件标记为已读。