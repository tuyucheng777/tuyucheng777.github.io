---
layout: post
title:  Reflections简介
category: libraries
copyright: libraries
excerpt: Reflections
---

## 1. 简介

[Reflections](https://github.com/ronmamo/reflections)库充当类路径扫描器，它索引扫描到的元数据，并允许我们在运行时查询它。它还可以保存这些信息，因此我们可以在项目的任何阶段收集和使用这些信息，而无需再次扫描类路径。

在本教程中，我们将展示如何在Java项目中配置和使用Reflections库。

## 2. Maven依赖

要使用Reflections，我们需要在项目中包含它的依赖：

```xml
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.9.11</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/org.reflections/reflections)上找到该库的最新版本。

## 3. 配置反射

接下来，我们需要配置库，**配置的主要元素是URL和扫描器**。

URL告诉库要扫描类路径的哪些部分，而扫描器是扫描给定URL的对象。

如果没有配置扫描器，该库将使用TypeAnnotationsScanner和SubTypesScanner作为默认扫描器。

### 3.1 添加URL

我们可以通过提供配置的元素作为可变参数构造函数的参数或使用ConfigurationBuilder对象来配置Reflections。

例如，我们可以通过使用表示包名称、类或类加载器的字符串实例化Reflections来添加URL：

```java
Reflections reflections = new Reflections("cn.tuyucheng.taketoday.reflections");
Reflections reflections = new Reflections(MyClass.class);
Reflections reflections = new Reflections(MyClass.class.getClassLoader());
```

此外，由于Reflections具有可变参数构造函数，我们可以组合所有上述配置的类型来实例化它：

```java
Reflections reflections = new Reflections("cn.tuyucheng.taketoday.reflections", MyClass.class);
```

在这里，我们通过指定要扫描的包和类来添加URL。

我们可以使用ConfigurationBuilder来实现相同的结果：

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
    .setUrls(ClasspathHelper.forPackage("cn.tuyucheng.taketoday.reflections"))));
```

除了forPackage()方法之外，ClasspathHelper还提供了其他方法，例如 forClass()和forClassLoader()，用于将 URL 添加到配置中。

### 3.2 添加扫描器

**Reflections库附带许多内置扫描器**：

- FieldAnnotationsScanner：查找字段的注解
- MethodParameterScanner：扫描方法/构造函数，然后索引参数，并返回类型和参数注解
- MethodParameterNamesScanner：检查方法/构造函数，然后索引参数名称
- TypeElementsScanner：检查字段和方法，然后将完全限定名称存储为键，将元素存储为值
- MemberUsageScanner：扫描方法/构造函数/字段用法
- TypeAnnotationsScanner：查找类的运行时注解
- SubTypesScanner：搜索类的超类和接口，允许反向查找子类型
- MethodAnnotationsScanner：扫描方法的注解
- ResourcesScanner：将所有非类资源收集到一个集合中

我们可以将扫描器作为Reflections构造函数的参数添加到配置中。

例如，让我们从上面的列表中添加前两个扫描器：

```java
Reflections reflections = new Reflections("cn.tuyucheng.taketoday.reflections"), 
    new FieldAnnotationsScanner(), 
    new MethodParameterScanner());
```

再次，可以使用ConfigurationBuilder帮助类来配置这两个扫描器：

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
    .setUrls(ClasspathHelper.forPackage("cn.tuyucheng.taketoday.reflections"))
    .setScanners(new FieldAnnotationsScanner(), new MethodParameterScanner()));
```

### 3.3 添加ExecutorService

除了URL和扫描器之外，**Reflections还允许我们使用[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)异步扫描类路径**。

我们可以将其添加为Reflections构造函数的参数或通过ConfigurationBuilder添加：

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
    .setUrls(ClasspathHelper.forPackage("cn.tuyucheng.taketoday.reflections"))
    .setScanners(new SubTypesScanner(), new TypeAnnotationsScanner())
    .setExecutorService(Executors.newFixedThreadPool(4)));
```

另一个选项是直接调用useParallelExecutor()方法，此方法配置一个默认的FixedThreadPool ExecutorService，其大小等于可用核心处理器的数量。

### 3.4 添加过滤器

另一个重要的配置元素是过滤器，**过滤器告诉扫描器在扫描类路径时要包含什么和排除什么**。

举例来说，我们可以配置过滤器以排除测试包的扫描：

```java
Reflections reflections = new Reflections(new ConfigurationBuilder()
    .setUrls(ClasspathHelper.forPackage("cn.tuyucheng.taketoday.reflections"))
    .setScanners(new SubTypesScanner(), new TypeAnnotationsScanner())
    .filterInputsBy(new FilterBuilder().excludePackage("cn.tuyucheng.taketoday.reflections.test")));
```

现在，到目前为止，我们已经快速概述了Reflections配置的不同元素。接下来，我们将了解如何使用该库。

## 4. 使用反射进行查询

调用Reflections构造函数之一后，配置的扫描器将扫描所有提供的URL。然后，库将结果放入每个扫描器的Multimap存储中。因此，为了使用Reflections，我们需要通过调用提供的查询方法来查询这些存储。

让我们看一下这些查询方法的一些例子。

### 4.1 子类型

让我们首先检索Reflection提供的所有扫描器：

```java
public Set<Class<? extends Scanner>> getReflectionsSubTypes() {
    Reflections reflections = new Reflections("org.reflections", new SubTypesScanner());
    return reflections.getSubTypesOf(Scanner.class);
}
```

### 4.2 注解类型

接下来，我们可以获取实现给定注解的所有类和接口。

因此，让我们检索java.util.function包的所有函数接口：

```java
public Set<Class<?>> getJDKFunctionalInterfaces() {
    Reflections reflections = new Reflections("java.util.function", new TypeAnnotationsScanner());
    return reflections.getTypesAnnotatedWith(FunctionalInterface.class);
}
```

### 4.3 带注解的方法

现在，让我们使用MethodAnnotationsScanner获取所有用给定注解标注的方法：

```java
public Set<Method> getDateDeprecatedMethods() {
    Reflections reflections = new Reflections("java.util.Date", 
        new MethodAnnotationsScanner());
    return reflections.getMethodsAnnotatedWith(Deprecated.class);
}
```

### 4.4 带注解的构造函数

此外，我们可以获取所有已弃用的构造函数：

```java
public Set<Constructor> getDateDeprecatedConstructors() {
    Reflections reflections = new Reflections(
        "java.util.Date", 
        new MethodAnnotationsScanner());
    return reflections.getConstructorsAnnotatedWith(Deprecated.class);
}
```

### 4.5 方法参数

此外，我们可以使用MethodParameterScanner来查找具有给定参数类型的所有方法：

```java
public Set<Method> getMethodsWithDateParam() {
    Reflections reflections = new Reflections(
        java.text.SimpleDateFormat.class, 
        new MethodParameterScanner());
    return reflections.getMethodsMatchParams(Date.class);
}
```

### 4.6 方法的返回类型

此外，我们还可以使用相同的扫描器来获取具有给定返回类型的所有方法。

假设我们想要找到SimpleDateFormat中所有返回void的方法：

```java
public Set<Method> getMethodsWithVoidReturn() {
    Reflections reflections = new Reflections(
        "java.text.SimpleDateFormat", 
        new MethodParameterScanner());
    return reflections.getMethodsReturn(void.class);
}
```

### 4.7 资源

最后，让我们使用ResourcesScanner在类路径中查找给定的文件名：

```java
public Set<String> getPomXmlPaths() {
    Reflections reflections = new Reflections(new ResourcesScanner());
    return reflections.getResources(Pattern.compile(".*pom\\.xml"));
}
```

### 4.8 额外查询方法

以上只是展示如何使用Reflections查询方法的几个示例，不过，还有其他查询方法我们在这里没有介绍：

- getMethodsWithAnyParamAnnotated
- getConstructorsMatchParams
- getConstructorsWithAnyParamAnnotated
- getFieldsAnnotatedWith
- getMethodParamNames
- getConstructorParamNames
- getFieldUsage
- getMethodUsage
- getConstructorUsage

## 5. 将Reflections集成到构建生命周期中

我们可以使用[gmavenplus-plugin](https://mvnrepository.com/artifact/org.codehaus.gmavenplus/gmavenplus-plugin)轻松地将Reflections集成到我们的Maven构建中。

让我们对其进行配置以将扫描结果保存到文件中：

```xml
<plugin>
    <groupId>org.codehaus.gmavenplus</groupId>
    <artifactId>gmavenplus-plugin</artifactId>
    <version>1.5</version>
    <executions>
        <execution>
            <phase>generate-resources</phase>
            <goals>
                <goal>execute</goal>
            </goals>
            <configuration>
                <scripts>
                    <script><![CDATA[
                        new org.reflections.Reflections(
                          "cn.tuyucheng.taketoday.refelections")
                            .save("${outputDirectory}/META-INF/reflections/reflections.xml")]]>
                    </script>
                </scripts>
            </configuration>
        </execution>
    </executions>
</plugin>
```

稍后，**通过调用collect()方法，我们可以检索保存的结果并使其可供进一步使用，而无需执行新的扫描**：

```java
Reflections reflections = isProduction() ? Reflections.collect() : new Reflections("cn.tuyucheng.taketoday.reflections");
```

## 6. 总结

在本文中，我们探索了Reflections库。我们介绍了不同的配置元素及其用法，最后，我们了解了如何将Reflections集成到Maven项目的构建生命周期中。