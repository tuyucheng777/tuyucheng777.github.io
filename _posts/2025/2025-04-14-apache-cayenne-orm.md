---
layout: post
title:  Apache Cayenne ORM简介
category: persistence
copyright: persistence
excerpt: Apache Cayenne
---

## 1. 概述

[Apache Cayenne](https://cayenne.apache.org/)是一个开源库，提供建模工具、对象关系映射(又名ORM)等功能，用于本地持久化操作和远程服务。

在以下部分中，我们将了解如何使用Apache Cayenne ORM与MySQL数据库交互。

## 2. Maven依赖

首先，我们只需要添加以下依赖来启动Apache Cayenne和MySQL连接器以及JDBC驱动程序来访问我们的intro_cayenne数据库：

```xml
<dependency>
    <groupId>org.apache.cayenne</groupId>
    <artifactId>cayenne-server</artifactId>
    <version>4.0.M5</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.44</version>
    <scope>runtime</scope>
</dependency>
```

让我们配置Cayenne建模器插件，它将用于设计或设置我们的映射文件，作为数据库模式和Java对象之间的桥梁：

```xml
<plugin>
    <groupId>org.apache.cayenne.plugins</groupId>
    <artifactId>maven-cayenne-modeler-plugin</artifactId>
    <version>4.0.M5</version>
</plugin>
```

不要手动构建XML映射文件(很少这样做)，建议使用建模器，它是Cayenne发行版附带的一个非常先进的工具。

你可以根据你的操作系统从该[存档](http://cayenne.apache.org/download.html)中下载它，或者只需使用其中包含的Maven插件跨平台版本(JAR)。

Maven Central仓库托管[Apache Cayenne](https://mvnrepository.com/search?q=cayenne-server)、[建模器](https://mvnrepository.com/search?q=cayenne-modeler-maven-plugin)和[MySQL Connector](https://mvnrepository.com/artifact/mysql/mysql-connector-java)的最新版本。

接下来，让我们使用mvn install构建项目，并使用命令mvn cayenne-modeler:run启动建模器GUI以获取此屏幕的输出：

![](/assets/images/2025/persistence/apachecayenneorm01.png)

## 3. 设置

为了让Apache Cayenne查找正确的本地数据库，我们只需要在位于资源目录中的cayenne-project.xml文件中，使用正确的驱动程序、URL和用户填充其配置文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<domain project-version="9">
    <node name="datanode"
          factory
                  ="org.apache.cayenne.configuration.server.XMLPoolingDataSourceFactory"
          schema-update-strategy
                  ="org.apache.cayenne.access.dbsync.CreateIfNoSchemaStrategy">
        <data-source>
            <driver value="com.mysql.jdbc.Driver"/>
            <url value
                         ="jdbc:mysql://localhost:3306/intro_cayenne;create=true"/>
            <connectionPool min="1" max="1"/>
            <login userName="root" password="root"/>
        </data-source>
    </node>
</domain>
```

这里我们可以看到：

- 本地数据库名为intro_cayenne
- 如果尚未创建，Cayenne会为我们创建
- 我们将使用用户名root和密码root进行连接(根据数据库管理系统中注册的用户进行更改)

在内部，XMLPoolingDataSourceFactory负责从与DataNodeDescriptor关联的XML资源加载JDBC连接信息。

请注意，这些参数与数据库管理系统和JDBC驱动程序相关，因为该库可以支持许多不同的数据库。

每个版本都有一个适配器，详情请见此详细[列表](https://cayenne.apache.org/database-support.html)。请注意，4.0版本的完整文档尚未提供，因此我们在此参考之前的版本。

## 4. 映射与数据库设计

### 4.1 建模

现在让我们点击“Open Project”，导航到项目的资源文件夹并选择文件cayenne-project.xml，建模器将显示以下内容：

![](/assets/images/2025/persistence/apachecayenneorm02.png)

在这里，我们可以选择从现有数据库创建映射结构，也可以手动操作。本文将介绍如何使用建模器和现有数据库来快速入门Cayenne，并了解其工作原理。

让我们看一下我们的intro_cayenne数据库，它在两个表之间具有一对多关系，因为作者可以发布或拥有多篇文章：

- author：id(PK)和name
- article：id(主键)、title、content、author_id(外键)

现在，我们进入“Tools > Reengineer Database Schema”，所有映射配置将自动填充。在弹出的提示框中，只需填写cayenne-project.xml文件中提供的数据源配置，然后点击“Continue”即可：

![](/assets/images/2025/persistence/apachecayenneorm03.png)

在下一个屏幕上，我们需要勾上“Use Java primitive types”，如下所示：

![](/assets/images/2025/persistence/apachecayenneorm04.png)

我们还需要确保将cn.tuyucheng.taketoday.apachecayenne.persistent作为Java包并保存它；我们将看到XML配置文件已更新其defaultPackage属性以匹配Java包：

![](/assets/images/2025/persistence/apachecayenneorm05.png)

在每个ObjEntity中，我们必须指定子类的包，如下图所示，然后再次单击“save”图标：

![](/assets/images/2025/persistence/apachecayenneorm06.png)

现在在“Tools > Generate Classes”菜单上，选择“Standard Persistent Objects”作为类型；并在“Classes”选项卡上检查所有类并点击“generate”。

我们回到源代码看看我们的持久对象是否已经成功生成，也就是_Article.java和_Author.java。

请注意，所有这些配置都保存在同样位于资源文件夹中的datamap.map.xml文件中。

### 4.2 映射结构

资源文件夹中生成的XML映射文件使用了一些与Apache Cayenne相关的特定标签：

- DataNode(<node\>)：数据库模型，其内容包括连接数据库所需的所有信息(数据库名称、驱动程序和用户凭证)
- DataMap(<data-map\>)：持久实体及其关系的容器
- DbAttribute(<db-attribute\>)：表示数据库表中的一列
- DbEntity(<db-entity\>)：单个数据库表或视图的模型，它可以具有DbAttributes和关系
- ObjEntity(<obj-entity\>)：单个持久化Java类的模型；由对应于实体类属性的ObjAttributes和具有另一个实体类型的属性的ObjRelationships组成
- Embeddable(<embeddable\>)：Java类的模型，充当ObjEntity的属性，但对应于数据库中的多个列
- Procedure(<procedure\>)：在数据库中注册存储过程
- Query(<query\>)：查询模型，用于在配置文件中映射查询，我们也可以在代码中执行此操作

以下是全部[详细信息](https://cayenne.apache.org/docs/3.1/cayenne-guide/cayenne-mapping-structure.html)。

## 5. Cayenne API

剩下的唯一步骤是使用Cayenne API通过生成的类来执行数据库操作，同时要知道，对我们的持久类进行子类化只是以后自定义模型的最佳实践。

### 5.1 创建对象

在这里，我们只保存一个Author对象，稍后检查数据库中是否只有一条此类型的记录：

```java
@Test
public void whenInsert_thenWeGetOneRecordInTheDatabase() {
    Author author = context.newObject(Author.class);
    author.setName("Paul");

    context.commitChanges();

    long records = ObjectSelect.dataRowQuery(Author.class)
            .selectCount(context);

    assertEquals(1, records);
}
```

### 5.2 读取对象

保存Author后，我们只需通过特定属性的简单查询来查询它：

```java
@Test
public void whenInsert_andQueryByFirstName_thenWeGetTheAuthor() {
    Author author = context.newObject(Author.class);
    author.setName("Paul");

    context.commitChanges();

    Author expectedAuthor = ObjectSelect.query(Author.class)
            .where(Author.NAME.eq("Paul"))
            .selectOne(context);

    assertEquals("Paul", expectedAuthor.getName());
}
```

### 5.3 检索某个类的所有记录

我们将保存两个作者并检索作者对象集合以检查是否只保存了这两个作者：

```java
@Test
public void whenInsert_andQueryAll_thenWeGetTwoAuthors() {
    Author firstAuthor = context.newObject(Author.class);
    firstAuthor.setName("Paul");

    Author secondAuthor = context.newObject(Author.class);
    secondAuthor.setName("Ludovic");

    context.commitChanges();

    List<Author> authors = ObjectSelect
            .query(Author.class)
            .select(context);

    assertEquals(2, authors.size());
}
```

### 5.4 更新对象

更新过程也很简单，但我们首先需要拥有所需的对象，然后才能修改其属性并将其应用到数据库：

```java
@Test
public void whenUpdating_thenWeGetAnUpatedeAuthor() {
    Author author = context.newObject(Author.class);
    author.setName("Paul");
    context.commitChanges();

    Author expectedAuthor = ObjectSelect.query(Author.class)
            .where(Author.NAME.eq("Paul"))
            .selectOne(context);
    expectedAuthor.setName("Garcia");
    context.commitChanges();

    assertEquals(author.getName(), expectedAuthor.getName());
}
```

### 5.5 附加对象

我们可以将一篇文章分配给一个作者：

```java
@Test
public void whenAttachingToArticle_thenTheRelationIsMade() {
    Author author = context.newObject(Author.class);
    author.setName("Paul");

    Article article = context.newObject(Article.class);
    article.setTitle("My post title");
    article.setContent("The content");
    article.setAuthor(author);

    context.commitChanges();

    Author expectedAuthor = ObjectSelect.query(Author.class)
            .where(Author.NAME.eq("Smith"))
            .selectOne(context);

    Article expectedArticle = (expectedAuthor.getArticles()).get(0);

    assertEquals(article.getTitle(), expectedArticle.getTitle());
}
```

### 5.6 删除对象

删除已保存的对象会将其从数据库中完全移除，此后我们将看到查询结果为null：

```java
@Test
public void whenDeleting_thenWeLostHisDetails() {
    Author author = context.newObject(Author.class);
    author.setName("Paul");
    context.commitChanges();

    Author savedAuthor = ObjectSelect.query(Author.class)
            .where(Author.NAME.eq("Paul"))
            .selectOne(context);
    if(savedAuthor != null) {
        context.deleteObjects(author);
        context.commitChanges();
    }

    Author expectedAuthor = ObjectSelect.query(Author.class)
            .where(Author.NAME.eq("Paul"))
            .selectOne(context);

    assertNull(expectedAuthor);
}
```

### 5.7 删除某个类的所有记录

还可以使用SQLTemplate删除表中的所有记录，这里我们在每个测试方法之后执行此操作，以便在每次测试启动之前始终有一个空数据库：

```java
@After
public void deleteAllRecords() {
    SQLTemplate deleteArticles = new SQLTemplate(
            Article.class, "delete from article");
    SQLTemplate deleteAuthors = new SQLTemplate(
            Author.class, "delete from author");

    context.performGenericQuery(deleteArticles);
    context.performGenericQuery(deleteAuthors);
}
```

## 6. 总结

在本教程中，我们重点介绍如何使用Apache Cayenne ORM通过一对多关系执行CRUD操作。