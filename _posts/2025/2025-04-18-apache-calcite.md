---
layout: post
title:  Apache Calcite简介
category: persistence
copyright: persistence
excerpt: Apache Calcite
---

## 1. 概述

在本教程中，我们将学习[Apache Calcite](https://calcite.apache.org/)，**它是一个强大的数据管理框架，可用于各种数据访问场景**。Calcite专注于从任何来源检索数据，而不是存储数据。此外，它的查询优化功能可以实现更快、更高效的数据检索。

让我们深入了解更多细节，从Apache Calcite相关的用例开始。

## 2. Apache Calcite用例

由于其功能，Apache Calcite可以在多种用例中得到利用：

![](/assets/images/2025/persistence/springboot31connectiondetailsabstraction01.png)

为新数据库构建查询引擎需要数年时间，但是，Calcite提供了一个开箱即用的可扩展[SQL](https://www.baeldung.com/cs/sql-statements-queries)解析器、验证器和优化器，可以帮助我们立即上手。Calcite已用于构建数据库，例如[HerdDB](https://herddb.org/)、[Apache Druid](https://druid.apache.org/)、[MapD](https://github.com/wesm/mapd-core)等等。

由于Calcite具有与多种数据库集成的能力，因此它被广泛用于构建数据仓库和商业智能工具，例如[Apache Kyline](https://kylin.apache.org/)、[Apache Wayang](https://wayang.apache.org/)、[Alibaba MaxCompute](https://calcite.apache.org/docs/powered_by.html#alibaba-maxcompute)等等。

Calcite是流媒体平台(例如[Apache Kafka](https://www.baeldung.com/apache-kafka)、[Apache Apex](https://apex.apache.org/)和[Flink](https://www.baeldung.com/apache-flink) )不可或缺的组件，它们有助于构建可以呈现和分析实时信息的工具。

## 3. 任何数据，任何地点

**Apache Calcite提供现成的[适配器](https://calcite.apache.org/docs/adapter.html)来与第三方数据源集成，包括[Cassandra](https://www.baeldung.com/cassandra-with-java)、[Elasticsearch](https://www.baeldung.com/elasticsearch-java)、[MongoDB](https://www.baeldung.com/java-mongodb)等**。

让我们更详细地探讨这个问题。

### 3.1 高级重要类

Apache Calcite提供了一个强大的数据检索框架，该框架可扩展，因此可以创建自定义的新适配器；我们来看看其中重要的Java类：

![](/assets/images/2025/persistence/springboot31connectiondetailsabstraction02.png)

Apache Calcite适配器提供了一些类，例如[ElasticsearchSchemaFactory](https://calcite.apache.org/javadocAggregate/org/apache/calcite/adapter/elasticsearch/ElasticsearchSchemaFactory.html)、[MongoSchemaFactory](https://calcite.apache.org/javadocAggregate/org/apache/calcite/adapter/mongodb/MongoSchemaFactory.html)、[FileSchemaFactory，](https://calcite.apache.org/javadocAggregate/org/apache/calcite/adapter/file/FileSchemaFactory.html)它们实现了[SchemaFactory](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/SchemaFactory.html)接口。**SchemaFactory通过在[JSON/YAML模型文件](https://calcite.apache.org/docs/model.html)中定义虚拟[Schema](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/Schema.html)，以统一的方式连接底层数据源**。

### 3.2 CSV适配器

现在让我们看一个例子，我们将使用SQL查询从CSV文件读取数据。首先，在pom.xml文件中导入使用文件适配器所需的必要[Maven依赖](https://mvnrepository.com/artifact/org.apache.calcite)：

```xml
<dependency>
    <groupId>org.apache.calcite</groupId>
    <artifactId>calcite-core</artifactId>
    <version>1.36.0</version>
</dependency>
<dependency>
    <groupId>org.apache.calcite</groupId>
    <artifactId>calcite-file</artifactId>
    <version>1.36.0</version>
</dependency>
```

接下来，让我们在model.json中定义模型：

```json
{
    "version": "1.0",
    "defaultSchema": "TRADES",
    "schemas": [
        {
            "name": "TRADES",
            "type": "custom",
            "factory": "org.apache.calcite.adapter.file.FileSchemaFactory",
            "operand": {
                "directory": "trades"
            }
        }
    ]
}
```

**model.json中指定的FileSchemaFactory会在trades目录中查找CSV文件，并创建一个虚拟的TRADES模式**。随后，trades目录下的CSV文件将被视为表格。

在我们继续查看文件适配器的运行之前，让我们先看一下trade.csv文件，我们将使用calcite适配器进行查询：

```csv
tradeid:int,product:string,qty:int
232312123,"RFTXC",100
232312124,"RFUXC",200
232312125,"RFSXC",1000
```

该CSV文件包含三列：tradeid、product和qty。此外，列标头还指定了数据类型，CSV文件中总共有3条事务记录。

最后，让我们看看如何使用Calcite适配器获取记录：

```java
@Test
void whenCsvSchema_thenQuerySuccess() throws SQLException {
    Properties info = new Properties();
    info.put("model", getPath("model.json"));
    try (Connection connection = DriverManager.getConnection("jdbc:calcite:", info);) {
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("select * from trades.trade");

        assertEquals(3, resultSet.getMetaData().getColumnCount());

        List<Integer> tradeIds = new ArrayList<>();
        while (resultSet.next()) {
            tradeIds.add(resultSet.getInt("tradeid"));
        }

        assertEquals(3, tradeIds.size());
    }
}
```

Calcite适配器利用model属性创建一个模拟文件系统的虚拟模式，然后，它使用常见的JDBC语义从trade.csv文件中提取记录。

文件适配器不仅可以读取CSV文件，还可以读取[HTML和JSON文件](https://calcite.apache.org/docs/file_adapter.html)。此外，为了处理CSV文件，Apache Calcite还提供了一个特殊的CSV适配器，用于处理使用[CSVSchemaFactory](https://calcite.apache.org/docs/tutorial.html)的高级用例。

### 3.3 Java对象的内存SQL操作

与CSV适配器示例类似，让我们看另一个示例，在Apache Calcite的帮助下，我们将对Java对象运行SQL查询。

假设CompanySchema类中有两个Employee和Department类的数组：

```java
public class CompanySchema {
    public Employee[] employees;
    public Department[] departments;
}
```

现在让我们看一下Employee类：

```java
public class Employee {
    public String name;
    public String id;

    public String deptId;

    public Employee(String name, String id, String deptId) {
        this.name = name;
        this.id = id;
        this.deptId = deptId;
    }
}
```

与Employee类类似，我们来定义Department类：

```java
public class Department {
    public String deptId;
    public String deptName;

    public Department(String deptId, String deptName) {
        this.deptId = deptId;
        this.deptName = deptName;
    }
}
```

假设有3个部门：财务、市场营销和人力资源，我们将对CompanySchema对象运行查询，以查找每个部门的员工人数：

```java
@Test
void whenQueryEmployeesObject_thenSuccess() throws SQLException {
    Properties info = new Properties();
    info.setProperty("lex", "JAVA");
    Connection connection = DriverManager.getConnection("jdbc:calcite:", info);
    CalciteConnection calciteConnection = connection.unwrap(CalciteConnection.class);
    SchemaPlus rootSchema = calciteConnection.getRootSchema();
    Schema schema = new ReflectiveSchema(companySchema);
    rootSchema.add("company", schema);
    Statement statement = calciteConnection.createStatement();
    String query = "select dept.deptName, count(emp.id) "
            + "from company.employees as emp "
            + "join company.departments as dept "
            + "on (emp.deptId = dept.deptId) "
            + "group by dept.deptName";

    assertDoesNotThrow(() -> {
        ResultSet resultSet = statement.executeQuery(query);
        while (resultSet.next()) {
            logger.info("Dept Name:" + resultSet.getString(1) + " No. of employees:" + resultSet.getInt(2));
        }
    });
}
```

有趣的是，该方法运行良好，并能获取结果。在该方法中，Apache Calcite类[ReflectiveSchema](https://calcite.apache.org/javadocAggregate/org/apache/calcite/adapter/java/ReflectiveSchema.html)帮助创建CompanySchema对象的Schema，然后，它运行SQL查询并使用标准JDBC语义获取记录。

**这个例子证明，无论来源如何，Calcite都可以使用SQL语句从任何地方获取数据**。

## 4. 查询处理

查询处理是Apache calcite的核心功能。

标准JDBC驱动程序或SQL客户端会对数据库执行查询，相比之下，**Apache Calcite在解析和验证查询后，会对其进行智能优化，以实现高效执行，从而节省资源并提升性能**。

### 4.1 解码查询处理步骤

Calcite提供了非常标准的组件来帮助查询处理：

![](/assets/images/2025/persistence/springboot31connectiondetailsabstraction03.png)

有趣的是，我们还可以扩展这些组件以满足任何数据库的特定需求，让我们详细了解这些步骤。

### 4.2 SQL解析器和验证器

**作为解析过程的一部分，解析器将SQL查询转换为称为AST(抽象语法树)的树状结构**。

假设对两个表Teacher和Department进行SQL查询：

```sql
Select Teacher.name, Department.name 
From Teacher join 
Department On (Department.deptid = Teacher.deptid)
Where Department.name = 'Science'
```

首先，查询解析器将查询转换为AST，然后执行基本的语法验证：

![](/assets/images/2025/persistence/springboot31connectiondetailsabstraction04.png)

此外，**验证器从语义上验证节点**：

- 验证函数和运算符
- 根据数据库目录验证数据库对象(如表和列)

### 4.3 关系表达式构建器

随后，在验证步骤之后，关系表达式构建器使用一些常见的关系运算符转换语法树：

- LogicalTableScan：从表中读取数据
- LogicalFilter：根据条件选择行
- LogicalProject：选择要包含的特定列
- LogicalJoin：根据匹配值合并两个表中的行

考虑到前面显示的AST，从中得出的相应逻辑关系表达式将是：

```text
LogicalProject(
    projects=[
        $0.name AS name0,
        $1.name AS name1
    ],
    input=LogicalFilter(
        condition=[
            ($1.name = 'Science')
        ],
        input=LogicalJoin(
            condition=[
                ($0.deptid = $1.deptid)
            ],
            left=LogicalTableScan(table=[[Teacher]]),
            right=LogicalTableScan(table=[[Department]])
        )
    )
)
```

在关系表达式中，\$0和\$1分别代表Teacher表和Department表。本质上，**它是一个数学表达式，有助于理解需要执行哪些操作才能获得结果**。但是，它没有与执行相关的信息。

### 4.4 查询优化器

然后，Calcite优化器对关系表达式进行优化，一些常见的优化包括：

- 谓词下推：将过滤器尽可能推近数据源，以减少获取的数据量
- 连接重新排序：重新排列连接顺序以最小化中间结果并提高效率
- 投影下推：下推投影以避免处理不必要的列
- 索引使用：识别和利用索引来加速数据检索

### 4.5 查询规划器、生成器和执行器

优化之后，Calcite查询规划器会创建一个执行计划来执行优化后的查询。**该执行计划指定了查询引擎获取和处理数据的具体步骤**，这也称为特定于后端查询引擎的物理计划。

然后，**Calcite查询生成器以特定于所选执行引擎的语言生成代码**。

最后，执行器连接到数据库来执行最终的查询。

## 5. 总结

在本文中，我们探讨了Apache Calcite的功能，它能够快速为数据库配备标准化的SQL解析器、验证器和优化器。这使得供应商无需再耗费数年时间开发查询引擎，从而能够优先考虑后端存储。此外，Calcite的现成适配器简化了与不同数据库的连接，有助于开发统一的集成接口。

此外，通过利用Calcite，数据库开发人员可以加快产品上市时间，并提供强大、多功能的SQL功能。