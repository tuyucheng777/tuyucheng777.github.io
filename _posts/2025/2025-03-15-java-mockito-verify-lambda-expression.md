---
layout: post
title:  使用Mockito验证是否进行了Lambda调用
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们将介绍如何测试我们的代码是否调用了[Lambda](https://www.baeldung.com/java-8-lambda-expressions-tips)函数。有两种方法可以实现此目标。我们首先检查是否使用正确的参数调用了Lambda。然后，我们将测试行为并检查Lambda代码是否已执行并产生了预期结果。

## 2. 测试示例类

首先，让我们创建一个LambdaExample类，其中包含一个我们称之为bricksList的[ArrayList](https://www.baeldung.com/java-arraylist)：

```java
class LambdaExample {
    ArrayList<String> bricksList = new ArrayList<>();
}
```

现在，让我们添加一个名为BrickLayer的内部类，它可以为我们添加元素：

```java
class LambdaExample {

    BrickLayer brickLayer = new BrickLayer();

    class BrickLayer {
        void layBricks(String bricks) {
            bricksList.add(bricks);
        }
    }
}
```

BrickLayer的功能不多，它只有一个方法layBricks()，它将为我们向List添加一个元素。这本来可以是一个外部类，但是为了保持概念的统一和简单，这里使用了一个内部类。

最后，我们可以向LambdaExample添加一个方法，通过Lambda调用layBricks()：

```java
void createWall(String bricks) {
    Runnable build = () -> brickLayer.layBricks(bricks);
    build.run();
}
```

再次强调，我们尽量保持简单。实际应用更加复杂，但这个简化的示例将有助于解释测试方法。

在接下来的部分中，我们将测试调用createWall()是否会导致我们的Lambda中layBricks()的预期执行。

## 3. 测试正确调用

**我们将要研究的第一个测试方法是基于确认Lambda是否按预期被调用**，此外，我们需要确认它是否收到了正确的参数。首先，我们需要创建BrickLayer和LambdaExample的[Mocks](https://www.baeldung.com/mockito-series)：

```java
@Mock
BrickLayer brickLayer;
@InjectMocks
LambdaExample lambdaExample;
```

我们已经将[@InjectMocks](https://www.baeldung.com/mockito-annotations)注解应用于LambdaExample，以便它使用Mock的BrickLayer对象。因此，我们将能够确认对layBricks()方法的调用。

现在我们可以编写测试了：

```java
@Test
void whenCallingALambda_thenTheInvocationCanBeConfirmedWithCorrectArguments() {
    String bricks = "red bricks";
    lambdaExample.createWall(bricks);
    verify(brickLayer).layBricks(bricks);
}
```

在此测试中，我们定义了要添加到bricksList的字符串，并将其作为参数传递给createWall()。请记住，我们使用先前创建的Mock作为LambdaExample的实例。

然后我们使用了Mockito的[verify()](https://www.baeldung.com/mockito-verify)函数，**Verify()对于这种测试非常有用，它确认函数layBricks()被调用，并且参数符合我们的预期**。

我们可以用verify()做很多事情。例如，确认某个方法被调用了多少次。但就我们的目的而言，确认我们的Lambda是否按预期调用了该方法就足够了。

## 4. 测试正确的行为

我们可以进行测试的第二种方法是，不要担心调用什么以及何时调用。**相反，我们将确认Lambda函数的预期行为是否发生**。我们调用函数几乎总是有充分的理由，也许是为了执行计算或获取或设置变量。

在我们的示例中，Lambda将给定的String添加到ArrayList。在本节中，让我们验证Lambda是否成功执行该任务：

```java
@Test
void whenCallingALambda_thenCorrectBehaviourIsPerformed() {
    LambdaExample lambdaExample = new LambdaExample();
    String bricks = "red bricks";
        
    lambdaExample.createWall(bricks);
    ArrayList<String> bricksList = lambdaExample.getBricksList();
        
    assertEquals(bricks, bricksList.get(0));
}
```

这里，我们创建了LambdaExample类的一个实例。接下来，我们调用createWall()将一个元素添加到ArrayList中。

我们现在应该看到bricksList包含我们刚刚添加的字符串，假设代码正确执行了Lambda，我们通过从lambdaExample中检索bricksList并检查内容来确认这一点。

我们可以得出结论，Lambda正在按预期执行，因为这是我们的String最终进入ArrayList的唯一方式。

## 5. 总结

在本文中，我们研究了两种测试Lambda调用的方法。第一种方法很有用，因为我们可以Mock包含该函数的类，并将其注入到将其作为Lambda调用的类中。在这种情况下，我们可以使用Mockito来验证对函数的调用和正确的参数。然而，这并不能保证Lambda会按照我们的预期继续执行。

另一种方法是测试Lambda在调用时是否产生预期结果，这提供了更多的测试覆盖率，并且如果访问和确认函数调用的正确行为很简单，那么这种方法通常是更好的选择。