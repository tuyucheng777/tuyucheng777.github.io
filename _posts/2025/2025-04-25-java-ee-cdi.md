---
layout: post
title:  Java CDI简介
category: libraries
copyright: libraries
excerpt: CDI
---

## 1. 概述

CDI(上下文和依赖注入)是Java EE 6及更高版本中包含的标准[依赖注入框架](https://en.wikipedia.org/wiki/Dependency_injection)。

**它允许我们通过特定于域的生命周期上下文管理有状态组件的生命周期，并以类型安全的方式将组件(服务)注入客户端对象**。

在本教程中，我们将深入了解CDI最相关的功能，并实现在客户端类中注入依赖的不同方法。

## 2. DYDI(自行依赖注入)

简而言之，无需借助任何框架就可以实现DI。

这种方法通常被称为DYDI(Do-it-Yourself Dependency Injection)。

利用DYDI，我们通过普通的旧工厂/构建器将所需的依赖传递到客户端类，从而使应用程序代码与对象创建隔离。

基本的DYDI实现可能如下所示：

```java
public interface TextService {
    String doSomethingWithText(String text);
    String doSomethingElseWithText(String text);    
}
```

```java
public class SpecializedTextService implements TextService { ... }
```

```java
public class TextClass {
    private TextService textService;
    
    // constructor
}
```

```java
public class TextClassFactory {
      
    public TextClass getTextClass() {
        return new TextClass(new SpecializedTextService(); 
    }    
}
```

当然DYDI适合一些比较简单的用例。

如果我们的示例应用程序的规模和复杂性不断增加，并实现了更大的互连对象网络，那么我们最终会用大量的对象图工厂来污染它。

仅仅为了创建对象图就需要大量的样板代码。这不是一个完全可扩展的解决方案。

我们能把DI做得更好吗？当然可以，这正是CDI发挥作用的地方。

## 3. 一个简单的例子

**CDI将 DI变成一个无需动脑筋的过程，归结为仅使用一些简单的注解来装饰服务类，并在客户端类中定义相应的注入点**。

为了展示CDI如何在最基本的层面上实现DI，假设我们要开发一个简单的图像文件编辑应用程序。该应用程序能够打开、编辑、写入、保存图像文件等。

### 3.1 beans.xml文件

首先，我们必须在“src/main/resources/META-INF/”文件夹中放置一个“beans.xml”文件，**即使此文件根本不包含任何特定的DI指令，它也是启动和运行CDI所必需的**：

```xml
<beans xmlns="http://java.sun.com/xml/ns/javaee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
  http://java.sun.com/xml/ns/javaee/beans_1_0.xsd">
</beans>
```

### 3.2 服务类

接下来，让我们创建对GIF、JPG和PNG文件执行上述文件操作的服务类：

```java
public interface ImageFileEditor {
    String openFile(String fileName);
    String editFile(String fileName);
    String writeFile(String fileName);
    String saveFile(String fileName);
}
```

```java
public class GifFileEditor implements ImageFileEditor {
    
    @Override
    public String openFile(String fileName) {
        return "Opening GIF file " + fileName;
    }
    
    @Override
    public String editFile(String fileName) {
      return "Editing GIF file " + fileName;
    }
    
    @Override
    public String writeFile(String fileName) {
        return "Writing GIF file " + fileName;
    }

    @Override
    public String saveFile(String fileName) {
        return "Saving GIF file " + fileName;
    }
}
```

```java
public class JpgFileEditor implements ImageFileEditor {
    // JPG-specific implementations for openFile() / editFile() / writeFile() / saveFile()
    ...
}
```

```java
public class PngFileEditor implements ImageFileEditor {
    // PNG-specific implementations for openFile() / editFile() / writeFile() / saveFile()
    ...
}
```

### 3.3 客户端类

最后，让我们实现一个客户端类，该类在构造函数中接收ImageFileEditor实现，并使用@Inject注解定义一个注入点：

```java
public class ImageFileProcessor {
    
    private ImageFileEditor imageFileEditor;
    
    @Inject
    public ImageFileProcessor(ImageFileEditor imageFileEditor) {
        this.imageFileEditor = imageFileEditor;
    }
}
```

简而言之，**@Inject注解是CDI的真正主力，它允许我们在客户端类中定义注入点**。

在这种情况下，@Inject指示CDI在构造函数中注入ImageFileEditor实现。

此外，还可以通过在字段(字段注入)和Setter(Setter注入)中使用@Inject注解来注入服务，我们稍后会讨论这些选项。

### 3.4 使用Weld构建ImageFileProcessor对象图

当然，我们需要确保CDI将正确的ImageFileEditor实现注入到ImageFileProcessor类构造函数中。

为此，首先，我们应该获取该类的一个实例。

**由于我们不会依赖任何Java EE应用服务器来使用CDI，因此我们将使用Java SE中的CDI参考实现[Weld](http://weld.cdi-spec.org/)来实现**：

```java
public static void main(String[] args) {
    Weld weld = new Weld();
    WeldContainer container = weld.initialize();
    ImageFileProcessor imageFileProcessor = container.select(ImageFileProcessor.class).get();
 
    System.out.println(imageFileProcessor.openFile("file1.png"));
 
    container.shutdown();
}
```

在这里，我们创建一个WeldContainer对象，然后获取一个ImageFileProcessor对象，最后调用它的openFile()方法。

正如预期的那样，如果我们运行该应用程序，CDI将抛出DeploymentException并大声抱怨：

```text
Unsatisfied dependencies for type ImageFileEditor with qualifiers @Default at injection point...
```

**我们得到这个异常是因为CDI不知道要将哪个ImageFileEditor实现注入到ImageFileProcessor构造函数中**。

在CDI的术语中，**这被称为模糊注入异常**。

### 3.5 @Default和@Alternative注解

解决这个歧义很容易，**CDI默认使用@Default注解来标注接口的所有实现**。

因此，我们应该明确地告诉它应该将哪个实现注入到客户端类中：

```java
@Alternative
public class GifFileEditor implements ImageFileEditor { ... }
```

```java
@Alternative
public class JpgFileEditor implements ImageFileEditor { ... }
```

```java
public class PngFileEditor implements ImageFileEditor { ... }
```

在这种情况下，我们已经使用@Alternative注解对GifFileEditor和JpgFileEditor进行了标注，因此CDI现在知道PngFileEditor(默认使用@Default注解进行标注)是我们想要注入的实现。

如果我们重新运行该应用程序，这次它将按预期执行：

```text
Opening PNG file file1.png
```

此外，使用@Default注解标注PngFileEditor并保留其他实现作为替代将产生相同的上述结果。

简而言之，**这表明我们如何通过简单地切换服务类中的@Alternative注解来非常轻松地交换运行时实现的注入**。

## 4. 字段注入

CDI支持开箱即用的字段和Setter注入。

下面介绍如何执行字段注入(**使用@Default和@Alternative注解限定服务的规则保持不变**)：

```java
@Inject
private final ImageFileEditor imageFileEditor;
```

## 5. Setter注入

类似地，下面是如何进行Setter注入：

```java
@Inject 
public void setImageFileEditor(ImageFileEditor imageFileEditor) { ... }
```

## 6. @Named注解

到目前为止，我们已经学习了如何在客户端类中定义注入点，以及如何使用@Inject、@Default和@Alternative注解注入服务，这些注解涵盖了大多数用例。

尽管如此，CDI还允许我们使用@Named注解执行服务注入。

**此方法通过将有意义的名称绑定到实现来提供注入服务的更语义的方式**：

```java
@Named("GifFileEditor")
public class GifFileEditor implements ImageFileEditor { ... }

@Named("JpgFileEditor")
public class JpgFileEditor implements ImageFileEditor { ... }

@Named("PngFileEditor")
public class PngFileEditor implements ImageFileEditor { ... }
```

现在，我们应该重构ImageFileProcessor类中的注入点以匹配命名实现：

```java
@Inject 
public ImageFileProcessor(@Named("PngFileEditor") ImageFileEditor imageFileEditor) { ... }
```

还可以使用命名实现来执行字段和setter注入，这看起来与使用@Default和@Alternative注解非常相似：

```java
@Inject 
private final @Named("PngFileEditor") ImageFileEditor imageFileEditor;

@Inject 
public void setImageFileEditor(@Named("PngFileEditor") ImageFileEditor imageFileEditor) { ... }
```

## 7. @Produces注解

有时，服务需要先完全初始化一些配置，然后才能注入以处理其他依赖。

CDI通过@Produces注解为这些情况提供支持。

**@Produces允许我们实现工厂类，其职责是创建完全初始化的服务**。

为了理解@Produces注解的工作原理，让我们重构ImageFileProcessor类，以便它可以在构造函数中接收额外的TimeLogger服务。

该服务将用于记录执行某个图像文件操作的时间：

```java
@Inject
public ImageFileProcessor(ImageFileEditor imageFileEditor, TimeLogger timeLogger) { ... } 
    
public String openFile(String fileName) {
    return imageFileEditor.openFile(fileName) + " at: " + timeLogger.getTime();
}
    
// additional image file methods
```

在这种情况下，TimeLogger类需要两个附加服务，SimpleDateFormat和Calendar：

```java
public class TimeLogger {
    
    private SimpleDateFormat dateFormat;
    private Calendar calendar;
    
    // constructors
    
    public String getTime() {
        return dateFormat.format(calendar.getTime());
    }
}
```

我们如何告诉CDI在哪里获取完全初始化的TimeLogger对象？

我们只需创建一个TimeLogger工厂类并使用@Produces注解标注其工厂方法：

```java
public class TimeLoggerFactory {
    
    @Produces
    public TimeLogger getTimeLogger() {
        return new TimeLogger(new SimpleDateFormat("HH:mm"), Calendar.getInstance());
    }
}
```

每当我们获得一个ImageFileProcessor实例时，CDI都会扫描TimeLoggerFactory类，然后调用getTimeLogger()方法(因为它带有@Produces注解)，最后注入TimeLogger服务。

如果我们使用Weld运行重构的示例应用程序，它将输出以下内容：

```text
Opening PNG file file1.png at: 17:46
```

## 8. 自定义限定符

CDI支持使用自定义限定符来限定依赖并解决模糊的注入点。

自定义限定符是一个非常强大的功能，**它们不仅将语义名称绑定到服务，还将注入元数据绑定到服务上**，元数据包括[RetentionPolicy](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/annotation/RetentionPolicy.html)和合法的注解目标([ElementType](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/annotation/ElementType.html))。

让我们看看如何在应用程序中使用自定义限定符：

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
public @interface GifFileEditorQualifier {}
```

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
public @interface JpgFileEditorQualifier {}
```

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE, ElementType.PARAMETER})
public @interface PngFileEditorQualifier {}
```

现在，让我们将自定义限定符绑定到ImageFileEditor实现：

```java
@GifFileEditorQualifier
public class GifFileEditor implements ImageFileEditor { ... }
```

```java
@JpgFileEditorQualifier
public class JpgFileEditor implements ImageFileEditor { ... }
```

```java
@PngFileEditorQualifier
public class PngFileEditor implements ImageFileEditor { ... }
```

最后，让我们重构ImageFileProcessor类中的注入点：

```java
@Inject
public ImageFileProcessor(@PngFileEditorQualifier ImageFileEditor imageFileEditor, TimeLogger timeLogger) { ... }
```

如果我们再次运行我们的应用程序，它应该生成上面显示的相同输出。

自定义限定符提供了一种简洁的语义方法，用于将名称和注解元数据绑定到实现。

此外，**自定义限定符允许我们定义更具限制性的类型安全注入点(优于@Default和@Alternative注解的功能)**。

**如果类型层次结构中仅有一个子类型符合条件，那么CDI将只注入子类型，而不是基类型**。

## 9. 总结

毫无疑问，CDI使依赖注入变得轻而易举，额外注解的成本对于有组织的依赖注入的收益来说微不足道。

有时候，DYDI仍然比CDI更有优势，例如，在开发仅包含简单对象图的相当简单的应用程序时。