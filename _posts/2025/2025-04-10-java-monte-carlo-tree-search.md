---
layout: post
title:  Java中的井字游戏蒙特卡洛树搜索
category: algorithms
copyright: algorithms
excerpt: 蒙特卡洛树
---

## 1. 概述

在本文中，我们将探讨蒙特卡洛树搜索(MCTS)算法及其应用。

我们将通过用Java实现井字游戏来详细了解其各个阶段，我们将设计一个通用解决方案，只需进行很少的更改即可将其用于许多其他实际应用中。

## 2. 简介

简单来说，蒙特卡洛树搜索是一种概率搜索算法。它是一种独特的决策算法，因为它在具有大量可能性的开放式环境中非常高效。

如果你已经熟悉诸如[Minimax](https://en.wikipedia.org/wiki/Minimax)之类的博弈论算法，它需要一个函数来评估当前状态，并且必须计算博弈树中的多个级别才能找到最佳举动。

不幸的是，在[围棋](https://en.wikipedia.org/wiki/Go_(game))这样的游戏中这样做是不可行的，因为围棋的分支因子很高(随着树的高度增加，会产生数百万种可能性)，而且很难编写一个好的评估函数来计算当前状态有多好。

**蒙特卡洛树搜索将[蒙特卡洛方法](https://en.wikipedia.org/wiki/Monte_Carlo_method)应用于博弈树搜索，由于它基于博弈状态的随机采样，因此无需强行推导出每种可能性。此外，它也不一定要求我们编写评估或良好的启发式函数**。

顺便提一下，它彻底改变了计算机围棋的世界。自2016年3月以来，随着谷歌的[AlphaGo](https://en.wikipedia.org/wiki/AlphaGo)(基于MCTS和神经网络构建)击败[李世石](https://en.wikipedia.org/wiki/Lee_Sedol)(围棋世界冠军)，它已成为一个热门的研究课题。

## 3. 蒙特卡洛树搜索算法

现在，让我们探索一下该算法的工作原理。首先，我们将构建一个具有根节点的前瞻树(游戏树)，然后通过随机展开不断扩展它。在此过程中，我们将维护每个节点的访问次数和获胜次数。

最后，我们将选择具有最有希望的统计数据的节点。

该算法由4个阶段组成；让我们详细探讨它们。

### 3.1 选择

在这个初始阶段，算法从根节点开始，选择一个子节点，使其选择胜率最高的节点，我们还希望确保每个节点都有公平的机会。

**这个想法是不断选择最优子节点，直到我们到达树的叶节点**；选择此类子节点的一个好方法是使用UCT(应用于树的置信上限)公式：

![](/assets/images/2025/algorithms/javamontecarlotreesearch01.png)

- w<sub>i</sub> = 第i步之后获胜的次数
- n<sub>i</sub> = 第i步之后的模拟次数
- c = 探索参数(理论上等于√2)
- t = 父节点的模拟总数

这一方案确保没有一个州会成为饥荒的受害者，并且它比其他州更频繁地发挥有希望的分支。

### 3.2 扩展

当它无法再应用UCT来寻找后继节点时，它会通过附加叶节点中的所有可能状态来扩展游戏树。

### 3.3 模拟

扩展后，算法会任意选择一个子节点，并从选定节点开始模拟随机游戏，直到达到游戏的结果状态。如果在游戏过程中随机或半随机地选择节点，则称为轻度游戏；你也可以通过编写高质量的启发式方法或评估函数来选择重度游戏。

### 3.4 反向传播

这也称为更新阶段，一旦算法到达游戏结束，它会评估状态以确定哪个玩家获胜。它会向上遍历到根节点，并增加所有访问过的节点的访问分数。如果该位置的玩家赢得了比赛，它还会更新每个节点的获胜分数。

**MCTS不断重复这四个阶段，直到达到固定的迭代次数或固定的时间量**。

在这种方法中，我们根据随机移动来估算每个节点的获胜分数。因此，迭代次数越多，估算结果就越可靠。算法估算结果在搜索开始时可能不太准确，但经过足够长的时间后会不断改进。同样，这完全取决于问题的类型。

## 4. 试运行

![](/assets/images/2025/algorithms/javamontecarlotreesearch02.png)

这里，节点包含总访问次数/获胜分数等统计数据。

## 5. 实现

现在，让我们使用蒙特卡洛树搜索算法来实现井字游戏。

我们将设计一个通用的MCTS解决方案，该方案也适用于许多其他棋盘游戏。

虽然为了使解释清晰，我们可能不得不跳过一些小细节(与MCTS没有特别关系)，但你始终可以在GitHub上找到完整的实现。

首先，我们需要对Tree和Node类进行基本的实现，以实现树搜索功能：
```java
public class Node {
    State state;
    Node parent;
    List<Node> childArray;
    // setters and getters
}

public class Tree {
    Node root;
}
```

由于每个节点都有特定的问题状态，因此我们也实现一个State类：
```java
public class State {
    Board board;
    int playerNo;
    int visitCount;
    double winScore;

    // copy constructor, getters, and setters

    public List<State> getAllPossibleStates() {
        // constructs a list of all possible states from current state
    }
    public void randomPlay() {
        /* get a list of all possible positions on the board and 
           play a random move */
    }
}
```

现在，让我们实现MonteCarloTreeSearch类，它将负责从给定的游戏位置找到下一个最佳动作：
```java
public class MonteCarloTreeSearch {
    static final int WIN_SCORE = 10;
    int level;
    int opponent;

    public Board findNextMove(Board board, int playerNo) {
        // define an end time which will act as a terminating condition

        opponent = 3 - playerNo;
        Tree tree = new Tree();
        Node rootNode = tree.getRoot();
        rootNode.getState().setBoard(board);
        rootNode.getState().setPlayerNo(opponent);

        while (System.currentTimeMillis() < end) {
            Node promisingNode = selectPromisingNode(rootNode);
            if (promisingNode.getState().getBoard().checkStatus()
                    == Board.IN_PROGRESS) {
                expandNode(promisingNode);
            }
            Node nodeToExplore = promisingNode;
            if (promisingNode.getChildArray().size() > 0) {
                nodeToExplore = promisingNode.getRandomChildNode();
            }
            int playoutResult = simulateRandomPlayout(nodeToExplore);
            backPropogation(nodeToExplore, playoutResult);
        }

        Node winnerNode = rootNode.getChildWithMaxScore();
        tree.setRoot(winnerNode);
        return winnerNode.getState().getBoard();
    }
}
```

在这里，我们不断迭代所有4个阶段，直到预定的时间，最后我们得到一棵具有可靠统计数据的树来做出明智的决策。

现在，让我们实现所有阶段的方法。

**我们将从选择阶段开始**，这也需要UCT实施：
```java
private Node selectPromisingNode(Node rootNode) {
    Node node = rootNode;
    while (node.getChildArray().size() != 0) {
        node = UCT.findBestNodeWithUCT(node);
    }
    return node;
}
```

```java
public class UCT {
    public static double uctValue(
            int totalVisit, double nodeWinScore, int nodeVisit) {
        if (nodeVisit == 0) {
            return Integer.MAX_VALUE;
        }
        return ((double) nodeWinScore / (double) nodeVisit)
                + 1.41 * Math.sqrt(Math.log(totalVisit) / (double) nodeVisit);
    }

    public static Node findBestNodeWithUCT(Node node) {
        int parentVisit = node.getState().getVisitCount();
        return Collections.max(
                node.getChildArray(),
                Comparator.comparing(c -> uctValue(parentVisit,
                        c.getState().getWinScore(), c.getState().getVisitCount())));
    }
}
```

**此阶段推荐一个叶节点，该节点应在扩展阶段进一步扩展**：
```java
private void expandNode(Node node) {
    List<State> possibleStates = node.getState().getAllPossibleStates();
    possibleStates.forEach(state -> {
        Node newNode = new Node(state);
        newNode.setParent(node);
        newNode.getState().setPlayerNo(node.getState().getOpponent());
        node.getChildArray().add(newNode);
    });
}
```

**接下来，我们编写代码来选择一个随机节点并模拟随机播放**。此外，我们还将编写一个更新函数，用于从叶子节点到根节点传递得分和访问次数：
```java
private void backPropogation(Node nodeToExplore, int playerNo) {
    Node tempNode = nodeToExplore;
    while (tempNode != null) {
        tempNode.getState().incrementVisit();
        if (tempNode.getState().getPlayerNo() == playerNo) {
            tempNode.getState().addScore(WIN_SCORE);
        }
        tempNode = tempNode.getParent();
    }
}

private int simulateRandomPlayout(Node node) {
    Node tempNode = new Node(node);
    State tempState = tempNode.getState();
    int boardStatus = tempState.getBoard().checkStatus();
    if (boardStatus == opponent) {
        tempNode.getParent().getState().setWinScore(Integer.MIN_VALUE);
        return boardStatus;
    }
    while (boardStatus == Board.IN_PROGRESS) {
        tempState.togglePlayer();
        tempState.randomPlay();
        boardStatus = tempState.getBoard().checkStatus();
    }
    return boardStatus;
}
```

现在我们已经完成了MCTS的实现，我们需要的只是一个井字游戏特定的Board类实现。注意，要使用我们的实现玩其他游戏；我们只需更改Board类即可。
```java
public class Board {
    int[][] boardValues;
    public static final int DEFAULT_BOARD_SIZE = 3;
    public static final int IN_PROGRESS = -1;
    public static final int DRAW = 0;
    public static final int P1 = 1;
    public static final int P2 = 2;

    // getters and setters
    public void performMove(int player, Position p) {
        this.totalMoves++;
        boardValues[p.getX()][p.getY()] = player;
    }

    public int checkStatus() {
        /* Evaluate whether the game is won and return winner.
           If it is draw return 0 else return -1 */
    }

    public List<Position> getEmptyPositions() {
        int size = this.boardValues.length;
        List<Position> emptyPositions = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                if (boardValues[i][j] == 0)
                    emptyPositions.add(new Position(i, j));
            }
        }
        return emptyPositions;
    }
}
```

我们刚刚实现了一个在井字游戏中无法被击败的AI，让我们写一个单元测试来证明AI与AI之间的对决总是平局：
```java
@Test
void givenEmptyBoard_whenSimulateInterAIPlay_thenGameDraw() {
    Board board = new Board();
    int player = Board.P1;
    int totalMoves = Board.DEFAULT_BOARD_SIZE * Board.DEFAULT_BOARD_SIZE;
    for (int i = 0; i < totalMoves; i++) {
        board = mcts.findNextMove(board, player);
        if (board.checkStatus() != -1) {
            break;
        }
        player = 3 - player;
    }
    int winStatus = board.checkStatus();

    assertEquals(winStatus, Board.DRAW);
}
```

## 6. 优点

- 它不一定需要任何关于游戏的战术知识
- 通用的MCTS实现只需稍加修改即可重复用于任意数量的游戏
- 关注获胜几率较高的节点
- 适用于高分支因子的问题，因为它不会在所有可能的分支上浪费计算
- 算法实现起来非常简单
- 执行可以在任何给定时间停止，并且它仍然会建议迄今为止计算出的下一个最佳状态

## 7. 缺点

如果直接使用MCTS的基本形式而不进行任何改进，它可能无法提出合理的移动建议，这种情况可能发生在节点访问不足导致估计不准确的情况下。

**然而，MCTS可以通过一些技术得到改进，它涉及领域特定技术以及领域独立技术**。

在特定领域的技术中，模拟阶段比随机模拟能产生更真实的结果，尽管它需要了解游戏特定的技术和规则。

## 8. 总结

乍一看，很难相信一个依赖随机选择的算法能够产生智能AI。然而，MCTS的巧妙实现确实可以为我们提供一种解决方案，它不仅可用于许多游戏，也可用于决策问题。
