---
layout: post
title:  Java中使用后缀树进行字符串的快速模式匹配
category: algorithms
copyright: algorithms
excerpt: 后缀树
---

## 1. 概述

在本教程中，我们将探讨字符串模式匹配的概念以及如何使其更快。然后，我们将介绍其在Java中的实现。

## 2. 字符串模式匹配

### 2.1 定义

在字符串中，模式匹配是检查给定字符序列(称为“文本”)中是否存在给定字符序列(称为“模式”)的过程。

当模式不是正则表达式时，模式匹配的基本期望是：

- 匹配应该完全匹配，而不是部分匹配
- 结果应该包含所有匹配项-而不仅仅是第一个匹配项
- 结果应包含文本中每个匹配项的位置

### 2.2 搜索模式

让我们通过一个例子来理解一个简单的模式匹配问题：

```text
Pattern:   NA
Text:      HAVANABANANA
Match1:    ----NA------
Match2:    --------NA--
Match3:    ----------NA
```

我们可以看到模式NA在文本中出现了三次，为了得到这个结果，我们可以考虑将模式一次一个字符地向下滑动文本并检查是否匹配。

然而，这是一种蛮力方法，时间复杂度为O(p*t)，其中p是模式的长度，t是文本的长度。

假设我们要搜索的模式不止一个，那么时间复杂度也会线性增加，因为每个模式都需要单独的迭代。

### 2.3 用于存储模式的Trie数据结构

我们可以通过将模式存储在[trie数据结构](https://www.baeldung.com/trie-java)中来缩短搜索时间，该结构以快速检索元素而闻名。

我们知道，trie数据结构以树状结构存储字符串的字符。因此，对于两个字符串{NA, NAB}，我们将得到一棵具有两条路径的树：

![](/assets/images/2025/algorithms/javapatternmatchingsuffixtree01.png)

创建trie后，可以将一组模式沿文本向下滑动，并在一次迭代中检查匹配情况。

请注意，我们使用$字符来指示字符串的结束。

### 2.4 后缀Trie数据结构存储文本

另一方面，后缀trie是**使用单个字符串的所有可能后缀构建的trie数据结构**。

对于前面的例子HAVANABANANA，我们可以构建一个后缀trie：

![](/assets/images/2025/algorithms/javapatternmatchingsuffixtree02.png)

后缀trie是为文本创建的，通常作为预处理步骤的一部分。之后，可以通过查找与模式序列匹配的路径来快速搜索模式。

然而，众所周知，后缀trie会占用大量空间，因为字符串的每个字符都存储在边中。

在下一节中，我们将讨论后缀trie的改进版本。

## 3. 后缀树

后缀树其实就是一个压缩的后缀树，这意味着，通过连接边，我们可以存储一组字符，从而大大减少存储空间。

因此，我们可以为同一文本HAVANABANANA创建后缀树：

![](/assets/images/2025/algorithms/javapatternmatchingsuffixtree03.png)

从根到叶的每条路径代表字符串HAVANABANANA的后缀。

后缀树还会存储后缀在叶节点中的位置，例如，BANANA\$是从第7个位置开始的后缀。因此，使用从0开始的编号，其值将为6。同样，A->BANANA\$是另一个从第5个位置开始的后缀，如上图所示。

因此，从整体上看，我们可以看到，**当我们能够获得一条从根节点开始的路径，并且其边在位置上与给定模式完全匹配时，就会发生模式匹配**。

如果路径以叶节点结束，则进行后缀匹配；否则，仅进行子字符串匹配。例如，模式NA是HAVANABANA[NA\]的后缀，也是HAVA[NA\]BANANA的子字符串。

在下一节中，我们将看到如何在Java中实现这种数据结构。

## 4. 数据结构

让我们创建一个后缀树数据结构，我们需要两个域类。

首先，我们需要一个类来表示树节点，它需要存储树的边及其子节点。此外，当它是叶节点时，它需要存储后缀的位置值。

那么，让我们创建Node类：

```java
public class Node {
    private String text;
    private List<Node> children;
    private int position;

    public Node(String word, int position) {
        this.text = word;
        this.position = position;
        this.children = new ArrayList<>();
    }

    // getters, setters, toString()
}
```

其次，我们需要一个类来表示树并存储根节点，它还需要存储生成后缀的全文。

因此，我们有一个SuffixTree类：

```java
public class SuffixTree {
    private static final String WORD_TERMINATION = "$";
    private static final int POSITION_UNDEFINED = -1;
    private Node root;
    private String fullText;

    public SuffixTree(String text) {
        root = new Node("", POSITION_UNDEFINED);
        fullText = text;
    }
}
```

## 5. 添加数据的辅助方法

在编写存储数据的核心逻辑之前，我们先添加一些辅助方法，这些方法稍后会派上用场。

让我们修改SuffixTree类来添加构建树所需的一些方法。

### 5.1 添加子节点

首先，有一个方法addChildNode来向任何给定的父节点添加一个新的子节点：

```java
private void addChildNode(Node parentNode, String text, int index) {
    parentNode.getChildren().add(new Node(text, index));
}
```

### 5.2 寻找两个字符串的最长公共前缀

其次，我们将编写一个简单的实用方法getLongestCommonPrefix来查找两个字符串的最长公共前缀：

```java
private String getLongestCommonPrefix(String str1, String str2) {
    int compareLength = Math.min(str1.length(), str2.length());
    for (int i = 0; i < compareLength; i++) {
        if (str1.charAt(i) != str2.charAt(i)) {
            return str1.substring(0, i);
        }
    }
    return str1.substring(0, compareLength);
}
```

### 5.3 分裂节点

第三，让我们创建一个方法，从给定的父节点中分离出一个子节点。在这个过程中，父节点的text值将被截断，右截断的字符串将成为子节点的text值。此外，父节点的子节点将被转移到子节点。

从下图可以看出，ANA被拆分为A->NA。之后，新的后缀ABANANA\$可以添加为A->BANANA\$：

![](/assets/images/2025/algorithms/javapatternmatchingsuffixtree04.png)

简而言之，这是一个插入新节点时非常有用的便捷方法：

```java
private void splitNodeToParentAndChild(Node parentNode, String parentNewText, String childNewText) {
    Node childNode = new Node(childNewText, parentNode.getPosition());

    if (parentNode.getChildren().size() > 0) {
        while (parentNode.getChildren().size() > 0) {
            childNode.getChildren()
                    .add(parentNode.getChildren().remove(0));
        }
    }

    parentNode.getChildren().add(childNode);
    parentNode.setText(parentNewText);
    parentNode.setPosition(POSITION_UNDEFINED);
}
```

## 6. 遍历辅助方法

现在让我们创建遍历树的逻辑，我们将使用此方法来构建树并搜索模式。

### 6.1 部分匹配与完全匹配

首先，让我们通过考虑一棵包含几个后缀的树来理解部分匹配和完全匹配的概念：

![](/assets/images/2025/algorithms/javapatternmatchingsuffixtree05.png)

要添加新后缀ANABANANA\$，我们需要检查是否存在可以修改或扩展以适应新值的节点。为此，我们将新文本与所有节点进行比较，发现现有节点[A\]VANABANANA$在第一个字符处匹配。因此，这是我们需要修改的节点，这种匹配可以称为部分匹配。

另一方面，假设我们在同一棵树上搜索模式VANE，我们知道它在前3个字符上与[VAN\]ABANANA$部分匹配，如果所有4个字符都匹配，我们可以称之为完全匹配。对于模式搜索，完全匹配是必要的。

总而言之，我们在构建树时使用部分匹配，在搜索模式时使用完全匹配；我们将使用标志isAllowPartialMatch来指示我们在每种情况下需要的匹配类型。

### 6.2 遍历树

现在，让我们编写逻辑来遍历树，只要我们能够在位置上匹配给定的模式：

```java
List<Node> getAllNodesInTraversePath(String pattern, Node startNode, boolean isAllowPartialMatch) {
    // ...
}
```

**我们将递归调用它并返回我们在路径中找到的所有节点的列表**。

我们首先将模式文本的第一个字符与节点文本进行比较：

```java
if (pattern.charAt(0) == nodeText.charAt(0)) {
    // logic to handle remaining characters       
}
```

对于部分匹配，如果模式短于或等于节点文本的长度，我们将当前节点添加到nodes列表中并在此处停止：

```java
if (isAllowPartialMatch && pattern.length() <= nodeText.length()) {
    nodes.add(currentNode);
    return nodes;
}
```

然后我们将此节点文本的其余字符与模式的其余字符进行比较，如果模式与节点文本的位置不匹配，则停止。当前节点仅在部分匹配时才包含在nodes列表中：

```java
int compareLength = Math.min(nodeText.length(), pattern.length());
for (int j = 1; j < compareLength; j++) {
    if (pattern.charAt(j) != nodeText.charAt(j)) {
        if (isAllowPartialMatch) {
            nodes.add(currentNode);
        }
        return nodes;
    }
}
```

如果模式与节点文本匹配，我们将当前节点添加到我们的nodes列表中：
```java
nodes.add(currentNode);
```

但是，如果模式的字符数多于节点文本，则我们需要检查子节点。为此，我们进行递归调用，将currentNode作为起始节点，将pattern的剩余部分作为新模式传递。如果节点列表不为空，则此调用返回的节点列表将附加到nodes列表中。如果在完全匹配的情况下，列表为空，则表示存在不匹配的情况，因此为了指示这一点，我们添加一个null项，然后返回nodes：

```java
if (pattern.length() > compareLength) {
    List nodes2 = getAllNodesInTraversePath(pattern.substring(compareLength), currentNode, 
      isAllowPartialMatch);
    if (nodes2.size() > 0) {
        nodes.addAll(nodes2);
    } else if (!isAllowPartialMatch) {
        nodes.add(null);
    }
}
return nodes;
```

把所有这些放在一起，让我们创建getAllNodesInTraversePath：

```java
private List<Node> getAllNodesInTraversePath(String pattern, Node startNode, boolean isAllowPartialMatch) {
    List<Node> nodes = new ArrayList<>();
    for (int i = 0; i < startNode.getChildren().size(); i++) {
        Node currentNode = startNode.getChildren().get(i);
        String nodeText = currentNode.getText();
        if (pattern.charAt(0) == nodeText.charAt(0)) {
            if (isAllowPartialMatch && pattern.length() <= nodeText.length()) {
                nodes.add(currentNode);
                return nodes;
            }

            int compareLength = Math.min(nodeText.length(), pattern.length());
            for (int j = 1; j < compareLength; j++) {
                if (pattern.charAt(j) != nodeText.charAt(j)) {
                    if (isAllowPartialMatch) {
                        nodes.add(currentNode);
                    }
                    return nodes;
                }
            }

            nodes.add(currentNode);
            if (pattern.length() > compareLength) {
                List<Node> nodes2 = getAllNodesInTraversePath(pattern.substring(compareLength),
                        currentNode, isAllowPartialMatch);
                if (nodes2.size() > 0) {
                    nodes.addAll(nodes2);
                } else if (!isAllowPartialMatch) {
                    nodes.add(null);
                }
            }
            return nodes;
        }
    }
    return nodes;
}
```

## 7. 算法

### 7.1 存储数据

现在我们可以编写逻辑来存储数据了，首先在SuffixTree类上定义一个新方法addSuffix：

```java
private void addSuffix(String suffix, int position) {
    // ...
}
```

调用者将提供后缀的位置。

接下来，我们来编写处理后缀的逻辑。首先，我们需要通过调用辅助方法getAllNodesInTraversePath并将isAllowPartialMatch设置为true来**检查是否存在至少部分匹配后缀的路径**。如果不存在路径，我们可以将后缀作为子节点添加到根节点：

```java
List<Node> nodes = getAllNodesInTraversePath(pattern, root, true);
if (nodes.size() == 0) {
    addChildNode(root, suffix, position);
}
```

但是，**如果路径存在，则意味着我们需要修改现有节点**，此节点将是nodes列表中的最后一个节点。我们还需要确定此现有节点的新文本应该是什么，如果nodes列表只有一个元素，则我们使用suffix。否则，我们从suffix中排除到最后一个节点的公共前缀以获取newText：

```java
Node lastNode = nodes.remove(nodes.size() - 1);
String newText = suffix;
if (nodes.size() > 0) {
    String existingSuffixUptoLastNode = nodes.stream()
        .map(a -> a.getText())
        .reduce("", String::concat);
    newText = newText.substring(existingSuffixUptoLastNode.length());
}
```

为了修改现有节点，我们创建一个新方法extendNode，并从addSuffix方法的上次中断处调用它。此方法有两个主要职责：一个是将现有节点拆分为父节点和子节点，另一个是将子节点添加到新创建的父节点，我们拆分父节点只是为了使其成为所有子节点的公共节点。因此，我们的新方法已准备就绪：

```java
private void extendNode(Node node, String newText, int position) {
    String currentText = node.getText();
    String commonPrefix = getLongestCommonPrefix(currentText, newText);

    if (commonPrefix != currentText) {
        String parentText = currentText.substring(0, commonPrefix.length());
        String childText = currentText.substring(commonPrefix.length());
        splitNodeToParentAndChild(node, parentText, childText);
    }

    String remainingText = newText.substring(commonPrefix.length());
    addChildNode(node, remainingText, position);
}
```

现在我们可以回到添加后缀的方法，该方法现在已经具备了所有逻辑：

```java
private void addSuffix(String suffix, int position) {
    List<Node> nodes = getAllNodesInTraversePath(suffix, root, true);
    if (nodes.size() == 0) {
        addChildNode(root, suffix, position);
    } else {
        Node lastNode = nodes.remove(nodes.size() - 1);
        String newText = suffix;
        if (nodes.size() > 0) {
            String existingSuffixUptoLastNode = nodes.stream()
                    .map(a -> a.getText())
                    .reduce("", String::concat);
            newText = newText.substring(existingSuffixUptoLastNode.length());
        }
        extendNode(lastNode, newText, position);
    }
}
```

最后，让我们修改SuffixTree构造函数来生成后缀，并调用我们之前的方法addSuffix将它们迭代地添加到我们的数据结构中：

```java
public void SuffixTree(String text) {
    root = new Node("", POSITION_UNDEFINED);
    for (int i = 0; i < text.length(); i++) {
        addSuffix(text.substring(i) + WORD_TERMINATION, i);
    }
    fullText = text;
}
```

### 7.2 搜索数据

定义了后缀树结构来存储数据后，我们现在可以编写执行搜索的逻辑。

我们首先在SuffixTree类上添加一个新方法searchText，将要搜索的pattern作为输入：

```java
public List<String> searchText(String pattern) {
    // ...
}
```

接下来，为了检查该pattern是否存在于我们的后缀树中，我们调用我们的辅助方法getAllNodesInTraversePath，并设置仅用于精确匹配的标志，这与在添加数据期间允许部分匹配不同：

```java
List<Node> nodes = getAllNodesInTraversePath(pattern, root, false);
```

然后，我们获取与模式匹配的节点列表，列表中的最后一个节点表示模式精确匹配的节点。因此，我们的下一步将是获取源自最后一个匹配节点的所有叶节点，并获取存储在这些叶节点中的位置。

让我们创建一个单独的方法getPositions来执行此操作，我们将检查给定节点是否存储后缀的最后部分，以决定是否需要返回其位置值。并且，我们将对给定节点的每个子节点递归执行此操作：

```java
private List<Integer> getPositions(Node node) {
    List<Integer> positions = new ArrayList<>();
    if (node.getText().endsWith(WORD_TERMINATION)) {
        positions.add(node.getPosition());
    }
    for (int i = 0; i < node.getChildren().size(); i++) {
        positions.addAll(getPositions(node.getChildren().get(i)));
    }
    return positions;
}
```

一旦我们有了位置集，下一步就是用它来标记我们存储在后缀树中的文本上的模式。位置值表示后缀的起始位置，模式的长度表示从起始点偏移多少个字符。应用这个逻辑，让我们创建一个简单的实用方法：

```java
private String markPatternInText(Integer startPosition, String pattern) {
    String matchingTextLHS = fullText.substring(0, startPosition);
    String matchingText = fullText.substring(startPosition, startPosition + pattern.length());
    String matchingTextRHS = fullText.substring(startPosition + pattern.length());
    return matchingTextLHS + "[" + matchingText + "]" + matchingTextRHS;
}
```

现在，我们已经准备好了支持方法。因此，我们可以将它们添加到搜索方法中并完成逻辑：

```java
public List<String> searchText(String pattern) {
    List<String> result = new ArrayList<>();
    List<Node> nodes = getAllNodesInTraversePath(pattern, root, false);

    if (nodes.size() > 0) {
        Node lastNode = nodes.get(nodes.size() - 1);
        if (lastNode != null) {
            List<Integer> positions = getPositions(lastNode);
            positions = positions.stream()
                    .sorted()
                    .collect(Collectors.toList());
            positions.forEach(m -> result.add((markPatternInText(m, pattern))));
        }
    }
    return result;
}
```

## 8. 测试

现在我们已经有了算法，让我们测试一下。

首先，我们将文本存储在SuffixTree中：

```java
SuffixTree suffixTree = new SuffixTree("havanabanana");
```

接下来，让我们搜索有效的模式a：

```java
List<String> matches = suffixTree.searchText("a");
matches.stream().forEach(m -> LOGGER.debug(m));
```

运行代码后，我们得到了预期的6次匹配结果：

```text
h[a]vanabanana
hav[a]nabanana
havan[a]banana
havanab[a]nana
havanaban[a]na
havanabanan[a]
```

接下来，让我们搜索另一个有效模式nab：

```java
List<String> matches = suffixTree.searchText("nab");
matches.stream().forEach(m -> LOGGER.debug(m));
```

运行代码后，正如预期的那样，只有1个匹配结果：

```text
hava[nab]anana
```

最后，让我们搜索一个无效的模式nag：

```java
List<String> matches = suffixTree.searchText("nag");
matches.stream().forEach(m -> LOGGER.debug(m));
```

运行代码没有结果，因为匹配必须是完全匹配，而不是部分匹配。

因此，我们的模式搜索算法已经能够满足我们在本教程开始时提出的所有期望。

## 9. 时间复杂度

对于给定长度为t的文本，构建后缀树时，**时间复杂度为O(t)**。

然后，对于搜索长度为p的模式，**时间复杂度为O(p)**。回想一下，暴力搜索的时间复杂度是O(p*t)。因此，**在对文本进行预处理后，模式搜索会变得更快**。

## 10. 总结

在本文中，我们首先了解了三种数据结构的概念：trie、后缀trie和后缀树。然后，我们了解了如何使用后缀树来紧凑地存储后缀。

之后，我们了解了如何使用后缀树来存储数据并执行模式搜索。