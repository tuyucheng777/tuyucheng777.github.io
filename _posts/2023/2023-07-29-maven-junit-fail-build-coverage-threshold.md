---
layout: post
title:  如果JUnit覆盖率低于特定阈值，则使Maven构建失败
category: coverage
copyright: coverage
excerpt: Jacoco
---

## 1. 概述

在本教程中，我们将了解当[JaCoCo](https://www.baeldung.com/jacoco-report-exclude)代码覆盖率低于特定阈值时如何导致[Maven](https://www.baeldung.com/maven)构建失败。我们将从没有阈值的普通JaCoCo插件开始，然后我们将向现有的[JaCoCo](https://www.baeldung.com/jacoco)插件添加一个新的execution，重点检查覆盖范围。

在这里，我们将讨论这个新execution的一些显著元素。然后，我们将扩展ProductService的一个简单示例，以查看添加BRANCH和INSTRUCTION覆盖率规则的效果。我们将看到构建因特定规则而失败。最后，我们将总结与JaCoCo一起执行质量控制规则的潜在好处。

## 2. JaCoCo Maven插件

**让我们首先使用简单形式的JaCoCo插件**。这意味着当我们执行mvn clean install时，我们将使用它来计算代码覆盖率并生成报告：

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>${jacoco.version}</version>
    <configuration>
        <excludes>
            <exclude>cn/tuyucheng/taketoday/**/ExcludedPOJO.class</exclude>
            <exclude>cn/tuyucheng/taketoday/**/*DTO.*</exclude>
            <exclude>**/config/*</exclude>
        </excludes>
    </configuration>
    <executions>
        <execution>
            <id>jacoco-initialize</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-site</id>
            <phase>package</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 3. 设置示例

接下来，**我们以两个简单的服务为例，即CustomerService和ProductService**。让我们在ProductService中添加一个getSalePrice()方法，该方法有两个分支，一个用于flag设置为true的情况，另一个用于flag设置为false的情况：

```java
public double getSalePrice(double originalPrice, boolean flag) {
    double discount;
    if (flag) {
        discount = originalPrice - originalPrice * DISCOUNT;
    } else {
        discount = originalPrice;
    }
    return discount;
}
```

让我们编写两个单独的测试来覆盖布尔值true和false的两个条件：

```java
@Test
public void givenOriginalPrice_whenGetSalePriceWithFlagTrue_thenReturnsDiscountedPrice() {
    ProductService productService = new ProductService();
    double salePrice = productService.getSalePrice(100, true);
    assertEquals(salePrice, 75);
}

@Test
public void givenOriginalPrice_whenGetSalePriceWithFlagFalse_thenReturnsDiscountedPrice() {
    ProductService productService = new ProductService();
    double salePrice = productService.getSalePrice(100, false);
    assertEquals(salePrice, 100);
}
```

类似地，CustomerService包含一个简单的方法getCustomerName()：

```java
public String getCustomerName() {
    return "some name";
}
```

接下来，这是相应的单元测试，用于在报告中显示此方法的覆盖率：

```java
@Test
public void givenCustomer_whenGetCustomer_thenReturnNewCustomer() {
    CustomerService customerService = new CustomerService();
    assertNotNull(customerService.getCustomerName());
}
```

## 4. 使用JaCoCo设置测试覆盖率基线

通过一组基本的类和测试设置，让我们首先执行mvn clean install并观察JaCoCo报告的结果。这有助于我们设置当前模块测试覆盖率的基线：

[![img](https://www.baeldung.com/wp-content/uploads/2023/07/successful-build-with-jacoco-300x51.png)](https://www.baeldung.com/wp-content/uploads/2023/07/successful-build-with-jacoco.png)

至此，模块构建成功。接下来我们看一下JaCoCo的报告，看看目前的覆盖率：

[![img](https://www.baeldung.com/wp-content/uploads/2023/07/jacoco-report-300x36.png)](https://www.baeldung.com/wp-content/uploads/2023/07/jacoco-report.png)

从报告中我们可以看到，目前的覆盖率是72%，我们已经成功构建了Maven。

## 5. 在JaCoCo插件中添加规则

**现在，我们将规则应用于插件，以便基于指令数量的总体覆盖率不低于70%，并且分支覆盖率不应低于68%**。

为此，我们在插件中定义一个新的execution：

```xml
<execution>
    <id>check-coverage</id>
    <phase>verify</phase>
    <goals>
        <goal>check</goal>
    </goals>
    <configuration>
        <rules>
            <rule>
                <element>BUNDLE</element>
                <limits>
                    <limit>
                        <counter>INSTRUCTION</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.70</minimum>
                    </limit>
                    <limit>
                        <counter>BRANCH</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.68</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</execution>
```

在这里，我们在JaCoCo插件中使用id check-coverage定义了一个新的execution，我们已将其与Maven构建的verify阶段相关联。**与此execution相关的目标设置为check，这意味着此时应执行jacoco:check**。接下来，我们定义了jacoco:check期间需要检查的规则。

接下来，让我们仔细研究<rule\>标签中定义的组件：

### 5.1 JaCoCo中的<element\>

规则的第一个重要部分是<element\>标签，它定义应用规则的覆盖率元素。覆盖率元素代表代码中不同的粒度级别，例如类、包、方法或行。我们可以通过指定适当的覆盖率元素来为该特定级别设置特定的覆盖率标准。

**在这里，我们在BUNDLE级别设置了覆盖率阈值，它代表了正在分析的整个代码库的总体覆盖率。它将来自不同单元(例如类、包或模块)的覆盖率信息合并到单个聚合结果。它通常用于定义最高级别的覆盖率规则或阈值，给出整个项目的代码覆盖率的概述**。

此外，如果我们想在类级别设置覆盖率阈值，我们将使用<element\>CLASS</element\>。

同样，如果我们想在行级别设置阈值，我们可以使用<element\>LINE</element\>。

### 5.2 JaCoCo中的<limit\>

一旦我们在这里定义了粒度(BUNDLE)，我们就可以定义一个或多个限制。本质上，limit封装了有关我们希望如何使用一个或多个计数器强制执行代码覆盖率的信息。

每个<limit\>包含相应的<counter\>、<value\>和<minimum\>或<maximum\>标签，<value\>标签的可能选项包括COVEREDRATIO、COVEREDCOUNT、MISSEDCOUNT和TOTALCOUNT。**在我们的示例中，我们使用COVEREDRATIO表示覆盖率**。

我们将在下一节中更详细地讨论计数器。

### 5.3 JaCoCo中的<counter\>

本质上，这里我们希望基于两个计数器对BUNDLE强制执行阈值；BRANCH计数器和INSTRUCTION计数器。对于这两个计数器，我们使用COVEREDRATIO作为阈值的度量。对于此示例，我们希望指令覆盖率至少为72%，分支覆盖率至少为68%。

**INSTRUCTION计数器对于了解代码执行级别和识别测试可能遗漏的区域非常有价值。BRANCH计数器能够识别决策点覆盖范围，确保它执行true分支和false分支**。

LINE计数器可以洞察代码覆盖的粒度，使我们能够确定覆盖了哪些单独的代码行。

计数器的其他可能选项包括LINE、COMPLEXITY、METHOD和CLASS。这些计数器提供了有关代码覆盖率的不同视角，可用于分析代码库的不同方面。

选择哪些计数器更有用取决于项目的具体目标和要求。但是，INSTRUCTION、BRANCH和LINE计数器通常是最常用的，可以很好地全面了解代码覆盖率。

## 6. 覆盖率降低导致构建失败

要查看我们的构建因新实施的规则而失败，让我们禁用以下测试：

```java
@Test
@Disabled
public void givenOriginalPrice_whenGetSalePriceWithFlagFalse_thenReturnsDiscountedPrice() {//...}
```

现在，让我们再次运行命令mvn clean install：

[![img](https://www.baeldung.com/wp-content/uploads/2023/07/failed-maven-jacoco-rules-300x45.png)](https://www.baeldung.com/wp-content/uploads/2023/07/failed-maven-jacoco-rules.png)

我们发现此时构建失败，参考失败的覆盖率检查：

```text
[ERROR] Failed to execute goal org.jacoco:jacoco-maven-plugin:0.8.8:check (check-coverage) on project testing-libraries-2: Coverage checks have not been met.
```

重要的是，我们观察到jacoco: check目标执行了。它使用Jacoco.exec作为分析源。接下来，它按照我们在规则的element标签中指定的那样对BUNDLE级别执行分析，并发现Maven构建对于INSTRUCTION和BRANCH计数器均失败。

如结果所示，COVEREDRATIO的INSTRUCTION计数器值降至0.68或68%，而最小值为0.70或70%。此外，BRANCH计数器值为0.50或50%，而预期最小值为68%。

为了修复构建，我们需要恢复BUNDLE元素规定的整体覆盖率，使指令覆盖率 >= 70%，分支覆盖率 >= 68%。

## 7. 总结

在本文中，我们了解了如果代码覆盖率低于特定阈值，如何使Maven构建失败。我们研究了JaCoCo Maven插件提供的各种选项，以在不同粒度级别强制执行阈值。该示例重点关注值为COVEREDRATIO的BRANCH和INSTRUCTION计数器。

执行覆盖率规则可确保我们的代码库始终满足最低水平的测试覆盖率。总而言之，这有助于通过识别缺乏足够测试的领域来提高软件的整体质量。通过在不满足覆盖率规则时使构建失败，我们可以防止覆盖率不足的代码在开发生命周期中进一步发展，从而降低发布未经测试或低质量代码的可能性。

与往常一样，示例代码可以在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-libraries-2)上找到。