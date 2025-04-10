---
layout: post
title:  蚁群优化与Java示例
category: algorithms
copyright: algorithms
excerpt: 蚁群优化
---

## 1. 简介

[本系列](https://www.baeldung.com/java-genetic-algorithm)的目的是**解释遗传算法的思想并展示最为人所知的实现**。

在本教程中，我们将**描述蚁群优化(ACO)的概念，然后提供代码示例**。

## 2. ACO的工作原理

ACO是一种受蚂蚁自然行为启发的遗传算法，为了充分理解ACO算法，我们需要熟悉其基本概念：

- 蚂蚁利用信息素寻找家和食物源之间的最短路径
- 信息素蒸发很快
- 蚂蚁更喜欢使用信息素更密集的较短路径

让我们展示一个在[旅行商问题](https://www.baeldung.com/java-simulated-annealing-for-traveling-salesman)中使用的ACO的简单示例，在以下情况下，我们需要找到图中所有节点之间的最短路径：

![](/assets/images/2025/algorithms/javaantcolonyoptimization01.png)

按照自然行为，蚂蚁在探索过程中会开始探索新的路径，颜色较深的蓝色表示比其他路径更常用的路径，而绿色表示当前找到的最短路径：

![](/assets/images/2025/algorithms/javaantcolonyoptimization02.png)

最终我们将实现所有节点之间的最短路径：

![](/assets/images/2025/algorithms/javaantcolonyoptimization03.png)

你可以在[此处](http://www.theprojectspot.com/downloads/tsp-aco.html)找到用于ACO测试的基于GUI的优质工具。

## 3. Java实现

### 3.1 ACO参数

让我们讨论一下AntColonyOptimization类中声明的ACO算法的主要参数：
```java
private double c = 1.0;
private double alpha = 1;
private double beta = 5;
private double evaporation = 0.5;
private double Q = 500;
private double antFactor = 0.8;
private double randomFactor = 0.01;
```

参数c表示模拟开始时的原始轨迹数，此外，alpha控制信息素重要性，而beta控制距离优先级。**通常，为了获得最佳效果，beta参数应大于alpha**。

接下来，evaporation变量显示每次迭代中信息素蒸发的百分比，而Q提供有关每只蚂蚁在路径上留下的信息素总量的信息，antFactor告诉我们每个城市将使用多少只蚂蚁。

最后，我们需要在模拟中加入一些随机性，这可以通过randomFactor来实现。

### 3.2 创建蚂蚁

每只蚂蚁将能够访问一个特定的城市，记住所有访问过的城市，并跟踪路径长度：
```java
public void visitCity(int currentIndex, int city) {
    trail[currentIndex + 1] = city;
    visited[city] = true;
}

public boolean visited(int i) {
    return visited[i];
}

public double trailLength(double graph[][]) {
    double length = graph[trail[trailSize - 1]][trail[0]];
    for (int i = 0; i < trailSize - 1; i++) {
        length += graph[trail[i]][trail[i + 1]];
    }
    return length;
}
```

### 3.3 设置Ant

首先，我们需要通过提供路径和蚂蚁矩阵来初始化我们的ACO代码实现：
```java
graph = generateRandomMatrix(noOfCities);
numberOfCities = graph.length;
numberOfAnts = (int) (numberOfCities * antFactor);

trails = new double[numberOfCities][numberOfCities];
probabilities = new double[numberOfCities];
ants = new Ant[numberOfAnts];
IntStream.range(0, numberOfAnts).forEach(i -> ants.add(new Ant(numberOfCities)));
```

接下来，我们需要设置蚂蚁矩阵，从一个随机城市开始：
```java
public void setupAnts() {
    IntStream.range(0, numberOfAnts)
            .forEach(i -> {
                ants.forEach(ant -> {
                    ant.clear();
                    ant.visitCity(-1, random.nextInt(numberOfCities));
                });
            });
    currentIndex = 0;
}
```

对于循环的每次迭代，我们将执行以下操作：
```java
IntStream.range(0, maxIterations).forEach(i -> {
    moveAnts();
    updateTrails();
    updateBest();
});
```

### 3.4 移动蚂蚁

让我们从moveAnts()方法开始，我们需要为所有蚂蚁选择下一个城市，记住每只蚂蚁都会尝试追随其他蚂蚁的踪迹：
```java
public void moveAnts() {
    IntStream.range(currentIndex, numberOfCities - 1).forEach(i -> {
        ants.forEach(ant -> {
            ant.visitCity(currentIndex, selectNextCity(ant));
        });
        currentIndex++;
    });
}
```

**最重要的部分是正确选择下一个要访问的城市**，我们应该根据概率逻辑选择下一个城镇。首先，我们可以检查蚂蚁是否应该访问一个随机城市：
```java
int t = random.nextInt(numberOfCities - currentIndex);
if (random.nextDouble() < randomFactor) {
    OptionalInt cityIndex = IntStream.range(0, numberOfCities)
        .filter(i -> i == t && !ant.visited(i))
        .findFirst();
    if (cityIndex.isPresent()) {
        return cityIndex.getAsInt();
    }
}
```

如果我们没有选择任何随机城市，我们需要计算选择下一个城市的概率，记住蚂蚁更喜欢沿着更远、更短的路径移动，我们可以通过在数组中存储移动到每个城市的概率来实现这一点：
```java
public void calculateProbabilities(Ant ant) {
    int i = ant.trail[currentIndex];
    double pheromone = 0.0;
    for (int l = 0; l < numberOfCities; l++) {
        if (!ant.visited(l)){
            pheromone += Math.pow(trails[i][l], alpha) * Math.pow(1.0 / graph[i][l], beta);
        }
    }
    for (int j = 0; j < numberOfCities; j++) {
        if (ant.visited(j)) {
            probabilities[j] = 0.0;
        } else {
            double numerator = Math.pow(trails[i][j], alpha) * Math.pow(1.0 / graph[i][j], beta);
            probabilities[j] = numerator / pheromone;
        }
    }
}
```

计算概率后，我们可以使用以下方法决定去哪个城市：
```java
double r = random.nextDouble();
double total = 0;
for (int i = 0; i < numberOfCities; i++) {
    total += probabilities[i];
    if (total >= r) {
        return i;
    }
}
```

### 3.5 更新轨迹

在这一步中，我们应该更新路径和左侧的信息素：
```java
public void updateTrails() {
    for (int i = 0; i < numberOfCities; i++) {
        for (int j = 0; j < numberOfCities; j++) {
            trails[i][j] *= evaporation;
        }
    }
    for (Ant a : ants) {
        double contribution = Q / a.trailLength(graph);
        for (int i = 0; i < numberOfCities - 1; i++) {
            trails[a.trail[i]][a.trail[i + 1]] += contribution;
        }
        trails[a.trail[numberOfCities - 1]][a.trail[0]] += contribution;
    }
}
```

### 3.6 更新最佳解决方案

这是每次迭代的最后一步，我们需要更新最佳解决方案以保留对它的引用：
```java
private void updateBest() {
    if (bestTourOrder == null) {
        bestTourOrder = ants[0].trail;
        bestTourLength = ants[0].trailLength(graph);
    }
    for (Ant a : ants) {
        if (a.trailLength(graph) < bestTourLength) {
            bestTourLength = a.trailLength(graph);
            bestTourOrder = a.trail.clone();
        }
    }
}
```

经过所有迭代后，最终结果将显示ACO找到的最佳路径。**请注意，随着城市数量的增加，找到最短路径的概率会降低**。

## 4. 总结

本教程介绍了蚁群优化算法，你无需任何该领域的知识，只需具备基本的计算机编程技能即可学习遗传算法。

对于本系列的所有文章，包括遗传算法的其他示例，请查看以下链接：

- [如何用Java设计遗传算法](https://www.baeldung.com/java-genetic-algorithm)
- [Java中的旅行商问题](https://www.baeldung.com/java-simulated-annealing-for-traveling-salesman)
- 蚁群优化(本文)