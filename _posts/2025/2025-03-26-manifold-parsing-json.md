---
layout: post
title:  使用Manifold解析JSON
category: libraries
copyright: libraries
excerpt: Manifold
---

## 1. 简介

在本文中，我们将介绍[Manifold JSON](https://github.com/manifold-systems/manifold/tree/master/manifold-deps-parent/manifold-json)，它能够对JSON文档进行JSON Schema感知解析。我们将了解它是什么、可以用它做什么以及如何使用它。

Manifold是一套编译器插件，每个插件都提供了一些我们可以在应用程序中使用的高效功能。在本文中，我们将探索如何使用它根据提供的[JSON Schema](https://www.baeldung.com/introduction-to-json-schema-in-java)文件解析和构建JSON文档。

## 2. 安装

**在使用Manifold之前，我们需要确保它可用于我们的编译器**。最重要的集成是作为Java编译器的插件，我们通过适当配置我们的Maven或Gradle项目来实现这一点。此外，Manifest为一些IDE提供了插件，使我们的开发过程更加轻松。

## 2.1 Maven中的编译器插件

**在Maven中将Manifold集成到我们的应用程序中需要我们添加[依赖](https://mvnrepository.com/artifact/systems.manifold/manifold-json-rt)和[编译器插件](https://mvnrepository.com/artifact/systems.manifold/manifold-json)，它们都需要是同一版本，在撰写本文时为[2024.1.20](https://mvnrepository.com/artifact/systems.manifold/manifold-json/2024.1.20)**。

添加依赖与其他依赖相同：

```xml
<dependency>
    <groupId>systems.manifold</groupId>
    <artifactId>manifold-json-rt</artifactId>
    <version>2024.1.20</version>
</dependency>
```

添加我们的编译器插件需要我们在模块中配置maven-compiler-plugin：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <configuration>
        <compilerArgs>
            <!-- Configure manifold plugin-->
            <arg>-Xplugin:Manifold</arg>
        </compilerArgs>
        <!-- Add the processor path for the plugin -->
        <annotationProcessorPaths>
            <path>
                <groupId>systems.manifold</groupId>
                <artifactId>manifold-json</artifactId>
                <version>2024.1.20</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

这里我们在执行编译器时添加-Xplugin:Manifold命令行参数，以及添加Manifold JSON注解处理器。

这个注解处理器负责从我们的JSON模式文件生成代码的所有工作。

## 2.2 Gradle中的编译器插件

**使用Gradle将Manifold集成到我们的应用程序中需要实现相同的功能，但配置稍微简单一些**。Gradle支持以与任何其他依赖相同的方式将注解处理器添加到Java编译器：

```groovy
dependencies {
    implementation 'systems.manifold:manifold-json-rt:2024.1.20'
    annotationProcessor 'systems.manifold:manifold-json:2024.1.20'
    testAnnotationProcessor 'systems.manifold:manifold-json:2024.1.20'
}
```

不过，我们仍然需要确保-Xplugin:Manifold参数传递给编译器：

```groovy
tasks.withType(JavaCompile) {
    options.compilerArgs += ['-Xplugin:Manifold']
}
```

此时，我们已准备好开始使用Manifold JSON。

## 2.3 IDE插件

**除了Maven或Gradle构建中的插件外，Manifold还为IntelliJ和Android Studio提供了插件**，这些可以从标准插件市场安装：

![](/assets/images/2025/libraries/manifoldparsingjson01.png)

一旦安装，这将允许我们之前在项目中设置的编译器插件在IDE内部编译代码时使用，而不是仅仅依靠Maven或Gradle构建来做正确的事情。

## 3. 使用JSON Schema定义类

**一旦我们在项目中设置了Manifold JSON，我们就可以使用它来定义类，我们通过在src/main/resources(或src/test/resources)目录中编写JSON Schema文件来实现这一点**。然后，文件的完整路径将成为完全限定的类名。

例如，我们可以通过在src/main/resources/cn/tuyucheng/taketoday/manifold/SimpleUser.json中编写JSON模式来创建cn.tuyucheng.taketoday.manifold.SimpleUser类：

```json
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "$id": "http://tuyucheng.com/manifold/SimpleUser.json",
    "type": "object",
    "properties": {
        "username": {
            "type": "string",
            "maxLength": 100
        },
        "name": {
            "type": "string",
            "maxLength": 80
        },
        "email": {
            "type": "string",
            "format": "email",
            "maxLength": 260
        }
    },
    "required": [
        "username",
        "name"
    ]
}
```

这表示一个具有三个字段的类-username、name和email，所有字段均为String类型。

但是，Manifold不会做任何它不需要做的事情。因此，如果没有任何东西引用它，写入此文件将不会生成任何代码。不过，此引用可以是编译器需要的任何内容-甚至像变量定义这样简单的东西。

## 4. 实例化Manifold类

一旦我们有了类定义，我们就想使用它。然而，我们很快就会发现这些不是常规类。Manifold将我们预期的所有类都生成为Java接口，这意味着我们不能简单地使用new创建新实例。

但是，**生成的代码提供了一个静态create()方法，我们可以将其用作构造函数方法**。此方法将需要所有必需的值，这些值按照在required数组中指定的顺序排列。例如，上面的JSON模式将生成以下内容：

```java
public static SimpleUser create(String username, String name);
```

然后我们有可以用来填充任何剩余字段的设置器。

因此我们可以使用以下命令创建SimpleUser类的新实例：

```java
SimpleUser user = SimpleUser.create("testuser", "Test User");
user.setEmail("testuser@example.com");
```

**除此之外，Manifold还为我们提供了一个build()方法，可用于构建器模式**。它接收相同的一组必需参数，但返回一个构建器对象，我们可以使用它来填充任何其他字段：

```java
SimpleUser user = SimpleUser.builder("testuser", "Test User")
    .withEmail("testuser@example.com")
    .build();
```

如果我们使用的字段既是可选的又是只读的，那么构建器模式是我们可以提供值的唯一方法-一旦我们得到了创建的实例，我们就不能再设置这些字段的值。

## 5. 解析JSON

**一旦我们获得了生成的类，我们就可以使用它们与JSON源进行交互。我们生成的类提供了从各种类型的输入加载数据并将其直接解析到目标类中的方法**。

进入此路由始终使用我们生成的类上的静态load()方法触发，这为我们提供了一个manifold.json.rt.api.Loader<\>接口，该接口为我们提供加载JSON数据的各种方法。

其中最简单的就是能够解析以某种方式提供给我们的JSON字符串，这是使用fromJson()方法完成的，该方法接收一个字符串并返回我们类的完整实例：

```java
SimpleUser user = SimpleUser.load().fromJson("""
    {
        "username": "testuser",
        "name": "Test User",
        "email": "testuser@example.com"
    }
    """);
```

如果成功，这将为我们带来想要的结果。如果失败，我们将得到一个RuntimeException，它包装了一个manifold.rt.api.ScriptException，准确指出了哪里出了问题。

我们还可以采用其他多种方式来提供数据：

- fromJsonFile()从java.io.File读取数据
- fromJsonReader()从java.io.Reader读取数据
- fromJsonUrl()从URL读取数据-在这种情况下，Manifold将去获取该URL指向的数据

但请注意，fromJsonUrl()方法最终使用java.net.URL.openStream()来读取数据，这可能不像我们希望的那样高效。

所有这些都以相同的方式工作-我们使用适当的数据源调用它们，它会返回一个完整格式的对象，或者如果数据无法解析则抛出RuntimeException：

```java
InputStream is = getClass().getResourceAsStream("/cn/tuyucheng/taketodaymanifold/simpleUserData.json");
InputStreamReader reader = new InputStreamReader(is);
SimpleUser user = SimpleUser.load().fromJsonReader(reader);
```

## 6. 生成JSON

**除了能够将JSON解析为我们的对象之外，我们还可以反过来从我们的对象生成JSON**。

Manifold生成load()方法将JSON加载到我们的对象中，同样，我们也有一个write()方法从我们的对象写入JSON。这将返回manifold.json.rt.api.Writer的一个实例，它为我们提供了从我们的对象写入JSON的方法。

其中最简单的是toJson()方法，它将JSON以字符串形式返回，我们可以从中执行任何我们想做的事情：

```java
SimpleUser user = SimpleUser.builder("testuser", "Test User")
    .withEmail("testuser@example.com")
    .build();
String json = user.write().toJson();
```

另外，我们可以将JSON写入任何实现Appendable接口的对象，其中包括java.io.Writer接口或StringBuilder和StringBuffer类：

```java
SimpleUser user = SimpleUser.builder("testuser", "Test User")
    .withEmail("testuser@example.com")
    .build();
user.write().toJson(writer);
```

**然后，该写入器可以写入任何目标-包括内存缓冲区、文件或网络连接**。

## 7. 替代格式

**manifold-json的主要目的是能够解析和生成JSON内容，但是，我们也可以处理其他格式-CSV、XML和YAML**。

对于每个格式，我们都需要向项目添加特定的依赖-systems.manifold:manifold-csv-rt用于CSV，systems.manifold:manifold-xml-rt用于XML，systems.manifold:manifold-yaml-rt用于YAML，我们可以根据需要添加任意数量的依赖。

完成后，我们就可以使用Manifold提供给我们的Loader和Writer接口上的适当方法：

```java
SimpleUser user = SimpleUser.load().fromXmlReader(reader);
String yaml = user.write().toYaml();
```

## 8. JSON Schema特性

Manifold使用JSON Schema文件来描述类的结构，我们可以使用这些文件来描述要生成的类及其中存在的字段。但是，**我们可以使用JSON Schema描述更多内容，并且Manifold JSON支持其中一些额外的功能**。

### 8.1 只读和只写字段

**在模式中标记为只读的字段无需设置器即可生成**，这意味着我们可以在构造时或解析输入文件时填充它们，但之后永远无法更改这些值：

```json
"username": {
    "type": "string",
    "readOnly": true
},
```

**相反，生成器会在没有Getter的情况下生成模式中创建标记为只写的字段**。这意味着我们可以随心所欲地填充它们-在构造时、从输入文件解析或使用Setter，但我们永远无法取回值。

```json
"mfaCode": {
    "type": "string",
    "writeOnly": true
},
```

请注意，系统仍会在任何生成的输出中呈现只写属性，因此我们可以通过该路径访问它们。但是，我们无法从Java类中读取这些属性。

### 8.2 格式化类型

**JSON Schema中的某些类型允许我们提供额外的格式信息**。例如，我们可以指定一个字段是字符串，但格式为date-time，这暗示了我们应该在这些字段中看到的数据类型的模式。

**Manifold JSON将尽力理解这些格式并为它们生成适当的Java类型**，例如，格式化为日date-time的string将在我们的代码中生成为java.time.LocalDateTime。

### 8.3 附加属性

JSON Schema让我们能够通过使用additionalProperties和patternProperties来定义开放式模式。

**additionalProperties标志表示该类型可以具有任意数量的任何类型的额外属性**。本质上，这意味着该模式允许任何其他JSON匹配。Manifold JSON默认将其定义为true，但如果愿意，我们可以在模式中将其明确设置为false。

如果将其设置为true，则Manifold将在生成的类上提供两个附加方法-get(name)和put(name, value)。使用这些方法，我们可以处理任意字段：

```java
SimpleUser user = SimpleUser.builder("testuser", "Test User")
    .withEmail("testuser@example.com")
    .build();

user.put("note", "This isn't specified in the schema");
```

请注意，这些值没有经过验证，这包括检查名称是否与其他定义的字段冲突。因此，这可用于覆盖我们模式中定义的字段-包括忽略type或readOnly等验证规则。

**JSON Schema还支持更高级的概念“模式属性”，我们将其定义为完整属性定义，但我们使用正则表达式来定义属性名称，而不是固定字符串**。例如，此定义允许我们将字段note0至note9全部指定为字符串类型：

```json
"patternProperties": {
    "note[0-9]": {
        "type": "string"
    }
}
```

Manifold JSON对此有部分支持。它不会生成显式代码来处理此问题，而是将类型中任何patternProperties的存在视为与将additionalProperties设置为true相同。这包括我们是否显式将additionalProperties设置为false。

### 8.4 嵌套类型

**JSON Schema支持我们定义一种嵌套在另一种中的类型，而不需要定义顶层的所有内容并使用引用**。这对于为我们的JSON提供本地化结构非常有用。例如：

```json
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "$id": "http://tuyucheng.com/manifold/User.json",
    "type": "object",
    "properties": {
        "email": {
            "type": "object",
            "properties": {
                "address": {
                    "type": "string",
                    "maxLength": 260
                },
                "verified": {
                    "type": "boolean"
                }
            },
            "required": ["address", "verified"]
        }
    }
}
```

这里我们定义了一个对象，其中一个字段是另一个对象。然后我们用JSON表示它，如下所示：

```json
{
    "email": {
        "address": "testuser@example.com",
        "verified": false
    }
}
```

Manifold JSON完全支持此功能，并将自动生成适当命名的内部类。然后我们可以像其他任何类一样使用这些类：

```java
User user = User.builder()
    .withEmail(User.email.builder("testuser@example.com", false).build())
    .build();
```

### 8.5 组合

**JSON Schema支持使用allOf、anyOf或oneOf关键字将不同类型组合在一起**，每个关键字都采用其他模式的集合(通过引用或直接指定)，并且生成的模式必须分别与所有模式匹配、至少匹配其中一个模式或完全匹配其中一个模式。Manifold JSON对这些关键字具有一定程度的支持。

**当使用allOf时，Manifold将生成一个包含所有组合类型定义的类**。如果我们以内联方式定义这些，系统将创建一个包含所有定义的单个类：

```json
"allOf": [
    {
        "type": "object",
        "properties": {
            "username": {
                "type": "string"
            }
        }
    },
    {
        "type": "object",
        "properties": {
            "roles": {
                "type": "array",
                "items": {
                    "type": "string"
                }
            }
        }
    }
]
```

或者，如果我们使用引用进行组合，则生成的接口将扩展我们引用的所有类型：

```json
"allOf": [
    {"$ref":  "#/definitions/user"},
    {"$ref":  "#/definitions/adminUser"}
]
```

在这两种情况下，我们都可以使用该生成的类，就好像它符合这两组定义一样：

```java
Composed.user.builder()
    .withUsername("testuser")
    .withRoles(List.of("admin"))
    .build()
```

**如果我们使用anyOf或oneOf，那么Manifold将生成能够以类型安全的方式接收替代选项的代码**。这要求我们使用引用，以便Manifold可以推断出类型名称：

```json
"anyOf": [
    {"$ref":  "#/definitions/Dog"},
    {"$ref":  "#/definitions/Cat"}
]
```

当这样做时，我们的类型获得了与不同选项交互的类型安全方法：

```java
Composed composed = Composed.builder()
    .withAnimalAsCat(Composed.Cat.builder()
        .withColor("ginger")
        .build())
    .build();

assertEquals("ginger", composed.getAnimalAsCat().getColor());
```

这里我们可以看到我们的构建器获得了一个withAnimalAsCat方法-其中“Animal”部分是外部对象中的字段名称，“Cat”部分是从我们的定义推断出的类型名称。我们的实际对象也以同样的方式获得了getAnimalAsCat和setAnimalAsCat方法。

## 9. 总结

在本文中，我们广泛介绍了Manifold JSON。