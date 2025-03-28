---
layout: post
title:  Simple Java Mail简介
category: libraries
copyright: libraries
excerpt: Simple Java Mail
---

## 1. 简介

[Simple Java Mail](https://www.simplejavamail.org/)是一个流行的开源库，可简化Java应用程序中的电子邮件发送过程。与标准JavaMail API 相比，它提供了用户友好的API，让我们可以专注于电子邮件的内容和收件人，而不是低级细节。

在本教程中，我们将探讨设置Simple Java Mail的过程，并学习如何发送电子邮件(包括附件和HTML内容)、处理异常等。

## 2. 设置项目

我们首先将[Simple Java Mail](https://mvnrepository.com/artifact/org.simplejavamail/simple-java-mail)依赖添加到项目的配置文件pom.xml中：

```xml
<dependency>
    <groupId>org.simplejavamail</groupId>
    <artifactId>simple-java-mail</artifactId>
    <version>8.7.0</version>
</dependency>
```

## 3. 发送电子邮件

让我们首先使用Simple Java Mail发送一封简单的电子邮件。

电子邮件正文内容主要分为两个部分：纯文本和HTML。**纯文本是指大多数电子邮件客户端中显示的未经格式化的内容，没有任何特殊格式或样式**。无论电子邮件客户端是否能够显示HTML内容，它们都能保证在所有电子邮件客户端中读取。

这是一个基本示例，演示如何构建简单的电子邮件：

```java
Email email = EmailBuilder.startingBlank()
    .from("sender@example.com")
    .to("recipient@example.com")
    .withSubject("Email with Plain Text!")
    .withPlainText("This is a test email sent using SJM.")
    .buildEmail();
```

在此示例中，我们使用EmailBuilder类中的StartingBlank()方法以默认值初始化新的Email对象，**它通过创建没有任何预定义内容或配置的空白电子邮件模板来提供构建电子邮件消息的起点**。

随后，我们可以使用EmailBuilder类提供的各种方法逐步向此模板填充发件人、收件人、主题、正文和其他属性。**例如，withPlainText()方法接收一个字符串参数，其中包含我们要包含在电子邮件正文中的实际文本消息**。

接下来，我们使用MailerBuilder配置SMTP服务器详细信息，包括服务器的地址、端口、用户名和密码。最后，使用Mailer类发送电子邮件：

```java
Mailer mailer = MailerBuilder
    .withSMTPServer("smtp.example.com", 25, "username", "password")
    .buildMailer();

mailer.sendMail(email);
```

**此外，要将一封电子邮件发送给多个收件人，我们可以在to()方法中简单地指定多个用逗号分隔的电子邮件地址**：

```java
Email email = EmailBuilder.startingBlank()
    // ...
    .to("recipient1@example.com, recipient2@example.com, recipient3@example.com")
    // ...
    .buildEmail();
```

## 4. 发送附件

向电子邮件添加附件很简单，**我们可以使用EmailBuilder类的withAttachment()方法将各种类型的文件(例如图像、文档或档案)附加到电子邮件消息中**。

下面是使用withAttachment()方法在电子邮件中附加文件的示例：

```java
Email email = EmailBuilder.startingBlank()
    .from("sender@example.com")
    .to("recipient@example.com")
    .withSubject("Email with Plain Text and Attachment!")
    .withPlainText("This is a test email with attachment sent using SJM.")
    .withAttachment("important_document.pdf", new FileDataSource("path/to/important_document.pdf"))
    .buildEmail();
```

在示例中，withAttachment()方法接收文件名和表示附件数据的FileDataSource对象。此外，如果我们需要将多个文件附加到电子邮件，我们可以使用AttachmentResource对象列表：

```java
List<AttachmentResource> arList = new ArrayList<>();
arList.add(new AttachmentResource("important_document.pdf", new FileDataSource("path/to/important_document.pdf")));
arList.add(new AttachmentResource("company_logo.png", new FileDataSource("path/to/company_logo.png")));
```

我们将使用withAttachments()方法，而不是withAttachment()方法。此方法允许我们传入附件资源集合：

```java
Email email = EmailBuilder.startingBlank()
    // ...
    .withAttachments(arList)
    .buildEmail();
```

## 5. 发送HTML内容

Simple Java Mail允许我们发送包含HTML内容的电子邮件，并直接在电子邮件中嵌入图像，这对于创建具有视觉吸引力和信息量丰富的电子邮件非常有用。**要在HTML电子邮件内容中包含嵌入图像，我们需要使用CID方案引用图像**。

**此机制的作用类似于一个唯一标识符，将HTML内容中的图像引用与电子邮件中附加的实际图像数据联系起来**。本质上，它告诉电子邮件客户端将图像定位在何处才能正确显示。

以下是如何使用CID创建包含图像引用的HTML电子邮件：

```java
String htmlContent = "<h1>This is an email with HTML content</h1>" +
    "<p>This email body contains additional information and formatting.</p>" +
    "<img src=\"cid:company_logo\" alt=\"Company Logo\">";
```

在此示例中，<img\>标签使用标识符cid:company_logo引用图像，这在HTML内容中建立了链接。**在构建电子邮件时，我们使用withEmbeddedImage()方法将图像附件与所选标识符company_logo关联，这确保它与HTML内容中使用的标识符匹配**。

以下示例演示如何发送包含HTML内容和嵌入图像的电子邮件：

```java
Email email = EmailBuilder.startingblank()
    .from("sender@example.com")
    .to("recipient@example.com")
    .withSubject("Email with HTML and Embedded Image!")
    .withHTMLText(htmlContent)
    .withEmbeddedImage("company_logo", new FileDataSource("path/to/company_logo.png"))
    .buildEmail();
```

图像数据本身并不直接嵌入HTML内容中，它通常作为单独的附件包含在电子邮件中。

还建议使用withPlainText()提供HTML内容旁边的纯文本版本，**这为使用不支持HTML呈现的电子邮件客户端的收件人提供了后备选项**：

```java
Email email = EmailBuilder.startingBlank()
    // ...
    .withHTMLText(htmlContent)
    .withPlainText("This message is displayed as plain text because your email client doesn't support HTML.")
    .buildEmail();...
```

## 6. 回复和转发电子邮件

Simple Java Mail提供构建电子邮件对象以回复和转发现有电子邮件的功能。**回复电子邮件时，原始电子邮件会引用在回复正文中**。同样，转发电子邮件时，原始电子邮件会作为单独的正文包含在转发中。

要回复电子邮件，我们使用replyingTo()方法并提供收到的电子邮件作为参数。然后，我们根据需要配置电子邮件并使用EmailBuilder构建它：

```java
Email email = EmailBuilder
    .replyingTo(receivedEmail)
    .from("sender@example.com")
    .prependText("This is a Reply Email. Original email included below:")
    .buildEmail();
```

要转发电子邮件，我们使用forwarding()方法并提供收到的电子邮件作为参数。然后，我们根据需要配置电子邮件，包括任何其他文本，并使用EmailBuilder构建它：

```java
Email email = EmailBuilder
    .forwarding(receivedEmail)
    .from("sender@example.com")
    .prependText("This is a Forwarded Email. See below email:")
    .buildEmail();
```

但是，重要的是要了解Simple Java Mail本身无法直接从电子邮件服务器检索电子邮件。**我们需要利用[javax.mail](https://www.baeldung.com/java-email)等其他库来访问和检索电子邮件以进行回复或转发**。

## 7. 处理异常

发送电子邮件时，处理传输过程中可能发生的[异常](https://www.baeldung.com/java-exceptions)以确保功能的健壮性至关重要。**当发送电子邮件时发生错误时，Simple Java Mail会抛出一个名为MailException的受检异常**。

为了有效地处理潜在的错误，我们可以将电子邮件发送逻辑封装在try–catch块中，以捕获MailException类的实例。以下代码片段演示了如何使用try-catch块处理MailException：

```java
try {
    // ...
    mailer.sendMail(email);
} catch (MailException e) {
    // ...
}
```

此外，在发送邮件之前验证邮件地址可以降低邮件传输过程中出错的可能性，我们可以通过使用邮件对象提供的validate()方法来实现这一点：

```java
boolean validEmailAdd = mailer.validate(email);
if (validEmailAdd) {
    mailer.sendMail(email);
} else {
    // prompt user invalid email address
}
```

## 8. 高级配置选项

此外，Simple Java Mail除了基本功能外，还提供各种配置选项。让我们探索可用的其他配置选项，例如设置自定义标题、配置发送或阅读回执以及限制电子邮件大小。

### 8.1 设置自定义标头

**我们可以使用MailerBuilder类上的withHeader()方法为电子邮件定义自定义标头**，这使我们能够在标准标头之外包含其他信息：

```java
Email email = Email.Builder
    .from("sender@example.com")
    .to("recipient@example.com")
    .withSubject("Email with Custom Header")
    .withPlainText("This is an important message.")
    .withHeader("X-Priority", "1")
    .buildEmail();
```

### 8.2 配置送达/已读回执

**送达回执确认电子邮件已送达收件人的邮箱，而已读回执确认收件人已打开或阅读电子邮件**。Simple Java Mail提供内置支持，可使用特定电子邮件标头配置这些回执：送达回执为Return-Receipt-To，已读回执为Disposition-Notification-To。

我们可以明确定义应将收据发送到的电子邮件地址，如果我们不提供地址，它将默认使用replyTo地址(如果可用)，或者使用fromAddress。

我们可以这样配置这些收据：

```java
Email email = EmailBuilder.startingBlank()
    .from("sender@example.com")
    .to("recipient@example.com")
    .withSubject("Email with Delivery/Read Receipt Configured!")
    .withPlainText("This is an email sending with delivery/read receipt.")
    .withDispositionNotificationTo(new Recipient("name", "address@domain.com", Message.RecipientType.TO))
    .withReturnReceiptTo(new Recipient("name", "address@domain.com", Message.RecipientType.TO))
    .buildEmail();
```

### 8.3 限制最大电子邮件大小

许多电子邮件服务器对它们可以接受的电子邮件的最大大小都有限制，**Simple Java Mail提供了一个功能来帮助我们防止发送超出这些限制的电子邮件**，这可以避免我们在尝试发送电子邮件时遇到错误。

我们可以使用MailerBuilder对象上的withMaximumEmailSize()方法配置允许的最大电子邮件大小，此方法接收一个整数值，表示以字节为单位的大小限制：

```java
Mailer mailer = MailerBuilder
    .withMaximumEmailSize(1024 * 1024 * 5)
    .buildMailer();
```

在此示例中，最大电子邮件大小设置为5MB。如果我们尝试发送超过定义限制的电子邮件，Simple Java Mail将抛出MailerException，原因为EmailTooBigException。

## 9. 总结

在本文中，我们探讨了Simple Java Mail的各个方面，包括发送带有附件的电子邮件和带有嵌入图像的HTML内容。此外，我们还深入研究了其高级功能，例如管理递送、阅读回执以及处理电子邮件回复和转发。

总的来说，它最适合需要直接高效地发送电子邮件的Java应用程序。