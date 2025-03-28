---
layout: post
title:  JasperReports与Spring
category: libraries
copyright: libraries
excerpt: JasperReports
---

## 1. 概述

[JasperReports](http://community.jaspersoft.com/project/jasperreports-library)是一个开源报告库，使用户能够创建像素完美的报告，这些报告可以打印或导出为多种格式，包括PDF、HTML和XLS。

在本文中，我们将探讨其主要特性和类，并实现示例来展示其功能。

## 2. Maven依赖

首先，我们需要将jasperreports依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>net.sf.jasperreports</groupId>
    <artifactId>jasperreports</artifactId>
    <version>6.4.0</version>
</dependency>
```

可以在[此处](https://mvnrepository.com/artifact/net.sf.jasperreports/jasperreports)找到此工件的最新版本。

## 3. 报告模板

报告设计在JRXML文件中定义，这些是具有JasperReports引擎可以解释的特定结构的普通XML文件。

现在让我们只看一下JRXML文件的相关结构-以便更好地理解报告生成过程的Java部分，这是我们的主要关注点。

让我们创建一个简单的报表来显示员工信息：

```xml
<jasperReport ... >
    <field name="FIRST_NAME" class="java.lang.String"/>
    <field name="LAST_NAME" class="java.lang.String"/>
    <field name="SALARY" class="java.lang.Double"/>
    <field name="ID" class="java.lang.Integer"/>
    <detail>
        <band height="51" splitType="Stretch">
            <textField>
                <reportElement x="0" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{FIRST_NAME}]]></textFieldExpression>
            </textField>
            <textField>
                <reportElement x="100" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{LAST_NAME}]]></textFieldExpression>
            </textField>
            <textField>
                <reportElement x="200" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{SALARY}]]></textFieldExpression>
            </textField>
        </band>
    </detail>
</jasperReport>
```

### 3.1 编译报告

JRXML文件需要进行编译，以便报告引擎可以用数据填充其中。

让我们在JasperCompilerManager类的帮助下执行此操作：

```java
InputStream employeeReportStream = getClass().getResourceAsStream("/employeeReport.jrxml");
JasperReport jasperReport = JasperCompileManager.compileReport(employeeReportStream);
```

为了避免每次都编译它，我们可以将它保存到一个文件：

```java
JRSaver.saveObject(jasperReport, "employeeReport.jasper");
```

## 4. 填充报告

填充编译报告的最常见方法是使用数据库中的记录，这要求报告包含引擎将执行以获取数据的SQL查询。

首先，让我们修改报表以添加SQL查询：

```xml
<jasperReport ... >
    <queryString>
        <![CDATA[SELECT  FROM EMPLOYEE]]>
    </queryString>
    ...
</jasperReport>
```

现在，让我们创建一个简单的数据源：

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:employee-schema.sql")
            .build();
}
```

现在，我们可以填写报告：

```java
JasperPrint jasperPrint = JasperFillManager.fillReport(jasperReport, null, dataSource.getConnection());
```

请注意，我们将null传递给第2个参数，因为我们的报告尚未收到任何参数。

### 4.1 参数

参数对于将无法在其数据源中找到的数据传递到报表引擎或当数据根据不同的运行时条件发生变化时非常有用。

我们还可以使用在报告填充操作中收到的参数更改部分甚至整个SQL查询。

首先，让我们修改报告以接收3个参数：

```java
<jasperReport ... >
    <parameter name="title" class="java.lang.String" />
    <parameter name="minSalary" class="java.lang.Double" />
    <parameter name="condition" class="java.lang.String">
        <defaultValueExpression>
          <![CDATA["1 = 1"]]></defaultValueExpression>
    </parameter>
    // ...
</jasperreport>
```

现在，让我们添加一个标题部分来显示title参数：

```xml
<jasperreport ... >
    // ...
    <title>
        <band height="20" splitType="Stretch">
            <textField>
                <reportElement x="238" y="0" width="100" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$P{title}]]></textFieldExpression>
            </textField>
        </band>
    </title>
    ...
</jasperreport/>
```

接下来，让我们更改查询以使用minSalary和condition参数：

```sql
SELECT  FROM EMPLOYEE
    WHERE SALARY >= $P{minSalary} AND $P!{condition}
```

请注意使用condition参数时的不同语法，这告诉引擎该参数不应该用作标准的PreparedStatement参数，而应像该参数的值最初写入SQL查询中一样。

最后，让我们准备参数并填写报告：

```java
Map<String, Object> parameters = new HashMap<>();
parameters.put("title", "Employee Report");
parameters.put("minSalary", 15000.0);
parameters.put("condition", " LAST_NAME ='Smith' ORDER BY FIRST_NAME");

JasperPrint jasperPrint = JasperFillManager.fillReport(..., parameters, ...);
```

请注意，parameters的键与报表中的参数名称相对应。如果引擎检测到某个参数缺失，它会从该参数的defaultValueExpression中获取值(如果有)。

## 5. 导出

要导出报告，首先，我们实例化一个与我们需要的文件格式匹配的导出器类的对象。

然后，我们将之前填写的报告设置为输入并定义输出结果文件的位置。

或者，我们可以设置相应的报告和导出配置对象来自定义导出过程。

### 5.1 PDF

```java
JRPdfExporter exporter = new JRPdfExporter();

exporter.setExporterInput(new SimpleExporterInput(jasperPrint));
exporter.setExporterOutput(new SimpleOutputStreamExporterOutput("employeeReport.pdf"));

SimplePdfReportConfiguration reportConfig = new SimplePdfReportConfiguration();
reportConfig.setSizePageToContent(true);
reportConfig.setForceLineBreakPolicy(false);

SimplePdfExporterConfiguration exportConfig = new SimplePdfExporterConfiguration();
exportConfig.setMetadataAuthor("baeldung");
exportConfig.setEncrypted(true);
exportConfig.setAllowedPermissionsHint("PRINTING");

exporter.setConfiguration(reportConfig);
exporter.setConfiguration(exportConfig);

exporter.exportReport();
```

### 5.2 XLS

```java
JRXlsxExporter exporter = new JRXlsxExporter();
 
// Set input and output ...
SimpleXlsxReportConfiguration reportConfig = new SimpleXlsxReportConfiguration();
reportConfig.setSheetNames(new String[] { "Employee Data" });

exporter.setConfiguration(reportConfig);
exporter.exportReport();
```

### 5.3 CSV文件

```java
JRCsvExporter exporter = new JRCsvExporter();
 
// Set input ...
exporter.setExporterOutput(new SimpleWriterExporterOutput("employeeReport.csv"));

exporter.exportReport();
```

### 5.4 HTML

```java
HtmlExporter exporter = new HtmlExporter();
 
// Set input ...
exporter.setExporterOutput(new SimpleHtmlExporterOutput("employeeReport.html"));

exporter.exportReport();
```

## 6. 子报表

子报表只不过是嵌入在另一个报表中的标准报表。

首先，让我们创建一个报告来显示员工的电子邮件：

```xml
<jasperReport ... >
    <parameter name="idEmployee" class="java.lang.Integer" />
    <queryString>
        <![CDATA[SELECT  FROM EMAIL WHERE ID_EMPLOYEE = $P{idEmployee}]]>
    </queryString>
    <field name="ADDRESS" class="java.lang.String"/>
    <detail>
        <band height="20" splitType="Stretch">
            <textField>
                <reportElement x="0" y="0" width="156" height="20"/>
                <textElement/>
                <textFieldExpression class="java.lang.String">
                  <![CDATA[$F{ADDRESS}]]></textFieldExpression>
            </textField>
        </band>
    </detail>
</jasperReport>
```

现在，让我们修改员工报告以包括前一个报表：

```xml
<detail>
    <band ... >
        <subreport>
            <reportElement x="0" y="20" width="300" height="27"/>
            <subreportParameter name="idEmployee">
                <subreportParameterExpression>
                  <![CDATA[$F{ID}]]></subreportParameterExpression>
            </subreportParameter>
            <connectionExpression>
              <![CDATA[$P{REPORT_CONNECTION}]]></connectionExpression>
            <subreportExpression class="java.lang.String">
              <![CDATA["employeeEmailReport.jasper"]]></subreportExpression>
        </subreport>
    </band>
</detail>
```

请注意，我们通过编译文件的名称引用子报表，并将idEmployee和当前报表连接作为参数传递给它。

接下来，让我们编译两个报表：

```java
InputStream employeeReportStream = getClass().getResourceAsStream("/employeeReport.jrxml");
JasperReport jasperReport = JasperCompileManager.compileReport(employeeReportStream);
JRSaver.saveObject(jasperReport, "employeeReport.jasper");

InputStream emailReportStream = getClass().getResourceAsStream("/employeeEmailReport.jrxml");
JRSaver.saveObject(
    JasperCompileManager.compileReport(emailReportStream),
    "employeeEmailReport.jasper");
```

我们用于填写和导出报告的代码不需要修改。

## 7. 使用printWhenExpression进行条件显示

此外，我们可以使用printWhenExpression根据特定条件有条件地显示报表元素，这意味着可以根据报表中的数据或参数动态显示或隐藏文本字段、图像和带区等元素。

下面是一个如何使用printWhenExpression修改JRXML文件以包含空值检查的示例。在详细信息带中呈现内容之前，我们将检查FIRST_NAME、LAST_NAME和SALARY字段中的非空值：

```xml
<jasperReport ... >
    <field name="SALARY" class="java.lang.Double"/>
    <!-- other fields -->
    
    <detail>
        <band height="51" splitType="Stretch">
            <printWhenExpression><![CDATA[$F{FIRST_NAME} != null && $F{LAST_NAME} != null && $F{SALARY} != null]]></printWhenExpression>

            <!-- Existing text fields and subreport-->
        
        </band>
    </detail>
</jasperReport>
```

此表达式确保只有当所有这些字段都具有有效(非空)值时，才会显示带区的全部内容。更新JRXML文件后，我们需要像以前一样编译和填充报告。

## 8. 总结

在本文中，我们简要介绍了JasperReports库的核心功能。

我们能够使用数据库中的记录来编译和填充报告；我们传递参数以根据不同的运行时条件更改报表中显示的数据，嵌入子报表并将它们导出为最常见的格式。