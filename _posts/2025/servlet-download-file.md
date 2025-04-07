---
layout: post
title:  在Servlet中下载文件的示例
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

##  1. 概述

Web应用程序的一个常见功能是能够下载文件。

在本教程中，**我们将介绍一个简单的示例，即创建可下载文件并通过Java Servlet应用程序提供该文件**。

我们使用的文件将来自webapp资源。

## 2. Maven依赖

**如果使用Jakarta EE，则不需要添加任何依赖**。但是，如果我们使用Java SE，则需要javax.servlet-api依赖：

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.1.0</version>
    <scope>provided</scope>
</dependency>
```

依赖的最新版本可以在[这里](https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api)找到。

## 3. Servlet

我们先看一下代码：

```java
@WebServlet("/download")
public class DownloadServlet extends HttpServlet {
    private final int ARBITARY_SIZE = 1048;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/plain");
        resp.setHeader("Content-disposition", "attachment; filename=sample.txt");

        try(InputStream in = req.getServletContext().getResourceAsStream("/WEB-INF/sample.txt");
            OutputStream out = resp.getOutputStream()) {

            byte[] buffer = new byte[ARBITARY_SIZE];

            int numBytesRead;
            while ((numBytesRead = in.read(buffer)) > 0) {
                out.write(buffer, 0, numBytesRead);
            }
        }
    }
}
```

### 3.1 请求端点

@WebServlet("/download")注解标记DownloadServlet类用于处理针对“/download”端点的请求。

或者，我们可以通过在web.xml文件中描述映射来实现这一点。

### 3.2 响应Content-Type

HttpServletResponse对象有一个名为setContentType的方法，我们可以使用它来设置HTTP响应的Content-Type标头。

Content-Type是标头属性的历史名称，另一个名称是MIME类型(多用途互联网邮件扩展)，我们现在简单地将该值称为媒体类型。

**该值可以是“application/pdf”、“text/plain”、“text/html”、“image/jpg”等**，官方列表由互联网号码分配机构(IANA)维护，可以在[此处](https://www.iana.org/assignments/media-types/media-types.xhtml#application)找到。

在我们的示例中，我们使用了一个简单的文本文件，**文本文件的Content-Type是“text/plain”**。

### 3.3 响应Content-Disposition

在响应对象中设置Content-Disposition标头会告诉浏览器如何处理它正在访问的文件。

浏览器理解Content-Disposition的使用惯例，但它实际上并不是HTTP标准的一部分。W3有一份关于Content-Disposition使用的备忘录，可在[此处](http://www.ietf.org/rfc/rfc1806.txt)阅读。

**响应主体的Content-Disposition值将是“inline”(用于要呈现的网页内容)或“attachment”(用于可下载文件)**。

如果未指定，默认的Content-Disposition是“inline”。

使用可选的标头参数，我们可以指定文件名“sample.txt”。

一些浏览器会立即使用给定的文件名下载文件，而其他浏览器则会显示包含我们预定义值的下载对话框。

采取的具体行动取决于浏览器。

### 3.4 读取文件并写入输出流

在剩下的代码行中，我们从请求中获取ServletContext，并使用它来获取“/WEB-INF/sample.txt”处的文件。

**然后，我们使用HttpServletResponse#getOutputStream()从资源的输入流读取数据，并将其写入响应的OutputStream**。

我们使用的字节数组的大小是任意的，可以根据从InputStream向OutputStream传递数据所分配的内存量来决定其大小；数字越小，循环次数越多；数字越大，内存使用率越高。

这个循环持续到numByteRead为0，因为这表示文件结束。

### 3.5 关闭并刷新

使用后必须关闭Stream实例以释放其当前持有的所有资源，还必须刷新Writer实例以将任何剩余的缓冲字节写入其目标。

使用try-with-resources语句，应用程序将自动关闭作为try语句一部分定义的任何AutoCloseable实例；在[此处](https://www.baeldung.com/java-try-with-resources)阅读有关try-with-resources的更多信息。

**我们通过这两种方法来释放内存，确保我们准备好的数据从我们的应用程序中发出去**。

### 3.6 下载文件

一切就绪后，我们现在可以运行Servlet了。

现在，当我们访问相对端点“/download”时，我们的浏览器将尝试将文件下载为“simple.txt”。

## 4. 总结

从Servlet下载文件的过程变得非常简单，使用流允许我们将数据作为字节传递出去，媒体类型会告知客户端浏览器需要什么类型的数据。

**如何处理响应取决于浏览器，但是，我们可以使用Content-Disposition标头提供一些指导**。
