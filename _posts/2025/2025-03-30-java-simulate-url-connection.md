---
layout: post
title:  在Java中Mock URL连接
category: libraries
copyright: libraries
excerpt: Mock
---

## 1. 概述

UrlConnection是一个抽象类，它提供了一个处理[Web资源](https://www.baeldung.com/java-http-request)的接口，例如从URL检索数据并向其发送数据。

**在编写单元测试时，我们通常需要一种方法来模拟网络连接和响应，而无需实际发出实际的网络请求**。

在本教程中，我们将介绍在Java中Mock URL连接的几种方法。

## 2. 一个简单的URL获取器类

在本教程中，我们测试的重点是一个简单的URL获取器类：

```java
public class UrlFetcher {

    private URL url;

    public UrlFetcher(URL url) throws IOException {
        this.url = url;
    }

    public boolean isUrlAvailable() throws IOException {
        return getResponseCode() == HttpURLConnection.HTTP_OK;
    }

    private int getResponseCode() throws IOException {
        HttpURLConnection con = (HttpURLConnection) this.url.openConnection();
        return con.getResponseCode();
    }
}
```

为了演示目的，我们有一个公共方法isUrlAvailable()，它指示给定地址的URL是否可用。返回值基于我们收到的HTTP响应消息的状态码。

## 3. 使用纯Java进行单元测试

通常，使用Mock的第一个方法是使用第三方测试框架。但是，在某些情况下，这可能不是一个可行的选择。

**幸运的是，URL类提供了一种机制，允许我们提供一个知道如何建立连接的自定义处理程序**。我们可以使用它来提供我们的处理程序，它将返回一个虚拟连接对象和响应。

### 3.1 支持类

对于这种方法，我们需要几个支持类。让我们首先定义一个MockHttpURLConnection：

```java
public class MockHttpURLConnection extends HttpURLConnection {

    protected MockHttpURLConnection(URL url) {
        super(url);
    }

    @Override
    public int getResponseCode() {
        return responseCode;
    }

    public void setResponseCode(int responseCode) {
        this.responseCode = responseCode;
    }

    @Override
    public void disconnect() {
    }

    @Override
    public boolean usingProxy() {
        return false;
    }

    @Override
    public void connect() throws IOException {
    }
}
```

我们可以看到，这个类是一个简单的扩展，具有HttpURLConnection类的最小实现，这里最重要的部分是我们提供了一种设置和获取HTTP响应码的机制。

接下来，我们需要一个Mock流处理程序来返回我们新创建的MockHttpURLConnection：

```java
public class MockURLStreamHandler extends URLStreamHandler {

    private MockHttpURLConnection mockHttpURLConnection;

    public MockURLStreamHandler(MockHttpURLConnection mockHttpURLConnection) {
        this.mockHttpURLConnection = mockHttpURLConnection;
    }

    @Override
    protected URLConnection openConnection(URL url) throws IOException {
        return this.mockHttpURLConnection;
    }
}
```

最后，我们需要提供一个流处理程序工厂，它将返回我们新创建的流处理程序：

```java
public class MockURLStreamHandlerFactory implements URLStreamHandlerFactory {

    private MockHttpURLConnection mockHttpURLConnection;

    public MockURLStreamHandlerFactory(MockHttpURLConnection mockHttpURLConnection) {
        this.mockHttpURLConnection = mockHttpURLConnection;
    }

    @Override
    public URLStreamHandler createURLStreamHandler(String protocol) {
        return new MockURLStreamHandler(this.mockHttpURLConnection);
    }
}
```

### 3.2 综合起来

现在我们已经准备好支持类，我们可以继续编写第一个单元测试：

```java
private static MockHttpURLConnection mockHttpURLConnection;

@BeforeAll
public static void setUp() {
    mockHttpURLConnection = new MockHttpURLConnection(null);
    URL.setURLStreamHandlerFactory(new MockURLStreamHandlerFactory(mockHttpURLConnection));
}

@Test
void givenMockedUrl_whenRequestSent_thenIsUrlAvailableTrue() throws Exception {
    mockHttpURLConnection.setResponseCode(HttpURLConnection.HTTP_OK);
    URL url = new URL("https://www.tuyucheng.com/");

    UrlFetcher fetcher = new UrlFetcher(url);
    assertTrue(fetcher.isUrlAvailable(), "Url should be available: ");
}
```

让我们来看看测试的关键部分：

- 首先，我们首先定义setUp()方法，**在其中创建MockHttpURLConnection并通过静态方法setURLStreamHandlerFactory()将其注入到URL类中**。
- 现在我们可以开始编写测试主体了。首先，我们需要使用mockHttpURLConnection变量上的setResponseCode()方法设置预期的响应码。
- 然后我们可以创建一个新的URL并构造我们的UrlFetcher，最后对isUrlAvailable()方法进行断言。

当我们运行测试时，无论网址是否可用，我们都应该始终获得相同的行为。确保这一点的一个好方法是关闭WiFi或网络连接，并检查测试是否仍然以完全相同的方式运行。

### 3.3 这种方法的问题

虽然此解决方案有效并且不依赖于第三方库，但由于多种原因，它有点麻烦。

首先，我们需要创建几个Mock支持类，随着测试需求变得越来越复杂，我们的Mock对象也会变得越来越复杂。例如，如果我们需要开始Mock不同的响应主体。

**同样，我们的测试有一些重要的设置，我们将静态方法调用与URL类的新实例混合在一起**。这很容易让人困惑，并且可能会导致后续出现意想不到的结果。

## 4. 使用Mockito

在下一部分中，我们将看到如何使用著名的单元测试框架[Mockito](https://www.baeldung.com/tag/mockito)简化测试。

首先，我们需要将[mockito](https://mvnrepository.com/artifact/org.mockito/mockito-core)依赖添加到项目中：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
```

现在可以定义我们的测试：

```java
@Test
void givenMockedUrl_whenRequestSent_thenIsUrlAvailableFalse() throws Exception {
    HttpURLConnection mockHttpURLConnection = mock(HttpURLConnection.class);
    when(mockHttpURLConnection.getResponseCode()).thenReturn(HttpURLConnection.HTTP_NOT_FOUND);

    URL mockURL = mock(URL.class);
    when(mockURL.openConnection()).thenReturn(mockHttpURLConnection);

    UrlFetcher fetcher = new UrlFetcher(mockURL);
    assertFalse(fetcher.isUrlAvailable(), "Url should be available: ");
}
```

这次，我们使用Mockito的mock方法创建一个模拟URL连接。**然后，我们将Mock对象配置为在调用其openConnection方法时返回模拟HTTP URL连接**。当然，我们的模拟HTTP连接已经包含存根响应代码。

我们应该注意，对于低于4.8.0的Mockito版本，我们在运行此测试时可能会收到错误：

```text
org.mockito.exceptions.base.MockitoException: 
Cannot mock/spy class java.net.URL
Mockito cannot mock/spy because :
 - final class
```

发生这种情况是因为URL是一个最final类，而在以前的Mockito版本中，无法直接[Mock final类和方法](https://www.baeldung.com/mockito-final)。

为了解决这个问题，我们可以简单地在pom.xml中添加一个额外的[依赖](https://mvnrepository.com/artifact/org.mockito/mockito-inline)：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-inline</artifactId>
    <version>5.2.0</version>
    <scope>test</scope>
</dependency>
```

现在我们的测试将成功运行。

## 5. 使用JMockit

在我们的最后一个例子中，为了完整性，我们将研究另一个名为[JMockit](https://www.baeldung.com/tag/jmockit)的测试库。

首先，我们需要将[jmockit](https://mvnrepository.com/artifact/org.jmockit/jmockit)依赖添加到我们的项目中：

```xml
<dependency> 
    <groupId>org.jmockit</groupId> 
    <artifactId>jmockit</artifactId> 
    <version>1.49</version>
</dependency>
```

现在可以继续定义我们的测试类：

```java
@ExtendWith(JMockitExtension.class)
class UrlFetcherJMockitUnitTest {

    @Test
    void givenMockedUrl_whenRequestSent_thenIsUrlAvailableTrue(@Mocked URL anyURL,
                                                               @Mocked HttpURLConnection mockConn) throws Exception {
        new Expectations() {{
            mockConn.getResponseCode();
            result = HttpURLConnection.HTTP_OK;
        }};

        UrlFetcher fetcher = new UrlFetcher(new URL("https://www.tuyucheng.com/"));
        assertTrue(fetcher.isUrlAvailable(), "Url should be available: ");
    }
}
```

JMockit最强大的一点就是它的可表达性，**为了创建Mock并定义其行为，我们只需直接定义它们，而不是从模拟API调用方法**。

## 6. 总结

在本文中，我们了解了几种Mock URL连接的方法，以便编写不依赖任何外部服务的独立单元测试。首先，我们查看了一个使用原生Java的示例，然后探索了使用Mockito和JMockit的另外两个选项。