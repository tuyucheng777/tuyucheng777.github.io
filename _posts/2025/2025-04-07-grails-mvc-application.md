---
layout: post
title:  使用Grails构建MVC Web应用程序
category: webmodules
copyright: webmodules
excerpt: Grails
---

## 1. 概述

在本教程中，我们将学习如何使用[Grails](http://www.grails.org/)创建一个简单的Web应用程序。

Grails(更准确地说是它的最新主要版本)是一个建立在Spring Boot项目之上的框架，使用Apache Groovy语言来开发Web应用程序。

**它受到Ruby的Rails框架的启发，并围绕约定优于配置的理念构建，从而可以减少样板代码**。

## 2. 设置

首先，让我们前往官方页面准备环境。在本教程撰写时，最新版本是3.3.3。

简而言之，有两种安装Grails的方法：通过SDKMAN或下载发行版并将二进制文件添加到PATH环境变量中。

我们不会逐步介绍设置过程，因为[Grails Docs](http://docs.grails.org/latest/guide/single.html#downloadingAndInstalling)中已有详细记录。

## 3. Grails应用程序的剖析

在本节中，我们将更好地了解Grails应用程序结构。正如我们前面提到的，Grails更倾向于约定而不是配置，因此文件的位置定义了它们的用途。让我们看看grails-app目录中有什么：

- **assets**：存储静态资产文件(如样式、JavaScript文件或图像)的地方

- **conf**：包含项目配置文件：

  - application.yml包含标准Web应用程序设置，例如数据源、MIME类型以及其他Grails或Spring相关设置
  - resources.groovy包含Spring Bean定义
  - logback.groovy包含日志配置

- **controllers**：负责处理请求并生成响应或将其委托给视图，按照惯例，当文件名以\*Controller结尾时，框架会为控制器类中定义的每个操作创建一个默认URL映射

- **domain**：包含Grails应用程序的业务模型，这里的每个类都将由GORM映射到数据库表

- **i18n**：用于国际化支持

- **init**：应用程序的入口点

- **services**：应用程序的业务逻辑将在此处存在，按照惯例，Grails将为每个服务创建一个Spring单例Bean

- **taglib**：自定义标签库的位置

- **views**：包含视图和模板

## 4. 简单的Web应用程序

在本章中，我们将创建一个简单的Web应用程序来管理学生；让我们首先调用CLI命令来创建应用程序骨架：

```shell
grails create-app
```

当项目的基本结构生成后，让我们继续实现实际的Web应用程序组件。

### 4.1 域层

因为我们正在实现一个用于处理学生的Web应用程序，所以我们首先要生成一个名为Student的域类：

```shell
grails create-domain-class cn.tuyucheng.taketoday.grails.Student
```

最后，让我们添加firstName和lastName属性：

```groovy
class Student {
    String firstName
    String lastName
}
```

Grails应用其约定并将为位于grails-app/domain目录中的所有类设置对象关系映射。

此外，由于[GormEntity](https://grails.github.io/legacy-gorm-doc/6.0.x/hibernate/api/org/grails/datastore/gorm/GormEntity.html)特性，**所有域类都可以访问所有CRUD操作**，我们将在下一节中使用这些操作来实现Service。

### 4.2 服务层

我们的应用程序将处理以下用例：

- 查看学生列表
- 创建新学生
- 删除现有学生

让我们实现这些用例，我们将首先生成一个Service类：

```shell
grails create-service cn.tuyucheng.taketoday.grails.Student
```

让我们转到grails-app/services目录，在适当的包中找到我们新创建的Service并添加所有必要的方法：

```groovy
@Transactional
class StudentService {

    def get(id){
        Student.get(id)
    }

    def list() {
        Student.list()
    }

    def save(student){
        student.save()
    }

    def delete(id){
        Student.get(id).delete()
    }
}
```

**请注意，Service默认不支持事务**，我们可以通过在类中添加@Transactional注解来启用此功能。

### 4.3 控制器层

为了使业务逻辑可用于UI，让我们通过调用以下命令创建一个StudentController：

```shell
grails create-controller cn.tuyucheng.taketoday.grails.Student
```

默认情况下，**Grails通过名称注入Bean**，这意味着我们可以通过声明一个名为studentsService的实例变量轻松地将StudentService单例实例注入到我们的控制器中。

我们现在可以定义读取、创建和删除学生的操作。

```groovy
class StudentController {

    def studentService

    def index() {
        respond studentService.list()
    }

    def show(Long id) {
        respond studentService.get(id)
    }

    def create() {
        respond new Student(params)
    }

    def save(Student student) {
        studentService.save(student)
        redirect action:"index", method:"GET"
    }

    def delete(Long id) {
        studentService.delete(id)
        redirect action:"index", method:"GET"
    }
}
```

按照惯例，**此控制器的index()操作将映射到URI /student/index**，show()操作将映射到URI /student/show，依此类推。

### 4.4 视图层

设置完控制器操作后，我们现在可以继续创建UI视图。我们将创建3个Groovy服务器页面，用于列出、创建和删除学生。

按照惯例，Grails将根据控制器名称和操作呈现视图。例如，StudentController中的**index()操作将解析为/grails-app/views/student/index.gsp**。

让我们从实现视图/grails-app/views/student/index.gsp开始，该视图将显示学生列表。我们将使用标签<f:table/\>创建一个HTML表格，显示从控制器中的index()操作返回的所有学生。

按照惯例，当我们使用对象列表进行响应时，**Grails会在模型名称中添加“List”后缀**，以便我们可以使用变量studentList访问学生对象列表：

```xml
<!DOCTYPE html>
<html>
    <head>
        <meta name="layout" content="main" />
    </head>
    <body>
        <div class="nav" role="navigation">
            <ul>
                <li><g:link class="create" action="create">Create</g:link></li>
            </ul>
        </div>
        <div id="list-student" class="content scaffold-list" role="main">
            <f:table collection="${studentList}" 
                properties="['firstName', 'lastName']" />
        </div>
    </body>
</html>
```

现在我们进入视图/grails-app/views/student/create.gsp，该视图允许用户创建新的学生。我们将使用内置的<f:all/\>标签，该标签显示给定Bean的所有属性的表单：

```xml
<!DOCTYPE html>
<html>
    <head>
        <meta name="layout" content="main" />
    </head>
    <body>
        <div id="create-student" class="content scaffold-create" role="main">
            <g:form resource="${this.student}" method="POST">
                <fieldset class="form">
                    <f:all bean="student"/>
                </fieldset>
                <fieldset class="buttons">
                    <g:submitButton name="create" class="save" value="Create" />
                </fieldset>
            </g:form>
        </div>
    </body>
</html>
```

最后，让我们创建视图/grails-app/views/student/show.gsp来查看并最终删除学生。

除其他标签外，我们将利用<f:display/\>，它以Bean作为参数并显示其所有字段：

```xml
<!DOCTYPE html>
<html>
    <head>
        <meta name="layout" content="main" />
    </head>
    <body>
        <div class="nav" role="navigation">
            <ul>
                <li><g:link class="list" action="index">Students list</g:link></li>
            </ul>
        </div>
        <div id="show-student" class="content scaffold-show" role="main">
            <f:display bean="student" />
            <g:form resource="${this.student}" method="DELETE">
                <fieldset class="buttons">
                    <input class="delete" type="submit" value="delete" />
                </fieldset>
            </g:form>
        </div>
    </body>
</html>
```

### 4.5 单元测试

Grails主要利用[Spock](http://spockframework.org/)进行测试，如果你不熟悉Spock，我们强烈建议你先阅读[本教程](https://www.baeldung.com/groovy-spock)。

让我们开始对StudentController的index()操作进行单元测试。

我们将Mock StudentService中的list()方法并测试index()是否返回预期的模型：

```groovy
void "Test the index action returns the correct model"() {
    given:
    controller.studentService = Mock(StudentService) {
        list() >> [new Student(firstName: 'John',lastName: 'Doe')]
    }
 
    when:"The index action is executed"
    controller.index()

    then:"The model is correct"
    model.studentList.size() == 1
    model.studentList[0].firstName == 'John'
    model.studentList[0].lastName == 'Doe'
}
```

现在，让我们测试delete()操作；我们将验证delete()是否从StudentService调用，并验证重定向到index页面：

```groovy
void "Test the delete action with an instance"() {
    given:
    controller.studentService = Mock(StudentService) {
      1 * delete(2)
    }

    when:"The domain instance is passed to the delete action"
    request.contentType = FORM_CONTENT_TYPE
    request.method = 'DELETE'
    controller.delete(2)

    then:"The user is redirected to index"
    response.redirectedUrl == '/student/index'
}
```

### 4.6 集成测试

接下来，我们来看看如何创建服务层的集成测试，我们主要测试与grails-app/conf/application.yml中配置的数据库的集成。

**默认情况下，Grails使用内存H2数据库来实现此目的**。

首先，让我们开始定义一个用于创建数据来填充数据库的辅助方法：

```groovy
private Long setupData() {
    new Student(firstName: 'John',lastName: 'Doe')
        .save(flush: true, failOnError: true)
    new Student(firstName: 'Max',lastName: 'Foo')
        .save(flush: true, failOnError: true)
    Student student = new Student(firstName: 'Alex',lastName: 'Bar')
        .save(flush: true, failOnError: true)
    student.id
}
```

得益于我们集成测试类上的@Rollback注解，**每种方法将在单独的事务中运行，并将在测试结束时回滚**。

看看我们如何实现list()方法的集成测试：

```groovy
void "test list"() {
    setupData()

    when:
    List<Student> studentList = studentService.list()

    then:
    studentList.size() == 3
    studentList[0].lastName == 'Doe'
    studentList[1].lastName == 'Foo'
    studentList[2].lastName == 'Bar'
}
```

另外，让我们测试一下delete()方法并验证学生总数是否减少了1：

```groovy
void "test delete"() {
    Long id = setupData()

    expect:
    studentService.list().size() == 3

    when:
    studentService.delete(id)
    sessionFactory.currentSession.flush()

    then:
    studentService.list().size() == 2
}
```

## 5. 运行和部署

通过Grails CLI调用单个命令即可运行和部署应用程序。

运行应用程序使用：

```shell
grails run-app
```

默认情况下，Grails将在端口8080上设置Tomcat。

让我们导航到http://localhost:8080/student/index来查看我们的Web应用程序是什么样子的：

![](/assets/images/2025/webmodules/grailsmvcapplication01.png)

如果要将应用程序部署到Servlet容器，请使用：

```shell
grails war
```

创建一个可随时部署的WAR工件。

## 6. 总结

在本文中，我们重点介绍了如何使用约定优于配置的理念创建Grails Web应用程序，我们还了解了如何使用Spock框架执行单元测试和集成测试。
