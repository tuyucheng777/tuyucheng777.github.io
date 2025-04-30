---
layout: post
title:  使用Gradle构建Java应用程序
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

**本教程提供了如何使用Gradle构建基于Java的项目的实用指南**。

我们将讲解手动创建项目结构、执行初始配置以及添加Java插件和JUnit依赖的步骤。然后，我们将构建并运行应用程序。

最后，在最后一节中，我们将举例说明如何使用Gradle Build Init插件来实现这一点，一些基本的介绍也可以在[Gradle简介](https://www.baeldung.com/gradle)一文中找到。

## 2. Java项目结构

**在我们手动创建Java项目并准备构建之前，我们需要[安装Gradle](https://docs.gradle.org/current/userguide/installation.html)**。

让我们开始使用PowerShell控制台创建一个名为gradle-employee-app的项目文件夹：

```powershell
> mkdir gradle-employee-app
```

之后，让我们导航到项目文件夹并创建子文件夹：

```powershell
> mkdir src/main/java/employee
```

结果输出如下：

```text
Directory: D:\gradle-employee-app\src\main\java

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        4/10/2020  12:14 PM                employee
```

在上面的项目结构中，让我们创建两个类。一个是简单的Employee类，包含姓名、电子邮件地址和出生年份等数据：

```java
public class Employee {
    String name;
    String emailAddress;
    int yearOfBirth;
}
```

第二个是打印Employee数据的主应用程序类：

```java
public class EmployeeApp {

    public static void main(String[] args){
        Employee employee = new Employee();

        employee.name = "John";
        employee.emailAddress = "john@tuyucheng.com";
        employee.yearOfBirth = 1978;

        System.out.println("Name: " + employee.name);
        System.out.println("Email Address: " + employee.emailAddress);
        System.out.println("Year Of Birth:" + employee.yearOfBirth);
    }
}
```

## 3. 构建Java项目

**接下来，为了构建我们的Java项目，我们在项目根文件夹中创建一个build.gradle配置文件**。

PowerShell命令行中的内容如下：

```powershell
Echo > build.gradle
```

我们跳过与输入参数相关的下一步：

```powershell
cmdlet Write-Output at command pipeline position 1
Supply values for the following parameters:
InputObject[0]:
```

为了构建成功，我们需要添加[Application插件](https://docs.gradle.org/current/userguide/application_plugin.html)：

```groovy
plugins {
    id 'application'
}
```

然后，我们应用Application插件并**添加主类的完全限定名称**：

```groovy
apply plugin: 'application'
mainClassName = 'employee.EmployeeApp'
```

每个项目都由任务组成，任务代表构建过程中执行的一项工作，例如编译源代码。

例如，我们可以向配置文件添加一个任务，打印有关已完成的项目配置的消息：

```groovy
println 'This is executed during configuration phase'
task configured {
    println 'The project is configured'
}
```

**通常，gradle build是主要任务，也是最常用的任务**。此任务会编译、测试并将代码组装成JAR文件，构建操作通过键入以下内容开始：

```powershell
> gradle build
```

执行上面命令输出：

```powershell
> Configure project :
This is executed during configuration phase
The project is configured
BUILD SUCCESSFUL in 1s
2 actionable tasks: 2 up-to-date
```

**要查看构建结果，我们先来看一下包含子文件夹的构建文件夹**：classes、distributions、libs和reports，输入Tree/ F可以查看构建文件夹的结构：

```powershell
├───build
│   ├───classes
│   │   └───java
│   │       ├───main
│   │       │   └───employee
│   │       │           Employee.class
│   │       │           EmployeeApp.class
│   │       │
│   │       └───test
│   │           └───employee
│   │                   EmployeeAppTest.class
│   │
│   ├───distributions
│   │       gradle-employee-app.tar
│   │       gradle-employee-app.zip
       
│   ├───libs
│   │       gradle-employee-app.jar
│   │
│   ├───reports
│   │   └───tests
│   │       └───test
│   │           │   index.html
│   │           │
│   │           ├───classes
│   │           │       employee.EmployeeAppTest.html
```

如你所见，classes子文件夹包含我们之前创建的两个已编译的.class文件，distributions子文件夹包含应用程序jar包的存档版本，libs保存着我们应用程序的jar文件。

通常，报告中会包含运行JUnit测试时生成的文件。 

现在一切准备就绪，可以通过输入gradle run来运行Java项目，退出时执行应用程序的结果如下：

```powershell
> Configure project :
This is executed during configuration phase
The project is configured

> Task :run
Name: John
Email Address: john@tuyucheng.com
Year Of Birth:1978

BUILD SUCCESSFUL in 1s
2 actionable tasks: 1 executed, 1 up-to-date
```

### 3.1 使用Gradle包装器构建

**Gradle Wrapper是一个调用Gradle声明版本的脚本**。

首先，我们在build.gradle文件中定义一个包装任务：

```groovy
task wrapper(type: Wrapper){
    gradleVersion = '5.3.1'
}
```

让我们使用Power Shell中的gradle包装器运行此任务：

```powershell
> Configure project :
This is executed during configuration phase
The project is configured

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

项目文件夹下会创建几个文件，包括/gradle/wrapper位置下的文件：

```powershell
│   gradlew
│   gradlew.bat
│   
├───gradle
│   └───wrapper
│           gradle-wrapper.jar
│           gradle-wrapper.properties
```

- gradlew：用于在Linux上创建Gradle任务的shell脚本
- gradlew.bat：Windows用户创建Gradle任务的.bat脚本
- gradle-wrapper.jar：我们的应用程序的包装可执行jar
- gradle-wrapper.properties：用于配置包装器的属性文件

## 4. 添加Java依赖并运行简单测试

首先，在我们的配置文件中，我们需要设置一个远程仓库，用于下载依赖包。通常，这些仓库是mavenCentral()或jcenter()，我们选择后者：

```groovy
repositories {
    jcenter()
}
```

创建仓库后，我们可以指定要下载的依赖。在本例中，我们添加了Apache Commons和JUnit库。**要实现此功能，请在依赖配置中添加testImplementation和testRuntime部分**。

它建立在一个额外的test块上：

```groovy
dependencies {
    compile group: 'org.apache.commons', name: 'commons-lang3', version: '3.12.0'
    testImplementation('junit:junit:4.13')
    testRuntime('junit:junit:4.13')
}

test {
    useJUnit()
}
```

完成后，让我们在一个简单的测试中尝试一下JUnit的工作，导航到src文件夹并为测试创建子文件夹：

```powershell
src> mkdir test/java/employee
```

在最后一个子文件夹中，让我们创建EmployeeAppTest.java：

```java
public class EmployeeAppTest {

    @Test
    public void testData() {
        Employee testEmp = this.getEmployeeTest();
        assertEquals(testEmp.name, "John");
        assertEquals(testEmp.emailAddress, "john@tuyucheng.com");
        assertEquals(testEmp.yearOfBirth, 1978);
    }

    private Employee getEmployeeTest() {
        Employee employee = new Employee();
        employee.name = "John";
        employee.emailAddress = "john@tuyucheng.com";
        employee.yearOfBirth = 1978;

        return employee;
    }
}
```

与之前类似，让我们从命令行运行gradle clean测试，测试应该可以顺利通过。

## 5. 使用Gradle初始化Java项目

在本节中，我们将讲解迄今为止创建和构建Java应用程序的步骤。不同之处在于，这次我们借助Gradle Build Init插件进行操作。

创建一个新的项目文件夹，并将其命名为gradle-java-example。然后，切换到该空项目文件夹并运行初始化脚本：

```powershell
> gradle init
```

Gradle会询问我们几个问题，并提供创建项目的选项，第一个问题是我们要生成什么类型的项目：

```powershell
Select type of project to generate:
  1: basic
  2: cpp-application
  3: cpp-library
  4: groovy-application
  5: groovy-library
  6: java-application
  7: java-library
  8: kotlin-application
  9: kotlin-library
  10: scala-library
Select build script DSL:
  1: groovy
  2: kotlin
Enter selection [1..10] 6
```

**选择项目类型的选项6，然后选择构建脚本的第一个选项(groovy)**。

接下来，会出现一系列问题：

```powershell
Select test framework:
  1: junit
  2: testng
  3: spock
Enter selection (default: junit) [1..3] 1

Project name (default: gradle-java-example):
Source package (default: gradle.java.example): employee

BUILD SUCCESSFUL in 57m 45s
2 actionable tasks: 2 executed
```

这里，我们选择第一个选项junit作为测试框架，选择项目的默认名称，并输入“employee”作为源包的名称。

要查看/src项目文件夹中的完整目录结构，请在PowerShell中输入Tree/F：

```powershell
├───main
│   ├───java
│   │   └───employee
│   │           App.java
│   │
│   └───resources
└───test
    ├───java
    │   └───employee
    │           AppTest.java
    │
    └───resources
```

最后，如果我们使用gradle run构建项目，我们会在退出时得到“Hello World”：

```powershell
> Task :run
Hello world.

BUILD SUCCESSFUL in 1s
2 actionable tasks: 1 executed, 1 up-to-date
```

## 6. 总结

在本文中，我们介绍了两种使用Gradle创建和构建Java应用程序的方法。事实上，我们手动操作了部分步骤，从命令行开始编译和构建应用程序需要花费一些时间。在这种情况下，如果应用程序使用了多个库，我们应该注意导入一些必需的包和类。

另一方面，Gradle初始化脚本具有生成项目轻量级骨架的功能，以及一些与Gradle相关的配置文件。