---
layout: post
title:  克鲁斯卡尔生成树算法及其Java实现
category: algorithms
copyright: algorithms
excerpt: 克鲁斯卡尔生成树
---

## 1. 概述

在上一篇文章中，我们介绍了[Prim算法](https://www.baeldung.com/java-prim-algorithm)来寻找最小生成树。在本文中，我们将使用另一种方法-Kruskal算法，来解决最小和最大生成树问题。

## 2. 生成树

**无向图的生成树是一个连通子图，它覆盖图中的所有节点，并具有尽可能少的边数。通常，一个图可能包含多棵生成树**。下图显示了一个包含生成树的图(生成树的边为红色)：

![](/assets/images/2025/algorithms/javaspanningtreeskruskal01.png)

如果图是边加权的，我们可以将生成树的权重定义为其所有边的权重之和，**最小生成树是所有可能生成树中权重最小的生成树**，下图显示了边加权图上的最小生成树：

![](/assets/images/2025/algorithms/javaspanningtreeskruskal02.png)

类似地，**最大生成树在所有生成树中具有最大的权重**，下图显示了边加权图上的最大生成树：

![](/assets/images/2025/algorithms/javaspanningtreeskruskal03.png)

## 3. Kruskal算法

给定一个图，我们可以使用Kruskal算法来找到其最小生成树。如果图中的节点数为V，则它的每棵生成树都应该有(V-1)条边且不包含循环，我们可以用以下伪代码描述Kruskal算法：
```text
Initialize an empty edge set T. 
Sort all graph edges by the ascending order of their weight values. 
foreach edge in the sorted edge list
    Check whether it will create a cycle with the edges inside T.
    If the edge doesn't introduce any cycles, add it into T. 
    If T has (V-1) edges, exit the loop. 
return T
```

让我们在示例图上逐步运行Kruskal最小生成树算法：

![](/assets/images/2025/algorithms/javaspanningtreeskruskal04.png)

首先，我们选择边(0,2)，因为它的权重最小。然后，我们可以添加边(3,4)和(0,1)，因为它们不会产生任何循环。现在下一个候选是权重为9的边(1,2)，但是，如果我们添加这条边，就会产生一个循环(0,1,2)。因此，我们丢弃这条边并继续选择下一个最小边。最后，算法以添加权重为10的边(2,4)结束。

**为了计算最大生成树，我们可以将排序顺序改为降序，其他步骤保持不变**。下图展示了在示例图上逐步构建最大生成树的过程。

![](/assets/images/2025/algorithms/javaspanningtreeskruskal05.png)

## 4. 使用不相交集进行循环检测

在Kruskal算法中，关键部分是检查一条边在添加到现有边集后是否会产生循环，我们可以使用多种图循环检测算法。例如，我们可以使用[深度优先搜索(DFS)算法](https://www.baeldung.com/java-graph-has-a-cycle)遍历图并检测是否存在循环。

但是，每次测试新边时，我们都需要对现有边进行循环检测。**一个更快的解决方案是使用并查集算法和不相交数据结构，因为它也使用增量式边添加方法来检测循环**，我们可以将其融入到生成树的构建过程中。

### 4.1 不相交集和生成树的构造

首先，我们将图中的每个节点视为仅包含一个节点的独立集合，然后，每次引入一条边时，我们都会检查该边的两个节点是否在同一集合中。如果是，则表示存在循环。否则，我们将两个不相交的集合合并为一个集合，并将该边添加到生成树中。

我们可以重复上述步骤，直到构建整个生成树。

例如，在上面的最小生成树构造中，我们首先有5个节点集：{0}、{1}、{2}、{3}、{4}。当我们检查第一条边(0,2)时，它的两个节点位于不同的节点集中。因此，我们可以包含这条边并将{0}和{2}合并为一个集合{0,2}。

我们可以对边(3,4)和(0,1)进行类似的操作，节点集随后变为{0,1,2}和{3,4}。检查下一条边(1,2)时，我们发现这条边的两个节点都在同一个集合中。因此，我们丢弃这条边并继续检查下一条边。最后，边(2,4)满足我们的条件，我们可以将其添加到最小生成树中。

### 4.2 不相交集实现

我们可以用树结构来表示一个不相交的集合，每个节点都有一个父指针来引用其父节点。在每个集合中，都有一个唯一的根节点代表这个集合，根节点有一个自引用的父指针。

让我们使用Java类来定义不相交集信息：
```java
public class DisjointSetInfo {
    private Integer parentNode;
    DisjointSetInfo(Integer parent) {
        setParentNode(parent);
    }

    //standard setters and getters
}
```

让我们用一个整数标记每个图节点，从0开始，我们可以使用列表数据结构List<DisjointSetInfo\> nodes来存储图的不相交集信息。一开始，每个节点都是其自身集合的代表成员：
```java
void initDisjointSets(int totalNodes) {
    nodes = new ArrayList<>(totalNodes);
    for (int i = 0; i < totalNodes; i++) {
        nodes.add(new DisjointSetInfo(i));
    }
}
```

### 4.3 查找操作

为了找到一个节点所属的集合，我们可以沿着该节点的父链向上查找，直到到达根节点：
```java
Integer find(Integer node) {
    Integer parent = nodes.get(node).getParentNode();
    if (parent.equals(node)) {
        return node;
    } else {
        return find(parent);
    }
}
```

对于不相交的集合，树结构可能高度不平衡，**我们可以利用路径压缩技术来改进查找操作**。

由于我们在前往根节点的路径上访问的每个节点都属于同一个集合，因此我们可以将根节点直接附加到其父节点的引用上，下次访问此节点时，我们需要一条查找路径来获取根节点：
```java
Integer pathCompressionFind(Integer node) {
    DisjointSetInfo setInfo = nodes.get(node);
    Integer parent = setInfo.getParentNode();
    if (parent.equals(node)) {
        return node;
    } else {
        Integer parentNode = find(parent);
        setInfo.setParentNode(parentNode);
        return parentNode;
    }
}
```

### 4.4 并集运算

如果一条边的两个节点位于不同的集合中，我们会将这两个集合合并为一个，我们可以通过将一个代表节点的根设置为另一个代表节点来实现此合并操作：
```java
void union(Integer rootU, Integer rootV) {
    DisjointSetInfo setInfoU = nodes.get(rootU);
    setInfoU.setParentNode(rootV);
}
```

由于我们为合并集选择了随机根节点，这个简单的并集运算可能会产生高度不平衡的树，**我们可以使用按秩并集技术来提高性能**。

由于树的深度会影响查找操作的运行时间，我们将较短的树集合附加到较长的树集合，仅当原始两棵树的深度相同时，此技术才会增加合并树的深度。

为了实现这一点，我们首先向DisjointSetInfo类添加一个rank属性：
```java
public class DisjointSetInfo {
    private Integer parentNode;
    private int rank;
    DisjointSetInfo(Integer parent) {
        setParentNode(parent);
        setRank(0);
    }

    //standard setters and getters
}
```

最初，单个不相交节点的秩为0，在两个集合合并时，秩较高的根节点将成为合并后集合的根节点。仅当原始两个根节点的秩相同时，我们才将新根节点的秩加1：
```java
void unionByRank(int rootU, int rootV) {
    DisjointSetInfo setInfoU = nodes.get(rootU);
    DisjointSetInfo setInfoV = nodes.get(rootV);
    int rankU = setInfoU.getRank();
    int rankV = setInfoV.getRank();
    if (rankU < rankV) {
        setInfoU.setParentNode(rootV);
    } else {
        setInfoV.setParentNode(rootU);
        if (rankU == rankV) {
            setInfoU.setRank(rankU + 1);
        }
    }
}
```

### 4.5 循环检测

我们可以通过比较两次查找操作的结果来确定两个节点是否位于同一个不相交集合中，如果它们具有相同的代表性根节点，则我们检测到一个循环。否则，我们使用并集操作合并这两个不相交集合：
```java
boolean detectCycle(Integer u, Integer v) {
    Integer rootU = pathCompressionFind(u);
    Integer rootV = pathCompressionFind(v);
    if (rootU.equals(rootV)) {
        return true;
    }
    unionByRank(rootU, rootV);
    return false;
}
```

仅使用按秩并集技术，循环检测的[运行时间为O(logV)](https://en.wikipedia.org/wiki/Disjoint-set_data_structure)，我们可以结合路径压缩和按秩并集技术来获得更好的性能，运行时间为[O(α(V))](https://en.wikipedia.org/wiki/Disjoint-set_data_structure)，其中α(V)是节点总数的[逆阿克曼函数](https://en.wikipedia.org/wiki/Ackermann_function#Inverse)。在我们的实际计算中，它是一个小于5的小常数。

## 5. Kruskal算法的Java实现

我们可以使用[Google Guava](https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20a%3A%22guava%22)中的[ValueGraph](https://guava.dev/releases/snapshot-jre/api/docs/com/google/common/graph/ValueGraph.html)数据结构来表示边加权图。

要使用ValueGraph，我们首先需要将Guava依赖添加到项目的pom.xml文件中：
```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.2.1-jre</version>
</dependency>
```

我们可以将上述循环检测方法包装成CycleDetector类，并在Kruskal算法中使用它。由于最小生成树和最大生成树的构建算法只有细微的差别，我们可以使用一个通用函数来实现这两种构建方法：
```java
ValueGraph<Integer, Double> spanningTree(ValueGraph<Integer, Double> graph, boolean minSpanningTree) {
    Set<EndpointPair> edges = graph.edges();
    List<EndpointPair> edgeList = new ArrayList<>(edges);

    if (minSpanningTree) {
        edgeList.sort(Comparator.comparing(e -> graph.edgeValue(e).get()));
    } else {
        edgeList.sort(Collections.reverseOrder(Comparator.comparing(e -> graph.edgeValue(e).get())));
    }

    int totalNodes = graph.nodes().size();
    CycleDetector cycleDetector = new CycleDetector(totalNodes);
    int edgeCount = 0;

    MutableValueGraph<Integer, Double> spanningTree = ValueGraphBuilder.undirected().build();
    for (EndpointPair edge : edgeList) {
        if (cycleDetector.detectCycle(edge.nodeU(), edge.nodeV())) {
            continue;
        }
        spanningTree.putEdgeValue(edge.nodeU(), edge.nodeV(), graph.edgeValue(edge).get());
        edgeCount++;
        if (edgeCount == totalNodes - 1) {
            break;
        }
    }
    return spanningTree;
}
```

在Kruskal算法中，我们首先按权重对所有图边进行排序，此操作需要O(ElogE)时间，其中E是边的总数。

然后我们使用循环遍历已排序的边列表，在每次迭代中，我们检查将边添加到当前生成树边集是否会形成循环，此循环包含循环检测，最多耗时O(ElogV)。

因此整体运行时间为O(ELogE + ELogV)，由于E的值在O(V<sup>2</sup>)的规模内，因此Kruskal算法的时间复杂度为O(ElogE)或O(ElogV)。

## 6. 总结

在本文中，我们学习了如何使用Kruskal算法来查找图的最小或最大生成树。