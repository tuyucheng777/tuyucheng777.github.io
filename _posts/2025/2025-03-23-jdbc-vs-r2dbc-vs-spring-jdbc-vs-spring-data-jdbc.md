---
layout: post
title:  JDBC与R2DBC与Spring JDBC与Spring Data JDBC
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 简介

在Java应用程序中使用数据库时，我们有多种选择。JDBC、R2DBC、Spring JDBC和Spring Data JDBC是与数据库交互的最流行的框架，每个框架都提供独特的功能和优势，以高效处理数据库操作。

在本快速教程中，我们将深入研究数据库连接框架的世界，并探索每个框架如何发挥其独特的优势。从传统的JDBC到尖端的R2DBC以及介于两者之间的一切，我们将揭示它们的内部工作原理并并排比较它们的功能以选择合适的工具。

## 2. JDBC

[JDBC(Java数据库连接)](https://www.baeldung.com/java-jdbc)是Java中访问数据库的最古老和最广泛使用的标准，它提供了一组接口和类来执行SQL查询、检索结果和执行其他数据库操作。

它的优势在于能够高效处理简单和复杂的数据库操作。**此外，由于其在管理数据库连接和查询方面的广泛接受度、可靠性和多功能性，它仍然是一个首选框架**。

JDBC的局限性之一是它使用阻塞I/O模型，如果有大量并发请求，这可能会导致性能问题。

## 3. R2DBC

**与JDBC不同，[R2DBC(响应式关系数据库连接)](https://www.baeldung.com/r2dbc)使用响应式流和非阻塞I/O模型来处理数据库操作**，响应式和非阻塞I/O的这种组合使其非常适合并发系统。

R2DBC可用于RxJava和Reactor等现代响应式编程框架，它支持事务和非事务操作。

R2BC是一项较新的技术，因此，并非所有数据库都支持它。此外，可用的驱动程序可能因我们使用的数据库而异。此外，它的学习曲线也比较陡峭。

## 4. Spring JDBC

Spring JDBC是JDBC之上的轻量级抽象层，它通过提供更高级别的API和处理许多常见任务(例如连接管理和异常处理)来简化数据库访问。**此外，它通过提供参数化查询并将查询结果映射到Java对象来高效管理重复任务，从而减少样板代码**。

[Spring JDBC](https://www.baeldung.com/spring-jdbc-jdbctemplate)的一个显著优点是它与其他Spring组件和框架的无缝集成。

由于它依赖于JDBC的阻塞IO模型，因此可能会限制高并发系统中的可扩展性。此外，与其他框架(即Hibernate)相比，它在功能集方面有所欠缺。

## 5. Spring Data JDBC

Spring生态系统提供的另一个数据库访问工具是[Spring Data JDBC](https://www.baeldung.com/spring-data-jdbc-intro)，**与JDBC和R2DBC相比，它遵循Repository风格的方法进行数据库交互**。

对于重视域和代码生成简单性的应用程序来说，Spring Data JDBC是自动选择。它与域驱动设计配合良好，并支持使用注解和约定将域对象映射到数据库表。除了将Java对象映射到数据库表之外，它还为常见的CRUD操作提供了易于使用的Repository接口。

Spring Data JDBC是一个相对较新的框架，因此，它的成熟度不如其他框架。例如，它对复杂查询的支持不多，因此我们必须自己编写查询。此外，它不支持事务，这在某些情况下可能会成为问题。

## 6. 比较

最终，JDBC、R2DBC、Spring JDBC和Spring Data JDBC之间的最佳选择取决于具体要求。但是，应用程序的性质也可能影响决策。

下表可以帮助我们做出决定：

|  特征  |  JDBC  | R2DBC | Spring JDBC | Spring Data JDBC |
|:----:|:------:|:-----:|:-----------:|:----------------:|
| API  |   低级   |  低级   |     高级      |        高级        |
|  性能  |   好    |  出色   |      好      |        好         |
|  通信  |   同步   |  响应式  |     同步      |        异步        |
|  稳定  |   成熟   |  较新   |     成熟      |        较新        |
|  特征  |  功能较少  | 功能较少  |    更多功能     |       更多功能       |
| 易于使用 |   简单   |  缓和   |     简单      |        简单        |
|  支持  |   广泛   |  生长   |     广泛      |        生长        |

## 7. 总结

在本文中，我们研究了Java生态系统中的几种数据库方法。

对于传统、同步且得到广泛支持的方法，JDBC或Spring JDBC可能是正确的选择。同样，对于具有非阻塞数据库访问的响应式应用程序，R2DBC可能是一个不错的选择。最后，为了简单性和更高级别的抽象，Spring Data JDBC可能是理想的选择。

通过了解每个框架的优点和缺点，我们可以做出最适合应用程序需求的决定，并有助于构建健壮、可扩展和可维护的数据库访问代码。