---
layout: post
title:  Gradle构建缓存基础知识
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

构建缓存可以加快代码构建过程，从而提高开发人员的工作效率。在本文中，我们将学习Gradle构建缓存的基础知识。

## 2. 什么是Gradle构建缓存？

Gradle构建缓存是一种半永久性存储，用于保存构建任务的输出，它允许重用之前构建中已生成的工件。**Gradle构建缓存背后的指导原则是，只要输入保持不变，就应该避免重新构建已构建的任务，这样可以缩短后续构建的完成时间**。

**在Gradle中，构建缓存键唯一地标识一个工件或任务输出**。在执行任务之前，Gradle会通过对任务的每个输入进行哈希处理来计算缓存键。然后，它会检查远程或本地缓存，以检查与计算出的缓存键对应的任务输出是否已存在。如果不存在，则执行该任务。否则，Gradle会重用现有的任务输出。

现在，让我们看看两种不同的Gradle构建缓存。

### 2.1 本地构建缓存

本地构建缓存使用系统目录来存储任务输出，**默认位置是Gradle用户主目录，指向$USER_HOME/.gradle/caches**。每次我们在系统中运行构建时，构建文件都会存储在这里，它默认启用，并且其位置可配置。

### 2.2 远程构建缓存

远程缓存是一个共享的构建缓存，通过HTTP读取和写入远程缓存。远程缓存最常见的用例之一是持续集成构建，每次清除构建时，CI服务器都会填充远程缓存。因此，更改的组件将不会被重新构建，从而加快了CI构建速度。此外，任务输出也可以在CI代理之间共享；**远程缓存默认不启用**。

当远程和本地缓存都启用时，构建输出会首先在本地缓存中检查。如果本地缓存中不存在该输出，则会从远程缓存下载并存储在本地缓存中。然后，在下一次构建时，将从本地缓存中获取相同的任务输出，以加快构建过程。

## 3. 配置Gradle构建缓存

**我们可以通过在settings.gradle文件中提供Settings.build-cache块来配置缓存**，在这里，我们使用[Groovy闭包](https://www.baeldung.com/groovy-closures#:~:text=A%20closure%20is%20an%20anonymous,its%20local%20variables%20%E2%80%94%20during%20execution.)来编写配置，让我们看看如何配置不同类型的缓存。

### 3.1 配置本地构建缓存

我们在settings.gradle文件中添加本地构建缓存配置：

```groovy
buildCache {
    local {
        directory = new File(rootDir, 'build-cache')
        removeUnusedEntriesAfterDays = 30
    }
}
```

在上面的代码块中，directory对象表示存储构建输出的位置。这里，rootDir变量表示项目的根目录，我们也可以更改目录对象以指向其他位置。

**为了节省空间，本地构建缓存还会定期删除在指定时间段内未使用的条目**。属性removeUnusedEntriesAfterDays用于配置在多少天后从本地缓存中删除未使用的构建项，其默认值为7天。

**我们也可以通过删除$USER_HOME/.gradle/caches文件夹中的条目来手动清理缓存**，在Linux系统上，我们可以使用rm命令来清理该目录：

```shell
rm -r $HOME/.gradle/caches
```

我们还可以配置一些额外的属性，例如，enabled是一个布尔属性，表示是否启用本地缓存。**如果push属性设置为true，则构建输出将存储在缓存中。对于本地构建缓存，其默认值为true**。

### 3.2 配置远程缓存

让我们在settings.gradle文件中添加buildCache块来配置远程缓存。

对于远程缓存，我们需要以URL的形式提供位置以及访问它的用户名和密码：

```groovy
buildCache {
    remote(HttpBuildCache) {
        url = 'https://example.com:8123/cache/'
        credentials {
            username = 'build-cache-user-name'
            password = 'build-cache-password'
        }
    }
}
```

## 4. 使用Gradle构建缓存的优势

现在让我们看看使用Gradle构建缓存的一些好处：

- 它可以通过减少构建时间来提高开发人员的工作效率。
- 如果输入没有改变，任务输出就会被重复使用，它还可以帮助减少在版本控制分支之间来回切换时的构建时间。
- 使用远程缓存可以显著减少CI代理生成构建所需的工作量，这也减少了CI构建所需的基础设施。
- 减少CI机器上的网络使用量，因为远程缓存的结果将存储在本地缓存中。
- 远程缓存可以帮助开发人员共享结果，这样就无需在本地重新构建同一项目上其他开发人员所做的更改。
- 远程缓存还可以实现CI代理之间的构建共享。
- 随着构建时间的缩短，反馈周期也会缩短，因此，构建频率会更高；最终，这可以提高软件质量。

## 5. 总结

在本文中，我们了解了Gradle构建缓存以及它如何加速构建过程，我们还探讨了它的不同类型及其配置。最后，我们讨论了Gradle构建缓存的优势。