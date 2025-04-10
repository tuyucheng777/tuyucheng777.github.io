---
layout: post
title:  使用Java枚举实现简单状态机
category: algorithms
copyright: algorithms
excerpt: 状态机
---

## 1. 概述

在本教程中，我们将介绍[状态机](https://www.baeldung.com/cs/state-machines)以及如何使用枚举在Java中实现它们。

我们还将解释这种实现与为每个状态使用一个接口和具体类相比的优势。

## 2. Java枚举

Java [Enum](https://www.baeldung.com/a-guide-to-java-enums)是一种特殊类型的类，它定义常量列表，**这允许实现类型安全，并使代码更具可读性**。

举个例子，假设我们有一个人力资源软件系统，可以批准员工提交的休假申请。该申请由团队负责人审核，然后上报给部门经理，部门经理是负责批准该申请的人。

保存休假申请状态的最简单的枚举是：
```java
public enum LeaveRequestState {
    Submitted,
    Escalated,
    Approved
}
```

我们可以引用这个枚举的常量：
```java
LeaveRequestState state = LeaveRequestState.Submitted;
```

**枚举也可以包含方法**，我们可以在枚举中编写一个抽象方法，这将强制每个枚举实例实现此方法。这对于状态机的实现非常重要，我们将在下面看到。

由于Java枚举隐式扩展了java.lang.Enum类，因此它们无法扩展其他类。但是，它们可以像任何其他类一样实现接口。

以下是包含抽象方法的枚举的示例：
```java
public enum LeaveRequestState {
    Submitted {
        @Override
        public String responsiblePerson() {
            return "Employee";
        }
    },
    Escalated {
        @Override
        public String responsiblePerson() {
            return "Team Leader";
        }
    },
    Approved {
        @Override
        public String responsiblePerson() {
            return "Department Manager";
        }
    };

    public abstract String responsiblePerson();
}
```

注意最后一个枚举常量末尾的分号用法，当常量后面有一个或多个方法时，必须使用分号。

在本例中，我们扩展了第一个示例，添加了responsiblePerson()方法，这告诉我们负责执行每个操作的人员。因此，如果我们尝试检查Escalated状态的人员，它将给我们“Team Leader”：
```java
LeaveRequestState state = LeaveRequestState.Escalated;
assertEquals("Team Leader", state.responsiblePerson());
```

同样，如果我们检查谁负责批准请求，它会给我们“Department Manager”：
```java
LeaveRequestState state = LeaveRequestState.Approved;
assertEquals("Department Manager", state.responsiblePerson());
```

## 3. 状态机

状态机(也称为有限状态机或有限自动机)是一种用于构建抽象机器的计算模型，**这些机器在给定时间内只能处于一种状态**。每个状态都是系统的状态，会转变为另一种状态。这些状态变化称为转换。

在数学中，图表和符号可能会很复杂，但对于我们程序员来说，事情要容易得多。

[状态模式](https://www.baeldung.com/java-state-design-pattern)是GoF著名的二十三种设计模式之一，该模式借用了数学模型的概念，**它允许一个对象根据其状态为同一对象封装不同的行为。我们可以对状态之间的转换进行编程，然后定义单独的状态**。

为了更好地解释这个概念，我们将扩展我们的休假请求示例来实现状态机。

## 4. 枚举作为状态机

我们将重点介绍Java中状态机的枚举实现，还有[其他实现](https://www.baeldung.com/java-state-design-pattern)，我们将在下一节中对它们进行比较。

使用枚举实现状态机的要点是我们不必明确设置状态；相反，我们可以只提供如何从一个状态转换到下一个状态的逻辑：
```java
public enum LeaveRequestState {

    Submitted {
        @Override
        public LeaveRequestState nextState() {
            return Escalated;
        }

        @Override
        public String responsiblePerson() {
            return "Employee";
        }
    },
    Escalated {
        @Override
        public LeaveRequestState nextState() {
            return Approved;
        }

        @Override
        public String responsiblePerson() {
            return "Team Leader";
        }
    },
    Approved {
        @Override
        public LeaveRequestState nextState() {
            return this;
        }

        @Override
        public String responsiblePerson() {
            return "Department Manager";
        }
    };

    public abstract LeaveRequestState nextState(); 
    public abstract String responsiblePerson();
}
```

在此示例中，**状态机转换是使用枚举的抽象方法实现的**。更准确地说，使用每个枚举常量上的nextState()指定到下一个状态的转换。如果需要，我们还可以实现previousState()方法。

下面是检查我们实现的测试：
```java
LeaveRequestState state = LeaveRequestState.Submitted;

state = state.nextState();
assertEquals(LeaveRequestState.Escalated, state);

state = state.nextState();
assertEquals(LeaveRequestState.Approved, state);

state = state.nextState();
assertEquals(LeaveRequestState.Approved, state);
```

我们在Submitted初始状态下启动休假申请，然后我们使用上面实现的nextState()方法验证状态转换。

请注意，**由于“Approved”是最终状态，因此不能发生其他转换**。

## 5. 使用Java枚举实现状态机的优点

使用接口和实现类的[状态机的实现](https://www.baeldung.com/java-state-design-pattern)可能需要大量代码来开发和维护。

由于Java枚举最简单的形式是常量列表，因此我们可以使用枚举来定义状态。而且由于枚举还可以包含行为，因此我们可以使用方法来提供状态之间的转换实现。

**将所有逻辑放在一个简单的枚举中可以提供一个干净、直接的解决方案**。

## 6. 总结

在本文中，我们研究了状态机以及如何使用枚举在Java中实现它们，并给出了一个示例并对其进行了测试。

最后，我们还讨论了使用枚举实现状态机的优点。作为接口和实现解决方案的替代方案，枚举提供了一种更清晰、更易于理解的状态机实现。