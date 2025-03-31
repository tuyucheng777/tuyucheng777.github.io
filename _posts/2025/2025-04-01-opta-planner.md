---
layout: post
title:  OptaPlanner指南
category: libraries
copyright: libraries
excerpt: OptaPlanner
---

## 1. OptaPlanner简介

在本教程中，我们将研究一个名为[OptaPlanner](https://www.optaplanner.org/)的Java约束满足求解器。

**注意：要了解它的fork/延续，请查看[Timefold Solver指南](https://www.baeldung.com/java-timefold-solver-guide)**。

OptaPlanner使用一套算法以最少的设置来解决规划问题。

尽管对算法的理解可能会提供有用的细节，但框架会为我们完成艰苦的工作。

## 2. Maven依赖

首先，我们将为OptaPlanner添加Maven依赖：

```xml
<dependency>
    <groupId>org.optaplanner</groupId>
    <artifactId>optaplanner-core</artifactId>
    <version>8.24.0.Final</version>
</dependency>

```

可以从[Maven Central](https://mvnrepository.com/artifact/org.optaplanner/optaplanner-core)找到最新版本的OptaPlanner。

## 3. 问题/解决方案类

为了解决一个问题我们当然需要一个具体的案例作为例子。

由于平衡房间、时间和教师等资源的困难，讲座时间表就是一个合适的例子。

### 3.1 CourseSchedule

CourseSchedule包含我们的问题变量和计划实体的组合，因此它是解决方案类。因此，我们使用多个注解来配置它。

让我们分别仔细看看：

```java
@PlanningSolution
public class CourseSchedule {

    @ValueRangeProvider(id = "availableRooms")
    @ProblemFactCollectionProperty
    private List<Integer> roomList;
    @ValueRangeProvider(id = "availablePeriods")
    @ProblemFactCollectionProperty
    private List<Integer> periodList;
    @ProblemFactCollectionProperty
    private List<Lecture> lectureList;
    @PlanningScore
    private HardSoftScore score;
}
```

PlanningSolution注解告诉OptaPlanner此类包含涵盖解决方案的数据。

**OptaPlanner期望这些最低限度的组成部分：规划实体、问题事实和分数**。

### 3.2 Lecture

Lecture是一个POJO，其形式如下：

```java
@PlanningEntity
public class Lecture {

    @PlaningId
    private Long id;
    public Integer roomNumber;
    public Integer period;
    public String teacher;

    @PlanningVariable(
            valueRangeProviderRefs = {"availablePeriods"})
    public Integer getPeriod() {
        return period;
    }

    @PlanningVariable(
            valueRangeProviderRefs = {"availableRooms"})
    public Integer getRoomNumber() {
        return roomNumber;
    }
}
```

我们使用Lecture类作为规划实体，因此我们在CourseSchedule中的Getter上添加另一个注解：

```java
@PlanningEntityCollectionProperty
public List<Lecture> getLectureList() {
    return lectureList;
}
```

我们的规划实体包含正在设置的约束。

PlanningVariable注解和valueRangeProviderRef注解将约束与问题事实联系起来。

这些约束值稍后将在所有规划实体中进行评分。

### 3.3 问题事实

roomNumber和period变量彼此之间起着类似的约束作用。

OptaPlanner使用这些变量根据逻辑对解决方案进行评分。我们为两个Getter方法添加了注解：

```java
@ValueRangeProvider(id = "availableRooms")
@ProblemFactCollectionProperty
public List<Integer> getRoomList() {
    return roomList;
}

@ValueRangeProvider(id = "availablePeriods")
@ProblemFactCollectionProperty
public List<Integer> getPeriodList() {
    return periodList;
}
```

这些列表是Lecture字段中使用的所有可能的值。

OptaPlanner将它们填充到搜索空间的所有解决方案中。

最后，它会为每个解决方案设置一个分数，因此我们需要一个字段来存储分数：

```java
@PlanningScore
public HardSoftScore getScore() {
    return score;
}
```

**如果没有分数，OptaPlanner就无法找到最优解，因此之前强调了重要性**。

## 4. 评分

与我们迄今为止所见的不同，评分类需要更多的自定义代码。

这是因为分数计算器特定于问题和域模型。

### 4.1. 自定义Java

我们使用一个简单的分数计算来解决这个问题(尽管看起来可能不像)：

```java
public class ScoreCalculator implements EasyScoreCalculator<CourseSchedule, HardSoftScore> {

    @Override
    public HardSoftScore calculateScore(CourseSchedule courseSchedule) {
        int hardScore = 0;
        int softScore = 0;

        Set<String> occupiedRooms = new HashSet<>();
        for(Lecture lecture : courseSchedule.getLectureList()) {
            String roomInUse = lecture.getPeriod()
                    .toString() + ":" + lecture.getRoomNumber().toString();
            if(occupiedRooms.contains(roomInUse)){
                hardScore += -1;
            } else {
                occupiedRooms.add(roomInUse);
            }
        }

        return HardSoftScore.Of(hardScore, softScore);
    }
}
```

如果我们仔细看看上面的代码，重要的部分就变得更加清晰了。**我们在循环中计算分数，因为List<Lecture\>包含特定的非唯一房间和时段组合**。

HashSet用于保存唯一的键(字符串)，以便我们可以对同一房间和同一时段的重复讲座进行惩罚。

因此，我们收到了独特的房间和时期。

## 5. 测试

我们配置了解决方案、求解器和问题类，让我们测试一下。

### 5.1 设置测试

首先，我们做一些设置：

```java
SolverFactory<CourseSchedule> solverFactory = SolverFactory.create(new SolverConfig() 
                                                      .withSolutionClass(CourseSchedule.class)
                                                      .withEntityClasses(Lecture.class)
                                                      .withEasyScoreCalculatorClass(ScoreCalculator.class)
                                                      .withTerminationSpentLimit(Duration.ofSeconds(1))); 
solver = solverFactory.buildSolver();
unsolvedCourseSchedule = new CourseSchedule();
```

其次，我们将数据填充到规划实体集合和问题事实List对象中。

### 5.2 测试执行和验证

最后，我们通过调用solve来测试它。

```java
CourseSchedule solvedCourseSchedule = solver.solve(unsolvedCourseSchedule);

assertNotNull(solvedCourseSchedule.getScore());
assertEquals(-4, solvedCourseSchedule.getScore().getHardScore());
```

我们检查solvedCourseSchedule是否有分数，这告诉我们，我们拥有“最佳”解决方案。

作为奖励，我们创建了一种打印方法来显示我们的优化解决方案：

```java
public void printCourseSchedule() {
    lectureList.stream()
            .map(c -> "Lecture in Room "
                    + c.getRoomNumber().toString()
                    + " during Period " + c.getPeriod().toString())
            .forEach(k -> logger.info(k));
}
```

此方法显示：

```text
Lecture in Room 1 during Period 1
Lecture in Room 2 during Period 1
Lecture in Room 1 during Period 2
Lecture in Room 2 during Period 2
Lecture in Room 1 during Period 3
Lecture in Room 2 during Period 3
Lecture in Room 1 during Period 1
Lecture in Room 1 during Period 1
Lecture in Room 1 during Period 1
Lecture in Room 1 during Period 1
```

注意最后3个条目是如何重复的，发生这种情况是因为我们的问题没有最佳解决方案。我们选择了3个课时、2个教室和10个讲座。

由于这些资源有限，因此只有6堂课可供选择，这个答案至少向用户表明没有足够的房间或课时来容纳所有课程。

## 6. 额外功能

我们为OptaPlanner创建的示例很简单，但是该框架包含更多功能，可用于更多样化的用例。我们可能希望实现或更改我们的优化算法，然后指定要使用它的框架。

由于Java多线程功能的最新改进，OptaPlanner还使开发人员能够使用多线程的多种实现，例如fork和join、增量求解和多租户。

请参阅[文档](https://docs.optaplanner.org/7.9.0.Final/optaplanner-docs/html_single/index.html#multithreadedSolving)以了解更多信息。

## 7. 总结

OptaPlanner框架为开发人员提供了强大的工具来解决调度和资源分配等约束满足问题。

OptaPlanner提供最少的JVM资源使用量以及与Jakarta EE集成，作者继续支持该框架，Red Hat已将其添加为其业务规则管理套件的一部分。