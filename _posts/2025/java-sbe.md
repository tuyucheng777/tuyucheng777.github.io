---
layout: post
title:  Simple Binary Encoding指南
category: libraries
copyright: libraries
excerpt: Simple Binary Encoding
---

## 1. 简介

效率和性能是现代数据服务的两个重要方面，尤其是当我们传输大量数据时。当然，使用高性能编码减少消息大小是实现这一目标的关键。

然而，内部编码/解码算法可能很麻烦且脆弱，这使得它们难以长期维护。

幸运的是，[Simple Binary Encoding](https://real-logic.github.io/simple-binary-encoding/)可以帮助我们以实用的方式实现和维护量身定制的编码/解码系统。

在本教程中，我们将讨论Simple Binary Encoding(SBE)的用途以及如何将其与代码示例一起使用。

## 2. 什么是SBE？

SBE是一种用于编码/解码消息的二进制表示，用于支持低延迟流式传输。它也是[FIX SBE](https://github.com/FIXTradingCommunity/fix-simple-binary-encoding)标准的参考实现，该标准是金融数据编码的标准。

### 2.1 消息结构

为了保留流式语义，**消息必须能够按顺序读取或写入，并且没有回溯**。这消除了额外的操作(例如取消引用、处理位置指针、管理其他状态等)，并更好地利用硬件支持以保持最高的性能和效率。

让我们看一下SBE中的消息结构：

![](/assets/images/2025/libraries/javasbe01.png)

- 标头：包含消息版本等必填字段，必要时，还可以包含更多字段。
- 根字段：消息的静态字段。它们的块大小是预定义的，无法更改。它们也可以定义为可选的。
- 重复组：这些表示集合类型的表示。组可以包含字段和内部组，以便能够表示更复杂的结构。
- 可变数据字段：这些字段我们无法提前确定其大小，字符串和Blob数据类型就是两个示例。它们将位于消息的末尾。

接下来，我们将了解为什么这种消息结构很重要。

### 2.2 什么时候SBE有用(不有用)？

SBE的强大之处在于其消息结构，它针对顺序访问数据进行了优化。因此，**SBE非常适合固定大小的数据，例如数字、位集、枚举和数组**。

SBE的一个常见用例是财务数据流(主要包含数字和枚举)，SBE是专门为此设计的。

另一方面，**SBE不太适合可变长度的数据类型，如字符串和Blob**，原因是我们很可能不知道确切的数据大小。因此，这将导致在流式传输时进行额外的计算以检测消息中数据的边界。毫不奇怪，如果我们谈论的是毫秒级的延迟，这可能会对我们的业务造成影响。

尽管SBE仍然支持String和Blob数据类型，**但它们始终放在消息的末尾，以将可变长度计算的影响降至最低**。

## 3. 设置库

要使用SBE库，让我们将以下Maven[依赖](https://mvnrepository.com/artifact/uk.co.real-logic/sbe-tool)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>uk.co.real-logic</groupId>
    <artifactId>sbe-all</artifactId>
    <version>1.27.0</version>
</dependency>
```

## 4. 生成Java存根

在生成Java存根之前，显然我们需要形成消息模式，**SBE提供了通过XML定义模式的功能**。

接下来，我们将看到如何为我们的消息定义一个模式，用于传输样本市场交易数据。

### 4.1 创建消息模式

我们的模式将是一个基于FIX协议特殊[XSD](https://www.baeldung.com/java-validate-xml-xsd)的[XML](https://www.baeldung.com/java-xml)文件，它将定义我们的消息格式。

因此，让我们创建模式文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sbe:messageSchema xmlns:sbe="http://fixprotocol.io/2016/sbe"
                   package="cn.tuyucheng.taketoday.sbe.stub" id="1" version="0" semanticVersion="5.2"
                   description="A schema represents stock market data.">
    <types>
        <composite name="messageHeader"
                   description="Message identifiers and length of message root.">
            <type name="blockLength" primitiveType="uint16"/>
            <type name="templateId" primitiveType="uint16"/>
            <type name="schemaId" primitiveType="uint16"/>
            <type name="version" primitiveType="uint16"/>
        </composite>
        <enum name="Market" encodingType="uint8">
            <validValue name="NYSE" description="New York Stock Exchange">0</validValue>
            <validValue name="NASDAQ"
                        description="National Association of Securities Dealers Automated Quotations">1</validValue>
        </enum>
        <type name="Symbol" primitiveType="char" length="4" characterEncoding="ASCII"
              description="Stock symbol"/>
        <composite name="Decimal">
            <type name="mantissa" primitiveType="uint64" minValue="0"/>
            <type name="exponent" primitiveType="int8"/>
        </composite>
        <enum name="Currency" encodingType="uint8">
            <validValue name="USD" description="US Dollar">0</validValue>
            <validValue name="EUR" description="Euro">1</validValue>
        </enum>
        <composite name="Quote"
                   description="A quote represents the price of a stock in a market">
            <ref name="market" type="Market"/>
            <ref name="symbol" type="Symbol"/>
            <ref name="price" type="Decimal"/>
            <ref name="currency" type="Currency"/>
        </composite>
    </types>
    <sbe:message name="TradeData" id="1" description="Represents a quote and amount of trade">
        <field name="quote" id="1" type="Quote"/>
        <field name="amount" id="2" type="uint16"/>
    </sbe:message>
</sbe:messageSchema>
```

如果我们详细查看该模式，我们会注意到它有两个主要部分，\<types\>和\<sbe:message\>，我们首先开始定义<types\>。

作为我们的第一种类型，我们创建messageHeader。它是必需的，并且还具有四个必需字段：

```xml
<composite name="messageHeader" description="Message identifiers and length of message root.">
    <type name="blockLength" primitiveType="uint16"/>
    <type name="templateId" primitiveType="uint16"/>
    <type name="schemaId" primitiveType="uint16"/>
    <type name="version" primitiveType="uint16"/>
</composite>
```

- blockLength：表示为消息中的根字段保留的总空间，它不计算重复字段或可变长度字段，例如字符串和Blob。
- templateId：消息模板的标识符。
- schemaId：消息模式的标识符，模式始终包含一个模板。
- version：我们定义消息时的消息模式的版本。

接下来，我们定义一个枚举，Market：

```xml
<enum name="Market" encodingType="uint8">
    <validValue name="NYSE" description="New York Stock Exchange">0</validValue>
    <validValue name="NASDAQ"
                description="National Association of Securities Dealers Automated Quotations">1</validValue>
</enum>
```

我们的目标是保留一些众所周知的交换名称，我们可以将其硬编码在模式文件中。它们不会经常更改或增加。因此，类型<enum\>非常适合这里。

通过设置encodingType=”uint8″，我们保留8位空间用于在单个消息中存储市场名称。这使我们能够支持2^8 = 256个不同的市场(0到255)-无符号8位整数的大小。

紧接着，我们定义另一种类型，Symbol。这将是一个3或4个字符的字符串，用于标识金融工具，例如AAPL(Apple)、MSFT(Microsoft)等：

```xml
<type name="Symbol" primitiveType="char" length="4" characterEncoding="ASCII" description="Instrument symbol"/>
```

如我们所见，我们用characterEncoding=”ASCII”限制字符-7位，最多128个字符，并且我们设置了length=”4″的上限，不允许超过4个字符。因此，我们可以尽可能地减小大小。

之后，我们需要一个用于价格数据的复合类型。因此，我们创建Decimal类型：

```xml
<composite name="Decimal">
    <type name="mantissa" primitiveType="uint64" minValue="0"/>
    <type name="exponent" primitiveType="int8"/>
</composite>
```

Decimal由两种类型组成：

- mantissa：十进制数的有效数字
- exponent：十进制数的小数位数

例如，mantissa=98765和exponent=-3表示数字98.765。

接下来，与Market非常相似，我们创建另一个<enum\>来表示Currency，其值映射为uint8：

```xml
<enum name="Currency" encodingType="uint8">
    <validValue name="USD" description="US Dollar">0</validValue>
    <validValue name="EUR" description="Euro">1</validValue>
</enum>
```

最后，我们通过组合之前创建的其他类型来定义Quote：

```xml
<composite name="Quote" description="A quote represents the price of an instrument in a market">
    <ref name="market" type="Market"/>
    <ref name="symbol" type="Symbol"/>
    <ref name="price" type="Decimal"/>
    <ref name="currency" type="Currency"/>
</composite>
```

最后，我们完成了类型定义。

但是，我们仍然需要定义一条消息。因此，让我们定义我们的消息TradeData：

```xml
<sbe:message name="TradeData" id="1" description="Represents a quote and amount of trade">
    <field name="quote" id="1" type="Quote"/>
    <field name="amount" id="2" type="uint16"/>
</sbe:message>
```

当然，就类型而言，我们可以从[规范](https://github.com/FIXTradingCommunity/fix-simple-binary-encoding/blob/master/v1-0-STANDARD/doc/publication/Simple_Binary_Encoding_V1.0_with_Errata_November_2020.pdf)中找到更多细节。

在接下来的两节中，我们将讨论如何使用我们的模式来生成最终用于编码/解码消息的Java代码。

### 4.2 使用SbeTool

生成Java存根的直接方法是使用SBE jar文件，这将自动运行实用程序类SbeTool：

```shell
java -jar -Dsbe.output.dir=target/generated-sources/java 
  <local-maven-directory>/repository/uk/co/real-logic/sbe-all/1.26.0/sbe-all-1.26.0.jar 
  src/main/resources/schema.xml
```

要注意的是，必须用我们本地的Maven路径来调整占位符<local-maven-directory\>来运行该命令。

生成成功后，我们将在文件夹target/generated-sources/java中看到生成的Java代码。

### 4.3 将SbeTool与Maven结合使用

使用SbeTool非常简单，但我们可以通过将其集成到Maven中使其更加实用。

因此，让我们将以下Maven插件添加到我们的pom.xml中：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>java</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <includeProjectDependencies>false</includeProjectDependencies>
                <includePluginDependencies>true</includePluginDependencies>
                <mainClass>uk.co.real_logic.sbe.SbeTool</mainClass>
                <systemProperties>
                    <systemProperty>
                        <key>sbe.output.dir</key>
                        <value>${project.build.directory}/generated-sources/java</value>
                    </systemProperty>
                </systemProperties>
                <arguments>
                    <argument>${project.basedir}/src/main/resources/schema.xml</argument>
                </arguments>
                <workingDirectory>${project.build.directory}/generated-sources/java</workingDirectory>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>uk.co.real-logic</groupId>
                    <artifactId>sbe-tool</artifactId>
                    <version>1.27.0</version>
                </dependency>
            </dependencies>
        </plugin>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>build-helper-maven-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <id>add-source</id>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>add-source</goal>
                    </goals>
                    <configuration>
                        <sources>
                            <source>${project.build.directory}/generated-sources/java/</source>
                        </sources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

因此，**典型的Maven clean install命令会自动生成我们的Java存根**。

此外，我们可以随时查看[SBE的Maven文档](https://github.com/real-logic/simple-binary-encoding/wiki/SBE-Tool-Maven)以获取更多配置选项。

## 5. 基本消息传递

我们已经准备好Java存根，让我们看看如何使用它们。

首先，我们需要一些数据来进行测试。因此，我们创建一个类，MarketData：

```java
public class MarketData {

    private int amount;
    private double price;
    private Market market;
    private Currency currency;
    private String symbol;

    // Constructor, getters and setters
}
```

我们应该注意到，我们的MarketData由SBE为我们生成的Market和Currency类组成。

接下来，让我们定义一个MarketData对象，以便稍后在单元测试中使用：

```java
private MarketData marketData;

@BeforeEach
public void setup() {
    marketData = new MarketData(2, 128.99, Market.NYSE, Currency.USD, "IBM");
}
```

由于我们已经准备好了MarketData，我们将在下一节中了解如何将其写入和读入我们的TradeData。

### 5.1 写消息

大多数情况下，我们希望将数据写入ByteBuffer，因此我们在生成的编码器、MessageHeaderEncoder和TradeDataEncoder旁边创建一个具有初始容量的ByteBuffer：

```java
@Test
public void givenMarketData_whenEncode_thenDecodedValuesMatch() {
    // our buffer to write encoded data, initial cap. 128 bytes
    UnsafeBuffer buffer = new UnsafeBuffer(ByteBuffer.allocate(128));
    MessageHeaderEncoder headerEncoder = new MessageHeaderEncoder();
    TradeDataEncoder dataEncoder = new TradeDataEncoder();
    
    // we'll write the rest of the code here
}
```

在写入数据之前，我们需要将价格数据解析为尾数和指数两部分：

```java
BigDecimal priceDecimal = BigDecimal.valueOf(marketData.getPrice());
int priceMantissa = priceDecimal.scaleByPowerOfTen(priceDecimal.scale()).intValue();
int priceExponent = priceDecimal.scale() * -1;
```

我们应该注意到，我们使用[BigDecimal](https://www.baeldung.com/java-bigdecimal-biginteger#bigdecimal)进行此转换。**处理货币值时使用BigDecimal始终是一种好习惯，因为我们不想丢失精度**。

最后，让我们编码并编写我们的TradeData：

```java
TradeDataEncoder encoder = dataEncoder.wrapAndApplyHeader(buffer, 0, headerEncoder);
encoder.amount(marketData.getAmount());
encoder.quote()
    .market(marketData.getMarket())
    .currency(marketData.getCurrency())
    .symbol(marketData.getSymbol())
    .price()
        .mantissa(priceMantissa)
        .exponent((byte) priceExponent);
```

### 5.2 读取消息

要读取消息，我们将使用写入数据的同一缓冲区实例。但是，这次我们需要解码器MessageHeaderDecoder和TradeDataDecoder：

```java
MessageHeaderDecoder headerDecoder = new MessageHeaderDecoder();
TradeDataDecoder dataDecoder = new TradeDataDecoder();
```

接下来，解码我们的TradeData：

```java
dataDecoder.wrapAndApplyHeader(buffer, 0, headerDecoder);
```

同样，我们需要将价格数据从尾数和指数两部分解码，以便将价格数据转换为double值。当然，我们再次使用BigDecimal：

```java
double price = BigDecimal.valueOf(dataDecoder.quote().price().mantissa())
    .scaleByPowerOfTen(dataDecoder.quote().price().exponent())
    .doubleValue();
```

最后，让我们确保解码的值与原始值相匹配：

```java
Assertions.assertEquals(2, dataDecoder.amount());
Assertions.assertEquals("IBM", dataDecoder.quote().symbol());
Assertions.assertEquals(Market.NYSE, dataDecoder.quote().market());
Assertions.assertEquals(Currency.USD, dataDecoder.quote().currency());
Assertions.assertEquals(128.99, price);
```

## 6. 总结

在本文中，我们学习了如何设置SBE，通过XML定义消息结构以及如何使用它在Java中对我们的消息进行编码/解码。