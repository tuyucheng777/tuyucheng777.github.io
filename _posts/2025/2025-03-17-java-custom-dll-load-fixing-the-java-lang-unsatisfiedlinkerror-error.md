---
layout: post
title:  自定义DLL加载 – 修复“java.lang.UnsatisfiedLinkError”错误
category: java
copyright: java
excerpt: Java Native
---

## 1. 简介

在本快速教程中，我们将探讨UnsatisfiedLinkError的不同原因和解决方案。这是使用本机库时遇到的常见且令人沮丧的错误，解决此错误需要彻底了解其原因和适当的纠正措施。

我们将讨论诸如库和方法名称不正确、缺少库目录规范、与类加载器冲突、不兼容的架构以及Java安全策略的作用等场景。

## 2. 场景和设置

**我们将创建一个简单的类，说明加载外部库时可能出现的错误**。考虑到我们在Linux上，让我们加载一个名为“libtest.so”的简单库并调用其test()方法：

```java
public class JniUnsatisfiedLink {

    public static final String LIB_NAME = "test";

    public static void main(String[] args) {
        System.loadLibrary(LIB_NAME);
        new JniUnsatisfiedLink().test();
    }

    public native String test();

    public native String nonexistentDllMethod();
}
```

**通常，我们希望将库加载到静态块中，以确保它只加载一次。但是，为了更好地模拟错误，我们在main()方法中加载它**。在本例中，我们的库只包含一个有效方法test()，它返回一个String。**我们还声明了一个nonexistentDllMethod()来查看我们的应用程序的行为**。

## 3. 未指定库目录

UnsatisfiedLinkError的最直接原因是我们的库不在Java期望库所在的任何目录中，这可能是在系统变量中，例如Unix或Linux上的LD_LIBRARY_PATH或Windows上的PATH。**也可以使用System.load()而不是loadLibrary()来使用我们库的完整路径**：

```java
System.load("/full/path/to/libtest.so");
```

**但是，为了避免使用特定于系统的解决方案，我们可以设置java.library.path VM属性**，此属性接收一个或多个包含我们需要加载的库的目录路径：

```shell
-Djava.library.path=/any/library/dir
```

目录分隔符取决于我们的操作系统，对于Unix或Linux，它是冒号；对于Windows，它是分号。

## 4. 库名称或权限不正确

**导致UnsatisfiedLinkError的最常见原因可能是使用了错误的库名称**，这是因为Java为了使代码尽可能与平台无关，对库名称做了一些假设：

- 对于[Windows](https://hg.openjdk.org/jdk8/jdk8/jdk/file/687fd7c7986d/src/windows/javavm/export/jvm_md.h#l42)，它假定库文件名以“.dll”结尾。
- 对于大多数类[Unix](https://hg.openjdk.org/jdk8/jdk8/jdk/file/687fd7c7986d/src/solaris/javavm/export/jvm_md.h#l43)系统，它假定一个“lib”前缀和一个“.so”扩展名。
- 最后，具体对于[Mac](https://hg.openjdk.org/jdk8/jdk8/jdk/file/687fd7c7986d/src/macosx/javavm/export/jvm_md.h#l43)，它采用“lib”前缀和“.dylib”(以前称为“.jnilib”)扩展名。

因此，如果我们包含任何这些前缀或后缀，我们就会收到错误：

```java
@Test
public void whenIncorrectLibName_thenLibNotFound() {
    String libName = "lib" + LIB_NAME + ".so";

    Error error = assertThrows(UnsatisfiedLinkError.class, () -> System.loadLibrary(libName));

    assertEquals(
        String.format("no %s in java.library.path", libName), 
        error.getMessage()
    );
}
```

顺便说一句，这使得我们无法尝试加载为不同于我们运行应用程序的平台构建的库。在这种情况下，如果我们希望我们的应用程序是多平台的，我们必须为所有平台提供二进制文件。**如果我们在Linux环境中的库目录中只有一个“test.dll”，则System.loadLibrary(“test”)将导致相同的错误**。

类似地，如果我们在loadLibrary()中包含路径分隔符，我们将会收到错误：

```java
@Test
public void whenLoadLibraryContainsPathSeparator_thenErrorThrown() {
    String libName = "/" + LIB_NAME;

    Error error = assertThrows(UnsatisfiedLinkError.class, () -> System.loadLibrary(libName));

    assertEquals(
        String.format("Directory separator should not appear in library name: %s", libName), 
        error.getMessage()
    );
}
```

最后，如果我们的库目录权限不足，也会导致同样的错误。**例如，在Linux中，我们至少需要“execute”权限。另一方面，如果我们的文件至少没有“read”权限，我们将收到类似以下消息**：

```text
java.lang.UnsatisfiedLinkError: /path/to/libtest.so: cannot open shared object file: Permission denied
```

## 5. 方法名称/用法不正确

如果我们声明的本机方法与本机源代码中声明的任何方法都不匹配，我们也会收到错误，**但仅当我们尝试调用不存在的方法时才会出现**：

```java
@Test
public void whenUnlinkedMethod_thenErrorThrown() {
    System.loadLibrary(LIB_NAME);

    Error error = assertThrows(UnsatisfiedLinkError.class, () -> new JniUnsatisfiedLink().nonexistentDllMethod());

    assertTrue(error.getMessage()
        .contains("JniUnsatisfiedLink.nonexistentDllMethod"));
}
```

请注意， loadLibrary()中没有引发异常。

## 6. 库已被另一个类加载器加载

如果我们在同一个Web应用服务器(如Tomcat)中的不同Web应用中加载同一个库，则很可能会发生这种情况。然后，我们会收到错误：

```text
Native Library libtest.so already loaded in another classloader
```

或者，如果它处于加载过程的中间，我们将得到：

```text
Native Library libtest.so is being loaded in another classloader
```

**解决此问题最简单的方法是将用于加载库的代码放入Web应用服务器共享目录中的JAR中**，例如，在Tomcat中，该目录为“<tomcat home\>/lib”。

## 7. 不兼容的架构

使用旧库时最有可能出现这种情况，我们无法加载为不同于我们运行应用程序的架构而编译的库-**例如，如果我们尝试在64位系统上加载32位库**：

```java
@Test
public void whenIncompatibleArchitecture_thenErrorThrown() {
    Error error = assertThrows(UnsatisfiedLinkError.class, () -> System.loadLibrary(LIB_NAME + "32"));

    assertTrue(error.getMessage()
        .contains("wrong ELF class: ELFCLASS32"));
}
```

在上面的例子中，我们将库与[32位标志](https://www.baeldung.com/linux/compile-32-bit-binary-on-64-bit-os)链接起来以进行测试。以下是一些补充说明：

- 如果我们尝试通过重命名文件来加载不同平台的DLL，则会发生类似的错误。然后，我们的错误将包含“invalid ELF header”消息。
- 如果我们尝试在不兼容的平台上加载我们的库，那么就找不到该库。

## 8. 文件损坏

**尝试加载损坏的文件时，它始终会导致UnsatisfiedLinkError**。为了说明这一点，让我们看看尝试加载空文件时会发生什么(请注意，此测试针对单个库路径进行了简化，并考虑了Linux环境)：

```java
@Test
public void whenCorruptedFile_thenErrorThrown() {
    String libPath = System.getProperty("java.library.path");

    String dummyLib = LIB_NAME + "-dummy";
    assertTrue(new File(libPath, "lib" + dummyLib + ".so").isFile());
    Error error = assertThrows(UnsatisfiedLinkError.class, () -> System.loadLibrary(dummyLib));

    assertTrue(error.getMessage().contains("file too short"));
}
```

为了避免这种情况，我们通常将[MD5校验](https://www.baeldung.com/java-md5-checksum-file)和与二进制文件一起分发，以便检查其完整性。

## 9. Java安全策略

**如果我们使用[Java策略](https://www.baeldung.com/java-security-manager)文件，我们需要为loadLibrary()和我们的库名称授予RuntimePermission**：

```plaintext
grant {
    permission java.lang.RuntimePermission "loadLibrary.test";
};
```

否则，当我们尝试加载我们的库时，我们会收到类似这样的错误：

```text
java.security.AccessControlException: access denied ("java.lang.RuntimePermission" "loadLibrary.test")
```

请注意，要使自定义策略文件生效，我们需要指定我们要使用安全管理器：

```shell
-Djava.security.manager
```

## 10. 总结

在本文中，我们探讨了解决Java应用程序中UnsatisfiedLinkError的解决方案。我们讨论了此错误的常见原因，并提供了有效解决这些错误的见解。通过实施这些见解并根据应用程序的特定需求进行定制，我们可以有效地解决UnsatisfiedLinkError的发生。