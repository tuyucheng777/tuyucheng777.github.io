---
layout: post
title:  使用MapReduce视图查询Couchbase
category: persistence
copyright: persistence
excerpt: Couchbase
---

## 1. 概述

在本教程中，我们将介绍一些简单的MapReduce视图并演示如何使用[Couchbase Java SDK](https://docs.couchbase.com/java-sdk/current/hello-world/start-using-sdk.html)查询它们。

## 2. Maven依赖

要在Maven项目中使用Couchbase，请将Couchbase SDK导入到你的pom.xml中：

```xml
<dependency>
    <groupId>com.couchbase.client</groupId>
    <artifactId>java-client</artifactId>
    <version>2.7.2</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/com.couchbase.client/java-client)上找到最新版本。

## 3. MapReduce视图

在Couchbase中，MapReduce视图是一种可用于查询数据存储桶的索引，它使用JavaScript的map函数和可选的reduce函数进行定义。

### 3.1 map函数

map函数会针对每个文档运行一次，创建视图时，会针对存储桶中的每个文档运行一次map函数，并将结果存储在存储桶中。

一旦创建了视图，map函数仅针对新插入或更新的文档运行，以便逐步更新视图。

由于map函数的结果存储在数据存储桶中，因此针对视图的查询表现出较低的延迟。

让我们看一个map函数的例子，该函数在存储桶中所有type字段等于“StudentGrade”的文档的name字段上创建索引：

```javascript
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.name) {    
        emit(doc.name, null);
    }
}
```

emit函数告诉Couchbase在索引键中存储哪些数据字段(第一个参数)以及与索引文档关联的值(第二个参数)。

在本例中，我们只在索引键中存储文档名称属性。由于我们不想将任何特定值与每个条目关联，因此我们将null作为值参数传递。

当Couchbase处理视图时，它会创建由map函数发出的键的索引，并将每个键与发出该键的所有文档关联起来。

例如，如果3个文档的name属性设置为“John Doe”，则索引键“John Doe”将与这3个文档相关联。

### 3.2 reduce函数

Reduce函数用于对Map函数的结果进行聚合计算，Couchbase管理界面提供了一种简单的方法，可以将内置的Reduce函数“_count”、“_sum”和“_stats”应用于Map函数。

你还可以编写自己的Reduce函数来实现更复杂的聚合，本教程后面将介绍如何使用内置Reduce函数的示例。

## 4. 使用视图和查询

### 4.1 组织视图

每个存储桶会将视图组织成一个或多个设计文档，理论上，每个设计文档的视图数量没有限制。但是，为了获得最佳性能，建议你将每个设计文档的视图数量限制在10个以内。

当你首次在设计文档中创建视图时，Couchbase会将其指定为开发视图，你可以针对开发视图运行查询来测试其功能。一旦你对视图感到满意，就可以发布设计文档，该视图将成为生产视图。

### 4.2 构建查询

为了构建针对Couchbase视图的查询，你需要提供其设计文档名称和视图名称来创建ViewQuery对象：

```java
ViewQuery query = ViewQuery.from("design-document-name", "view-name");
```

执行时，此查询将返回视图的所有行，我们将在后面的部分中看到如何根据键值限制结果集。

要针对开发视图构建查询，可以在创建查询时应用development()方法：

```java
ViewQuery query = ViewQuery.from("design-doc-name", "view-name").development();
```

### 4.3 执行查询

一旦我们有了ViewQuery对象，就可以执行查询来获取ViewResult：

```java
ViewResult result = bucket.query(query);
```

### 4.4 处理查询结果

现在我们有了ViewResult，可以遍历行来获取文档ID和/或内容：

```java
for(ViewRow row : result.allRows()) {
    JsonDocument doc = row.document();
    String id = doc.id();
    String json = doc.content().toString();
}
```

## 5. 示例应用程序

在本教程的剩余部分，我们将为具有以下格式的一组学生成绩文档编写MapReduce视图和查询，成绩范围限制在0到100之间：

```json
{ 
    "type": "StudentGrade",
    "name": "John Doe",
    "course": "History",
    "hours": 3,
    "grade": 95
}
```

我们将这些文档存储在“tuyucheng-tutorial”存储桶中，并将所有视图存储在名为“studentGrades”的设计文档中；让我们看一下打开存储桶以便查询它所需的代码：

```java
Bucket bucket = CouchbaseCluster.create("127.0.0.1")
    .openBucket("tuyucheng-tutorial");
```

## 6. 精确匹配查询

假设你想查找某一门或一组课程的所有学生成绩，让我们使用以下map函数编写一个名为“findByCourse”的视图：

```javascript
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.course && doc.grade) {
        emit(doc.course, null);
    }
}
```

请注意，在这个简单的视图中，我们只需要发出course字段。

### 6.1 单个键的匹配

为了查找历史课程的所有成绩，我们将key方法应用于基本查询：

```java
ViewQuery query = ViewQuery.from("studentGrades", "findByCourse").key("History");
```

### 6.2 多个键匹配

如果要查找数学和科学课程的所有成绩，可以将keys方法应用于基本查询，并向其传递一个键值数组：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "findByCourse")
    .keys(JsonArray.from("Math", "Science"));
```

## 7. 范围查询

为了查询包含一个或多个字段的一系列值的文档，我们需要一个发出我们感兴趣的字段的视图，并且必须为查询指定下限和/或上限。

我们来看看如何进行单字段、多字段的范围查询。

### 7.1 涉及单个字段的查询

为了查找所有包含一定范围内成绩值(无论course字段的值如何)的文档，我们需要一个只输出grade字段的视图，让我们为“findByGrade”视图编写map函数：

```javascript
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.grade) {
        emit(doc.grade, null);
    }
}
```

让我们使用此视图在Java中编写一个查询来查找所有相当于“B”字母等级的成绩(包括80到89)：

```java
ViewQuery query = ViewQuery.from("studentGrades", "findByGrade")
    .startKey(80)
    .endKey(89)
    .inclusiveEnd(true);
```

请注意，范围查询中的起始键值始终被视为包含在内。

如果已知所有成绩都是整数，则以下查询将产生相同的结果：

```java
ViewQuery query = ViewQuery.from("studentGrades", "findByGrade")
    .startKey(80)
    .endKey(90)
    .inclusiveEnd(false);
```

要查找所有“A”级(90及以上)，我们只需要指定下限：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "findByGrade")
    .startKey(90);
```

为了找出所有不及格的成绩(低于60分)，我们只需要指定上限：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "findByGrade")
    .endKey(60)
    .inclusiveEnd(false);
```

### 7.2 涉及多个字段的查询

现在，假设我们想找出特定课程中成绩在特定范围内的所有学生，此查询需要一个同时包含course和grade字段的新视图。

在多字段视图中，每个索引键都会以值数组的形式发出。由于我们的查询涉及course的固定值和grade值的范围，因此我们将编写map函数，将每个键以[course, grade\]形式的数组发出。

让我们看一下视图“findByCourseAndGrade”的map函数：

```javascript
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.course && doc.grade) {
        emit([doc.course, doc.grade], null);
    }
}
```

当此视图在Couchbase中填充时，索引条目会按course和grade排序，以下是“findByCourseAndGrade”视图中按自然排序显示的部分键子集：

```text
["History", 80]
["History", 90]
["History", 94]
["Math", 82]
["Math", 88]
["Math", 97]
["Science", 78]
["Science", 86]
["Science", 92]
```

由于此视图中的键是数组，因此在针对此视图指定范围查询的下限和上限时，你也将使用此格式的数组。

这意味着，为了找到所有在数学课程中获得“B”级(80到89)的学生，你需要将下限设置为：

```text
["Math", 80]
```

其上限为：

```text
["Math", 89]
```

让我们用Java编写范围查询：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "findByCourseAndGrade")
    .startKey(JsonArray.from("Math", 80))
    .endKey(JsonArray.from("Math", 89))
    .inclusiveEnd(true);
```

如果我们想找出所有数学成绩为“A”(90分及以上)的学生，那么我们可以这样写：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "findByCourseAndGrade")
    .startKey(JsonArray.from("Math", 90))
    .endKey(JsonArray.from("Math", 100));
```

请注意，由于我们将课程值固定为“Math”，因此必须包含一个最高可能成绩值的上限。否则，我们的结果集将包含所有课程值按字典顺序大于“Math”的文档。

查找所有不及格的数学成绩(低于60分)：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "findByCourseAndGrade")
    .startKey(JsonArray.from("Math", 0))
    .endKey(JsonArray.from("Math", 60))
    .inclusiveEnd(false);
```

与前面的例子类似，我们必须指定一个最低可能的成绩下限。否则，我们的结果集将包含所有course值按字典顺序小于“Math”的成绩。

最后，为了找到数学成绩最高的5个(除非有平分)，你可以告诉Couchbase执行降序排序并限制结果集的大小：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "findByCourseAndGrade")
    .descending()
    .startKey(JsonArray.from("Math", 100))
    .endKey(JsonArray.from("Math", 0))
    .inclusiveEnd(true)
    .limit(5);
```

请注意，执行降序排序时，startKey和endKey值会反转，因为Couchbase在应用limit之前应用了排序。

## 8. 聚合查询

MapReduce视图的一大优势在于，它们能够高效地针对大型数据集运行聚合查询。例如，在我们的学生成绩数据集中，我们可以轻松计算以下聚合：

- 每门课程的学生人数
- 每个学生的学分总和
- 每个学生所有课程的平均绩点

让我们使用内置的reduce函数为每个计算构建一个视图并进行查询。

### 8.1 使用count()函数

首先，让我们为视图编写map函数来计算每门课程的学生人数：

```javascript
function (doc, meta) {
    if(doc.type == "StudentGrade" && doc.course && doc.name) {
        emit([doc.course, doc.name], null);
    }
}
```

我们将此视图命名为“countStudentsByCourse”，并指定它使用内置的“_count”函数。由于我们只执行简单的计数，因此我们仍然可以为每个条目发出null作为值。

统计每门课程的学生人数：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "countStudentsByCourse")
    .reduce()
    .groupLevel(1);
```

从聚合查询中提取数据与我们之前所见的不同，我们不再为结果中的每一行提取匹配的Couchbase文档，而是提取聚合键和结果。

让我们运行查询并将计数提取到java.util.Map中：

```java
ViewResult result = bucket.query(query);
Map<String, Long> numStudentsByCourse = new HashMap<>();
for(ViewRow row : result.allRows()) {
    JsonArray keyArray = (JsonArray) row.key();
    String course = keyArray.getString(0);
    long count = Long.valueOf(row.value().toString());
    numStudentsByCourse.put(course, count);
}
```

### 8.2 使用sum()函数

接下来，让我们编写一个视图，计算每个学生已修学分的总和。我们将此视图命名为“sumHoursByStudent”，并指定它使用内置的“_sum”函数：

```javascript
function (doc, meta) {
    if(doc.type == "StudentGrade"
         && doc.name
         && doc.course
         && doc.hours) {
        emit([doc.name, doc.course], doc.hours);
    }
}
```

请注意，在应用“_sum”函数时，我们必须为每个条目发出要求和的值-在本例中为积分数。

让我们编写一个查询来查找每个学生的总学分数：

```java
ViewQuery query = ViewQuery
    .from("studentGrades", "sumCreditsByStudent")
    .reduce()
    .groupLevel(1);
```

现在，让我们运行查询并将聚合总和提取到java.util.Map中：

```java
ViewResult result = bucket.query(query);
Map<String, Long> hoursByStudent = new HashMap<>();
for(ViewRow row : result.allRows()) {
    String name = (String) row.key();
    long sum = Long.valueOf(row.value().toString());
    hoursByStudent.put(name, sum);
}
```

### 8.3 计算平均绩点

假设我们要计算每个学生所有课程的平均绩点(GPA)，使用基于所得成绩和课程学分数的传统绩点标准(A = 每学分4分，B = 每学分3分，C = 每学分2分，D = 每学分1分)。

没有内置的reduce函数来计算平均值，所以我们将结合两个视图的输出来计算GPA。

我们已经有了“sumHoursByStudent”视图，用于统计每个学生修读的学分时长，现在我们需要计算每个学生获得的总绩点。

让我们创建一个名为“sumGradePointsByStudent”的视图，用于计算每门课程获得的绩点，我们将使用内置的“_sum”函数来简化以下map函数：

```javascript
function (doc, meta) {
    if(doc.type == "StudentGrade"
         && doc.name
         && doc.hours
         && doc.grade) {
        if(doc.grade >= 90) {
            emit(doc.name, 4*doc.hours);
        }
        else if(doc.grade >= 80) {
            emit(doc.name, 3*doc.hours);
        }
        else if(doc.grade >= 70) {
            emit(doc.name, 2*doc.hours);
        }
        else if(doc.grade >= 60) {
            emit(doc.name, doc.hours);
        }
        else {
            emit(doc.name, 0);
        }
    }
}
```

现在让我们查询此视图并将总和提取到java.util.Map中：

```java
ViewQuery query = ViewQuery.from(
    "studentGrades",
    "sumGradePointsByStudent")
    .reduce()
    .groupLevel(1);
ViewResult result = bucket.query(query);

Map<String, Long> gradePointsByStudent = new HashMap<>();
for(ViewRow row : result.allRows()) {
    String course = (String) row.key();
    long sum = Long.valueOf(row.value().toString());
    gradePointsByStudent.put(course, sum);
}
```

最后，让我们将两个Map结合起来，计算每个学生的GPA：

```java
Map<String, Float> result = new HashMap<>();
for(Entry<String, Long> creditHoursEntry : hoursByStudent.entrySet()) {
    String name = creditHoursEntry.getKey();
    long totalHours = creditHoursEntry.getValue();
    long totalGradePoints = gradePointsByStudent.get(name);
    result.put(name, ((float) totalGradePoints / totalHours));
}
```

## 9. 总结

我们演示了如何在Couchbase中编写一些基本的MapReduce视图，以及如何构建和执行针对视图的查询并提取结果。