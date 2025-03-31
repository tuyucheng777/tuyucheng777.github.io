---
layout: post
title:  EthereumJ简介
category: libraries
copyright: libraries
excerpt: EthereumJ
---

## 1. 简介

在本文中，我们将了解[EthereumJ](https://github.com/ethereum/ethereumj)库，它允许我们使用Java与[以太坊](https://www.ethereum.org/)区块链进行交互。

首先，让我们简单了解一下这项技术的全部内容。

**请注意，本教程使用[EthereumJ](https://github.com/ethereum/ethereumj)，现已弃用。因此，此处显示的支持代码不再维护**。

## 2. 关于以太坊

[以太坊](https://www.ethereum.org/)是一种加密货币，利用分布式、点对点的数据库，以可编程[区块链](https://www.baeldung.com/cs/blockchain)的形式存在，即以太坊虚拟机(EVM)。它通过分散但相互连接的节点进行同步和操作。

截至2017年，节点通过共识同步区块链，通过挖矿([工作量证明](https://www.baeldung.com/cs/blockchain-proof-of-work))创建货币，验证交易，执行用[Solidity](https://solidity.readthedocs.io/en/develop/)编写的智能合约，并运行EVM。

区块链分为多个区块，其中包含账户状态(包括账户之间的交易)和工作量证明。

## 3. Ethereum门面

org.ethereum.facade.Ethereum类将EthereumJ的许多包抽象并统一为一个易于使用的接口。

可以连接到一个节点来与整个网络同步，一旦连接，我们就可以使用区块链。

创建门面对象非常简单：

```java
Ethereum ethereum = EthereumFactory.createEthereum();
```

## 4. 连接到以太坊网络

要连接到网络，我们必须首先连接到一个节点，即运行官方客户端的服务器。节点由org.ethereum.net.rlpx.Node类表示。

org.ethereum.listener.EthereumListenerAdapter处理在成功建立与节点的连接后我们的客户端检测到的区块链事件。

### 4.1 连接到以太坊网络

让我们连接到网络上的一个节点，这可以手动完成：

```java
String ip = "http://localhost";
int port = 8345;
String nodeId = "a4de274d3a159e10c2c9a68c326511236381b84c9ec...";

ethereum.connect(ip, port, nodeId);
```

也可以使用Bean自动连接网络：

```java
public class EthBean {
    private Ethereum ethereum;

    public void start() {
        ethereum = EthereumFactory.createEthereum();
        ethereum.addListener(new EthListener(ethereum));
    }

    public Block getBestBlock() {
        return ethereum.getBlockchain().getBestBlock();
    }

    public BigInteger getTotalDifficulty() {
        return ethereum.getBlockchain().getTotalDifficulty();
    }
}
```

然后，我们可以将EthBean注入到我们的应用程序配置中，然后它会自动连接到以太坊网络并开始下载区块链。

事实上，大多数连接处理都可以方便地包装和抽象，只需在我们创建的org.ethereum.facade.Ethereum实例中添加一个org.ethereum.listener.EthereumListenerAdapter实例，就像我们在上面的start()方法中所做的那样：

```java
EthBean eBean = new EthBean();
Executors.newSingleThreadExecutor().submit(eBean::start);
```

### 4.2 使用监听器处理区块链

我们还可以子类化EthereumListenerAdapter来处理客户端检测到的区块链事件。

为了完成这一步，我们需要创建子类监听器：

```java
public class EthListener extends EthereumListenerAdapter {

    private void out(String t) {
        l.info(t);
    }

    //...

    @Override
    public void onBlock(Block block, List receipts) {
        if (syncDone) {
            out("Net hash rate: " + calcNetHashRate(block));
            out("Block difficulty: " + block.getDifficultyBI().toString());
            out("Block transactions: " + block.getTransactionsList().toString());
            out("Best block (last block): " + ethereum
                    .getBlockchain()
                    .getBestBlock().toString());
            out("Total difficulty: " + ethereum
                    .getBlockchain()
                    .getTotalDifficulty().toString());
        }
    }

    @Override
    public void onSyncDone(SyncState state) {
        out("onSyncDone " + state);
        if (!syncDone) {
            out(" ** SYNC DONE ** ");
            syncDone = true;
        }
    }
}
```

onBlock()方法在收到任何新块(无论是旧块还是当前块)时触发，EthereumJ使用org.ethereum.core.Block类来表示和处理块。

同步完成后，onSyncDone()方法将触发，使我们的本地以太坊数据保持最新。

## 5. 使用区块链

现在我们可以连接到以太坊网络并直接使用区块链，我们将深入研究我们经常使用的几个基本但非常重要的操作。

### 5.1 提交交易

现在，我们已经连接到区块链，可以提交交易了。提交交易相对容易，但创建实际交易本身就是一个冗长的话题：

```java
ethereum.submitTransaction(new Transaction(new byte[]));
```

### 5.2 访问Blockchain对象

getBlockchain()方法返回一个Blockchain门面对象，该对象带有用于获取当前网络难度和特定区块的Getter。

由于我们在第4.3节中设置了EthereumListener，因此我们可以使用上述方法访问区块链：

```java
ethereum.getBlockchain();
```

### 5.3 返回以太坊账户地址

我们还可以返回一个以太坊Address。

要获得以太坊账户，我们首先需要在区块链上验证公钥和私钥对。

让我们用一个新的随机密钥对创建一个新密钥：

```java
org.ethereum.crypto.ECKey key = new ECKey();
```

让我们从给定的私钥创建一个密钥：

```java
org.ethereum.crypto.ECKey key = ECKey.fromPivate(privKey);
```

然后我们可以使用密钥初始化Account，通过调用.init()，我们在Account对象上设置ECKey和关联的Address：

```java
org.ethereum.core.Account account = new Account();
account.init(key);
```

## 6. 其他功能

该框架还提供另外两个主要功能，我们在这里不会介绍，但值得一提。

首先，我们有能力编译和执行Solidity智能合约。然而，用Solidity创建合约，然后编译和执行它们本身就是一个广泛的话题。

其次，虽然框架支持使用CPU进行有限的挖矿，但考虑到CPU缺乏盈利能力，建议使用GPU矿机。

关于以太坊本身的更多高级主题可以在[官方文档](https://www.ethereum.org/developers/)中找到。

## 7. 总结

在此快速教程中，我们展示了如何连接到以太坊网络以及使用区块链的几种重要方法。