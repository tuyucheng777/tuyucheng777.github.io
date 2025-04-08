---
layout: post
title:  使用具有继承的MapStruct
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

[MapStruct](https://www.baeldung.com/mapstruct)是一个Java注解处理器，在为Java Bean类生成类型安全且有效的映射器时非常有用。

在本教程中，我们将具体**学习如何将Mapstruct映射器与继承的Java Bean类一起使用**。

我们将讨论三种方法；第一种方法是实例检查，第二种方法是使用[访问者模式](https://www.baeldung.com/java-visitor-pattern)，最后一种也是推荐的方法是使用Mapstruct 1.5.0中引入的@SubclassMapping注解。

## 2. Maven依赖

让我们将以下[mapstruct](https://mvnrepository.com/artifact/org.mapstruct/mapstruct)依赖添加到Maven pom.xml中：

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0.Beta1</version>
</dependency>
```

## 3. 理解问题

默认情况下，Mapstruct不够智能，无法为所有从基类或接口继承的类生成映射器，Mapstruct也不支持在运行时识别对象层次结构下提供的实例。

### 3.1 创建POJO

假设我们的Car和Bus POJO类扩展了Vehicle POJO类：

```java
public abstract class Vehicle {
    private String color;
    private String speed;
    // standard constructors, getters and setters
}
```

```java
public class Car extends Vehicle {
    private Integer tires;
    // standard constructors, getters and setters
}
```

```java
public class Bus extends Vehicle {
    private Integer capacity;
    // standard constructors, getters and setters
}
```

### 3.2 创建DTO

让我们的CarDTO类和BusDTO类扩展VehicleDTO类：

```java
public class VehicleDTO {
    private String color;
    private String speed;
    // standard constructors, getters and setters
}
```

```java
public class CarDTO extends VehicleDTO {
    private Integer tires;
    // standard constructors, getters and setters
}
```

```java
public class BusDTO extends VehicleDTO {
    private Integer capacity;
    // standard constructors, getters and setters
}
```

### 3.3 Mapper接口

让我们使用子类映射器定义基本映射器接口VehicleMapper：

```java
@Mapper(uses = { CarMapper.class, BusMapper.class })
public interface VehicleMapper {
    VehicleDTO vehicleToDTO(Vehicle vehicle);
}
```

```java
@Mapper()
public interface CarMapper {
    CarDTO carToDTO(Car car);
}
```

```java
@Mapper()
public interface BusMapper {
    BusDTO busToDTO(Bus bus);
}
```

在这里，我们分别定义了所有子类映射器，并在生成基映射器时使用了它们。这些子类映射器可以是手写的，也可以是由Mapstruct生成的。在我们的用例中，我们使用Mapstruct生成的子类映射器。

### 3.4 识别问题

当我们在/target/generated-sources/annotations/下生成实现类之后，我们来编写一个测试用例，验证我们的基础映射器VehicleMapper是否可以根据提供的子类POJO实例动态映射到正确的子类DTO。

我们将通过提供一个Car对象并验证生成的DTO实例是否属于CarDTO类型来测试这一点：

```java
@Test
void whenVehicleTypeIsCar_thenBaseMapperNotMappingToSubclass() {
    Car car = getCarInstance();

    VehicleDTO vehicleDTO = vehicleMapper.vehicleToDTO(car);
    Assertions.assertFalse(vehicleDTO instanceof CarDTO);
    Assertions.assertTrue(vehicleDTO instanceof VehicleDTO);

    VehicleDTO carDTO = carMapper.carToDTO(car);
    Assertions.assertTrue(carDTO instanceof CarDTO);
}
```

因此，我们可以看到，基础映射器无法将提供的POJO对象识别为Car的实例。此外，它也无法动态选择相关的子类映射器CarMapper。因此，基础映射器只能映射到VehicleDTO对象，而不管提供的子类实例是什么。

## 4. MapStruct继承与实例检查

第一种方法是指示Mapstruct为每种Vehicle类型生成映射器方法，然后，我们可以通过使用Java [instanceof](https://www.baeldung.com/java-instanceof)运算符进行实例检查，为每个子类调用适当的转换器方法，从而实现基类的通用转换方法：

```java
@Mapper()
public interface VehicleMapperByInstanceChecks {
    CarDTO map(Car car);
    BusDTO map(Bus bus);

    default VehicleDTO mapToVehicleDTO(Vehicle vehicle) {
        if (vehicle instanceof Bus) {
            return map((Bus) vehicle);
        } else if (vehicle instanceof Car) {
            return map((Car) vehicle);
        } else {
            return null;
        }
    }
}
```

成功生成实现类后，我们来用泛型方法验证一下各个子类类型的映射：

```java
@Test
void whenVehicleTypeIsCar_thenMappedToCarDTOCorrectly() {
    Car car = getCarInstance();

    VehicleDTO vehicleDTO = vehicleMapper.mapToVehicleDTO(car);
    Assertions.assertTrue(vehicleDTO instanceof CarDTO);
    Assertions.assertEquals(car.getTires(), ((CarDTO) vehicleDTO).getTires());
    Assertions.assertEquals(car.getSpeed(), vehicleDTO.getSpeed());
    Assertions.assertEquals(car.getColor(), vehicleDTO.getColor());
}

@Test
void whenVehicleTypeIsBus_thenMappedToBusDTOCorrectly() {
    Bus bus = getBusInstance();

    VehicleDTO vehicleDTO = vehicleMapper.mapToVehicleDTO(bus);
    Assertions.assertTrue(vehicleDTO instanceof BusDTO);
    Assertions.assertEquals(bus.getCapacity(), ((BusDTO) vehicleDTO).getCapacity());
    Assertions.assertEquals(bus.getSpeed(), vehicleDTO.getSpeed());
    Assertions.assertEquals(bus.getColor(), vehicleDTO.getColor());
}
```

我们可以使用这种方法来处理任何深度的继承，我们只需要使用实例检查为每个层次结构级别提供一种映射方法。

## 5. MapStruct继承与访问者模式

第二种方法是使用访问者模式；**使用访问者模式方法时，我们可以跳过实例检查，因为Java使用多态性来确定在运行时究竟要调用哪个方法**。

### 5.1 应用访问者模式

 我们首先在抽象类Vehicle中定义抽象方法accept()来接收任何Visitor对象：

```java
public abstract class Vehicle {
    public abstract VehicleDTO accept(Visitor visitor);
}
```

```java
public interface Visitor {
    VehicleDTO visit(Car car);
    VehicleDTO visit(Bus bus);
}
```

现在，我们需要为每种Vehicle类型实现accept()方法：

```java
public class Bus extends Vehicle {
    @Override
    VehicleDTO accept(Visitor visitor) {
        return visitor.visit(this);
    }
}

public class Car extends Vehicle {
    @Override
    VehicleDTO accept(Visitor visitor) {
        return visitor.visit(this);
    }
}
```

最后，我们可以通过实现Visitor接口来实现映射器：

```java
@Mapper()
public abstract class VehicleMapperByVisitorPattern implements Visitor {
    public VehicleDTO mapToVehicleDTO(Vehicle vehicle) {
        return vehicle.accept(this);
    }

    @Override
    public VehicleDTO visit(Car car) {
        return map(car);
    }

    @Override
    public VehicleDTO visit(Bus bus) {
        return map(bus);
    }

    abstract CarDTO map(Car car);
    abstract BusDTO map(Bus bus);
}
```

访问者模式方法比实例检查方法更加优化，因为当深度较高时，不需要检查所有子类，从而节省映射时间。

### 5.2 测试访问者模式

成功生成实现类之后，我们来验证一下各个Vehicle类型与使用Visitor接口实现的映射器的映射情况：

```java
@Test
void whenVehicleTypeIsCar_thenMappedToCarDTOCorrectly() {
    Car car = getCarInstance();

    VehicleDTO vehicleDTO = vehicleMapper.mapToVehicleDTO(car);
    Assertions.assertTrue(vehicleDTO instanceof CarDTO);
    Assertions.assertEquals(car.getTires(), ((CarDTO) vehicleDTO).getTires());
    Assertions.assertEquals(car.getSpeed(), vehicleDTO.getSpeed());
    Assertions.assertEquals(car.getColor(), vehicleDTO.getColor());
}

@Test
void whenVehicleTypeIsBus_thenMappedToBusDTOCorrectly() {
    Bus bus = getBusInstance();

    VehicleDTO vehicleDTO = vehicleMapper.mapToVehicleDTO(bus);
    Assertions.assertTrue(vehicleDTO instanceof BusDTO);
    Assertions.assertEquals(bus.getCapacity(), ((BusDTO) vehicleDTO).getCapacity());
    Assertions.assertEquals(bus.getSpeed(), vehicleDTO.getSpeed());
    Assertions.assertEquals(bus.getColor(), vehicleDTO.getColor());
}
```

## 6. 使用@SubclassMapping实现Mapstruct继承

正如我们前面提到的，Mapstruct 1.5.0引入了[@SubclassMapping](https://mapstruct.org/documentation/stable/api/org/mapstruct/SubclassMappings.html)注解，这使我们能够配置映射以处理源类型的层次结构。**source()函数定义要映射的子类，而target()指定要映射到的子类**：

```java
public @interface SubclassMapping {
    Class<?> source();
    Class<?> target();
    // other methods
}
```

### 6.1 应用注解

让我们应用@SubclassMapping注解来实现Vehicle层次结构中的继承：

```java
@Mapper()
public interface VehicleMapperBySubclassMapping {
    @SubclassMapping(source = Car.class, target = CarDTO.class)
    @SubclassMapping(source = Bus.class, target = BusDTO.class)
    VehicleDTO mapToVehicleDTO(Vehicle vehicle);
}
```

为了了解@SubclassMapping内部是如何工作的，让我们看一下/target/generated-sources/annotations/下生成的实现类：

```java
@Generated
public class VehicleMapperBySubclassMappingImpl implements VehicleMapperBySubclassMapping {
    @Override
    public VehicleDTO mapToVehicleDTO(Vehicle vehicle) {
        if (vehicle == null) {
            return null;
        }

        if (vehicle instanceof Car) {
            return carToCarDTO((Car) vehicle);
        } else if (vehicle instanceof Bus) {
            return busToBusDTO((Bus) vehicle);
        } else {
            VehicleDTO vehicleDTO = new VehicleDTO();

            vehicleDTO.setColor(vehicle.getColor());
            vehicleDTO.setSpeed(vehicle.getSpeed());

            return vehicleDTO;
        }
    }
}
```

根据实现，我们可以注意到，Mapstruct在内部使用实例检查来动态选择子类。因此，我们可以推断，对于层次结构中的每一层，我们都需要定义一个@SubclassMapping。

### 6.2 测试注解

我们现在可以使用通过@SubclassMapping注解实现的Mapstruct映射器来验证每种Vehicle类型的映射：

```java
@Test
void whenVehicleTypeIsCar_thenMappedToCarDTOCorrectly() {
    Car car = getCarInstance();

    VehicleDTO vehicleDTO = vehicleMapper.mapToVehicleDTO(car);
    Assertions.assertTrue(vehicleDTO instanceof CarDTO);
    Assertions.assertEquals(car.getTires(), ((CarDTO) vehicleDTO).getTires());
    Assertions.assertEquals(car.getSpeed(), vehicleDTO.getSpeed());
    Assertions.assertEquals(car.getColor(), vehicleDTO.getColor());
}

@Test
void whenVehicleTypeIsBus_thenMappedToBusDTOCorrectly() {
    Bus bus = getBusInstance();

    VehicleDTO vehicleDTO = vehicleMapper.mapToVehicleDTO(bus);
    Assertions.assertTrue(vehicleDTO instanceof BusDTO);
    Assertions.assertEquals(bus.getCapacity(), ((BusDTO) vehicleDTO).getCapacity());
    Assertions.assertEquals(bus.getSpeed(), vehicleDTO.getSpeed());
    Assertions.assertEquals(bus.getColor(), vehicleDTO.getColor());
}
```

## 7. 总结

在本文中，我们讨论了如何使用继承的对象类编写Mapstruct映射器。

我们讨论的第一种方法使用了instanceof检查，而第二种方法使用了著名的访问者模式；此外，使用Mapstruct功能@SubclassMapping可以使事情变得更简单。