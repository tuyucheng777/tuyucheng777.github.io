---
layout: post
title:  在Gradle中使用多个仓库
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

在本教程中，我们将了解如何在Gradle项目中使用多个仓库。当我们需要使用Maven Central上没有的JAR文件时，这非常有用，我们还将学习如何使用GitHub发布Java包并在不同项目之间共享它们。

## 2. 在Gradle中使用多个仓库

在使用[Gradle](https://www.baeldung.com/gradle-building-a-java-app)作为构建工具时，我们经常在build.gradle的repositories部分遇到mavenCentral()。如果我们想要添加其他仓库，可以将它们添加到同一部分以指示库的来源：

```groovy
repositories {
    mavenLocal()
    mavenCentral()
}
```

这里，mavenLocal()用于在[Maven的本地缓存](https://www.baeldung.com/maven-local-repository)中查找所有依赖，任何在本地缓存中找不到的仓库都将从Maven Central下载。

## 3. 使用经过身份验证的仓库

我们还可以通过提供有效的身份验证来使用非公开的仓库，例如，**GitHub和其他一些平台提供了将我们的包发布到其仓库的功能，之后可以在其他项目中使用**。

### 3.1 将包发布到GitHub包注册表

我们将把下面的类发布到GitHub注册表，然后在另一个项目中使用它：

```java
public class User {
    private Integer id;
    private String name;
    private Date dob;
    
   // standard constructors, getters and setters
}
```

要发布代码，**我们需要一个来自GitHub的个人访问令牌**，我们可以按照[GitHub文档](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)中的说明创建一个，然后，我们使用用户名和此令牌向build.gradle文件添加一个发布任务：

```groovy
publishing {
    publications {
        register("jar", MavenPublication) {
            from(components["java"])
            pom {
                url.set("https://github.com/eugenp/tutorials.git")
            }
        }
    }
    repositories {
        maven {
            name = "GitHubPackages"
            url = "https://maven.pkg.github.com/eugenp/tutorials"
            credentials {
                username = project.USERNAME
                password = project.GITHUB_TOKEN
            }
        }
    }
}
```

在上面的代码片段中，**username和password是执行Gradle发布任务时提供的项目级变量**。

### 3.2 使用已发布的包作为库

成功发布我们的软件包后，**我们可以从经过身份验证的仓库中将其作为库安装**。让我们在build.gradle中添加以下代码，以便在新项目中使用已发布的软件包：

```groovy
repositories {
    // other repositories
    maven {
        name = "GitHubPackages"
        url = "https://maven.pkg.github.com/eugenp/tutorials"
        credentials {
            username = project.USERNAME
            password = project.GITHUB_TOKEN
        }
    }
}
dependencies {
    implementation('cn.tuyucheng.taketoday.gradle:publish-package:1.0.0-SNAPSHOT')
    testImplementation("org.junit.jupiter:junit-jupiter-engine:5.9.0")
    // other dependencies
}
```

这将从GitHub包注册表安装库并允许我们在项目中扩展该类：

```java
public class Student extends User {
    private String studentCode;
    private String lastInstitution;
    // standard constructors, getters and setters
}
```

让我们使用一个简单的测试方法来测试我们的代码：

```java
@Test
public void testPublishedPackage(){
    Student student = new Student();
    student.setId(1);
    student.setStudentCode("CD-875");
    student.setName("John Doe");
    student.setLastInstitution("Institute of Technology");

    assertEquals("John Doe", student.getName());
}
```

## 4. 总结

在本文中，我们了解了如何在Gradle项目中使用来自多个仓库的库，我们还学习了如何使用GitHub Package Registry来管理经过身份验证的仓库。