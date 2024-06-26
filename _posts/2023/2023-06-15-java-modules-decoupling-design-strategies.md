---
layout: post
title:  解耦Java模块的设计策略
category: java
copyright: java
excerpt: Java JPMS
---

## 1. 概述

[Java平台模块系统](https://www.baeldung.com/java-9-modularity)(JPMS)提供了更强大的封装、更高的可靠性和更好的关注点分离。

但所有这些方便的功能都是有代价的。由于模块化应用程序建立在依赖其他模块才能正常工作的模块网之上，因此**在许多情况下，模块彼此紧密耦合**。

这可能会让我们认为模块化和松散耦合是不能在同一系统中共存的特性。但实际上，他们可以！

在本教程中，我们将深入研究两种众所周知的设计模式，它们可用于轻松解耦Java模块。

## 2. 父模块

为了展示我们将用于解耦Java模块的设计模式，我们将构建一个Demo多模块Maven项目。

为了保持代码简单，该项目最初将包含两个Maven模块，[每个Maven模块将包装到一个Java模块中](https://www.baeldung.com/maven-multi-module-project-java-jpms)。

第一个模块将包括一个服务接口，以及两个实现-服务提供者。第二个模块将使用提供者来解析字符串值。

让我们从创建名为demoproject的项目根目录开始，我们将定义项目的父POM：

```xml
<packaging>pom</packaging>

<modules>
    <module>servicemodule</module>
    <module>consumermodule</module>
</modules>
    
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

在父POM的定义中有一些细节值得强调。

首先，**该文件包含我们上面提到的两个子模块**，即servicemodule和consumermodule(稍后我们将详细讨论)。

接下来，**由于我们使用的是Java 11，因此我们的系统至少需要Maven 3.5.0，因为Maven从该版本开始支持Java 9及更高版本**。

最后，我们还需要至少版本3.8.0的[Maven编译器插件](https://www.baeldung.com/maven-compiler-plugin)。因此，确保检查[Maven Central](https://search.maven.org/classic/#search|ga|1|g%3A"org.apache.maven.plugins"ANDa%3A"maven-compiler-plugin")以获取最新版本的Maven编译器插件。

## 3. 服务模块

出于演示目的，让我们使用一种快速而简单的方法来实现servicemodule模块，这样我们就可以清楚地发现此设计中出现的缺陷。

让我们**将服务接口和服务提供者公开**，方法是将它们放在同一个包中并全部导出。这似乎是一个相当不错的设计选择，但正如我们稍后将看到的那样，**它极大提高了项目模块之间的耦合程度**。

在项目的根目录下，我们将创建servicemodule/src/main/java目录。然后，我们需要定义包cn.tuyucheng.taketoday.servicemodule，并在其中放置以下TextService接口：

```java
public interface TextService {

    String processText(String text);
}
```

TextService接口非常简单，现在让我们定义服务提供者。

在同一个包中，让我们添加一个Lowercase实现：

```java
public class LowercaseTextService implements TextService {

    @Override
    public String processText(String text) {
        return text.toLowerCase();
    }
}
```

现在，让我们添加一个Uppercase实现：

```java
public class UppercaseTextService implements TextService {

    @Override
    public String processText(String text) {
        return text.toUpperCase();
    }
}
```

最后，在servicemodule/src/main/java目录下，让我们包含模块描述符module-info.java：

```java
module cn.tuyucheng.taketoday.servicemodule {
    exports cn.tuyucheng.taketoday.servicemodule;
}
```

## 4. 消费者模块

现在我们需要创建一个使用我们之前创建的服务提供者之一的消费者模块。

让我们添加以下cn.tuyucheng.taketoday.consumermodule.Application类：

```java
public class Application {
    public static void main(String[] args) {
        TextService textService = new LowercaseTextService();
        System.out.println(textService.processText("Hello from Tuyucheng!"));
    }
}
```

现在，让我们在源代码根目录中(它应该是consumermodule/src/main/java)包含模块描述符module-info.java：

```java
module cn.tuyucheng.taketoday.consumermodule {
    requires cn.tuyucheng.taketoday.servicemodule;
}
```

最后，让我们从IDE或命令控制台编译源文件并运行应用程序。

正如我们所料，我们应该看到以下输出：

```text
hello from tuyucheng!
```

显然这是可行的，但有一个值得注意的重要警告：**我们不必要地将服务提供者耦合到消费者模块**。

**因为我们让提供者对外界可见，所以消费者模块知道它们**。

此外，这违背于使软件组件依赖于抽象。

## 5. 服务提供者工厂

我们可以**通过只导出服务接口来轻松消除模块之间的耦合**。相比之下，服务提供者不会导出，因此对消费者模块保持隐藏状态。消费者模块只能看到服务接口类型。

为此，我们需要：

1.  将服务接口放在单独的包中，对外导出
2.  将服务提供者放在不同的包中，不导出
3.  创建一个工厂类，它被导出。消费者模块使用工厂类查找服务提供者

我们可以将上述步骤以设计模式的形式概念化：**公开服务接口、私有服务提供者和公开服务提供者工厂**。

### 5.1 公开服务接口

为了清楚地了解此模式的工作原理，让我们将服务接口和服务提供者放在不同的包中。接口将被导出，但提供者实现不会。

因此，让我们将TextService移动到一个名为cn.tuyucheng.taketoday.servicemodule.external的新包中。

### 5.2 私有服务提供者

然后，让我们以类似的方式将LowercaseTextService和UppercaseTextService移动到cn.tuyucheng.taketoday.servicemodule.internal。

### 5.3 公开服务提供者工厂

由于服务提供者类现在是私有的，不能从其他模块访问，**因此我们将使用公共工厂类来提供一种简单的机制，消费者模块可以使用该机制来获取服务提供者的实例**。

在cn.tuyucheng.taketoday.servicemodule.external包中，让我们定义以下TextServiceFactory类：

```java
public class TextServiceFactory {

    private TextServiceFactory() {
    }

    public static TextService getTextService(String name) {
        return name.equalsIgnoreCase("lowercase") ? new LowercaseTextService() : new UppercaseTextService();
    }
}
```

当然，我们可以使工厂类稍微复杂一些。为了简单起见，服务提供者只是根据传递给getTextService()方法的字符串值创建的。

现在，让我们替换我们的module-info.java文件以仅导出我们的external包：

```java
module cn.tuyucheng.taketoday.servicemodule {
    exports cn.tuyucheng.taketoday.servicemodule.external;
}
```

**请注意，我们只导出服务接口和工厂类**。这些实现是私有的，因此它们对其他模块不可见。

### 5.4 应用程序类

现在，让我们重构Application类，以便它可以使用服务提供者工厂类：

```java
public static void main(String args[]) {
    TextService textService = TextServiceFactory.getTextService("lowercase");
    System.out.println(textService.processText("Hello from Tuyucheng!"));
}
```

正如预期的那样，如果我们运行该应用程序，我们应该会看到控制台打印出相同的文本：

```text
hello from tuyucheng!
```

**通过公开服务接口和私有服务提供者，我们可以通过一个简单的工厂类有效地分离服务和消费者模块**。

当然，没有模式是灵丹妙药。与往常一样，我们应该首先分析我们的用例是否合适。

## 6. 服务和消费者模块

JPMS通过provides...with和uses指令为开箱即用的服务和消费者模块提供支持。

因此，我们可以使用此功能来解耦模块，而**无需创建额外的工厂类**。

为了让服务和消费者模块一起工作，我们需要执行以下操作：

1.  将服务接口放在一个模块中，该模块导出接口
2.  将服务提供者放在另一个模块中-导出提供者
3.  在提供者的模块描述符中使用provides...with指令指定我们要提供TextService实现
4.  将Application类放在它自己的模块中-消费者模块
5.  在消费者模块的模块描述符中指定该模块是具有uses指令的消费者模块
6.  使用消费者模块中的[Service Loader API](https://www.baeldung.com/java-spi)查找服务提供者

这种方法非常强大，因为它利用了服务和消费者模块带来的所有功能。但这也有些棘手。

一方面，我们让消费者模块只依赖于服务接口，而不依赖于服务提供者。另一方面，**我们甚至可以完全不定义服务提供者，应用程序仍然会编译**。

### 6.1 父模块

要实现此模式，我们还需要重构父POM和现有模块。

由于服务接口、服务提供者和消费者现在将放置在不同的模块中，我们首先需要修改父POM的<modules\>部分，以反映这个新结构：

```xml
<modules>
    <module>servicemodule</module>
    <module>providermodule</module>
    <module>consumermodule</module>
</modules>
```

### 6.2 服务模块

我们的TextService接口将位于cn.tuyucheng.taketoday.servicemodule。

相应地更改模块描述符：

```java
module cn.tuyucheng.taketoday.servicemodule {
    exports cn.tuyucheng.taketoday.servicemodule;
}
```

### 6.3 提供者模块

如前所述，提供者模块用于我们的实现，所以现在让我们在该模块放置LowerCaseTextService和UppercaseTextService。我们将把它们放在一个称为cn.tuyucheng.taketoday.providermodule的包中。

最后，让我们添加一个module-info.java文件：

```java
module cn.tuyucheng.taketoday.providermodule {
    requires cn.tuyucheng.taketoday.servicemodule;
    provides cn.tuyucheng.taketoday.servicemodule.TextService with cn.tuyucheng.taketoday.providermodule.LowercaseTextService;
}
```

### 6.4 消费者模块

现在，让我们重构消费者模块。首先，我们将Application放到cn.tuyucheng.taketoday.consumermodule包中。

接下来，我们将重构Application类的main()方法，以便它可以使用[ServiceLoader](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ServiceLoader.html)类来发现适当的实现：

```java
public static void main(String[] args) {
    ServiceLoader<TextService> services = ServiceLoader.load(TextService.class);
    for (final TextService service: services) {
        System.out.println("The service " + service.getClass().getSimpleName() + " says: " + service.parseText("Hello from Tuyucheng!"));
    }
}
```

最后，我们将重构module-info.java文件：

```java
module cn.tuyucheng.taketoday.consumermodule {
    requires cn.tuyucheng.taketoday.servicemodule;
    uses cn.tuyucheng.taketoday.servicemodule.TextService;
}
```

现在，让我们运行应用程序。正如预期的那样，我们应该看到以下文本打印到控制台：

```text
The service LowercaseTextService says: hello from tuyucheng!
```

如我们所见，实现此模式比使用工厂类的模式稍微复杂一些。即便如此，额外的努力会得到更灵活、松散耦合的设计的高度回报。

**消费者模块依赖于抽象，在运行时也很容易放入不同的服务提供者**。

## 7. 总结

在本教程中，我们学习了如何实现两种模式来解耦Java模块。

**这两种方法都使消费者模块依赖于抽象，这在软件组件的设计中始终是一个理想的特性**。

当然，每一种都有其优点和缺点。对于第一个，我们得到了很好的解耦，但我们必须创建一个额外的工厂类。

对于第二个，为了让模块解耦，我们必须创建一个额外的抽象模块，并使用Service Loader API添加一个新的[间接级别](https://en.wikipedia.org/wiki/Indirection)。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-jpms)上获得。