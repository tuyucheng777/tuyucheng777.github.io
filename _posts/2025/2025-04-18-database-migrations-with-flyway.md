---
layout: post
title:  使用Flyway进行数据库迁移
category: persistence
copyright: persistence
excerpt: Flyway
---

## 1. 简介

在本教程中，我们将探讨[Flyway](https://flywaydb.org/)的关键概念，以及如何使用此框架可靠、轻松地持续重构应用程序的数据库模式。此外，我们还将提供一个使用Maven Flyway插件管理内存H2数据库的示例。

**Flyway使用迁移将数据库从一个版本更新到下一个版本**，我们可以使用特定于数据库的语法，用SQL编写迁移，也可以使用Java编写迁移以实现高级数据库转换。

迁移可以是版本化的，也可以是可重复的。前者具有唯一版本，并且只应用一次。后者没有版本，而是每次校验和发生变化时都会(重新)应用。

在单次迁移运行中，可重复迁移始终在待执行的版本化迁移执行完毕后最后应用。可重复迁移按其描述的顺序应用，对于单次迁移，所有语句都在单个数据库事务中运行。

在本教程中，我们将主要关注如何使用Maven插件执行数据库迁移。

## 2. Flyway Maven插件

要安装Flyway Maven插件，让我们将以下插件定义添加到pom.xml中：

```xml
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>10.17.0</version> 
</plugin>
```

该插件的最新版本可在[Maven Central](https://mvnrepository.com/artifact/org.flywaydb/flyway-maven-plugin)上找到。

我们可以通过四种不同的方式配置此Maven插件，在以下部分中，我们将逐一介绍这些选项。

请参阅[文档](https://flywaydb.org/documentation/usage/maven/migrate)以获取所有可配置属性的列表。

### 2.1 插件配置

我们可以**通过pom.xml的插件定义中的<configuration\>标签直接配置插件**：

```xml
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>10.17.0</version>
    <configuration>
        <user>databaseUser</user>
        <password>databasePassword</password>
        <schemas>
            <schema>schemaName</schema>
        </schemas>
        ...
    </configuration>
</plugin>
```

### 2.2 Maven属性

我们还可以**通过在pom中将可配置属性指定为Maven属性来配置插件**：

```xml
<project>
    ...
    <properties>
        <flyway.user>databaseUser</flyway.user>
        <flyway.password>databasePassword</flyway.password>
        <flyway.schemas>schemaName</flyway.schemas>
        ...
    </properties>
    ...
</project>
```

### 2.3 外部配置文件

另一种选择**是在单独的.conf文件中提供插件配置**：

```properties
flyway.user=databaseUser
flyway.password=databasePassword
flyway.schemas=schemaName
...
```

**默认配置文件名是flyway.conf，默认从以下位置加载配置文件**：

- installDir/conf/flyway.conf
- userhome/flyway.conf
- workingDir/flyway.conf

编码由flyway.encoding属性指定(默认为UTF-8)。

如果我们使用任何其他名称(例如customConfig.conf)作为配置文件，那么我们必须在调用Maven命令时明确指定它：

```shell
$ mvn -Dflyway.configFiles=customConfig.conf
```

### 2.4 系统属性

最后，在命令行上调用Maven时，**所有配置属性也可以指定为系统属性**：

```shell
$ mvn -Dflyway.user=databaseUser -Dflyway.password=databasePassword 
  -Dflyway.schemas=schemaName
```

当以多种方式指定配置时，优先顺序如下：

1. 系统属性
2. 外部配置文件
3. Maven属性
4. 插件配置

## 3. 迁移示例

在本节中，**我们将逐步介绍使用Maven插件将数据库模式迁移到内存H2数据库所需的步骤**，我们使用外部文件来配置Flyway。

### 3.1 更新POM

首先，让我们添加H2作为依赖：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

同样，我们可以在[Maven Central](https://mvnrepository.com/artifact/com.h2database/h2)上检查驱动程序的最新版本。我们还添加了Flyway插件，如前所述。

### 3.2 使用外部文件配置Flyway

接下来，我们在$PROJECT_ROOT中创建myFlywayConfig.conf，内容如下：

```properties
flyway.user=databaseUser
flyway.password=databasePassword
flyway.schemas=app-db
flyway.url=jdbc:h2:mem:DATABASE
flyway.locations=filesystem:db/migration
```

上述配置指定我们的迁移脚本位于db/migration目录中，它使用databaseUser和databasePassword连接到内存中的H2实例。

应用程序数据库模式是app-db。

当然，我们把flyway.user、flyway.password、flyway.url分别替换成我们自己的数据库用户名、数据库密码、数据库URL。

### 3.3 定义第一次迁移

**Flyway遵循以下迁移脚本的命名约定**：

<Prefix\><Version\>__<Description\>.sql

其中：

- <Prefix\>：默认前缀是V，我们可以使用flyway.sqlMigrationPrefix属性在上面的配置文件中更改它。
- <Version\>：迁移版本号，主版本号和次版本号可以用下划线分隔，迁移版本号应始终以1开头。
- <Description\>：迁移的文本描述，描述与版本号之间用双下划线分隔。

示例：V1_1_0__my_first_migration.sql

因此，让我们在$PROJECT_ROOT中创建一个目录db/migration，其中有一个名为V1_0__create_employee_schema.sql的迁移脚本，其中包含用于创建employee表的SQL指令：

```sql
CREATE TABLE IF NOT EXISTS `employee` (

    `id` int NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `name` varchar(20),
    `email` varchar(50),
    `date_of_birth` timestamp

)ENGINE=InnoDB DEFAULT CHARSET=UTF8;
```

### 3.4 执行迁移

接下来，我们从$PROJECT_ROOT调用以下Maven命令来执行数据库迁移：

```shell
$ mvn clean flyway:migrate -Dflyway.configFiles=myFlywayConfig.conf
```

这应该会导致我们的第一次迁移成功。

数据库模式现在应如下所示：

```text
employee:
+----+------+-------+---------------+
| id | name | email | date_of_birth |
+----+------+-------+---------------+
```

我们可以重复定义和执行步骤来进行更多的迁移。

### 3.5 定义并执行第二次迁移

让我们通过创建第二个迁移文件(名称为V2_0_create_department_schema.sql)来查看第二次迁移是什么样子，其中包含以下两个查询：

```sql
CREATE TABLE IF NOT EXISTS `department` (

`id` int NOT NULL AUTO_INCREMENT PRIMARY KEY,
`name` varchar(20)

)ENGINE=InnoDB DEFAULT CHARSET=UTF8; 

ALTER TABLE `employee` ADD `dept_id` int AFTER `email`;
```

然后我们将执行与第一次类似的迁移。

现在我们的数据库模式已经改变，为employee添加了一个新列并添加了一个新表：

```text
employee:
+----+------+-------+---------+---------------+
| id | name | email | dept_id | date_of_birth |
+----+------+-------+---------+---------------+
```

```text
department:
+----+------+
| id | name |
+----+------+
```

最后，我们可以通过调用以下Maven命令来验证两次迁移确实成功：

```shell
$ mvn flyway:info -Dflyway.configFiles=myFlywayConfig.conf
```

## 4. 在IntelliJ IDEA中生成版本化迁移

手动编写迁移非常耗时；我们可以基于JPA实体生成迁移，**我们可以通过使用IntelliJ IDEA的[JPA Buddy](https://www.baeldung.com/jpa-buddy-post)插件来实现这一点**。

要生成差异版本迁移，只需安装插件并从JPA Structure面板调用操作。

我们只需选择要比较的源(数据库或JPA实体)与目标(数据库或数据模型快照)，然后，JPA Buddy将生成如动画所示的迁移：

[![flyway迁移生成jpa](https://www.baeldung.com/wp-content/uploads/2016/09/flyway-migration-generation-jpa.gif)](https://www.baeldung.com/wp-content/uploads/2016/09/flyway-migration-generation-jpa.gif)

JPA Buddy的另一个优势是能够定义Java和数据库类型之间的映射，此外，它还能与Hibernate自定义类型和JPA转换器完美兼容。

## 5. 在Spring Boot中禁用Flyway

**有时我们可能需要在某些情况下禁用Flyway迁移**。

例如，在测试过程中，根据实体生成数据库模式是一种常见的做法。在这种情况下，我们可以在test Profile下禁用Flyway。

### 5.1 Spring Boot 1.x

我们需要做的就是**在application-test.properties文件中设置flyway.enabled属性**：

```properties
flyway.enabled=false
```

### 5.2 Spring Boot 2.x、3.x

**在Spring Boot的较新版本中，此属性已更改为spring.flyway.enabled**：

```properties
spring.flyway.enabled=false
```

### 5.3 FlywayMigrationStrategy

**如果我们只想在启动时禁用自动Flyway迁移，但仍然能够手动触发迁移，那么使用上面描述的属性并不是一个好的选择**。

这是因为在这种情况下，Spring Boot将不再自动配置Flyway Bean。因此，我们必须自己提供它，这不太方便。

因此，如果这是我们的用例，我们可以保持Flyway启用并实现一个空的[FlywayMigrationStrategy](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/flyway/FlywayMigrationStrategy.html)：

```java
@Configuration
public class EmptyMigrationStrategyConfig {

    @Bean
    public FlywayMigrationStrategy flywayMigrationStrategy() {
        return flyway -> {
            // do nothing  
        };
    }
}
```

**这将有效地禁用应用程序启动时的Flyway迁移**。

但是，我们仍然可以手动触发迁移：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ManualFlywayMigrationIntegrationTest {

    @Autowired
    private Flyway flyway;

    @Test
    public void skipAutomaticAndTriggerManualFlywayMigration() {
        flyway.migrate();
    }
}
```

## 6. Flyway的工作原理

为了追踪我们已经应用了哪些迁移以及何时应用，它会在模式中添加一个特殊的元数据表，此元数据表还会追踪迁移校验和，以及迁移是否成功。

该框架执行以下步骤来适应不断发展的数据库模式：

1. 它检查数据库模式以定位其元数据表(默认情况下为SCHEMA_VERSION)，如果元数据表不存在，它将创建一个。
2. 它扫描应用程序类路径以查找可用的迁移。
3. 它会将迁移与元数据表进行比较，如果版本号低于或等于标记为当前的版本，则会被忽略。
4. 它将任何剩余的迁移标记为待处理迁移，这些迁移将根据版本号排序并按顺序执行。
5. 随着每次迁移的应用，元数据表也会相应更新。

## 7. 命令

Flyway支持以下基本命令来管理数据库迁移：

- **Info**：打印数据库模式的当前状态/版本，它打印哪些迁移处于待处理状态、哪些迁移已应用、已应用迁移的状态以及应用时间。
- **Migrate**：将数据库模式迁移到当前版本，它会扫描Classpath中可用的迁移，并应用待处理的迁移。
- **Baseline**：对现有数据库进行基线设置，排除所有迁移(包括baseVersion)。Baseline有助于在现有数据库中启动Flyway，之后，较新的迁移即可正常应用。
- **Validate**：根据可用的迁移验证当前数据库模式。
- **Repair**：修复元数据表。
- **Clean**：删除已配置模式中的所有对象，当然，我们永远不应该在任何生产数据库上使用clean。

## 8. 总结

在本文中，我们了解了Flyway的工作原理以及如何使用该框架可靠地重塑我们的应用程序数据库。