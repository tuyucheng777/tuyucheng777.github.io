---
layout: post
title:  Java中的动态代理
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 简介

本文介绍了[Java的动态代理](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html)–这是该语言中可用的主要代理机制之一。

简而言之，代理是通过其自身的设施(通常是真实方法)传递函数调用的前端或包装器-可能会增加一些功能。

动态代理允许一个具有单一方法的类为具有任意数量方法的任意类提供多个方法调用服务，动态代理可以被认为是一种装饰器，但它可以假装是任何接口的实现。在幕后，**它将所有方法调用路由到单个处理程序-invoke()方法**。

虽然动态代理不是一种用于日常编程任务的工具，但它对于框架编写者来说非常有用。它还可用于那些直到运行时才知道具体类实现的情况。

此功能内置于标准JDK中，因此不需要额外的依赖项。

## 2. InvocationHandler

让我们构建一个简单的代理，它实际上不执行任何操作，除了打印请求调用的方法并返回硬编码的数字。

首先，我们需要创建java.lang.reflect.InvocationHandler的子类型：

```java
public class DynamicInvocationHandler implements InvocationHandler {

    private static Logger LOGGER = LoggerFactory.getLogger(DynamicInvocationHandler.class);

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        LOGGER.info("Invoked method: {}", method.getName());

        return 42;
    }
}
```

这里我们定义了一个简单的代理，它记录调用了哪个方法并返回42。

## 3. 创建代理实例

我们刚刚定义的调用处理程序所服务的代理实例是通过对java.lang.reflect.Proxy类的工厂方法调用创建的：

```java
Map proxyInstance = (Map) Proxy.newProxyInstance(
    DynamicProxyTest.class.getClassLoader(), 
    new Class[] { Map.class }, 
    new DynamicInvocationHandler());
```

一旦我们有了代理实例，我们就可以正常调用其接口方法：

```java
proxyInstance.put("hello", "world");
```

正如预期的那样，有关调用put()方法的消息被打印在日志文件中。

## 4. 通过Lambda表达式调用处理程序

由于InvocationHandler是一个函数式接口，因此可以使用Lambda表达式内联定义处理程序：

```java
Map proxyInstance = (Map) Proxy.newProxyInstance(
    DynamicProxyTest.class.getClassLoader(), 
    new Class[] { Map.class }, 
    (proxy, method, methodArgs) -> {
        if (method.getName().equals("get")) {
            return 42;
        } else {
            throw new UnsupportedOperationException("Unsupported method: " + method.getName());
        }
});
```

在这里，我们定义了一个处理程序，它对所有get操作返回42，对其他所有操作抛出UnsupportedOperationException。

它的调用方式完全相同：

```java
(int) proxyInstance.get("hello"); // 42
proxyInstance.put("hello", "world"); // exception
```

## 5. 计时器动态代理示例

让我们研究一下动态代理的一个潜在的真实场景。

假设我们想要记录函数执行的时间，为此，我们首先定义一个能够包装“真实”对象的处理程序，跟踪时间信息和反射调用：

```java
public class TimingDynamicInvocationHandler implements InvocationHandler {

    private static Logger LOGGER = LoggerFactory.getLogger(TimingDynamicInvocationHandler.class);

    private final Map<String, Method> methods = new HashMap<>();

    private Object target;

    public TimingDynamicInvocationHandler(Object target) {
        this.target = target;

        for(Method method: target.getClass().getDeclaredMethods()) {
            this.methods.put(method.getName(), method);
        }
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long start = System.nanoTime();
        Object result = methods.get(method.getName()).invoke(target, args);
        long elapsed = System.nanoTime() - start;

        LOGGER.info("Executing {} finished in {} ns", method.getName(),
                elapsed);

        return result;
    }
}
```

随后，该代理可用于各种对象类型：

```java
Map mapProxyInstance = (Map) Proxy.newProxyInstance(
    DynamicProxyTest.class.getClassLoader(), new Class[] { Map.class }, 
    new TimingDynamicInvocationHandler(new HashMap<>()));

mapProxyInstance.put("hello", "world");

CharSequence csProxyInstance = (CharSequence) Proxy.newProxyInstance(
    DynamicProxyTest.class.getClassLoader(), 
    new Class[] { CharSequence.class }, 
    new TimingDynamicInvocationHandler("Hello World"));

csProxyInstance.length()
```

这里，我们代理了一个Map和一个CharSequence(字符串)。

代理方法的调用将委托给包装对象并生成日志语句：

```text
Executing put finished in 19153 ns 
Executing get finished in 8891 ns 
Executing charAt finished in 11152 ns 
Executing length finished in 10087 ns
```

## 6. 总结

在本快速教程中，我们研究了Java的动态代理及其一些可能的用途。