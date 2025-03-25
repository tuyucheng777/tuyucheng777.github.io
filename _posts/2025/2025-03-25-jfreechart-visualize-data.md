---
layout: post
title:  JFreeChart简介
category: libraries
copyright: libraries
excerpt: JFreeChart
---

## 1. 概述

在本教程中，我们将了解如何使用[JFreeChart](https://github.com/jfree/jfreechart)，这是一个用于创建各种图表的综合Java库。我们可以使用它来将图形数据表示集成到Swing应用程序中。它还包括一个单独的[JavaFX扩展](https://github.com/jfree/jfreechart-fx)。

我们将从基础开始，涵盖设置和图表创建，并尝试几种不同类型的图表。

## 2. 创建我们的第一个图表

JFreeChart允许我们创建折线图、条形图、饼图、散点图、时间序列图、直方图等，它还可以将不同的图表组合成一个图。

### 2.1 设置依赖

首先，我们需要将[jfreechart](https://mvnrepository.com/artifact/org.jfree/jfreechart)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.jfree</groupId>
    <artifactId>jfreechart</artifactId>
    <version>1.5.4</version>
</dependency>
```

**我们应该始终检查[最新版本](https://github.com/jfree/jfreechart/releases)及其JDK兼容性**，以确保我们的项目是最新的且正常运行。在这种情况下，版本1.5.4需要JDK 8或更高版本。

### 2.2 创建基本折线图

让我们首先使用[DefaultCategoryDataset](https://www.jfree.org/jfreechart/javadoc/org/jfree/data/category/DefaultCategoryDataset.html)为我们的图表创建一个数据集：

```java
DefaultCategoryDataset dataset = new DefaultCategoryDataset();
dataset.addValue(200, "Sales", "January");
dataset.addValue(150, "Sales", "February");
dataset.addValue(180, "Sales", "March");
dataset.addValue(260, "Sales", "April");
dataset.addValue(300, "Sales", "May");
```

现在，我们可以创建一个[JFreeChart](https://www.jfree.org/jfreechart/javadoc/org/jfree/chart/JFreeChart.html)对象，使用上面的数据集来绘制折线图。

ChartFactory.[createLineChart](https://www.jfree.org/jfreechart/javadoc/org/jfree/chart/ChartFactory.html#createLineChart(java.lang.String,java.lang.String,java.lang.String,org.jfree.data.category.CategoryDataset))方法将图表标题、x轴和y轴标签以及数据集作为参数：

```java
JFreeChart chart = ChartFactory.createLineChart(
    "Monthly Sales",
    "Month",
    "Sales",
    dataset);
```

接下来，[ChartPanel](https://www.jfree.org/jfreechart/javadoc/org/jfree/chart/ChartPanel.html)对象对于在Swing组件中显示图表至关重要。然后，此对象用作[JFrame](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JFrame.html)内容窗格来创建应用程序窗口：

```java
ChartPanel chartPanel = new ChartPanel(chart);
JFrame frame = new JFrame();
frame.setSize(800, 600);
frame.setContentPane(chartPanel);
frame.setLocationRelativeTo(null);
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setVisible(true);
```

当我们运行此代码时，我们将看到月度销售额图表：

![](/assets/images/2025/libraries/jfreechartvisualizedata01.png)

如我们所见，我们仅使用少量代码就创建了一个图表。

## 3. 探索不同类型的图表

在剩下的例子中，我们将尝试一些不同类型的图表。我们不需要对代码进行太多更改。

### 3.1 条形图

我们可以修改JFreeChart创建代码，将折线图转换为条形图：

```java
JFreeChart chart = ChartFactory.createBarChart(
    "Monthly Sales",
    "Month",
    "Sales",
    dataset);
```

绘制上一个示例中的数据集：

![](/assets/images/2025/libraries/jfreechartvisualizedata02.png)

我们可以看到，JFreeChart非常灵活，可以轻松地使用不同类型的图表显示相同的数据。

### 3.2 饼图

**饼图显示整体中各部分的比例**。要创建饼图，我们需要使用[DefaultPieDataset](https://www.jfree.org/jfreechart/javadoc/org/jfree/data/general/DefaultPieDataset.html)类来创建数据集，我们还使用createPieChart()方法来构建JFreeChart对象：

```java
DefaultPieDataset<String> dataset = new DefaultPieDataset<>();
dataset.setValue("January", 200);
dataset.setValue("February", 150);
dataset.setValue("March", 180);

JFreeChart chart = ChartFactory.createPieChart(
    "Monthly Sales",
    dataset,
    true,    // include legend
    true,    // generate tooltips
    false);  // no URLs
```

当我们将鼠标悬停在饼图的某个部分上时，可以看到工具提示，显示一个月的绝对销售额和总额的相对百分比：

![](/assets/images/2025/libraries/jfreechartvisualizedata03.png)

最后，我们应该注意到， ChartFactory.createPieChart （）方法有几种变体，以实现更精细[的](https://www.jfree.org/jfreechart/javadoc/org/jfree/chart/ChartFactory.html#createPieChart(java.lang.String,org.jfree.data.general.PieDataset))定制。

### 3.3 时间序列图

**时间序列图显示数据随时间的变化趋势**。要构建数据集，我们需要一个[TimeSeriesCollection](https://www.jfree.org/jfreechart/javadoc/org/jfree/data/time/TimeSeriesCollection.html)对象，它是[TimeSeries](https://www.jfree.org/jfreechart/javadoc/org/jfree/data/time/TimeSeries.html)对象的集合，每个对象都是包含与特定时间段相关的值的数据项序列：

```java
TimeSeries series = new TimeSeries("Monthly Sales");
series.add(new Month(1, 2024), 200);
series.add(new Month(2, 2024), 150);
series.add(new Month(3, 2024), 180);

TimeSeriesCollection dataset = new TimeSeriesCollection();
dataset.addSeries(series);

JFreeChart chart = ChartFactory.createTimeSeriesChart(
    "Monthly Sales",
    "Date",
    "Sales",
    dataset,
    true,    // legend
    false,   // tooltips
    false);  // no URLs
```

让我们看看结果：

![](/assets/images/2025/libraries/jfreechartvisualizedata04.png)

此示例展示了JFreeChart在绘制时序数据方面的强大功能，使其能够轻松跟踪随时间的变化。

### 3.4 组合图

**组合图表允许我们将不同类型的图表组合成一个图表**，与前面的示例相比，代码稍微复杂一些。

我们需要使用DefaultCategoryDataset来存储数据，但在这里，我们创建两个实例，每个图表类型一个：

```java
DefaultCategoryDataset lineDataset = new DefaultCategoryDataset();
lineDataset.addValue(200, "Sales", "January");
lineDataset.addValue(150, "Sales", "February");
lineDataset.addValue(180, "Sales", "March");

DefaultCategoryDataset barDataset = new DefaultCategoryDataset();
barDataset.addValue(400, "Profit", "January");
barDataset.addValue(300, "Profit", "February");
barDataset.addValue(250, "Profit", "March");
```

[CategoryPlot](https://www.jfree.org/jfreechart/javadoc/org/jfree/chart/plot/CategoryPlot.html)创建包含两种图表类型的绘图区，它允许我们将数据集分配给渲染器-[LineAndShapeRenderer](https://www.jfree.org/jfreechart/javadoc/org/jfree/chart/renderer/category/LineAndShapeRenderer.html)用于线条，[BarRenderer](https://www.jfree.org/jfreechart/javadoc/org/jfree/chart/renderer/category/BarRenderer.html)用于条形：

```java
CategoryPlot plot = new CategoryPlot();
plot.setDataset(0, lineDataset);
plot.setRenderer(0, new LineAndShapeRenderer());

plot.setDataset(1, barDataset);
plot.setRenderer(1, new BarRenderer());

plot.setDomainAxis(new CategoryAxis("Month"));
plot.setRangeAxis(new NumberAxis("Value"));

plot.setOrientation(PlotOrientation.VERTICAL);
plot.setRangeGridlinesVisible(true);
plot.setDomainGridlinesVisible(true);
```

最后，让我们使用JFreeChart创建最终的图表：

```java
JFreeChart chart = new JFreeChart(
    "Monthly Sales and Profit",
    null,  // null means to use default font
    plot,  // combination chart as CategoryPlot
    true); // legend
```

此设置允许组合呈现数据，以直观的方式展示销售额和利润之间的协同作用：

![](/assets/images/2025/libraries/jfreechartvisualizedata05.png)

这样，JFreeChart可以呈现需要多种图表类型的复杂数据集，以便更好地理解和分析。

## 4. 总结

在本文中，我们探讨了使用JFreeChart创建不同类型的图表，包括折线图、条形图、饼图、时序图和组合图。

本介绍仅仅触及了JFreeChart功能的皮毛。