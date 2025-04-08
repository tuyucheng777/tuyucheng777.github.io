---
layout: post
title:  使用Java中的Play框架开发REST API
category: webmodules
copyright: webmodules
excerpt: Play
---

## 1. 概述

本教程的目的是探索Play框架并学习如何使用Java构建REST服务。

我们将组合一个REST API来创建、检索、更新和删除学生记录。

在这样的应用中，我们通常会有一个数据库来存储学生记录。Play Framework有一个内置的H2数据库，并支持JPA与Hibernate和其他持久层框架。

但是，为了简单起见并专注于最重要的内容，我们将使用一个简单的Map来存储具有唯一ID的学生对象。

## 2. 创建新的应用程序

一旦我们按照Play框架[简介](https://www.baeldung.com/java-intro-to-the-play-framework)中所述安装了Play框架，我们就可以创建我们的应用程序了。

让我们使用sbt命令使用play-java-seed创建一个名为student-api的新应用程序：

```shell
sbt new playframework/play-java-seed.g8
```

## 3. 模型

在我们的应用程序脚手架创建后，让我们导航到student-api/app/models并创建一个用于处理学生信息的Java Bean：

```java
public class Student {
    private String firstName;
    private String lastName;
    private int age;
    private int id;

    // standard constructors, getters and setters
}
```

我们现在将为学生数据创建一个由HashMap支持的简单数据存储，并使用辅助方法来执行CRUD操作：

```java
public class StudentStore {
    private Map<Integer, Student> students = new HashMap<>();

    public Optional<Student> addStudent(Student student) {
        int id = students.size();
        student.setId(id);
        students.put(id, student);
        return Optional.ofNullable(student);
    }

    public Optional<Student> getStudent(int id) {
        return Optional.ofNullable(students.get(id));
    }

    public Set<Student> getAllStudents() {
        return new HashSet<>(students.values());
    }

    public Optional<Student> updateStudent(Student student) {
        int id = student.getId();
        if (students.containsKey(id)) {
            students.put(id, student);
            return Optional.ofNullable(student);
        }
        return null;
    }

    public boolean deleteStudent(int id) {
        return students.remove(id) != null;
    }
}
```

## 4. 控制器

让我们转到student-api/app/controllers并创建一个名为StudentController.java的新控制器，我们将逐步完成代码。

首先，我们需要配置一个HttpExecutionContext。我们将使用异步、非阻塞代码实现我们的操作；这意味着我们的操作方法将返回CompletionStage<Result\>而不仅仅是Result。这样做的好处是，我们可以编写长时间运行的任务而不会发生阻塞。

在Play框架控制器中处理异步编程时，只有一个注意事项：我们必须提供HttpExecutionContext。如果我们不提供HTTP执行上下文，则在调用操作方法时，我们会收到臭名昭著的错误“There is no HTTP Context available from here”。

让我们注入它：

```java
private HttpExecutionContext ec;
private StudentStore studentStore;

@Inject
public StudentController(HttpExecutionContext ec, StudentStore studentStore) {
    this.studentStore = studentStore;
    this.ec = ec;
}
```

请注意，我们还添加了StudentStore，并使用@Inject注解在控制器的构造函数中注入了两个字段。完成此操作后，我们现在可以继续实现操作方法。

请注意，Play附带Jackson以允许数据处理-因此我们可以导入我们需要的任何Jackson类，而无需外部依赖。

让我们定义一个实用程序类来执行重复操作；在本例中，构建HTTP响应。

因此让我们创建student-api/app/utils包并在其中添加Util.java：

```java
public class Util {
    public static ObjectNode createResponse(Object response, boolean ok) {
        ObjectNode result = Json.newObject();
        result.put("isSuccessful", ok);
        if (response instanceof String) {
            result.put("body", (String) response);
        } else {
            result.putPOJO("body", response);
        }
        return result;
    }
}
```

使用此方法，我们将使用布尔isSuccessful键和响应主体创建标准JSON响应。

我们现在可以逐步完成控制器类的操作。

### 4.1 创建操作

映射为POST操作，此方法处理Student对象的创建：

```java
public CompletionStage<Result> create(Http.Request request) {
    JsonNode json = request.body().asJson();
    return supplyAsync(() -> {
        if (json == null) {
            return badRequest(Util.createResponse("Expecting Json data", false));
        }

        Optional<Student> studentOptional = studentStore.addStudent(Json.fromJson(json, Student.class));
        return studentOptional.map(student -> {
            JsonNode jsonObject = Json.toJson(student);
            return created(Util.createResponse(jsonObject, true));
        }).orElse(internalServerError(Util.createResponse("Could not create data.", false)));
    }, ec.current());
}
```

我们使用注入的Http.Request类中的调用将请求主体放入Jackson的JsonNode类中，请注意，如果主体为null，我们如何使用实用方法来创建响应。

我们还返回一个CompletionStage<Result\>，这使我们能够使用CompletedFuture.supplyAsync方法编写非阻塞代码。

我们可以向它传递任何String或JsonNode以及boolean标志来指示状态。

还要注意我们如何使用Json.fromJson()将传入的JSON对象转换为Student对象，然后将其转换回JSON以进行响应。

最后，我们使用play.mvc.results包中的created辅助方法，而不是我们习惯的ok()，这个想法是使用一种方法来为特定上下文中执行的操作提供正确的HTTP状态。例如，ok()表示HTTP OK 200状态，而created()表示HTTP CREATED 201-是上述结果状态；这个概念将贯穿其余操作。

### 4.2 更新操作

对http://localhost:9000/的PUT请求命中StudentController.update方法，该方法通过调用StudentStore的updateStudent方法来更新学生信息：

```java
public CompletionStage<Result> update(Http.Request request) {
    JsonNode json = request.body().asJson();
    return supplyAsync(() -> {
        if (json == null) {
            return badRequest(Util.createResponse("Expecting Json data", false));
        }
        Optional<Student> studentOptional = studentStore.updateStudent(Json.fromJson(json, Student.class));
        return studentOptional.map(student -> {
            if (student == null) {
                return notFound(Util.createResponse("Student not found", false));
            }
            JsonNode jsonObject = Json.toJson(student);
            return ok(Util.createResponse(jsonObject, true));
        }).orElse(internalServerError(Util.createResponse("Could not create data.", false)));
    }, ec.current());
}
```

### 4.3 检索操作

为了检索学生，我们将学生的id作为路径参数传入GET请求中，地址为http://localhost:9000/:id；这将触发retrieve操作：

```java
public CompletionStage<Result> retrieve(int id) {
    return supplyAsync(() -> {
        final Optional<Student> studentOptional = studentStore.getStudent(id);
        return studentOptional.map(student -> {
            JsonNode jsonObjects = Json.toJson(student);
            return ok(Util.createResponse(jsonObjects, true));
        }).orElse(notFound(Util.createResponse("Student with id:" + id + " not found", false)));
    }, ec.current());
}
```

### 4.4 删除操作

delete操作映射到http://localhost:9000/:id，我们提供id来标识要删除的记录：

```java
public CompletionStage<Result> delete(int id) {
    return supplyAsync(() -> {
        boolean status = studentStore.deleteStudent(id);
        if (!status) {
            return notFound(Util.createResponse("Student with id:" + id + " not found", false));
        }
        return ok(Util.createResponse("Student with id:" + id + " deleted", true));
    }, ec.current());
}
```

### 4.5 listStudents操作

最后， listStudents操作返回迄今为止已存储的所有学生的列表，它作为GET请求映射到http://localhost:9000/：

```java
public CompletionStage<Result> listStudents() {
    return supplyAsync(() -> {
        Set<Student> result = studentStore.getAllStudents();
        ObjectMapper mapper = new ObjectMapper();
        JsonNode jsonData = mapper.convertValue(result, JsonNode.class);
        return ok(Util.createResponse(jsonData, true));
    }, ec.current());
}
```

## 5. 映射

设置好控制器操作后，我们现在可以通过打开文件student-api/conf/routes并添加以下路由来映射它们：

```text
GET     /                           controllers.StudentController.listStudents()
GET     /:id                        controllers.StudentController.retrieve(id:Int)
POST    /                           controllers.StudentController.create(request: Request)
PUT     /                           controllers.StudentController.update(request: Request)
DELETE  /:id                        controllers.StudentController.delete(id:Int)
GET     /assets/*file               controllers.Assets.versioned(path="/public", file: Asset)
```

**/assets端点必须始终存在，以便下载静态资源**。

此后，我们就完成了Student API的构建。

要了解有关定义路由映射的更多信息，请访问我们的[Play应用程序中的路由](https://www.baeldung.com/routing-in-play)教程。

## 6. 测试

现在，我们可以通过向http://localhost:9000/发送请求并添加适当的上下文来对我们的API进行测试；从浏览器运行基本路径应该输出：

```json
{
     "isSuccessful":true,
     "body":[]
}
```

我们可以看到，由于我们尚未添加任何记录，因此主体是空的。使用curl，让我们运行一些测试(或者，我们可以使用像Postman这样的REST客户端)。

让我们打开一个终端窗口并执行curl命令来添加一名学生：

```shell
curl -X POST -H "Content-Type: application/json" \
 -d '{"firstName":"John","lastName":"Tuyucheng","age": 18}' \
 http://localhost:9000/
```

这将返回新创建的学生：

```json
{
    "isSuccessful":true,
    "body":{
        "firstName":"John",
        "lastName":"Tuyucheng",
        "age":18,
        "id":0
    }
}
```

运行上述测试后，从浏览器加载http://localhost:9000现在应该会得到：

```json
{ 
    "isSuccessful":true,
    "body":[ 
        { 
            "firstName":"John",
            "lastName":"Tuyucheng",
            "age":18,
            "id":0
        }
    ]
}
```

每添加一条新记录，id属性就会增加。

要删除记录，我们发送DELETE请求：

```shell
curl -X DELETE http://localhost:9000/0
{ 
    "isSuccessful":true,
    "body":"Student with id:0 deleted"
}
```

在上面的测试中，我们删除了第一个测试中创建的记录，现在让我们再次创建它，以便我们可以测试更新方法：

```shell
curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"John","lastName":"Tuyucheng","age": 18}' \
http://localhost:9000/
{ 
    "isSuccessful":true,
    "body":{ 
        "firstName":"John",
        "lastName":"Tuyucheng",
        "age":18,
        "id":0
    }
}
```

现在让我们通过将firstName设置为“Andrew”并将age设置为30来更新记录：

```shell
curl -X PUT -H "Content-Type: application/json" \
-d '{"firstName":"Andrew","lastName":"Tuyucheng","age": 30,"id":0}' \
http://localhost:9000/
{ 
    "isSuccessful":true,
    "body":{ 
        "firstName":"Andrew",
        "lastName":"Tuyucheng",
        "age":30,
        "id":0
    }
}
```

上述测试演示了更新记录后firstName和age字段值的变化。

让我们创建一些额外的虚拟记录，我们将添加两个：John Doe和Sam Tuyucheng：

```shell
curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"John","lastName":"Doe","age": 18}' \
http://localhost:9000/
```

```shell
curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"Sam","lastName":"Tuyucheng","age": 25}' \
http://localhost:9000/
```

现在，让我们获取所有记录：

```shell
curl -X GET http://localhost:9000/
{ 
    "isSuccessful":true,
    "body":[ 
        { 
            "firstName":"Andrew",
            "lastName":"Tuyucheng",
            "age":30,
            "id":0
        },
        { 
            "firstName":"John",
            "lastName":"Doe",
            "age":18,
            "id":1
        },
        { 
            "firstName":"Sam",
            "lastName":"Tuyucheng",
            "age":25,
            "id":2
        }
    ]
}
```

通过上述测试，我们确定listStudents控制器操作是否正常运行。

## 7. 总结

在本文中，我们展示了如何使用Play框架构建成熟的REST API。