---
layout: post
title:  运行Spock测试时捕获方法参数
category: unittest
copyright: unittest
excerpt: Spock
---

## 1. 简介

当我们测试代码时，有时我们想要捕获传递给我们方法的参数。

在本教程中，我们将学习如何使用[Stubs、Mock和Spie](https://www.baeldung.com/spock-stub-mock-spy)捕获[Spock](https://www.baeldung.com/groovy-spock)测试中的参数并检查我们捕获的内容。我们还将学习如何使用不同的参数验证对同一Mock的多次调用并断言这些调用的顺序。

## 2. 测试对象

首先，我们需要一个接收我们想要捕获的单个参数的方法。

因此，让我们创建一个ArgumentCaptureSubject，它带有一个catchMeIfYouCan()方法，该方法接收一个字符串并返回前面带有“Received”的字符串：

```java
public class ArgumentCaptureSubject {
    public String catchMeIfYouCan(String input) {
        return "Received " + input;
    }
}
```

## 3. 准备数据驱动测试

我们将以Stub的典型用法开始我们的测试，并对其进行改进以捕获参数。

让我们创建一个类的Stub来返回存根响应“42”并调用其catchMeIfYouCan()方法：

```groovy
def "given a Stub when we invoke it then we capture the stubbed response"() {
    given: "an input and a result"
    def input = "Input"
    def stubbedResponse = "42"

    and: "a Stub for our response"
    @Subject
    ArgumentCaptureSubject stubClass = Stub()
    stubClass.catchMeIfYouCan(_) >> stubbedResponse

    when: "we invoke our Stub's method"
    def result = stubClass.catchMeIfYouCan(input)

    then: "we get our stubbed response"
    result == stubbedResponse
}
```

由于我们没有验证任何方法调用，因此我们在这个例子中使用了一个简单的Stub。

## 4. 捕捉参数

现在我们有了基本的测试，让我们看看如何捕获用来调用方法的参数。

首先，我们将声明一个方法作用域的变量，以便在捕获参数时进行分配：

```groovy
def captured
```

接下来，我们将用Groovy Closure替换静态stubbedResponse。**当调用存根方法时，Spock将向我们的Closure传递一个方法参数列表**。

让我们创建一个简单的闭包来捕获参数列表并将其分配给我们捕获的变量：

```groovy
{ arguments -> captured = arguments }
```

对于我们的断言，我们将断言捕获参数列表中索引0处的第一个元素等于我们的输入：

```groovy
captured[0] == input
```

因此，让我们用captured变量声明来更新我们的测试，用捕获参数的Closure替换stubbedResponse，并添加我们的断言：

```groovy
def "given a Stub when we invoke it then we capture the argument"() {
    given: "an input"
    def input = "Input"

    and: "a variable and a Stub with a Closure to capture our arguments"
    def captured
    @Subject
    ArgumentCaptureSubject stubClass = Stub()
    stubClass.catchMeIfYouCan(_) >> { arguments -> captured = arguments }

    when: "we invoke our method"
    stubClass.catchMeIfYouCan(input)

    then: "we captured the method argument"
    captured[0] == input
}
```

当我们想要返回stubbedResponse以及捕获参数时，我们更新闭包以返回它：

```groovy
{ arguments -> captured = arguments; return stubbedResponse }

...

then: "what we captured matches the input and we got our stubbed response"
captured == input
result == stubbedResponse
```

请注意，尽管我们为了清楚起见使用了“return”，但这并不是绝对必要的，因为Groovy闭包默认返回最后执行的语句的结果。

当我们只想捕获其中一个参数时，**我们可以通过在Closure中使用索引来捕获我们想要的参数**：

```groovy
{ arguments -> captured = arguments[0] }

...

then: "what we captured matches the input"
captured == input
```

在这种情况下，captured变量将与我们的参数具有相同的类型-String。

## 5. 间谍捕获

**当我们想要捕获一个值但又希望方法继续执行时，我们添加对Spy的callRealMethod()的调用**。

让我们更新我们的测试来使用Spy而不是Stub并在Closure中使用Spy的callRealMethod()：

```groovy
def "given a Spy when we invoke it then we capture the argument and then delegate to the real method"() {
    given: "an input string"
    def input = "Input"

    and: "a variable and a Spy with a Closure to capture the first argument and call the underlying method"
    def captured
    @Subject
    ArgumentCaptureSubject spyClass = Spy()
    spyClass.catchMeIfYouCan(_) >> { arguments -> captured = arguments[0]; callRealMethod() }

    when: "we invoke our method"
    def result = spyClass.catchMeIfYouCan(input)

    then: "what we captured matches the input and our result comes from the real method"
    captured == input
    result == "Received Input"
}
```

在这里，我们捕获了输入参数，而不会影响方法的返回值。

**当我们想要在将捕获的参数传递给真实方法之前对其进行更改时，我们会在闭包内部对其进行更新，然后使用Spy的callRealMethodWithArgs传递更新后的参数**。

因此，让我们更新闭包，在将其传递给真实方法之前，将“Tampered：”添加到字符串前面：

```groovy
spyClass.catchMeIfYouCan(_) >> { arguments -> captured = arguments[0]; callRealMethodWithArgs('Tampered:' + captured) }
```

让我们更新我们的断言来预期篡改的结果：

```groovy
result == "Received Tampered:Input"
```

## 6. 使用注入Mock捕获参数

现在我们已经了解了如何使用Spock的Mock框架来捕获参数，让我们将这种技术应用到具有我们可以模拟的依赖项的类中。

首先，让我们创建一个ArgumentCaptureDependency类，我们的主体可以使用一个简单的catchMe()方法调用它，该方法接收并修改String：

```java
public class ArgumentCaptureDependency {
    public String catchMe(String input) {
        return "***" + input + "***";
    }
}
```

现在，让我们使用一个接收ArgumentCaptureDependency的构造函数来更新我们的ArgumentCaptureSubject。我们还添加一个不带参数的callOtherClass方法，并使用一个参数调用我们的ArgumentCaptureDependency的catchMe()方法：

```java
public class ArgumentCaptureSubject {
    ArgumentCaptureDependency calledClass;

    public ArgumentCaptureSubject(ArgumentCaptureDependency calledClass) {
        this.calledClass = calledClass;
    }

    public String callOtherClass() {
        return calledClass.catchMe("Internal Parameter");
    }
}
```

最后，我们像之前一样创建一个测试。这次，我们在创建ArgumentCaptureSubject时注入一个Spy，以便我们也可以callRealMethod()并比较结果：

```groovy
def "given an internal method call when we invoke our subject then we capture the internal argument and return the result of the real method"() {
    given: "a mock and a variable for our captured argument"
    ArgumentCaptureDependency spyClass = Spy()
    def captured
    spyClass.catchMe(_) >> { arguments -> captured = arguments[0]; callRealMethod() }

    and: "our subject with an injected Spy"
    @Subject argumentCaptureSubject = new ArgumentCaptureSubject(spyClass)

    when: "we invoke our method"
    def result = argumentCaptureSubject.callOtherClass(input)

    then: "what we captured matches the internal method argument"
    captured == "Internal Parameter"
    result == "***Internal Parameter***"
}
```

我们的测试捕获了内部参数“Internal Parameter”。此外，我们对Spy的callRealMethod的调用确保我们不会影响方法的结果：“\*\*\*Internal Parameter\*\*\*”。

当我们不需要返回真实结果时，我们可以简单地使用Stub或Mock。

**请注意，当我们测试Spring应用程序时，我们可以使用Spock的[@SpringBean](https://spockframework.org/spock/docs/2.3/all_in_one.html#_using_springbean)[注解](https://spockframework.org/spock/docs/2.3/all_in_one.html#_using_springbean)注入我们的Mock**。

## 7. 从多个调用中捕获参数

有时，我们的代码会多次调用一个方法，并且我们希望捕获每次调用的值。

因此，让我们在ArgumentCaptureSubject中的callOtherClass()方法中添加一个字符串参数，我们将使用不同的参数调用该方法并捕获它们。

```java
public String callOtherClass(String input) {
    return calledClass.catchMe(input);
}
```

我们需要一个集合来捕获每次调用的参数。因此，我们将一个caughtStrings变量声明为ArrayList：

```groovy
def capturedStrings = new ArrayList()
```

现在，让我们创建测试并让它调用两次callOtherClass()，第一次使用“First”作为参数，第二次使用“Second”：

```groovy
def "given an dynamic Mock when we invoke our subject then we capture the argument for each invocation"() {
    given: "a variable for our captured arguments and a mock to capture them"
    def capturedStrings = new ArrayList()
    ArgumentCaptureDependency mockClass = Mock()

    and: "our subject"
    @Subject argumentCaptureSubject = new ArgumentCaptureSubject(mockClass)

    when: "we invoke our method"
    argumentCaptureSubject.callOtherClass("First")
    argumentCaptureSubject.callOtherClass("Second")
}
```

现在，让我们向Mock添加一个闭包，以捕获每次调用的参数并将其添加到我们的列表中。我们还通过在语句前加上“2 \*”来让Mock验证我们的方法是否被调用了两次：

```groovy
then: "our method was called twice and captured the argument"
2 * mockClass.catchMe(_ as String) >> { arguments -> capturedStrings.add(arguments[0]) }
```

最后，让我们断言我们按照正确的顺序捕获了两个参数：

```groovy
and: "we captured the list and it contains an entry for both of our input values"
capturedStrings[0] == "First"
capturedStrings[1] == "Second"
```

当我们不关心顺序时，我们可以使用List的contains方法：

```groovy
capturedStrings.contains("First")
```

## 8. 使用多个Then块

有时，我们想要断言使用相同方法但参数不同的一系列调用，但不需要捕获它们。Spock允许在同一个then块中以任何顺序进行验证，因此我们以什么顺序编写它们并不重要。但是，我们可以通过添加多个then块来强制执行顺序。

**Spock验证一个then块中的断言在下一个then块中的断言之前得到满足**。

因此，让我们添加两个then块来验证我们的方法是否按照正确的顺序使用正确的参数调用：

```groovy
def "given a Mock when we invoke our subject twice then our Mock verifies the sequence"() {
    given: "a mock"
    ArgumentCaptureDependency mockClass = Mock()

    and: "our subject"
    @Subject argumentCaptureSubject = new ArgumentCaptureSubject(mockClass)

    when: "we invoke our method"
    argumentCaptureSubject.callOtherClass("First")
    argumentCaptureSubject.callOtherClass("Second")

    then: "we invoked our Mock with 'First' the first time"
    1 * mockClass.catchMe( "First")

    then: "we invoked our Mock with 'Second' the next time"
    1 * mockClass.catchMe( "Second")
}
```

当我们的调用以错误的顺序发生时，例如当我们首先调用callOtherClass("Second")时，Spock会向我们提供一条有用的消息：

```text
Wrong invocation order for:

1 * mockClass.catchMe( "First")   (1 invocation)

Last invocation: mockClass.catchMe('First')

Previous invocation:
	mockClass.catchMe('Second')
```

## 9. 总结

在本教程中，我们学习了如何使用Spock的Stub、Mock和使用Closure的Spies捕获方法参数。接下来，我们学习了如何在调用实际方法之前使用Spy更改捕获的参数。我们还学习了如何在多次调用我们的方法时收集参数。最后，作为捕获参数的替代方法，我们学习了如何使用多个then块来检查我们的调用是否按正确的顺序进行。