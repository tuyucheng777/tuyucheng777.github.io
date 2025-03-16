## 1. 概述

在本教程中，我们将简要解释为什么直接测试私有方法[通常不是一个好主意](https://www.baeldung.com/cs/testing-public-vs-private-methods)。然后，我们将演示如何在Java中测试私有方法(如果有必要)。

## 2. 为什么我们不应该测试私有方法

通常，我们编写的单元测试应该只检查公共方法契约。私有方法是公共方法的调用者不知道的实现细节。此外，更改实现细节不应导致我们更改测试。

一般来说，敦促测试私有方法会凸显以下问题之一：

- 我们的私有方法中有死代码。
- 我们的私有方法太复杂，应该属于另一个类。
- 我们的方法一开始就不是私密的。

因此，当我们觉得需要测试私有方法时，我们真正应该做的是修复底层的设计问题。

## 3.示例：从私有方法中删除死代码

让我们展示一个简单的例子。

我们将编写一个私有方法，该方法将返回Integer的双精度值。对于空值，我们希望返回null：

```java
private static Integer doubleInteger(Integer input) {
    if (input == null) {
        return null;
    }
    return 2  input;
}
```

现在，让我们编写公共方法。它将是类外部的唯一入口点。

此方法接收一个Integer作为输入。它验证此Integer不为空； 否则，它将抛出[IllegalArgumentException](https://www.baeldung.com/java-illegalargumentexception-or-nullpointerexception)。之后，它调用私有方法返回两倍的Integer值：

```java
public static Integer validateAndDouble(Integer input) {
    if (input == null) {
        throw new IllegalArgumentException("input should not be null");
    }
    return doubleInteger(input);
}
```

让我们遵循良好的做法并测试我们的公共方法合同。

首先，让我们编写一个测试，确保当输入为null时抛出[IllegalArgumentException](https://www.baeldung.com/java-illegalargumentexception-or-nullpointerexception)：

```java
@Test
void givenNull_WhenValidateAndDouble_ThenThrows() {
    assertThrows(IllegalArgumentException.class, () -> validateAndDouble(null));
}
```

现在让我们检查非空整数是否正确加倍：

```java
@Test
void givenANonNullInteger_WhenValidateAndDouble_ThenDoublesIt() {
    assertEquals(4, validateAndDouble(2));
}
```

让我们看一下[JaCoCo 插件报告的覆盖率](https://www.baeldung.com/jacoco)：

[![我们的方法的代码覆盖率](https://www.baeldung.com/wp-content/uploads/2022/06/public-and-private-method-code-coverage.png)](https://www.baeldung.com/wp-content/uploads/2022/06/public-and-private-method-code-coverage.png)我们可以看到，私有方法中的空值检查未被单元测试覆盖。那么我们应该测试它吗？

答案是否定的。重要的是要明白我们的私有方法不是凭空而来的。它只会在我们的公共方法中验证数据后才被调用。因此，我们私有方法中的空值检查永远不会被达到；它是死代码，应该被删除。

## 4.如何在Java中测试私有方法

假设我们不会气馁，让我们具体解释一下如何测试我们的私有方法。

为了测试它，如果我们的私有方法有另一种可见性将会很有帮助。好消息是我们能够使用[反射](https://www.baeldung.com/java-reflection)来模拟它。

我们的封装类名为Utils。其思想是访问名为doubleInteger 的私有方法，该方法接受一个Integer作为参数。然后我们将修改其可见性，以便从Utils类外部访问。让我们看看如何做到这一点：

```java
private Method getDoubleIntegerMethod() throws NoSuchMethodException {
    Method method = Utils.class.getDeclaredMethod("doubleInteger", Integer.class);
    method.setAccessible(true);
    return method;
}
```

现在我们可以使用这个方法了。让我们编写一个测试来确保给定一个null对象，我们的私有方法返回null。我们需要将该方法应用于将为null 的参数：

```java
@Test
void givenNull_WhenDoubleInteger_ThenNull() throws InvocationTargetException, IllegalAccessException, NoSuchMethodException {
    assertEquals(null, getDoubleIntegerMethod().invoke(null, new Integer[] { null }));
}
```

让我们进一步解释一下[invoke](https://www.baeldung.com/java-method-reflection)方法的用法。第一个参数是我们应用该方法的对象。由于doubleInteger是静态的，因此我们传入了一个null。第二个参数是一个参数数组。在本例中，我们只有一个参数，即null。

最后，让我们演示如何测试非空输入的情况：

```java
@Test
void givenANonNullInteger_WhenDoubleInteger_ThenDoubleIt() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    assertEquals(74, getDoubleIntegerMethod().invoke(null, 37));
}
```

## 5. 总结

在本文中，我们了解了为什么测试私有方法通常不是一个好主意。然后我们演示了如何使用反射来测试Java中的私有方法。