---
layout: post
title:  Java中的范围搜索算法
category: algorithms
copyright: algorithms
excerpt: 搜索
---

## 1. 概述

在本教程中，我们将探索在二维空间中搜索邻居的概念。然后，我们将介绍其在Java中的实现。

## 2. 一维搜索与二维搜索

我们知道[二分查找](https://www.baeldung.com/java-binary-search)是一种使用分治方法在元素列表中找到完全匹配的有效算法。

现在让我们**考虑一个二维区域，其中每个元素都由平面上的XY坐标(点)表示**。

但是，假设我们想要找到平面中给定点的邻居，而不是精确匹配。显然，如果我们想要最近的n个匹配，那么二分查找将不起作用。这是因为二分查找只能在一个轴上比较两个元素，而我们需要能够在两个轴上比较它们。

在下一节中，我们将讨论二叉树数据结构的替代方案。

## 3. 四叉树

四叉树是一种空间树数据结构，其中每个节点恰好有四个子节点，每个子节点可以是一个点，也可以是一个包含四个子四叉树的列表。

点存储数据— 例如XY坐标，区域表示可以存储点的封闭边界，它用于定义四叉树的可达范围。

让我们使用任意顺序的10个坐标的示例来进一步理解这一点：
```text
(21,25), (55,53), (70,318), (98,302), (49,229), (135,229), (224,292), (206,321), (197,258), (245,238)
```

前三个值将作为根节点下的点存储，如最左边的图片所示。

![](/assets/images/2025/algorithms/javarangesearch01.png)

根节点现在无法容纳新的点，因为它已经达到了3个点的容量，因此，**我们将根节点的区域划分为4个相等的象限**。

每个象限都可以存储3个点，并且在其边界内还包含4个象限。这可以递归地完成，从而形成一个象限树，四叉树数据结构因此而得名。

在上图中间，我们可以看到从根节点创建的象限以及接下来的四个点如何存储在这些象限中。

最后，最右边的图片显示了一个象限如何再次细分以容纳该区域中的更多点，而其他象限仍然可以接受新的点。

现在我们将看到如何用Java实现该算法。

## 4. 数据结构

让我们创建一个四叉树数据结构，我们需要三个域类。

首先，我们创建一个Point类来存储XY坐标：
```java
public class Point {
    private float x;
    private float y;

    public Point(float x, float y) {
        this.x = x;
        this.y = y;
    }

    // getters & toString()
}
```

其次，让我们创建一个Region类来定义象限的边界：
```java
public class Region {
    private float x1;
    private float y1;
    private float x2;
    private float y2;

    public Region(float x1, float y1, float x2, float y2) {
        this.x1 = x1;
        this.y1 = y1;
        this.x2 = x2;
        this.y2 = y2;
    }

    // getters & toString()
}
```

最后，我们有一个**QuadTree类，将数据存储为Point实例，将子项存储为QuadTree类**：
```java
public class QuadTree {
    private static final int MAX_POINTS = 3;
    private Region area;
    private List<Point> points = new ArrayList<>();
    private List<QuadTree> quadTrees = new ArrayList<>();

    public QuadTree(Region area) {
        this.area = area;
    }
}
```

为了实例化QuadTree对象，我们通过构造函数使用Region类指定其区域。

## 5. 算法

在编写存储数据的核心逻辑之前，让我们添加一些辅助方法，这些方法稍后会派上用场。

### 5.1 辅助方法

让我们修改Region类。

首先，有一个方法**containsPoint来指示给定点是否位于区域范围之内或之外**：
```java
public boolean containsPoint(Point point) {
    return point.getX() >= this.x1
            && point.getX() < this.x2
            && point.getY() >= this.y1
            && point.getY() < this.y2;
}
```

接下来，让我们用方法**doesOverlap来指示给定区域是否与另一个区域重叠**：
```java
public boolean doesOverlap(Region testRegion) {
    if (testRegion.getX2() < this.getX1()) {
        return false;
    }
    if (testRegion.getX1() > this.getX2()) {
        return false;
    }
    if (testRegion.getY1() > this.getY2()) {
        return false;
    }
    if (testRegion.getY2() < this.getY1()) {
        return false;
    }
    return true;
}
```

最后，让我们创建一个方法getQuadrant，将一个范围分成四个相等的象限并返回一个指定的象限：
```java
public Region getQuadrant(int quadrantIndex) {
    float quadrantWidth = (this.x2 - this.x1) / 2;
    float quadrantHeight = (this.y2 - this.y1) / 2;

    // 0=SW, 1=NW, 2=NE, 3=SE
    switch (quadrantIndex) {
        case 0:
            return new Region(x1, y1, x1 + quadrantWidth, y1 + quadrantHeight);
        case 1:
            return new Region(x1, y1 + quadrantHeight, x1 + quadrantWidth, y2);
        case 2:
            return new Region(x1 + quadrantWidth, y1 + quadrantHeight, x2, y2);
        case 3:
            return new Region(x1 + quadrantWidth, y1, x2, y1 + quadrantHeight);
    }
    return null;
}
```

### 5.2 存储数据

现在我们可以编写逻辑来存储数据了，首先在QuadTree类上定义一个新方法addPoint来添加一个新点，如果成功添加点，此方法将返回true：
```java
public boolean addPoint(Point point) {
    // ...
}
```

接下来，让我们编写处理该点的逻辑。首先，我们需要检查该点是否包含在QuadTree实例的边界内，我们还需要确保QuadTree实例的点数未达到MAX_POINTS个点的容量。

如果两个条件都满足，我们就可以添加新的点：
```java
if (this.area.containsPoint(point)) {
    if (this.points.size() < MAX_POINTS) {
        this.points.add(point);
        return true;
    }
}
```

另一方面，**如果已经达到MAX_POINTS值，则需要将新点添加到其中一个子象限**。为此，我们循环遍历子quadTrees列表并调用相同的addPoint方法，该方法在成功添加后返回true值。然后我们立即退出循环，因为**一个点只需要被添加到一个象限即可**。

我们可以将所有这些逻辑封装在一个辅助方法中：
```java
private boolean addPointToOneQuadrant(Point point) {
    boolean isPointAdded;
    for (int i = 0; i < 4; i++) {
        isPointAdded = this.quadTrees.get(i)
                .addPoint(point);
        if (isPointAdded)
            return true;
    }
    return false;
}
```

此外，我们还有一个方便的方法createQuadrants将当前四叉树细分为4个象限：
```java
private void createQuadrants() {
    Region region;
    for (int i = 0; i < 4; i++) {
        region = this.area.getQuadrant(i);
        quadTrees.add(new QuadTree(region));
    }
}
```

仅当我们无法再添加任何新点时，我们才会调用此方法来创建象限，这可确保我们的数据结构使用最佳内存空间。

综合起来，我们得到了更新的addPoint方法：
```java
public boolean addPoint(Point point) {
    if (this.area.containsPoint(point)) {
        if (this.points.size() < MAX_POINTS) {
            this.points.add(point);
            return true;
        } else {
            if (this.quadTrees.size() == 0) {
                createQuadrants();
            }
            return addPointToOneQuadrant(point);
        }
    }
    return false;
}
```

### 5.3 搜索数据

定义了四叉树结构来存储数据后，我们现在可以考虑执行搜索的逻辑了。

当我们寻找相邻项时，我们可以**指定一个searchRegion作为起点**。然后，我们检查它是否与根区域重叠，如果重叠，我们将所有位于searchRegion内的子点添加到该区域。

到达根区域后，我们进入每个象限并重复该过程，如此循环，直到到达树的末端。

我们将上述逻辑写成QuadTree类中的递归方法：
```java
public List<Point> search(Region searchRegion, List<Point> matches) {
    if (matches == null) {
        matches = new ArrayList<Point>();
    }
    if (!this.area.doesOverlap(searchRegion)) {
        return matches;
    } else {
        for (Point point : points) {
            if (searchRegion.containsPoint(point)) {
                matches.add(point);
            }
        }
        if (this.quadTrees.size() > 0) {
            for (int i = 0; i < 4; i++) {
                quadTrees.get(i)
                        .search(searchRegion, matches);
            }
        }
    }
    return matches;
}
```

## 6. 测试

现在我们已经有了算法，让我们来测试它。

### 6.1 填充数据

首先，让我们用之前使用的10个坐标填充四叉树：
```java
Region area = new Region(0, 0, 400, 400);
QuadTree quadTree = new QuadTree(area);

float[][] points = new float[][] { { 21, 25 }, { 55, 53 }, { 70, 318 }, { 98, 302 }, 
    { 49, 229 }, { 135, 229 }, { 224, 292 }, { 206, 321 }, { 197, 258 }, { 245, 238 } };

for (int i = 0; i < points.length; i++) {
    Point point = new Point(points[i][0], points[i][1]);
        quadTree.addPoint(point);
}
```

### 6.2 范围搜索

接下来我们在下界坐标(200,200)与上限坐标(250,250)所包围的区域内进行范围搜索：
```java
Region searchArea = new Region(200, 200, 250, 250);
List<Point> result = quadTree.search(searchArea, null);
```

运行代码将为我们提供一个包含在搜索区域内的附近坐标：
```text
[[245.0 , 238.0]]
```

让我们尝试在坐标(0,0)和(100,100)之间进行不同的搜索：
```java
Region searchArea = new Region(0, 0, 100, 100);
List<Point> result = quadTree.search(searchArea, null);
```

运行代码将为我们提供指定搜索区域的两个附近的坐标：
```text
[[21.0 , 25.0], [55.0 , 53.0]]
```

我们观察到，根据搜索区域的大小，我们会得到0个、1个或多个点。因此，**如果给定一个点，并要求我们找到最近的n个邻居，我们可以定义一个合适的搜索区域，并以给定点为中心**。

然后，从搜索操作得到的所有点中，我们可以**计算给定点之间的欧几里得距离，并对它们进行排序以获得最近的邻居**。

## 7. 时间复杂度

范围查询的时间复杂度仅为O(n)，原因是，在最坏的情况下，如果指定的搜索区域等于或大于填充区域，则必须遍历每个元素。

## 8. 总结

在本文中，我们首先通过将四叉树与二叉树进行比较来理解四叉树的概念。接下来，我们了解了如何有效地使用它来存储分布在二维空间中的数据。

然后我们了解了如何存储数据和执行范围搜索。