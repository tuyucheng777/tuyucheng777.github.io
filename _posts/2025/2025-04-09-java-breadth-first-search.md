---
layout: post
title:  Java中的广度优先搜索算法
category: algorithms
copyright: algorithms
excerpt: 搜索
---

## 1. 概述

在本教程中，我们将学习[广度优先搜索算法](https://www.baeldung.com/cs/graph-algorithms-bfs-dijkstra)，该算法允许我们通过广度优先而不是深度优先遍历节点来搜索树或图中的节点。

首先，我们将介绍一些有关树和图算法的理论。然后，我们将深入研究该算法在Java中的实现。最后，我们将介绍它们的时间复杂度。

## 2. 广度优先搜索算法

广度优先搜索(BFS)算法的基本方法是通过先探索邻居再探索子节点来在树或图结构中搜索节点。

首先，我们将了解该算法如何适用于树。然后，我们将它应用于图，图具有特定的约束，即有时包含循环。最后，我们将讨论该算法的性能。

### 2.1. 树

[树的BFS算法](http://opendatastructures.org/versions/edition-0.1e/ods-java/6_1_BinaryTree_Basic_Binary.html#sec:bintree:traversal)背后的思想是**维护一个节点队列，以确保遍历的顺序**。在算法开始时，队列仅包含根节点。只要队列仍包含一个或多个节点，我们就重复以下步骤：

- 从队列中弹出第一个节点
- 如果该节点是我们要搜索的节点，则搜索结束
- 否则，将此节点的子节点添加到队列末尾并重复步骤

**执行终止是通过消除循环来保证的**，我们将在下一节中了解如何管理循环。

### 2.2 图

对于[图](http://opendatastructures.org/versions/edition-0.1e/ods-java/12_3_Graph_Traversal.html#SECTION001531000000000000000)，我们必须考虑结构中可能存在的循环，如果我们简单地将之前的算法应用于有循环的图，它将永远循环下去。因此，**我们需要保存已访问节点的集合，并确保不会重复访问它们**：

- 从队列中弹出第一个节点
- 检查节点是否已被访问，如果是则跳过
- 如果该节点是我们要搜索的节点，则搜索结束
- 否则，将其添加到访问过的节点
- 将此节点的子节点添加到队列并重复这些步骤

## 3. Java实现

现在已经介绍了理论，让我们开始编写代码并用Java实现这些算法。

### 3.1 树

首先，我们将实现树算法；让我们设计Tree类，它由一个值和由其他Tree列表表示的子节点组成：
```java
public class Tree<T> {
    private T value;
    private List<Tree<T>> children;

    private Tree(T value) {
        this.value = value;
        this.children = new ArrayList<>();
    }

    public static <T> Tree<T> of(T value) {
        return new Tree<>(value);
    }

    public Tree<T> addChild(T value) {
        Tree<T> newChild = new Tree<>(value);
        children.add(newChild);
        return newChild;
    }
}
```

为了避免产生循环，子类由类本身根据给定值创建。

之后，让我们提供一个search()方法：
```java
public static <T> Optional<Tree<T>> search(T value, Tree<T> root) {
    // ...
}
```

前面提到过，**BFS算法使用队列来遍历节点**。首先，我们将根节点添加到此队列中：
```java
Queue<Tree<T>> queue = new ArrayDeque<>();
queue.add(root);
```

然后，我们必须在队列不为空时循环，并且每次从队列中弹出一个节点：
```java
while(!queue.isEmpty()) {
    Tree<T> currentNode = queue.remove();
}
```

**如果该节点是我们正在搜索的节点，则返回它，否则将其子节点添加到队列中**：
```java
if (currentNode.getValue().equals(value)) {
    return Optional.of(currentNode);
} else {
    queue.addAll(currentNode.getChildren());
}
```

最后，如果我们访问了所有节点却没有找到我们要搜索的节点，我们将返回一个空结果：
```java
return Optional.empty();
```

现在让我们想象一个树结构的示例：

![](/assets/images/2025/algorithms/javabreadthfirstsearch01.png)

转化为Java代码如下：
```java
Tree<Integer> root = Tree.of(10);
Tree<Integer> rootFirstChild = root.addChild(2);
Tree<Integer> depthMostChild = rootFirstChild.addChild(3);
Tree<Integer> rootSecondChild = root.addChild(4);
```

然后，如果搜索值4，我们期望算法按以下顺序遍历值为10、2和4的节点：
```java
BreadthFirstSearchAlgorithm.search(4, root)
```

我们可以通过记录访问过的节点的值来验证这一点：
```text
[main] DEBUG  c.t.t.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 10
[main] DEBUG  c.t.t.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 2 
[main] DEBUG  c.t.t.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 4
```

### 3.2 图

现在让我们看看如何处理图，与树相反，**图可以包含循环**。这意味着，正如我们在上一节中看到的，我们必须记住访问过的节点以避免无限循环。稍后我们会看到如何更新算法来考虑这个问题，但首先，让我们定义一下图的结构：
```java
public class Node<T> {
    private T value;
    private Set<Node<T>> neighbors;

    public Node(T value) {
        this.value = value;
        this.neighbors = new HashSet<>();
    }

    public void connect(Node<T> node) {
        if (this == node) throw new IllegalArgumentException("Can't connect node to itself");
        this.neighbors.add(node);
        node.neighbors.add(this);
    }
}
```

现在，我们可以看到，与树相反，我们可以自由地将一个节点与另一个节点连接起来，从而创建循环，唯一的例外是节点不能连接到自身。

还值得注意的是，这种表示法没有根节点。但这不成问题，因为我们将节点之间的连接也设置为双向的，这意味着我们可以从任意节点开始搜索整个图。

首先，让我们重用上面的算法，并使其适应新的结构：
```java
public static <T> Optional<Node<T>> search(T value, Node<T> start) {
    Queue<Node<T>> queue = new ArrayDeque<>();
    queue.add(start);

    Node<T> currentNode;

    while (!queue.isEmpty()) {
        currentNode = queue.remove();

        if (currentNode.getValue().equals(value)) {
            return Optional.of(currentNode);
        } else {
            queue.addAll(currentNode.getNeighbors());
        }
    }

    return Optional.empty();
}
```

我们不能像这样运行算法，否则任何循环都会让它永远运行下去。因此，我们必须添加指令来处理已经访问过的节点：
```java
while (!queue.isEmpty()) {
    currentNode = queue.remove();
    LOGGER.debug("Visited node with value: {}", currentNode.getValue());

    if (currentNode.getValue().equals(value)) {
        return Optional.of(currentNode);
    } else {
        alreadyVisited.add(currentNode);
        queue.addAll(currentNode.getNeighbors());
        queue.removeAll(alreadyVisited);
    }
}

return Optional.empty();
```

正如我们所见，我们首先初始化一个包含已访问节点的集合。
```java
Set<Node<T>> alreadyVisited = new HashSet<>();
```

然后，**当值比较失败时，我们将该节点添加到已访问的节点中**：
```java
alreadyVisited.add(currentNode);
```

最后，**将节点的邻居添加到队列后，我们从中删除已经访问过的节点**(这是检查当前节点是否存在于该集合中的另一种方法)：
```java
queue.removeAll(alreadyVisited);
```

**通过这样做，我们确保算法不会陷入无限循环**。

让我们通过一个例子看看它是如何工作的，首先，我们定义一个带有循环的图：

![](/assets/images/2025/algorithms/javabreadthfirstsearch02.png)

Java代码中也是一样：
```java
Node<Integer> start = new Node<>(10);
Node<Integer> firstNeighbor = new Node<>(2);
start.connect(firstNeighbor);

Node<Integer> firstNeighborNeighbor = new Node<>(3);
firstNeighbor.connect(firstNeighborNeighbor);
firstNeighborNeighbor.connect(start);

Node<Integer> secondNeighbor = new Node<>(4);
start.connect(secondNeighbor);
```

再次假设我们要搜索值4，由于没有根节点，因此可以从任何我们想要的节点开始搜索，我们将选择firstNeighborNeighbor：
```java
BreadthFirstSearchAlgorithm.search(4, firstNeighborNeighbor);
```

再次，我们将添加一些日志来查看哪些节点被访问了，我们期望它们是3、2、10和4，并且按这个顺序每个节点只访问一次：
```text
[main] DEBUG c.t.t.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 3 
[main] DEBUG c.t.t.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 2 
[main] DEBUG c.t.t.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 10 
[main] DEBUG c.t.t.a.b.BreadthFirstSearchAlgorithm - Visited node with value: 4
```

### 3.3 复杂度

既然我们已经介绍了Java中的这两种算法，那么接下来我们来讨论一下它们的时间复杂度。我们将使用[大O表示法](https://www.baeldung.com/big-o-notation)来表达它们。

让我们从树算法开始，它最多将节点添加到队列一次，因此也最多访问一次。**因此，如果n是树中的节点数，则算法的时间复杂度将为O(n)**。

现在，对于图算法来说，事情有点复杂。我们最多会遍历每个节点一次，但要做到这一点，我们会使用具有线性复杂度的操作，例如addAll()和removeAll()。

假设节点数为n，连接数为c，那么，在最坏的情况下(即未找到节点)，我们可以使用addAll()和removeAll()方法添加和删除节点，直到达到连接数，这些操作的复杂度为O(c)。**因此，假设c > n，整个算法的复杂度将为O(c)；否则，它将是O(n)**，这通常[记为O(n + c)](https://www.khanacademy.org/computing/computer-science/algorithms/breadth-first-search/a/analysis-of-breadth-first-search)，可以理解为复杂度取决于n和c中的最大值。

为什么树搜索没有这个问题？因为树中的连接数受节点数的限制，n个节点的树中的连接数为n – 1。

## 4. 总结

在本文中，我们了解了广度优先搜索算法以及如何在Java中实现它。

在介绍了一些理论之后，我们编写了该算法的Java实现并讨论了其复杂性。