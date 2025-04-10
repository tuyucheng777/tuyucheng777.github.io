---
layout: post
title:  Java中的迷宫求解器
category: algorithms
copyright: algorithms
excerpt: 迷宫求解器
---

## 1. 简介

在本文中，我们将探讨使用Java穿越迷宫的可能方法。

假设迷宫是一张黑白图像，黑色像素代表墙壁，白色像素代表路径。两个白色像素比较特殊，一个代表迷宫的入口，另一个代表出口。

给定这样一个迷宫，我们想找到一条从入口到出口的路径。

## 2. 迷宫建模

我们将迷宫视为一个二维整数数组，数组中数值的含义遵循以下约定：

- 0 -> 道路
- 1 -> 墙
- 2 -> 迷宫入口
- 3 -> 迷宫出口
- 4 -> 从入口到出口的路径的单元格部分

**我们将迷宫建模为一个图**，入口和出口是两个特殊节点，需要在它们之间确定路径。

典型的图具有两个属性：节点和边；边决定图的连通性，并将一个节点连接到另一个节点。

**因此，我们假设每个节点有4条隐式边，将给定节点链接到其左、右、顶部和底部节点**。

让我们定义方法签名：
```java
public List<Coordinate> solve(Maze maze) {
}
```

该方法的输入是一个迷宫，包含二维数组，其命名约定如上所述。

该方法的响应是一个节点列表，形成从入口节点到出口节点的路径。

## 3. 递归回溯器(DFS)

### 3.1 算法

一种相当明显的方法是探索所有可能的路径，最终会找到一条存在的路径。但这种方法的复杂性将呈指数级增长，并且扩展性不佳。

**但是，可以通过回溯和标记已访问过的节点来定制上述蛮力解决方案，以在合理的时间内获得路径，这种算法也称为[深度优先搜索](https://www.baeldung.com/cs/graph-algorithms-bfs-dijkstra)**。

该算法可以概括为：

1. 如果我们在墙或者已经访问过的节点，则返回失败
2. 否则，如果我们是出口节点，则返回成功
3. 否则，将节点添加到路径列表中并沿所有四个方向递归行进。如果返回失败，则从路径中删除节点并返回失败。找到出口时，路径列表将包含唯一路径

我们将该算法应用到图1(a)所示的迷宫中，其中S是起点，E是出口。

对于每个节点，我们按顺序遍历每个方向：右、下、左、上。

在1(b)中，我们探索了一条路径，但撞到了墙。然后我们回溯直到找到一个没有墙邻居的节点，并探索另一条路径，如1(c)所示。

我们再次撞墙，重复上述过程，最终找到出口，如1(d)所示：

![](/assets/images/2025/algorithms/javasolvemaze01.png)

### 3.2 实现

现在我们看看Java实现：

首先，我们需要定义四个方向，我们可以用坐标来定义，这些坐标与任何给定坐标相加时，将返回相邻坐标之一：
```java
private static int[][] DIRECTIONS = { { 0, 1 }, { 1, 0 }, { 0, -1 }, { -1, 0 } };
```

我们还需要一个实用方法来相加两个坐标：
```java
private Coordinate getNextCoordinate(
  int row, int col, int i, int j) {return new Coordinate(row + i, col + j);
}
```

现在我们可以定义方法签名solve，这里的逻辑很简单-如果从入口到出口有一条路径，则返回该路径，否则返回一个空列表：
```java
public List<Coordinate> solve(Maze maze) {
   List<Coordinate> path = new ArrayList<>();
   if (
           explore(
                   maze,
                   maze.getEntry().getX(),
                   maze.getEntry().getY(),
                   path
           )
   ) {
      return path;
   }
   return Collections.emptyList();
}
```

让我们定义上面提到的explore方法，如果存在路径，则返回true，参数path中是坐标列表；此方法包含3个主要代码块。

首先，我们丢弃无效节点，即位于迷宫外部或属于墙壁节点。之后，我们将当前节点标记为已访问，这样我们就不会重复访问同一个节点。

最后，如果找不到出口，我们将递归地向所有方向移动：
```java
private boolean explore(
        Maze maze, int row, int col, List<Coordinate> path) {
   if (
           !maze.isValidLocation(row, col)
                   || maze.isWall(row, col)
                   || maze.isExplored(row, col)
   ) {
      return false;
   }

   path.add(new Coordinate(row, col));
   maze.setVisited(row, col, true);

   if (maze.isExit(row, col)) {
      return true;
   }

   for (int[] direction : DIRECTIONS) {
      Coordinate coordinate = getNextCoordinate(
              row, col, direction[0], direction[1]);
      if (
              explore(
                      maze,
                      coordinate.getX(),
                      coordinate.getY(),
                      path
              )
      ) {
         return true;
      }
   }

   path.remove(path.size() - 1);
   return false;
}
```

这个解决方案使用的栈大小与迷宫的大小相同。

## 4. 变体-最短路径(BFS)

### 4.1 算法

上面描述的递归算法可以找到路径，但它不一定是最短路径。为了找到最短路径，我们可以使用另一种称为[广度优先搜索](https://en.wikipedia.org/wiki/Breadth-first_search)的图遍历方法。

在深度优先搜索(DFS)中，我们会先探索一个子节点及其所有孙节点，然后再探索下一个子节点。**而在广度优先搜索(BFS)中，我们会先探索所有直属子节点，然后再探索孙子节点**，这样可以确保所有距离父节点一定距离的节点都会被同时探索。

该算法可以概括如下：

1. 将起始节点添加到队列
2. 当队列不为空时，弹出一个节点，执行以下操作：
   - 2.1 如果我们到达墙或节点已被访问，则跳到下一次迭代
   - 2.2 如果到达出口节点，则从当前节点回溯到起始节点以找到最短路径
   - 2.3 否则，将4个方向的所有直接邻居添加到队列中

**这里很重要的一点是节点必须跟踪其父节点，也就是它们被添加到队列的位置**，这对于在遇到出口节点后找到路径至关重要。

以下动画展示了使用此算法探索迷宫的所有步骤，我们可以观察到，在进入下一层之前，所有距离相同的节点都会被首先探索：

![](/assets/images/2025/algorithms/javasolvemaze02.png)

### 4.2 实现

现在让我们用Java实现这个算法，我们将重用上一节中定义的DIRECTIONS变量。

首先定义一个实用方法，用于从给定节点回溯到其根节点，一旦找到出口，该方法将用于跟踪路径：
```java
private List<Coordinate> backtrackPath(Coordinate cur) {
   List<Coordinate> path = new ArrayList<>();
   Coordinate iter = cur;

   while (iter != null) {
      path.add(iter);
      iter = iter.parent;
   }

   return path;
}
```

现在让我们定义核心方法solve，我们将重用DFS实现中使用的3个块，即验证节点、标记已访问节点和遍历相邻节点。

我们只需做一点小小的修改，我们将使用FIFO数据结构来跟踪邻居并对其进行迭代，而不是递归遍历：
```java
public List<Coordinate> solve(Maze maze) {
   LinkedList<Coordinate> nextToVisit = new LinkedList<>();
   Coordinate start = maze.getEntry();
   nextToVisit.add(start);

   while (!nextToVisit.isEmpty()) {
      Coordinate cur = nextToVisit.remove();

      if (!maze.isValidLocation(cur.getX(), cur.getY())
              || maze.isExplored(cur.getX(), cur.getY())
      ) {
         continue;
      }

      if (maze.isWall(cur.getX(), cur.getY())) {
         maze.setVisited(cur.getX(), cur.getY(), true);
         continue;
      }

      if (maze.isExit(cur.getX(), cur.getY())) {
         return backtrackPath(cur);
      }

      for (int[] direction : DIRECTIONS) {
         Coordinate coordinate
                 = new Coordinate(
                 cur.getX() + direction[0],
                 cur.getY() + direction[1],
                 cur
         );
         nextToVisit.add(coordinate);
         maze.setVisited(cur.getX(), cur.getY(), true);
      }
   }
   return Collections.emptyList();
}
```

## 5. 总结

在本教程中，我们描述了两种主要的图算法：深度优先搜索和广度优先搜索，用于解决迷宫问题，我们还介绍了BFS如何给出从入口到出口的最短路径。

如需进一步阅读，请查找解决迷宫的其他方法，例如A\*和Dijkstra算法。