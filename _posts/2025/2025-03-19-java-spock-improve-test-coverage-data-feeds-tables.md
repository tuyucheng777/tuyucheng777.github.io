---
layout: post
title:  使用Spock的数据管道和表提高测试覆盖率和可读性
category: unittest
copyright: unittest
excerpt: Spock
---

## 1. 简介

[Spock](https://www.baeldung.com/groovy-spock)是一个很棒的测试框架，尤其是在提高测试覆盖率方面。

在本教程中，我们将探索Spock的数据管道，以及如何通过向数据管道添加额外数据来提高我们的行和分支代码覆盖率。我们还将研究当数据太大时该怎么办。

## 2. 测试对象

让我们从一种将两个数字相加但略有不同的方法开始。如果第一个或第二个数字是42，则返回42：

```java
public class DataPipesSubject {
    int addWithATwist(final int first, final int second) {
        if (first == 42 || second == 42) {
            return 42;
        }
        return first + second;
    }
}
```

我们想使用各种输入组合来测试这个方法。

让我们看看如何编写和开发一个简单的测试来通过数据管道提供输入。

## 3. 准备数据驱动测试

让我们创建一个针对单个场景进行测试的测试类，然后在其基础上添加数据管道：

首先，让我们使用测试主题创建DataPipesTest类：

```groovy
@Title("Test various ways of using data pipes")
class DataPipesTest extends Specification {
    @Subject
    def dataPipesSubject = new DataPipesSubject()
    // ...
}
```

我们在类中使用了[Spock的@Title注解](https://www.baeldung.com/spock-extensions#10-human-friendly-titles)，为即将进行的测试提供一些额外的背景信息。

我们还用[Spock的@Subject标注](https://www.baeldung.com/spock-extensions#13-subject)了测试的主题。请注意，我们应该小心地从spock.lang而不是从javax.security.auth导入Subject。

尽管不是绝对必要的，但这种语法糖可以帮助我们快速识别正在测试的内容。

现在让我们使用Spock的given/when/then语法，用前两个输入1和2创建一个测试：

```groovy
def "given two numbers when we add them then our result is the sum of the inputs"() {
    given: "some inputs"
    def first = 1
    def second = 2

    and: "an expected result"
    def expectedResult = 3

    when: "we add them together"
    def result = dataPipesSubject.addWithATwist(first, second)

    then: "we get our expected answer"
    result == expectedResult
}
```

为了准备数据管道测试，我们将输入从given/and块移到where块中：

```groovy
def "given a where clause with our inputs when we add them then our result is the sum of the inputs"() {
    when: "we add our inputs together"
    def result = dataPipesSubject.addWithATwist(first, second)

    then: "we get our expected answer"
    result == expectedResult

    where: "we have various inputs"
    first = 1
    second = 2
    expectedResult = 3
}
```

Spock评估where块并隐式地将任何变量作为参数添加到测试中。因此，Spock看到我们的方法声明如下：

```groovy
def "given some declared method parameters when we add our inputs then those types are used"(int first, int second, int expectedResult)
```

**请注意，当我们将数据强制转换为特定类型时，我们将类型和变量声明为方法参数**。

因为我们的测试非常简单，所以我们将when和then块压缩为一个expect块：

```groovy
def "given an expect block to simplify our test when we add our inputs then our result is the sum of the two numbers"() {
    expect: "our addition to get the right result"
    dataPipesSubject.addWithATwist(first, second) == expectedResult

    where: "we have various inputs"
    first = 1
    second = 2
    expectedResult = 3
}
```

现在我们已经简化了测试，我们可以添加第一个数据管道了。

## 4. 什么是数据管道？

**Spock中的数据管道是一种将不同组合的数据输入到测试中的方式**，当我们需要考虑多个场景时，这有助于保持测试代码的可读性。

**管道可以是任何Iterable-如果它实现了Iterable接口，我们甚至可以创建自己的**。

### 4.1 简单数据管道

由于数组是Iterable，我们首先将单个输入转换为数组，然后使用数据管道'<<'将它们输入到我们的测试中：

```groovy
where: "we have various inputs"
first << [1]
second << [2]
expectedResult << [3]
```

我们可以通过向每个数组数据管道添加条目来添加额外的测试用例。

因此，让我们针对场景2 + 2 = 4和3 + 5 = 8向管道添加一些数据：

```groovy
first << [1, 2, 3]
second << [2, 2, 5]
expectedResult << [3, 4, 8]
```

为了让我们的测试更具可读性，我们将第一个和第二个输入组合成一个多变量数组数据管道，暂时将我们的expectedResult分开：

```groovy
where: "we have various inputs"
[first, second] << [
    [1, 2],
    [2, 2],
    [3, 5]
]

and: "an expected result"
expectedResult << [3, 4, 8]
```

由于**我们可以引用已经定义的feed**，因此我们可以用以下内容替换预期结果数据管道：

```groovy
expectedResult = first + second
```

但是让我们将它与输入管道结合起来，因为我们正在测试的方法有一些微妙之处，会破坏简单的加法：

```groovy
[first, second, expectedResult] << [
    [1, 2, 3],
    [2, 2, 4],
    [3, 5, 8]
]
```

### 4.2 Map和方法

当我们想要更多的灵活性并且使用Spock 2.2或更高版本时，我们可以使用Map作为数据管道来提供数据：

```groovy
where: "we have various inputs in the form of a map"
[first, second, expectedResult] << [
    [
        first : 1,
        second: 2,
        expectedResult: 3
    ],
    [
        first : 2,
        second: 2, 
        expectedResult: 4
    ]
]
```

我们还可以通过单独的方法传入数据。

```groovy
[first, second, expectedResult] << dataFeed()
```

让我们看看当我们将Map数据管道移到dataFeed方法中时它是什么样子：

```groovy
def dataFeed() {
    [ 
        [
            first : 1,
            second: 2,
            expectedResult: 3
        ],
        [
            first : 2,
            second: 2,
            expectedResult: 4
        ]
    ]
}
```

虽然这种方法有效，但使用多个输入仍然感觉很笨重。让我们看看Spock的数据表如何改善这种情况。

## 5. 数据表

Spock的[数据表](https://www.baeldung.com/groovy-spock#3-using-datatables-in-spock)格式采用一个或多个数据管道，使其更具视觉吸引力。

让我们重写测试方法中的where块以使用数据表而不是数据管道集合：

```groovy
where: "we have various inputs"
first | second || expectedResult
1     | 2      || 3
2     | 2      || 4
3     | 5      || 8
```

所以现在，每一行都包含特定场景的输入和预期结果，这使得我们的测试场景更容易阅读。

**作为视觉提示和最佳实践，我们使用双“||”将输入与预期结果分开**。

当我们对这三次迭代运行代码覆盖测试时，我们发现并非所有执行行都得到了覆盖。当任一输入为42时，我们的addWithATwist方法会出现特殊情况：

```java
if (first == 42 || second == 42) {
    return 42;
}
```

因此，让我们添加一个场景，其中第一个输入是42，以确保我们的代码执行if语句中的行。我们还添加一个场景，其中第二个输入是42，以确保我们的测试涵盖所有执行分支：

```groovy
42    | 10     || 42
1     | 42     || 42
```

因此，这是我们的最终where块，其中包含提供代码行和分支覆盖范围的迭代：

```groovy
where: "we have various inputs"
first | second || expectedResult
1     | 2      || 3
2     | 2      || 4
3     | 5      || 8
42    | 10     || 42
1     | 42     || 42
```

当我们执行这些测试时，我们的测试运行器会为每次迭代呈现一行：

```text
DataPipesTest
 - use table to supply the inputs
    - use table to supply the inputs [first: 1, second: 2, expectedResult: 3, #0]
    - use table to supply the inputs [first: 2, second: 2, expectedResult: 4, #1]
...
```

## 6. 可读性改进

我们可以使用一些技术来使我们的测试更具可读性。

### 6.1 在方法名中插入变量

**当我们想要更具表现力的测试执行时，我们可以将变量添加到方法名中**。

因此，让我们通过插入表中的列标头变量(以“#”为前缀)来增强测试的方法名，并添加一个场景列：

```groovy
def "given a #scenario case when we add our inputs, #first and #second, then we get our expected result: #expectedResult"() {
    expect: "our addition to get the right result"
    dataPipesSubject.addWithATwist(first, second) == expectedResult

    where: "we have various inputs"
    scenario       | first | second || expectedResult
    "simple"       | 1     | 2      || 3
    "double 2"     | 2     | 2      || 4
    "special case" | 42    | 10     || 42
}
```

现在，当我们运行测试时，我们的测试运行器会将输出呈现得更具表现力：

```text
DataPipesTest
- given a #scenario case when we add our inputs, #first and #second, then we get our expected result: #expectedResult
  - given a simple case when we add our inputs, 1 and 2, then we get our expected result: 3
  - given a double 2 case when we add our inputs, 2 and 2, then we get our expected result: 4
...
```

当我们使用此方法但错误地输入数据管道名称时，Spock将测试失败并显示类似如下的消息：

```text
Error in @Unroll, could not find a matching variable for expression: myWrongVariableName
```

与以前一样，**我们可以使用已经声明的feed在表数据中引用feed，即使在同一行中**。

因此，让我们添加一行引用我们的列标头变量：first和second：

```text
scenario              | first | second || expectedResult
"double 2 referenced" | 2     | first  || first + second
```

### 6.2 当表格列太宽时

我们的IDE可能包含对Spock表格的内在支持-我们可以使用IntelliJ的“format code”功能(Ctrl + Alt + L)来为我们对齐表格中的列！了解了这一点，我们可以快速添加数据，而不必担心布局和格式化。

然而，有时，我们表格中的数据项的长度会导致格式化的表格行变得太宽，无法在一行中容纳。通常，当我们的输入中有字符串时，就会出现这种情况。

为了演示这一点，让我们创建一个以字符串作为输入并简单添加感叹号的方法：

```java
String addExclamation(final String first) {
    return first + '!';
}
```

现在让我们创建一个以长字符串作为输入的测试：

```groovy
def "given long strings when our tables our too big then we can use shared or static variables to shorten the table"() {
    expect: "our addition to get the right result"
    dataPipesSubject.addExclamation(longString) == expectedResult

    where: "we have various inputs"
    longString                                                                                                  || expectedResult
    'When we have a very long string we can use a static or @Shared variable to make our tables easier to read' || 'When we have a very long string we can use a static or @Shared variable to make our tables easier to read!'
}
```

现在，让我们通过将字符串替换为静态或@Shared变量来使此表更紧凑。请注意，我们的表不能使用测试中声明的变量-**表只能使用静态、@Shared或计算值**。

因此，让我们声明一个静态和共享变量，并在表中使用它们：

```groovy
static def STATIC_VARIABLE = 'When we have a very long string we can use a static variable'
@Shared
def SHARED_VARIABLE = 'When we have a very long string we can annotate our variable with @Shared'
...
scenario         | longString      || expectedResult
'use of static'  | STATIC_VARIABLE || "$STATIC_VARIABLE!"
'use of @Shared' | SHARED_VARIABLE || "$SHARED_VARIABLE!"
```

现在我们的表格更加紧凑了！我们还使用了[Groovy的字符串插值](https://www.baeldung.com/groovy-strings#string-interpolation)来扩展预期结果中双引号字符串中的变量，以显示它如何提高可读性。请注意，仅使用\$就足以进行简单的变量替换，但对于更复杂的情况，我们需要将表达式括在花括号\${}中。

使大表格更具可读性的另一种方法是**使用两个或多个下划线'__'将表格分成多个部分**：

```groovy
where: "we have various inputs"
first | second
1     | 2
2     | 3
3     | 5
__
expectedResult | _
3              | _
5              | _
8              | _
```

当然，我们需要在分割表中拥有相同的行数。

**Spock表必须至少有两列，但是在我们拆分表之后，expectedResult将会单独存在，因此我们添加了一个空的“_”列来满足此要求**。

### 6.3 表格分隔符替代方案

有时我们可能不想使用'|'作为分隔符，在这种情况下，我们可以使用';'代替：

```groovy
first ; second ;; expectedResult
1     ; 2      ;; 3
2     ; 3      ;; 5
3     ; 5      ;; 8
```

但是我们不能在同一个表中混合搭配“|”和“;”列分隔符。

## 7. 总结

在本文中，我们学习了如何在where块中使用Spock的数据馈送。我们了解了数据表如何以更美观的方式呈现数据馈送，以及如何通过简单地向数据表添加一行数据来提高测试覆盖率。我们还探索了一些使数据更具可读性的方法，尤其是在处理大数据值或表格太大时。