---
layout: post
title:  Gradle中的implementation和compile之间的区别
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

**[Gradle](https://www.baeldung.com/gradle) 8提供了三种主要的依赖配置，用于配置软件项目中的依赖：compileOnly、implementation和api**。虽然这些依赖配置可能存在一些重叠，但它们的含义和用法却有所不同。因此，了解它们之间的差异对于有效地使用它们至关重要。

在本教程中，我们将讨论Gradle 8中这三种依赖管理配置之间的区别。此外，我们将提供有效依赖管理的最佳实践。

## 2. Gradle依赖管理中的compileOnly是什么？

**当我们将依赖配置为compileOnly时，Gradle只会将其添加到编译时classpath中**，而不会将其添加到构建输出中。因此，Gradle会在编译期间使依赖可用，但在程序执行时不会在运行时可用。compileOnly依赖配置有助于减小构建输出的大小；从而减少构建时间和内存使用。

让我们考虑一个简单的Gradle脚本：

```shell
dependencies {
    compileOnly 'org.hibernate.orm:hibernate-core:6.6.3.Final'
    testCompileOnly 'org.junit.jupiter:junit-jupiter:5.11.4'
}
```

在此示例中，我们使用compileOnly配置从org.hibernate.orm组中添加了对版本6.6.3.Final的[hibernate-core](https://mvnrepository.com/artifact/org.hibernate.orm/hibernate-core)库的依赖，此Gradle仅在编译时类路径中包含此依赖。

我们还使用testCompileOnly配置添加了对[JUnit测试框架](https://docs.gradle.org/current/userguide/java_testing.html)5.11.4版本的依赖，Gradle仅在测试编译类路径(用于编译测试)中包含此依赖，Gradle不会在测试运行时类路径(用于运行测试)中包含此依赖。

## 3. Gradle依赖管理中的implementation是什么？

**当我们将依赖配置为implementation时，Gradle会将其添加到编译时和运行时的类路径中**。这意味着Gradle不仅在编译时提供该依赖，在程序执行时也提供该依赖，Gradle会将依赖打包到构建输出中。但是，与仅编译期的配置相比，implementation依赖配置会增加构建输出的大小，从而增加构建时间和随之而来的内存使用。

让我们考虑一个简单的Gradle脚本：

```shell
dependencies {
    implementation 'org.hibernate.orm:hibernate-core:6.6.3.Final'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.11.4'
}
```

在此示例中，我们使用implementation关键字添加了对org.hibernate.orm组中版本为6.6.3.Final的hibernate-core库的依赖，Gradle会将此依赖添加到编译时类路径和运行时类路径中。

我们还使用testImplementation关键字添加了对JUnit测试框架5.11.4版本的依赖，Gradle将此依赖同时添加到测试编译时类路径和测试运行时类路径中，使其可用于编译和运行测试。

## 4. compileOnly和implementation的区别

**Gradle中compileOnly和implementation的主要区别在于，compileOnly仅在编译时类路径中包含依赖，而implementation在编译时和运行时类路径中都包含依赖**。这意味着当我们将依赖配置为implementation时，Gradle会在运行时使其可用，而当我们将依赖配置为compileOnly时，Gradle不会在运行时使其可用。

此外，compileOnly更好地区分了编译时和运行时类路径，从而更轻松地管理依赖并避免版本冲突。选择正确的依赖配置会影响项目性能、构建时间以及与其他依赖的兼容性。

## 5. Gradle依赖管理中的api是什么？

**当我们将依赖配置为api时，Gradle会将其添加到编译时和运行时类路径，并将其包含在已发布的API中**。这意味着Gradle在程序编译、程序运行以及依赖模块程序编译时都会使依赖可用，Gradle将依赖打包到构建输出中。此外，Gradle还会将依赖包含在已发布的API中。与implementation配置相比，api依赖配置会增加构建时间并增加相应的内存使用量。

让我们考虑一个简单的Gradle脚本：

```shell
dependencies {
    api 'org.hibernate.orm:hibernate-core:6.6.3.Final'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter:5.11.4'
}
```

在此示例中，我们使用api关键字从org.hibernate.orm组中添加了对版本6.6.3.Final的hibernate-core库的依赖，此依赖将包含在编译时类路径和运行时类路径中。此外，Gradle会将该依赖包含在已发布的API中。

我们还使用testRuntimeOnly关键字添加了对JUnit测试框架5.11.4版本的依赖，Gradle仅在测试运行时类路径中包含此依赖，而不在已发布的API中包含。

## 6. implementation和api之间的区别

**Gradle中，implementation和api的主要区别在于，implementation不会将依赖以传递方式导出到依赖于此模块的其他模块**。相反，api会将依赖以传递方式导出到其他模块。这意味着使用api配置的依赖在运行时和编译时都可供依赖于此模块的其他模块使用，但是，使用implementation配置的依赖仅在运行时才可供依赖于此模块的其他模块使用。

因此，我们应该谨慎使用api配置，因为它会显著增加构建时间，相比于implementation配置。例如，如果api依赖更改了其外部API，Gradle会重新编译所有可以访问该依赖的模块，即使在编译时也是如此。相反，如果implementation依赖更改了其外部API，即使模块中的软件未运行，Gradle也不会重新编译所有模块，因为依赖模块在编译时无法访问该依赖。

Gradle提供了另外两种依赖配置：compileOnlyApi和runtimeOnly。当我们将依赖配置为compileOnlyApi时，Gradle会将其仅用于编译，就像compileOnly一样。此外，Gradle会将其包含在已发布的API中。当我们将依赖配置为runtimeOnly时，Gradle会将其包含在运行时的构建输出中，尽管它在编译时不会使用它。

## 7. Gradle依赖管理的最佳实践

为了确保Gradle中有效的依赖管理，我们应该考虑一些最佳实践：

- 默认使用implementation配置。
- 如果你不想在构建输出中打包依赖，请使用compileOnly配置；一个示例用例是仅包含编译时注解的软件库，这些注解通常用于生成代码，但在运行时不需要。
- 避免使用api配置，因为它会导致更长的构建时间并增加内存使用量。
- 使用依赖的特定版本而不是动态版本控制来确保构建过程中的行为一致。
- 保持依赖关系图尽可能小，以降低复杂性并缩短构建时间。
- 定期检查依赖的更新，并根据需要进行更新，以确保项目使用最新、最安全的版本。
- 使用依赖锁定来确保构建在不同的机器和环境中可重复且一致。

## 8. 总结

在本文中，我们讨论了Gradle中compileOnly、implementation和api依赖配置之间的区别。此外，我们还讨论了它们如何影响项目中依赖的范围，并提供了Gradle依赖管理的示例和最佳实践。

通过遵循这些实践，我们可以确保我们的构建可靠、高效且易于维护。