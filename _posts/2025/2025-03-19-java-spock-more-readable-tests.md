---
layout: post
title:  使用Spock的数据管道和表提高测试覆盖率和可读性
category: unittest
copyright: unittest
excerpt: Spock
---

## 1. 简介

编写测试时，我们经常对对象的各种属性进行断言比较。[Spock](https://www.baeldung.com/groovy-spock)有一些有用的语言功能，可以帮助我们在比较对象时消除重复。

在本教程中，我们将学习如何使用Spock的辅助方法with()和verifyAll()重构我们的测试，以使我们的测试更具可读性。

## 2. 设置

首先，让我们创建一个具有名称、当前余额和透支限额的Account类：

```java
public class Account {
    private String accountName;
    private BigDecimal currentBalance;
    private long overdraftLimit;

    // getters and setters
}
```

## 3. 我们的基本测试

现在让我们创建一个AccountTest规范，其中包含一个测试，该测试用于验证帐户名称、当前余额和透支限额的Getter和Setter。我们将Account声明为我们的@Subject，并为每个测试创建一个新帐户：

```groovy
class AccountTest extends Specification {
    @Subject
    Account account = new Account()

    def "given an account when we set its attributes then we get the same values back"() {
        when: "we set attributes on our account"
        account.setAccountName("My Account")
        account.setCurrentBalance(BigDecimal.TEN)
        account.setOverdraftLimit(0)

        then: "the values we retrieve match the ones that we set"
        account.getAccountName() == "My Account"
        account.getCurrentBalance() == BigDecimal.TEN
        account.getOverdraftLimit() == 0
    }
}
```

在这里，我们在期望中重复引用了account对象。我们需要比较的属性越多，我们需要的重复次数就越多。

## 4. 重构选项

让我们重构代码来删除一些重复。

### 4.1 断言陷阱

我们最初的尝试可能是将Getter比较提取到一个单独的方法中：

```groovy
void verifyAccount(Account accountToVerify) {
    accountToVerify.getAccountName() == "My Account"
    accountToVerify.getCurrentBalance() == BigDecimal.TEN
    accountToVerify.getOverdraftLimit() == 0
}
```

但是，这样做有一个问题。让我们通过创建一个verifyAccountRefactoringTrap方法来比较我们的帐户，但使用来自其他人帐户的值来查看它是什么：

```groovy
void verifyAccountRefactoringTrap(Account accountToVerify) {
    accountToVerify.getAccountName() == "Someone else's account"
    accountToVerify.getCurrentBalance() == BigDecimal.ZERO
    accountToVerify.getOverdraftLimit() == 9999
}
```

现在，让我们在then块中调用我们的方法：

```groovy
then: "the values we retrieve match the ones that we set"
verifyAccountRefactoringTrap(account)
```

当我们运行测试时，即使值不匹配，它也会通过！那么，发生了什么事？

尽管代码看起来像是在比较值，但我们的verifyAccountRefactoringTrap方法包含布尔表达式但没有断言！

**Spock的隐式断言仅当我们在测试方法中使用它们时才会发生，而不是在被调用方法的内部**。

那么，我们该如何解决这个问题呢？

### 4.2 方法内部断言

**当我们将比较移到单独的方法中时，Spock不再能够自动强制执行它们，因此我们必须自己添加assert关键字**。

因此，让我们创建一个verifyAccountAsserted方法来断言我们的原始帐户值：

```groovy
void verifyAccountAsserted(Account accountToVerify) {
    assert accountToVerify.getAccountName() == "My Account"
    assert accountToVerify.getCurrentBalance() == BigDecimal.TEN
    assert accountToVerify.getOverdraftLimit() == 0
}
```

让我们在then块中调用verifyAccountAsserted方法：

```groovy
then: "the values we retrieve match the ones that we set"
verifyAccountAsserted(account)
```

当我们运行测试时，它仍然会通过，而当我们改变其中一个断言值时，它就会像我们预期的那样失败。

### 4.3 返回布尔值

确保我们的方法被视为断言的另一种方法是返回布尔结果。让我们将断言与布尔和运算符结合起来，知道Groovy将返回方法中最后执行的语句的结果：

```groovy
boolean matchesAccount(Account accountToVerify) {
    accountToVerify.getAccountName() == "My Account"
        && accountToVerify.getCurrentBalance() == BigDecimal.TEN
        && accountToVerify.getOverdraftLimit() == 0
}
```

当所有条件都符合时，我们的测试通过；当一个或多个条件不符合时，我们的测试失败，但这也有一个缺点。当我们的测试失败时，我们不知道三个条件中的哪一个没有满足。

虽然我们可以使用这些方法来重构我们的测试，但是Spock的辅助方法为我们提供了更好的方法。

## 5. 辅助方法

Spock提供了两个辅助方法“with”和“verifyAll”，可帮助我们更优雅地解决问题！两者的工作方式大致相同，因此让我们从学习如何使用with开始。

### 5.1 with()辅助方法

Spock的with()辅助方法接收一个对象和该对象的闭包。**当我们将对象传递给辅助方法with()时，对象的属性和方法将添加到我们的上下文中。这意味着当我们在with()闭包的范围内时，我们不需要在其前面加上对象名称**。

因此，一个选择是重构我们的方法使用with：

```groovy
void verifyAccountWith(Account accountToVerify) {
    with(accountToVerify) {
        getAccountName() == "My Account"
        getCurrentBalance() == BigDecimal.TEN
        getOverdraftLimit() == 0
    }
}
```

**请注意，强力断言适用于Spock的辅助方法内部，因此即使它是在单独的方法中，我们也不需要任何断言**！

**通常，我们甚至不需要单独的方法，所以让我们在测试的期望中直接使用with()**：

```groovy
then: "the values we retrieve match the ones that we set"
with(account) {
    getAccountName() == "My Account"
    getCurrentBalance() == BigDecimal.TEN
    getOverdraftLimit() == 0
}
```

现在让我们从断言中调用我们的方法：

```groovy
then: "the values we retrieve match the ones that we set"
verifyAccountWith(account)
```

**我们的with()方法通过了测试，但是在第一次不匹配的比较中测试失败**。

### 5.2 对Mock使用with

我们还可以在断言交互时使用with，让我们创建一个Mock帐户并调用它的一些Setter：

```groovy
given: 'a mock account'
Account mockAccount = Mock()

when: "we invoke its setters"
mockAccount.setAccountName("A Name")
mockAccount.setOverdraftLimit(0)
```

现在让我们验证一下使用mockAccount进行的一些交互：

```groovy
with(mockAccount) {
    1  setAccountName(_ as String)
    1  setOverdraftLimit(_)
}
```

请注意，在验证mockAccount.setAccountName时我们可以省略mockAccount，因为它在我们的with范围内。

### 5.3 verifyAll()辅助方法

有时我们更希望知道运行测试时失败的每个断言。在这种情况下，我们可以使用Spock的verifyAll()辅助方法，就像我们使用with一样。

因此，让我们使用verifyAll()添加一个检查：

```groovy
verifyAll(accountToVerify) {
    getAccountName() == "My Account"
    getCurrentBalance() == BigDecimal.TEN
    getOverdraftLimit() == 0
}
```

**当一个比较失败时，verifyAll()方法不会使测试失败，而是会继续执行并报告verifyAll范围内所有失败的比较**。

### 5.4 嵌套辅助方法

当我们有一个对象内的对象需要比较时，我们可以嵌套我们的辅助方法。

让我们首先创建一个包含街道和城市的地址并将其添加到我们的帐户：

```java
public class Address {
    String street;
    String city;

    // getters and setters
}
```

```java
public class Account {
    private Address address;

    // getter and setter and rest of class
}
```

现在我们有了一个Address类，让我们在测试中创建一个：

```groovy
given: "an address"
Address myAddress = new Address()
def myStreet = "1, The Place"
def myCity = "My City"
myAddress.setStreet(myStreet)
myAddress.setCity(myCity)
```

并将其添加到我们的帐户：

```groovy
when: "we set attributes on our account"
account.setAddress(myAddress)
account.setAccountName("My Account")
```

接下来我们来比较一下我们的地址，当我们不使用辅助方法时，我们最基本的做法是：

```groovy
account.getAddress().getStreet() == myStreet
account.getAddress().getCity() == myCity
```

我们可以通过提取address变量来改进这一点：

```groovy
def address = account.getAddress()
address.getCity() == myCity
address.getStreet() == myStreet
```

但更好的是，让我们使用辅助方法进行更清晰的比较：

```groovy
with(account.getAddress()) {
    getStreet() == myStreet
    getCity() == myCity
}
```

现在我们使用with来比较地址，让我们将其嵌套在帐户比较中。由于with(account)将我们的帐户纳入范围，我们可以将其从account.getAddress()中删除，而改用with(getAddress())：

```groovy
then: "the values we retrieve match the ones that we set"
with(account) {
    getAccountName() == "My Account"
    with(getAddress()) {
        getStreet() == myStreet
        getCity() == myCity
    }
}
```

**由于Groovy可以为我们派生Getter和Setter，因此我们也可以通过名称引用对象的属性**。

因此，让我们通过使用属性名称而不是Getter来让我们的测试更具可读性：

```groovy
with(account) {
    accountName == "My Account"
    with(address) {
        street == myStreet
        city == myCity
    }
}
```

## 6. 它们如何工作？

我们已经了解了with()和verifyAll()如何帮助我们使测试更具可读性，但它们是如何做到这一点的呢？

让我们看一下with()的方法签名：

```groovy
with(Object, Closure)
```

因此我们可以通过传递闭包作为第二个参数来使用with()：

```groovy
with(account, (acct) -> {
    acct.getAccountName() == "My Account"
    acct.getOverdraftLimit() == 0
})
```

**但是Groovy对最后一个参数为闭包的方法有特殊支持，它允许我们在括号外声明闭包**。

因此，让我们使用Groovy更易读且更简洁的形式：

```groovy
with(account) {
    getAccountName() == "My Account"
    getOverdraftLimit() == 0
}
```

请注意，虽然有两个参数，但在我们的测试中，我们只向with()传递了一个参数，即account。

第二个参数Closure是紧跟with之后的花括号内的代码，它传递了第一个参数account来进行操作。

## 7. 总结

在本教程中，我们学习了如何使用Spock的with()和verifyAll()辅助方法来减少在比较对象时测试中的样板代码。我们学习了如何将辅助方法用于简单对象，以及如何在对象更复杂时嵌套使用辅助方法。我们还了解了如何使用Groovy的属性表示法使我们的辅助断言更加清晰，以及如何将辅助方法与Mock一起使用。最后，我们学习了在使用辅助方法时，对于最后一个参数为闭包的方法，我们如何从Groovy的替代、更清晰的语法中获益。