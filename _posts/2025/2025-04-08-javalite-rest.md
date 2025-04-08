---
layout: post
title:  JavaLite指南-构建RESTful CRUD应用程序
category: webmodules
copyright: webmodules
excerpt: JavaLite
---

## 1. 简介

**[JavaLite](http://javalite.io/)是一个框架集合，用于简化每个开发人员在构建应用程序时必须处理的常见任务**。

在本教程中，我们将了解专注于构建简单API的JavaLite功能。

## 2. 设置

在本教程中，我们将创建一个简单的RESTful CRUD应用程序。为此，我们将使用ActiveWeb和ActiveJDBC-JavaLite集成的两个框架。

那么，让我们开始并添加我们需要的第一个依赖：

```xml
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activeweb</artifactId>
    <version>1.15</version>
</dependency>
```

ActiveWeb工件包含ActiveJDBC，因此无需单独添加。请注意，可以在Maven Central中找到最新的[activeweb版本](https://mvnrepository.com/artifact/org.javalite/activeweb)。

我们需要的第二个依赖是数据库连接器，在这个例子中，我们将使用MySQL，因此我们需要添加：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.45</version>
</dependency>
```

同样，可以在Maven Central上找到最新的[mysql-connector-java](https://mvnrepository.com/artifact/mysql/mysql-connector-java)依赖。

我们必须添加的最后一个依赖是JavaLite特有的：

```xml
<plugin>
    <groupId>org.javalite</groupId>
    <artifactId>activejdbc-instrumentation</artifactId>
    <version>1.4.13</version>
    <executions>
        <execution>
            <phase>process-classes</phase>
            <goals>
                <goal>instrument</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

最新的[activejdbc-instrumentation](https://mvnrepository.com/artifact/org.javalite/activejdbc-instrumentation)插件也可以在Maven Central中找到。

在添加所有这些依赖后，在开始使用实体、表和映射之前，我们将确保其中一个[受支持的数据库](http://javalite.io/activejdbc#supported-databases)已启动并正在运行。如前所述，我们将使用MySQL。

## 3. 对象关系映射

### 3.1 映射和插桩

让我们首先**创建一个作为我们主要实体的Product类**：

```java
public class Product {}
```

并且，我们还**为其创建相应的表**：

```sql
CREATE TABLE PRODUCTS (
    id int(11) DEFAULT NULL auto_increment PRIMARY KEY,
    name VARCHAR(128)
);
```

最后，我们可以**修改Product类来进行映射**：

```java
public class Product extends Model {}
```

我们只需要扩展org.javalite.activejdbc.Model类，**ActiveJDBC从数据库推断出DB模式参数**，借助此功能，**无需添加Getter和Setter或任何注解**。

此外，ActiveJDBC会自动识别Product类需要映射到PRODUCTS表，它利用英语词形变化将模型的单数形式转换为表的复数形式。当然，它也能处理异常。

为了让映射工作，我们还需要做最后一件事：检测。**检测是ActiveJDBC所需的一个额外步骤**，它允许我们使用Product类，就好像它有Getter、Setter和类似DAO的方法一样。

运行检测后，我们将能够做如下事情：

```java
Product p = new Product();
p.set("name","Bread");
p.saveIt();
```

或者：

```java
List<Product> products = Product.findAll();
```

这就是activejdbc-instrumentation插件的作用所在，由于我们的pom中已经有了依赖，我们应该看到在构建过程中被检测的类：

```shell
...
[INFO] --- activejdbc-instrumentation:1.4.11:instrument (default) @ javalite ---
**************************** START INSTRUMENTATION ****************************
Directory: ...\tutorials\java-lite\target\classes
Instrumented class: .../tutorials/java-lite/target/classes/app/models/Product.class
**************************** END INSTRUMENTATION ****************************
...
```

接下来，我们将创建一个简单的测试来确保它正常运行。

### 3.2 测试

最后，为了测试我们的映射，我们将遵循三个简单的步骤：打开与数据库的连接，保存新Product并检索它：

```java
@Test
public void givenSavedProduct_WhenFindFirst_ThenSavedProductIsReturned() {
    Base.open(
            "com.mysql.jdbc.Driver",
            "jdbc:mysql://localhost/dbname",
            "user",
            "password");

    Product toSaveProduct = new Product();
    toSaveProduct.set("name", "Bread");
    toSaveProduct.saveIt();

    Product savedProduct = Product.findFirst("name = ?", "Bread");

    assertEquals(
            toSaveProduct.get("name"),
            savedProduct.get("name"));
}
```

请注意，仅凭一个空的模型和插桩，就可以实现所有这些(以及更多)。

## 4. 控制器

现在我们的映射已经准备好了，可以开始考虑我们的应用程序及其CRUD方法。

为此，我们将使用处理HTTP请求的控制器。

让我们创建ProductsController：

```java
@RESTful
public class ProductsController extends AppController {

    public void index() {
        // ...
    }
}
```

通过此实现，ActiveWeb将自动将index()方法映射到以下URI：

```text
http://<host>:<port>/products
```

用@RESTful标注的控制器提供了一组固定的方法，**这些方法会自动映射到不同的URI**，让我们看看哪些方法对我们的CRUD示例有用：

|  控制器方法   | HTTP方法|  URI   |                              |
|:--------:| :--------: |:------:| :--------------------------: |
|  CREATE  | create() |  POST  |   http://host:port/products    |
| READ ONE | show() |  GET   | http://host:port/products/{id} |
| READ ALL | index() |  GET   |   http://host:port/products    |
|  UPDATE  | update() |  PUT   | http://host:port/products/{id} |
|  DELETE  | destroy() | DELETE | http://host:port/products/{id} |

如果我们将这组方法添加到我们的ProductsController中：

```java
@RESTful
public class ProductsController extends AppController {

    public void index() {
        // code to get all products
    }

    public void create() {
        // code to create a new product
    }

    public void update() {
        // code to update an existing product
    }

    public void show() {
        // code to find one product
    }

    public void destroy() {
        // code to remove an existing product 
    }
}
```

在继续我们的逻辑实现之前，我们将快速看一下需要配置的一些内容。

## 5. 配置

ActiveWeb主要基于约定，项目结构就是一个例子；**ActiveWeb项目需要遵循预定义的包布局**：

```text
src
 |----main
       |----java.app
       |     |----config
       |     |----controllers
       |     |----models
       |----resources
       |----webapp
             |----WEB-INF
             |----views
```

我们需要查看一个特定的包-pp.config。

在该包中我们将创建三个类：

```java
public class DbConfig extends AbstractDBConfig {
    @Override
    public void init(AppContext appContext) {
        this.configFile("/database.properties");
    }
}
```

此类使用项目根目录中包含所需参数的属性文件配置数据库连接：

```properties
development.driver=com.mysql.jdbc.Driver
development.username=user
development.password=password
development.url=jdbc:mysql://localhost/dbname
```

这将自动创建连接，取代我们在映射测试第一行所做的操作。

我们需要在app.config包中包含的第二个类是：

```java
public class AppControllerConfig extends AbstractControllerConfig {
 
    @Override
    public void init(AppContext appContext) {
        add(new DBConnectionFilter()).to(ProductsController.class);
    }
}
```

**此代码将把我们刚刚配置的连接绑定到我们的控制器**。

第三个类将配置我们的应用程序的上下文：

```java
public class AppBootstrap extends Bootstrap {
    public void init(AppContext context) {}
}
```

创建这三个类之后，关于配置的最后一件事是在webapp/WEB-INF目录下创建web.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns=...>

    <filter>
        <filter-name>dispatcher</filter-name>
        <filter-class>org.javalite.activeweb.RequestDispatcher</filter-class>
        <init-param>
            <param-name>exclusions</param-name>
            <param-value>css,images,js,ico</param-value>
        </init-param>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>dispatcher</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

现在配置已经完成，我们可以继续添加我们的逻辑。

## 6. 实现CRUD逻辑

借助我们的Product类提供的类似DAO的功能，添加基本的CRUD功能非常简单：

```java
@RESTful
public class ProductsController extends AppController {

    private ObjectMapper mapper = new ObjectMapper();

    public void index() {
        List<Product> products = Product.findAll();
        // ...
    }

    public void create() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        Product p = new Product();
        p.fromMap(payload);
        p.saveIt();
        // ...
    }

    public void update() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        String id = getId();
        Product p = Product.findById(id);
        p.fromMap(payload);
        p.saveIt();
        // ...
    }

    public void show() {
        String id = getId();
        Product p = Product.findById(id);
        // ...
    }

    public void destroy() {
        String id = getId();
        Product p = Product.findById(id);
        p.delete();
        // ...
    }
}
```

但是，这还没有返回任何内容；为了做到这一点，我们必须创建一些视图。

## 7. 视图

**ActiveWeb使用[FreeMarker](http://freemarker.org/)作为模板引擎，其所有模板都应位于src/main/webapp/WEB-INF/views下**。

在该目录中，我们将把视图放在名为products的文件夹中(与我们的控制器相同)，让我们创建第一个名为_product.ftl的模板：

```text
{
    "id" : ${product.id},
    "name" : "${product.name}"
}
```

到目前为止，很明显这是一个JSON响应。当然，这只适用于单个Product，所以让我们继续创建另一个名为index.ftl的模板：

```text
[<@render partial="product" collection=products/>]
```

**这将基本上呈现一个名为projects的集合，每个Product都由_product.ftl格式化**。

最后，**我们需要将控制器的结果绑定到相应的视图**：

```java
@RESTful
public class ProductsController extends AppController {

    public void index() {
        List<Product> products = Product.findAll();
        view("products", products);
        render();
    }

    public void show() {
        String id = getId();
        Product p = Product.findById(id);
        view("product", p);
        render("_product");
    }
}
```

在第一种情况下，我们将products列表分配给名为products的模板集合。

然后，由于我们没有指定任何视图，因此将使用index.ftl。

在第二种方法中，我们将产品p分配给视图中的product产品，并明确说明要渲染哪个视图。

我们还可以创建一个视图message.ftl：

```text
{
    "message" : "${message}",
    "code" : ${code}
}
```

然后从我们的任何ProductsController的方法中调用它：

```java
view("message", "There was an error.", "code", 200);
render("message");
```

现在让我们看看最终的ProductsController：

```java
@RESTful
public class ProductsController extends AppController {

    private ObjectMapper mapper = new ObjectMapper();

    public void index() {
        view("products", Product.findAll());
        render().contentType("application/json");
    }

    public void create() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        Product p = new Product();
        p.fromMap(payload);
        p.saveIt();
        view("message", "Successfully saved product id " + p.get("id"), "code", 200);
        render("message");
    }

    public void update() {
        Map payload = mapper.readValue(getRequestString(), Map.class);
        String id = getId();
        Product p = Product.findById(id);
        if (p == null) {
            view("message", "Product id " + id + " not found.", "code", 200);
            render("message");
            return;
        }
        p.fromMap(payload);
        p.saveIt();
        view("message", "Successfully updated product id " + id, "code", 200);
        render("message");
    }

    public void show() {
        String id = getId();
        Product p = Product.findById(id);
        if (p == null) {
            view("message", "Product id " + id + " not found.", "code", 200);
            render("message");
            return;
        }
        view("product", p);
        render("_product");
    }

    public void destroy() {
        String id = getId();
        Product p = Product.findById(id);
        if (p == null) {
            view("message", "Product id " + id + " not found.", "code", 200);
            render("message");
            return;
        }
        p.delete();
        view("message", "Successfully deleted product id " + id, "code", 200);
        render("message");
    }

    @Override
    protected String getContentType() {
        return "application/json";
    }

    @Override
    protected String getLayout() {
        return null;
    }
}
```

至此，我们的应用程序已完成并准备运行它。

## 8. 运行应用程序

我们将使用Jetty插件：

```xml
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.8.v20171121</version>
</plugin>
```

在Maven Central中查找最新的[jetty-maven-plugin](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-maven-plugin)。

可以运行我们的应用程序了：

```shell
mvn jetty:run
```

让我们创建几个Product：

```shell
$ curl -X POST http://localhost:8080/products 
  -H 'content-type: application/json' 
  -d '{"name":"Water"}'
{
    "message" : "Successfully saved product id 1",
    "code" : 200
}
$ curl -X POST http://localhost:8080/products 
  -H 'content-type: application/json' 
  -d '{"name":"Bread"}'
{
    "message" : "Successfully saved product id 2",
    "code" : 200
}
```

读取它们：

```shell
$ curl -X GET http://localhost:8080/products
[
    {
        "id" : 1,
        "name" : "Water"
    },
    {
        "id" : 2,
        "name" : "Bread"
    }
]
```

更新其中一个：

```shell
$ curl -X PUT http://localhost:8080/products/1 
  -H 'content-type: application/json' 
  -d '{"name":"Juice"}'
{
    "message" : "Successfully updated product id 1",
    "code" : 200
}
```

读取我们刚刚更新的内容：

```shell
$ curl -X GET http://localhost:8080/products/1
{
    "id" : 1,
    "name" : "Juice"
}
```

最后，删除一个：

```shell
$ curl -X DELETE http://localhost:8080/products/2
{
    "message" : "Successfully deleted product id 2",
    "code" : 200
}
```

## 9. 总结

JavaLite有很多工具可以帮助开发人员在几分钟内启动并运行应用程序。