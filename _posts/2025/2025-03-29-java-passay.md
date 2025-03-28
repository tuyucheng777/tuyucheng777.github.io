---
layout: post
title:  Passay指南
category: libraries
copyright: libraries
excerpt: Passay
---

## 1. 简介

如今，大多数Web应用程序都有自己的密码策略-简单地说，就是为了强制用户创建难以破解的密码。

要生成这样的密码或验证它们，我们可以使用[Passay库](http://www.passay.org/)。

## 2. Maven依赖

如果我们想在项目中使用Passay库，则需要在我们的pom.xml中添加以下依赖：

```xml
<dependency>
    <groupId>org.passay</groupId>
    <artifactId>passay</artifactId>
    <version>1.3.1</version>
</dependency>
```

我们可以在[这里](https://mvnrepository.com/artifact/org.passay/passay)找到它。

## 3. 密码验证

密码验证是Passay库提供的两个主要功能之一，它简单又直观，让我们来探讨它。

### 3.1 PasswordData

为了验证密码，我们应该使用PasswordData，**它是验证所需信息的容器**，它可以存储以下数据：

- 密码
- 用户名
- 密码参考列表
- 起源

密码和用户名属性不言而喻，Passay库为我们提供了HistoricalReference和SourceReference，我们可以将其添加到密码引用列表中。

我们可以使用来源字段来保存有关密码是由用户生成还是定义的信息。

### 3.2 PasswordValidator

**我们应该知道，我们需要PasswordData和PasswordValidator对象来开始验证密码**。我们已经讨论过PasswordData，现在让我们创建PasswordValidator。

首先，我们应该定义一组密码验证规则，我们必须在创建PasswordValidator对象时将它们传递给构造函数：

```java
PasswordValidator passwordValidator = new PasswordValidator(new LengthRule(5));
```

有两种方法可以将密码传递给PasswordData对象，我们将其传递给构造函数或Setter方法：

```java
PasswordData passwordData = new PasswordData("1234");

PasswordData passwordData2 = new PasswordData();
passwordData.setPassword("1234");
```

我们可以通过调用PasswordValidator上的validate()方法来验证我们的密码：

```java
RuleResult validate = passwordValidator.validate(passwordData);
```

结果，我们将得到一个RuleResult对象。

### 3.3 RuleResult

RuleResult保存了有关验证过程的有趣信息，它是validate()方法的结果。

首先，它可以告诉我们密码是否有效：

```java
Assert.assertEquals(false, validate.isValid());
```

**此外，我们可以了解密码无效时会返回哪些错误**。错误代码和验证描述保存在RuleResultDetail中：

```java
RuleResultDetail ruleResultDetail = validate.getDetails().get(0);
Assert.assertEquals("TOO_SHORT", ruleResultDetail.getErrorCode());
Assert.assertEquals(5, ruleResultDetail.getParameters().get("minimumLength"));
Assert.assertEquals(5, ruleResultDetail.getParameters().get("maximumLength"));
```

最后，我们可以使用RuleResultMetadata探索密码验证的元数据：

```java
Integer lengthCount = validate
    .getMetadata()
    .getCounts()
    .get(RuleResultMetadata.CountCategory.Length);
Assert.assertEquals(Integer.valueOf(4), lengthCount);
```

## 4. 密码生成

除了验证之外，[Passay](https://mvnrepository.com/artifact/org.passay/passay)库还允许我们生成密码，**我们可以提供生成器应使用的规则**。

要生成密码，我们需要一个PasswordGenerator对象。一旦我们有了它，我们调用generatePassword()方法并传递CharacterRules列表。以下是示例代码：

```java
CharacterRule digits = new CharacterRule(EnglishCharacterData.Digit);

PasswordGenerator passwordGenerator = new PasswordGenerator();
String password = passwordGenerator.generatePassword(10, digits);

Assert.assertTrue(password.length() == 10);
Assert.assertTrue(containsOnlyCharactersFromSet(password, "0123456789"));
```

我们应该知道，我们需要一个CharacterData对象来创建CharacterRule。**另一个有趣的事实是，该库为我们提供了EnglishCharacterData**，它是五组字符的枚举：

- 数字
- 小写英文字母
- 大写英文字母
- 小写字母和大写字母的组合
- 特殊字符

**但是，没有什么可以阻止我们定义自己的字符集**，这就像实现CharacterData接口一样简单，让我们看看如何做到这一点：

```java
CharacterRule specialCharacterRule = new CharacterRule(new CharacterData() {
    @Override
    public String getErrorCode() {
        return "SAMPLE_ERROR_CODE";
    }

    @Override
    public String getCharacters() {
        return "ABCxyz123!@#";
    }
});

PasswordGenerator passwordGenerator = new PasswordGenerator();
String password = passwordGenerator.generatePassword(10, specialCharacterRule);

Assert.assertTrue(containsOnlyCharactersFromSet(password, "ABCxyz123!@#"));
```

## 5. 正向匹配规则

我们已经了解了如何生成和验证密码，为此，我们需要定义一组规则。**因此，我们应该知道Passay中有两种类型的规则：正向匹配规则和负向匹配规则**。

首先，让我们了解一下正向规则是什么以及我们如何使用它们。

正向匹配规则接受包含提供的字符、正则表达式或符合某些限制的密码。

有6条正向匹配规则：

- AllowedCharacterRule：定义密码必须包含的所有字符
- AllowedRegexRule：定义密码必须匹配的正则表达式
- CharacterRule：定义密码中应包含的字符集和最少字符数
- LengthRule：定义密码的最小长度
- CharacterCharacteristicsRule：检查密码是否满足N个定义的规则
- LengthComplexityRule：允许我们针对不同的密码长度定义不同的规则

### 5.1 简单的正向匹配规则

现在，我们将介绍所有具有简单配置的规则，**它们定义一组合法字符或模式或可接受的密码长度**。

以下是所讨论规则的一个简短示例：

```java
PasswordValidator passwordValidator = new PasswordValidator(
    new AllowedCharacterRule(new char[] { 'a', 'b', 'c' }), 
    new CharacterRule(EnglishCharacterData.LowerCase, 5), 
    new LengthRule(8, 10)
);

RuleResult validate = passwordValidator.validate(new PasswordData("12abc"));

assertFalse(validate.isValid());
assertEquals(
    "ALLOWED_CHAR:{illegalCharacter=1, matchBehavior=contains}", 
    getDetail(validate, 0));
assertEquals(
    "ALLOWED_CHAR:{illegalCharacter=2, matchBehavior=contains}", 
    getDetail(validate, 1));
assertEquals(
    "TOO_SHORT:{minimumLength=8, maximumLength=10}", 
    getDetail(validate, 4));
```

**我们可以看到，每条规则都明确说明了密码是否有效**。有通知说密码太短，并且有两个非法字符。我们还可以注意到密码与提供的正则表达式不匹配。

而且，我们得知它包含的小写字母不足。

### 5.2 CharacterCharacterisitcsRule

CharacterCharacterisitcsRule比之前介绍的规则更复杂，**要创建CharcterCharacterisitcsRule对象，我们需要提供一个CharacterRule列表**。此外，我们还必须设置密码必须匹配多少个CharacterRule，我们可以这样做：

```java
CharacterCharacteristicsRule characterCharacteristicsRule = new CharacterCharacteristicsRule(
    3, 
    new CharacterRule(EnglishCharacterData.LowerCase, 5), 
    new CharacterRule(EnglishCharacterData.UpperCase, 5), 
    new CharacterRule(EnglishCharacterData.Digit),
    new CharacterRule(EnglishCharacterData.Special)
);
```

所呈现的CharacterCharacteristicsRule要求密码包含所提供的四个规则中的三个。

### 5.3 LengthComplexityRule

另一方面，Passay库为我们提供了LengthComplexityRule，**它允许我们定义哪些规则应该应用于哪个长度的密码**。与CharacterCharacteristicsRule相比，它们允许我们使用所有类型的规则-而不仅仅是CharacterRule。

我们来分析一下这个例子：

```java
LengthComplexityRule lengthComplexityRule = new LengthComplexityRule();
lengthComplexityRule.addRules("[1,5]", new CharacterRule(EnglishCharacterData.LowerCase, 5));
lengthComplexityRule.addRules("[6,10]", 
    new AllowedCharacterRule(new char[] { 'a', 'b', 'c', 'd' }));
```

我们可以看到，对于包含一到五个字符的密码，我们应用CharacterRule。但是对于包含六到十个字符的密码，我们希望密码符合AllowedCharacterRule。

## 6. 负向匹配规则

**与正向匹配规则不同，负向匹配规则拒绝包含提供的字符、正则表达式、条目等的密码**。

让我们来看看否定匹配规则是什么：

- IllegalCharacterRule：定义密码不得包含的所有字符
- IllegalRegexRule：定义不得匹配的正则表达式
- IllegalSequenceRule：检查密码是否包含非法字符序列
- NumberRangeRule：定义密码中不能包含的数字范围
- WhitespaceRule：检查密码是否包含空格
- DictionaryRule：检查密码是否与任何字典记录相等
- DictionarySubstringRule：检查密码是否包含任何字典记录
- HistoryRule：检查密码是否包含任何历史密码引用
- DigestHistoryRule：检查密码是否包含任何摘要历史密码引用
- SourceRule：检查密码是否包含任何源密码引用
- DigestSourceRule：检查密码是否包含任何摘要源密码引用
- UsernameRule：检查密码是否包含用户名
- RepeatCharacterRegexRule：检查密码是否包含重复的ASCII字符

### 6.1 简单的负向匹配规则

首先，我们将了解如何使用简单规则，例如IllegalCharacterRule、IllegalRegexRule等。这是一个简短的示例：

```java
PasswordValidator passwordValidator = new PasswordValidator(
    new IllegalCharacterRule(new char[] { 'a' }), 
    new NumberRangeRule(1, 10), 
    new WhitespaceRule()
);

RuleResult validate = passwordValidator.validate(new PasswordData("abcd22 "));

assertFalse(validate.isValid());
assertEquals(
    "ILLEGAL_CHAR:{illegalCharacter=a, matchBehavior=contains}", 
    getDetail(validate, 0));
assertEquals(
    "ILLEGAL_NUMBER_RANGE:{number=2, matchBehavior=contains}", 
    getDetail(validate, 4));
assertEquals(
    "ILLEGAL_WHITESPACE:{whitespaceCharacter= , matchBehavior=contains}", 
    getDetail(validate, 5));
```

该示例向我们展示了所述规则的工作原理，与正向匹配规则类似，它们为我们提供了有关验证的完整反馈。

### 6.2 字典规则

如果我们想检查密码是否不等于提供的单词，该怎么办？

因此，Passay库为我们提供了出色的工具，让我们来探索一下DictionaryRule和DictionarySubstringRule：

```java
WordListDictionary wordListDictionary = new WordListDictionary(new ArrayWordList(new String[] { "bar", "foobar" }));

DictionaryRule dictionaryRule = new DictionaryRule(wordListDictionary);
DictionarySubstringRule dictionarySubstringRule = new DictionarySubstringRule(wordListDictionary);
```

**我们可以看到，字典规则使我们能够提供禁用词列表**。如果我们有最常见或最容易破解的密码列表，这将大有裨益。因此，禁止用户使用它们是合理的。

在现实生活中，我们肯定会从文本文件或数据库中加载单词列表。在这种情况下，我们可以使用WordLists。它有三个重载方法，它们接收Reader数组并创建ArrayWordList。

### 6.3 HistoryRule和SourceRule

此外，Passay库还提供了HistoryRule和SourceRule，它们可以根据历史密码或来自各种来源的文本内容来验证密码。

我们来看一个例子：

```java
SourceRule sourceRule = new SourceRule();
HistoryRule historyRule = new HistoryRule();

PasswordData passwordData = new PasswordData("123");
passwordData.setPasswordReferences(
    new PasswordData.SourceReference("source", "password"), 
    new PasswordData.HistoricalReference("12345")
);

PasswordValidator passwordValidator = new PasswordValidator(historyRule, sourceRule);
```

HistoryRules帮助我们检查密码是否曾经被使用过，由于这种做法不安全，我们不希望用户使用旧密码。

另一方面，SourceRule允许我们检查密码是否与SourceReferences中提供的密码不同。我们可以避免在不同的系统或应用程序中使用相同密码的风险。

值得一提的是，还有DigestSourceRule和DigestHistoryRule等规则。我们将在下一段中介绍它们。

### 6.4 摘要规则

Passay库中有两个摘要规则：DigestHistoryRule和DigestSourceRule。**摘要规则旨在处理以摘要或哈希形式存储的密码**，因此，要定义它们，我们需要提供一个EncodingHashBean对象。

让我们看看它是如何完成的：

```java
List<PasswordData.Reference> historicalReferences = Arrays.asList(
    new PasswordData.HistoricalReference(
        "SHA256",
        "2e4551de804e27aacf20f9df5be3e8cd384ed64488b21ab079fb58e8c90068ab"
));

EncodingHashBean encodingHashBean = new EncodingHashBean(
    new CodecSpec("Base64"), 
    new DigestSpec("SHA256"), 
    1, 
    false
);
```

这次我们通过标签和编码密码创建HistoricalReference到构造函数。之后，我们用适当的Codec和摘要算法实例化了EncodingHashBean。

此外，我们可以指定迭代次数以及算法是否加盐。

一旦我们有了编码Bean，我们就可以验证我们的摘要密码：

```java
PasswordData passwordData = new PasswordData("example!");
passwordData.setPasswordReferences(historicalReferences);

PasswordValidator passwordValidator = new PasswordValidator(new DigestHistoryRule(encodingHashBean));

RuleResult validate = passwordValidator.validate(passwordData);

Assert.assertTrue(validate.isValid());
```

我们可以在[Cryptacular库](http://www.cryptacular.org/about.html)网页上了解有关EncodingHashinBean的更多信息。

### 6.5 RepeatCharacterRegexRule

另一个有趣的验证规则是RepeatCharacterRegexRule，**我们可以使用它来检查密码是否包含重复的ASCII字符**。

以下是示例代码：

```java
PasswordValidator passwordValidator = new PasswordValidator(new RepeatCharacterRegexRule(3));

RuleResult validate = passwordValidator.validate(new PasswordData("aaabbb"));

assertFalse(validate.isValid());
assertEquals("ILLEGAL_MATCH:{match=aaa, pattern=([^\\x00-\\x1F])\\1{2}}", getDetail(validate, 0));
```

### 6.6 UsernameRule

本章我们要讨论的最后一条规则是UsernameRule，**它使我们能够禁止在密码中使用用户名**。 

正如我们之前所了解的，我们应该将用户名存储在PasswordData中：

```java
PasswordValidator passwordValidator = new PasswordValidator(new UsernameRule());

PasswordData passwordData = new PasswordData("testuser1234");
passwordData.setUsername("testuser");

RuleResult validate = passwordValidator.validate(passwordData);

assertFalse(validate.isValid());
assertEquals("ILLEGAL_USERNAME:{username=testuser, matchBehavior=contains}", getDetail(validate, 0));
```

## 7. 定制信息

Passay库允许我们自定义验证规则返回的消息。**首先，我们应该定义消息并为其分配错误代码**。

我们可以将它们放入一个简单的文件中，让我们看看它有多简单：

```text
TOO_LONG=Password must not have more characters than %2$s.
TOO_SHORT=Password must not contain less characters than %2$s.
```

一旦我们有了消息，我们就必须加载该文件。最后，我们可以将其传递给PasswordValidator对象。

以下是示例代码：

```java
URL resource = this.getClass().getClassLoader().getResource("messages.properties");
Properties props = new Properties();
props.load(new FileInputStream(resource.getPath()));

MessageResolver resolver = new PropertiesMessageResolver(props);
```

我们看到，我们加载了message.properties文件并将其传递给Properties对象。然后，我们可以使用Properties对象来创建PropertiesMessageResolver。

让我们看一下如何使用消息解析器的示例：

```java
PasswordValidator validator = new PasswordValidator(
    resolver, 
    new LengthRule(8, 16), 
    new WhitespaceRule()
);

RuleResult tooShort = validator.validate(new PasswordData("XXXX"));
RuleResult tooLong = validator.validate(new PasswordData("ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ"));

Assert.assertEquals(
    "Password must not contain less characters than 16.", 
    validator.getMessages(tooShort).get(0));
Assert.assertEquals(
    "Password must not have more characters than 16.", 
    validator.getMessages(tooLong).get(0));
```

该示例清楚地表明，我们可以使用配备消息解析器的验证器翻译所有错误代码。

## 8. 总结

在本教程中，我们学习了如何使用Passay库。我们分析了几个示例，说明如何轻松使用该库进行密码验证，提供的规则涵盖了确保密码安全的大多数常用方法。

但我们要记住，Passay库本身并不能保证我们的密码安全。首先，我们应该了解什么是通用规则，然后使用该库来实现它们。