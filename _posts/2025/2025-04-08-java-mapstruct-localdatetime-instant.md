---
layout: post
title:  在MapStruct中将LocalDateTime映射到Instant
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

在Java中处理[日期和时间](https://www.baeldung.com/java-dates-series)时，我们经常会遇到不同的格式，例如[LocalDateTime和Instant](https://www.baeldung.com/java-instant-vs-localdatetime)。LocalDateTime表示没有时区的日期时间，而Instant表示特定的时间点，通常参考纪元(1970年1月1日，00:00:00 UTC)。在许多情况下，我们需要在这两种类型之间进行映射。幸运的是，强大的Java映射框架[MapStruct](http://www.mapstruct.org/)允许我们轻松完成此操作。

在本教程中，我们将学习如何在[MapStruct](https://www.baeldung.com/mapstruct)中将LocalDateTime映射到Instant。

## 2. 理解LocalDateTime和Instant

我们可能需要将LocalDateTime映射到Instant的原因有几个。

LocalDateTime可用于表示在特定本地时间发生的事件，而无需考虑时区，我们通常使用它来在数据库和日志文件中存储时间戳。**对于所有用户都在同一时区运行的应用程序，LocalDateTime是一个不错的选择**。

**Instant非常适合跟踪全球事件、确保时区一致性以及提供与外部系统或API交互的可靠格式**。此外，它还可用于将时间戳存储在需要时区一致性的数据库中。

我们将处理LocalDateTime和Instant值并频繁转换它们。

## 3. 映射场景

假设我们正在实现一个订单处理服务，我们有两种类型的订单-订单和本地订单。Order使用Instant来支持全球订单处理，而本地订单使用LocalDateTime来表示本地时间。

以下是订单模型的实现：

```java
public class Order {
    private Long id;
    private Instant created;
    // other fields
    // getters and setters
}
```

然后，这是本地订单的实现：

```java
public class LocalOrder {
    private Long id;
    private LocalDateTime created;
    // other fields
    // getters and setters
}
```

## 4. 将LocalDateTime映射到Instant

现在让我们学习如何实现映射器将LocalDateTime转换为Instant。

让我们从OrderMapper接口开始：

```java
@Mapper
public interface OrderMapper {
    OrderMapper INSTANCE = Mappers.getMapper(OrderMapper.class);

    ZoneOffset DEFAULT_ZONE = ZoneOffset.UTC;

    @Named("localDateTimeToInstant")
    default Instant localDateTimeToInstant(LocalDateTime localDateTime) {
        return localDateTime.toInstant(DEFAULT_ZONE);
    }

    @Mapping(target = "id", source = "id")
    @Mapping(target = "created", source = "created", qualifiedByName = "localDateTimeToInstant")
    Order toOrder(LocalOrder source);
}
```

此OrderMapper接口是一个MapStruct映射器，它将LocalOrder对象转换为Order对象，同时处理日期时间字段的自定义转换。它声明一个常量DEFAULT_ZONE，其值为ZoneOffset.UTC，代表UTC时区。

用@Named标注的localDateTimeToInstant()方法使用指定的ZoneOffset将LocalDateTime转换为Instant。

toOrder()方法将LocalOrder映射到Order，它使用@Mapping来定义字段的映射方式，id字段直接从source映射到target。**对于created字段，它通过指定eligibleByName = "localDateTimeToInstant"来应用自定义localDateTimeToInstant()方法**，这可确保LocalOrder中的LocalDateTime正确转换为Order中的Instant。

MapStruct使用约定来映射数据类型，映射具有嵌套属性的复杂对象或在某些数据类型之间进行转换可能会引发错误，**MapStruct接口中的默认方法可以定义MapStruct本身不支持的类型之间的显式转换**。这些方法解决了歧义问题并提供了转换的必要说明，这确保了准确可靠的映射。此外，它们对于复杂或嵌套的属性映射也很有用。总之，它们是在MapStruct映射中维护干净、可维护代码的最佳实践。

让我们测试一下映射器：

```java
class OrderMapperUnitTest {
    private OrderMapper mapper = OrderMapper.INSTANCE;

    @Test
    void whenLocalOrderIsMapped_thenGetsOrder() {
        LocalDateTime localDateTime = LocalDateTime.now();
        long sourceEpochSecond = localDateTime.toEpochSecond(OrderMapper.DEFAULT_ZONE);
        LocalOrder localOrder = new LocalOrder();
        localOrder.setCreated(localDateTime);

        Order target = mapper.toOrder(localOrder);

        Assertions.assertNotNull(target);
        long targetEpochSecond = target.getCreated().getEpochSecond();
        Assertions.assertEquals(sourceEpochSecond, targetEpochSecond);
    }
}
```

此测试用例检查OrderMapper是否正确地将LocalOrder转换为Order，重点关注从LocalDateTime到Instant的created字段映射。它创建一个LocalDateTime，计算其纪元秒值，将其设置为新的LocalOrder，将其映射到Order，并检查结果是否不为空。最后，它比较LocalOrder的LocalDateTime和Order的Instant之间的纪元秒值，如果它们匹配，则测试通过。

## 5. 将Instant映射到LocalDateTime

现在我们将看到如何将Instant映射回LocalDateTime：

```java
@Mapper
public interface OrderMapper {
    OrderMapper INSTANCE = Mappers.getMapper(OrderMapper.class);
    ZoneOffset DEFAULT_ZONE = ZoneOffset.UTC;

    @Named("instantToLocalDateTime")
    default LocalDateTime instantToLocalDateTime(Instant instant) {
        return LocalDateTime.ofInstant(instant, DEFAULT_ZONE);
    }
  
    @Mapping(target = "id", source = "id")
    @Mapping(target = "created", source = "created", qualifiedByName = "instantToLocalDateTime")
    LocalOrder toLocalOrder(Order source);
}
```

OrderMapper现在定义了将Order对象转换为LocalOrder对象的映射，它包含一个自定义映射方法instantToLocalDateTime()，该方法使用预定义的ZoneOffset(UTC)将Instant转换为LocalDateTime。

toLocalOrder()中的@Mapping注解表明id字段直接从Order映射到LocalOrder，**然后它对created字段使用自定义方法(qualifiedByName = "instantToLocalDateTime")，并将Instant转换为LocalDateTime**。

让我们验证我们的映射：

```java
@Test
void whenOrderIsMapped_thenGetsLocalOrder() {
    Instant source = Instant.now();
    long sourceEpochSecond = source.getEpochSecond();
    Order order = new Order();
    order.setCreated(source);
    
    LocalOrder target = mapper.toLocalOrder(order);
    
    Assertions.assertNotNull(target);
    long targetEpochSecond = target.getCreated().toEpochSecond(OrderMapper.DEFAULT_ZONE);
    Assertions.assertEquals(sourceEpochSecond, targetEpochSecond);
}
```

此测试验证OrderMapper是否正确将Order对象转换为LocalOrder对象，重点是将Instant映射到LocalDateTime。

该测试使用当前时间戳创建一个Instant对象并计算其纪元秒数，然后创建一个Order对象并将Instant值设置为其created字段。

该测试使用mapper.toLocalOrder()将Order对象映射到LocalOrder，它检查生成的LocalOrder是否不为空，并验证LocalOrder中LocalDateTime的纪元秒是否与Order中Instant的纪元秒匹配，以确保与指定的ZoneOffset正确映射。

## 6. 总结

在本文中，我们学习了如何使用MapStruct将LocalDateTime映射到Instant，我们了解了如何使用@Named创建自定义映射方法来在这些类型之间进行转换，以及正确使用@Mapping和eligibleByName，这种方法可确保Java应用程序中的数据转换顺畅和时区一致性。