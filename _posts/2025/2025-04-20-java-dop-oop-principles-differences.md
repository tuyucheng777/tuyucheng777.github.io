---
layout: post
title:  Java中的面向数据编程
category: designpattern
copyright: designpattern
excerpt: 面向数据编程
---

## 1. 概述

在本教程中，我们将学习一种不同的软件开发范式-面向数据编程。我们将首先将其与更传统的面向对象编程进行比较，并重点介绍它们之间的区别。

之后，我们将进行一个动手练习，运用面向数据编程(DOP)来实现Yahtzee游戏。**在整个练习过程中，我们将重点学习面向数据编程(DOP)原则，并利用[记录](https://www.baeldung.com/java-record-keyword)、[密封接口](https://www.baeldung.com/java-sealed-classes-interfaces)和[模式匹配](https://www.baeldung.com/java-switch-pattern-matching)等现代Java特性**。

## 2. 原则

面向数据编程是一种范式，我们围绕数据结构和流程而不是对象或函数来设计应用程序，这种软件设计方法围绕三个关键原则：

- 数据与操作它的逻辑分离
- 数据存储在通用且透明的数据结构中
- 数据不可变，并且始终处于有效状态

通过仅允许创建有效实例并阻止更改，我们确保应用程序始终拥有有效数据。因此，**遵循这些规则将导致非法状态无法表示**。

## 3. 面向数据与面向对象编程

如果我们遵循这些原则，最终会得到与传统[面向对象编程](https://www.baeldung.com/java-oop)(OOP)截然不同的设计。一个关键区别在于，OOP使用接口来实现依赖反转和多态性，这是将逻辑与跨边界依赖解耦的有效工具。

相反，使用DOP时，我们不允许混合数据和逻辑，因此，我们无法多态地调用数据类的行为。此外，OOP使用封装来隐藏数据，而DOP则倾向于使用通用且透明的数据结构，例如Map、Tuple和Record。

**总而言之，面向数据编程(OOP)适用于数据所有权明确且对外部依赖保护要求不高的小型应用程序**。另一方面，OOP仍然是定义清晰模块边界或允许客户端通过插件扩展软件功能的可靠选择。

## 4. 数据模型

在本文的代码示例中，我们将实现Yahtzee游戏的规则。首先，让我们回顾一下游戏的主要规则：

- 每轮游戏开始时，玩家要掷5个六面骰子
- 玩家可以选择重新掷部分或全部骰子，最多3次
- 然后玩家选择一种得分策略，例如“一”、“二”、“对”、“两对”、“三条”...等等
- 最后玩家根据得分策略和骰子获得分数

**现在我们有了游戏规则，我们可以应用面向数据的原则来模拟我们的域**。

### 4.1 将数据与逻辑分离

**我们讨论的第一个原则是分离数据和行为，我们将应用它来创建各种评分策略**。

我们可以将策略视为具有多个实现的接口，此时，我们无需支持所有可能的策略；我们可以专注于其中几个，并通过封装接口来指明我们允许的策略：

```java
sealed interface Strategy permits Ones, Twos, OnePair, TwoPairs, ThreeOfaKind {
}
```

我们可以观察到，Strategy接口没有定义任何方法。**从OOP的角度来看，这可能看起来很奇怪，但将数据与操作数据的行为分离至关重要**。因此，具体的策略也不会暴露任何行为：

```java
class Strategies {
    record Ones() implements Strategy {
    }

    record Twos() implements Strategy {
    }

    record OnePair() implements Strategy {
    }

    // other strategies...
}
```

### 4.2 数据不变性和验证

众所周知，面向数据编程提倡使用存储在通用数据结构中的不可变数据。**Java记录非常适合这种方法，因为它们为不可变数据创建了透明的载体**，让我们使用记录来表示骰子Roll的值：

```java
record Roll(List<Integer> dice, int rollCount) { 
}
```

尽管记录本质上是不可变的，但它们的组成部分也必须是不可变的。例如，从可变列表创建Roll允许我们稍后修改骰子的值，为了避免这种情况，我们可以使用紧凑的构造函数将列表包装为unmodifiableList()：

```java
record Roll(List<Integer> dice, int rollCount) {
    public Roll {
        dice = Collections.unmodifiableList(dice);
    }
}
```

此外，我们可以使用此构造函数来验证数据：

```java
record Roll(List<Integer> dice, int rollCount) {
    public Roll {
        if (dice.size() != 5) {
            throw new IllegalArgumentException("A Roll needs to have exactly 5 dice.");
        }
        if (dice.stream().anyMatch(die -> die < 1 || die > 6)) {
            throw new IllegalArgumentException("Dice values should be between 1 and 6.");
        }

        dice = Collections.unmodifiableList(dice);
    }
}
```

### 4.3 数据组成

这种方法有助于使用数据类来捕获领域模型，**利用通用数据结构，无需特定的行为或封装，我们能够从较小的数据模型创建更大的数据模型**。

例如，我们可以将Turn表示为Roll和Strategy的并集：

```java
record Turn(Roll roll, Strategy strategy) {
}
```

**由此可见，我们仅通过数据建模就捕获了相当一部分业务规则**。虽然我们尚未实现任何行为，但通过检查数据可以发现，玩家通过掷骰子并选择策略来完成自己的回合。此外，我们还可以观察到支持的策略包括：Ones、Twos、OnePair和ThreeOfaKind。

## 5. 实现行为

现在我们有了数据模型，下一步就是实现操作它的逻辑。**为了保持数据和逻辑之间的明确分离，我们将使用静态函数并确保类保持无状态**。

让我们首先创建一个roll()函数，该函数返回一个包含五个骰子的Roll值：

```java
class Yahtzee {
    // private default constructor

    static Roll roll() {
        List<Integer> dice = IntStream.rangeClosed(1, 5)
                .mapToObj(__ -> randomDieValue())
                .toList();
        return new Roll(dice, 1);
    }

    static int randomDieValue() { /* ... */ }
}
```

然后，我们需要允许玩家重新掷出特定的骰子值。

```java
static Roll rerollValues(Roll roll, Integer... values) {
    List<Integer> valuesToReroll = new ArrayList<>(List.of(values));
    // arguments validation

    List<Integer> newDice = roll.dice()
            .stream()
            .map(it -> {
                if (!valuesToReroll.contains(it)) {
                    return it;
                }
                valuesToReroll.remove(it);
                return randomDieValue();
            }).toList();

    return new Roll(newDice, roll.rollCount() + 1);
}
```

正如我们所见，我们替换了重新掷出的骰子值并增加了rollCount，返回了Roll记录的新实例。

接下来，我们让玩家通过接收一个String参数来选择得分策略，我们将使用[静态工厂方法](https://www.baeldung.com/java-constructors-vs-static-factory-methods)创建相应的实现。当玩家完成回合后，我们将返回一个Turn实例，其中包含玩家的Roll值和所选的Strategy值：

```java
static Turn chooseStrategy(Roll roll, String strategyStr) {
    Strategy strategy = Strategies.fromString(strategyStr);
    return new Turn(roll, strategy); 
}
```

最后，我们将编写一个函数，根据所选的Strategy计算玩家在特定Turn的得分，我们将使用switch表达式和Java的模式匹配功能。

```java
static int score(Turn turn) {
    var dice = turn.roll().dice();
    return switch (turn.strategy()) {
        case Ones __ -> specificValue(dice, 1);
        case Twos __ -> specificValue(dice, 2);
        case OnePair __ -> pairs(dice, 1);
        case TwoPairs __ -> pairs(dice, 2);
        case ThreeOfaKind __ -> moreOfSameKind(dice, 3);
    };
}

static int specificValue(List<Integer> dice, int value) { /* ... */ }

static int pairs(List<Integer> dice, int nrOfPairs) { /* ... */ }

static int moreOfSameKind(List<Integer> dice, int nrOfDicesOfSameKind) { /* ... */ }
```

**使用不带default分支的模式匹配可以确保完整性，确保所有可能的情况都得到明确处理**。换句话说，如果我们决定支持新的Strategy，那么代码将无法编译，除非我们更新此switch表达式，使其包含新实现的评分规则。

我们可以观察到，我们的函数是无状态且无副作用的，仅对不可变数据结构执行转换。**此管道中的每个步骤都返回后续逻辑步骤所需的数据类型，从而定义转换的正确序列和顺序**：

```java
@Test
void whenThePlayerRerollsAndChoosesTwoPairs_thenCalculateCorrectScore() {
    enqueueFakeDiceValues(1, 1, 2, 2, 3, 5, 5);

    Roll roll = roll(); // => { dice: [1,1,2,2,3] }
    roll = rerollValues(roll, 1, 1); // => { dice: [5,5,2,2,3] }
    Turn turn = chooseStrategy(roll, "TWO_PAIRS");
    int score = score(turn);

    assertEquals(14, score);
}
```

## 6. 总结

在本文中，我们介绍了面向数据编程的关键原则以及它与OOP的区别。之后，我们探讨了Java语言的新功能如何为开发面向数据的软件奠定坚实的基础。