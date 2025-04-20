---
layout: post
title:  构建器模式和继承
category: designpattern
copyright: designpattern
excerpt: 构建器模式
---

## 1. 概述

在本教程中，我们将学习在处理层次继承时实现[构建器设计模式](https://www.baeldung.com/java-fluent-interface-vs-builder-pattern)的挑战。层次继承的一个例子是电动汽车、轿车和车辆之间的继承。

构建器模式是一种[创建型设计模式](https://www.baeldung.com/creational-design-patterns)，它通过方法链，简化了构建具有众多属性的复杂对象的逐步过程。虽然[继承](https://www.baeldung.com/java-inheritance)有助于简化设计，但它也增加了在构建器模式中实现方法链创建对象的复杂性。

此外，我们将借助[Java泛型API](https://www.baeldung.com/java-generics)提出一种有效的实现方法。

## 2. 问题描述

让我们举一个应用构建器模式创建Vehicle、Car和ElectricCar类型对象的例子：

![](/assets/images/2025/designpattern/javabuilderpatterninheritance01.png)

在对象层次结构的顶部，是Vehicle类。**Car类扩展了Vehicle类，而ElectricCar类又扩展了Car类。与这些对象类似，它们的构建器之间也存在层级关系**。

让我们实例化CarBuilder类，用方法链设置它的属性，最后调用build()方法来获取Car对象：

```java
CarBuilder carBuilder = new CarBuilder();
Car car = carBuilder.make("Ford")
    .model("F")
    .fuelType("Petrol")
    .colour("red")
    .build();
```

让我们**尝试改变方法调用的顺序**：

```java
CarBuilder carBuilder = new CarBuilder();
Car car = carBuilder.make("Ford")
    .colour("red")
    .fuelType("Petrol")
    .model("F")
    .build();
```

colour()和fuelType()方法返回VehicleBuilder类，因此，后续调用model()会导致编译错误，因为VehicleBuilder类中不存在该类。这很不方便，也是一个缺点。当我们尝试使用ElectricVehicleBuilder类构建ElectricVehicle对象时，也会出现类似的行为。

## 3. 不使用泛型的解决方案

**这是一个非常简单的实现，其中子构建器类覆盖了层次结构中所有基构建器类的链接方法**。因此，在方法链接期间设置属性值时不会出现编译错误。

让我们首先看一下Vehicle类来理解这一点：

```java
public class Vehicle {

    private String fuelType;
    private String colour;

    // Standard Getter methods..
    public Vehicle(VehicleBuilder builder) {
        this.colour = builder.colour;
        this.fuelType = builder.fuelType;
    }

    public static class VehicleBuilder {

        protected String fuelType;
        protected String colour;

        public VehicleBuilder fuelType(String fuelType) {
            this.fuelType = fuelType;
            return this;
        }

        public VehicleBuilder colour(String colour) {
            this.colour = colour;
            return this;
        }

        public Vehicle build() {
            return new Vehicle(this);
        }
    }
}
```

Vehicle类有两个属性fuelType和colour，它还有一个[内部类](https://www.baeldung.com/java-nested-classes)VehicleBuilder，其中包含一些方法，其名称与Vehicle类中的属性类似。它们返回构建器类，以便支持方法链。

现在，让我们看一下Car类：

```java
public class Car extends Vehicle {

    private String make;
    private String model;

    // Standard Getter methods..

    public Car(CarBuilder builder) {
        super(builder);
        this.make = builder.make;
        this.model = builder.model;
    }

    public static class CarBuilder extends VehicleBuilder {
        protected String make;
        protected String model;

        @Override
        public CarBuilder colour(String colour) {
            this.colour = colour;
            return this;
        }

        @Override
        public CarBuilder fuelType(String fuelType) {
            this.fuelType = fuelType;
            return this;
        }

        public CarBuilder make(String make) {
            this.make = make;
            return this;
        }

        public CarBuilder model(String model) {
            this.model = model;
            return this;
        }

        public Car build() {
            return new Car(this);
        }
    }
}
```

Car类继承自Vehicle类，类似地，CarBuilder类也继承自VehicleBuilder类。此外，CarBuilder类必须重写colour()和fuelType()方法。

现在让我们构建一个Car对象：

```java
@Test
void givenNoGenericImpl_whenBuild_thenReturnObject() {
    Car car = new Car.CarBuilder().colour("red")
        .fuelType("Petrol")
        .make("Ford")
        .model("F")
        .build();
    assertEquals("red", car.getColour());
    assertEquals("Ford", car.getMake());
}
```

在调用build()方法之前，我们可以按任意顺序设置Car的属性。

然而，对于Car的子类，例如ElectricCar，我们必须重写ElectricCarBuilder中CarBuilder和VehicleBuilder的所有方法。因此，**这不是一个非常高效的实现**。

## 4. 使用泛型的解决方案

泛型可以帮助克服前面讨论过的实现过程中面临的挑战。

为此，让我们修改Vehicle类中的内部Builder类：

```java
public class Vehicle {

    private String colour;
    private String fuelType;

    public Vehicle(Builder builder) {
        this.colour = builder.colour;
        this.fuelType = builder.fuelType;
    }

    //Standard getter methods..
    public static class Builder<T extends Builder> {
        protected String colour;
        protected String fuelType;

        T self() {
            return (T) this;
        }

        public T colour(String colour) {
            this.colour = colour;
            return self();
        }

        public T fuelType(String fuelType) {
            this.fuelType = fuelType;
            return self();
        }

        public Vehicle build() {
            return new Vehicle(this);
        }
    }
}
```

值得注意的是，内部Builder类中的fuelType()和colour()方法返回的是泛型类型，这种实现有利于流式的编码风格或方法链，**这是一种名为[“奇异循环模板模式”(CRTP)](https://nuah.livejournal.com/328187.html)的设计模式**。

现在让我们实现Car类：

```java
public class Car extends Vehicle {

    private String make;
    private String model;

    //Standard Getters..
    public Car(Builder builder) {
        super(builder);
        this.make = builder.make;
        this.model = builder.model;
    }

    public static class Builder<T extends Builder<T>> extends Vehicle.Builder<T> {
        protected String make;
        protected String model;

        public T make(String make) {
            this.make = make;
            return self();
        }

        public T model(String model) {
            this.model = model;
            return self();
        }

        @Override
        public Car build() {
            return new Car(this);
        }
    }
}
```

**我们在内部Builder类的签名中应用了CRTP，并使内部类中的方法返回泛型类型以支持方法链**。

类似地，让我们实现Car的子类ElectricCar：

```java
public class ElectricCar extends Car {
    private String batteryType;

    public String getBatteryType() {
        return batteryType;
    }

    public ElectricCar(Builder builder) {
        super(builder);
        this.batteryType = builder.batteryType;
    }

    public static class Builder<T extends Builder<T>> extends Car.Builder<T> {
        protected String batteryType;

        public T batteryType(String batteryType) {
            this.batteryType = batteryType;
            return self();
        }

        @Override
        public ElectricCar build() {
            return new ElectricCar(this);
        }
    }
}
```

除了内部Builder类继承了其父类Builder.Car<T\>之外，其实现几乎相同。相同的技术必须应用于ElectricCar的后续子类，依此类推。

让我们看看实际的实现情况：

```java
@Test
void givenGenericImpl_whenBuild_thenReturnObject() {
    Car.Builder<?> carBuilder = new Car.Builder();
    Car car = carBuilder.colour("red")
            .fuelType("Petrol")
            .make("Ford")
            .model("F")
            .build();

    ElectricCar.Builder<?> ElectricCarBuilder = new ElectricCar.Builder();
    ElectricCar eCar = ElectricCarBuilder.make("Mercedes")
            .colour("White")
            .model("G")
            .fuelType("Electric")
            .batteryType("Lithium")
            .build();

    assertEquals("red", car.getColour());
    assertEquals("Ford", car.getMake());

    assertEquals("Electric", eCar.getFuelType());
    assertEquals("Lithium", eCar.getBatteryType());
}
```

该方法成功构建了Car和ElectricCar类型的对象。

**有趣的是，我们使用了原始[泛型类型?](https://www.baeldung.com/java-generics-vs-extends-object)来声明内部类Car.Builder<?\>和ElectricCar.Builder<?\>**，这是因为我们需要确保诸如carBuilder.colour()和carBuilder.fuelType()之类的方法调用返回的是Car.Builder，而不是其父类Vehicle.Builder。

类似地，调用ElectricCarBuilder.make()和ElectricCarBuilder.model()方法应该返回ElectricCarBuilder类，而不是CarBuilder类。如果没有这些方法，链式调用就无法实现。

## 5. 总结

在本文中，我们讨论了构建器设计模式在处理继承时遇到的挑战。Java泛型和循环模板模式帮助我们实现了解决方案，有了它，我们可以使用方法链来设置构建器类中的属性值，而无需担心方法调用的顺序。