---
layout: post
title:  用Java设计遗传算法
category: algorithms
copyright: algorithms
excerpt: 遗传算法
---

## 1. 简介

本系列的目的是**解释遗传算法的思想**。

遗传算法旨在使用与自然界相同的过程来解决问题-它们使用选择、重组和变异的组合来演化出问题的解决方案。

让我们首先使用最简单的二元遗传算法示例来解释这些算法的概念。

## 2. 遗传算法的工作原理

**遗传算法是进化计算的一部分**，是人工智能中一个快速发展的领域。

算法从**一组解决方案**(用**个体**表示)开始，称为**种群**，从一个种群中选取解决方案并用于形成一个**新的种群**，因为新种群有可能比旧种群更优。

被选中形成新解决方案(**后代**)的个体是根据其适应度来选择的-它们越适应，它们繁殖的机会就越大。

## 3. 二元遗传算法

让我们看一下简单遗传算法的基本过程。

### 3.1 初始化

在初始化步骤中，我们**生成一个随机Population作为第一个解**。首先，我们需要确定Population的规模以及我们期望的最终解是什么：
```java
SimpleGeneticAlgorithm.runAlgorithm(50, "1011000100000100010000100000100111001000000100000100000000001111");
```

在上面的例子中，Population大小为50，正确的解决方案由我们随时可能更改的二进制位串表示。

在下一步中，我们将保存我们想要的解决方案并创建一个随机Population：
```java
setSolution(solution);
Population myPop = new Population(populationSize, true);
```

现在我们准备运行程序的主循环。

### 3.2 健康状况检查

在程序的主循环中，我们将**通过适应度函数来评估每个个体**(简单来说，个体越好，其适应度函数值就越高)：
```java
while (myPop.getFittest().getFitness() < getMaxFitness()) {
    System.out.println("Generation: " + generationCount + " Correct genes found: " + myPop.getFittest().getFitness());
    
    myPop = evolvePopulation(myPop);
    generationCount++;
}
```

让我们首先解释如何获得最适合的个体：
```java
public int getFitness(Individual individual) {
    int fitness = 0;
    for (int i = 0; i < individual.getDefaultGeneLength()
      && i < solution.length; i++) {
        if (individual.getSingleGene(i) == solution[i]) {
            fitness++;
        }
    }
    return fitness;
}
```

正如我们所观察到的，我们一点一点地比较两个Individual对象，如果我们找不到完美的解决方案，就需要进行下一步，即Population的演变。

### 3.3 后代

在此步骤中，我们需要创建一个新的Population。首先，我们需要根据个体的适应度，从Population中选择两个父级Individual对象，请注意，让当前代中的最佳Individual不加改变地延续到下一代是有益的，这种策略称为**精英主义**：
```java
if (elitism) {
    newPopulation.getIndividuals().add(0, pop.getFittest());
    elitismOffset = 1;
} else {
    elitismOffset = 0;
}
```

为了选择两个最佳的个体对象，我们将应用[锦标赛选择策略](https://en.wikipedia.org/wiki/Tournament_selection)：
```java
private Individual tournamentSelection(Population pop) {
    Population tournament = new Population(tournamentSize, false);
    for (int i = 0; i < tournamentSize; i++) {
        int randomId = (int) (Math.random() * pop.getIndividuals().size());
        tournament.getIndividuals().add(i, pop.getIndividual(randomId));
    }
    Individual fittest = tournament.getFittest();
    return fittest;
}
```

每场比赛的获胜者(适应度最好的一方)将被选入下一阶段，即**交叉阶段**：
```java
private Individual crossover(Individual indiv1, Individual indiv2) {
    Individual newSol = new Individual();
    for (int i = 0; i < newSol.getDefaultGeneLength(); i++) {
        if (Math.random() <= uniformRate) {
            newSol.setSingleGene(i, indiv1.getSingleGene(i));
        } else {
            newSol.setSingleGene(i, indiv2.getSingleGene(i));
        }
    }
    return newSol;
}
```

在交叉过程中，我们会在随机选择的点交换每个选定个体的比特位，整个过程在以下循环内运行：
```java
for (int i = elitismOffset; i < pop.getIndividuals().size(); i++) {
    Individual indiv1 = tournamentSelection(pop);
    Individual indiv2 = tournamentSelection(pop);
    Individual newIndiv = crossover(indiv1, indiv2);
    newPopulation.getIndividuals().add(i, newIndiv);
}
```

可以看到，交叉之后，我们将新的后代放入新的Population中，此步骤称为接受。

最后，我们可以进行突变。突变用于维持种群从一代到下一代的遗传多样性，我们使用了位反转类型的突变，其中随机位被简单地反转：
```java
private void mutate(Individual indiv) {
    for (int i = 0; i < indiv.getDefaultGeneLength(); i++) {
        if (Math.random() <= mutationRate) {
            byte gene = (byte) Math.round(Math.random());
            indiv.setSingleGene(i, gene);
        }
    }
}
```

[本教程](http://www.obitko.com/tutorials/genetic-algorithms/crossover-mutation.php)详细描述了所有类型的变异和交叉。

**然后，我们重复3.2和3.3小节中的步骤，直到达到终止条件，例如最佳解决方案**。

## 4. 技巧和窍门

为了实现高效的遗传算法，我们需要调整一组参数。本节将为你提供一些关于如何从最重要的参数入手的基本建议：

- **交叉率**：应该很高，大约**80%-95%**
- **突变率**：应该很低，大约**0.5%-1%**
- **种群规模**：理想的种群规模约为**20-30**，但对于某些问题，规模最好为50-100
- **选择**：基本的[轮盘选择](https://www.baeldung.com/cs/genetic-algorithms-roulette-selection)可以与精英主义的概念一起使用
- **交叉和变异类型**：取决于编码和问题

请注意，调整建议通常是对遗传算法进行实证研究的结果，并且可能会根据所提出的问题而有所不同。

## 5. 总结

本教程介绍遗传算法的基础知识，你无需具备该领域的任何知识，只需具备基本的计算机编程技能即可学习遗传算法。

有关遗传算法的更多示例，请查看我们系列的所有文章：

- 如何设计遗传算法？(本文)
- [Java中的旅行商问题](https://www.baeldung.com/java-simulated-annealing-for-traveling-salesman)