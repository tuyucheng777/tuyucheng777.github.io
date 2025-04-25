---
layout: post
title:  Java版Maven Spotless插件
category: staticanalysis
copyright: staticanalysis
excerpt: Spotless
---

## 1. 概述

在本教程中，我们将探索Maven Spotless插件，并使用它在整个项目中强制执行一致的代码样式。首先，我们将使用最小配置来分析源代码并解决潜在的格式违规问题。之后，我们将逐步更新插件的配置，以使用自定义规则并在特定的Maven阶段执行这些检查。

## 2. 入门

Maven Spotless插件是一个工具，可以在构建过程中自动格式化并强制执行各种编程语言的代码样式标准。**Spotless的使用非常简单，我们只需要在[spotless-maven-plugin](https://mvnrepository.com/artifact/com.diffplug.spotless/spotless-maven-plugin)中指定喜欢的编码格式即可**。

让我们首先将插件添加到我们的pom.xml并将其配置为使用[Google Java Style](https://google.github.io/styleguide/javaguide.html)：

```xml
<plugin>
    <groupId>com.diffplug.spotless</groupId>
    <artifactId>spotless-maven-plugin</artifactId>
    <version>2.43.0</version>
    <configuration>
        <java>
            <googleJavaFormat/>
        </java>
    </configuration>
</plugin>
```

就这样，**我们现在可以运行“mvn spotless:check”，插件将自动扫描我们的Java文件，并检查我们是否使用了正确的格式**。在控制台中，我们将看到已扫描文件的摘要，以及其中有多少文件失败：

![](/assets/images/2025/staticanalysis/javamavenspotlessplugin01.png)

**如我们所见，如果插件发现至少一处格式违规，构建就会失败**。向下滚动，我们会看到检测到的格式问题的显示。在本例中，我们的代码使用了制表符，而Google规范要求缩进块包含两个空格：

![](/assets/images/2025/staticanalysis/javamavenspotlessplugin02.png)

**此外，当我们执行命令“mvn spotless::apply”时，Spotless将自动修复所有违规行为**。让我们使用该命令来纠正违规行为，并将源代码与远程分支进行比较：

![](/assets/images/2025/staticanalysis/javamavenspotlessplugin03.png)

我们可以注意到，我们的源代码格式正确，现在符合Google Java标准。

## 3. 自定义格式规则

到目前为止，我们已验证代码库使用了一致的格式，并且使用了Spotless插件的最低配置。**不过，我们可以使用Eclipse Formatter Profile来配置自己的格式规则**，此Profile是一个具有标准化结构的XML文件，与各种IDE和格式插件兼容。

我们将其中一个文件添加到项目的根文件夹中，我们将其命名为tuyucheng-style.xml：

```xml
<profiles version="21">
    <profile kind="CodeFormatterProfile" name="tuyucheng-style" version="21">
        <setting id="org.eclipse.jdt.core.formatter.tabulation.char" value="space"/>
        <setting id="org.eclipse.jdt.core.formatter.use_tabs_only_for_leading_indentations" value="true"/>
        <setting id="org.eclipse.jdt.core.formatter.indentation.size" value="4"/>
        <!--   other settings...   -->
        <setting id="org.eclipse.jdt.core.formatter.enabling_tag" value="@formatter:on"/>
        <setting id="org.eclipse.jdt.core.formatter.disabling_tag" value="@formatter:off"/>
    </profile>
</profiles>
```

现在，让我们更新pom.xml并添加自定义格式化程序配置文件。我们将删除<googleJavaFormat/\>步骤，并将其替换为使用自定义XML文件中设置的<eclipse\>格式化程序：

```xml
<plugin>
    <groupId>com.diffplug.spotless</groupId>
    <artifactId>spotless-maven-plugin</artifactId>
    <version>2.43.0</version>
    <configuration>
        <java>
            <eclipse>
                <file>${project.basedir}/tuyucheng-style.xml</file>
            </eclipse>
        </java>
    </configuration>
</plugin>
```

就这样，现在我们可以重新运行“mvn spotless:check”来确保项目遵循我们自定义的约定。

## 4. 其他步骤

除了验证代码格式是否正确之外，**我们还可以使用Spotless进行静态分析并应用一些小的改进**，在插件配置中指定首选代码样式后，我们可以执行其他步骤：

```xml
<java>
    <eclipse>
        <file>${project.basedir}/tuyucheng-style.xml</file>
    </eclipse>

    <licenseHeader>
        <content>/* (C)$YEAR */</content>
    </licenseHeader>

    <importOrder/>
    <removeUnusedImports />
    <formatAnnotations />
</java>
```

**当我们执行spotless:apply时，每个“步骤”都会验证并执行一个特定的规则**：

- <licenseHeader\>检查文件是否包含正确的版权标头
- <importOrder\>和<removeUnusedImports\>确保导入是相关的并且遵循一致的顺序
- <formatAnnotations>确保类型注释与它们描述的字段位于同一行

如果我们运行该命令，我们可以预期所有这些更改将自动应用：

![](/assets/images/2025/staticanalysis/javamavenspotlessplugin04.png)

## 5. 绑定到Maven阶段

到目前为止，我们仅通过直接触发Maven目标“spotless:check”和“spotless:apply”来使用Spotless插件。**但是，我们也可以将这些目标绑定到特定的[Maven阶段](https://www.baeldung.com/maven-goals-phases)**，阶段是Maven构建生命周期中预定义的阶段，它们按特定顺序执行任务，以自动化软件构建过程。

例如，“package”阶段将编译后的代码和其他资源打包成可分发的格式，例如“Jar”或“War”文件，让我们使用此阶段与Spotless插件集成并执行“spotless:check”：

```xml
<plugin>
    <groupId>com.diffplug.spotless</groupId>
    <artifactId>spotless-maven-plugin</artifactId>
    <version>2.43.0</version>
    
    <configuration>
        <java>
            <!--  formatter and additional steps  -->
        </java>
    </configuration>
    
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
            <phase>package</phase>
        </execution>
    </executions>
</plugin>
```

因此，Spotless的检查目标将在Maven的打包阶段自动执行。**换句话说，如果源代码不符合指定的格式和样式指南，我们可以通过导致Maven构建失败来强制执行一致的代码样式**。

## 6. 总结

在本文中，我们了解了Maven Spotless插件，最初使用它来强制使用Google的Java格式对我们的项目进行静态代码分析。然后，我们转换到自定义的Eclipse Formatter Profile，并自定义了格式规则。

除了格式化之外，我们还探索了其他可配置的步骤，这些步骤可以改进代码并执行一些细微的重构。最后，我们讨论了将Spotless目标绑定到特定的Maven阶段，以确保在整个构建过程中强制执行一致的代码样式。