---
layout: post
title:  Java Properties入门
category: java
copyright: java
excerpt: Java Properties
---

## 1. 概述

大多数Java应用程序有时都需要使用属性，通常是将简单参数作为键值对存储在编译代码之外。因此，**该语言通过java.util.Properties为属性提供了一流的支持**，java.util.Properties是一个专为处理这些类型的配置文件而设计的实用程序类。

这就是我们在本教程中要重点关注的内容。

## 2. 加载属性

### 2.1 从属性文件

让我们从加载属性文件键值对的示例开始，我们将加载类路径上可用的两个文件：

app.properties:

```properties
version=1.0
name=TestApp
date=2016-11-12
```

和catalog：

```properties
c1=files
c2=images
c3=videos
```

请注意，虽然建议属性文件使用后缀“.properties”，但这不是必须的。

我们现在可以非常简单地将它们加载到Properties实例中：

```java
String rootPath = Thread.currentThread().getContextClassLoader().getResource("").getPath();
String appConfigPath = rootPath + "app.properties";
String catalogConfigPath = rootPath + "catalog";

Properties appProps = new Properties();
appProps.load(new FileInputStream(appConfigPath));

Properties catalogProps = new Properties();
catalogProps.load(new FileInputStream(catalogConfigPath));

String appVersion = appProps.getProperty("version");
assertEquals("1.0", appVersion);
        
assertEquals("files", catalogProps.getProperty("c1"));
```

只要文件的内容符合属性文件格式要求，它就可以被Properties类正确解析。有关[属性文件格式](https://en.wikipedia.org/wiki/.properties)的更多详细信息，请参见此处。

### 2.2 从XML文件加载

除了属性文件之外，Properties类还可以加载符合特定DTD规范的XML文件。

下面是从XML文件icons.xml加载键值对的示例：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>xml example</comment>
    <entry key="fileIcon">icon1.jpg</entry>
    <entry key="imageIcon">icon2.jpg</entry>
    <entry key="videoIcon">icon3.jpg</entry>
</properties>
```

现在让我们加载它：

```java
String rootPath = Thread.currentThread().getContextClassLoader().getResource("").getPath();
String iconConfigPath = rootPath + "icons.xml";
Properties iconProps = new Properties();
iconProps.loadFromXML(new FileInputStream(iconConfigPath));

assertEquals("icon1.jpg", iconProps.getProperty("fileIcon"));
```

## 3. 获取属性

我们可以使用getProperty(String key)和getProperty(String key, String defaultValue)通过其键来获取值。

如果键值对存在，这两个方法都会返回相应的值。但如果没有这样的键值对，前者会返回null，而后者会返回defaultValue：

```java
String appVersion = appProps.getProperty("version");
String appName = appProps.getProperty("name", "defaultName");
String appGroup = appProps.getProperty("group", "taketoday");
String appDownloadAddr = appProps.getProperty("downloadAddr");

assertEquals("1.0", appVersion);
assertEquals("TestApp", appName);
assertEquals("taketoday", appGroup);
assertNull(appDownloadAddr);
```

请注意，尽管Properties类从Hashtable类继承了get()方法，但我们不建议使用它来获取值。get()方法将返回一个Object值，该值只能转换为String，而getProperty()方法已经为我们正确处理了原始Object值。

下面的代码将引发异常：

```java
float appVerFloat = (float) appProps.get("version");
```

## 4. 设置属性

**我们可以使用setProperty()方法来更新现有的键值对或者添加新的键值对**：

```java
appProps.setProperty("name", "NewAppName"); // update an old value
appProps.setProperty("downloadAddr", "www.taketoday.com/downloads"); // add new key-value pair

String newAppName = appProps.getProperty("name");
assertEquals("NewAppName", newAppName);
        
String newAppDownloadAddr = appProps.getProperty("downloadAddr");
assertEquals("www.taketoday.com/downloads", newAppDownloadAddr);
```

请注意，尽管Properties类从Hashtable类继承了put()和putAll()方法，但是出于与get()方法相同的原因，我们建议不要使用它们；在Properties中只能使用String值。

下面的代码无法按预期工作；当我们使用getProperty()获取其值时，它将返回null：

```java
appProps.put("version", 2);
```

## 5. 删除属性

如果我们想删除一个键值对，我们可以使用remove()方法；

```java
String versionBeforeRemoval = appProps.getProperty("version");
assertEquals("1.0", versionBeforeRemoval);

appProps.remove("version");    
String versionAfterRemoval = appProps.getProperty("version");
assertNull(versionAfterRemoval);
```

## 6. 存储

### 6.1 存储到属性文件

Properties类提供了store()方法来输出键值对：

```java
String newAppConfigPropertiesFile = rootPath + "newApp.properties";
appProps.store(new FileWriter(newAppConfigPropertiesFile), "store to properties file");
```

第二个参数用于注释，如果我们不想写任何注释，我们可以简单地将其设置为null。

### 6.2 存储到XML文件

Properties类还提供了storeToXML()方法，用于以XML格式输出键值对：

```java
String newAppConfigXmlFile = rootPath + "newApp.xml";
appProps.storeToXML(new FileOutputStream(newAppConfigXmlFile), "store to xml file");
```

第二个参数与store()方法中的相同。

## 7. 其他常见操作

Properties类还提供了一些其他的方法来操作属性：

```java
appProps.list(System.out); // list all key-value pairs

Enumeration<Object> valueEnumeration = appProps.elements();
while (valueEnumeration.hasMoreElements()) {
    System.out.println(valueEnumeration.nextElement());
}

Enumeration<Object> keyEnumeration = appProps.keys();
while (keyEnumeration.hasMoreElements()) {
    System.out.println(keyEnumeration.nextElement());
}

int size = appProps.size();
assertEquals(3, size);
```

## 8. 默认属性列表

Properties对象可以包含另一个Properties对象作为其默认属性列表，如果在原始属性列表中找不到属性键，则会搜索默认属性列表。

除了“app.properties”之外，我们的类路径上还有另一个文件“default.properties”：

默认属性：

```properties
site=www.google.com
name=DefaultAppName
topic=Properties
category=core-java
```

示例代码：

```java
String rootPath = Thread.currentThread().getContextClassLoader().getResource("").getPath();

String defaultConfigPath = rootPath + "default.properties";
Properties defaultProps = new Properties();
defaultProps.load(new FileInputStream(defaultConfigPath));

String appConfigPath = rootPath + "app.properties";
Properties appProps = new Properties(defaultProps);
appProps.load(new FileInputStream(appConfigPath));

assertEquals("1.0", appVersion);
assertEquals("TestApp", appName);
assertEquals("www.google.com", defaultSite);
```

## 9. 属性和编码

默认情况下，属性文件应采用ISO-8859-1(Latin-1)编码，因此通常不应使用ISO-8859-1之外的字符的属性。

如果有必要，我们可以借助工具(例如JDK native2ascii工具或文件上的显式编码)来解决此限制。

对于XML文件，loadFromXML()方法和storeToXML()方法默认使用UTF-8字符编码。

但是，当读取编码不同的XML文件时，我们可以在DOCTYPE声明中指定。写入也足够灵活，我们可以在storeToXML()API的第三个参数中指定编码。

## 10. 总结

在本文中，我们讨论了Properties类的基本用法。我们学习了如何使用Properties；以属性和XML格式加载和存储键值对；操作Properties对象中的键值对，例如检索值、更新值和获取其大小；最后，如何使用Properties对象的默认列表。