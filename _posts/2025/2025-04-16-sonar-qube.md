---
layout: post
title:  使用SonarQube进行代码分析
category: security
copyright: security
excerpt: SonarQube
---

## 1. 概述

在本文中，我们将研究使用[SonarQube](https://www.sonarqube.org/)进行静态源代码分析-这是一个确保代码质量的开源平台。

让我们从一个核心问题开始-为什么要分析源代码？很简单，为了确保项目生命周期内的质量、可靠性和可维护性；糟糕的代码库维护成本总是更高。

## 2. 本地运行SonarQube

要在本地运行SonarQube，有两种方法：

- 从zip文件运行Sonarqube服务器，要下载最新的LTS版本的SonarQube，你可以在[此处](https://www.sonarqube.org/downloads/)找到，按照本[快速入门指南](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/)中概述的步骤设置本地服务器。

- 在Docker容器中运行Sonarqube，通过运行以下命令启动服务器：

  ```shell
  docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
  ```

## 3. 在SonarQube中生成Token

按照前面的步骤启动新的SonarQube服务器后，现在是时候使用admin:admin登录了(这些是初始默认凭据，系统会要求你更改它)。

为了能够在项目中使用此扫描器，你必须从SonarQube界面生成访问令牌。登录后，转到你的帐户页面(http://localhost:9000/account)并选择“Security”选项卡，从该选项卡中，你可以生成3种类型的令牌以用于你的项目：

- Project Analysis Token：这适用于项目级别
- Global Analysis Token：将在所有项目之间共享
- User Token：基于用户级别访问，该用户将有权访问哪个项目

![](/assets/images/2025/security/sonarqube01.png)

## 4. 分析源代码

稍后我们将在分析项目时使用生成的令牌，我们还需要选择项目的主要语言(Java)和构建技术(Maven)。

让我们在pom.xml中定义插件：

```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.4.0.905</version>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

插件的最新版本可在[此处](https://mvnrepository.com/artifact/org.sonarsource.scanner.maven/sonar-maven-plugin)获取。现在，我们需要从项目目录的根目录执行此命令来扫描它：

```shell
mvn clean verify sonar:sonar -Dsonar.projectKey=PROJECT_KEY 
                             -Dsonar.projectName='PROJECT_NAME' 
                             -Dsonar.host.url=http://localhost:9000 
                             -Dsonar.token=THE_GENERATED_TOKEN
```

我们需要用我们在步骤3中设置的信息和之前SonarQube提供的生成访问令牌替换project_key、project_name和the-generated-token。

**我们在本文中使用的项目可以在[这里](https://github.com/eugenp/tutorials/tree/master/security-modules/cas/cas-secured-app)获得**。

我们将SonarQube服务器的主机URL和登录名(生成的令牌)指定为Maven插件的参数。

执行命令后，结果将显示在项目仪表板上-http://localhost:9000。

我们还可以向Maven插件传递其他参数，甚至可以从Web界面设置；sonar.host.url、sonar.projectKey和sonar.sources是必需的，而其他参数是可选的。

其他分析参数及其默认值请参见[此处](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)，另请注意，每个语言插件都有分析兼容源代码的规则。

## 5. 分析结果

现在我们已经分析了我们的第一个项目，我们可以转到http://localhost:9000的Web界面并刷新页面。

我们将在那里看到报告摘要：

![](/assets/images/2025/security/sonarqube02.png)

发现的问题可以是Bug、漏洞、代码异味、覆盖率或重复，每个类别都有相应的问题数量或百分比值。

此外，问题可以分为五种严重程度：blocker、critical、major、minor和info。项目名称前面有一个图标，显示质量门状态-通过(绿色)或失败(红色)。

单击项目名称将带我们进入专用仪表板，我们可以在其中更详细地探讨特定于项目的问题。

我们可以从项目仪表板查看项目代码、活动并执行管理任务-每个任务都在单独的选项卡上显示。

虽然有一个全局的“Issues”选项卡，但项目仪表板上的Issues选项卡仅显示特定于相关项目的问题：

![](/assets/images/2025/security/sonarqube03.png)

问题选项卡始终显示类别、严重性级别、标签以及纠正问题所需的计算工作量(关于时间)。

在Issues选项卡中，你可以将问题分配给其他用户、对其进行评论并更改其严重性级别；单击问题本身将显示有关该问题的更多详细信息。

Issues选项卡左侧带有复杂的过滤器，这些过滤器有助于精准定位问题。那么，如何知道代码库是否足够健康，可以部署到生产中呢？这就是质量门的作用所在。

## 6. SonarQube质量门

在本节中，我们将介绍SonarQube的一项主要功能-质量门。然后，我们将通过一个示例来了解如何设置自定义质量门。

### 6.1 什么是质量门？

质量门是项目在符合生产发布条件之前必须满足的一组条件，**它回答了一个问题：我目前的代码状态是否可以投入生产**？

确保“新”代码的质量，同时修复现有代码，是长期维护良好代码库的好方法之一。质量门有助于设置规则，以便在后续分析中验证添加到代码库的每一段新代码。

质量门中设置的条件仍然会影响未修改的代码段，如果我们能够防止新问题的出现，那么随着时间的推移，所有问题都将得到解决。

这种方法类似于从源头上[解决漏水问题](https://www.sonarsource.com/blog/water-leak-changes-the-game-for-technical-debt-management/)，**这就引出了一个术语-漏水期，这是项目两次分析/版本之间的时间间隔**。

如果我们在同一个项目上重新运行分析，项目仪表板的概览选项卡将显示泄漏期间的结果：

![](/assets/images/2025/security/sonarqube04.png)

从Web界面，我们可以在“Quality Gates”选项卡中访问所有定义的质量门。默认情况下，SonarQube方式已预装在服务器中。

SonarQube方式的默认配置在以下情况下将代码标记为失败：

- 新代码覆盖率不足80%
- 新代码中重复行的百分比大于3
- 可维护性、可靠性或安全性评级低于A

有了这种理解，我们可以创建一个自定义的质量门。

### 6.2 添加自定义质量门

首先，**我们需要点击“Quality Gates”选项卡，然后点击页面左侧的“Create”按钮**，我们需要给它起个名字tuyucheng。

现在我们可以设置我们想要的条件：

![](/assets/images/2025/security/sonarqube05.png)

**从Add Condition下拉菜单中，选择Blocker Issues**；它会立即显示在条件列表中。

我们将指定“is greater than”作为Operator，将“Error”列设置为0，并勾中“Over Leak Period”列：

![](/assets/images/2025/security/sonarqube06.png)

**然后我们点击“Add”按钮使更改生效**，让我们按照与上述相同的步骤添加另一个条件。

我们将从Add Condition下拉列表中选择issues并勾选Over Leak Period列。

Operator列的值将设置为“is less than”，我们将添加1作为Error列的值。这意味着如果新添加的代码中的问题数量小于1，则将质量门标记为失败。

我知道这在技术上不太合理，但为了学习，我们还是用它吧；别忘了点击“Add”按钮来保存规则。

**最后一步，我们需要将一个项目附加到自定义质量门，我们可以向下滚动页面到“Projects”部分来执行此操作**。

我们需要点击“Projects”，然后标记我们选择的项目，我们也可以在页面右上角将其设置为默认的质量门。

我们将再次扫描项目源代码，就像我们之前使用Maven命令所做的那样。完成后，我们将转到“Projects”选项卡并刷新。

这次，项目将无法满足质量门标准，因此会失败。为什么？因为我们的其中一条规则规定，如果没有新问题，项目就应该失败。

让我们回到“Quality Gates”选项卡并将issues的条件更改为is greater than，我们需要单击更新按钮以使此更改生效。

这次将对源代码进行新的扫描。

## 7. 将SonarQube集成到CI

可以将SonarQube作为持续集成过程的一部分，如果代码分析不满足质量门条件，这将自动导致构建失败。

为了实现这一点，我们将使用[SonarCloud](https://sonarcloud.io/)，它是SonaQube服务器的云托管版本，我们可以在[这里](https://sonarcloud.io/sessions/new)创建一个帐户。

从My Account > Organizations，我们可以看到组织密钥，它通常采用xxxx-github或xxxx-bitbucket的形式。

同样，从My Account > Security，我们可以像在服务器的本地实例中一样生成令牌，请记下该令牌和组织密钥，以便稍后使用。

在本文中，我们将使用Travis CI，我们将在[此处](https://travis-ci.org/)使用现有的Github Profile创建一个帐户。它将加载我们所有的项目，我们可以在任何项目上切换开关来激活Travis CI。

我们需要将我们在SonarCloud上生成的令牌添加到Travis环境变量中。我们可以通过单击为CI激活的项目来执行此操作。

然后，我们点击“More Options” > “Settings”，然后向下滚动到“Environment Variables”：

![](/assets/images/2025/security/sonarqube07.png)

我们将添加一个名为SONAR_TOKEN的新条目，并使用在SonarCloud上生成的令牌作为值，Travis CI将对其进行加密并隐藏，使其不公开：

![](/assets/images/2025/security/sonarqube08.png)

最后，我们需要在项目根目录中添加一个.travis.yml文件，其内容如下：

```yaml
language: java
sudo: false
install: true
addons:
    sonarcloud:
        organization: "your_organization_key"
        token:
            secure: "$SONAR_TOKEN"
jdk:
    - oraclejdk8
script:
    - mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent package sonar:sonar
cache:
    directories:
        - '$HOME/.m2/repository'
        - '$HOME/.sonar/cache'
```

记得用上面描述的组织密钥替换你的组织密钥，提交新代码并推送到Github仓库将触发Travis CI构建，进而激活Sonar扫描。

## 8. 总结

在本教程中，我们研究了如何在本地设置SonarQube服务器以及如何使用质量门来定义项目是否适合生产发布的标准。

[SonarQube文档](https://docs.sonarqube.org/latest/)包含有关该平台其他方面的更多信息。