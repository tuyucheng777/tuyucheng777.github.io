---
layout: post
title:  检查Java图是否有循环
category: algorithms
copyright: algorithms
excerpt: 图
---

## 1. 概述

在本快速教程中，我们将学习如何**检测给定有向图中的循环**。

## 2. 图表示

在本教程中，我们使用邻接表[图表示](https://www.baeldung.com/java-graphs#graph_representations)。

首先，让我们从在Java中定义一个顶点开始：
```java
public class Vertex {
    private String label;
    private boolean beingVisited;
    private boolean visited;
    private List<Vertex> adjacencyList;

    public Vertex(String label) {
        this.label = label;
        this.adjacencyList = new ArrayList<>();
    }

    public void addNeighbor(Vertex adjacent) {
        this.adjacencyList.add(adjacent);
    }
    //getters and setters
}
```

**这里，顶点v的adjacencyList保存了与v相邻的所有顶点的列表**，addNeighbor()方法将相邻顶点添加到v的邻接列表中。

我们还定义了**两个boolean参数beingVisited和visited，分别表示该节点当前是否正在被访问或者已经被访问过**。

图可以被认为是一组通过边连接的顶点或节点。

那么，现在让我们用Java快速表示一个图：
```java
public class Graph {

    private List<Vertex> vertices;

    public Graph() {
        this.vertices = new ArrayList<>();
    }

    public void addVertex(Vertex vertex) {
        this.vertices.add(vertex);
    }

    public void addEdge(Vertex from, Vertex to) {
        from.addNeighbor(to);
    }

   // ...
}
```

我们将使用addVertex()和addEdge()方法在图中添加新的顶点和边。

## 3. 循环检测

为了检测有向图中的循环，**我们将使用DFS遍历的变体**：

- 选取一个未访问的顶点v并将其状态标记为beingVisited

- 对于v的每个相邻顶点u，检查：

  - 如果u已经处于beingVisited状态，则显然存在后向边，因此检测到了循环
  - 如果u仍处于未访问状态，我们将以深度优先的方式递归访问u

- 将顶点v的beingVisited标志更新为false，并将其visited标志更新为true

请注意，**我们图的所有顶点最初都处于未访问状态，因为它们的beingVisited和visited标志均初始化为false**。 

现在让我们看看我们的Java解决方案：
```java
public boolean hasCycle(Vertex sourceVertex) {
    sourceVertex.setBeingVisited(true);

    for (Vertex neighbor : sourceVertex.getAdjacencyList()) {
        if (neighbor.isBeingVisited()) {
            // backward edge exists
            return true;
        } else if (!neighbor.isVisited() && hasCycle(neighbor)) {
            return true;
        }
    }

    sourceVertex.setBeingVisited(false);
    sourceVertex.setVisited(true);
    return false;
}
```

我们可以使用图中的任何顶点作为源或起始顶点。

**对于断开连接的图，我们必须添加额外的包装方法**：
```java
public boolean hasCycle() {
    for (Vertex vertex : vertices) {
        if (!vertex.isVisited() && hasCycle(vertex)) {
            return true;
        }
    }
    return false;
}
```

这是为了确保我们访问断开连接的图的每个组件来检测循环。

## 4. 实现测试

让我们考虑下面的循环有向图：

![](/assets/images/2025/algorithms/javagraphhasacycle01.png)

我们可以快速编写一个JUnit来验证该图的hasCycle()方法：
```java
@Test
void givenGraph_whenCycleExists_thenReturnTrue() {
    Vertex vertexA = new Vertex("A");
    Vertex vertexB = new Vertex("B");
    Vertex vertexC = new Vertex("C");
    Vertex vertexD = new Vertex("D");

    Graph graph = new Graph();
    graph.addVertex(vertexA);
    graph.addVertex(vertexB);
    graph.addVertex(vertexC);
    graph.addVertex(vertexD);

    graph.addEdge(vertexA, vertexB);
    graph.addEdge(vertexB, vertexC);
    graph.addEdge(vertexC, vertexA);
    graph.addEdge(vertexD, vertexC);

    assertTrue(graph.hasCycle());

}
```

这里，我们的hasCycle()方法返回true，表示我们的图是循环的。

## 5. 总结

在本教程中，我们学习了如何在Java中检查给定有向图中是否存在循环。