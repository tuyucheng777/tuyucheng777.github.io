---
layout: post
title:  什么是样板代码
category: programming
copyright: programming
excerpt: 编程
---

## 1. 概述

在编程中，我们偶尔会遇到“样板代码”这个术语。该术语起源于金属印刷行业，在金属印刷中，样板指的是可以重复用于相同目的的金属印刷板。

在软件开发中，“样板”的含义大致相同，尽管它的含义更丰富。本文将介绍编程环境中的样板代码，然后，我们将探讨一些实际使用样板代码的案例。

最后，我们将讨论如何避免编写重复的样板代码以提高生产力。

## 2. 什么是样板代码

**样板代码是一段重复的代码，几乎无需修改即可反复使用。样板代码通常代表整个代码库的标准实践和模式，从而创建一种开发人员可以遵循的熟悉结构**。

在编程中，我们经常在面向对象语言中看到样板代码，其中类的私有属性通过访问器和修改器方法进行访问和修改。同样，在HTML等标记语言中，样板代码位于head部分，并且在多个HTML网页中都类似。

在下一节中，我们将深入研究几个示例，看看样板代码是什么样的。

## 3. 样板代码实践

样板代码常见于C++、Java和Go等冗长的语言中。有时，这些冗长的语言往往是面向对象的。相比之下，函数式编程语言往往简洁得多。

### 3.1 面向对象语言

在面向对象语言中，我们经常处理类和封装。因此，我们将类属性声明为私有的，客户端访问或修改它们的唯一方法是使用[Setter和Getter](https://www.baeldung.com/java-why-getters-setters)方法。

因此，**我们为代码库中的大多数类重复编写这些Setter和Getter，这导致了样板代码**，让我们在Java中观察一下：

```java
public class Taco {
    private String name;
    private double price;
    private List<Ingredient> ingredients;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public List<Ingredient> getIngredients() {
        return ingredients;
    }

    public void setIngredients(List<Ingredient> ingredients) {
        this.ingredients = ingredients;
    }
}
```

值得注意的是，每个属性都有一对Setter和Getter，它们在整个类中遵循类似的模式。

此外，Java中的类还可以包含[equals](https://www.baeldung.com/java-equals-hashcode-contracts#equals)、[hashCode](https://www.baeldung.com/java-equals-hashcode-contracts#hashcode)和[toString](https://www.baeldung.com/java-tostring)方法，分别用于比较、哈希码生成和字符串表示：

```java
public class Taco {
    // ...
    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Taco taco = (Taco) o;
        return Double.compare(price, taco.price) == 0 &&
                Objects.equals(name, taco.name) &&
                Objects.equals(ingredients, taco.ingredients);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, price, ingredients);
    }

    @Override
    public String toString() {
        return String.format("Taco{name='%s', price=%.2f, ingredients=%s}", name, price, ingredients);
    }
}
```

**这些方法在许多Java类中实现，但略有不同**。具体实现可能略有不同，但总体结构保持一致。

稍后，我们将学习如何尽量减少自己编写样板代码。

### 3.2 标记语言

在HTML和XML等标记语言中，我们经常在head部分看到样板代码，让我们来看一个典型的HTML样板代码：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Title</title>
    <meta name="description" content="Description of the web page">
    <meta name="keywords" content="keyword1, keyword2, keyword3">
    <link rel="stylesheet" href="/static/styles.css">
    <link rel="icon" href="/static/favicon.ico" type="image/x-icon">
    <script src="/static/scripts/script.js" defer></script>
</head>
<body>
    <!-- Webpage elements -->
</body>
</html>
```

在示例中，我们看到了HTML网页的一个常见起始点。head元素由meta、title、link和script元素组成，**这些元素提供了网页正常运行所需的[元数据](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/The_head_metadata_in_HTML)和必要资源。因此，它们出现在我们在网上遇到的大多数网页上**。

另一方面，我们在XML中也看到了类似的模式：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <!-- Document elements -->
</root>
```

xml元素定义文档的版本和编码，[根](https://en.wikipedia.org/wiki/Root_element)元素充当其他元素的容器，我们可以用更具描述性的名称替换根元素。

我们经常在使用XML的配置文件中遇到这种样板代码，我们将在下文中介绍。

### 3.3 配置文件

在软件开发过程中，程序员还会处理配置文件，**这些配置文件通常在各个项目中遵循一致的结构**。

例如，在基于[Maven](https://www.baeldung.com/maven-guide)的项目中，有一个[pom.xml](https://www.baeldung.com/maven#introduction-2)文件，它允许我们管理项目依赖项、配置和构建设置；让我们看一下[Spring Boot](https://www.baeldung.com/spring-boot)项目的pom.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>cn.tuyucheng.taketoday</groupId>
    <artifactId>example</artifactId>
    <version>1.0.0</version>
    <name>example</name>
    <description>An example pom.xml to demonstrate boilerplate code</description>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- Additional dependencies -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

其他项目可能有不同的pom.xml文件，但基础结构保持不变。在顶部，我们定义项目元数据，然后定义项目依赖项和配置。**我们可以看到，配置使用了groupId和artifactId等元素，这些元素在其他Spring Boot项目中也很常见**。

这种模式在其他项目中也存在，例如，Node项目包含一个[package.json](https://docs.npmjs.com/files/package.json/)文件，Rust项目包含[cargo.toml](https://doc.rust-lang.org/cargo/reference/manifest.html)，而.NET项目包含[.csproj](https://learn.microsoft.com/en-us/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/understanding-the-project-file)文件。

## 4. 避免样板代码

在编程中，样板代码有时会显得冗长乏味，令人厌烦。不仅如此，它还会对项目的开发时间产生负面影响，并使代码库变得混乱。

幸运的是，有很多解决方案可以解决这个问题。

### 4.1 库

**大多数编程语言都有提供常见模式的样板代码生成的库**，例如，Java中的[Lombok](https://www.baeldung.com/lombok-configuration-system)项目就提供了一个库，可以帮助我们最大限度地减少样板代码的编写。

它可以处理Getter、Setter、构造函数、equals、hashCode等的自动方法生成：

```java
import lombok.Data;

@Data
public class Taco {
    private String name;
    private double price;
    private List<Ingredient> ingredients;
}
```

在示例中，我们使用[@Data](https://projectlombok.org/features/Data)标注该类，Lombok在编译时生成上述方法，这提高了生产力，并使代码更具可读性。

### 4.2 IDE中的代码生成

**大多数集成开发环境(IDE)都带有代码生成功能**；例如，IntelliJ IDEA、Eclipse和Netbeans都提供了生成类样板的工具。如果我们不需要使用第三方库，这些工具就非常有用。

举个例子，让我们在IntelliJ IDEA中看看这个[功能](https://www.jetbrains.com/help/idea/generating-code.html)。我们只需打开一个Java文件，将光标放在类主体的任意位置。然后，前往“Code” → “Generate”：

![](/assets/images/2025/programming/boilerplatecode01.png)

在菜单中，我们选择所需的选项，它将向我们显示一个对话框：

![](/assets/images/2025/programming/boilerplatecode02.png)

在生成Getter和Setter函数时，我们需要选择相关的属性。然后，点击“OK”即可生成代码。同样，我们也可以生成构造函数以及equals、toString和hashCode方法。

### 4.3 插件

大多数IDE和代码编辑器都支持插件，有些插件是预装的。例如，VSCode、IntelliJ和许多其他IDE都预装了[Emmet](https://emmet.io/)插件，这让我们可以快速生成HTML/XML标记。

例如，如果我们想要创建一个包含列表项的无序列表，我们可以输入ul.list > li.list-item * 10并按Tab：

```xml
<ul class="list">
    <li class="list-item"></li>
    <li class="list-item"></li>
    <li class="list-item"></li>
    <li class="list-item"></li>
    <li class="list-item"></li>
</ul>
```

**同样，有一些代码片段插件可以让我们快速生成简单的代码**。例如，在IntelliJ中，我们可以输入sout并按Tab键，在光标位置生成System.out.println();。

### 4.4 项目初始化器

在软件工程中，**有一些模式非常常见，开发人员创建了项目启动器或项目模板，以帮助以最少的设置和样板代码启动新项目**。它包括项目结构、默认配置和初始代码。

其中一个项目启动器是[Spring Initializr](https://start.spring.io/)，它可以帮助我们启动一个新的Spring Boot项目。

## 5. 总结

在本文中，我们了解了软件开发中的样板代码。我们讨论了编程语言、配置文件和标记语言中的样板代码是什么样的。

之后，我们学习了如何通过使用库、代码生成和项目启动器等各种方法来减少样板代码。