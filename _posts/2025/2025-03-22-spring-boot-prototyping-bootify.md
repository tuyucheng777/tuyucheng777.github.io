---
layout: post
title:  使用Bootify快速进行Spring Boot原型设计
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 简介

在当今快节奏的开发环境中，加快开发流程对于高效交付项目至关重要。生成样板代码和配置文件可以大大简化此流程。

[Bootify](https://bootify.io/)为此提供了强大的[Spring Boot](https://www.baeldung.com/spring-boot-start)原型解决方案，**通过自动创建标准组件和配置，Bootify使我们能够绕过重复且耗时的设置任务。这使开发人员能够将精力投入到应用程序更具创新性和独特性的方面，例如完善业务逻辑和添加自定义功能**。

在本教程中，我们将探索Bootify项目的基础知识，全面概述其核心特性和功能。我们还将通过一个实际示例来展示如何在实际场景中有效利用Bootify。

## 2. 什么是Bootify？

Bootify是一款专为Spring Boot框架设计的用户友好型应用程序生成器，它通过自动创建构建Spring Boot应用程序所需的样板代码和配置文件来简化和加速开发过程。

**借助Bootify，开发人员可以通过直观的基于Web的界面轻松设置实体、关系和各种应用程序组件**。此工具简化了初始设置，并确保了代码结构的一致性和遵循最佳实践。

通过生成基础代码和配置，Bootify允许开发人员专注于制定独特的业务逻辑和自定义功能，使其成为快速原型设计和全面应用程序开发的宝贵资产。

## 3. 创建项目

使用Bootify创建项目非常简单、高效，并且可以帮助我们快速启动并运行Spring Boot应用程序。

为了简化，我们将构建一个Spring Boot CRUD应用程序，使用[H2](https://www.baeldung.com/spring-boot-h2-database)作为数据库，且没有前端。

### 3.1 开始新项目

首先，我们进入[Bootify.io](https://bootify.io/)并点击“打Open Project开项目”链接，我们将看到启动新项目或修改现有项目的选项。我们选择“Start new project”选项来创建一个新项目以开始设置过程：

![](/assets/images/2025/springboot/springbootprototypingbootify01.png)

现在可以通过其唯一的URL访问我们的新项目。

### 3.2 配置项目设置

现在，我们配置项目的基本细节，例如项目名称、包结构和任何其他常规设置。Bootify提供了一个用户友好的界面来指定这些参数，确保我们的项目按照我们的偏好设置。

我们选择[Maven](https://www.baeldung.com/maven)作为构建工具，Java和[Lombok](https://www.baeldung.com/intro-to-project-lombok)作为语言。此外，我们选择H2作为数据库，Bootify会为我们选择的数据库添加必要的依赖项。

![](/assets/images/2025/springboot/springbootprototypingbootify02.png)

在开发人员偏好设置中，我们还激活了用于记录REST API的[OpenAPI](https://www.baeldung.com/spring-rest-openapi-documentation)：

![](/assets/images/2025/springboot/springbootprototypingbootify03.png)

### 3.3 定义我们的领域模型

现在，我们可以在”Entities”选项卡中创建数据库模式。Bootify提供了一个图形界面来定义应用程序的实体及其关系，我们可以创建实体、指定其属性，并建立它们之间的关系，例如一对多或多对多关联。

![](/assets/images/2025/springboot/springbootprototypingbootify04.png)

我们将创建一个简单的数据库模式来管理Post和PostComment，让我们创建Post实体：

![](/assets/images/2025/springboot/springbootprototypingbootify05.png)

另外，我们创建PostComment实体：

![](/assets/images/2025/springboot/springbootprototypingbootify06.png)

对于两个实体中的每一个，我们激活”CRUD Options”。

现在，我们可以创建两个实体之间的关系。Post和PostComment之间存在1:N关系，因此我们在此处创建一对多关系：

![](/assets/images/2025/springboot/springbootprototypingbootify07.png)

下图显示了实体、实体的属性以及它们之间的关系：

![](/assets/images/2025/springboot/springbootprototypingbootify08.png)

### 3.4 定义我们的数据对象和控制器

接下来是“Data Objects”，我们可以在其中定义[DTO](https://www.baeldung.com/java-dto-pattern)和[枚举](https://www.baeldung.com/a-guide-to-java-enums)。Bootify会自动添加PostDTO和PostCommentDTO。

最后一部分是“Controllers”，Bootify自动添加PostResource和PostCommentResource，这正是我们所需要的：

![](/assets/images/2025/springboot/springbootprototypingbootify09.png)

### 3.5 生成代码

**一旦完成配置，Bootify将为我们生成相应的Spring Boot代码，这包括我们应用程序所需的必要实体类、Repository、Service、Controller和其他样板组件**。

我们可以使用”Explore”来查看所有生成的文件：

![](/assets/images/2025/springboot/springbootprototypingbootify10.png)

另外，我们可以将生成的项目下载为ZIP文件。

## 4. 生成代码概述

下载zip文件后，我们可以在我们最喜欢的IDE(例如[IntelliJ IDEA](https://www.baeldung.com/intellij-basics)或[Eclipse](https://www.baeldung.com/eclipse-debugging))中打开它以开始在本地处理它：

![](/assets/images/2025/springboot/springbootprototypingbootify11.png)

以下是生成的代码中包含的关键组件和文件的细目分类。

**domain组件包括实体类**。这些类使用JPA注解(例如@Entity、@Id和@GeneratedValue)进行标注，以将它们映射到相应的数据库表。每个实体类都包含表示已定义属性的字段以及Getter和Setter方法。

**repos组件代表Repository接口**。Bootify生成扩展JpaRepository的接口，后者为CRUD(创建、读取、更新、删除)操作提供内置方法。这些Repository接口支持数据库交互，无需对常见数据库操作进行自定义实现。

**service组件负责提供服务层**。生成的代码包括封装业务逻辑的服务类，这些类用[@Service](https://www.baeldung.com/spring-component-repository-service)标注，通常包含处理与实体相关的业务操作的方法。Service与Repository交互以执行数据访问操作并在需要时实现其他逻辑。

**rest组件包含REST控制器**。Bootify生成使用[@RestController](https://www.baeldung.com/spring-controller-vs-restcontroller)和[@RequestMapping](https://www.baeldung.com/spring-requestmapping)标注的REST控制器来管理HTTP请求。这些控制器将传入的请求映射到适当的服务方法并返回正确的响应，它们包括CRUD操作的端点，并使用@GetMapping、@PostMapping、@PutMapping和@DeleteMapping等注解来定义其操作。

**model组件包括DTO类**，Bootify生成这些类以方便客户端和服务器之间的数据传输。DTO用于构造API返回的数据或从客户端接收的数据，从而有效地将内部数据模型与外部API表示分离。

**config组件由SwaggerConfig和JacksonConfig等配置类组成**，这些类管理与API文档和对象映射相关的设置。

**最后，应用程序属性在application.properties或application.yml文件中定义，用于管理应用程序配置**。这些文件处理数据库连接详细信息、服务器端口配置和其他特定于环境的属性等设置。

## 5. 总结

在本文中，我们探索了Bootify应用程序生成器。有了Bootify处理常规设置，我们可以专注于构建和增强应用程序的核心功能，而不必花时间进行重复的设置任务。