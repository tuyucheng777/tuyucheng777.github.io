---
layout: post
title:  用于Spring Boot测试的嵌入式PostgreSQL
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

使用数据库编写集成测试提供了多种测试数据库选项。一种有效的选项是使用真实数据库，以确保我们的集成测试与生产行为紧密相关。

**在本教程中，我们将演示如何使用[嵌入式PostgreSQL](https://github.com/opentable/otj-pg-embedded)进行Spring Boot测试并回顾一些替代方案**。

## 2. 依赖和配置

我们首先添加[Spring Data JPA依赖](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)，因为我们将使用它来创建我们的Repository：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

要[为Spring Boot应用程序编写集成测试](https://www.baeldung.com/spring-boot-testing)，我们需要包含[Spring Test依赖](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

最后，我们需要包含[嵌入式Postgres依赖](https://mvnrepository.com/artifact/com.opentable.components/otj-pg-embedded)：

```xml
<dependency>
    <groupId>com.opentable.components</groupId>
    <artifactId>otj-pg-embedded</artifactId>
    <version>1.0.3</version>
    <scope>test</scope>
</dependency>
```

另外，让我们为集成测试设置基本配置：

```properties
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=create-drop
```

我们在执行测试之前指定了PostgreSQLDialect并启用了模式重新创建。

## 3. 使用方法

首先，让我们创建在测试中使用的Person实体：

```java
@Entity
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column
    private String name;

    // getters and setters
}
```

现在，让我们为实体创建一个Spring Data Repository：

```java
public interface PersonRepository extends JpaRepository<Person, Long> {
}
```

之后我们来创建一个测试配置类：

```java
@Configuration
@EnableJpaRepositories(basePackageClasses = PersonRepository.class)
@EntityScan(basePackageClasses = Person.class)
public class EmbeddedPostgresConfiguration {
    private static EmbeddedPostgres embeddedPostgres;

    @Bean
    public DataSource dataSource() throws IOException {
        embeddedPostgres = EmbeddedPostgres.builder()
                .setImage(DockerImageName.parse("postgres:14.1"))
                .start();

        return embeddedPostgres.getPostgresDatabase();
    }

    public static class EmbeddedPostgresExtension implements AfterAllCallback {
        @Override
        public void afterAll(ExtensionContext context) throws Exception {
            if (embeddedPostgres == null) {
                return;
            }
            embeddedPostgres.close();
        }
    }
}
```

在这里，我们指定了Repository和实体的路径。**我们使用EmbeddedPostgres构建器创建了数据源，并选择了测试期间要使用的Postgres数据库的版本**。此外，我们添加了EmbeddedPostgresExtension以确保在执行测试类后关闭嵌入式Postgres连接。最后，让我们创建测试类：

```java
@DataJpaTest
@ExtendWith(EmbeddedPostgresExtension.class)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@ContextConfiguration(classes = {EmbeddedPostgresConfiguration.class})
public class EmbeddedPostgresIntegrationTest {
    @Autowired
    private PersonRepository repository;

    @Test
    void givenEmbeddedPostgres_whenSavePerson_thenSavedEntityShouldBeReturnedWithExpectedFields(){
        Person person = new Person();
        person.setName("New user");

        Person savedPerson = repository.save(person);
        assertNotNull(savedPerson.getId());
        assertEquals(person.getName(), savedPerson.getName());
    }
}
```

我们使用[@DataJpaTest](https://www.baeldung.com/junit-datajpatest-repository)注解来设置基本的Spring测试上下文。**我们使用EmbeddedPostgresExtension扩展了测试类，并将EmbeddedPostgresConfiguration附加到测试上下文**。之后，我们成功创建了一个Person实体并将其保存在数据库中。

## 4. Flyway集成

[Flyway](https://www.baeldung.com/database-migrations-with-flyway)是一种流行的迁移工具，可帮助管理模式更改。当我们使用它时，将其包含在我们的集成测试中非常重要。在本节中，我们将了解如何使用嵌入式Postgres来完成此操作。让我们从[依赖](https://mvnrepository.com/artifact/org.flywaydb/flyway-core)开始：

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

之后，让我们在Flyway迁移脚本中指定数据库模式：

```sql
CREATE SEQUENCE IF NOT EXISTS person_seq INCREMENT 50;
;

CREATE TABLE IF NOT EXISTS person(
    id bigint NOT NULL,
    name character varying(255)
)
;
```

现在我们可以创建测试配置：

```java
@Configuration
@EnableJpaRepositories(basePackageClasses = PersonRepository.class)
@EntityScan(basePackageClasses = Person.class)
public class EmbeddedPostgresWithFlywayConfiguration {
    @Bean
    public DataSource dataSource() throws SQLException {
        return PreparedDbProvider
                .forPreparer(FlywayPreparer.forClasspathLocation("db/migrations"))
                .createDataSource();
    }
}
```

**我们指定了数据源Bean，并使用PreparedDbProvider和FlywayPreparer定义了迁移脚本的位置**。最后，这是我们的测试类：

```java
@DataJpaTest(properties = { "spring.jpa.hibernate.ddl-auto=none" })
@ExtendWith(EmbeddedPostgresExtension.class)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@ContextConfiguration(classes = {EmbeddedPostgresWithFlywayConfiguration.class})
public class EmbeddedPostgresWithFlywayIntegrationTest {
    @Autowired
    private PersonRepository repository;

    @Test
    void givenEmbeddedPostgres_whenSavePerson_thenSavedEntityShouldBeReturnedWithExpectedFields(){
        Person person = new Person();
        person.setName("New user");

        Person savedPerson = repository.save(person);
        assertNotNull(savedPerson.getId());
        assertEquals(person.getName(), savedPerson.getName());

        List<Person> allPersons = repository.findAll();
        Assertions.assertThat(allPersons).contains(person);
    }
}
```

**我们禁用了spring.jpa.hibernate.ddl-auto属性，以允许Flyway处理模式更改**。之后，我们将Person实体保存在数据库中并成功检索它。

## 5. 替代方案

### 5.1 Testcontainers

嵌入式Postgres项目的最新版本在后台使用[TestContainers](https://www.baeldung.com/docker-test-containers)。**因此，一种替代方法是直接使用TestContainers库**。让我们从添加必要的[依赖](https://mvnrepository.com/artifact/org.testcontainers/postgresql)开始：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.8</version>
    <scope>test</scope>
</dependency>
```

现在我们将创建初始化类，在其中为我们的测试配置PostgreSQLContainer：

```java
public class TestContainersInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext>, AfterAllCallback {

    private static final PostgreSQLContainer postgreSQLContainer = new PostgreSQLContainer(
            "postgres:14.1")
            .withDatabaseName("postgres")
            .withUsername("postgres")
            .withPassword("postgres");


    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        postgreSQLContainer.start();

        TestPropertyValues.of(
                "spring.datasource.url=" + postgreSQLContainer.getJdbcUrl(),
                "spring.datasource.username=" + postgreSQLContainer.getUsername(),
                "spring.datasource.password=" + postgreSQLContainer.getPassword()
        ).applyTo(applicationContext.getEnvironment());
    }

    @Override
    public void afterAll(ExtensionContext context) throws Exception {
        if (postgreSQLContainer == null) {
            return;
        }
        postgreSQLContainer.close();
    }
}
```

我们创建了PostgreSQLContainer实例并实现了ApplicationContextInitializer接口来设置测试上下文的配置属性。此外，我们还实现了AfterAllCallback以在测试后关闭Postgres容器连接。现在，让我们创建测试类：

```java
@DataJpaTest
@ExtendWith(TestContainersInitializer.class)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@ContextConfiguration(initializers = TestContainersInitializer.class)
public class TestContainersPostgresIntegrationTest {
    @Autowired
    private PersonRepository repository;

    @Test
    void givenTestcontainersPostgres_whenSavePerson_thenSavedEntityShouldBeReturnedWithExpectedFields() {
        Person person = new Person();
        person.setName("New user");

        Person savedPerson = repository.save(person);
        assertNotNull(savedPerson.getId());
        assertEquals(person.getName(), savedPerson.getName());
    }
}
```

**在这里，我们使用TestContainersInitializer扩展了测试，并使用@ContextConfiguration注解指定了测试配置的初始化程序**。我们创建了与上一节相同的测试用例，并成功地将Person实体保存在测试容器中运行的Postgres数据库中。

### 5.2 Zonky嵌入式数据库

**[Zonky嵌入式数据库](https://github.com/zonkyio/embedded-database-spring-test)是作为嵌入式Postgres的一个分支创建的，并继续支持不使用Docker的测试数据库选项**。让我们添加使用此库所需的[依赖](https://mvnrepository.com/artifact/io.zonky.test)：

```xml
<dependency>
    <groupId>io.zonky.test</groupId>
    <artifactId>embedded-postgres</artifactId>
    <version>2.0.7</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.zonky.test</groupId>
    <artifactId>embedded-database-spring-test</artifactId>
    <version>2.5.1</version>
    <scope>test</scope>
</dependency>
```

之后我们就可以编写测试类了：

```java
@DataJpaTest
@AutoConfigureEmbeddedDatabase(provider = ZONKY)
public class ZonkyEmbeddedPostgresIntegrationTest {
    @Autowired
    private PersonRepository repository;

    @Test
    void givenZonkyEmbeddedPostgres_whenSavePerson_thenSavedEntityShouldBeReturnedWithExpectedFields(){
        Person person = new Person();
        person.setName("New user");

        Person savedPerson = repository.save(person);
        assertNotNull(savedPerson.getId());
        assertEquals(person.getName(), savedPerson.getName());
    }
}
```

**在这里，我们使用ZONKY提供程序指定了@AutoConfigureEmbeddedDatabase注解，使我们能够在没有Docker的情况下使用嵌入式Postgres数据库**。此库还支持其他提供程序，例如Embedded和Docker。最后，我们已成功将Person实体保存在数据库中。

## 6. 总结

在本文中，我们探讨了如何使用嵌入式Postgres数据库进行测试，并回顾了一些替代方案。有多种方法可以将Postgres数据库纳入测试中，无论是否使用Docker容器。最佳选择取决于你的具体用例。