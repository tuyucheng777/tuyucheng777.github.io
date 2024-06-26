---
layout: post
title:  单例模式与双重检查锁定
category: designpattern
copyright: designpattern
excerpt: Design Pattern
---

## 1. 简介

在本教程中，我们介绍双重检查锁定设计模式。这种模式通过简单地预先检查锁定条件来减少锁定获取的次数。因此，通常会提高性能。但是，应该注意的是，[双重检查锁定](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)在Java 5之前已被破坏。

## 2. 实现

首先，让我们考虑一个具有严格同步的简单单例：

```java
public class DraconianSingleton {
    private static DraconianSingleton instance;

    public static synchronized DraconianSingleton getInstance() {
        if (instance == null) {
            instance = new DraconianSingleton();
        }
        return instance;
    }

    // private constructor and other methods ...
}
```

尽管这个类是线程安全的，但我们可以看到存在明显的性能缺陷：每次我们想要获取单例的实例时，我们都需要获取一个可能不必要的锁。

**为了解决这个问题，我们可以先验证是否需要首先创建对象，只有在这种情况下我们才会获取锁**。

更进一步，我们希望在进入同步块后立即再次执行相同的检查，以保持操作的原子性：

```java
public class DclSingleton {
    private static volatile DclSingleton instance;

    public static DclSingleton getInstance() {
        if (instance == null) {
            synchronized (DclSingleton.class) {
                if (instance == null) {
                    instance = new DclSingleton();
                }
            }
        }
        return instance;
    }

    // private constructor and other methods...
}
```

使用此模式要记住的一件事是，**字段需要保持volatile以防止缓存不一致问题**。事实上，Java 内存模型允许发布部分初始化的对象，这反过来可能会导致细微的错误。

## 3. 备选方案

尽管双重检查锁定可能会加快速度，但它至少有两个问题：

-   因为它需要volatile关键字才能正常工作，所以它与Java 1.4及更低版本不兼容
-   它非常冗长并且使代码难以阅读

由于这些原因，接下来我们介绍一些没有这些缺陷的其他选择，以下所有方法都将同步任务委托给JVM。

### 3.1 早期初始化

实现线程安全的最简单方法是内联对象创建或使用等效的静态块。这利用了静态字段和块一个接一个地初始化这一事实([Java语言规范12.4.2](https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.4.2))：

```java
public class EarlyInitSingleton {
    private static final EarlyInitSingleton INSTANCE = new EarlyInitSingleton();

    public static EarlyInitSingleton getInstance() {
        return INSTANCE;
    }

    // private constructor and other methods...
}
```

### 3.2 按需初始化

此外，由于我们从上一段的Java语言规范参考中知道，类初始化发生在我们第一次使用其中一个方法或字段时，因此我们可以使用嵌套静态类来实现延迟初始化：

```java
public class InitOnDemandSingleton {
    private static class InstanceHolder {
        private static final InitOnDemandSingleton INSTANCE = new InitOnDemandSingleton();
    }

    public static InitOnDemandSingleton getInstance() {
        return InstanceHolder.INSTANCE;
    }

    // private constructor and other methods...
}
```

在这种情况下，InstanceHolder类将在我们第一次通过调用getInstance访问它时分配该字段。

### 3.3 枚举单例

最后一个解决方案来自Joshua Block的Effective Java书(第3项)，并使用枚举而不是类。在撰写本文时，这被认为是编写单例最简洁、最安全的方法：

```java
public enum EnumSingleton {
    INSTANCE

    // other methods...
}
```

## 4. 总结

这篇简短的文章介绍了双重检查锁定模式、它的限制和一些替代方案。在实践中，过于冗长和缺乏向后兼容性使这种模式容易出错，因此我们应该避免它。相反，我们应该使用一种让JVM进行同步的替代方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/design-patterns-modules)上获得。