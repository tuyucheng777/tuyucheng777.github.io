---
layout: post
title:  XMPP Smack客户端指南
category: libraries
copyright: libraries
excerpt: Smack
---

## 1. 简介

[XMPP](https://xmpp.org/rfcs/rfc3921.html)是一个丰富且复杂的即时通讯协议。

在本教程中，我们不需要从头编写自己的客户端，而是使用Smack，这是一个用Java编写的模块化、可移植的开源XMPP客户端 它为我们完成了大部分繁重的工作。

## 2. 依赖

**Smack被组织为几个模块以提供更大的灵活性**，因此我们可以轻松地包含我们需要的功能。

其中包括：

- XMPP模块
- 支持XMPP标准基金会定义的许多扩展的模块
- 旧版扩展支持
- 用于调试的模块

我们可以在[XMPP文档](https://download.igniterealtime.org/smack/docs/latest/javadoc/)中找到所有支持的模块。

但是，在本教程中，我们仅使用tcp、im、extensions和java7模块：

```xml
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-tcp</artifactId>
</dependency>
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-im</artifactId>
</dependency>
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-extensions</artifactId>
</dependency>
<dependency>
    <groupId>org.igniterealtime.smack</groupId>
    <artifactId>smack-java7</artifactId>
</dependency>
```

最新版本可以在[Maven Repository](https://mvnrepository.com/artifact/org.igniterealtime.smack)找到。

## 3. 设置

为了测试客户端，我们需要一个XMPP服务器。为此，我们将在[jabber.hot-chilli.net](https://jabber.hot-chilli.net/)上创建一个帐户，这是一个面向所有人的免费Jabber/XMPP服务。

之后，我们可以使用XMPPTCPConnectionConfiguration类来配置Smack，该类提供了一个构建器来设置连接的参数：

```java
XMPPTCPConnectionConfiguration config = XMPPTCPConnectionConfiguration.builder()
    .setUsernameAndPassword("tuyucheng","tuyucheng")
    .setXmppDomain("jabb3r.org")
    .setHost("jabb3r.org")
    .build();
```

**构建器允许我们设置执行连接所需的基本信息**，如果需要，我们还可以设置其他参数，例如端口、SSL协议和超时。

## 4. 连接

使用XMPPTCPConnection类即可轻松建立连接：

```java
AbstractXMPPConnection connection = new XMPPTCPConnection(config);
connection.connect(); //Establishes a connection to the server
connection.login(); //Logs in
```

该类包含一个构造函数，用于接收先前构建的配置，它还提供了连接服务器和登录的方法。

**一旦建立连接，我们就可以使用Smack的功能**，比如聊天，我们将在下一节中描述。

如果连接突然中断，默认情况下，Smack将尝试重新连接。

由于连续重新连接不断失败，[ReconnectionManager](https://download.igniterealtime.org/smack/docs/latest/javadoc/org/jivesoftware/smack/ReconnectionManager.html)将尝试立即重新连接到服务器，并增加重试之间的延迟。

## 5. 聊天

**该库的主要功能之一是聊天支持**。

使用Chat类可以在两个用户之间创建新的消息线程：

```java
ChatManager chatManager = ChatManager.getInstanceFor(connection);
EntityBareJid jid = JidCreate.entityBareFrom("tuyucheng2@jabb3r.org");
Chat chat = chatManager.chatWith(jid);
```

请注意，为了构建聊天，我们使用了ChatManager，并且显然指定了与谁聊天。我们通过使用EntityBareJid对象实现了后者，该对象包装了一个**XMPP地址(也称为JID)**，由本地部分(tuyucheng2)和域部分(jabb3r.org)组成。 

之后，我们可以使用send()方法发送消息：

```java
chat.send("Hello!");
```

并通过设置监听器接收消息：

```java
chatManager.addIncomingListener(new IncomingChatMessageListener() {
    @Override
    public void newIncomingMessage(EntityBareJid from, Message message, Chat chat) {
        System.out.println("New message from " + from + ": " + message.getBody());
    }
});
```

### 5.1 房间

除端到端用户聊天外，**Smack还通过使用房间提供对群聊的支持**。

**房间有两种类型，即时房间和预订房间**。

即时房间可立即进入，并根据某些默认配置自动创建。另一方面，预订房间由房间所有者手动配置，然后才允许任何人进入。

让我们看看如何使用MultiUserChatManager创建即时房间：

```java
MultiUserChatManager manager = MultiUserChatManager.getInstanceFor(connection);
MultiUserChat muc = manager.getMultiUserChat(jid);
Resourcepart room = Resourcepart.from("tuyucheng_room");
muc.create(room).makeInstant();
```

以类似的方式，我们可以创建一个预订房间：

```java
Set<Jid> owners = JidUtil.jidSetFrom(
  new String[] { "tuyucheng@jabb3r.org", "tuyucheng2@jabb3r.org" });

muc.create(room)
    .getConfigFormManger()
    .setRoomOwners(owners)
    .submitConfigurationForm();
```

## 6. 名册

**Smack提供的另一个功能是可以跟踪其他用户的存在**。

通过Roster.getInstanceFor()，我们可以获得一个Roster实例：

```java
Roster roster = Roster.getInstanceFor(connection);
```

**[Roster](https://download.igniterealtime.org/smack/docs/latest/javadoc/org/jivesoftware/smack/roster/Roster.html)是一个联系人列表，它将用户表示为RosterEntry对象，并允许我们将用户组织成组**。

我们可以使用getEntries()方法打印Roster中的所有条目：

```java
Collection<RosterEntry> entries = roster.getEntries();
for (RosterEntry entry : entries) {
    System.out.println(entry);
}
```

此外，它允许我们使用RosterListener监听其条目和存在数据的变化：

```java
roster.addRosterListener(new RosterListener() {
    public void entriesAdded(Collection<String> addresses) { // handle new entries }
    public void entriesDeleted(Collection<String> addresses) { // handle deleted entries }
    public void entriesUpdated(Collection<String> addresses) { // handle updated entries }
    public void presenceChanged(Presence presence) { // handle presence change }
});
```

它还提供了一种保护用户隐私的方法，即确保只有经过批准的用户才能订阅名册。**为此，Smack实现了一种基于权限的模型**。

有3种方法可以使用Roster.setSubscriptionMode()方法处理状态订阅请求：

- Roster.SubscriptionMode.accept_all：接受所有订阅请求
- Roster.SubscriptionMode.reject_all：拒绝所有订阅请求
- Roster.SubscriptionMode.manual：手动处理存在订阅请求

如果我们选择手动处理订阅请求，我们将需要注册一个StanzaListener(下一节介绍)并处理Presence.Type.subscribe类型的数据包。

## 7. 节

除了聊天之外，Smack还提供了一个灵活的框架来发送节并监听传入的节。

**需要澄清的是，节是XMPP中一个离散的语义单位，它是通过XML流从一个实体发送到另一个实体的结构化信息**。

我们可以使用send()方法通过Connection传输Stanza：

```java
Stanza presence = new Presence(Presence.Type.subscribe);
connection.sendStanza(presence);
```

在上面的例子中，我们发送了一个Presence节来订阅名册。

另一方面，为了处理传入的节，该库提供了两种构造：

- StanzaCollector 
- StanzaListener

特别是，StanzaCollector允许我们同步等待新的节：

```java
StanzaCollector collector = connection.createStanzaCollector(StanzaTypeFilter.MESSAGE);
Stanza stanza = collector.nextResult();
```

**而StanzaListener是一个用于异步通知我们传入节的接口**：

```java
connection.addAsyncStanzaListener(new StanzaListener() {
    public void processStanza(Stanza stanza) throws SmackException.NotConnectedException,InterruptedException, 
        SmackException.NotLoggedInException {
            // handle stanza
        }
}, StanzaTypeFilter.MESSAGE);
```

### 7.1 过滤器

此外，**该库还提供了一组内置过滤器来处理传入的节**。

我们可以使用StanzaTypeFilter按类型过滤节，或者使用StanzaIdFilter按ID过滤节：

```java
StanzaFilter messageFilter = StanzaTypeFilter.MESSAGE;
StanzaFilter idFilter = new StanzaIdFilter("123456");
```

或者，通过特定地址来辨别：

```java
StanzaFilter fromFilter = FromMatchesFilter.create(JidCreate.from("tuyucheng@jabb3r.org"));
StanzaFilter toFilter = ToMatchesFilter.create(JidCreate.from("tuyucheng2@jabb3r.org"));
```

我们可以使用逻辑过滤运算符(AndFilter，OrFilter，NotFilter)来创建复杂的过滤器：

```java
StanzaFilter filter = new AndFilter(StanzaTypeFilter.Message, FromMatchesFilter.create("tuyucheng@jabb3r.org"));
```

## 8. 总结

在本文中，我们介绍了Smack现成提供的最有用的类。

我们学习了如何配置库以发送和接收XMPP节。

随后，我们学习了如何使用ChatManager和Roster功能处理群聊。