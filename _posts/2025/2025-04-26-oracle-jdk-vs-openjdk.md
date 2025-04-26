---
layout: post
title:  Oracle JDK和OpenJDK之间的区别
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 简介

在本教程中，我们将探讨[Oracle JDK](https://www.oracle.com/technetwork/java/index.html)和[OpenJDK](https://openjdk.java.net/)之间的区别。首先，我们将仔细研究它们，然后进行比较。最后，我们将列出其他JDK实现。

## 2. Oracle JDK和Java SE历史

JDK(Java开发工具包)是Java平台编程中使用的软件开发环境，它包含一个完整的Java运行时环境，即所谓的私有运行时。它之所以得名，是因为它包含比独立JRE更多的工具，以及开发Java应用程序所需的其他组件。

**Oracle强烈建议使用术语JDK来指代Java SE(标准版)开发工具包(还有企业版和微型版平台)**。

让我们回顾一下Java SE的历史：

-   JDK Beta：1995
-   JDK 1.0：1996年1月
-   JDK 1.1：1997年2月
-   J2SE 1.2：1998年12月
-   J2SE 1.3：2000年5月
-   J2SE 1.4：2002年2月
-   J2SE 5.0：2004年9月
-   Java SE 6：2006年12月
-   Java SE 7：2011年7月
-   Java SE 8(LTS)：2014年3月
-   Java SE 9：2017年9月
-   Java SE 10(18.3)：2018年3月
-   Java SE 11(18.9LTS)：2018年9月
-   Java SE 12(19.3)：2019年3月
-   Java SE 13：2019年9月
-   Java SE 14：2020年3月
-   Java SE 15：2020年9月
-   Java SE 16：2021年3月
-   Java SE 17(LTS)：2021年9月
-   Java SE 18：2022年3月
-   Java SE 19：2022年9月
-   Java SE 20：2023年3月
-   Java SE 21(LTS)：2023年9月

我们可以看到，Java SE的主要版本大约每两年发布一次，直到Java SE 7。从Java SE 6开始花了五年时间，之后又花了三年时间才到达Java SE 8。

自Java SE 10以来，我们预计每六个月就会发布一次新版本。但是，并非所有版本都是长期支持(LTS)版本，根据Oracle的发布计划，LTS产品每三年才会发布一次。

Java SE 17是最新的LTS版本，Java SE 8可免费公开更新至2020年12月，供非商业用途使用。

该开发工具包在2010年Oracle收购Sun Microsystems后获得了现在的名称，在此之前，它的名称是SUN JDK，它是Java编程语言的官方实现。

## 3. OpenJDK

OpenJDK是Java SE平台版本的免费[开源](https://www.baeldung.com/cs/open-source-explained)实现，它最初于2007年发布，是Sun Microsystems于2006年开始开发的成果。

**需要强调的是，OpenJDK是自SE 7版本以来Java标准版的官方参考实现**。

最初，它仅基于JDK 7，但**从Java 10开始，Java SE平台的开源参考实现由[JDK项目](https://openjdk.java.net/projects/jdk/)负责**。并且，与Oracle一样，JDK项目也会每六个月发布一次新的功能版本。

我们应该注意到，在这个长期运行的项目之前，有一些JDK发布项目发布了一个功能，然后就停止了。

现在让我们检查一下OpenJDK版本：

- OpenJDK 6项目：基于JDK 7，但经过修改，提供Java 6的开源版本
- OpenJDK 7项目：2011年7月28日
- OpenJDK 7u项目：该项目开发Java开发工具包7的更新
- OpenJDK 8项目：2014年3月18日
- OpenJDK 8u项目：该项目开发Java开发工具包8的更新
- OpenJDK 9项目：2017年9月21日
- JDK 10：2018年3月10日至20日
- JDK 11：2018年9月11日至25日
- JDK 12：[稳定阶段](https://openjdk.java.net/jeps/3)

## 4. Oracle JDK与OpenJDK

在本节中，我们将重点介绍Oracle JDK和OpenJDK之间的主要区别。

### 4.1 发布时间表

正如我们所提到的，**Oracle每三年发布一次版本，而OpenJDK每六个月发布一次版本**。

Oracle为其版本提供长期支持，而OpenJDK仅支持对当前版本的更改，直到下一个版本发布。

### 4.2 许可证

**Oracle JDK是根据Oracle二进制代码许可协议获得许可的，而OpenJDK则采用GNU通用公共许可证(GNU GPL)版本2并带有链接例外**。

使用Oracle平台时，存在一些许可方面的问题，Oracle已[宣布](https://java.com/en/download/release_notice.jsp)，2019年1月之后发布的Oracle Java SE 8公开更新，未经商业许可将无法用于商业、商业或生产用途。不过，OpenJDK是完全开源的，可以免费使用。

### 4.3 性能

**两者之间没有真正的技术差异，因为Oracle JDK的构建过程基于OpenJDK的构建过程**。

在性能方面，**Oracle在响应能力和JVM性能方面表现更佳**，由于其重视企业客户，因此更加注重稳定性。

相比之下，OpenJDK的发布频率更高。因此，我们可能会遇到一些不稳定的问题。根据[社区反馈](https://www.reddit.com/r/java/comments/6g86p9/openjdk_vs_oraclejdk_which_are_you_using/)，我们了解到一些OpenJDK用户遇到了性能问题。

### 4.4 功能

如果我们比较功能和选项，我们会发现**Oracle产品具有Flight Recorder、Java Mission Control和应用程序类数据共享功能，而OpenJDK具有字体渲染器功能**。

此外，**Oracle拥有更多的垃圾回收选项和更好的渲染器**。

### 4.5 发展与普及

**Oracle JDK完全由Oracle公司开发，而OpenJDK由Oracle、OpenJDK和Java社区开发**。但是，Red Hat、Azul Systems、IBM、Apple Inc和SAP AG等顶尖公司也积极参与其开发。

从上一节的链接中我们可以看出，当谈到在其工具中使用Java开发工具包(例如Android Studio或IntelliJ IDEA)的顶级公司的受欢迎程度时，**Oracle JDK曾经是更受欢迎的，但这两家公司都已转向基于OpenJDK的[JetBrains版本](https://bintray.com/jetbrains/intellij-jdk/)**。

此外，主要的Linux发行版(Fedora、Ubuntu、Red Hat Enterprise Linux)都提供OpenJDK作为默认的Java SE实现。

## 5. 自Java 11以来的变化

正如我们在[Oracle的博客文章](https://blogs.oracle.com/java-platform-group/oracle-jdk-releases-for-java-11-and-later)中看到的，从Java 11开始有一些重要的变化。

首先，**当使用Oracle JDK作为Oracle产品或服务的一部分，或者当开源软件不受欢迎时，Oracle将把其历史性的“[BCL](https://www.oracle.com/technetwork/java/javase/terms/license/index.html)”许可证更改为带有[类路径例外(GPLv2+CPE)](https://openjdk.java.net/legal/gplv2+ce.html)的开源GNU通用公共许可证v2和商业许可证的组合**。

每个许可证都有不同的版本，但它们的功能相同，只有一些外观和包装上的差异。

此外，传统的“商业功能”，例如JFR、JMC、应用程序类数据共享以及ZGC，现在也已在OpenJDK中提供。因此，**从Java 11开始，Oracle JDK和OpenJDK的版本基本相同**。

让我们来看看主要的区别：

- 使用-XX:+UnlockCommercialFeatures选项时，Oracle的Java 11工具包会发出警告，而在OpenJDK构建中，此选项会导致错误。
- Oracle JDK提供配置，向“Advanced Management Console”工具提供使用日志数据。
- Oracle一直要求第三方加密提供程序必须由已知证书签名，而OpenJDK中的加密框架具有开放的加密接口，这意味着对于可以使用哪些提供程序没有任何限制。
- Oracle JDK 11将继续包含安装程序、品牌和JRE打包，而OpenJDK版本目前以zip和tar.gz文件的形式提供。
-  由于Oracle发行版中存在一些附加模块，因此javac–release命令对于Java 9和Java 10目标的行为有所不同。
- java–version和java-fullversion命令的输出将区分Oracle的构建和OpenJDK的构建。

## 6. 其他JDK实现

现在让我们快速看一下其他活跃的Java开发工具包实现。

### 6.1 免费且开源

以下按字母顺序列出的实现都是开源的，可以免费使用：

- AdoptOpenJDK
- Amazon Corretto
- Azul Zulu
- Bck2Brwsr
- CACAO
- Codename One
- DoppioJVM
- Eclipse OpenJ9
- GraalVM CE
- HaikuVM
- HotSpot
- Jamiga
- JamVM
- Jelatine JVM
- Jikes RVM (Jikes Research Virtual Machine)
- JVM.go
- Liberica JDK
- leJOS
- Maxine
- Multi-OS Engine
- RopeVM
- uJVM

### 6.2 专有实现

还有一些受版权保护的实现：

- Azul Zing JVM
- CEE-J
- Excelsior JET
- GraalVM EE
- Imsys AB
- JamaicaVM(aicas)
- JBlend(Aplix)
- MicroJvm(IS2T–Industrial Smart Software Technology)
- OJVM
- PTC Perc
- SAP JVM
- Waratek CloudVM for Java

除了上面列出的[活跃实现](https://en.wikipedia.org/wiki/List_of_Java_virtual_machines#Active)之外，我们还可以查看[非活跃实现](https://en.wikipedia.org/wiki/List_of_Java_virtual_machines#Inactive)的列表以及每个实现的简短描述。

### 6.3 Amazon Corretto JDK和OpenJDK之间的差异

Java开发工具包(Java Development Kit)的一个流行实现是Amazon Corretto，虽然它基于OpenJDK，但它们之间还是存在一些差异。

Amazon Corretto是OpenJDK的一个免费、多平台、生产就绪型发行版，它由Amazon Web Services(AWS)开发、维护和支持，专为在AWS和其他云平台上运行Java应用程序而设计，Amazon Corretto免费提供长期支持(LTS)和安全更新。

OpenJDK和Amazon Corretto JDK之间的另一个主要区别是，Amazon Corretto包含专门设计用于提高生产环境中的性能、安全性和可靠性的附加功能和优化。

总而言之，OpenJDK和Amazon Corretto JDK都是JDK的实现，但它们的设计目的不同。**OpenJDK是一个开源开发工具，而Amazon Corretto JDK是一个可用于生产的OpenJDK发行版，专为在云环境中使用而设计，并提供长期的免费支持**。

## 7. OpenJDK和AdoptOpenJDK之间的区别

**2020年9月，AdoptOpenJDK进行了品牌重塑，并过渡到Adoptium**。此次品牌变更意义重大，它反映了其超越仅仅提供OpenJDK二进制文件的更广泛使命。

OpenJDK和AdoptOpenJDK/Adoptium虽然相互关联，但却是两个不同的项目，它们都提供了Java平台标准版(Java SE)的免费开源实现。为了阐明OpenJDK和AdoptOpenJDK/Adoptium之间的区别，让我们深入探讨一下它们之间的主要区别。

### 7.1 项目起源与治理

**OpenJDK是Java平台的官方参考实现，由Oracle监管**。作为一个开源项目，它汇聚了众多个人和组织的贡献，并作为主要的开发工具包发挥着核心作用。另一方面，**AdoptOpenJDK/Adoptium则是一个独立的社区驱动项目**。它旨在为各种平台提供对精心设计、预构建的OpenJDK二进制文件的便捷访问，并独立于Oracle官方主导的生态系统运行。

### 7.2 构建和分发

**OpenJDK提供源代码，开发者可以使用这些源代码构建二进制文件**。因此，用户需要自行编译代码并生成适合其特定平台的Java运行时。相比之下，**AdoptOpenJDK/Adoptium则致力于通过为各种操作系统提供预构建的二进制文件来简化此过程**，这使得用户可以方便地直接下载和使用这些二进制文件，无需手动编译。

### 7.3 分发模型

OpenJDK采用开源模式，源代码通常通过软件包管理器或直接从OpenJDK官方网站发布，用户需要独立编译源代码才能获得可用的Java运行时。相比之下，**AdoptOpenJDK/Adoptium通过向用户提供预构建的二进制文件简化了这一过程**。这些二进制文件可以轻松下载和安装，为源代码编译提供了一种用户友好的替代方案。AdoptOpenJDK/Adoptium提供与apt、yum等各种软件包管理器兼容的安装选项，进一步提升了用户的便利性。

### 7.4 社区参与

**OpenJDK社区拥有形形色色的贡献者，包括个人和公司**，他们都积极参与其中。该项目由OpenJDK社区管理，Oracle在其持续发展中扮演着重要角色。**AdoptOpenJDK/Adoptium则是一个社区驱动的计划，积极鼓励个人和组织做出贡献**。AdoptOpenJDK/Adoptium独立于Oracle运营，致力于营造协作环境，确保提供可自由访问的高质量OpenJDK二进制文件。

### 7.5 许可

**OpenJDK和AdoptOpenJDK/Adoptium共享基础，依赖相同的源代码，并且通常遵循相同的许可模式**。具体来说，它们均遵循GNU通用公共许可证第2版，并附带类路径例外(GPLv2+CPE)，这种许可结构凸显了两个项目对开源原则和可访问性的承诺。

## 8. 总结

在本文中，我们重点介绍了当今最流行的两种Java开发工具包。

首先，我们分别描述了它们，然后强调了它们之间的差异，我们还特别关注了自Java 11以来的变化和差异。最后，我们列出了目前可用的其他活跃实现。