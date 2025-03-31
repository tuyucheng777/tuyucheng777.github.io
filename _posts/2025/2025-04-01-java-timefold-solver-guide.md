---
layout: post
title:  员工排班Timefold求解器指南
category: libraries
copyright: libraries
excerpt: Timefold
---

## 1. 概述

### 1.1 什么是Timefold求解器？

[Timefold Solver](https://timefold.ai/)是一款纯Java规划求解器AI。Timefold可优化规划问题，例如车辆路径问题(VRP)、维护调度、车间作业调度和学校时间表。它可生成物流计划，大幅降低成本、提高服务质量并减少环境足迹(通常可减少高达25%)，适用于复杂的实际调度操作。

Timefold是[OptaPlanner的延续](https://timefold.ai/blog/2023/optaplanner-fork)，它是一种数学优化形式(在更广泛的运筹学和人工智能领域)，支持以代码形式编写的约束。

### 1.2 我们将构建什么

在本教程中，让我们**使用Timefold Solver来优化简化的员工轮班调度问题**。

我们将自动为员工分配轮班，例如：

- 没有员工同一天上两个班次
- 每个班次都分配给具有适当技能的员工

具体来说，我们将分配以下5个班次：

```text
2030-04-01 06:00 - 14:00 (waiter)
2030-04-01 09:00 - 17:00 (bartender)
2030-04-01 14:00 - 22:00 (bartender)
2030-04-02 06:00 - 14:00 (waiter)
2030-04-02 14:00 - 22:00 (bartender)
```

致这3位员工：

```text
Ann (bartender)
Beth (waiter, bartender)
Carl (waiter)
```

这比看上去要难得多，在纸上试一试。

## 2. 依赖

Maven Central上的Timefold Solver工件是根据Apache许可证发布的，让我们使用它们：

### 2.1 纯Java

我们在Maven或Gradle中添加对[timefold-solver-core](https://mvnrepository.com/artifact/ai.timefold.solver/timefold-solver-core)的依赖以及对[timefold-solver-test](https://mvnrepository.com/artifact/ai.timefold.solver/timefold-solver-test)的测试依赖：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>ai.timefold.solver</groupId>
            <artifactId>timefold-solver-bom</artifactId>
            <version>...</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>ai.timefold.solver</groupId>
        <artifactId>timefold-solver-core</artifactId>
    </dependency>
    <dependency>
        <groupId>ai.timefold.solver</groupId>
        <artifactId>timefold-solver-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2.2 Spring Boot

在Spring Boot中，我们改用[timefold-solver-spring-boot-starter](https://mvnrepository.com/artifact/ai.timefold.solver/timefold-solver-spring-boot-starter)依赖，它会自动处理大部分求解器配置(稍后我们将看到)，并允许在application.properties中配置求解器时间和其他属性。

1. 转到[start.spring.io](https://start.spring.io/)
2. 单击“Add dependencies”以添加Timefold Solver依赖
3. 生成项目并在你最喜欢的IDE中打开它

### 2.3 Quarkus

在Quarkus中，类似地，我们使用[code.quarkus.io](https://code.quarkus.io/)中的[timefold-solver-quarkus](https://mvnrepository.com/artifact/ai.timefold.solver/timefold-solver-quarkus)依赖来实现自动求解器配置和application.properties支持。

## 3. 域类

域类表示输入数据和输出数据，我们创建Employee和Shift类，以及包含特定数据集的员工和班次列表的ShiftSchedule。

### 3.1 Employee

员工是我们可以分配轮班的人，每个员工都有一个姓名和一项或多项技能。

Employee类不需要任何Timefold注解，因为它在求解过程中不会改变：

```java
public class Employee {

    private String name;
    private Set<String> skills;

    public Employee(String name, Set<String> skills) {
        this.name = name;
        this.skills = skills;
    }

    @Override
    public String toString() {
        return name;
    }

    // Getters and setters
}
```

### 3.2 Shift

班次是指在特定日期从开始时间到结束时间分配给一名员工的工作，可以同时有两个班次，每个班次都有一项必备技能。

Shift对象在求解过程中发生变化：每个班次都分配给一名员工。Timefold需要知道这一点，只有employy字段在求解过程中发生变化。因此，我们用@PlanningEntity标注类，用@PlanningVariable标注employee字段，这样Timefold就知道它应该为每个班次填充员工：

```java
@PlanningEntity
public class Shift {

    private LocalDateTime start;
    private LocalDateTime end;
    private String requiredSkill;

    @PlanningVariable
    private Employee employee;

    // A no-arg constructor is required for @PlanningEntity annotated classes
    public Shift() {
    }

    public Shift(LocalDateTime start, LocalDateTime end, String requiredSkill) {
        this(start, end, requiredSkill, null);
    }

    public Shift(LocalDateTime start, LocalDateTime end, String requiredSkill, Employee employee) {
        this.start = start;
        this.end = end;
        this.requiredSkill = requiredSkill;
        this.employee = employee;
    }

    @Override
    public String toString() {
        return start + " - " + end;
    }

    // Getters and setters
}
```

### 3.3 ShiftSchedule

时间表代表员工和班次的单个数据集，它既是Timefold的输入，也是输出：

- 我们用@PlanningSolution标注ShiftSchedule类，以便Timefold知道它代表输入和输出。
- 我们用@ValueRangeProvider标注employee字段，以告诉Timefold它包含员工列表，可以从中选择要分配给Shift.employee的实例。
- 我们用@PlanningEntityCollectionProperty标注shifts字段，以便Timefold找到所有Shift实例并分配给员工。
- 我们包含一个带有@PlanningScore注解的score字段，Timefold将为我们填充此字段，允许我们使用HardSoftScore，以便我们稍后可以区分硬约束和软约束。

现在，让我们看一下我们的类：

```java
@PlanningSolution
public class ShiftSchedule {

    @ValueRangeProvider
    private List<Employee> employees;
    @PlanningEntityCollectionProperty
    private List<Shift> shifts;

    @PlanningScore
    private HardSoftScore score;

    // A no-arg constructor is required for @PlanningSolution annotated classes
    public ShiftSchedule() {
    }

    public ShiftSchedule(List<Employee> employees, List<Shift> shifts) {
        this.employees = employees;
        this.shifts = shifts;
    }

    // Getters and setters
}
```

## 4. 约束

如果没有约束，Timefold会将所有班次分配给第一位员工，这不是一个可行的时间表。

为了教会它如何区分好的和坏的时间表，让我们添加两个硬约束：

- atMostOneShiftPerDay()约束检查是否在同一天将两个班次分配给了同一名员工，如果是，则会扣掉1分。
- requiredSkill()约束检查是否将班次分配给员工，并且该班次所需的技能是否是员工技能组合的一部分。如果不是，则会将分数扣掉1分。

**单一硬约束优先于所有软约束**。通常，硬约束无论是从物理上还是法律上都不可能被打破。另一方面，软约束可以被打破，但我们希望将其最小化。这些通常代表财务成本、服务质量或员工幸福感。硬约束和软约束使用相同的API实现。

### 4.1 ConstraintProvider

首先，我们为约束实现创建一个ConstraintProvider：

```java
public class ShiftScheduleConstraintProvider implements ConstraintProvider {

    @Override
    public Constraint[] defineConstraints(ConstraintFactory constraintFactory) {
        return new Constraint[] {
                atMostOneShiftPerDay(constraintFactory),
                requiredSkill(constraintFactory)
        };
    }

    // Constraint implementations
}
```

### 4.2 对ConstraintProvider进行单元测试

如果没有测试，它就无法工作-尤其是对于约束。让我们创建一个测试类来测试ConstraintProvider的每个约束。

测试范围的timefold-solver-test依赖包含ConstraintVerifier，它是用于单独测试每个约束的辅助程序。这改善了维护-我们可以重构单个约束而不会破坏其他约束的测试：

```java
public class ShiftScheduleConstraintProviderTest {

    private static final LocalDate MONDAY = LocalDate.of(2030, 4, 1);
    private static final LocalDate TUESDAY = LocalDate.of(2030, 4, 2);

    ConstraintVerifier<ShiftScheduleConstraintProvider, ShiftSchedule> constraintVerifier
            = ConstraintVerifier.build(new ShiftScheduleConstraintProvider(), ShiftSchedule.class, Shift.class);

    // Tests for each constraint
}
```

我们还准备了两个日期，以便在下面的测试中重复使用，接下来让我们添加实际的约束。

### 4.3 硬约束：每天最多一班

按照TDD(测试驱动开发)，让我们首先在测试类中为新的约束编写测试：

```java
@Test
void whenTwoShiftsOnOneDay_thenPenalize() {
    Employee ann = new Employee("Ann", null);
    constraintVerifier.verifyThat(ShiftScheduleConstraintProvider::atMostOneShiftPerDay)
            .given(
                    new Shift(MONDAY.atTime(6, 0), MONDAY.atTime(14, 0), null, ann),
                    new Shift(MONDAY.atTime(14, 0), MONDAY.atTime(22, 0), null, ann))
            // Penalizes by 2 because both {shiftA, shiftB} and {shiftB, shiftA} match.
            // To avoid that, use forEachUniquePair() in the constraint instead of forEach().join() in the implementation.
            .penalizesBy(2);
}

@Test
void whenTwoShiftsOnDifferentDays_thenDoNotPenalize() {
    Employee ann = new Employee("Ann", null);
    constraintVerifier.verifyThat(ShiftScheduleConstraintProvider::atMostOneShiftPerDay)
            .given(
                    new Shift(MONDAY.atTime(6, 0), MONDAY.atTime(14, 0), null, ann),
                    new Shift(TUESDAY.atTime(14, 0), TUESDAY.atTime(22, 0), null, ann))
            .penalizesBy(0);
}
```

然后，我们在ConstraintProvider中实现它：

```java
public Constraint atMostOneShiftPerDay(ConstraintFactory constraintFactory) {
    return constraintFactory.forEach(Shift.class)
            .join(Shift.class,
                    equal(shift -> shift.getStart().toLocalDate()),
                    equal(Shift::getEmployee))
            .filter((shift1, shift2) -> shift1 != shift2)
            .penalize(HardSoftScore.ONE_HARD)
            .asConstraint("At most one shift per day");
}
```

为了实现约束，我们使用ConstraintStreams API：一种类似于Stream/SQL的API，可在后台提供增量分数计算(deltas)和索引哈希表查找。此方法可扩展到单个计划中具有数十万个班次的数据集。

让我们运行测试并验证它们是可行的。

### 4.4 硬约束：所需技能

让我们在测试类中编写测试：

```java
@Test
void whenEmployeeLacksRequiredSkill_thenPenalize() {
    Employee ann = new Employee("Ann", Set.of("Waiter"));
    constraintVerifier.verifyThat(ShiftScheduleConstraintProvider::requiredSkill)
            .given(
                    new Shift(MONDAY.atTime(6, 0), MONDAY.atTime(14, 0), "Cook", ann))
            .penalizesBy(1);
}

@Test
void whenEmployeeHasRequiredSkill_thenDoNotPenalize() {
    Employee ann = new Employee("Ann", Set.of("Waiter"));
    constraintVerifier.verifyThat(ShiftScheduleConstraintProvider::requiredSkill)
            .given(
                    new Shift(MONDAY.atTime(6, 0), MONDAY.atTime(14, 0), "Waiter", ann))
            .penalizesBy(0);
}
```

然后，让我们在ConstraintProvider中实现新的约束：

```java
public Constraint requiredSkill(ConstraintFactory constraintFactory) {
    return constraintFactory.forEach(Shift.class)
            .filter(shift -> !shift.getEmployee().getSkills()
                    .contains(shift.getRequiredSkill()))
            .penalize(HardSoftScore.ONE_HARD)
            .asConstraint("Required skill");
}
```

我们再运行一下测试，测试结果仍然正常。

**为了使其成为软约束，我们将penalize(HardSoftScore.ONE_HARD)更改为penalize(HardSoftScore.ONE_SOFT)**。为了将其转变为输入数据集的动态决策，我们可以改用penalizeConfigurable()和@ConstraintWeight。

## 5. 应用程序

我们已准备好整理我们的应用程序。

### 5.1 求解

为了解决计划问题，我们从@PlanningSolution、@PlanningEntity和ConstraintProvider类中创建SolverFactory。SolverFactory是一个长期存在的对象，通常，每个应用程序只有一个实例。

我们还需要配置求解器运行的时间，对于大型数据集，由于存在数千个移位和更多约束，不可能在合理的时间范围内找到最佳解决方案(由于[NP难题](https://en.wikipedia.org/wiki/NP-hardness)的指数性质)。相反，我们希望在可用的时间内找到最佳解决方案。让我们暂时将其限制为2秒钟：

```java
SolverFactory<ShiftSchedule> solverFactory = SolverFactory.create(new SolverConfig()
    .withSolutionClass(ShiftSchedule.class)
    .withEntityClasses(Shift.class)
    .withConstraintProviderClass(ShiftScheduleConstraintProvider.class)
    // The solver runs only for 2 seconds on this tiny dataset.
    // It's recommended to run for at least 5 minutes ("5m") on large datasets.
    .withTerminationSpentLimit(Duration.ofSeconds(2)));
```

我们使用SolverFactory为每个数据集创建一个Solver实例，然后，我们调用Solver.solve()来求解数据集：

```java
Solver<ShiftSchedule> solver = solverFactory.buildSolver();
ShiftSchedule problem = loadProblem();
ShiftSchedule solution = solver.solve(problem);
printSolution(solution);
```

在Spring Boot中，SolverFactory会自动构建并注入到@Autowired字段中：

```java
@Autowired
SolverFactory<ShiftSchedule> solverFactory;
```

我们在application.properties中配置求解器时间：

```properties
timefold.solver.termination.spent-limit=5s
```

在Quarkus中，同样会自动构建SolverFactory并将其注入到@Inject字段中，求解器时间也可以在application.properties中配置。

为了异步求解，为了避免在调用Solver.solve()时占用当前线程，我们将注入并使用SolverManager。

### 5.2 测试数据

让我们生成一个包含5个班次和3名员工的小数据集作为输入问题：

```java
private ShiftSchedule loadProblem() {
    LocalDate monday = LocalDate.of(2030, 4, 1);
    LocalDate tuesday = LocalDate.of(2030, 4, 2);
    return new ShiftSchedule(List.of(
            new Employee("Ann", Set.of("Bartender")),
            new Employee("Beth", Set.of("Waiter", "Bartender")),
            new Employee("Carl", Set.of("Waiter"))
    ), List.of(
            new Shift(monday.atTime(6, 0), monday.atTime(14, 0), "Waiter"),
            new Shift(monday.atTime(9, 0), monday.atTime(17, 0), "Bartender"),
            new Shift(monday.atTime(14, 0), monday.atTime(22, 0), "Bartender"),
            new Shift(tuesday.atTime(6, 0), tuesday.atTime(14, 0), "Waiter"),
            new Shift(tuesday.atTime(14, 0), tuesday.atTime(22, 0), "Bartender")
    ));
}
```

### 5.3 结果

通过我们的求解器运行测试数据后，我们将输出解决方案打印到System.out：

```java
private void printSolution(ShiftSchedule solution) {
    logger.info("Shift assignments");
    for (Shift shift : solution.getShifts()) {
        logger.info("  " + shift.getStart().toLocalDate()
                + " " + shift.getStart().toLocalTime()
                + " - " + shift.getEnd().toLocalTime()
                + ": " + shift.getEmployee().getName());
    }
}
```

以下是我们的数据集的结果：

```text
Shift assignments
  2030-04-01 06:00 - 14:00: Carl
  2030-04-01 09:00 - 17:00: Ann
  2030-04-01 14:00 - 22:00: Beth
  2030-04-02 06:00 - 14:00: Beth
  2030-04-02 14:00 - 22:00: Ann
```

Ann没有被分配到第一班，因为她没有waiter技能。**但是Beth为什么没有被分配到第一班？因为她有waiter技能**。

如果Beth被分配到第一班，那么就不可能同时分配第二班和第三班，这两个班都需要bartender。所以Carl不能做。只有当Carl被分配到第一班时，才有可能找到可行的解决方案。在大型现实世界数据集中，这些错综复杂的情况会变得更加复杂，因此让求解器来处理它们是理想的方式。

## 6. 总结

Timefold Solver框架为开发人员提供了强大的工具来解决约束满足问题，例如调度和资源分配。它支持以代码(而不是数学方程式)编写自定义约束，这使得它易于维护。在底层，它支持各种可以进行功率调整的人工智能优化算法，但一般用户不需要这样做。