---
layout: post
title:  在Java中使用SOAP Web Service
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在本教程中，**我们将学习如何使用Java 11中的[JAX-WS RI](https://javaee.github.io/metro-jax-ws/)在Java中构建SOAP客户端**。

首先，我们将使用wsimport实用程序生成客户端代码，然后使用JUnit对其进行测试。

对于刚开始学习的人来说，我们对[JAX-WS的介绍](https://www.baeldung.com/jax-ws)提供了很好的背景信息。

## 2. Web Service

在构建客户端之前，我们需要一个SOAP Web Service来使用，我们将使用一个根据国家/地区名称获取国家/地区详细信息的服务。

### 2.1 实施总结

该Web服务通过CountryService接口公开，并使用[javax.xml.ws.Endpoint API](https://docs.oracle.com/javase/7/docs/api/javax/xml/ws/Endpoint.html)进行部署。接下来，我们通过运行CountryServicePublisher类作为Java应用程序来发布端点。

一旦服务器运行，Web服务WSDL可在以下位置获得：

```text
http://localhost:8888/ws/country?wsdl
```

此WSDL作为定义服务操作和数据类型的契约，使我们能够生成客户端代码。

### 2.2 理解Web服务描述语言(WSDL)

让我们看看我们的Web服务的WSDL、国家：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions <!-- namespace declarations -->
    targetNamespace="http://server.ws.soap.tuyucheng.com/" name="CountryServiceImplService">
    <types>
        <xsd:schema>
            <xsd:import namespace="http://server.ws.soap.tuyucheng.com/" 
              schemaLocation="http://localhost:8888/ws/country?xsd=1"></xsd:import>
        </xsd:schema>
    </types>
    <message name="findByName">
        <part name="arg0" type="xsd:string"></part>
    </message>
    <message name="findByNameResponse">
        <part name="return" type="tns:country"></part>
    </message>
    <portType name="CountryService">
        <operation name="findByName">
            <input wsam:Action="http://server.ws.soap.tuyucheng.com/CountryService/findByNameRequest" 
              message="tns:findByName"></input>
            <output wsam:Action="http://server.ws.soap.tuyucheng.com/CountryService/findByNameResponse" 
              message="tns:findByNameResponse"></output>
        </operation>
    </portType>
    <binding name="CountryServiceImplPortBinding" type="tns:CountryService">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="rpc"></soap:binding>
        <operation name="findByName">
            <soap:operation soapAction=""></soap:operation>
            <input>
                <soap:body use="literal" namespace="http://server.ws.soap.tuyucheng.com/"></soap:body>
            </input>
            <output>
                <soap:body use="literal" namespace="http://server.ws.soap.tuyucheng.com/"></soap:body>
            </output>
        </operation>
    </binding>
    <service name="CountryServiceImplService">
        <port name="CountryServiceImplPort" binding="tns:CountryServiceImplPortBinding">
            <soap:address location="http://localhost:8888/ws/country"></soap:address>
        </port>
    </service>
</definitions>
```

WSDL描述了Web服务的结构和操作，它定义了一个操作findByName，该操作接收表示国家/地区名称的字符串输入并返回国家/地区对象。

此country对象包括国家name、capital、currency和population等详细信息，currency字段进一步定义为枚举，可能值为USD、EUR和INR。

这种全面的结构使得客户端能够了解与Web服务有效交互的输入和输出要求。

### 2.3 数据模型模式(XSD)

country类型和相关数据在以下XSD中定义：

```text
http://localhost:8888/ws/country?xsd=1
```

下面是XSD的一个示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema <!-- namespace declarations -->
    targetNamespace="http://server.ws.soap.tuyucheng.com/">
    <xs:complexType name="country">
        <xs:sequence>
            <xs:element name="capital" type="xs:string" minOccurs="0"></xs:element>
            <xs:element name="currency" type="tns:currency" minOccurs="0"></xs:element>
            <xs:element name="name" type="xs:string" minOccurs="0"></xs:element>
            <xs:element name="population" type="xs:int"></xs:element>
        </xs:sequence>
    </xs:complexType>
    <xs:simpleType name="currency">
        <xs:restriction base="xs:string">
            <xs:enumeration value="EUR"></xs:enumeration>
            <xs:enumeration value="INR"></xs:enumeration>
            <xs:enumeration value="USD"></xs:enumeration>
        </xs:restriction>
    </xs:simpleType>
</xs:schema>
```

有了WSDL和XSD，我们就有了生成客户端代码并与服务交互所需的所有信息，让我们继续构建客户端。

## 3. 使用wsimport生成客户端代码

### 3.1 生成客户端代码

为了生成客户端代码，我们需要将[jakarta.xml.ws-api](https://mvnrepository.com/artifact/jakarta.xml.ws/jakarta.xml.ws-api)、[jaxws-rt](https://mvnrepository.com/artifact/com.sun.xml.ws/jaxws-rt)和[jaxws-ri](https://mvnrepository.com/artifact/com.sun.xml.ws/jaxws-ri)依赖添加到pom.xml文件中：

```xml
<dependencies>
    <dependency>
        <groupId>jakarta.xml.ws</groupId
        <artifactId>jakarta.xml.ws-api</artifactId
        <version>3.0.1</version>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.ws</groupId>
        <artifactId>jaxws-rt</artifactId>
        <version>3.0.0</version
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.ws</groupId>
        <artifactId>jaxws-ri</artifactId>
        <version>2.3.1</version
        <type>pom</type>
    </dependency>
</dependencies>

<build>
    <plugins>        
        <plugin>
            <groupId>com.sun.xml.ws</groupId>
            <artifactId>jaxws-maven-plugin</artifactId>
            <version>3.0.2</version>
            <configuration>
                <wsdlUrls>
                    <wsdlUrl>http://localhost:8888/ws/country?wsdl</wsdlUrl>
                </wsdlUrls>
                <keep>true</keep>
                <packageName>cn.tuyucheng.taketoday.soap.ws.client.generated</packageName>
                <sourceDestDir>src/main/java</sourceDestDir>
            </configuration>
        </plugin>
    </plugins>
</build>
```

接下来，我们可以运行以下Maven命令来生成客户端代码：

```shell
mvn clean jaxws:wsimport
```

### 3.2 生成的POJO

wsimport工具根据XSD模式生成POJO类Country.java，该类使用JAXB注解进行标注，以进行XML编组和解组：

```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "country", propOrder = { "capital", "currency", "name", "population" })
public class Country {
    protected String capital;
    @XmlSchemaType(name = "string")
    protected Currency currency;
    protected String name;
    protected int population;
    // standard getters and setters
}

@XmlType(name = "currency")
@XmlEnum
public enum Currency {
    EUR, INR, USD;
    public String value() {
        return name();
    }
    public static Currency fromValue(String v) {
        return valueOf(v);
    }
}
```

### 3.3 CountryService接口

CountryService接口充当实际Web服务的代理，它声明了服务器中定义的findByName()方法：

```java
@WebService(name = "CountryService", targetNamespace = "http://server.ws.soap.tuyucheng.com/")
@SOAPBinding(style = SOAPBinding.Style.RPC)
public interface CountryService {
    @WebMethod
    @WebResult(partName = "return")
    public Country findByName(@WebParam(name = "arg0") String arg0);
}
```

### 3.4 CountryServiceImplService

CountryServiceImplService类扩展了javax.xml.ws.Service，它提供了与Web服务交互的方法。

其注解[WebServiceClient](https://docs.oracle.com/javase/7/docs/api/javax/xml/ws/WebServiceClient.html)表示它是一个服务的客户端视图：

```java
@WebServiceClient(name = "CountryServiceImplService",
        wsdlLocation = "http://localhost:8888/ws/country?wsdl")
public class CountryServiceImplService extends Service {
    public CountryServiceImplService() {
        super(QName("http://server.ws.soap.tuyucheng.com/", "CountryServiceImplService"));
    }
    @WebEndpoint(name = "CountryServiceImplPort")
    public CountryService getCountryServiceImplPort() {
        return super.getPort(CountryService.class);
    }
}
```

要调用Web服务，我们使用CountryServiceImplService获取代理实例，这样我们就可以像在本地调用一样调用findByName()方法，从而消除远程通信的复杂性。

## 4. 测试客户端

接下来，为了测试客户端，我们将编写一个JUnit测试以使用生成的客户端代码连接到Web服务。

### 4.1 设置服务代理

在运行测试之前，我们需要获取客户端的CountryService的代理实例：

```java
@BeforeClass
public static void setup() {
    CountryServiceImplService service = new CountryServiceImplService();
    CountryService countryService = service.getCountryServiceImplPort();
}
```

### 4.2 编写测试

现在我们可以编写测试来检查findByName()方法的功能。

```java
@Test
public void givenCountryService_whenCountryIndia_thenCapitalIsNewDelhi() {
    assertEquals("New Delhi", countryService.findByName("India").getCapital());
}

@Test
public void givenCountryService_whenCountryFrance_thenPopulationCorrect() {
    assertEquals(66710000, countryService.findByName("France").getPopulation());
}

@Test
public void givenCountryService_whenCountryUSA_thenCurrencyUSD() {
    assertEquals(Currency.USD, countryService.findByName("USA").getCurrency());
}
```

设置代理实例后，调用远程服务的方法就像调用本地方法一样简单。

代理的findByName()方法返回一个Country对象，我们可以轻松访问其断言属性，例如capital、population和currency。

## 5. 总结

在本文中，我们演示如何使用JAX-WS RI和Java 11的wsimport实用程序在Java中使用SOAP Web服务。

或者，我们可以使用其他JAX-WS实现(例如Apache CXF、Apache Axis2和Spring)来执行相同操作。