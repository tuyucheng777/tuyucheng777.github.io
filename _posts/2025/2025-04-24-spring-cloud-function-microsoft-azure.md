---
layout: post
title:  Spring Cloud Azure Function
category: springcloud
copyright: springcloud
excerpt: Spring Cloud Azure
---

## 1. 概述

在本教程中，我们将学习如何使用[Spring Cloud Function(SCF)](https://docs.spring.io/spring-cloud-function/reference/spring-cloud-function/programming-model.html)框架开发可部署在[Microsoft Azure Functions](https://azure.microsoft.com/en-in/products/functions)中的Java应用程序。

我们将讨论它的关键概念，开发一个示例应用程序，将其部署在Azure Functions服务上，并最终对其进行测试。

## 2. 关键概念

**Azure Functions服务提供了一个[Serverless](https://www.baeldung.com/cs/serverless-architecture)环境，我们可以在其中部署应用程序而无需担心基础架构管理**。我们可以遵循为相应SDK库定义的框架，使用不同的编程语言(如Java、Python、C#等)编写应用程序，这些应用程序可以通过源自Azure服务(如Blob存储、表存储、Cosmos DB数据库、事件桥等)的各种事件来调用。最终，应用程序可以处理事件数据并将其发送到目标系统。

[Java Azure Function](https://www.baeldung.com/java-azure-functions)库提供了基于注解的强大编程模型，它有助于将方法注册到事件，从源系统接收数据，然后更新目标系统。

**SCF框架为Azure Functions和其他Serverless云原生服务(如[AWS Lambda](https://www.baeldung.com/spring-cloud-function)、[Google Cloud Functions](https://cloud.google.com/functions?hl=en)和[Apache OpenWhisk](https://openwhisk.apache.org/))编写的底层程序提供了抽象，这一切都归功于[SCF Azure适配器](https://docs.spring.io/spring-cloud-function/docs/current/reference/html/azure.html)**：

## [![scf azure 适配器](https://www.baeldung.com/wp-content/uploads/2024/08/scf-azure-adapter.svg)](https://www.baeldung.com/wp-content/uploads/2024/08/scf-azure-adapter.svg)

由于其统一的编程模型，它有助于跨不同平台移植相同的代码。此外，我们可以轻松地将Spring框架的[依赖注入](https://www.baeldung.com/spring-dependency-injection)等主要功能引入到Serverless应用程序中。

通常，我们实现核心[函数接口](https://www.baeldung.com/java-8-functional-interfaces)，例如Function<I, O\>、Consumer<I\>和Supplier<O\>，并将它们注册为Spring Bean。然后，此Bean会自动装配到事件处理程序类中，并在该类中使用@FunctionName注解来应用端点方法。

此外，SCF还提供了一个FunctionCatlog Bean，可以自动装配到事件处理程序类中。我们可以使用FunctionCatlaog#lookup("<<Bean name\>\>")方法检索已实现的函数接口。**FunctionCatalog类将其包装在[SimpleFunctionRegistry.FunctionInvocationWrapper](https://javadoc.io/doc/org.springframework.cloud/spring-cloud-function-context/4.0.1/org/springframework/cloud/function/context/catalog/SimpleFunctionRegistry.FunctionInvocationWrapper.html)类中，该类提供了其他功能，例如函数组合和路由**。

我们将在下一节中了解更多信息。

## 3. 先决条件

首先，我们需要一个有效的[Azure订阅](https://azure.microsoft.com/en-in/get-started/azure-portal)来部署Azure函数应用程序。

Java应用程序的端点必须遵循Azure函数的编程模型，因此我们必须对其使用[Maven依赖](https://mvnrepository.com/artifact/com.microsoft.azure.functions/azure-functions-java-library/)：
```xml
<dependency>
    <groupId>com.microsoft.azure.functions</groupId>
    <artifactId>azure-functions-java-library</artifactId>
    <version>3.1.0</version>
</dependency>
```

应用程序代码准备就绪后，我们将需要[Azure Functions Maven插件](https://repo1.maven.org/maven2/com/microsoft/azure/azure-functions-maven-plugin/)将其部署到Azure中：
```xml
<plugin>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-functions-maven-plugin</artifactId>
    <version>1.24.0</version>
</plugin>
```

Maven工具可帮助将应用程序打包到部署到Azure Functions服务所规定的标准结构中，与往常一样，该插件可帮助指定Azure Function的[部署配置](https://github.com/Microsoft/azure-maven-plugins/wiki/Azure-Functions:-Configuration-Details)，如appname、resourcegroup、appServicePlanName等。

现在，让我们定义SCF库[Maven依赖](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-function-adapter-azure/)：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-function-adapter-azure</artifactId>
    <version>4.1.3</version>
</dependency>
```

**该库在用Java编写的Azure函数处理程序中启用了SCF和Spring依赖注入功能**，处理程序指的是我们应用[@FunctionName](https://learn.microsoft.com/en-us/java/api/com.microsoft.azure.functions.annotation.functionname?view=azure-java-stable)标注的Java方法，也是处理来自Azure服务(如Blob存储、Cosmos DB事件桥等)或自定义应用程序的任何事件的入口点。

应用程序jar的[Manifest文件](https://www.baeldung.com/java-jar-manifest)必须将入口点指向使用[@SpringBootApplication](https://www.baeldung.com/spring-boot-annotations)标注的[Spring Boot](https://www.baeldung.com/spring-boot)类，我们可以在[maven-jar-plugin](https://maven.apache.org/plugins/maven-jar-plugin/usage.html)的帮助下明确设置它：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.4.2</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>cn.tuyucheng.taketoday.functions.AzureSpringCloudFunctionApplication</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

另一种方法是在pom.xml文件中设置start-class属性值，但这只有当我们将[spring-boot-starter-parent](https://www.baeldung.com/spring-boot-starter-parent)定义为父级时才有效：
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.11</version>
    <relativePath/>
</parent>
```

最后，我们设置start-class属性：
```xml
<properties>
    <start-class>cn.tuyucheng.taketoday.functions.AzureSpringCloudFunctionApplication</start-class>
</properties>
```

此属性确保调用Spring Boot主类，初始化Spring Bean并允许它们自动装配到事件处理程序类中。

最后，Azure需要对应用程序进行特定类型的打包，因此我们必须禁用默认的Spring Boot打包并启用[spring boot thin layout](https://www.baeldung.com/spring-boot-thin-jar)：
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <dependency>
	    <groupId>org.springframework.boot.experimental</groupId>
	    <artifactId>spring-boot-thin-layout</artifactId>
	</dependency>
    </dependencies>
</plugin>
```

## 4. Java实现

让我们考虑这样一个场景：Azure Function应用程序根据员工居住城市计算员工的津贴，该应用程序通过HTTP接收员工JSON字符串，并通过将津贴添加到工资中将其发送回去。

### 4.1 使用普通Spring Bean实现

首先，我们将定义开发此Azure函数应用程序的主要类：

![](/assets/images/2025/springcloud/springcloudfunctionmicrosoftazure01.png)

让我们首先定义EmployeeSalaryFunction：
```java
public class EmployeeSalaryFunction implements Function<Employee, Employee> {

    @Override
    public Employee apply(Employee employee) {
        int allowance;
        switch (employee.getCity()) {
            case "Chicago" -> allowance = 5000;
            case "California" -> allowance = 2000;
            case "New York" -> allowance = 2500;
            default -> allowance = 1000;
        }
        int finalSalary = employee.getSalary() + allowance;
        employee.setSalary(finalSalary);
        return employee;
    }
}
```

EmployeeSalaryFunction类实现了接口[java.util.function.Function](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/Function.html)，**EmployeeSalaryFunction#apply()方法将基于城市的津贴添加到员工的基本工资中**。

要将此类加载为Spring Bean，我们将在ApplicationConfiguration类中实例化它：
```java
@Configuration
public class ApplicationConfiguration {
    @Bean
    public Function<Employee, Employee> employeeSalaryFunction() {
        return new EmployeeSalaryFunction();
    }
}
```

我们已经将@Configuration注解应用于此类，以便Spring框架知道这是Bean定义的来源，@Bean方法employeeSalaryFunction()创建EmployeeSalaryFunction类型的Spring Bean employeeSalaryFunction。

现在，让我们使用@Autowired注解将这个employeeSalaryFunction Bean注入到EmployeeSalaryHandler类中：
```java
@Component
public class EmployeeSalaryHandler {
    @Autowired
    private Function<Employee, Employee> employeeSalaryFunction;

    @FunctionName("employeeSalaryFunction")
    public HttpResponseMessage calculateSalary(
            @HttpTrigger(
                    name="http",
                    methods = HttpMethod.POST,
                    authLevel = AuthorizationLevel.ANONYMOUS)HttpRequestMessage<Optional<Employee>> employeeHttpRequestMessage,
            ExecutionContext executionContext
    ) {
        Employee employeeRequest = employeeHttpRequestMessage.getBody().get();
        Employee employee = employeeSalaryFunction.apply(employeeRequest);
        return employeeHttpRequestMessage.createResponseBuilder(HttpStatus.OK)
                .body(employee)
                .build();
    }
}
```

Azure事件处理程序函数主要按照Java Azure Function SDK编程模型编写，但是，**它在类级别使用Spring框架的@Component注解，并在employeeSalaryFunction字段上使用@Autowired注解**。按照惯例，确保自动装配的Bean的名称与@FunctionName注解中指定的名称匹配是一种很好的做法。

类似地，我们可以扩展Spring框架对其他Azure函数触发器的支持，例如@BlobTrigger、@QueueTrigger、@TimerTrigger等。

### 4.2 使用SCF实现

**在必须动态检索函数Bean的情况下，显式自动装配所有函数并不是最佳解决方案**。

假设我们有多个实现来根据城市计算员工的最终工资：

![](/assets/images/2025/springcloud/springcloudfunctionmicrosoftazure02.png)

我们定义了NewYorkSalaryCalculatorFn、ChicagoSalaryCalculatorFn和CaliforniaSalaryCalculatorFn等函数，这些函数根据员工居住的城市计算员工的最终工资。

我们来看看CaliforniaSalaryCalculatorFn类：
```java
public class CaliforniaSalaryCalculatorFn implements Function<Employee, Employee> {
    @Override
    public Employee apply(Employee employee) {
        Integer finalSalary = employee.getSalary() + 3000;
        employee.setSalary(finalSalary);
        return employee;
    }
}
```

该方法在员工基本工资的基础上额外增加3000美元的津贴，其他城市员工工资计算函数大致类似。

**入口方法EmployeeSalaryHandler#calculateSalaryWithSCF()使用EmployeeSalaryFunctionWrapper#getCityBasedSalaryFunction()来检索适当的城市特定函数来计算员工的工资**：
```java
public class EmployeeSalaryFunctionWrapper {
    private FunctionCatalog functionCatalog;

    public EmployeeSalaryFunctionWrapper(FunctionCatalog functionCatalog) {
        this.functionCatalog = functionCatalog;
    }

    public Function<Employee, Employee> getCityBasedSalaryFunction(Employee employee) {
        Function<Employee, Employee> salaryCalculatorFunction;
        switch (employee.getCity()) {
            case "Chicago" -> salaryCalculatorFunction = functionCatalog.lookup("chicagoSalaryCalculatorFn");
            case "California" -> salaryCalculatorFunction = functionCatalog.lookup("californiaSalaryCalculatorFn|defaultSalaryCalculatorFn");
            case "New York" -> salaryCalculatorFunction = functionCatalog.lookup("newYorkSalaryCalculatorFn");
            default -> salaryCalculatorFunction = functionCatalog.lookup("defaultSalaryCalculatorFn");
        }
        return salaryCalculatorFunction;
    }
}
```

我们可以通过将FunctionCatalog对象传递给构造函数来实例化EmployeeSalaryFunctionWrapper，然后我们通过调用EmployeeSalaryFunctionWrapper#getCityBasedSalaryFunction()来检索正确的工资计算器函数Bean，FunctionCatalog#lookup(<<Bean name\>\>)方法有助于检索工资计算器函数Bean。

此外，**函数Bean是支持函数组合和路由的SimpleFunctionRegistry$FunctionInvocationWrapper的一个实例**。例如，functionCatalog.lookup(“californiaSalaryCalculatorFn|defaultSalaryCalculatorFn”)将返回一个组合函数，此函数上的apply()方法等效于：
```java
californiaSalaryCalculatorFn.andThen(defaultSalaryCalculatorFn).apply(employee)
```

这意味着来自加州的员工既可获得州政府补贴，又可获得额外的默认补贴。

最后我们来看一下事件处理函数：
```java
@Component
public class EmployeeSalaryHandler {
    @Autowired
    private FunctionCatalog functionCatalog;

    @FunctionName("calculateSalaryWithSCF")
    public HttpResponseMessage calculateSalaryWithSCF(
            @HttpTrigger(
                    name="http",
                    methods = HttpMethod.POST,
                    authLevel = AuthorizationLevel.ANONYMOUS)HttpRequestMessage<Optional<Employee>> employeeHttpRequestMessage,
            ExecutionContext executionContext
    ) {
        Employee employeeRequest = employeeHttpRequestMessage.getBody().get();
        executionContext.getLogger().info("Salary of " + employeeRequest.getName() + " is:" + employeeRequest.getSalary());
        EmployeeSalaryFunctionWrapper employeeSalaryFunctionWrapper = new EmployeeSalaryFunctionWrapper(functionCatalog);
        Function<Employee, Employee> cityBasedSalaryFunction = employeeSalaryFunctionWrapper.getCityBasedSalaryFunction(employeeRequest);
        Employee employee = cityBasedSalaryFunction.apply(employeeRequest);

        executionContext.getLogger().info("Final salary of " + employee.getName() + " is:" + employee.getSalary());
        return employeeHttpRequestMessage.createResponseBuilder(HttpStatus.OK)
                .body(employee)
                .build();
    }
}
```

与上一节讨论的calcuateSalary()方法不同，calculateSalaryWithSCF()使用自动注入到类的FunctionCatalog对象。

## 5. 部署并运行应用程序

**我们将使用Maven在Azure Functions上编译、打包和部署应用程序**，让我们从IntelliJ运行Maven目标：

![](/assets/images/2025/springcloud/springcloudfunctionmicrosoftazure03.png)

成功部署后，以下函数将显示在Azure门户上：

![](/assets/images/2025/springcloud/springcloudfunctionmicrosoftazure04.png)

最后，从Azure门户获取它们的端点后，我们可以调用它们并检查结果：

![](/assets/images/2025/springcloud/springcloudfunctionmicrosoftazure05.png)

此外，可以在Azure门户上确认函数调用：

![](/assets/images/2025/springcloud/springcloudfunctionmicrosoftazure06.png)

## 6. 总结

在本文中，我们学习了如何使用Spring Cloud Function框架开发Java Azure Function应用程序，该框架允许使用基本的Spring依赖注入功能，此外，FunctionCatalog类还提供了与组合和路由等函数相关的功能。

虽然与低级Java Azure Function库相比，该框架可能会增加一些开销，但它提供了显著的设计优势。因此，只有在仔细评估应用程序的性能需求后才应采用它。