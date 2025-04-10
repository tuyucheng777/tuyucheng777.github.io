---
layout: post
title:  Java贪心算法简介
category: algorithms
copyright: algorithms
excerpt: 贪心算法
---

## 1. 简介

在本教程中，我们将介绍Java生态系统中的[贪心算法](https://www.baeldung.com/cs/greedy-approach-vs-dynamic-programming)。

## 2. 贪心问题

面对数学问题时，设计解决方案的方法可能有多种。我们可以采用迭代解决方案，或使用一些高级技术，例如分而治之原则(例如[快速排序算法](https://www.baeldung.com/java-quicksort))或动态规划方法(例如[背包问题](https://www.baeldung.com/java-knapsack))等等。

大多数时候，我们都在寻找最优解，但遗憾的是，我们并不总是能得到这样的结果。然而，有些情况下，即使是次优结果也是有价值的。借助一些特定的策略或启发式方法，我们或许能获得如此宝贵的回报。

在这种情况下，给定一个可分问题，**在过程的每个阶段采取局部最优选择或“贪心选择”的策略称为[贪心算法](https://www.baeldung.com/cs/greedy-approach-vs-dynamic-programming)**。

我们指出，我们应该解决一个“可分”的问题：即可以被描述为一组具有几乎相同特征的子问题的情况。因此，大多数情况下，**贪心算法会被实现为递归算法**。

贪心算法可以引导我们在恶劣的环境下找到合理的解决方案；缺乏计算资源、执行时间约束、API限制或任何其他类型的限制。

### 2.1 场景

**在这个简短的教程中，我们将实现一个贪心策略，使用其API从社交网络中提取数据**。

假设我们想在“小蓝鸟”社交网站上吸引更多用户，实现我们目标的最佳方式是发布原创内容或转发一些能引起广大受众兴趣的内容。

我们如何找到这样的受众？好吧，我们必须找到一个拥有大量粉丝的帐户，并为他们发布一些内容。

### 2.2 经典算法与贪心算法

我们考虑以下情况：我们的帐户有4个关注者，每个关注者如下图所示分别有2、2、1和3个关注者，依此类推：

![](/assets/images/2025/algorithms/javagreedyalgorithms01.png)

带着这个目标，我们会从我们账号的粉丝中挑选出粉丝最多的那个，然后我们再重复这个过程两次，直到达到三度联系(总共4个步骤)。

**这样，我们就定义了一条由用户组成的路径，引导我们通过账户获得最庞大的粉丝群**。如果我们能向他们推送一些内容，他们肯定会访问我们的页面。

我们可以从“传统”方法开始，每一步，我们都会执行查询以获取帐户的关注者。由于我们的筛选过程，帐户数量每一步都会增加。

令人惊讶的是，我们最终总共执行了25个查询：

![](/assets/images/2025/algorithms/javagreedyalgorithms02.png)

这里出现了一个问题：例如，Twitter API将此类查询限制为每15分钟15次，如果我们尝试执行超过允许的次数的调用，我们将得到“Rate limit exceeded code – 88”或“Returned in API v1.1 when a request cannot be served due to the application’s rate limit having been exhausted for the resource”，我们如何克服这样的限制？

答案就在我们眼前：贪心算法。如果我们使用这种方法，在每一步中，我们都可以假设拥有最多粉丝的用户是唯一需要考虑的用户：最后，我们只需要4个查询，这是一个相当大的进步。

![](/assets/images/2025/algorithms/javagreedyalgorithms03.png)

这两种方法的结果会有所不同：在第一种情况下，我们得到的最佳解决方案是16，而在后一种情况下，可到达的关注者的最大数量仅为12。

## 3. 实现

**为了实现上述逻辑，我们初始化了一个小型Java程序，其中我们将模拟Twitter API**；我们还将使用[Lombok](https://www.baeldung.com/intro-to-project-lombok)库。

现在，我们来定义SocialConnector组件，并在其中实现逻辑。请注意，我们将添加一个计数器来模拟调用限制，但我们将其降低到4：
```java
public class SocialConnector {
    private boolean isCounterEnabled = true;
    private int counter = 4;
    @Getter @Setter
    List users;

    public SocialConnector() {
        users = new ArrayList<>();
    }

    public boolean switchCounter() {
        this.isCounterEnabled = !this.isCounterEnabled;
        return this.isCounterEnabled;
    }
}
```

然后我们将添加一种方法来检索特定帐户的关注者列表：
```java
public List getFollowers(String account) {
    if (counter < 0) {
        throw new IllegalStateException ("API limit reached");
    } else {
        if (this.isCounterEnabled) {
            counter--;
        }
        for (SocialUser user : users) {
            if (user.getUsername().equals(account)) {
                return user.getFollowers();
            }
        }
    }
    return new ArrayList<>();
}
```

为了支持我们的流程，我们需要一些类来建模我们的用户实体：
```java
public class SocialUser {
    @Getter
    private String username;
    @Getter
    private List<SocialUser> followers;

    @Override
    public boolean equals(Object obj) {
        return ((SocialUser) obj).getUsername().equals(username);
    }

    public SocialUser(String username) {
        this.username = username;
        this.followers = new ArrayList<>();
    }

    public SocialUser(String username, List<SocialUser> followers) {
        this.username = username;
        this.followers = followers;
    }

    public void addFollowers(List<SocialUser> followers) {
        this.followers.addAll(followers);
    }
}
```

### 3.1 贪心算法

最后，是时候实现我们的贪心策略了，所以让我们添加一个新组件GreedyAlgorithm，我们将在其中执行递归：
```java
public class GreedyAlgorithm {
    int currentLevel = 0;
    final int maxLevel = 3;
    SocialConnector sc;
    public GreedyAlgorithm(SocialConnector sc) {
        this.sc = sc;
    }
}
```

然后我们需要插入一个方法findMostFollowersPath，在其中我们将找到拥有最多关注者的用户，对其进行计数，然后继续下一步：
```java
public long findMostFollowersPath(String account) {
    long max = 0;
    SocialUser toFollow = null;

    List followers = sc.getFollowers(account);
    for (SocialUser el : followers) {
        long followersCount = el.getFollowersCount();
        if (followersCount > max) {
            toFollow = el;
            max = followersCount;
        }
    }
    if (currentLevel < maxLevel - 1) {
        currentLevel++;
        max += findMostFollowersPath(toFollow.getUsername());
    }
    return max;
}
```

记住：**这里我们执行贪心选择**。因此，每次调用此方法时，我们都会从列表中选择一个且仅一个元素并继续：我们永远不会改变我们的决定。

一切准备就绪，可以测试我们的应用程序了。在此之前，我们需要记住填充我们的微型网络，最后执行以下单元测试：
```java
public void greedyAlgorithmTest() {
    GreedyAlgorithm ga = new GreedyAlgorithm(prepareNetwork());
    assertEquals(ga.findMostFollowersPath("root"), 5);
}
```

### 3.2 非贪心算法

让我们创建一个非贪心方法，只是为了亲眼看看会发生什么。因此，我们需要先构建一个NonGreedyAlgorithm类：
```java
public class NonGreedyAlgorithm {
    int currentLevel = 0;
    final int maxLevel = 3;
    SocialConnector tc;

    public NonGreedyAlgorithm(SocialConnector tc, int level) {
        this.tc = tc;
        this.currentLevel = level;
    }
}
```

让我们创建一个等效的方法来检索关注者：
```java
public long findMostFollowersPath(String account) {
    List<SocialUser> followers = tc.getFollowers(account);
    long total = currentLevel > 0 ? followers.size() : 0;

    if (currentLevel < maxLevel ) {
        currentLevel++;
        long[] count = new long[followers.size()];
        int i = 0;
        for (SocialUser el : followers) {
            NonGreedyAlgorithm sub = new NonGreedyAlgorithm(tc, currentLevel);
            count[i] = sub.findMostFollowersPath(el.getUsername());
            i++;
        }

        long max = 0;
        for (; i > 0; i--) {
            if (count[i-1] > max) {
                max = count[i-1];
            }
        }
        return total + max;
    }
    return total;
}
```

当我们的类准备就绪后，可以准备一些单元测试：一个用于验证调用限制是否超出，另一个用于检查使用非贪心策略返回的值：
```java
public void nongreedyAlgorithmTest() {
    NonGreedyAlgorithm nga = new NonGreedyAlgorithm(prepareNetwork(), 0);
    Assertions.assertThrows(IllegalStateException.class, () -> {
        nga.findMostFollowersPath("root");
    });
}

public void nongreedyAlgorithmUnboundedTest() {
    SocialConnector sc = prepareNetwork();
    sc.switchCounter();
    NonGreedyAlgorithm nga = new NonGreedyAlgorithm(sc, 0);
    assertEquals(nga.findMostFollowersPath("root"), 6);
}
```

## 4. 结果

首先，我们尝试了贪心策略，检查其有效性。然后，我们通过穷举搜索验证了情况，包括有API限制和无API限制两种情况。

我们的快速贪心算法每次都会做出局部最优选择，并返回一个数值。另一方面，由于环境限制，非贪婪算法不会返回任何结果。

**比较两种方法的输出，我们可以理解贪心策略如何拯救了我们，即使检索到的值不是最优的**，我们可以称之为局部最优。

## 5. 总结

在社交媒体等瞬息万变且快速变化的环境中，需要找到最佳解决方案的问题可能会成为可怕的幻想：难以实现，同时又不切实际。

克服限制并优化API调用是一个相当重要的主题，但正如我们所讨论的，贪心策略是有效的，选择这种方法可以为我们省去很多麻烦，并带来有价值的结果。

请记住，并非所有情况都适合：我们每次都需要评估自己的情况。