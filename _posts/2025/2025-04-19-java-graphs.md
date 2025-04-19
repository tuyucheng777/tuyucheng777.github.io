---
layout: post
title:  Java中的图
category: algorithms
copyright: algorithms
excerpt: 图
---

## 1. 概述

在本教程中，我们将了解[图](https://www.baeldung.com/cs/graphs)作为数据结构的基本概念。

我们还将探索它在Java中的实现以及图上可能进行的各种操作，并讨论提供图实现的Java库。

## 2. 图数据结构

**图是一种用于存储连接数据的数据结构**，例如社交媒体平台上的人员网络。

图由[顶点和边](https://www.baeldung.com/cs/graph-theory-intro)组成，**顶点(或节点)表示实体(例如，人)，边表示实体之间的关系(例如，一个人的友谊)**。

让我们定义一个简单的图表来更好地理解这一点：

![](/assets/images/2025/algorithms/javagraphs01.png)

这里，我们定义了一个包含5个顶点和6条边的简单图。圆圈代表人物，连接两个顶点的线代表在线门户网站上的好友。

根据边的属性，这个简单的图可以有几种变化，我们将在下一节中简要介绍它们。

但是，本教程中我们只关注Java示例的简单图。

### 2.1 有向图

到目前为止，我们定义的图都包含无方向的边。**如果这些边有方向**，则生成的图称为[有向图](https://www.baeldung.com/cs/graphs-directed-vs-undirected-graph)。

一个例子是可以表示谁在在线门户网站上以友谊形式发送了好友请求：

![](/assets/images/2025/algorithms/javagraphs02.png)

这里我们可以看到边有一个固定的方向，边也可以是双向的。

### 2.2 加权图

同样，我们的简单图具有无偏或无加权的边。

相反，**如果这些边具有相对权重**，则该图称为加权图。

一个实际应用的例子是，可以在在线门户网站上显示友谊的相对持续时间：

![](/assets/images/2025/algorithms/javagraphs03.png)

在这里，我们可以看到边具有与之相关的权重，这为它们提供了相对的含义。

## 3. 图表示

图可以用不同的形式表示，例如邻接矩阵和邻接表，每种形式在不同的设置下都有其优缺点。

我们将在本节介绍这些图表示。

### 3.1 邻接矩阵

**邻接矩阵是一个方阵，其维度等于图中顶点的数量**。

矩阵元素通常具有0或1的值，值1表示行和列中的顶点相邻，否则为0。

让我们看看上一节中的简单图的邻接矩阵是什么样的：

![](/assets/images/2025/algorithms/javagraphs04.png)

这种表示形式实现起来相当容易，查询起来也比较高效。但是，就空间占用而言，效率较低。

### 3.2 邻接表

邻接表只不过是一个列表的数组，数组大小等于图中顶点的数量。

**特定数组索引处的列表表示该数组索引所表示的顶点的相邻顶点**。

让我们看看上一节中的简单图的邻接表是什么样的：

![](/assets/images/2025/algorithms/javagraphs05.png)

这种表示形式创建起来相对困难，查询效率也较低。但是，它提供了更好的空间效率。

在本教程中，我们将使用邻接表来表示图。

## 4. Java中的图

**Java没有图数据结构的默认实现**。

但是，我们可以使用Java集合来实现图。

**让我们首先定义一个顶点**：

```java
class Vertex {
    String label;
    Vertex(String label) {
        this.label = label;
    }

    // equals and hashCode
}
```

上述顶点的定义仅具有标签，但它可以代表任何可能的实体，例如Person或City。

**另外，请注意，我们必须重写equals()和hashCode()方法，因为这些方法是使用Java集合所必需的**。

正如我们之前讨论的，图只不过是顶点和边的集合，可以表示为邻接矩阵或邻接表。

让我们看看如何使用邻接表来定义它：

```java
class Graph {
    private Map<Vertex, List<Vertex>> adjVertices;
    
    // standard constructor, getters, setters
}
```

我们可以看到，Graph类使用Java Collections中的Map来定义邻接表。

图数据结构上可以进行多种操作，例如创建、更新或搜索图。

我们将介绍一些更常见的操作，并了解如何在Java中实现它们。

## 5. 图突变操作

首先，我们将定义一些方法来改变图数据结构。

让我们定义添加和删除顶点的方法：

```java
void addVertex(String label) {
    adjVertices.putIfAbsent(new Vertex(label), new ArrayList<>());
}

void removeVertex(String label) {
    Vertex v = new Vertex(label);
    adjVertices.values().stream().forEach(e -> e.remove(v));
    adjVertices.remove(new Vertex(label));
}
```

这些方法只是从顶点集合中添加和删除元素。

现在，我们还定义一个添加边的方法：

```java
void addEdge(String label1, String label2) {
    Vertex v1 = new Vertex(label1);
    Vertex v2 = new Vertex(label2);
    adjVertices.get(v1).add(v2);
    adjVertices.get(v2).add(v1);
}
```

此方法创建一个新的Edge并更新相邻顶点Map。

类似地，我们将定义removeEdge()方法：

```java
void removeEdge(String label1, String label2) {
    Vertex v1 = new Vertex(label1);
    Vertex v2 = new Vertex(label2);
    List<Vertex> eV1 = adjVertices.get(v1);
    List<Vertex> eV2 = adjVertices.get(v2);
    if (eV1 != null)
        eV1.remove(v2);
    if (eV2 != null)
        eV2.remove(v1);
}
```

接下来，让我们看看如何使用迄今为止定义的方法创建我们之前绘制的简单图：

```java
Graph createGraph() {
    Graph graph = new Graph();
    graph.addVertex("Bob");
    graph.addVertex("Alice");
    graph.addVertex("Mark");
    graph.addVertex("Rob");
    graph.addVertex("Maria");
    graph.addEdge("Bob", "Alice");
    graph.addEdge("Bob", "Rob");
    graph.addEdge("Alice", "Mark");
    graph.addEdge("Rob", "Mark");
    graph.addEdge("Alice", "Maria");
    graph.addEdge("Rob", "Maria");
    return graph;
}
```

最后，我们将定义一个方法来[获取特定顶点的相邻顶点](https://www.baeldung.com/cs/shortest-path-visiting-all-nodes)：

```java
List<Vertex> getAdjVertices(String label) {
    return adjVertices.get(new Vertex(label));
}
```

## 6. 遍历图

现在我们有了图数据结构和创建和更新它的函数，我们可以定义一些用于遍历图的额外函数。

我们需要遍历图来执行任何有意义的操作，例如在图内搜索。

遍历图的两种可能方法是**[深度优先遍历](https://www.baeldung.com/java-depth-first-search)和[广度优先遍历](https://www.baeldung.com/java-breadth-first-search)**。

### 6.1 深度优先遍历

**[深度优先遍历](https://www.baeldung.com/cs/depth-first-traversal-methods)从任意根顶点开始，沿着每个分支尽可能深入地探索顶点，然后再探索同一级别的顶点**。

让我们定义一个方法来执行深度优先遍历：

```java
Set<String> depthFirstTraversal(Graph graph, String root) {
    Set<String> visited = new LinkedHashSet<String>();
    Stack<String> stack = new Stack<String>();
    stack.push(root);
    while (!stack.isEmpty()) {
        String vertex = stack.pop();
        if (!visited.contains(vertex)) {
            visited.add(vertex);
            for (Vertex v : graph.getAdjVertices(vertex)) {
                stack.push(v.label);
            }
        }
    }
    return visited;
}
```

这里，**我们使用Stack来存储需要遍历的顶点**。

让我们使用上一节中创建的图运行深度优先遍历的JUnit测试：

```java
@Test
public void givenAGraph_whenTraversingDepthFirst_thenExpectedResult() {
    Graph graph = createGraph();
    assertEquals("[Bob, Rob, Maria, Alice, Mark]", GraphTraversal.depthFirstTraversal(graph, "Bob").toString());
}
```

请注意，我们在这里使用顶点“Bob”作为遍历的根，但这可以是任何其他顶点。

### 6.2 广度优先遍历

**[相比之下](https://www.baeldung.com/cs/dfs-vs-bfs)，广度优先遍历从任意根顶点开始，并探索同一级别的所有相邻顶点，然后再深入到图中**。

现在，让我们定义一个方法来执行广度优先遍历：

```java
Set<String> breadthFirstTraversal(Graph graph, String root) {
    Set<String> visited = new LinkedHashSet<String>();
    Queue<String> queue = new LinkedList<String>();
    queue.add(root);
    visited.add(root);
    while (!queue.isEmpty()) {
        String vertex = queue.poll();
        for (Vertex v : graph.getAdjVertices(vertex)) {
            if (!visited.contains(v.label)) {
                visited.add(v.label);
                queue.add(v.label);
            }
        }
    }
    return visited;
}
```

请注意，**广度优先遍历使用队列来存储需要遍历的顶点**。

让我们使用相同的图运行广度优先遍历的JUnit测试：

```java
@Test
public void givenAGraph_whenTraversingBreadthFirst_thenExpectedResult() {
    Graph graph = createGraph();
    assertEquals("[Bob, Alice, Rob, Mark, Maria]", GraphTraversal.breadthFirstTraversal(graph, "Bob").toString());
}
```

同样，根顶点(此处为“Bob”)也可以是任何其他顶点。

## 7. 初始化和打印图

我们可以使用Java应用程序初始化并打印图，为此，我们创建一个名为AppToPrintGraph的Java应用程序。我们需要向此类添加一个方法来初始化图，我们添加一个类方法createGraph，这样我们无需先创建类实例即可调用它：

```java
static Graph createGraph() {
    Graph graph = new Graph();
    graph.addVertex("Bob");
    graph.addVertex("Alice");
    graph.addVertex("Mark");
    graph.addVertex("Rob");
    graph.addVertex("Maria");
    graph.addEdge("Bob", "Maria");
    graph.addEdge("Bob", "Mark");
    graph.addEdge("Maria", "Alice");
    graph.addEdge("Mark", "Alice");
    graph.addEdge("Maria", "Rob");
    graph.addEdge("Mark", "Rob");

    return graph;
}
```

根据Java应用程序的要求，它应该包含main方法；因此，让我们添加一个。在main方法中，让我们调用createGraph方法来初始化一个图，并对其进行广度优先遍历，然后打印遍历结果：

```java
public static void main(String[] args) {
    Graph graph = createGraph();
    Set result = GraphTraversal.breadthFirstTraversal(graph, "Bob");
       
    System.out.println(result.toString());   
}
```

我们可以运行此应用程序将示例图打印到标准输出：

```text
[Bob, Maria, Mark, Alice, Rob]
```

## 8. Java图库

没有必要总是用Java从头开始实现图，一些成熟的开源库提供了图的实现。

我们将在接下来的几个小节中介绍其中一些库。

### 8.1 JGraphT

[JGraphT](https://jgrapht.org/)是Java中最流行的图数据结构库之一，它允许创建简单图、有向图、加权图等。

此外，它还为图数据结构提供了许多可能的算法，我们之前的一篇教程对[JGraphT](https://www.baeldung.com/jgrapht)进行了更详细的介绍。

### 8.2 Google Guava

[Google Guava](https://github.com/google/guava/wiki/GraphsExplained)是一组Java库，提供包括图数据结构及其算法在内的一系列功能。

它支持创建简单的Graph、ValueGraph和Network，这些可以定义为Mutable或Immutable。

### 8.3 Apache Commons

[Apache Commons](http://commons.apache.org/sandbox/commons-graph/)是一个提供可重用Java组件的Apache项目，其中包括Commons Graph，一个用于创建和管理图数据结构的工具包，该工具包还提供了用于操作该数据结构的通用图算法。

### 8.4 Sourceforge JUNG

[Java Universal Network/Graph(JUNG)](http://jung.sourceforge.net/)是一个Java框架，它提供了一种可扩展语言，用于对可以表示为图的任何数据进行建模、分析和可视化。

JUNG支持许多算法，包括聚类、分解和优化等例程。

这些库提供了许多基于图数据结构的实现。此外，还有一些基于图的更强大的框架，例如[Apache Giraph](http://giraph.apache.org/)(目前Facebook用它来分析用户生成的图谱)，以及[Apache TinkerPop](http://tinkerpop.apache.org/)(通常用于图数据库之上)。

## 9. 总结

在本文中，我们讨论了图作为一种数据结构及其表示形式。我们使用Java集合类定义了一个非常简单的图，并定义了该图的常见遍历方法。此外，我们还学习了如何初始化和打印图。

我们还简要讨论了Java平台之外提供图实现的各种Java库。