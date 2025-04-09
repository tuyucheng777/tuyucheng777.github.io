---
layout: post
title:  Java中的Boruvka最小生成树算法
category: algorithms
copyright: algorithms
excerpt: Boruvka最小生成树
---

## 1. 概述

在本教程中，**我们将研究Boruvka算法的Java实现，该算法用于查找边加权图的最小生成树(MST)**。

它早于[Prim算法](https://www.baeldung.com/java-prim-algorithm)和[Kruskal](https://www.baeldung.com/java-spanning-trees-kruskal)算法，但仍然可以被认为是两者的结合。

## 2. Boruvka算法

让我们先了解一下算法的历史，然后再了解算法本身。

### 2.1 历史

1926年，[Otakar Boruvka](https://en.wikipedia.org/wiki/Otakar_Borůvka)首次提出了一种查找给定图的MST的方法，这远在计算机出现之前，并且实际上是为了设计高效的电力分配系统而建模的。

乔治·索林(Georges Sollin)于1965年重新发现了它，并将其用于并行计算。

### 2.2 算法

该算法的核心思想是从一堆树开始，每个顶点代表一棵孤立树。然后，我们需要不断添加边以减少孤立树的数量，直到我们得到一棵连通的树。

让我们通过一个示例图来逐步了解这一点：

- 步骤0：创建图
- 步骤1：从一堆不相连的树开始(树的数量 = 顶点的数量)
- 步骤2：如果存在不相连的树，则对于每棵不相连的树：
  - 找到重量较小的边
  - 添加此边以连接另一棵树

![](/assets/images/2025/algorithms/javaboruvkaalgorithm01.png)

## 3. Java实现

现在让我们看看如何用Java实现它。

### 3.1 UnionFind数据结构

首先，**我们需要一个数据结构来存储顶点的父节点和等级**。

为此，我们定义一个UnionFind类，它有两种方法：union和find：
```java
public class UnionFind {
    private int[] parents;
    private int[] ranks;

    public UnionFind(int n) {
        parents = new int[n];
        ranks = new int[n];
        for (int i = 0; i < n; i++) {
            parents[i] = i;
            ranks[i] = 0;
        }
    }

    public int find(int u) {
        while (u != parents[u]) {
            u = parents[u];
        }
        return u;
    }

    public void union(int u, int v) {
        int uParent = find(u);
        int vParent = find(v);
        if (uParent == vParent) {
            return;
        }

        if (ranks[uParent] < ranks[vParent]) { 
            parents[uParent] = vParent; 
        } else if (ranks[uParent] > ranks[vParent]) {
            parents[vParent] = uParent;
        } else {
            parents[vParent] = uParent;
            ranks[uParent]++;
        }
    }
}
```

我们可以将此类视为一种辅助结构，用于维护顶点之间的关系并逐步建立MST。

**要确定两个顶点u和v是否属于同一棵树，我们会检查find(u)是否与find(v)返回相同的父节点**。union方法用于合并树，我们稍后会看到它的用法。

### 3.2 用户输入图

现在我们需要一种方法来从用户那里获取图的顶点和边，并将它们映射到我们可以在运行时在算法中使用的对象。

因为我们将使用JUnit来测试我们的算法，所以这部分内容放在@Before方法中：
```java
@BeforeEach
public void setup() {
    graph = ValueGraphBuilder.undirected().build();
    graph.putEdgeValue(0, 1, 8);
    graph.putEdgeValue(0, 2, 5);
    graph.putEdgeValue(1, 2, 9);
    graph.putEdgeValue(1, 3, 11);
    graph.putEdgeValue(2, 3, 15);
    graph.putEdgeValue(2, 4, 10);
    graph.putEdgeValue(3, 4, 7);
}
```

这里我们使用了Guava的MutableValueGraph<Integer, Integer\>来存储我们的图，然后我们使用ValueGraphBuilder来构建一个无向加权图。

方法putEdgeValue需要3个参数，两个Integer表示顶点，第三个Integer表示其权重，如MutableValueGraph的泛型类型声明所指定。

我们可以看到，这和我们之前的图中显示的输入相同。

### 3.3 导出最小生成树

最后，我们来到了问题的关键，即算法的实现。

我们将在名为BoruvkaMST的类中执行此操作，首先，我们声明几个实例变量：
```java
public class BoruvkaMST {
    private static MutableValueGraph<Integer, Integer> mst = ValueGraphBuilder.undirected().build();
    private static int totalWeight;
}
```

可以看到，我们在这里使用MutableValueGraph<Integer, Integer\>来表示MST。

其次，我们定义一个构造函数，所有主要的事情都在这里发生。它接收一个参数-我们之前构建的图。

它所做的第一件事是初始化输入图顶点的UnionFind，最初，所有顶点都是它们自己的父级，每个顶点的等级为0：
```java
public BoruvkaMST(MutableValueGraph<Integer, Integer> graph) {
    int size = graph.nodes().size();
    UnionFind uf = new UnionFind(size);
```

接下来，我们将创建一个循环，定义创建MST所需的迭代次数-最多log V次或直到我们有V - 1条边，其中V是顶点的数量：
```java
for (int t = 1; t < size && mst.edges().size() < size - 1; t = t + t) {
    EndpointPair<Integer>[] closestEdgeArray = new EndpointPair[size];
```

在这里我们还初始化一个边数组closestEdgeArray，来存储最近的、权重较小的边。

之后，我们将定义一个内部for循环来遍历图的所有边以填充我们的nearestEdgeArray。

如果两个顶点的父节点相同，则它们就是同一棵树，我们不会将其添加到数组中。否则，我们将当前边的权重与其父顶点的边的权重进行比较，如果较小，则将其添加到nearestEdgeArray：
```java
for (EndpointPair<Integer> edge : graph.edges()) {
    int u = edge.nodeU();
    int v = edge.nodeV();
    int uParent = uf.find(u);
    int vParent = uf.find(v);
    
    if (uParent == vParent) {
        continue;
    }

    int weight = graph.edgeValueOrDefault(u, v, 0);

    if (closestEdgeArray[uParent] == null) {
        closestEdgeArray[uParent] = edge;
    }
    if (closestEdgeArray[vParent] == null) {
        closestEdgeArray[vParent] = edge;
    }

    int uParentWeight = graph.edgeValueOrDefault(closestEdgeArray[uParent].nodeU(),
      closestEdgeArray[uParent].nodeV(), 0);
    int vParentWeight = graph.edgeValueOrDefault(closestEdgeArray[vParent].nodeU(),
      closestEdgeArray[vParent].nodeV(), 0);

    if (weight < uParentWeight) {
        closestEdgeArray[uParent] = edge;
    }
    if (weight < vParentWeight) {
        closestEdgeArray[vParent] = edge;
    }
}
```

然后，我们将定义第2个内循环来创建一棵树，我们将从上一步将边添加到这棵树中，而无需两次添加相同的边。此外，我们将对UnionFind执行并集，以导出和存储新创建的树的顶点的父节点和等级：
```java
for (int i = 0; i < size; i++) {
    EndpointPair<Integer> edge = closestEdgeArray[i];
    if (edge != null) {
        int u = edge.nodeU();
        int v = edge.nodeV();
        int weight = graph.edgeValueOrDefault(u, v, 0);
        if (uf.find(u) != uf.find(v)) {
            mst.putEdgeValue(u, v, weight);
            totalWeight += weight;
            uf.union(u, v);
        }
    }
}
```

重复这些步骤最多log V次或直到我们有V - 1条边后，得到的树就是我们的MST。

## 4. 测试

最后我们来看一个简单的JUnit来验证我们的实现：
```java
@Test
void givenInputGraph_whenBoruvkaPerformed_thenMinimumSpanningTree() {
    BoruvkaMST boruvkaMST = new BoruvkaMST(graph);
    MutableValueGraph<Integer, Integer> mst = boruvkaMST.getMST();

    assertEquals(30, boruvkaMST.getTotalWeight());
    assertEquals(4, mst.getEdgeCount());
}
```

我们可以看到，**得到了权重为30、边数为4的MST，与图示示例相同**。

## 5. 总结

在本教程中，我们介绍了Boruvka算法的Java实现；**它的时间复杂度为O(E log V)，其中E是边数，V是顶点数**。