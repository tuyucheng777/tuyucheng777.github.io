---
layout: post
title:  在Java中获取桌面路径
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个简短的教程中，我们将学习**两种在Java中获取桌面路径的方法**。第一种方式是使用System.getProperty()方法，第二种方式是使用FileSystemView类的getHomeDirectory()方法。

## 2. 使用System.getProperty()

Java的System类提供了Properties对象，它存储了当前工作环境的不同配置和属性。在我们的案例中，我们对一个特定属性感兴趣：**保存用户主目录的user.home属性**。可以使用[System.getProperty()](https://www.baeldung.com/java-system-get-property-vs-system-getenv)方法检索此属性，该方法允许获取特定系统属性的值。

让我们看一个如何使用user.home属性并在Java中获取桌面路径的示例：

```java
String desktopPath = System.getProperty("user.home") + File.separator +"Desktop";
```

要获取桌面路径，**我们必须在user.home的属性值之后添加“/Desktop”字符串**。

## 3. 使用FileSystemView.getHomeDirectory()

在Java中获取桌面路径的另一种方法是使用FileSystemView类，它提供有关文件系统及其组件的有价值的信息。此外，**我们可以使用getHomeDirectory()方法将用户的主目录作为File对象获取**。

让我们看看如何利用这个类来获取桌面路径：

```java
FileSystemView view = FileSystemView.getFileSystemView();
File file = view.getHomeDirectory();
String desktopPath = file.getPath();
```

在我们的示例中，我们首先使用getFileSystemView()方法获取FileSystemView类的实例，其次，我们对该实例调用getHomeDirectory()方法以获取用户的主目录作为File对象。最后，我们使用File.getPath()方法获取桌面路径作为String。

## 4. 总结

在这篇简短的文章中，我们解释了如何使用两种方法在Java中获取桌面路径。第一种方法是使用System.getProperty()方法从系统中获取user.home属性，第二种方法是使用FileSystemView类的 getHomeDirectory()方法。