---
layout: post
title: 在MapStruct中使用多个源对象
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

在本教程中，我们将介绍如何在[MapStruct](https://www.baeldung.com/mapstruct)中使用多个源对象。

## 2. 单一源对象

MapStruct最常见的用例是将一个对象映射到另一个对象，假设我们有一个Customer类：

```java
class Customer {

    private String firstName;
    private String lastName;

    // getters and setters
}
```

并有一个相应的CustomerDto：

```java
class CustomerDto {

    private String forename;
    private String surname;

    // getters and setters
}
```

我们现在可以定义一个映射器，将Customer对象映射到CustomerDto对象：

```java
@Mapper
public interface CustomerDtoMapper {

    @Mapping(source = "firstName", target = "forename")
    @Mapping(source = "lastName", target = "surname")
    CustomerDto from(Customer customer);
}
```

## 3. 多个源对象

**有时我们希望目标对象具有来自多个源对象的属性**，假设我们编写了一个购物应用程序。

我们需要构造一个送货地址来运送我们的货物：

```java
class DeliveryAddress {

    private String forename;
    private String surname;
    private String street;
    private String postalcode;
    private String county;

    // getters and setters
}
```

每个客户可以有多个地址；一个可以是家庭地址，另一个可以是工作地址：

```java
class Address {

    private String street;
    private String postalcode;
    private String county;

    // getters and setters
}
```

我们现在需要一个映射器，根据客户及其地址之一创建送货地址。MapStruct通过多个源对象来支持此功能：

```java
@Mapper
interface DeliveryAddressMapper {

    @Mapping(source = "customer.firstName", target = "forename")
    @Mapping(source = "customer.lastName", target = "surname")
    @Mapping(source = "address.street", target = "street")
    @Mapping(source = "address.postalcode", target = "postalcode")
    @Mapping(source = "address.county", target = "county")
    DeliveryAddress from(Customer customer, Address address);
}
```

让我们通过编写一个小测试来看一下实际效果：

```java
// given a customer
Customer customer = new Customer().setFirstName("Max")
    .setLastName("Powers");

// and some address
Address homeAddress = new Address().setStreet("123 Some Street")
    .setCounty("Nevada")
    .setPostalcode("89123");

// when calling DeliveryAddressMapper::from
DeliveryAddress deliveryAddress = deliveryAddressMapper.from(customer, homeAddress);

// then a new DeliveryAddress is created, based on the given customer and his home address
assertEquals(deliveryAddress.getForename(), customer.getFirstName());
assertEquals(deliveryAddress.getSurname(), customer.getLastName());
assertEquals(deliveryAddress.getStreet(), homeAddress.getStreet());
assertEquals(deliveryAddress.getCounty(), homeAddress.getCounty());
assertEquals(deliveryAddress.getPostalcode(), homeAddress.getPostalcode());
```

**当我们有多个参数时，我们可以在@Mapping注解中使用“.”符号来处理它们**。例如，要处理名为customer的参数的firstName属性，我们只需写入“customer.firstName”。

但是，我们并不局限于两个源对象，任意数量都可以。

## 4. 使用@MappingTarget更新现有对象

到目前为止，我们拥有创建目标类的新实例的映射器。有了多个源对象，我们现在还可以提供要更新的实例。

例如，假设我们要更新送货地址的客户相关属性，我们只需要让其中一个参数与方法返回的类型相同，并使用@MappingTarget对其进行标注：

```java
@Mapper
interface DeliveryAddressMapper {

    @Mapping(source = "address.postalcode", target = "postalcode")
    @Mapping(source = "address.county", target = "county")
    DeliveryAddress updateAddress(@MappingTarget DeliveryAddress deliveryAddress, Address address);
}
```

那么，让我们继续使用DeliveryAddress实例进行快速测试：

```java
// given a delivery address
DeliveryAddress deliveryAddress = new DeliveryAddress().setForename("Max")
    .setSurname("Powers")
    .setStreet("123 Some Street")
    .setCounty("Nevada")
    .setPostalcode("89123");

// and some new address
Address newAddress = new Address().setStreet("456 Some other street")
    .setCounty("Arizona")
    .setPostalcode("12345");

// when calling DeliveryAddressMapper::updateAddress
DeliveryAddress updatedDeliveryAddress = deliveryAddressMapper.updateAddress(deliveryAddress, newAddress);

// then the *existing* delivery address is updated
assertSame(deliveryAddress, updatedDeliveryAddress);

assertEquals(deliveryAddress.getStreet(), newAddress.getStreet());
assertEquals(deliveryAddress.getCounty(), newAddress.getCounty());
assertEquals(deliveryAddress.getPostalcode(), newAddress.getPostalcode());
```

## 5. 总结

MapStruct允许我们将多个源参数传递给映射方法，例如，当我们想要将多个实体合并为一个时，这非常方便。

另一个用例是让目标对象本身成为源参数之一，使用@MappingTarget注解可以就地更新给定的对象。