---
layout: post
title:  在Java中实现A*寻路
category: algorithms
copyright: algorithms
excerpt: A*寻路
---

## 1. 简介

**寻路算法是一种地图导航技术**，它使我们能够在两个不同的点之间找到一条路径。不同的算法各有优缺点，这通常体现在算法本身的效率和生成的路径的效率上。

## 2. 什么是寻路算法？

**寻路算法是一种将由节点和边组成的图转换为通过图的路径的技术**，图可以是任何需要遍历的对象。在本文中，我们将尝试遍历伦敦地铁系统的一部分：

![](/assets/images/2025/algorithms/javaastarpathfinding01.png)

这其中有很多有趣的部分：

- 我们的起点和终点之间可能有一条直接路线，也可能没有。例如，我们可以直接从“Earl's Court”前往“Monument”，但不能直接前往“Angel”。
- 每一步都有特定的成本，在我们的例子中，这是车站之间的距离。
- 每个站点仅与其他一小部分站点相连。例如，“Regent’s Park”仅与“Baker Street”和“Oxford Circus”直接相连。

所有寻路算法都将所有节点(在我们的例子中是车站)及其之间的连接以及所需的起点和终点作为输入，**输出通常是一组节点，这些节点将按照我们需要的顺序引导我们从起点走到终点**。

## 3. 什么是A\*？

**[A\*](https://www.baeldung.com/cs/a-star-algorithm)是一种特定的寻路算法**，由Peter Hart、Nils Nilsson和Bertram Raphael于1968年首次发表，**当没有机会预先计算路线并且对内存使用没有限制时，它通常被认为是最佳算法**。

在最坏的情况下，内存和性能复杂度都可能是O(b^d)，因此虽然它总是能找到最有效的路线，但它并不总是最有效的方法。

A\*实际上是[Dijkstra算法](https://www.baeldung.com/java-dijkstra)的变体，它提供了额外的信息来帮助选择下一个要使用的节点。这些额外的信息不需要完美无缺-如果我们已经拥有了完美的信息，那么寻路就毫无意义了。但这些信息越好，最终的结果就越好。

## 4. A\*工作原理

**A\*算法的工作原理是迭代地选择迄今为止的最佳路线，并尝试找出下一步的最佳路线**。

使用此算法时，我们需要跟踪几部分数据。“开放集”是我们当前正在考虑的所有节点，这并非指系统中的每个节点，而是我们可能采取下一步行动的每个节点。

我们还将跟踪系统中每个节点的当前最佳分数、估计总分数和当前最佳前一个节点。

为此，我们需要计算两个不同的分数，一个是从一个节点到另一个节点的分数，第二个是一种启发式方法，用于估算从任意节点到目的地所需的成本。这个估算不需要准确，但更高的准确度将产生更好的结果。唯一的要求是两个分数必须一致-也就是说，它们使用相同的单位。

最开始，我们的开放集由我们的起始节点组成，并且我们根本没有关于任何其他节点的信息。

在每次迭代中，我们将：

- 从我们的开放集中选择具有最低估计总分的节点
- 从开放集中移除此节点
- 将我们可以从中到达的所有节点添加到开放集中

**当我们这样做时，我们还会计算从这个节点到每个新节点的新分数，看看它是否比我们到目前为止得到的分数有所改进，如果是，那么我们就会更新我们对该节点的了解**。

然后重复此过程，直到我们的开放集中具有最低估计总分的节点成为我们的目的地，此时我们就得到了路线。

### 4.1 示例

例如，让我们从“Marylebone”出发，尝试找到前往“Bond Street”的路。

**一开始，我们的开放集只包含“Marylebone”**，这意味着，这个节点隐式地是我们获得了最佳“预估总分”的节点。

我们的下一站可以是“Edgware Road”，成本为0.4403公里，也可以是“Baker Street”，成本为0.4153公里。但是，“Edgware Road”的方向错误，因此从这里到目的地的启发式得分为1.4284公里，而“Baker Street”的启发式得分为1.0753公里。

这意味着经过这次迭代，我们的开放集由两个条目组成-“Edgware Road”(估计总分1.8687公里)和“Baker Street”(估计总分1.4906公里)。

**然后，我们的第二次迭代将从“Baker Street”开始，因为这条街的估计总分最低**。从这里开始，我们的下一站可以是“Marylebone”、“St. John’s Wood”、“Great Portland Street”、“Regent’s Park”或“Bond Street”。

我们不会逐一讨论所有这些，但让我们以“Marylebone”为例，到达那里的成本仍然是0.4153公里，但这意味着总成本现在是0.8306公里。此外，从这里到目的地的启发式得分为1.323公里。

**这意味着预估总得分将为2.1536公里，低于该节点之前的得分**。这是有道理的，因为在这种情况下，我们不得不付出额外的努力却一无所获，这意味着我们不会将其视为可行的路线。因此，“Marylebone”的详细信息不会更新，也不会被重新添加到开放集中。

## 5. Java实现

我们将构建一个通用解决方案，然后实现使其适用于伦敦地铁所需的代码。之后，我们只需实现其中特定的部分，就可以将其用于其他场景。

### 5.1 表示图

**首先，我们需要能够表示我们想要遍历的图**，这包括两个类-单个节点和整个图。

我们将用一个名为GraphNode的接口来表示我们的各个节点：
```java
public interface GraphNode {
    String getId();
}
```

每个节点都必须有一个ID，其他任何信息都特定于此特定图，对于通用解决方案而言并非必需；**这些类是简单的Java Bean，没有特殊逻辑**。

我们的整体图由一个简称为Graph的类来表示：
```java
public class Graph<T extends GraphNode> {
    private final Set<T> nodes;
    private final Map<String, Set<String>> connections;

    public T getNode(String id) {
        return nodes.stream()
                .filter(node -> node.getId().equals(id))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("No node found with ID"));
    }

    public Set<T> getConnections(T node) {
        return connections.get(node.getId()).stream()
                .map(this::getNode)
                .collect(Collectors.toSet());
    }
}
```

**这会存储图中的所有节点，并了解哪些节点与哪些节点相连**。然后我们可以通过ID获取任何节点，或获取与给定节点相连的所有节点。

此时，我们能够表示我们想要的任意形式的图，任意数量的节点之间有任意数量的边。

### 5.2 我们的路线步骤

我们接下来需要的是通过图表寻找路线的机制。

**第一部分是生成任意两个节点之间的分数的方法**，我们将Scorer接口用于下一个节点的分数和目标节点的估计值：
```java
public interface Scorer<T extends GraphNode> {
    double computeCost(T from, T to);
}
```

给定一个起点和一个终点节点，我们就可以得到在它们之间旅行的分数。

**我们还需要一个包装器来包装我们的节点，以便包含一些额外的信息**。这里我们使用的是RouteNode而不是GraphNode-因为它是我们计算路线中的一个节点，而不是整个图中的节点：
```java
class RouteNode<T extends GraphNode> implements Comparable<RouteNode> {
    private final T current;
    private T previous;
    private double routeScore;
    private double estimatedScore;

    RouteNode(T current) {
        this(current, null, Double.POSITIVE_INFINITY, Double.POSITIVE_INFINITY);
    }

    RouteNode(T current, T previous, double routeScore, double estimatedScore) {
        this.current = current;
        this.previous = previous;
        this.routeScore = routeScore;
        this.estimatedScore = estimatedScore;
    }
}
```

**与GraphNode类似，这些是简单的Java Bean，用于存储每个节点的当前状态，以便进行当前的路由计算**。我们为常见情况(即首次访问某个节点且尚无关于该节点的其他信息)提供了一个简单的构造函数。

**不过，它们也需要可比较，这样我们才能在算法中根据估计分数对它们进行排序**，这意味着需要添加一个compareTo()方法来满足Comparable接口的要求：
```java
@Override
public int compareTo(RouteNode other) {
    if (this.estimatedScore > other.estimatedScore) {
        return 1;
    } else if (this.estimatedScore < other.estimatedScore) {
        return -1;
    } else {
        return 0;
    }
}
```

### 5.3 寻找路线

现在我们可以在图中实际生成路线了，这将是一个名为RouteFinder的类：
```java
public class RouteFinder<T extends GraphNode> {
    private final Graph<T> graph;
    private final Scorer<T> nextNodeScorer;
    private final Scorer<T> targetScorer;

    public List<T> findRoute(T from, T to) {
        throw new IllegalStateException("No route found");
    }
}
```

**我们有一个用于寻找路线的图，以及两个计分器**-一个用于计算下一个节点的准确分数，另一个用于计算到达目的地的估计分数。我们还有一种方法来获取起始节点和终止节点，并计算两者之间的最佳路线。

这个方法就是我们的A\*算法。其余代码都放在这个方法里。

我们从一些基本设置开始-我们可以将其视为下一步的节点的“开放集”，以及迄今为止我们访问过的每个节点的Map以及我们对它的了解：
```java
Queue<RouteNode> openSet = new PriorityQueue<>();
Map<T, RouteNode<T>> allNodes = new HashMap<>();

RouteNode<T> start = new RouteNode<>(from, null, 0d, targetScorer.computeCost(from, to));
openSet.add(start);
allNodes.put(from, start);
```

**我们的开放集最初只有一个节点-我们的起点**，它没有前一个节点，到达起点的得分为0，并且我们估算了它与终点的距离。

对开放集使用PriorityQueue意味着我们可以根据之前的compareTo()方法自动从中获取最佳条目。

现在我们进行迭代，直到我们用完了要查看的节点，或者最佳可用节点是我们的目的地：
```java
while (!openSet.isEmpty()) {
    RouteNode<T> next = openSet.poll();
    if (next.getCurrent().equals(to)) {
        List<T> route = new ArrayList<>();
        RouteNode<T> current = next;
        do {
            route.add(0, current.getCurrent());
            current = allNodes.get(current.getPrevious());
        } while (current != null);
        return route;
    }

    // ...
```

当我们找到目的地后，我们可以通过反复查看前一个节点来构建路线，直到到达起点。

**接下来，如果我们还没有到达目的地，我们可以想想下一步该怎么做**：
```java
    graph.getConnections(next.getCurrent()).forEach(connection -> { 
        RouteNode<T> nextNode = allNodes.getOrDefault(connection, new RouteNode<>(connection));
        allNodes.put(connection, nextNode);

        double newScore = next.getRouteScore() + nextNodeScorer.computeCost(next.getCurrent(), connection);
        if (newScore < nextNode.getRouteScore()) {
            nextNode.setPrevious(next.getCurrent());
            nextNode.setRouteScore(newScore);
            nextNode.setEstimatedScore(newScore + targetScorer.computeCost(connection, to));
            openSet.add(nextNode);
        }
    });

    throw new IllegalStateException("No route found");
}
```

这里，我们遍历图中已连接的节点。对于每个节点，我们都会获取其RouteNode-如果需要，我们会创建一个新的RouteNode。

然后，我们计算此节点的新分数，看看它是否比之前的得分更低。如果是，我们就更新它以匹配这条新路线，并将其添加到开放集以供下次考虑。

这就是整个算法，我们不断重复这个过程，直到我们达到目标或无法达到目标。

### 5.4 伦敦地铁的具体细节

到目前为止，我们拥有的是一个通用的A\*寻路器，但它缺少我们确切用例所需的细节，这意味着我们需要GraphNode和Scorer的具体实现。

我们的节点是地铁上的车站，我们将使用Station类对它们进行建模：
```java
public class Station implements GraphNode {
    private final String id;
    private final String name;
    private final double latitude;
    private final double longitude;
}
```

name对于查看输出很有用，latitude和longitude是为了我们的评分。

在这个场景中，我们只需要一个Scorer实现，我们将使用[Haversine公式](https://www.baeldung.com/cs/haversine-formula)来计算两对纬度/经度之间的直线距离：
```java
public class HaversineScorer implements Scorer<Station> {
    @Override
    public double computeCost(Station from, Station to) {
        double R = 6372.8; // Earth's Radius, in kilometers

        double dLat = Math.toRadians(to.getLatitude() - from.getLatitude());
        double dLon = Math.toRadians(to.getLongitude() - from.getLongitude());
        double lat1 = Math.toRadians(from.getLatitude());
        double lat2 = Math.toRadians(to.getLatitude());

        double a = Math.pow(Math.sin(dLat / 2),2)
                + Math.pow(Math.sin(dLon / 2),2) * Math.cos(lat1) * Math.cos(lat2);
        double c = 2 * Math.asin(Math.sqrt(a));
        return R * c;
    }
}
```

现在，我们几乎已经掌握了计算任意两对站点之间路径所需的一切，唯一缺少的是它们之间的连接图。

让我们用它来规划一条路线，我们将生成一条从Earl's Court到Angel的路线，这条路线有多种不同的出行选择，至少有两条地铁线路：
```java
public void findRoute() {
    List<Station> route = routeFinder.findRoute(underground.getNode("74"), underground.getNode("7"));

    System.out.println(route.stream().map(Station::getName).collect(Collectors.toList()));
}
```

**这将生成一条路线：Earl’s Court -> South Kensington -> Green Park -> Euston -> Angel**。

许多人会选择的明显路线可能是Earl’s Count -> Monument -> Angel，因为这条路线变化较少。相反，**这条路线明显更直接，尽管这意味着变化更多**。

## 6. 总结

在本文中，我们了解了A\*算法是什么、它的工作原理以及如何在自己的项目中实现它。