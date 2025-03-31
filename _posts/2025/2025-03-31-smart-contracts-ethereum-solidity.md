---
layout: post
title:  使用Solidity创建和部署智能合约
category: libraries
copyright: libraries
excerpt: Solidity
---

## 1. 概述

运行智能合约的能力使得以太坊区块链如此受欢迎并具有颠覆性。

在解释什么是智能合约之前，我们先来了解一下区块链的定义：

> 区块链是一个公共数据库，用于永久保存数字交易记录。它是一种无需信任的交易系统，在这个框架中，个人可以进行点对点交易，而无需信任第三方或彼此。

让我们看看如何使用Solidity在以太坊上创建智能合约：

## 2. 以太坊

以太坊是一个允许人们高效地使用区块链技术编写去中心化应用程序的平台。

去中心化应用程序(Dapp)是一种供处于交互不同方的人员和组织在没有任何中心化中介的情况下进行交流的工具。Dapp的早期示例包括BitTorrent(文件共享)和比特币(货币)。

我们可以将以太坊描述为具有内置编程语言的区块链。

### 2.1 以太坊虚拟机(EVM)

从实际角度来看，EVM可以被视为一个大型、分散的系统，包含数百万个称为账户的对象，它们可以维护内部数据库、执行代码并相互通信。

第一种类型的账户可能是使用网络的普通用户最熟悉的，它的名字是EOA(外部拥有账户)；它用于传输价值(例如[Ether](https://www.ethereum.org/))，并由私钥控制。

另一方面，还有另一种类型的账户，即合约。让我们继续看看这是什么：

## 3. 什么是智能合约？

**[智能合约](https://www.baeldung.com/cs/smart-contracts-blokchain)是一个独立的脚本，通常用Solidity编写，编译为二进制或JSON并部署到区块链上的特定地址**。就像我们可以通过HttpRequest调用RESTful API的特定URL端点来执行某些逻辑一样，我们可以通过提交正确的数据以及调用已部署和编译的Solidity函数所需的以太坊，在特定地址执行已部署的智能合约。

从商业角度来看，**这意味着智能合约函数本质上可以货币化**(类似于AWS Lambda函数，它允许用户按计算周期而不是按实例付费)。重要的是，智能合约函数不需要花费以太坊来运行。

简单来说，我们可以将智能合约视为存储在区块链网络中的代码集合，它定义了使用合约的各方同意的条件。

这使得开发人员能够创造尚未发明的东西，想一想-不需要中间人，也没有交易对手风险。我们可以创建新的市场，存储债务或承诺的登记，并确保我们拥有验证交易的网络共识。

任何人都可以将智能合约部署到去中心化数据库中，费用与包含代码的存储大小成比例。希望使用智能合约的节点必须以某种方式向网络的其余部分表明其参与的结果。

### 3.1 Solidity

以太坊使用的主要语言是[Solidity](https://solidity.readthedocs.io/en/develop/)，这是一种类似于Javascript的语言，专门用于编写智能合约。Solidity是静态类型的，支持继承、库和复杂的用户定义类型等功能。

Solidity编译器将代码转换为EVM字节码，然后可以将其作为部署交易发送到以太坊网络。此类部署的交易费用比智能合约交互更高，必须由合约所有者支付。

## 4. 使用Solidity创建智能合约

Solidity合约的第一行设置了源代码版本，这是为了确保合约不会因为编译器版本发生变化而突然出现异常。

```java
pragma solidity ^0.4.0;
```

在我们的示例中，合约的名称是Greeting，我们可以看到它的创建类似于Java或其他面向对象编程语言中的类：

```javascript
contract Greeting {
    address creator;
    string message;

    // functions that interact with state variables
}
```

在这个例子中，我们声明了两个状态变量：creator和message。在Solidity中，我们使用名为address的数据类型来存储帐户地址。

接下来，我们需要在构造函数中初始化这两个变量。

### 4.1 构造函数

我们使用function关键字加上合约名称来声明一个构造函数(就像在Java中一样)。

构造函数是一个特殊函数，在合约首次部署到以太坊区块链时仅调用一次。我们只能为一个合约声明一个构造函数：

```javascript
function Greeting(string _message) {
    message = _message;
    creator = msg.sender;
}
```

我们还将初始字符串_message作为参数注入到构造函数中，并将其设置为message状态变量。

在构造函数的第二行中，我们将creator变量初始化为名为msg.sender的值。之所以不需要在构造函数中注入msg，是因为msg是一个全局变量，它提供有关消息的特定信息，例如发送消息的帐户的地址。

我们可能会使用这些信息来实现某些函数的访问控制。

### 4.2 Setter和Getter方法

最后，我们实现消息的Setter和Getter方法：

```java
function greet() constant returns (string) {
    return message;
}

function setGreeting(string _message) {
    message = _message;
}
```

调用函数greet将仅返回当前保存的消息，我们使用constant关键字来指定此函数不会修改合约状态，也不会触发对区块链的任何写入。

我们现在可以通过调用函数setGreeting来更改合约中状态的值，任何人都可以通过调用此函数来更改该值。此方法没有返回类型，但确实接收String类型作为参数。

现在我们已经创建了第一个智能合约，下一步就是将其部署到以太坊区块链中，以便每个人都可以使用它。我们可以使用[Remix](https://remix.ethereum.org/)，它是目前最好的在线IDE，使用起来毫不费力。

## 5. 与智能合约交互

为了与分散网络(区块链)中的智能合约进行交互，我们需要访问其中一个客户端。

有两种方法可以做到这一点：

- [自己运行客户端](https://github.com/web3j/web3j#start-a-client)
- 使用[Infura](https://infura.io/)之类的服务连接到远程节点。

Infura是最直接的选项，因此我们将请求一个[免费的访问令牌](https://infura.io/register)。注册后，我们需要选择Rinkeby测试网络的URL：“https://rinkeby.infura.io/<token\>”。

为了能够通过Java与智能合约进行交易，我们需要使用一个名为[Web3j](https://web3j.io/)的库，这是Maven依赖：

```xml
<dependency>
    <groupId>org.web3j</groupId>
    <artifactId>core</artifactId>
    <version>3.3.1</version>
</dependency>
```

在Gradle中：

```groovy
compile ('org.web3j:core:3.3.1')
```

在开始编写代码之前，我们需要先做一些事情。

### 5.1 创建钱包

Web3j允许我们从命令行使用它的一些功能：

- 创建钱包
- 钱包密码管理
- 将资金从一个钱包转移到另一个钱包
- 生成Solidity智能合约函数包装器

命令行工具可以从项目仓库的[release](https://github.com/web3j/web3j/releases/latest)页面的下载部分以zip文件/tarball的形式获取，或者对于OS X用户通过homebrew获取：

```shell
brew tap web3j/web3j
brew install web3j
```

要生成新的以太坊钱包，我们只需在命令行中输入以下内容：

```shell
$ web3j wallet create
```

它会要求我们输入密码和保存钱包的位置。该文件是JSON格式，需要记住的主要内容是以太坊地址。

我们将在下一步中使用它来请求以太币。

### 5.2 在Rinkeby测试网中请求Ether

Rinkeby测试网现已[弃用](https://www.alchemy.com/overviews/rinkeby-testnet)，你可以尝试使用Sepolia测试网。

为了防止恶意行为者耗尽所有可用资金，他们要求我们提供一个包含我们的以太坊地址的社交媒体帖子的公共链接。

这是一个非常简单的步骤，他们几乎立即就提供了以太币，以便我们运行测试。

### 5.3 生成智能合约包装器

Web3j可以自动生成智能合约包装器代码，无需离开JVM即可部署和与智能合约交互。

要生成包装器代码，我们需要编译智能合约，我们可以在[这里](https://github.com/ethereum/go-ethereum/wiki/Contract-Tutorial#installing-a-compiler)找到安装编译器的说明。然后，我们在命令行上输入以下内容：

```shell
$ solc Greeting.sol --bin --abi --optimize -o <output_dir>/
```

后者会创建两个文件：Greeting.bin和Greeting.abi。现在，我们可以使用web3j的命令行工具生成包装器代码：

```shell
$ web3j solidity generate /path/to/Greeting.bin 
  /path/to/Greeting.abi -o /path/to/src/main/java -p com.your.organisation.name
```

这样，我们现在就有了Java类来与主代码中的合约进行交互。

## 6. 与智能合约交互

在我们的主类中，我们首先创建一个新的Web3j实例来连接到网络上的远程节点：

```java
Web3j web3j = Web3j.build(new HttpService("https://rinkeby.infura.io/<your_token>"));
```

然后我们需要加载我们的以太坊钱包文件：

```java
Credentials credentials = WalletUtils.loadCredentials("<password>", "/path/to/<walletfile>");
```

现在让我们部署我们的智能合约：

```java
Greeting contract = Greeting.deploy(
    web3j, credentials,
    ManagedTransaction.GAS_PRICE, Contract.GAS_LIMIT,
    "Hello blockchain world!").send();
```

部署合约可能需要一段时间，具体取决于网络中的工作。部署后，我们可能希望存储部署合约的地址，我们可以通过以下方式获取地址：

```java
String contractAddress = contract.getContractAddress();
```

所有通过合约进行的交易都可以在以下网址看到：“https://rinkeby.etherscan.io/address/<contract_address\>”。

另一方面，我们可以修改执行交易的智能合约的值：

```java
TransactionReceipt transactionReceipt = contract.setGreeting("Hello again").send();
```

最后，如果我们想查看存储的新值，我们可以简单地这样写：

```java
String newValue = contract.greet().send();
```

## 7. 总结

在本教程中，我们看到[Solidity](https://solidity.readthedocs.io/en/develop/index.html)是一种静态类型编程语言，旨在开发在EVM上运行的智能合约。

我们还用这种语言创建了一个简单的合约，发现它与其他编程语言非常相似。

智能合约只是一个用来描述能够促进价值交换的计算机代码的短语，在区块链上运行时，智能合约成为一个自我运行的计算机程序，在满足特定条件时自动执行。

我们在本文中看到，在区块链中运行代码的能力是以太坊的主要区别，因为它允许开发人员构建一种超出我们以前所见的任何新型应用程序。