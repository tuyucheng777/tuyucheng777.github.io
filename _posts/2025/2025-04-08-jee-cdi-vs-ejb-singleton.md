---
layout: post
title:  CDI和EJB Singleton之间的区别
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在本教程中，我们将仔细研究Jakarta EE中提供的两种[单例](https://www.baeldung.com/java-singleton)类型。我们将解释和演示它们之间的区别，并了解每种类型的适用用法。

首先，在了解细节之前，让我们先了解一下单例是什么。

## 2. 单例设计模式

回想一下，实现[单例模式](https://www.baeldung.com/java-singleton)的常用方法是使用静态实例和私有构造函数：

```java
public final class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

但遗憾的是，这并不是真正面向对象的，而且它存在一些[多线程问题](https://www.baeldung.com/java-singleton#enum)。

**不过，CDI和EJB容器为我们提供了面向对象的替代方案**。

## 3. CDI单例

使用[CDI(上下文和依赖注入)](https://www.baeldung.com/java-ee-cdi)，我们可以使用@Singleton注解轻松创建单例。此注解是javax.inject包的一部分，它指示容器实例化单例一次，并在注入期间将其引用传递给其他对象。

我们可以看到，使用CDI的单例实现非常简单：

```java
@Singleton
public class CarServiceSingleton {
    // ...
}
```

我们的类模拟了一家汽车服务店，我们有很多不同Car的实例，但它们都使用同一家店进行维修。因此，单例很适合。

我们可以使用一个简单的JUnit测试来验证它是否是同一个实例，该测试两次询问该类的上下文。请注意，为了便于阅读，我们在这里提供了一个[getBean](https://github.com/eugenp/tutorials/blob/master/web-modules/jee-7/src/test/java/com/baeldung/singleton/CarServiceLiveTest.java)辅助方法：

```java
@Test
public void givenASingleton_whenGetBeanIsCalledTwice_thenTheSameInstanceIsReturned() {       
    CarServiceSingleton one = getBean(CarServiceSingleton.class);
    CarServiceSingleton two = getBean(CarServiceSingleton.class);
    assertTrue(one == two);
}
```

**由于@Singleton注解，容器两次都会返回相同的引用**。但是，如果我们使用普通托管Bean尝试此操作，容器每次都会提供不同的实例。

虽然这对于javax.inject.Singleton或javax.ejb.Singleton的作用相同，但两者之间存在一个关键区别。

## 4. EJB单例

要创建EJB单例，我们使用javax.ejb包中的@Singleton注解，这样我们就创建了一个[单例会话Bean](https://www.baeldung.com/java-ee-singleton-session-bean)。

我们可以按照上例中测试CDI实现的相同方式来测试此实现，结果将是相同的。正如预期的那样，EJB单例提供了类的单个实例。

但是，**EJB Singleton还以容器管理并发控制的形式提供了附加功能**。

当我们使用这种类型的实现时，EJB容器会确保类的每个公共方法每次都只由一个线程访问。如果多个线程尝试访问同一个方法，则只有一个线程可以使用它，而其他线程则等待轮到它们。

我们可以通过一个简单的测试来验证这个行为，为我们的单例类引入一个服务队列模拟：

```java
private static int serviceQueue;

public int service(Car car) {
    serviceQueue++;
    Thread.sleep(100);
    car.setServiced(true); 
    serviceQueue--;
    return serviceQueue;
}
```

serviceQueue实现为一个普通的静态整数，当汽车“进入”服务时，该整数增加，当汽车“离开”服务时，该整数减少。如果容器提供了适当的锁定，则该变量在服务前后应等于0，在服务期间应等于1。

我们可以通过一个简单的测试来检查这个行为：

```java
@Test
public void whenEjb_thenLockingIsProvided() {
    for (int i = 0; i < 10; i++) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                int serviceQueue = carServiceEjbSingleton.service(new Car("Speedster xyz"));
                assertEquals(0, serviceQueue);
            }
        }).start();
    }
    return;
}
```

此测试启动10个并行线程，每个线程实例化一个Car并尝试为其提供服务。服务结束后，它断言serviceQueue的值回到0。

此外，如果我们对CDI单例执行类似的测试，我们的测试将会失败。

## 5. 总结

在本文中，我们介绍了Jakarta EE中可用的两种单例实现；我们了解了它们的优点和缺点，并演示了如何以及何时使用每种方法。