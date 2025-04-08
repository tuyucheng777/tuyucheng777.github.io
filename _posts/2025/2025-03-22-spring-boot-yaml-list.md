---
layout: post
title:  Spring Boot中的YAML到对象列表
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在这个简短的教程中，我们将仔细研究**如何将YAML列表映射到Spring Boot中的列表**。

我们将首先了解如何在YAML中定义列表的一些背景知识。

然后我们将深入研究如何将YAML列表绑定到对象列表。

## 2. 快速回顾YAML中的列表

简而言之，[YAML](https://yaml.org/spec/1.2/spec.html)是一种人类可读的数据序列化标准，它提供了一种简洁明了的方式来编写配置文件。**YAML的优点在于它支持多种数据类型，例如List、Map和标量类型**。

YAML列表中的元素使用“-”字符定义，并且它们都具有相同的缩进级别：

```yaml
yamlconfig:
    list:
        - item1
        - item2
        - item3
        - item4
```

作为比较，基于properties的等价物使用索引：

```properties
yamlconfig.list[0]=item1
yamlconfig.list[1]=item2
yamlconfig.list[2]=item3
yamlconfig.list[3]=item4
```

要了解更多示例，请随意查看我们的文章，了解如何[使用YAML和属性文件定义List和Map](https://www.baeldung.com/spring-yaml-vs-properties#lists-and-maps)。

事实上，与properties文件相比，**YAML的层次结构特性显著提高了可读性**。YAML的另一个有趣的特性是可以[为不同的Spring Profile定义不同的属性](https://www.baeldung.com/spring-yaml#spring-yaml-file)。从Boot 2.4.0版本开始，properties文件也可以这样做。

值得一提的是，Spring Boot提供了对YAML配置的开箱即用支持。根据设计，Spring Boot在启动时从application.yml加载配置属性，无需任何额外工作。

## 3. 将YAML列表绑定到简单对象列表

**Spring Boot提供了[@ConfigurationProperties](https://www.baeldung.com/configuration-properties-in-spring-boot)注解来简化将外部配置数据映射到对象模型的逻辑**。

在本节中，我们将使用@ConfigurationProperties将YAML列表绑定到List<Object\>中。

我们首先在application.yml中定义一个简单的列表：

```yaml
application:
    profiles:
        - dev
        - test
        - prod
        - 1
        - 2
```

然后我们将创建一个简单的ApplicationProps [POJO](https://www.baeldung.com/java-pojo-class)来保存将YAML列表绑定到对象列表的逻辑：

```java
@Component
@ConfigurationProperties(prefix = "application")
public class ApplicationProps {

    private List<Object> profiles;

    // getter and setter
}
```

**ApplicationProps类需要用@ConfigurationProperties修饰，以表达将所有具有指定前缀的YAML属性映射到ApplicationProps对象的意图**。

要绑定配置文件列表，我们只需要定义一个List类型的字段，@ConfigurationProperties注解将处理其余部分。

请注意，我们使用@Component将ApplicationProps类注册为普通Spring Bean。**因此，我们可以像任何其他Spring Bean一样将其注入到其他类中**。

最后，我们将ApplicationProps Bean注入到测试类中，并验证我们的配置文件YAML列表是否正确注入为List<Object\>：

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(initializers = ConfigDataApplicationContextInitializer.class)
@EnableConfigurationProperties(value = ApplicationProps.class)
class YamlSimpleListUnitTest {

    @Autowired
    private ApplicationProps applicationProps;

    @Test
    public void whenYamlList_thenLoadSimpleList() {
        assertThat(applicationProps.getProfiles().get(0)).isEqualTo("dev");
        assertThat(applicationProps.getProfiles().get(4).getClass()).isEqualTo(Integer.class);
        assertThat(applicationProps.getProfiles().size()).isEqualTo(5);
    }
}
```

## 4. 将YAML列表绑定到复杂列表

现在让我们深入了解如何将嵌套的YAML列表注入复杂的结构化List中。

首先，让我们向application.yml添加一些嵌套列表：

```yaml
application:
    // ...
    props:
        -
        name: YamlList
        url: http://yamllist.dev
        description: Mapping list in Yaml to list of objects in Spring Boot
        -
            ip: 10.10.10.10
            port: 8091
        -
            email: support@yamllist.dev
            contact: http://yamllist.dev/contact
    users:
        -
            username: admin
            password: admin@10@
            roles:
                - READ
                - WRITE
                - VIEW
                - DELETE
        -
            username: guest
            password: guest@01
            roles:
                - VIEW
```

在此示例中，我们将props属性绑定到List<Map<String, Object\>\>。类似地，我们将users映射到User对象列表中。

由于props条目的每个元素都包含不同的键，我们可以将其作为Map的列表注入。请务必查看我们的文章，了解如何[在Spring Boot中从YAML文件注入Map](https://www.baeldung.com/spring-yaml-inject-map)。

然而，对于users来说，所有元素共享相同的键，**因此为了简化其映射，我们可能需要创建一个专门的User类来将键封装为字段**：

```java
public class ApplicationProps {

    // ...

    private List<Map<String, Object>> props;
    private List<User> users;

    // getters and setters

    public static class User {
        
        private String username;
        private String password;
        private List<String> roles;

        // getters and setters
    }
}
```

现在我们验证嵌套的YAML列表是否正确映射：

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(initializers = ConfigDataApplicationContextInitializer.class)
@EnableConfigurationProperties(value = ApplicationProps.class)
class YamlComplexListsUnitTest {

    @Autowired
    private ApplicationProps applicationProps;

    @Test
    public void whenYamlNestedLists_thenLoadComplexLists() {
        assertThat(applicationProps.getUsers().get(0).getPassword()).isEqualTo("admin@10@");
        assertThat(applicationProps.getProps().get(0).get("name")).isEqualTo("YamlList");
        assertThat(applicationProps.getProps().get(1).get("port").getClass()).isEqualTo(Integer.class);
    }
}
```

## 5. 总结

在本文中，我们学习了如何将YAML列表映射到Java List。

我们还检查了如何将复杂列表绑定到自定义POJO。