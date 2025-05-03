---
layout: post
title:  什么是鸭子类型
category: programming
copyright: programming
excerpt: 编程
---

## 1. 简介

“鸭子测试”是一种口语化的推理形式，通常用来表示一个人只需观察未知事物的独特特征就能识别它：

```text
If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.
```

这种推理并不总是正确的，但它经常在很多情况下使用，包括鸭子类型。

**更准确地说，在计算机编程和类型理论中，鸭子类型是一种[类型](https://www.baeldung.com/cs/programming-types-comparison)技术，其中对象被赋予一种类型当且仅当它定义了该类型所需的所有属性**。

在本教程中，我们将扩展上述定义，并通过一些示例了解鸭子类型是什么。

## 2. 定义和示例

我们上面看到的“鸭子类型”解释在类型论中是合理的,然而，它有更简单、更直观的定义。换句话说，鸭子类型意味着我们根据对象能做什么来定义它，而不是根据它是什么。**对象的类型在这里并不重要，它的结构也不重要：我们唯一感兴趣的是对象能做什么**。

更正式地说，鸭子类型语言在[编译时或运行时](https://www.baeldung.com/cs/runtime-vs-compile-time)都不会执行任何类型检查。相反，它会在运行时尝试调用指定的方法。如果成功，那就很好。否则，它会抛出某种形式的运行时错误。

让我们看一个鸭子类型语言[Python](https://docs.python.org/3/)中的例子：

```python
class Duck:
    def swim(self):
        print("The duck is swimming")

    def fly(self):
        print("The duck is flying")
        
class Pelican:
    def swim(self):
        print("The pelican is swimming")

    def fly(self):
        print("The pelican is flying")
        
class Penguin:
    def swim(self):
        print("The penguin is swimming")
        
        
duck = Duck()
pelican = Pelican()
penguin = Penguin()

duck.swim()
duck.fly()

pelican.swim()
pelican.fly()

penguin.swim()
penguin.fly()
```

在上面的例子中，我们定义了三个不同的类：Duck、Pelican和Penguin，并分别实现了fly和swim的方法。然而，Penguin不会飞。因此，我们的Penguin类不会实现任何fly()方法。

在类定义之后，我们为每个类创建了一个实例，并在每个实例上调用swim()和fly()方法，输出如下：

```text
The duck is swimming
The duck is flying
The pelican is swimming
The pelican is flying
The penguin is swimming
Traceback (most recent call last):
  File "main.py", line 31, in <module>
    penguin.fly()
AttributeError: 'Penguin' object has no attribute 'fly'
```

输出清晰地展现了鸭子类型的作用，Python中没有[编译器](https://www.baeldung.com/cs/how-compilers-work)预先告诉我们penguin.fly()是一个错误。相反，我们在运行时会得到一个AttributeError。

### 2.1 多态性

简而言之，“多态性”的意思是“具有多种形态”。**在类型论和计算机科学中，多态性是编程语言的一种属性，允许我们使用一个符号来表示多种类型**。换句话说，当需要超类型时，我们可以使用子类型的实例。

让我们看一个[Java](https://www.java.com/en/)中的例子：

```java
interface SwimmingAnimal {
    public void swim();
}

class Duck implements SwimmingAnimal {
    @Override
    public void swim() {
        System.out.println("The duck is swimming");
    }
}

public class Main {
    public static void test(SwimmingAnimal sa) {
        sa.swim();
    }

    public static void main(String[] args) {
        test(new Duck());
    }
}
```

在上面的例子中，test()方法输入了一个SwimmingAnimal类型的参数。因此，我们可以创建一个Duck的实例(实现了SwimmingAnimal接口)，并将其作为test()的参数。

**鸭子类型是多态性的一种形式**，正如我们上面所见，它允许我们交替使用不同类型的对象，前提是它们实现了某些行为(也就是它们的接口)。它与其他形式的多态性(例如Java的多态性)的主要区别在于，对象不必继承自相同的超类/接口。

## 3. 鸭子类型的优缺点

正如我们上面所看到的，在鸭子类型语言中，我们没有编译器在运行代码之前用类型标记表达式。这有几个优点，但也是有代价的：

| 优点|                |                                                        |
| -------- | -------------- | ------------------------------------------------------ |
|          | 轻松原型| 我们可以创建类似上面的代码片段，而不必担心定义类型或接口|
|          | 代码重用| 使用鸭子类型，无需创建复杂的类层次结构来重用代码|
|          | 灵活性| 我们可以根据对象的行为互换使用它们，而不必担心用类型标记它们|
| 缺点|                |                                                        |
|          | 运行时错误| 在上面的Python示例中，编译器会阻止我们运行在运行时会失败的代码片段|
|          | 可读性| 代码不太明确，因为我们没有任何类型来记录特定函数所需的行为|
|          | 调试与维护| 更难阅读的代码也更难调试和推理|

## 4. 适用性

**一般来说，鸭子类型对于原型设计和脚本编写非常有用**。如果我们编写的应用程序是一个简单的脚本，那么在代码中不显式地定义类型可以加快开发过程(同样，代价是增加了运行时错误的风险)。对于原型设计也是如此：概念验证的实现可能会避开类型的形式化，而专注于解决特定问题的业务逻辑。

**鸭子类型另一个相关的应用是序列化/反序列化**，例如，为每个JSON对象定义类型可能很麻烦，鸭子类型让我们假设发送/接收的对象存在某些属性，从而简化代码。

## 5. 总结

在本文中，我们深入探讨了鸭子类型，并从不同的角度对其进行了定义。此外，我们还通过代码示例演示了它的工作原理及其优缺点。最后，我们探讨了鸭子类型与多态性之间的关系。