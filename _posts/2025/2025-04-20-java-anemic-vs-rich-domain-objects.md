---
layout: post
title:  贫血与富域对象
category: designpattern
copyright: designpattern
excerpt: 贫血对象
---

## 1. 概述

在本文中，我们将探讨贫血领域模型和富领域模型之间的区别。首先，我们将定义富对象，并将其与贫血对象进行对比。然后，我们将分析一个实际的代码示例，并通过封装数据和为领域模型建立健壮的API来逐步增强其设计。

## 2. 贫血对象与富对象

让我们首先了解一下什么是富对象和贫血对象，《代码整洁之道》的作者Robert C.Martin在他的[个人博客](https://blog.cleancoder.com/uncle-bob/2019/06/16/ObjectsAndDataStructures.html)上讨论了贫血对象的概念，并将其称为“数据结构”。他强调了数据结构和对象之间的根本区别，并指出：“类使函数可见，同时隐含数据。数据结构使数据可见，同时隐含函数。”

**简而言之，富对象隐藏了其底层数据，仅暴露一组公共方法来与其交互。相比之下，贫血对象和数据结构则暴露其数据，并依赖外部组件进行操作**。

### 2.1 富对象

在面向对象编程(OOP)中，对象是一组操作封装数据的函数。一个常见的错误是将对象视为简单的元素集合，并通过直接操作其字段来满足业务需求，从而破坏其封装性。

**为了更深入地理解领域并构建丰富的领域模型，我们应该封装数据。因此，我们将对象视为自主实体，专注于它们的公共接口来实现业务用例**。 

### 2.2 贫血对象

**相反，贫血对象仅暴露一组旨在由隐式函数操作的数据元素**。例如，我们可以想到一个[DTO(数据传输对象)](https://www.baeldung.com/java-dto-pattern)：它通过Getter和Setter暴露其字段，但它不知道如何对它们执行任何操作。

对于本文中的代码示例，假设我们正在开发一个模拟网球比赛的应用程序，让我们看看这个应用程序的贫血领域模型是什么样子的：

```java
public class Player {
    private String name;
    private int points;

    // constructor, getters and setters
}
```

我们看到，Player类没有提供任何方法，它通过Getter和Setter暴露所有字段。随着文章的深入，我们将逐步丰富我们的领域模型，封装其数据。

## 3. 封装

**缺乏封装是贫血模型的主要症状之一，假设数据是通过Getter和Setter暴露的，在这种情况下，我们面临的风险是，与模型相关的逻辑会扩散到整个应用程序，并可能在不同的领域服务中重复出现**。

因此，丰富Player模型的第一步是探究其Getter和Setter方法。让我们看一下Player类的一些简单用法，并了解如何在网球比赛中使用这些数据：

```java
public class TennisGame {

    private Player server;
    private Player receiver;

    public TennisGame(String serverName, String receiverName) {
        this.server = new Player(serverName, 0);
        this.receiver = new Player(receiverName, 0);
    }

    public void wonPoint(String playerName) {
        if(server.getName().equals(playerName)) {
            server.setPoints(server.getPoints() + 1)
        } else {
            receiver.setPoints(receiver.getPoints() + 1);
        }
    }

    public String getScore() {
        // some logic that uses the private methods below
    }

    private boolean isScoreEqual() {
        return server.getPoints() == receiver.getPoints();
    }

    private boolean isGameFinished() {
        return leadingPlayer().getPoints() > Score.FORTY.points
                && Math.abs(server.getPoints() - receiver.getPoints()) >= 2;
    }

    private Player leadingPlayer() {
        if (server.getPoints() - receiver.getPoints() > 0) {
            return server;
        }
        return receiver;
    }

    public enum Score {
        LOVE(0, "Love"),
        FIFTEEN(1, "Fifteen"),
        THIRTY(2, "Thirty"),
        FORTY(3, "Forty");

        private final int points;
        private final String label;
        // constructor 
    }
}
```

### 3.1 质疑Setter

首先，我们来看一下代码中的Setter方法。目前，Player名称作为构造函数参数传递，之后不会更改。因此，我们可以通过移除相应的Setter方法，安全地将其设置为不可变。

接下来，我们观察到Player每次只能获得一分。因此，我们可以用更专业的wonPoint()方法替换现有的Setter方法，该方法会将Player的得分加1：

```java
public class Player {
    private final String name;
    private int points;

    public Player(String name) {
        this.name = name;
        this.points = 0;
    }

    public void wonPoint() {
        this.points++;
    }
    // getters
}
```

### 3.2 质疑Getter

分数Getter函数被多次使用，用于比较两个Player之间的分数差距，让我们引入一个返回当前Player与其对手之间分数差距的方法：

```java
public int pointsDifference(Player opponent) {
    return this.points - opponent.points;
}
```

为了检查Player是否处于“优势”或者是否赢得了游戏，我们需要一个额外的方法来检查Player的得分是否大于给定值：

```java
public boolean hasScoreBiggerThan(Score score) {
    return this.points > score.points();
}
```

现在，让我们删除Getter并改用Player对象上的丰富接口：

```java
private boolean isScoreEqual() {
    return server.pointsDifference(receiver) == 0;
}

private Player leadingPlayer() {
    if (server.pointsDifference(receiver) > 0) {
        return server;
    }
    return receiver;
}

private boolean isGameFinished() {
    return leadingPlayer().hasScoreBiggerThan(Score.FORTY)
            && Math.abs(server.pointsDifference(receiver)) >= 2;
}

private boolean isAdvantage() {
    return leadingPlayer().hasScoreBiggerThan(Score.FORTY)
            && Math.abs(server.pointsDifference(receiver)) == 1;
}
```

## 4. 低耦合

**富领域模型带来了低耦合的设计**，通过移除getPoints()和setPoints()方法并增强对象的API，我们成功地隐藏了实现细节。让我们再来看看Player类：

```java
public class Player {
    private final String name;
    private int points;

    public Player(String name) {
        this.name = name;
        this.points = 0;
    }

    public void gainPoint() {
        points++;
    }

    public boolean hasScoreBiggerThan(Score score) {
        return this.points > score.points();
    }

    public int pointsDifference(Player other) {
        return points - other.points;
    }

    public String name() {
        return name;
    }

    public String score() {
        return Score.from(points).label();
    }
}
```

如我们所见，我们可以轻松地更改内部数据结构。例如，我们可以创建一个新类来存储Player的分数(而不是依赖于int原始类型)，而不会影响任何使用Player类的客户端。

## 5. 高内聚

富模型还可以增强我们领域的凝聚力，并符合[单一责任原则](https://www.baeldung.com/java-single-responsibility-principle)。在本例中，Player实例负责管理其赢得的分数，而TennisGame类则负责协调两名Player并跟踪游戏的总得分。

**然而，在将这些小逻辑从用例实现移到模型中时，我们应该小心谨慎。根据经验法则，我们应该只移动与用例无关的函数，以保持较高的内聚性**。

换句话说，我们可能会想在Player类中添加一个类似“hasWonOver(Player opposition)”的方法，但这条规则只有在Player之间互相对抗时才有意义。此外，这不是一个独立于用例的规则：根据比赛赛制(例如，单打、双打、三局两胜、五局三胜或其他赛制)，获胜条件可能会有所不同。

## 6. 增强表现力

**富领域模型的另一个好处是，它使我们能够降低领域服务或用例类的复杂性**。换句话说，TennisGame类现在将更具表现力，通过隐藏与Player相关的细节，开发人员可以专注于业务规则。

**质疑Getter和Setter用法以及修改Player类公共API的过程，促使我们更深入地理解领域模型及其功能**。这一步非常重要，但由于使用IDE或[Lombok](https://www.baeldung.com/intro-to-project-lombok)等工具自动生成Getter和Setter的便利性，我们常常忽略它。

## 7. 总结

在本文中，我们讨论了贫血对象的概念以及采用富领域模型的优势。随后，我们提供了一个实际示例，展示了如何封装对象的数据并提供改进的接口。最后，我们发现了这种方法的诸多优势，包括提升表达能力、增强内聚力以及降低耦合度。