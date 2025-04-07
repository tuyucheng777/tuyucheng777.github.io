---
layout: post
title:  使用Servlet和JSP上传文件
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 简介

在本快速教程中，我们将了解如何从Servlet上传文件。

为了实现这一点，我们首先看到原生@MultipartConfig注解提供文件上传功能的原始Jakarta EE解决方案。

然后，我们将介绍适用于早期版本的Servlet API的[Apache Commons FileUpload](https://commons.apache.org/proper/commons-fileupload/using.html)库。

## 2. 使用Jakarta EE @MultipartConfig

**Jakarta EE具有开箱即用的支持多部分上传的能力**。

因此，当为Jakarta EE应用程序添加文件上传支持时，它可能是默认选项。

首先，让我们在HTML文件中添加一个表单：

```html
<form method="post" action="multiPartServlet" enctype="multipart/form-data">
    Choose a file: <input type="file" name="multiPartServlet" />
    <input type="submit" value="Upload" />
</form>
```

应使用enctype=“multipart/form-data”属性定义表单，以表示分段上传。

接下来，**我们将使用@MultipartConfig注解为我们的HttpServlet添加正确的信息**：

```java
@MultipartConfig(fileSizeThreshold = 1024 * 1024,
        maxFileSize = 1024 * 1024 * 5,
        maxRequestSize = 1024 * 1024 * 5 * 5)
public class MultipartServlet extends HttpServlet {
    // ...
}
```

然后，让我们确保设置了默认服务器上传文件夹：

```java
String uploadPath = getServletContext().getRealPath("") + File.separator + UPLOAD_DIRECTORY;
File uploadDir = new File(uploadPath);
if (!uploadDir.exists()) uploadDir.mkdir();
```

最后，我们可以使用getParts()方法轻松地从请求中检索File，并将其保存到磁盘：

```java
for (Part part : request.getParts()) {
    fileName = getFileName(part);
    part.write(uploadPath + File.separator + fileName);
}
```

请注意，在此示例中，我们使用了辅助方法getFileName()：

```java
private String getFileName(Part part) {
    for (String content : part.getHeader("content-disposition").split(";")) {
        if (content.trim().startsWith("filename"))
            return content.substring(content.indexOf("=") + 2, content.length() - 1);
        }
    return Constants.DEFAULT_FILENAME;
}
```

**对于Servlet 3.1项目，我们也可以使用Part.getSubmittedFileName()方法**：

```java
fileName = part.getSubmittedFileName();
```

## 3. 使用 Apache Commons FileUpload

**如果我们不是使用Servlet 3.0项目，我们可以直接使用Apache Commons FileUpload库**。

### 3.1 设置

我们需要使用以下pom.xml依赖来运行我们的示例：

```xml
<dependency> 
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-fileupload2-jakarta-servlet6</artifactId>
    <version>2.0.0-M2</version>
</dependency>
```

在Maven的中央仓库中快速搜索即可找到最新版本：[commons-fileupload2-jakarta-servlet6](https://mvnrepository.com/artifact/org.apache.commons/commons-fileupload2-jakarta-servlet6)。

### 3.2 上传Servlet

整合Apache的FileUpload库的3个主要部分如下：

- .jsp页面中的上传表单
- 配置你的DiskFileItemFactory和ServletFileUpload对象
- 处理多部分文件上传的实际内容

上传表单与上一节的相同。

让我们继续创建Jakarta EE Servlet。

在我们的请求处理方法中，我们可以对传入的HttpRequest进行检查，看它是否是多部分上传。

我们还将在DiskFileItemFactory上指定临时(在处理时)分配给文件上传的资源。

最后，**我们将创建一个ServletFileUpload对象，它将代表实际文件本身**；它将公开分段上传的内容以供最终持久化服务器端使用：

```java
if (ServletFileUpload.isMultipartContent(request)) {
    DiskFileItemFactory factory = new DiskFileItemFactory();
    factory.setSizeThreshold(MEMORY_THRESHOLD);
    factory.setRepository(new File(System.getProperty("java.io.tmpdir")));

    ServletFileUpload upload = new ServletFileUpload(factory);
    upload.setFileSizeMax(MAX_FILE_SIZE);
    upload.setSizeMax(MAX_REQUEST_SIZE);
    String uploadPath = getServletContext().getRealPath("") + File.separator + UPLOAD_DIRECTORY;
    File uploadDir = new File(uploadPath);
    if (!uploadDir.exists()) {
        uploadDir.mkdir();
    }
    //...
}
```

然后我们可以提取这些内容并将它们写入磁盘：

```java
if (ServletFileUpload.isMultipartContent(request)) {
    //...
    List<FileItem> formItems = upload.parseRequest(request);
    if (formItems != null && formItems.size() > 0) {
        for (FileItem item : formItems) {
	        if (!item.isFormField()) {
	            String fileName = new File(item.getName()).getName();
	            String filePath = uploadPath + File.separator + fileName;
                    File storeFile = new File(filePath);
                    item.write(storeFile);
                    request.setAttribute("message", "File " + fileName + " has uploaded successfully!");
	        }
        }
    }
}
```

## 4. 运行示例

将项目编译为.war后，我们可以将其放入本地Tomcat实例并启动它。

然后，我们可以访问主上传视图，其中显示一个表单：

![](/assets/images/2025/webmodules/uploadfileservlet01.png)

成功上传文件后，我们应该看到以下消息：

![](/assets/images/2025/webmodules/uploadfileservlet02.png)

最后，我们可以检查Servlet中指定的位置：

![](/assets/images/2025/webmodules/uploadfileservlet03.png)

## 5. 总结

就这样，我们学习了如何使用Jakarta EE以及Apache的 Common FileUpload库提供多部分文件上传。
