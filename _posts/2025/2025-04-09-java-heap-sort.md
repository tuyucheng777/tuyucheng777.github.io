---
layout: post
title:  Java中的堆排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在本教程中，我们将介绍[堆排序](https://www.baeldung.com/cs/understanding-heapsort)的工作原理，并用Java实现它。

**堆排序基于堆数据结构**，为了正确理解堆排序，我们首先深入研究堆及其实现方式。

## 2. 堆数据结构

堆是一种特殊的**基于树的数据结构**，因此它由节点组成。我们将元素分配给节点：每个节点只包含一个元素。

此外，节点可以有子节点，**如果一个节点没有任何子节点，我们称其为叶子节点**。

堆的特别之处在于两点：

1. 每个节点的值必须**小于或等于其子节点中存储的所有值**
2. 它是一棵完整的树，这意味着它有最小的高度

根据第一条规则，**最小元素总是位于树的根部**。

我们如何执行这些规则取决于具体实现。

堆通常用于实现优先级队列，因为堆是提取最小(或最大)元素的一种非常有效的实现。

### 2.1 堆变体

堆有很多变体，它们在一些实现细节上有所不同。

例如，我们**上面描述的是最小堆，因为父节点总是小于其所有子节点**。或者，我们可以定义最大堆，在这种情况下，父节点总是大于其子节点。因此，最大元素将位于根节点中。

我们可以从许多树实现中进行选择，最直接的是二叉树。**在二叉树中，每个节点最多可以有两个子节点**，我们将它们称为左子节点和右子节点。

执行第二条规则的最简单方法是使用完整二叉树，完整二叉树遵循一些简单的规则：

1. 如果一个节点只有一个子节点，那么这个子节点应该是它的左子节点
2. 只有最深层的最右边的节点才能有一个子节点
3. 叶子只能在最深的层

让我们通过一些例子来看一下这些规则：
```text
  1        2      3        4        5        6         7         8        9       10
 ()       ()     ()       ()       ()       ()        ()        ()       ()       ()
         /         \     /  \     /  \     /  \      /  \      /        /        /  \
        ()         ()   ()  ()   ()  ()   ()  ()    ()  ()    ()       ()       ()  ()
                                /          \       /  \      /  \     /        /  \
                               ()          ()     ()  ()    ()  ()   ()       ()  ()
                                                                             /
                                                                            ()
```

树1、2、4、5和7遵循规则。

树3和6违反了第1条规则，树8和9违反了第2条规则，树10违反了第3条规则。

在本教程中，我们将重点介绍使用二叉树实现的最小堆。

### 2.2 插入元素

我们应该以保持堆不变的方式实现所有操作，这样，我们就可以构建具有重复插入操作的堆，因此我们将专注于单个插入操作。

我们可以按照以下步骤插入元素：

1. 创建一个新叶子节点，该节点是最深层最右边的可用槽，并将元素存储在该节点中
2. 如果元素小于其父元素，则交换它们
3. 继续执行步骤2，直到元素大于其父元素或成为新的根元素

请注意，步骤2不会违反堆规则，因为如果我们用较小的节点替换节点的值，它仍然会小于它的子节点。

让我们看一个例子，我们想将4插入到这个堆中：
```text
        2
       / \
      /   \
     3     6
    / \
   5   7
```

第一步是创建一个存储4的新叶子节点：
```text
        2
       / \
      /   \
     3     6
    / \   /
   5   7 4
```

由于4小于其父级6，因此我们交换它们：
```text
        2
       / \
      /   \
     3     4
    / \   /
   5   7 6
```

现在我们检查4是否小于它的父元素，由于它的父元素是2，所以我们停止。堆仍然有效，我们插入了数字4。

让我们插入1：
```text
        2
       / \
      /   \
     3     4
    / \   / \
   5   7 6   1
```

我们必须交换1和4：
```text
        2
       / \
      /   \
     3     1
    / \   / \
   5   7 6   4
```

现在我们应该交换1和2：
```text
        1
       / \
      /   \
     3     2
    / \   / \
   5   7 6   4
```

由于1是新的根，所以我们停止。

## 3. Java中的堆实现

**由于我们使用了完整二叉树，因此可以用数组来实现**：数组中的元素将成为树中的节点，我们按照以下方式从左到右、从上到下用数组索引标记每个节点：
```text
        0
       / \
      /   \
     1     2
    / \   /
   3   4 5
```

我们唯一需要做的就是跟踪树中存储了多少个元素，这样，我们要插入的下一个元素的索引将是数组的大小。

利用这个索引，我们可以计算出父节点和子节点的索引：

- 父节点：(索引 - 1) / 2
- 左子节点：2 * 索引 + 1
- 右子节点：2 * 索引 + 2

因为我们不想为数组重新分配而烦恼，所以我们将进一步简化实现并使用ArrayList。

基本的二叉树实现如下所示：
```java
class BinaryTree<E> {

    List<E> elements = new ArrayList<>();

    void add(E e) {
        elements.add(e);
    }

    boolean isEmpty() {
        return elements.isEmpty();
    }

    E elementAt(int index) {
        return elements.get(index);
    }

    int parentIndex(int index) {
        return (index - 1) / 2;
    }

    int leftChildIndex(int index) {
        return 2 * index + 1;
    }

    int rightChildIndex(int index) {
        return 2 * index + 2;
    }
}
```

上面的代码只是将新元素添加到树的末尾，因此，如果需要，我们需要向上遍历新元素。我们可以使用以下代码来实现：
```java
class Heap<E extends Comparable<E>> {

    // ...

    void add(E e) {
        elements.add(e);
        int elementIndex = elements.size() - 1;
        while (!isRoot(elementIndex) && !isCorrectChild(elementIndex)) {
            int parentIndex = parentIndex(elementIndex);
            swap(elementIndex, parentIndex);
            elementIndex = parentIndex;
        }
    }

    boolean isRoot(int index) {
        return index == 0;
    }

    boolean isCorrectChild(int index) {
        return isCorrect(parentIndex(index), index);
    }

    boolean isCorrect(int parentIndex, int childIndex) {
        if (!isValidIndex(parentIndex) || !isValidIndex(childIndex)) {
            return true;
        }

        return elementAt(parentIndex).compareTo(elementAt(childIndex)) < 0;
    }

    boolean isValidIndex(int index) {
        return index < elements.size();
    }

    void swap(int index1, int index2) {
        E element1 = elementAt(index1);
        E element2 = elementAt(index2);
        elements.set(index1, element2);
        elements.set(index2, element1);
    }

    // ...
}
```

请注意，由于我们需要比较元素，因此它们需要实现java.util.Comparable。

## 4. 堆排序

由于堆的根始终包含最小的元素，因此**堆排序背后的想法非常简单：删除根节点，直到堆变空**。

我们唯一需要的是一个删除操作，它使堆保持一致状态；我们必须确保不破坏二叉树的结构或堆的性质。

**为了保持结构，我们不能删除除最右边叶子节点之外的任何元素**。因此，我们的想法是从根节点中删除元素，并将最右边的叶子存储在根节点中。

但此操作肯定会违反堆的性质，因此，**如果新的根节点大于其任何子节点，我们将它与其最小子节点交换**。由于最小子节点小于所有其他子节点，因此它并不违反堆的性质。

我们不断交换，直到元素变成叶子元素，或者小于其所有子元素。

让我们从这棵树中删除根：
```text
        1
       / \
      /   \
     3     2
    / \   / \
   5   7 6   4
```

首先，我们将最后一个叶子放在根部：
```text
        4
       / \
      /   \
     3     2
    / \   /
   5   7 6
```

然后，由于它比它的两个子元素都大，我们将它与它的最小子元素(即2)交换：
```text
        2
       / \
      /   \
     3     4
    / \   /
   5   7 6
```

4小于6，因此我们停止。

## 5. Java中的堆排序实现

有了这些，删除根(弹出)看起来就像这样：
```java
class Heap<E extends Comparable<E>> {

    // ...

    E pop() {
        if (isEmpty()) {
            throw new IllegalStateException("You cannot pop from an empty heap");
        }

        E result = elementAt(0);

        int lasElementIndex = elements.size() - 1;
        swap(0, lasElementIndex);
        elements.remove(lasElementIndex);

        int elementIndex = 0;
        while (!isLeaf(elementIndex) && !isCorrectParent(elementIndex)) {
            int smallerChildIndex = smallerChildIndex(elementIndex);
            swap(elementIndex, smallerChildIndex);
            elementIndex = smallerChildIndex;
        }

        return result;
    }

    boolean isLeaf(int index) {
        return !isValidIndex(leftChildIndex(index));
    }

    boolean isCorrectParent(int index) {
        return isCorrect(index, leftChildIndex(index)) && isCorrect(index, rightChildIndex(index));
    }

    int smallerChildIndex(int index) {
        int leftChildIndex = leftChildIndex(index);
        int rightChildIndex = rightChildIndex(index);

        if (!isValidIndex(rightChildIndex)) {
            return leftChildIndex;
        }

        if (elementAt(leftChildIndex).compareTo(elementAt(rightChildIndex)) < 0) {
            return leftChildIndex;
        }

        return rightChildIndex;
    }

    // ...
}
```

正如我们之前所说，排序只是创建一个堆，然后反复删除根：
```java
class Heap<E extends Comparable<E>> {

    // ...

    static <E extends Comparable<E>> List<E> sort(Iterable<E> elements) {
        Heap<E> heap = of(elements);

        List<E> result = new ArrayList<>();

        while (!heap.isEmpty()) {
            result.add(heap.pop());
        }

        return result;
    }

    static <E extends Comparable<E>> Heap<E> of(Iterable<E> elements) {
        Heap<E> result = new Heap<>();
        for (E element : elements) {
            result.add(element);
        }
        return result;
    }

    // ...
}
```

我们可以通过以下测试来验证它是否正常工作：
```java
@Test
void givenNotEmptyIterable_whenSortCalled_thenItShouldReturnElementsInSortedList() {
    // given
    List<Integer> elements = Arrays.asList(3, 5, 1, 4, 2);
    
    // when
    List<Integer> sortedElements = Heap.sort(elements);
    
    // then
    assertThat(sortedElements).isEqualTo(Arrays.asList(1, 2, 3, 4, 5));
}
```

请注意，**我们可以提供一个就地排序的实现**，这意味着我们将结果放在获取元素的同一个数组中。此外，这样我们就不需要任何中间内存分配。但是，这种实现理解起来会稍微困难一些。

## 6. 时间复杂度

堆排序包含两个关键步骤：**插入**元素和**移除**根节点，这两个步骤的复杂度均为O(log n)。

**由于我们重复这两个步骤n次，因此整体排序复杂度为O(n log n)**。

请注意，我们没有提到数组重新分配的成本，但由于它是O(n)，因此它不会影响整体复杂度。此外，正如我们之前提到的，可以实现就地排序，这意味着不需要重新分配数组。

还值得一提的是，50%的元素是叶子节点，75%的元素位于最底下的两层。因此，大多数插入操作不会超过两步。

**请注意，在实际数据中，快速排序通常比堆排序性能更好。值得庆幸的是，堆排序最坏情况下的时间复杂度始终为O(n log n)**。

## 7. 总结

在本教程中，我们看到了二叉堆和堆排序的实现。

**虽然它的时间复杂度为O(n log n)，但在大多数情况下，它并不是现实世界数据上的最佳算法**。