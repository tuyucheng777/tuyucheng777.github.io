---
layout: post
title:  Java中的多群优化算法
category: algorithms
copyright: algorithms
excerpt: 多群优化算法
---

## 1. 简介

在本文中，我们将介绍多群优化算法。与同类的其他算法一样，其目的是通过最大化或最小化特定函数(称为适应度函数)来找到问题的最佳解决方案。

让我们从一些理论开始。

## 2. 多群优化算法的工作原理

多群算法是[Swarm](https://en.wikipedia.org/wiki/Particle_swarm_optimization)算法的一种变体，顾名思义，**Swarm算法通过模拟一组对象在可能解决方案空间中的移动来解决问题**。在多群算法中，有多个群，而不仅仅是一个。

群的基本组成部分称为粒子，粒子由其实际位置(也是我们问题的一个可能解)和速度(用于计算下一个位置)定义。

粒子的速度不断变化，以一定的随机性倾向于在所有群体中的所有粒子中找到最佳位置，以增加覆盖的空间量。

这最终会导致大多数粒子趋向于适应度函数中的一组有限点，这些点是局部最小值或最大值，具体取决于我们是试图最小化还是最大化它。

虽然找到的点始终是函数的局部最小值或最大值，但它不一定是全局点，因为无法保证算法已完全探索解空间。

因此，多群体被称为[元启发式方法](https://www.baeldung.com/cs/nature-inspired-algorithms)-**它找到的解决方案是最好的，但可能不是绝对最好的**。

## 3. 实现

现在我们知道了多群是什么以及它是如何工作的，让我们看看如何实现它。

为了举个例子，我们将尝试解决[StackExchange](https://matheducators.stackexchange.com/questions/1550/optimization-problems-that-todays-students-might-actually-encounter/1561#1561)上发布的这个实际优化问题：

> 在英雄联盟中，玩家抵御物理伤害时的有效生命值是E = H(100+A) / 100，其中H代表生命值，A代表护甲。
>
> 生命值每单位花费2.5金币，护甲每单位花费18金币；你有3600金币，需要优化生命值和护甲的E效能，以便在敌方队伍的攻击下尽可能长时间存活。你应该分别购买多少？

### 3.1 粒子

我们首先对基本构造粒子进行建模，粒子的状态包括其当前位置(即用于解决问题的一对生命值和护甲值)、粒子在两个轴上的速度以及粒子适应度得分。

我们还将存储找到的最佳位置和适应度分数，因为我们需要它们来更新粒子速度：
```java
public class Particle {
    private long[] position;
    private long[] speed;
    private double fitness;
    private long[] bestPosition;	
    private double bestFitness = Double.NEGATIVE_INFINITY;

    // constructors and other methods
}
```

我们选择使用long数组来表示速度和位置，因为我们可以从问题陈述中推断出我们无法购买护甲或生命值的分数，因此解决方案必须在整数域中。

我们不想使用int，因为它会在计算过程中引起溢出问题。

### 3.2 群体

接下来，我们将群体定义为粒子的集合。同样，我们也会存储历史最佳位置和得分，以供后续计算。

群体还需要通过为每个粒子分配随机的初始位置和速度来处理粒子的初始化。

我们可以粗略地估计解决方案的边界，因此我们将这个限制添加到随机数生成器中。

这将减少运行算法所需的计算能力和时间：
```java
public class Swarm {
    private Particle[] particles;
    private long[] bestPosition;
    private double bestFitness = Double.NEGATIVE_INFINITY;

    public Swarm(int numParticles) {
        particles = new Particle[numParticles];
        for (int i = 0; i < numParticles; i++) {
            long[] initialParticlePosition = {
                    random.nextInt(Constants.PARTICLE_UPPER_BOUND),
                    random.nextInt(Constants.PARTICLE_UPPER_BOUND)
            };
            long[] initialParticleSpeed = {
                    random.nextInt(Constants.PARTICLE_UPPER_BOUND),
                    random.nextInt(Constants.PARTICLE_UPPER_BOUND)
            };
            particles[i] = new Particle(
                    initialParticlePosition, initialParticleSpeed);
        }
    }

    // methods omitted
}
```

### 3.3 多群体

最后，让我们通过创建Multiswarm类来结束我们的模型。

与群体类似，我们将跟踪群体集合以及在所有群体中找到的最佳粒子位置和适应度。

我们还将存储对适应度函数的引用以供以后使用：
```java
public class Multiswarm {
    private Swarm[] swarms;
    private long[] bestPosition;
    private double bestFitness = Double.NEGATIVE_INFINITY;
    private FitnessFunction fitnessFunction;

    public Multiswarm(int numSwarms, int particlesPerSwarm, FitnessFunction fitnessFunction) {
        this.fitnessFunction = fitnessFunction;
        this.swarms = new Swarm[numSwarms];
        for (int i = 0; i < numSwarms; i++) {
            swarms[i] = new Swarm(particlesPerSwarm);
        }
    }

    // methods omitted
}
```

### 3.4 适应度函数

现在让我们实现适应度函数。

为了将算法逻辑与这个特定问题分离，我们将引入一个具有单一方法的接口。

该方法以粒子位置作为参数，并返回一个表示其好坏程度的值：
```java
public interface FitnessFunction {
    public double getFitness(long[] particlePosition);
}
```

假设根据问题约束找到的结果有效，那么衡量适应度只是返回我们想要最大化的计算有效生命值的问题。

对于我们的问题，我们有以下具体的验证约束：

- 解只能是正整数
- 解决方案必须能够利用所提供的金币数量

当违反其中一个约束时，我们会返回一个负数，表示我们距离有效边界有多远。

这要么是前一种情况下发现的数量，要么是后一种情况下不可用的金币数量：
```java
public class LolFitnessFunction implements FitnessFunction {

    @Override
    public double getFitness(long[] particlePosition) {
        long health = particlePosition[0];
        long armor = particlePosition[1];

        if (health < 0 && armor < 0) {
            return -(health * armor);
        } else if (health < 0) {
            return health;
        } else if (armor < 0) {
            return armor;
        }

        double cost = (health * 2.5) + (armor * 18);
        if (cost > 3600) {
            return 3600 - cost;
        } else {
            long fitness = (health * (100 + armor)) / 100;
            return fitness;
        }
    }
}
```

### 3.5 主循环

主程序将在所有群中的所有粒子之间进行迭代并执行以下操作：

- 计算粒子适应度
- 如果找到了新的最佳位置，则更新粒子、群体和多群体历史记录
- 通过将当前速度添加到每个维度来计算新的粒子位置
- 计算新的粒子速度

目前，我们将通过创建专用方法将速度更新留到下一部分：
```java
public void mainLoop() {
    for (Swarm swarm : swarms) {
        for (Particle particle : swarm.getParticles()) {
            long[] particleOldPosition = particle.getPosition().clone();
            particle.setFitness(fitnessFunction.getFitness(particleOldPosition));

            if (particle.getFitness() > particle.getBestFitness()) {
                particle.setBestFitness(particle.getFitness());
                particle.setBestPosition(particleOldPosition);
                if (particle.getFitness() > swarm.getBestFitness()) {
                    swarm.setBestFitness(particle.getFitness());
                    swarm.setBestPosition(particleOldPosition);
                    if (swarm.getBestFitness() > bestFitness) {
                        bestFitness = swarm.getBestFitness();
                        bestPosition = swarm.getBestPosition().clone();
                    }
                }
            }

            long[] position = particle.getPosition();
            long[] speed = particle.getSpeed();
            position[0] += speed[0];
            position[1] += speed[1];
            speed[0] = getNewParticleSpeedForIndex(particle, swarm, 0);
            speed[1] = getNewParticleSpeedForIndex(particle, swarm, 1);
        }
    }
}
```

### 3.6 速度更新

粒子改变其速度至关重要，因为这是它探索不同可能解决方案的方式。

粒子的速度需要使粒子向其自身、其群体和所有群体找到的最佳位置移动，并为每个位置分配一定的权重。我们分别将这些权重称为**认知权重、社交权重和全局权重**。

为了增加一些变化，我们将每个权重乘以0到1之间的一个随机数。我们还将在公式中添加一个惯性因子，以激励粒子不要减速太多：
```java
private int getNewParticleSpeedForIndex(Particle particle, Swarm swarm, int index) {
    return (int) ((Constants.INERTIA_FACTOR * particle.getSpeed()[index])
            + (randomizePercentage(Constants.COGNITIVE_WEIGHT)
            * (particle.getBestPosition()[index] - particle.getPosition()[index]))
            + (randomizePercentage(Constants.SOCIAL_WEIGHT)
            * (swarm.getBestPosition()[index] - particle.getPosition()[index]))
            + (randomizePercentage(Constants.GLOBAL_WEIGHT)
            * (bestPosition[index] - particle.getPosition()[index])));
}
```

惯性、认知、社会和全局权重的可接受值分别为0.729、1.49445、1.49445和0.3645。

## 4. 总结

在本教程中，我们学习了群体算法的理论和实现，我们还了解了如何根据具体问题设计适应度函数。

如果你想了解有关该主题的更多信息，请阅读[这本书](https://books.google.it/books?id=Xl9uCQAAQBAJ)和[这篇文章](https://msdn.microsoft.com/en-us/magazine/dn385711.aspx)，它们也是本文的信息来源。