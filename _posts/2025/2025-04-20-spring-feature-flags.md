---
layout: post
title:  Spring功能标志
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本文中，我们将简要定义功能开关，并提出一种在Spring Boot应用程序中实现它们的实用方法。然后，我们将深入研究如何利用不同的Spring Boot功能进行更复杂的迭代。

我们将讨论可能需要功能标记的各种场景，并探讨可能的解决方案，我们将使用比特币矿工示例应用程序来演示。

## 2. 功能标志

**功能标志(有时称为功能切换)是一种机制，它允许我们启用或禁用应用程序的特定功能，而无需修改代码或理想情况下重新部署我们的应用程序**。

根据给定功能标志所需的动态，我们可能需要全局配置它们，每个应用程序实例，或者更细粒度地配置它们-也许每个用户或请求。

与软件工程中的许多情况一样，重要的是尝试使用最直接的方法来解决手头的问题而不增加不必要的复杂性。

**功能开关是一个强大的工具，如果使用得当，可以为系统带来可靠性和稳定性。然而，如果误用或维护不足，它们很快就会成为系统复杂性和麻烦的根源**。

在许多情况下，功能开关可能会派上用场：

- 基于主干的开发和重要特性：在基于主干的开发中，尤其是当我们想要频繁集成时，我们可能会发现自己还没有准备好发布某个功能。功能开关可以派上用场，让我们能够继续发布，直到功能完成，而无需公开我们的更改。
- 特定于环境的配置：我们可能会发现我们需要某些功能来为E2E测试环境重置数据库，或者，我们可能需要在非生产环境中使用与生产环境中不同的安全配置。因此，我们可以利用功能标志在正确的环境中切换正确的设置。
- A/B测试：针对同一个问题发布多个解决方案并衡量其影响是一种引人注目的技术，我们可以使用功能标志来实现。
- 金丝雀发布：在部署新功能时，我们可能会决定逐步进行，先从一小部分用户开始，随着我们验证其行为的正确性，逐渐扩大其采用范围，功能开关可以帮助我们实现这一点。

在以下章节中，我们将尝试提供一种实用的方法来解决上述情况。

让我们分解功能标记的不同策略，从最简单的场景开始，然后进入更细致、更复杂的设置。

## 3. 应用程序级功能标志

如果我们需要解决前两个用例中的任何一个，应用程序级功能标志是一种让事情正常运转的简单方法。

一个简单的功能标志通常涉及一个属性和一些基于该属性值的配置。

### 3.1 使用Spring Profile的功能开关

在Spring中，**我们可以利用[Profile](https://www.baeldung.com/spring-profiles)。Profile让我们能够选择性地配置某些Bean，非常方便。通过一些围绕Profile的构造，我们可以快速创建一个简单而优雅的应用程序级功能开关解决方案**。

假设我们正在构建一个比特币挖矿系统，我们的软件已经投入生产，我们的任务是创建一个实验性的、改进的挖矿算法。

在我们的JavaConfig中，我们可以分析我们的组件：

```java
@Configuration
public class ProfiledMiningConfig {

    @Bean
    @Profile("!experimental-miner")
    public BitcoinMiner defaultMiner() {
        return new DefaultBitcoinMiner();
    }

    @Bean
    @Profile("experimental-miner")
    public BitcoinMiner experimentalMiner() {
        return new ExperimentalBitcoinMiner();
    }
}
```

然后，**使用之前的配置，我们只需添加Profile即可启用新功能**。有[很多方法](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)可以配置我们的应用程序，特别是启用[Profile](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html)。同样，还有一些[测试实用程序](https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#activeprofiles)可以让我们的流程更轻松。

只要我们的系统足够简单，**我们就可以创建一个基于环境的配置来确定要应用哪些功能标志以及要忽略哪些功能标志**。

让我们想象一下，我们有一个基于卡片而不是表格的新用户界面，以及之前的实验矿工。

我们希望在验收环境(UAT)中启用这两项功能，我们可以在application.yml文件中创建以下Profile组：

```yaml
spring:
    profiles:
        group:
            uat: experimental-miner,ui-cards
```

有了上述属性，我们只需在UAT环境中启用UAT Profile即可获得所需的功能集。当然，我们也可以在项目中添加application-uat.yml文件，以包含环境设置的其他属性。

在我们的例子中，我们希望uat Profile还包括experiment-miner和ui-cards。

注意：如果我们使用的是Spring Boot 2.4.0之前的版本，则需要在UAT Profile专用文档中使用spring.profiles.include属性来配置其他Profile。与spring.profiles.active相比，前者允许我们以附加的方式添加Profile。

### 3.2 使用自定义属性的功能开关

Profile是一种简单易用的好方法，但是，我们可能需要Profile来实现其他目的。或者，我们可能想要构建一个更结构化的功能开关基础架构。

对于这些情况，自定义属性可能是一个理想的选择。

**让我们利用@ConditionalOnProperty和我们的命名空间重写前面的示例**：

```java
@Configuration
public class CustomPropsMiningConfig {

    @Bean
    @ConditionalOnProperty(
            name = "features.miner.experimental",
            matchIfMissing = true)
    public BitcoinMiner defaultMiner() {
        return new DefaultBitcoinMiner();
    }

    @Bean
    @ConditionalOnProperty(
            name = "features.miner.experimental")
    public BitcoinMiner experimentalMiner() {
        return new ExperimentalBitcoinMiner();
    }
}
```

**前面的示例建立在Spring Boot的条件配置之上，并根据属性设置为true还是false(或完全省略)来配置一个或另一个组件**。

结果与3.1中的结果非常相似，但现在我们有了自己的命名空间。有了命名空间，我们就可以创建有意义的YAML/properties文件：

```yaml
#[...] Some Spring config

features:
    miner:
        experimental: true
    ui:
        cards: true

#[...] Other feature flags
```

此外，这个新设置允许我们为功能标志添加前缀-在我们的例子中，使用features前缀。

这看起来像是一个小细节，但随着我们的应用程序的增长和复杂性的增加，这个简单的迭代将帮助我们控制我们的功能标志。

让我们来谈谈这种方法的其他好处。

### 3.3 使用@ConfigurationProperties

一旦我们获得一组前缀属性，我们就可以创建一个用[@ConfigurationProperties](https://www.baeldung.com/configuration-properties-in-spring-boot)装饰的POJO，以在我们的代码中获取编程句柄。

按照我们目前的例子：

```java
@Component
@ConfigurationProperties(prefix = "features")
public class ConfigProperties {

    private MinerProperties miner;
    private UIProperties ui;

    // standard getters and setters

    public static class MinerProperties {
        private boolean experimental;
        // standard getters and setters
    }

    public static class UIProperties {
        private boolean cards;
        // standard getters and setters
    }
}
```

**通过将我们的功能标志的状态放在一个有凝聚力的单元中，我们开辟了新的可能性，使我们能够轻松地将该信息公开给系统的其他部分，例如UI或下游系统**。

### 3.4 公开功能配置

我们的比特币挖矿系统进行了一次UI升级，但尚未完全完成。因此，我们决定将其添加到功能标记中。我们可能会使用React、Angular或Vue开发一个单页应用。

无论采用何种技术，**我们都需要知道启用了哪些功能，以便我们可以相应地呈现我们的页面**。

让我们创建一个简单的端点来提供我们的配置，以便我们的UI可以在需要时查询后端：

```java
@RestController
public class FeaturesConfigController {

    private ConfigProperties properties;

    // constructor

    @GetMapping("/feature-flags")
    public ConfigProperties getProperties() {
        return properties;
    }
}
```

可能有更复杂的方法来提供这些信息，例如[创建自定义Actuator端点](https://www.baeldung.com/spring-boot-actuators)。但就本指南而言，控制器端点似乎是一个足够好的解决方案。

### 3.5 保持清洁

虽然这听起来很明显，但是一旦我们深思熟虑地实现了我们的功能标志，同样重要的是在不再需要它们时保持纪律地摆脱它们。

**第一个用例(基于主干的开发和非平凡功能)的功能标志通常是短暂的**，这意味着我们需要确保我们的ConfigProperties、Java配置和YAML文件保持干净和最新。

## 4. 更细粒度的功能开关

**有时我们会遇到更复杂的情况，对于A/B测试或金丝雀发布，我们之前的方法根本不够用**。

为了更精细地获取功能开关，我们可能需要创建解决方案，这可能涉及自定义用户实体以包含特定于功能的信息，或者扩展我们的Web框架。

然而，用功能标志污染我们的用户可能对每个人来说都不是一个有吸引力的想法，而且还有其他解决方案。

另外，我们可以利用一些内置工具，例如[Togglz](https://www.togglz.org/)。这个工具虽然增加了一些复杂性，但提供了一个不错的开箱即用解决方案，并与Spring Boot实现了一流的[集成](https://www.togglz.org/documentation/spring-boot-starter.html)。

Togglz支持不同的[激活策略](https://www.togglz.org/documentation/activation-strategies.html)：

1. **用户名**：与特定用户关联的标志。
2. **逐步推出**：为一定比例的用户启用标记，这对于金丝雀发布非常有用，例如，当我们想要验证功能的行为时。
3. **发布日期**：我们可以安排在特定日期和时间启用标志，这可能适用于产品发布、协调发布或优惠和折扣。
4. **客户端IP**：根据客户端IP标记的功能，当将特定配置应用于特定客户(假设他们拥有静态IP)时，这些功能可能会派上用场。
5. **服务器IP**：在这种情况下，服务器IP用于确定是否应启用某项功能。这对于金丝雀发布可能也很有用，但方法与逐步发布略有不同-例如，当我们想要评估实例的性能影响时。
6. **脚本**：我们可以基于[任意脚本](https://docs.oracle.com/en/java/javase/21/docs/api/java.scripting/javax/script/ScriptEngine.html)启用功能开关，这可以说是最灵活的选项。
7. **系统属性**：我们可以设置某些系统属性来确定功能开关的状态，这与我们最直接的方法非常相似。

## 5. 总结

**在本文中，我们讨论了功能开关，此外，我们还讨论了Spring如何在不添加新库的情况下帮助我们实现部分功能**。

我们首先定义这种模式如何帮助我们解决一些常见的用例。

接下来，我们使用Spring和Spring Boot的开箱即用工具构建了一些简单的解决方案。最终，我们提出了一个简单但强大的功能标记结构。

下面，我们比较了几种方案，从简单、灵活性较差的解决方案，过渡到更精细、更复杂的模式。

最后，我们简要提供了一些构建更健壮解决方案的指导原则。当我们需要更高粒度的解决方案时，这些指导原则非常有用。