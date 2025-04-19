---
layout: post
title:  如何打印二叉树图
category: algorithms
copyright: algorithms
excerpt: 二叉树
---

## 1. 简介

打印是一种非常常见的数据结构可视化技术，但是，由于树状结构本身的层级结构，打印起来可能会比较棘手。

在本教程中，我们将学习Java中[二叉树](https://www.baeldung.com/java-binary-tree)的一些打印技术。

## 2. 树形图

尽管在控制台上仅使用字符进行绘制存在局限性，但仍有许多不同的图表形状可以表示树形结构，选择其中一种主要取决于树的大小和平衡性。

让我们看一下可以打印的一些图表类型：

![](/assets/images/2025/algorithms/javaprintbinarytreediagram01.png)

但是，我们将解释一个更实际、更容易实现的方法。通过考虑树的生长方向，我们可以将其称为水平树：

![](/assets/images/2025/algorithms/javaprintbinarytreediagram02.png)

由于水平树的流动方向始终与文本流动方向相同，因此选择水平图相对于其他图有一些好处：

1. 我们也可以想象大树和不平衡的树
2. 节点值的长度不影响显示结构
3. 实现起来更容易

因此，在接下来的部分中，我们将采用水平图并实现一个简单的二叉树打印器类。

## 3. 二叉树模型

首先，我们应该建立一个基本的二叉树模型，只需几行代码即可完成。

让我们定义一个简单的BinaryTreeModel类：

```java
public class BinaryTreeModel {

    private Object value;
    private BinaryTreeModel left;
    private BinaryTreeModel right;

    public BinaryTreeModel(Object value) {
        this.value = value;
    }

    // standard getters and setters
}
```

## 4. 示例测试数据

在开始实现二叉树打印器之前，我们需要创建一些示例数据来逐步测试我们的可视化效果：

```java
BinaryTreeModel root = new BinaryTreeModel("root");

BinaryTreeModel node1 = new BinaryTreeModel("node1");
BinaryTreeModel node2 = new BinaryTreeModel("node2");
root.setLeft(node1);
root.setRight(node2);
 
BinaryTreeModel node3 = new BinaryTreeModel("node3");
BinaryTreeModel node4 = new BinaryTreeModel("node4");
node1.setLeft(node3);
node1.setRight(node4);
 
node2.setLeft(new BinaryTreeModel("node5"));
node2.setRight(new BinaryTreeModel("node6"));
 
BinaryTreeModel node7 = new BinaryTreeModel("node7");
node3.setLeft(node7);
node7.setLeft(new BinaryTreeModel("node8"));
node7.setRight(new BinaryTreeModel("node9"));
```

## 5. 二叉树打印器

当然，为了遵循[单一职责原则](https://www.baeldung.com/solid-principles#s)，我们需要一个单独的类来保持我们的BinaryTreeModel干净。

现在，我们可以使用[访问者模式](https://www.baeldung.com/java-visitor-pattern)，这样树形结构就能处理层级结构，而打印器只负责打印。但在本教程中，为了简单起见，我们将它们放在一起。

因此，我们定义一个名为BinaryTreePrinter的类并开始实现它。

### 5.1 先序遍历

考虑到我们的水平图，为了正确打印它，我们可以使用前序遍历来做一个简单的开始。

因此，**为了执行前序遍历，我们需要实现一种递归方法，首先访问根节点，然后访问左子树，最后访问右子树**。

让我们定义一个遍历树的方法：

```java
public void traversePreOrder(StringBuilder sb, BinaryTreeModel node) {
    if (node != null) {
        sb.append(node.getValue());
        sb.append("\n");
        traversePreOrder(sb, node.getLeft());
        traversePreOrder(sb, node.getRight());
    }
}
```

接下来，让我们定义打印方法：

```java
public void print(PrintStream os) {
    StringBuilder sb = new StringBuilder();
    traversePreOrder(sb, this.tree);
    os.print(sb.toString());
}
```

因此，我们可以简单地打印我们的测试树：

```java
new BinaryTreePrinter(root).print(System.out);
```

输出将是按遍历顺序排列的树节点列表：

```text
root
node1
node3
node7
node8
node9
node4
node2
node5
node6
```

### 5.2 添加树边

为了正确设置我们的图，我们使用三种类型的字符“ ═──”、“└──”和“│”来可视化节点。前两个字符用于指针，最后一个字符用于填充边并连接指针。

让我们更新traversePreOrder方法，添加两个参数作为padding和pointer，并分别使用字符：

```java
public void traversePreOrder(StringBuilder sb, String padding, String pointer, BinaryTreeModel node) {
    if (node != null) {
        sb.append(padding);
        sb.append(pointer);
        sb.append(node.getValue());
        sb.append("\n");

        StringBuilder paddingBuilder = new StringBuilder(padding);
        paddingBuilder.append("│  ");

        String paddingForBoth = paddingBuilder.toString();
        String pointerForRight = "└──";
        String pointerForLeft = (node.getRight() != null) ? "├──" : "└──";

        traversePreOrder(sb, paddingForBoth, pointerForLeft, node.getLeft());
        traversePreOrder(sb, paddingForBoth, pointerForRight, node.getRight());
    }
}
```

此外，我们还更新了打印方法：

```java
public void print(PrintStream os) {
    StringBuilder sb = new StringBuilder();
    traversePreOrder(sb, "", "", this.tree);
    os.print(sb.toString());
}
```

那么，让我们再次测试我们的BinaryTreePrinter：

![](/assets/images/2025/algorithms/javaprintbinarytreediagram03.png)

因此，通过所有的填充和指针，我们的图表已经形成得很好了。

然而，我们仍然需要删除一些多余的行：

![](/assets/images/2025/algorithms/javaprintbinarytreediagram04.png)

当我们查看图表时，我们发现字符仍然出现在3个错误的位置：

1. 根节点下的额外行列
2. 右子树下的额外行
3. 左子树下没有右兄弟的额外行

### 5.3 根节点和子节点的不同实现

为了解决多余的行问题，我们可以拆分遍历方法。我们将对根节点应用一种行为，对子节点应用另一种行为。

让我们仅为根节点定制traversePreOrder：

```java
public String traversePreOrder(BinaryTreeModel root) {
    if (root == null) {
        return "";
    }

    StringBuilder sb = new StringBuilder();
    sb.append(root.getValue());

    String pointerRight = "└──";
    String pointerLeft = (root.getRight() != null) ? "├──" : "└──";

    traverseNodes(sb, "", pointerLeft, root.getLeft(), root.getRight() != null);
    traverseNodes(sb, "", pointerRight, root.getRight(), false);

    return sb.toString();
}
```

接下来，我们将为子节点创建另一个方法，即traverseNodes。此外，我们将添加一个新参数hasRightSibling来正确实现上述代码：

```java
public void traverseNodes(StringBuilder sb, String padding, String pointer, BinaryTreeModel node, boolean hasRightSibling) {
    if (node != null) {
        sb.append("\n");
        sb.append(padding);
        sb.append(pointer);
        sb.append(node.getValue());

        StringBuilder paddingBuilder = new StringBuilder(padding);
        if (hasRightSibling) {
            paddingBuilder.append("│  ");
        } else {
            paddingBuilder.append("   ");
        }

        String paddingForBoth = paddingBuilder.toString();
        String pointerRight = "└──";
        String pointerLeft = (node.getRight() != null) ? "├──" : "└──";

        traverseNodes(sb, paddingForBoth, pointerLeft, node.getLeft(), node.getRight() != null);
        traverseNodes(sb, paddingForBoth, pointerRight, node.getRight(), false);
    }
}
```

此外，我们需要对打印方法进行一些小的改变：

```java
public void print(PrintStream os) {
    os.print(traversePreOrder(tree));
}
```

最后，我们的图表形成了一个漂亮的形状，并有一个清晰的输出：

![](/assets/images/2025/algorithms/javaprintbinarytreediagram05.png)

## 6. 总结

在本文中，我们学习了一种在Java中打印二叉树的简单实用的方法。