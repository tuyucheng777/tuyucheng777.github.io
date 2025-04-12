---
layout: post
title:  使用Apache POI将JDBC结果集写入Excel文件
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

数据处理是软件开发中的关键任务之一，一个常见的用例是从数据库中检索数据，并将其导出为某种格式(例如Excel文件)以供进一步分析。

本教程将展示如何使用[Apache POI](https://www.baeldung.com/java-microsoft-excel#apache-poi)库将数据从JDBC ResultSet导出到Excel文件。

## 2. Maven依赖

在我们的示例中，我们将从数据库表中读取一些数据并将其写入Excel文件。让我们在pom.xml中定义[Apache POI](https://mvnrepository.com/artifact/org.apache.poi/poi)和[POI OOXML模式](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)依赖：

```xml
<dependency> 
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId> 
    <version>5.3.0</version> 
</dependency> 
<dependency> 
    <groupId>org.apache.poi</groupId> 
    <artifactId>poi-ooxml</artifactId> 
    <version>5.3.0</version> 
</dependency>
```

我们将采用[H2数据库](https://mvnrepository.com/artifact/com.h2database/h2)进行演示，因此也需要包含它的依赖：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.3.232</version>
</dependency>
```

## 3. 数据准备

接下来，让我们通过在H2数据库中创建products表并向其中插入行来准备一些数据以进行演示：

```sql
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY, 
    name VARCHAR(255) NOT NULL, 
    category VARCHAR(255), 
    price DECIMAL(10, 2) 
);

INSERT INTO products(name, category, price) VALUES ('Chocolate', 'Confectionery', 2.99);
INSERT INTO products(name, category, price) VALUES ('Fruit Jellies', 'Confectionery', 1.5);
INSERT INTO products(name, category, price) VALUES ('Crisps', 'Snacks', 1.69);
INSERT INTO products(name, category, price) VALUES ('Walnuts', 'Snacks', 5.95);
INSERT INTO products(name, category, price) VALUES ('Orange Juice', 'Juices', 2.19);
```

创建表并插入数据后，我们可以使用[JDBC](https://www.baeldung.com/java-jdbc)获取products表内存储的所有数据：

```java
try (Connection connection = getConnection();
    Statement statement = connection.createStatement();
    ResultSet resultSet = statement.executeQuery(dataPreparer.getSelectSql());) {
    // The logic of export data to Excel file.
}
```

我们忽略了getConnection()的实现细节，**通常，我们通过[原始JDBC连接](https://www.baeldung.com/jdbc-get-url-from-connection#example-class)、[连接池](https://www.baeldung.com/java-connection-pooling)或[DataSource](https://docs.oracle.com/en/java/javase/21/docs/api/java.sql/javax/sql/DataSource.html)获取JDBC连接**。

## 4. 创建工作簿

**一个Excel文件由一个Workbook组成，可以包含多个Sheet**。在我们的演示中，我们将创建一个Workbook和一个Sheet，稍后我们将把数据写入其中。首先，让我们创建一个Workbook：

```java
Workbook workbook = new XSSFWorkbook();
```

我们可以从Apache POI中选择一些工作簿变体：

- HSSFWorkbook：较旧的Excel格式(97-2003)生成器，扩展名为.xls
- XSSFWorkbook：用于创建较新的基于XML的Excel 2007格式(扩展名为.xlsx)
- SXSSFWorkbook：也创建带有.xlsx扩展名的文件，但通过流式传输，从而将内存使用量保持在最低水平

在这个例子中，我们将使用XSSFWorkbook。但是，**如果我们预计导出很多行，比如超过10000行，那么为了更有效地利用内存，最好使用SXSSFWorkbook而不是XSSFWorkbook**。

接下来，让我们在工作簿中创建一个名为“data”的工作表：

```java
Sheet sheet = workbook.createSheet("data");
```

## 5. 创建标题行

通常，标题会包含数据集中每一列的标题，由于我们这里处理的是从JDBC返回的[ResultSet](https://www.baeldung.com/jdbc-resultset#resultset-creation)对象，因此可以使用[ResultSetMetaData](https://www.baeldung.com/jdbc-resultset#metadata)接口，该接口提供有关ResultSet列的元数据。

让我们看看如何使用ResultSetMetaData获取列名并使用Apache POI创建Excel表的标题行：

```java
Row row = sheet.createRow(sheet.getLastRowNum() + 1);
for (int n = 0; n < numOfColumns; n++) {
    String label = resultSetMetaData.getColumnLabel(n + 1);
    Cell cell = row.createCell(n);
    cell.setCellValue(label);
}
```

**在此示例中，我们从ResultSetMetaData动态获取列名，并将其用作Excel工作表的标题单元格**。这样，我们就避免了列名的硬编码。

## 6. 创建数据行

添加标题行后，让我们将表格数据加载到Excel文件：

```java
while (resultSet.next()) {
    Row row = sheet.createRow(sheet.getLastRowNum() + 1);
    for (int n = 0; n < numOfColumns; n++) {
        Cell cell = row.createCell(n);
        cell.setCellValue(resultSet.getString(n + 1));
    }
}
```

我们对ResultSet进行迭代，每次迭代都会在工作表中创建一个新行。根据之前通过sheet.getLastRowNum()获取的标题行列号，我们对每一列进行迭代，以获取当前行的数据并写入相应的Excel单元格。

## 7. 写入工作簿

现在我们的Workbook已完全填充，我们可以将其写入Excel文件了，由于我们使用了XSSFWorkbook的实例作为实现，因此导出文件将以Excel 2007文件格式保存，扩展名为.xslx：

```java
File excelFile = // our file
try (OutputStream outputStream = new BufferedOutputStream(new FileOutputStream(excelFile))) {
    workbook.write(outputStream);
    workbook.close();
}
```

**写入后，通过在Workbook实例上调用close()方法显式关闭Workbook是一种很好的做法**，这将确保资源得到释放，并且数据被刷新到文件中。

现在，让我们看看导出的结果，这些结果符合我们的表定义，并且还保持了数据的插入顺序：

![](/assets/images/2025/apache/jdbcresultsetexcel01.png)

## 8. 总结

在本文中，我们介绍了如何使用Apache POI将JDBC ResultSet中的数据导出到Excel文件。我们创建了一个Workbook，从ResultSetMetaData中动态填充了标题行，并通过迭代ResultSet向工作表中填充了数据行。