---
layout: post
title:  如何用Java创建Slack插件
category: saas
copyright: saas
excerpt: Slack
---

## 1. 简介

Slack是一款深受全球用户和公司喜爱的聊天系统，它如此受欢迎的原因之一是，我们能够编写自定义插件，在一个Slack中与用户和频道进行交互，这使用了Slack的HTTP API。

Slack没有提供官方的Java插件开发SDK，不过，我们准备使用[官方认可的社区SDK](https://github.com/HubSpot/slack-client)。这样，我们几乎可以从Java代码库访问Slack的所有API，而无需关心API的具体细节。

我们将利用它构建一个小型系统监控机器人，它将定期检索本地计算机的磁盘空间，并在任何驱动器过满时发出警报。

## 2. 获取API凭证

在我们可以使用Slack做任何事情之前，我们需要**创建一个新的应用程序和一个机器人并将其连接到我们的频道**。

首先，让我们访问https://api.slack.com/apps，这是我们管理Slack应用的基础，从这里我们可以创建一个新的应用。

![](/assets/images/2025/saas/javaslackplugin01.png)

当我们这样做时，我们需要输入应用程序的名称和Slack工作区来创建它。

![](/assets/images/2025/saas/javaslackplugin02.png)

完成这些操作后，应用就创建好了，可以开始使用了。下一个界面允许我们创建一个机器人，这是一个插件将扮演的虚拟用户。

![](/assets/images/2025/saas/javaslackplugin03.png)

与任何普通用户一样，我们需要为其指定一个显示名称和用户名，这些是Slack工作区中的其他用户与该机器人用户交互时将看到的设置。

![](/assets/images/2025/saas/javaslackplugin04.png)

现在我们已经完成了这些，我们可以从侧面菜单中选择“Install App”，并将应用程序添加到我们的Slack工作区。完成此操作后，该应用程序就可以与我们的工作区进行交互。

![](/assets/images/2025/saas/javaslackplugin05.png)

这将为我们提供插件与Slack通信所需的令牌。

![](/assets/images/2025/saas/javaslackplugin06.png)

每个与不同Slack工作区交互的机器人都会拥有一组不同的令牌，我们的应用程序运行时需要“Bot User OAuth Access Token”值。

最后，我们需要**邀请机器人加入它应该参与的任何频道**，只需从频道(在本例中为@system_monitoring)向它发送消息即可。

## 3. 将Slack添加到项目

在使用它之前，我们首先需要将[Slack SDK依赖](https://mvnrepository.com/artifact/com.hubspot.slack)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>com.hubspot.slack</groupId>
    <artifactId>slack-base</artifactId>
    <version>${slack.version}</version>
</dependency>
<dependency>
    <groupId>com.hubspot.slack</groupId>
    <artifactId>slack-java-client</artifactId>
    <version>${slack.version}</version>
</dependency>
```

## 4. 应用程序结构

我们应用的核心是检查系统中的错误，我们将用错误检查器的概念来表示这一点。这是一个简单的接口，只有一个方法，触发它来检查错误并报告错误：

```java
public interface ErrorChecker {
    void check();
}
```

我们还希望能够报告发现的任何错误，这是另一个简单的接口，它可以接收问题陈述并进行相应的报告：

```java
public interface ErrorReporter {
    void reportProblem(String problem);
}
```

使用接口让我们能够以不同的方式报告问题，例如，我们可以发送电子邮件、联系错误报告系统，或者向Slack系统发送消息，以便人们立即收到通知。

其背后的设计是为每个ErrorChecker实例分配一个ErrorReporter供其使用，这使我们能够灵活地为不同的检查器使用不同的错误报告器，因为某些错误可能比其他错误更重要。例如，如果磁盘占用率超过90%，我们可能需要向Slack频道发送消息；但如果磁盘占用率超过98%，我们可能希望向特定人员发送私信。

## 5. 检查磁盘空间

我们的错误检查器将检查本地系统的磁盘空间大小，任何文件系统的可用空间不足特定百分比的情况都会被视为错误，并会被报告。

我们将利用Java 7中引入的NIO2 FileStore API以跨平台的方式获取此信息。

现在，让我们看一下错误检查器：

```java
public class DiskSpaceErrorChecker implements ErrorChecker {
    private static final Logger LOG = LoggerFactory.getLogger(DiskSpaceErrorChecker.class);

    private ErrorReporter errorReporter;

    private double limit;

    public DiskSpaceErrorChecker(ErrorReporter errorReporter, double limit) {
        this.errorReporter = errorReporter;
        this.limit = limit;
    }

    @Override
    public void check() {
        FileSystems.getDefault().getFileStores().forEach(fileStore -> {
            try {
                long totalSpace = fileStore.getTotalSpace();
                long usableSpace = fileStore.getUsableSpace();
                double usablePercentage = ((double) usableSpace) / totalSpace;

                if (totalSpace > 0 && usablePercentage < limit) {
                    String error = String.format("File store %s only has %d%% usable disk space",
                            fileStore.name(), (int)(usablePercentage * 100));
                    errorReporter.reportProblem(error);
                }
            } catch (IOException e) {
                LOG.error("Error getting disk space for file store {}", fileStore, e);
            }
        });
    }
}
```

这里，我们获取本地系统上所有文件存储的列表，然后逐个检查，任何可用空间小于我们定义的限制的文件存储都会通过错误报告器生成错误。

## 6. 向Slack频道发送错误

我们现在需要能够报告错误，**我们的第一个报告器会将消息发送到Slack频道**，这样频道中的任何人都可以看到这条消息，希望有人能对此做出反应。

它使用了Slack SDK中的SlackClient以及发送消息的频道名称，它还实现了ErrorReporter接口，以便我们可以轻松地将其插入到任何需要使用它的错误检查器中：

```java
public class SlackChannelErrorReporter implements ErrorReporter {
    private SlackClient slackClient;

    private String channel;

    public SlackChannelErrorReporter(SlackClient slackClient, String channel) {
        this.slackClient = slackClient;
        this.channel = channel;
    }

    @Override
    public void reportProblem(String problem) {
        slackClient.postMessage(
                ChatPostMessageParams.builder()
                        .setText(problem)
                        .setChannelId(channel)
                        .build()
        ).join().unwrapOrElseThrow();
    }
}
```

## 7. 应用连接

现在，我们可以连接应用程序并让它监控我们的系统了。在本教程中，我们将使用核心JVM中的[Java Timer和TimerTask](https://www.baeldung.com/java-timer-and-timertask)，但我们也可以很容易地使用Spring或任何其他框架来构建它。

目前，它将有一个DiskSpaceErrorChecker，用于向我们的“general”频道报告任何可用率低于10%的磁盘，并且每5分钟运行一次：

```java
public class MainClass {
    public static final long MINUTES = 1000 * 60;

    public static void main(String[] args) throws IOException {
        SlackClientRuntimeConfig runtimeConfig = SlackClientRuntimeConfig.builder()
                .setTokenSupplier(() -> "<Your API Token>")
                .build();

        SlackClient slackClient = SlackClientFactory.defaultFactory().build(runtimeConfig);

        ErrorReporter slackChannelErrorReporter = new SlackChannelErrorReporter(slackClient, "general");

        ErrorChecker diskSpaceErrorChecker10pct =
                new DiskSpaceErrorChecker(slackChannelErrorReporter, 0.1);

        Timer timer = new Timer();
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                diskSpaceErrorChecker10pct.check();
            }
        }, 0, 5 * MINUTES);
    }
}
```

我们需要将“<Your API Token\>”替换为之前获取的令牌，然后就可以运行了。完成后，如果一切正常，我们的插件就会检查本地驱动器，并在出现任何错误时向Slack发送消息。

![](/assets/images/2025/saas/javaslackplugin07.png)

## 8. 将错误作为私人消息发送

接下来，我们将添加一个错误报告器，用于发送私信，这对于更紧急的错误非常有用，因为它会立即**通知特定用户，而不是依赖频道中的某个人做出反应**。

这里的错误报告器更加复杂，因为它需要与单个目标用户进行交互：

```java
public class SlackUserErrorReporter implements ErrorReporter {
    private SlackClient slackClient;

    private String user;

    public SlackUserErrorReporter(SlackClient slackClient, String user) {
        this.slackClient = slackClient;
        this.user = user;
    }

    @Override
    public void reportProblem(String problem) {
        UsersInfoResponse usersInfoResponse = slackClient
                .lookupUserByEmail(UserEmailParams.builder()
                        .setEmail(user)
                        .build()
                ).join().unwrapOrElseThrow();

        ImOpenResponse imOpenResponse = slackClient.openIm(ImOpenParams.builder()
                .setUserId(usersInfoResponse.getUser().getId())
                .build()
        ).join().unwrapOrElseThrow();

        imOpenResponse.getChannel().ifPresent(channel -> {
            slackClient.postMessage(
                    ChatPostMessageParams.builder()
                            .setText(problem)
                            .setChannelId(channel.getId())
                            .build()
            ).join().unwrapOrElseThrow();
        });
    }
}
```

这里我们要做的是找到要发送消息的用户-通过电子邮件地址查找，因为这是无法更改的。接下来，**我们向该用户打开一个即时通讯频道，然后将错误消息发送到该频道**。

然后可以在主方法中将其连接起来，我们将直接提醒单个用户：

```java
ErrorReporter slackUserErrorReporter = new SlackUserErrorReporter(slackClient, "testuser@tuyucheng.com");

ErrorChecker diskSpaceErrorChecker2pct = new DiskSpaceErrorChecker(slackUserErrorReporter, 0.02);

timer.scheduleAtFixedRate(new TimerTask() {
    @Override
    public void run() {
        diskSpaceErrorChecker2pct.check();
    }
}, 0, 5 * MINUTES);
```

完成后，我们可以运行它并获取有关错误的私人消息。

![](/assets/images/2025/saas/javaslackplugin08.png)

## 9. 总结

我们介绍了如何将Slack集成到我们的工具中，以便我们可以将反馈发送给整个团队或单个成员。