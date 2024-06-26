---
layout: post
title:  带点(.)的Spring MVC @PathVariable被截断
category: springweb
copyright: springweb
excerpt: Spring Web
---

## 1. 概述

在这个简短的教程中，我们将讨论使用Spring MVC时的一个常见问题——当使用带有@RequestMapping的Spring @PathVariable来映射包含点的请求URI的结尾时，我们将得到一个部分值在我们的变量中，在最后一个点处被截断。

在接下来的部分中，我们将重点讨论为什么会发生这种情况以及如何改变这种行为。

Spring MVC的介绍可以参考[这篇文章](https://www.baeldung.com/spring-mvc-tutorial)。

## 2. 不需要的Spring帮助

由于框架解释路径变量的方式，它通常会导致这种不需要的行为。

具体来说，Spring认为最后一个点后面的任何内容都是文件扩展名，例如.json或.xml。

因此，它会截断值以检索参数。

让我们看一个使用路径变量的例子，然后用不同的可能值分析结果：

```java
@RestController
public class CustomController {
    @GetMapping("/example/{firstValue}/{secondValue}")
    public void example(@PathVariable("firstValue") String firstValue, @PathVariable("secondValue") String secondValue) {
        // ...  
    }
}
```

通过上面的示例，让我们考虑下一个请求并评估我们的变量：

-   URL example/gallery/link结果评估firstValue = “gallery”和secondValue = “link”
-   当使用example/gallery.df/link.ar URL时，我们将有firstValue = “gallery.df”和secondValue = “link”
-   对于example/gallery.df/link.com.ar URL，我们的变量将是：firstValue = “gallery.df”和secondValue = “link.com”

正如我们所见，第一个变量不受影响，但第二个变量总是被截断。

## 3. 解决方案

解决这种不便的一种方法是通过添加正则表达式映射来修改我们的@PathVariable定义。因此，任何点，包括最后一个点，都将被视为我们参数的一部分：

```java
@GetMapping("/example/{firstValue}/{secondValue:.+}")   
public void example(@PathVariable("firstValue") String firstValue, @PathVariable("secondValue") String secondValue) {
    //...
}
```

避免此问题的另一种方法是在@PathVariable的末尾添加斜杠。这将包含我们的第二个变量，以保护它免受Spring的默认行为的影响：

```java
@GetMapping("/example/{firstValue}/{secondValue}/")
```

上面的两个解决方案适用于我们正在修改的单个请求映射。

如果我们想在全局MVC级别更改行为，我们需要提供自定义配置。为此，我们可以扩展WebMvcConfigurationSupport并覆盖其getPathMatchConfigurer()方法来调整PathMatchConfigurer。

```java
@Configuration
public class CustomWebMvcConfigurationSupport extends WebMvcConfigurationSupport {

    @Override
    protected PathMatchConfigurer getPathMatchConfigurer() {
        PathMatchConfigurer pathMatchConfigurer = super.getPathMatchConfigurer();
        pathMatchConfigurer.setUseSuffixPatternMatch(false);

        return pathMatchConfigurer;
    }
}
```

我们必须记住，这种方法会影响所有URL。

使用这三个选项，我们将获得相同的结果：当调用example/gallery.df/link.com.ar URL时，我们的secondValue变量将被计算为“link.com.ar”，这正是我们想要的。

### 3.1 弃用通知

从Spring Framework [5.2.4](https://github.com/spring-projects/spring-framework/issues/24179)开始，不推荐使用setUseSuffixPatternMatch(boolean)方法，以阻止将路径扩展用于请求路由和内容协商。基本上，当前的实现很难保护Web应用程序免受[反射文件下载(RFD)](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-rfd)攻击。

此外，从Spring Framework 5.3开始，后缀模式匹配将仅适用于显式注册的后缀，以防止任意扩展。

最重要的是，从Spring 5.3开始，我们将不需要使用setUseSuffixPatternMatch(false)因为它在默认情况下是禁用的。

## 4. 总结

在这篇简短的文章中，我们探讨了在Spring MVC中使用@PathVariable和@RequestMapping时解决常见问题的不同方法以及此问题的根源。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-web-modules)上获得。