---
layout: post
title:  Java中的Dijkstra最短路径算法
category: algorithms
copyright: algorithms
excerpt: Dijkstra
---

## 1. 概述

本文重点讨论最短路径问题(SPP)，它是图论中已知的基本理论问题之一，以及如何使用[Dijkstra算法](https://www.baeldung.com/cs/dijkstra)来解决该问题。

该算法的基本目标是确定起始节点和图的其余部分之间的最短路径。

## 2. 使用Dijkstra解决最短路径问题

给定一个正[加权图](https://www.baeldung.com/cs/weighted-vs-unweighted-graphs)和一个起始节点(A)，Dijkstra确定从源到图中所有目的地的最短路径和距离：

![](/assets/images/2025/algorithms/javadijkstra01.png)

[Dijkstra算法](https://www.baeldung.com/cs/prim-dijkstra-difference)的核心思想是不断消除起始节点和所有可能的目的地之间的较长路径。

为了跟踪该过程，我们需要有两组不同的节点，即已解决节点和未解决节点。

已解决节点是那些与源节点之间已知最小距离的节点；未解决节点集合收集了那些我们可以从源节点到达的节点，但我们不知道与起始节点之间的最小距离。

下面是使用Dijkstra解决SPP需要遵循的步骤列表：

- 将到startNode的距离设置为0
- 将所有其他距离设置为无限值
- 将startNode添加到未解决节点集中
- 当未解决节点集不为空时，我们：
  - 从未解决的节点集合中选择一个评估节点，评估节点应该是距离源距离最短的节点
  - 通过在每次评估时保持最小距离来计算与直接邻居的新距离
  - 将尚未解决的邻居添加到未解决节点集中

这些步骤可以归纳为两个阶段：初始化和评估；让我们看看这如何应用于我们的示例图。

### 2.1 初始化

在开始探索图中的所有路径之前，我们首先需要初始化除源之外的所有具有无限距离和未知前任的节点。

作为初始化过程的一部分，我们需要将值0分配给节点A(我们知道从节点A到节点A的距离显然为0)。

因此，图的其余部分中的每个节点将通过前任和距离来区分：

![](/assets/images/2025/algorithms/javadijkstra02.png)

为了完成初始化过程，我们需要将节点A添加到未解决节点集，以便在评估步骤中首先选中它。请记住，已解决节点集仍然为空。

### 2.2 评估

现在我们已经初始化了图，我们选择与未解决集距离最短的节点，然后评估不在解决节点中的所有相邻节点：

![](/assets/images/2025/algorithms/javadijkstra03.png)

这个想法是将边权重添加到评估节点距离，然后将其与目的地的距离进行比较。例如对于节点B，0 + 10低于INFINITY，因此节点B的新距离为10，新的前任为A，节点C也是如此。

然后，节点A从未解决节点集移至解决节点集。

将节点B和C添加到未解决节点中，因为它们可以访问，但需要进行评估。

现在我们在未解决集中有两个节点，我们选择距离最小的节点(节点B)，然后我们重复，直到我们解决图中的所有节点：

![](/assets/images/2025/algorithms/javadijkstra04.png)

下表总结了评估步骤中执行的迭代：

| 迭代    | 未解决   | 已解决       | 评估节点 | A | B    | C    | D    | E    | F        |
|-------|-------|-----------|------|---|------|------|------|------| -------- |
| 1     | A     | –         | A    | 0 | A-10 | A-15 | X-∞  | X-∞  | X-∞      |
| 2     | B、C   | A         | B    | 0 | A-10 | A-15 | B-22 | X-∞  | B-25     |
| 3     | C、F、D | A、B       | C    | 0 | A-10 | A-15 | B-22 | C-25 | B-25     |
| 4     | D、E、F | A、B、C     | D    | 0 | A-10 | A-15 | B-22 | D-24 | D-23     |
| 5     | E、F   | A、B、C、D   | F    | 0 | A-10 | A-15 | B-22 | D-24 | D-23     |
| 6     | E     | A、B、C、D、F | E    | 0 | A-10 | A-15 | B-22 | D-24 | D-23     |
| Final | –     | ALL       | NONE | 0 | A-10 | A-15 | B-22 | D-24 | D-23 |

例如，符号B-22表示节点B是紧接的前任，与节点A的总距离为22。

最后，我们可以计算从节点A出发的最短路径如下：

- 节点B：A –> B(总距离 = 10)
- 节点C：A –> C(总距离 = 15)
- 节点D：A –> B –> D(总距离 = 22)
- 节点E：A –> B –> D –> E(总距离 = 24)
- 节点F：A –> B –> D –> F(总距离 = 23)

## 3. Java实现

在这个简单的实现中，我们将图表示为一组节点：

```java
public class Graph {

  private Set<Node> nodes = new HashSet<>();

  public void addNode(Node nodeA) {
    nodes.add(nodeA);
  }

  // getters and setters 
}
```

一个节点可以用一个name、一个引用shortestPath的LinkedList、一个与源的distance以及一个名为adjacentNodes的邻接列表来描述：

```java
public class Node {
    
    private String name;
    
    private List<Node> shortestPath = new LinkedList<>();
    
    private Integer distance = Integer.MAX_VALUE;
    
    Map<Node, Integer> adjacentNodes = new HashMap<>();

    public void addDestination(Node destination, int distance) {
        adjacentNodes.put(destination, distance);
    }
 
    public Node(String name) {
        this.name = name;
    }
    
    // getters and setters
}
```

adjacentNodes属性用于将直接邻居与边长关联起来。这是邻接表的简化实现，与邻接矩阵相比，它更适合Dijkstra算法。

至于shortestPath属性，它是一个节点列表，描述从起始节点计算的最短路径。

默认情况下，所有节点距离都用Integer.MAX_VALUE初始化，以模拟初始化步骤中描述的无限距离。

现在，我们来实现Dijkstra算法：

```java
public static Graph calculateShortestPathFromSource(Graph graph, Node source) {
    source.setDistance(0);

    Set<Node> settledNodes = new HashSet<>();
    Set<Node> unsettledNodes = new HashSet<>();

    unsettledNodes.add(source);

    while (unsettledNodes.size() != 0) {
        Node currentNode = getLowestDistanceNode(unsettledNodes);
        unsettledNodes.remove(currentNode);
        for (Entry < Node, Integer> adjacencyPair: currentNode.getAdjacentNodes().entrySet()) {
            Node adjacentNode = adjacencyPair.getKey();
            Integer edgeWeight = adjacencyPair.getValue();
            if (!settledNodes.contains(adjacentNode)) {
                calculateMinimumDistance(adjacentNode, edgeWeight, currentNode);
                unsettledNodes.add(adjacentNode);
            }
        }
        settledNodes.add(currentNode);
    }
    return graph;
}
```

getLowestDistanceNode()方法返回与未解决节点集距离最短的节点，而calculateMinimumDistance()方法在遵循新探索的路径时将实际距离与新计算的距离进行比较：

```java
private static Node getLowestDistanceNode(Set<Node> unsettledNodes) {
    Node lowestDistanceNode = null;
    int lowestDistance = Integer.MAX_VALUE;
    for (Node node: unsettledNodes) {
        int nodeDistance = node.getDistance();
        if (nodeDistance < lowestDistance) {
            lowestDistance = nodeDistance;
            lowestDistanceNode = node;
        }
    }
    return lowestDistanceNode;
}
private static void CalculateMinimumDistance(Node evaluationNode, Integer edgeWeigh, Node sourceNode) {
    Integer sourceDistance = sourceNode.getDistance();
    if (sourceDistance + edgeWeigh < evaluationNode.getDistance()) {
        evaluationNode.setDistance(sourceDistance + edgeWeigh);
        LinkedList<Node> shortestPath = new LinkedList<>(sourceNode.getShortestPath());
        shortestPath.add(sourceNode);
        evaluationNode.setShortestPath(shortestPath);
    }
}
```

现在所有必要的部分都已准备就绪，让我们将Dijkstra算法应用于本文主题的样本图：

```java
Node nodeA = new Node("A");
Node nodeB = new Node("B");
Node nodeC = new Node("C");
Node nodeD = new Node("D"); 
Node nodeE = new Node("E");
Node nodeF = new Node("F");

nodeA.addDestination(nodeB, 10);
nodeA.addDestination(nodeC, 15);

nodeB.addDestination(nodeD, 12);
nodeB.addDestination(nodeF, 15);

nodeC.addDestination(nodeE, 10);

nodeD.addDestination(nodeE, 2);
nodeD.addDestination(nodeF, 1);

nodeF.addDestination(nodeE, 5);

Graph graph = new Graph();

graph.addNode(nodeA);
graph.addNode(nodeB);
graph.addNode(nodeC);
graph.addNode(nodeD);
graph.addNode(nodeE);
graph.addNode(nodeF);

graph = Dijkstra.calculateShortestPathFromSource(graph, nodeA);
```

计算后，为图中的每个节点设置了shortestPath和distance属性，我们可以迭代它们以验证结果是否与上一节中找到的结果完全匹配。

## 4. 总结

在本文中，我们介绍了Dijkstra算法如何解决SPP，以及如何在Java中实现它。