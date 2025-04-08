---
layout: post
title:  MapStruct快速指南
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

在本教程中，我们将探索[MapStruct](http://www.mapstruct.org/)的使用，简单来说，它是一个Java Bean映射器。

此API包含自动在两个Java Bean之间进行映射的函数，使用MapStruct，我们只需要创建接口，库将在编译时自动创建具体实现。

## 2. MapStruct和传输对象模式

对于大多数应用程序，你会注意到许多将POJO转换为其他POJO的样板代码。

例如，一种常见的转换类型发生在持久层支持的实体和发往客户端的DTO之间。

所以，这就是MapStruct解决的问题：手动创建Bean映射器非常耗时，但**该库可以自动生成Bean映射器类**。

## 3. Maven

让我们将以下依赖添加到Maven pom.xml中：

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0.Beta1</version> 
</dependency>
```

[MapStruct](https://mvnrepository.com/artifact/org.mapstruct/mapstruct)及其[处理器](https://mvnrepository.com/artifact/org.mapstruct/mapstruct-processor)的最新稳定版本均可从Maven中央仓库获得。

我们还将annotationProcessorPaths部分添加到maven-compiler-plugin插件的配置部分。

mapstruct-processor用于在构建过程中生成映射器实现：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.6.0.Beta1</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

## 4. 基本映射

### 4.1 创建POJO

我们首先创建一个简单的Java POJO：

```java
public class SimpleSource {
    private String name;
    private String description;
    // getters and setters
}
 
public class SimpleDestination {
    private String name;
    private String description;
    // getters and setters
}
```

### 4.2 Mapper接口

```java
@Mapper
public interface SimpleSourceDestinationMapper {
    SimpleDestination sourceToDestination(SimpleSource source);
    SimpleSource destinationToSource(SimpleDestination destination);
}
```

请注意，我们没有为SimpleSourceDestinationMapper创建实现类-因为MapStruct会为我们创建它。

### 4.3 新的映射器

我们可以通过执行mvn clean install来触发MapStruct处理。

这将在/target/generated-sources/annotations/下生成实现类。

以下是MapStruct为我们自动创建的类：

```java
public class SimpleSourceDestinationMapperImpl implements SimpleSourceDestinationMapper {
    @Override
    public SimpleDestination sourceToDestination(SimpleSource source) {
        if ( source == null ) {
            return null;
        }
        SimpleDestination simpleDestination = new SimpleDestination();
        simpleDestination.setName( source.getName() );
        simpleDestination.setDescription( source.getDescription() );
        return simpleDestination;
    }
    @Override
    public SimpleSource destinationToSource(SimpleDestination destination){
        if ( destination == null ) {
            return null;
        }
        SimpleSource simpleSource = new SimpleSource();
        simpleSource.setName( destination.getName() );
        simpleSource.setDescription( destination.getDescription() );
        return simpleSource;
    }
}
```

### 4.4 测试用例

最后，所有内容都生成后，让我们编写一个测试用例，显示SimpleSource中的值与SimpleDestination中的值相匹配：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class SimpleSourceDestinationMapperIntegrationTest {

    @Autowired
    SimpleSourceDestinationMapper simpleSourceDestinationMapper;

    @Test
    public void givenSourceToDestination_whenMaps_thenCorrect() {
        SimpleSource simpleSource = new SimpleSource();
        simpleSource.setName("SourceName");
        simpleSource.setDescription("SourceDescription");

        SimpleDestination destination = simpleSourceDestinationMapper.sourceToDestination(simpleSource);

        assertEquals(simpleSource.getName(), destination.getName());
        assertEquals(simpleSource.getDescription(), destination.getDescription());
    }

    @Test
    public void givenDestinationToSource_whenMaps_thenCorrect() {
        SimpleDestination destination = new SimpleDestination();
        destination.setName("DestinationName");
        destination.setDescription("DestinationDescription");

        SimpleSource source = simpleSourceDestinationMapper.destinationToSource(destination);

        assertEquals(destination.getName(), source.getName());
        assertEquals(destination.getDescription(), source.getDescription());
    }
}
```

## 5. 使用依赖注入进行映射

接下来，我们通过仅调用Mappers.getMapper(YourClass.class)来获取MapStruct中映射器的实例。

当然，这是获取实例的一种非常手动的方式。但是，更好的选择是将映射器直接注入我们需要的地方(如果我们的项目使用任何依赖注入解决方案)。

**幸运的是，MapStruct对Spring和CDI(上下文和依赖注入)提供了可靠的支持**。

为了在我们的映射器中使用Spring IoC，我们需要将componentModel属性添加到@Mapper，其值为spring，对于CDI，该值为cdi。

### 5.1 修改Mapper

将以下代码添加到SimpleSourceDestinationMapper：

```java
@Mapper(componentModel = "spring")
public interface SimpleSourceDestinationMapper
```

### 5.2 将Spring组件注入Mapper

有时，我们需要在映射逻辑中使用其他Spring组件。在这种情况下，**我们必须使用抽象类而不是接口**：

```java
@Mapper(componentModel = "spring")
public abstract class SimpleDestinationMapperUsingInjectedService
```

然后，我们可以使用众所周知的@Autowired注解轻松地注入所需的组件并在我们的代码中使用它：

```java
@Mapper(componentModel = "spring")
public abstract class SimpleDestinationMapperUsingInjectedService {

    @Autowired
    protected SimpleService simpleService;

    @Mapping(target = "name", expression = "java(simpleService.enrichName(source.getName()))")
    public abstract SimpleDestination sourceToDestination(SimpleSource source);
}
```

我们必须记住**不要将注入的Bean设为私有**，这是因为MapStruct必须访问生成的实现类中的对象。

## 6. 映射具有不同字段名称的字段

从我们前面的示例来看，MapStruct能够自动映射我们的Bean，因为它们具有相同的字段名称。那么，如果我们要映射的Bean具有不同的字段名称怎么办？

在这个例子中，我们将创建一个名为Employee和EmployeeDTO的新Bean。

### 6.1 新的POJO

```java
public class EmployeeDTO {

    private int employeeId;
    private String employeeName;
    // getters and setters
}
```

```java
public class Employee {

    private int id;
    private String name;
    // getters and setters
}
```

### 6.2 映射器接口

当映射不同的字段名称时，我们需要将其源字段配置为其目标字段，为此，我们需要为每个字段添加@Mapping注解。

在MapStruct中，我们还可以使用“.”符号来定义Bean的成员：

```java
@Mapper
public interface EmployeeMapper {

    @Mapping(target = "employeeId", source = "entity.id")
    @Mapping(target = "employeeName", source = "entity.name")
    EmployeeDTO employeeToEmployeeDTO(Employee entity);

    @Mapping(target = "id", source = "dto.employeeId")
    @Mapping(target = "name", source = "dto.employeeName")
    Employee employeeDTOtoEmployee(EmployeeDTO dto);
}
```

### 6.3 测试用例

再次，我们需要测试源对象和目标对象的值是否匹配：

```java
@Test
public void givenEmployeeDTOwithDiffNametoEmployee_whenMaps_thenCorrect() {
    EmployeeDTO dto = new EmployeeDTO();
    dto.setEmployeeId(1);
    dto.setEmployeeName("John");

    Employee entity = mapper.employeeDTOtoEmployee(dto);

    assertEquals(dto.getEmployeeId(), entity.getId());
    assertEquals(dto.getEmployeeName(), entity.getName());
}
```

更多测试用例可以在[GitHub项目](https://github.com/eugenp/tutorials/tree/master/mapstruct)中找到。

## 7. 将Bean与子Bean进行映射

接下来，我们将展示如何将一个Bean映射到其他Bean的引用。

### 7.1 修改POJO

让我们向Employee对象添加一个新的Bean引用：

```java
public class EmployeeDTO {
    private int employeeId;
    private String employeeName;
    private DivisionDTO division;
    // getters and setters omitted
}
```

```java
public class Employee {
    private int id;
    private String name;
    private Division division;
    // getters and setters omitted
}
```

```java
public class Division {
    private int id;
    private String name;
    // default constructor, getters and setters omitted
}
```

### 7.2 修改Mapper

这里我们需要添加一个方法将Division转换为DivisionDTO；如果MapStruct检测到需要转换的对象类型，并且转换的方法存在于同一个类中，它就会自动使用它。

让我们将其添加到映射器中：

```java
DivisionDTO divisionToDivisionDTO(Division entity);

Division divisionDTOtoDivision(DivisionDTO dto);
```

### 7.3 修改测试用例

让我们修改并添加一些现有的测试用例：

```java
@Test
public void givenEmpDTONestedMappingToEmp_whenMaps_thenCorrect() {
    EmployeeDTO dto = new EmployeeDTO();
    dto.setDivision(new DivisionDTO(1, "Division1"));
    Employee entity = mapper.employeeDTOtoEmployee(dto);
    assertEquals(dto.getDivision().getId(), entity.getDivision().getId());
    assertEquals(dto.getDivision().getName(), entity.getDivision().getName());
}
```

## 8. 类型转换映射

MapStruct还提供了一些现成的隐式类型转换，对于我们的示例，我们将尝试将字符串日期转换为实际的Date对象。

有关隐式类型转换的更多详细信息，请查看[MapStruct参考指南](https://mapstruct.org/documentation/stable/reference/html/)。

### 8.1 修改Bean

我们为Employee添加开始日期：

```java
public class Employee {
    // other fields
    private Date startDt;
    // getters and setters
}
```

```java
public class EmployeeDTO {
    // other fields
    private String employeeStartDt;
    // getters and setters
}
```

### 8.2 修改Mapper

我们修改映射器并提供开始日期的dateFormat：

```java
@Mapping(target="employeeId", source = "entity.id")
@Mapping(target="employeeName", source = "entity.name")
@Mapping(target="employeeStartDt", source = "entity.startDt",
         dateFormat = "dd-MM-yyyy HH:mm:ss")
EmployeeDTO employeeToEmployeeDTO(Employee entity);

@Mapping(target="id", source="dto.employeeId")
@Mapping(target="name", source="dto.employeeName")
@Mapping(target="startDt", source="dto.employeeStartDt",
         dateFormat="dd-MM-yyyy HH:mm:ss")
Employee employeeDTOtoEmployee(EmployeeDTO dto);
```

### 8.3 修改测试用例

让我们添加一些测试用例来验证转换是否正确：

```java
private static final String DATE_FORMAT = "dd-MM-yyyy HH:mm:ss";

@Test
public void givenEmpStartDtMappingToEmpDTO_whenMaps_thenCorrect() throws ParseException {
    Employee entity = new Employee();
    entity.setStartDt(new Date());
    EmployeeDTO dto = mapper.employeeToEmployeeDTO(entity);
    SimpleDateFormat format = new SimpleDateFormat(DATE_FORMAT);
 
    assertEquals(format.parse(dto.getEmployeeStartDt()).toString(), entity.getStartDt().toString());
}

@Test
public void givenEmpDTOStartDtMappingToEmp_whenMaps_thenCorrect() throws ParseException {
    EmployeeDTO dto = new EmployeeDTO();
    dto.setEmployeeStartDt("01-04-2016 01:00:00");
    Employee entity = mapper.employeeDTOtoEmployee(dto);
    SimpleDateFormat format = new SimpleDateFormat(DATE_FORMAT);
 
    assertEquals(format.parse(dto.getEmployeeStartDt()).toString(), entity.getStartDt().toString());
}
```

## 9. 使用抽象类进行映射

有时，我们可能希望以超出@Mapping功能的方式定制我们的映射器。

例如，除了类型转换之外，我们可能还想以某种方式转换值，如下面的示例所示。

在这种情况下，我们可以创建一个抽象类并实现我们想要定制的方法，而将那些应该由MapStruct生成的方法保留为抽象。

### 9.1 基本模型

在此示例中，我们将使用以下类：

```java
public class Transaction {
    private Long id;
    private String uuid = UUID.randomUUID().toString();
    private BigDecimal total;

    //standard getters
}
```

以及匹配的DTO：

```java
public class TransactionDTO {

    private String uuid;
    private Long totalInCents;

    // standard getters and setters
}
```

这里棘手的部分是将BigDecimal total转换为Long totalInCents。

### 9.2 定义映射器

我们可以通过将Mapper创建为抽象类来实现这一点：

```java
@Mapper
abstract class TransactionMapper {

    public TransactionDTO toTransactionDTO(Transaction transaction) {
        TransactionDTO transactionDTO = new TransactionDTO();
        transactionDTO.setUuid(transaction.getUuid());
        transactionDTO.setTotalInCents(transaction.getTotal()
            .multiply(new BigDecimal("100")).longValue());
        return transactionDTO;
    }

    public abstract List<TransactionDTO> toTransactionDTO(
      Collection<Transaction> transactions);
}
```

在这里，我们为单个对象转换实现了完全定制的映射方法。

另一方面，我们留下了将Collection映射到List抽象的方法，因此MapStruct将为我们实现它。

### 9.3 生成结果

由于我们已经实现了将单个Transaction映射到TransactionDTO的方法，我们希望MapStruct在第二种方法中使用它。

将生成以下内容：

```java
@Generated
class TransactionMapperImpl extends TransactionMapper {

    @Override
    public List<TransactionDTO> toTransactionDTO(Collection<Transaction> transactions) {
        if ( transactions == null ) {
            return null;
        }

        List<TransactionDTO> list = new ArrayList<>();
        for ( Transaction transaction : transactions ) {
            list.add( toTransactionDTO( transaction ) );
        }

        return list;
    }
}
```

正如我们在第12行看到的，MapStruct在生成的方法中使用了我们的实现。

## 10. 映射前和映射后注解

这是使用@BeforeMapping和@AfterMapping注解自定义@Mapping功能的另一种方法，**这些注解用于标记在映射逻辑之前和之后调用的方法**。

在我们希望将此行为应用于所有映射超类型的场景中，它们非常有用。

让我们看一个将Car ElectricCar和BioDieselCar的子类型映射到CarDTO的示例。

在映射时，我们希望将类型概念映射到DTO中的FuelType枚举字段。映射完成后，我们希望将DTO的名称更改为大写。

### 10.1 基本模型

我们将使用以下类：

```java
public class Car {
    private int id;
    private String name;
}
```

Car子类型：

```java
public class BioDieselCar extends Car {
}
public class ElectricCar extends Car {
}
```

具有枚举字段类型FuelType的CarDTO：

```java
public class CarDTO {
    private int id;
    private String name;
    private FuelType fuelType;
}
```

```java
public enum FuelType {
    ELECTRIC, BIO_DIESEL
}
```

### 10.2 定义映射器

现在让我们继续编写将Car映射到CarDTO的抽象映射器类：

```java
@Mapper
public abstract class CarsMapper {
    @BeforeMapping
    protected void enrichDTOWithFuelType(Car car, @MappingTarget CarDTO carDto) {
        if (car instanceof ElectricCar) {
            carDto.setFuelType(FuelType.ELECTRIC);
        }
        if (car instanceof BioDieselCar) {
            carDto.setFuelType(FuelType.BIO_DIESEL);
        }
    }

    @AfterMapping
    protected void convertNameToUpperCase(@MappingTarget CarDTO carDto) {
        carDto.setName(carDto.getName().toUpperCase());
    }

    public abstract CarDTO toCarDto(Car car);
}
```

**@MappingTarget是一个参数注解，如果使用@BeforeMapping标注方法，则它在执行映射逻辑之前填充目标映射DTO；如果使用@AfterMapping标注方法，则在执行映射逻辑之后填充目标映射DTO**。

### 10.3 结果

上面定义的CarsMapper生成实现：

```java
@Generated
public class CarsMapperImpl extends CarsMapper {

    @Override
    public CarDTO toCarDto(Car car) {
        if (car == null) {
            return null;
        }

        CarDTO carDTO = new CarDTO();

        enrichDTOWithFuelType(car, carDTO);

        carDTO.setId(car.getId());
        carDTO.setName(car.getName());

        convertNameToUpperCase(carDTO);

        return carDTO;
    }
}
```

请注意**标注的方法调用如何围绕实现中的映射逻辑**。

## 11. 支持Lombok

在最新版本的MapStruct中，正式支持Lombok。**因此，我们可以使用Lombok轻松映射源实体和目标**。

要启用Lombok支持，我们需要在注解处理器路径中添加[依赖](https://mvnrepository.com/artifact/org.projectlombok/lombok)。从Lombok版本1.18.16开始，我们还必须添加对lombok-mapstruct-binding的依赖，现在我们在Maven编译器插件中有了mapstruct-processor以及Lombok：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.5.5.Final</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
	        <version>1.18.30</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok-mapstruct-binding</artifactId>
	        <version>0.2.0</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

让我们使用Lombok注解定义源实体：

```java
@Getter
@Setter
public class Car {
    private int id;
    private String name;
}
```

以及目标数据传输对象：

```java
@Getter
@Setter
public class CarDTO {
    private int id;
    private String name;
}
```

这个映射器接口与我们之前的示例类似：

```java
@Mapper
public interface CarMapper {
    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);
    CarDTO carToCarDTO(Car car);
}
```

## 12. 支持defaultExpression

从1.3.0版本开始，**我们可以使用@Mapping注解的defaultExpression属性来指定一个表达式，如果源字段为null，则该表达式确定目标字段的值**；这是对现有defaultValue属性功能的补充。

源实体：

```java
public class Person {
    private int id;
    private String name;
}
```

目标数据传输对象：

```java
public class PersonDTO {
    private int id;
    private String name;
}
```

如果源实体的id字段为null，我们希望生成一个随机id并将其分配给目标，同时保持其他属性值不变：

```java
@Mapper
public interface PersonMapper {
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);
    
    @Mapping(target = "id", source = "person.id", defaultExpression = "java(java.util.UUID.randomUUID().toString())")
    PersonDTO personToPersonDTO(Person person);
}
```

我们添加一个测试用例来验证表达式的执行：

```java
@Test
public void givenPersonEntitytoPersonWithExpression_whenMaps_thenCorrect() 
    Person entity  = new Person();
    entity.setName("Micheal");
    PersonDTO personDto = PersonMapper.INSTANCE.personToPersonDTO(entity);
    assertNull(entity.getId());
    assertNotNull(personDto.getId());
    assertEquals(personDto.getName(), entity.getName());
}
```

## 13. 使用MapStruct指定默认值

如果相应的源字段为空，MapStruct允许我们为目标字段指定默认值。此功能在映射实体时非常有用，可确保即使缺少某些源属性，生成的对象也始终具有有意义的值。

让我们考虑一个涉及Person实体及其对应的PersonDTO数据传输对象的示例，如果未提供Person中的name字段，我们可能希望在PersonDTO中分配一个默认值；这可以使用@Mapping注解的defaultValue属性来实现：

```java
public interface PersonMapper {
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);
    
    @Mapping(target = "id", source = "person.id", defaultExpression = "java(java.util.UUID.randomUUID().toString())")
    @Mapping(target = "name", source = "person.name", defaultValue = "anonymous")
    PersonDTO personToPersonDTO(Person person);
}
```

在这个映射配置中，如果Person实体中的名称为null，则PersonDTO中将默认使用值“anonymous”。

现在，让我们看一个单元测试来验证这个行为：

```java
@Test
public void givenPersonEntityWithNullName_whenMaps_thenCorrect() {
    Person entity = new Person();
    entity.setId("1");
    PersonDTO personDto = PersonMapper.INSTANCE.personToPersonDTO(entity);
    assertEquals("anonymous", personDto.getName());
}
```

在这个测试中，由于entity.getName()为null，映射器将personDto.getName()返回的name赋值为“anonymous”。**这证实了当源字段不存在时，默认值可以有效应用**。

## 14. 总结

本文介绍了MapStruct，我们介绍了Mapping库的大部分基础知识以及如何在我们的应用程序中使用它。