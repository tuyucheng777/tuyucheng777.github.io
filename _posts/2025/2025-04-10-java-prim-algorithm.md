---
layout: post
title:  Prim算法的Java实现
category: algorithms
copyright: algorithms
excerpt: Log4j
---

## 1. 简介

在本教程中，我们首先了解什么是最小生成树。然后，我们将使用[Prim算法](https://www.baeldung.com/cs/prim-algorithm)来查找最小生成树。

## 2. 最小生成树

**最小生成树(MST)是一个加权、无向、连通的图，通过删除较重的边，使其总边权重最小化**。换句话说，我们保持图的所有顶点不变，但可能会移除一些边，以使所有边的总权重最小。

我们从加权图开始，因为如果边本身没有权重，那么最小化边的总权重就没有意义。我们来看一个示例图：

![](/assets/images/2025/algorithms/javaprimalgorithm01.png)

上图是一个带权、无向的连通图，以下是该图的MST：

![](/assets/images/2025/algorithms/javaprimalgorithm02.png)

但是，图的MST并不是唯一的，如果图有多个MST，则每个MST的总边权重相同。

## 3. Prim算法

**Prim算法以加权、无向、连通图作为输入，并返回该图的MST作为输出**。

**它以贪婪的方式工作**；第一步，它选择一个任意顶点，此后，**每一步都会将最近的顶点添加到迄今为止构建的树中**，直到没有断开的顶点为止。

让我们逐步在该图上运行Prim算法：

![](/assets/images/2025/algorithms/javaprimalgorithm03.png)

假设算法的起始顶点是B，我们有3个选择：A、C和E，这3个边对应的权重分别为2、2和5，因此最小值为2。在本例中，我们有两条边权重为2，因此我们可以选择其中任意一条(选择哪一条都无所谓)，我们选择A：

![](/assets/images/2025/algorithms/javaprimalgorithm04.png)

现在我们有了一棵树，它有两个顶点A和B，我们可以选择A或B中任何一条尚未添加的边，只要它们指向一个未添加的顶点即可。因此，我们可以选择AC、BC或BE。

Prim算法选择最小值，即2，或BC：

![](/assets/images/2025/algorithms/javaprimalgorithm05.png)

现在我们有了一棵树，它有3个顶点和3条可能的边：CD、CE或BE。AC不包括在内，因为它不会在树中添加新顶点。这三者之间的最小权重为1。

但是，有两条边的权重均为1。因此，Prim算法在此步骤中选择其中一条边(同样，选择哪一条并不重要)：

![](/assets/images/2025/algorithms/javaprimalgorithm06.png)

现在只剩下一个顶点需要连接，因此我们可以从CE和BE中选择，连接我们的树的最小权重是1，Prim算法将选择它：

![](/assets/images/2025/algorithms/javaprimalgorithm07.png)

由于输入图的所有顶点现在都出现在输出树中，Prim算法结束。因此，这棵树是输入图的MST。

## 4. 实现

顶点和边构成了图，因此我们需要一个数据结构来存储这些元素，让我们创建Edge类：
```java
public class Edge {

    private int weight;
    private boolean isIncluded = false;
}
```

由于Prim算法适用于加权图，因此每条边都必须有一个权重，isIncluded显示边是否存在于最小生成树中。

现在，让我们添加Vertex类：
```java
public class Vertex {

    private String label = null;
    private Map<Vertex, Edge> edges = new HashMap<>();
    private boolean isVisited = false;
}
```

每个顶点可以可选地带有一个标签，我们使用边图来存储顶点之间的连接，最后，isVisited显示该顶点到目前为止是否已被Prim算法访问过。

让我们创建Prim类来实现逻辑：
```java
public class Prim {

    private List<Vertex> graph;
}
```

一个顶点列表足以存储整个图，因为每个Vertex内部都有一个Map<Vertex,Edge\>来标识所有连接。在Prim内部，我们创建一个run()方法：
```java
public void run() {
    if (graph.size() > 0) {
        graph.get(0).setVisited(true);
    }
    while (isDisconnected()) {
        Edge nextMinimum = new Edge(Integer.MAX_VALUE);
        Vertex nextVertex = graph.get(0);
        for (Vertex vertex : graph) {
            if (vertex.isVisited()) {
                Pair<Vertex, Edge> candidate = vertex.nextMinimum();
                if (candidate.getValue().getWeight() < nextMinimum.getWeight()) {
                    nextMinimum = candidate.getValue();
                    nextVertex = candidate.getKey();
                }
            }
        }
        nextMinimum.setIncluded(true);
        nextVertex.setVisited(true);
    }
}
```

我们首先将List<Vertex\> graph的第一个元素设置为已访问，第一个元素可以是任何顶点，具体取决于它们首先添加到列表中的顺序。如果存在尚未访问的顶点，则isDisconnected()返回true：
```java
private boolean isDisconnected() {
    for (Vertex vertex : graph) {
        if (!vertex.isVisited()) {
            return true;
        }
    }
    return false;
}
```

当最小生成树isDisconnected()时，我们循环遍历已经访问过的顶点，并找到具有最小权重的边作为nextVertex的候选：
```java
public Pair<Vertex, Edge> nextMinimum() {
    Edge nextMinimum = new Edge(Integer.MAX_VALUE);
    Vertex nextVertex = this;
    Iterator<Map.Entry<Vertex,Edge>> it = edges.entrySet()
            .iterator();
    while (it.hasNext()) {
        Map.Entry<Vertex,Edge> pair = it.next();
        if (!pair.getKey().isVisited()) {
            if (!pair.getValue().isIncluded()) {
                if (pair.getValue().getWeight() < nextMinimum.getWeight()) {
                    nextMinimum = pair.getValue();
                    nextVertex = pair.getKey();
                }
            }
        }
    }
    return new Pair<>(nextVertex, nextMinimum);
}
```

我们在主循环中找到所有候选的最小值并将其存储在nextVertex中，然后，我们将nextVertex设置为已访问。循环一直持续，直到所有顶点都被访问过。

最后，**每个isIncluded等于true的Edge都存在**。

请注意，由于nextMinimum()会遍历所有边，因此此实现的时间复杂度为O(V<sup>2</sup>)。如果我们将边存储在优先级队列中(按权重排序)，则该算法的执行时间复杂度为O(E log V)。

## 5. 测试

让我们用一个真实的例子来测试一下，首先，我们构建图：
```java
public static List<Vertex> createGraph() {
    List<Vertex> graph = new ArrayList<>();
    Vertex a = new Vertex("A");
    // ...
    Vertex e = new Vertex("E");
    Edge ab = new Edge(2);
    a.addEdge(b, ab);
    b.addEdge(a, ab);
    // ...
    Edge ce = new Edge(1);
    c.addEdge(e, ce);
    e.addEdge(c, ce);
    graph.add(a);
    // ...
    graph.add(e);
    return graph;
}
```

Prim类的构造函数接收该参数并将其存储在类内部，我们可以使用originalGraphToString()方法打印输入图：
```java
Prim prim = new Prim(createGraph());
System.out.println(prim.originalGraphToString());
```

我们的示例将输出：
```text
A --- 2 --- B
A --- 3 --- C
B --- 5 --- E
B --- 2 --- C
C --- 1 --- E
C --- 1 --- D
```

现在，我们运行Prim算法并使用minimumSpanningTreeToString()方法打印生成的MST：
```java
prim.run();
prim.resetPrintHistory();
System.out.println(prim.minimumSpanningTreeToString());
```

最后，打印出我们的MST：
```text
A --- 2 --- B
B --- 2 --- C
C --- 1 --- E
C --- 1 --- D
```

## 6. 总结

在本文中，我们学习了Prim算法如何找到图的最小生成树。