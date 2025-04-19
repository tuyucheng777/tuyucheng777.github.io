---
layout: post
title:  Java中的矩阵乘法
category: algorithms
copyright: algorithms
excerpt: 矩阵
---

## 1. 概述

在本教程中，我们将了解如何在Java中将两个矩阵相乘。

由于矩阵概念在语言中并不存在，我们将自己实现它，并且我们还将使用一些库来了解它们如何处理矩阵乘法。

最后，我们将对所探索的不同解决方案进行一些基准测试，以确定最快的解决方案。

## 2. 示例

让我们首先建立一个可以在本教程中参考的示例。

首先，我们想象一个3 × 2矩阵：

![](/assets/images/2025/algorithms/javamatrixmultiplication01.png)

现在让我们想象第二个矩阵，这次是2 * 4：

![](/assets/images/2025/algorithms/javamatrixmultiplication02.png)

然后，将第一个矩阵乘以第二个矩阵，得到一个3 × 4矩阵：

![](/assets/images/2025/algorithms/javamatrixmultiplication03.png)

提醒一下，**这个结果是通过使用以下公式计算结果矩阵的每个单元格获得的**：

![](/assets/images/2025/algorithms/javamatrixmultiplication04.png)

其中r是矩阵A的行数，c是矩阵B的列数，n是矩阵A的列数，必须与矩阵B的行数匹配。

## 3. 矩阵乘法

### 3.1 手动实现

让我们从我们自己的矩阵实现开始。

我们将保持简单并仅使用二维double数组：

```java
double[][] firstMatrix = {
        new double[]{1d, 5d},
        new double[]{2d, 3d},
        new double[]{1d, 7d}
};

double[][] secondMatrix = {
        new double[]{1d, 2d, 3d, 7d},
        new double[]{5d, 2d, 8d, 1d}
};
```

以上就是我们示例中的两个矩阵，让我们创建一个期望的矩阵作为它们相乘的结果：

```java
double[][] expected = {
        new double[]{26d, 12d, 43d, 12d},
        new double[]{17d, 10d, 30d, 17d},
        new double[]{36d, 16d, 59d, 14d}
};
```

现在一切都已设置完毕，让我们实现乘法算法。**首先创建一个空的结果数组，并遍历其单元格，将预期值存储在每个单元格中**：

```java
double[][] multiplyMatrices(double[][] firstMatrix, double[][] secondMatrix) {
    double[][] result = new double[firstMatrix.length][secondMatrix[0].length];

    for (int row = 0; row < result.length; row++) {
        for (int col = 0; col < result[row].length; col++) {
            result[row][col] = multiplyMatricesCell(firstMatrix, secondMatrix, row, col);
        }
    }

    return result;
}
```

最后，让我们实现单个单元格的计算。为了实现这一点，**我们将使用前面示例演示中显示的公式**：

```java
double multiplyMatricesCell(double[][] firstMatrix, double[][] secondMatrix, int row, int col) {
    double cell = 0;
    for (int i = 0; i < secondMatrix.length; i++) {
        cell += firstMatrix[row][i] * secondMatrix[i][col];
    }
    return cell;
}
```

最后，我们来检查一下算法的结果是否符合我们的预期结果：

```java
double[][] actual = multiplyMatrices(firstMatrix, secondMatrix);
assertThat(actual).isEqualTo(expected);
```

### 3.2 EJML

我们要看的第一个库是EJML，即[Efficient Java Matrix Library](http://ejml.org/wiki/index.php?title=Main_Page)。在撰写本教程时，**它是最新更新的Java矩阵库之一**，它的目的是尽可能提高计算和内存使用的效率。

我们必须在pom.xml中添加[依赖](https://mvnrepository.com/artifact/org.ejml/ejml-all)：

```xml
<dependency>
    <groupId>org.ejml</groupId>
    <artifactId>ejml-all</artifactId>
    <version>0.38</version>
</dependency>
```

我们将使用与以前几乎相同的模式：根据我们的例子创建两个矩阵，并检查它们相乘的结果是否是我们之前计算的结果。

那么，让我们使用EJML创建矩阵。为了实现这一点，**我们将使用库提供的SimpleMatrix类**。

它可以将二维double数组作为其构造函数的输入：

```java
SimpleMatrix firstMatrix = new SimpleMatrix(
        new double[][] {
                new double[] {1d, 5d},
                new double[] {2d, 3d},
                new double[] {1d ,7d}
        }
);

SimpleMatrix secondMatrix = new SimpleMatrix(
        new double[][] {
                new double[] {1d, 2d, 3d, 7d},
                new double[] {5d, 2d, 8d, 1d}
        }
);
```

现在，让我们定义乘法的预期矩阵：

```java
SimpleMatrix expected = new SimpleMatrix(
        new double[][] {
                new double[] {26d, 12d, 43d, 12d},
                new double[] {17d, 10d, 30d, 17d},
                new double[] {36d, 16d, 59d, 14d}
        }
);
```

现在一切准备就绪，让我们看看如何将两个矩阵相乘。**SimpleMatrix类提供了一个mult()方法**，该方法接收另一个SimpleMatrix作为参数，并返回两个矩阵的乘积：

```java
SimpleMatrix actual = firstMatrix.mult(secondMatrix);
```

让我们检查一下获得的结果是否与预期相符。

由于SimpleMatrix没有重写equals()方法，因此我们不能依赖它来进行验证。不过，它提供了一种替代方案：isIdentical()方法，该方法不仅接收另一个矩阵参数，还接收一个double容错参数，以忽略由于双精度导致的细微差异：

```java
assertThat(actual).matches(m -> m.isIdentical(expected, 0d));
```

以上就是使用EJML库进行矩阵乘法的介绍，让我们看看其他库提供了哪些功能。

### 3.3 ND4J

现在让我们尝试一下[ND4J库](https://deeplearning4j.konduit.ai/nd4j/tutorials/quickstart)，ND4J是一个计算库，是[deeplearning4j](https://deeplearning4j.konduit.ai/)项目的一部分。此外，ND4J还提供矩阵计算功能。

首先，我们必须定义[依赖](https://mvnrepository.com/artifact/org.nd4j/nd4j-native)：

```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>1.0.0-beta4</version>
</dependency>
```

请注意，我们在这里使用的是测试版，因为GA版本似乎存在一些错误。

为了简洁起见，我们不会重写二维double数组，而只关注它们在每个库中的使用方式。因此，使用ND4J，我们必须创建一个INDArray。为此，**我们将调用Nd4j.create()工厂方法，并向其传递一个表示矩阵的double数组**：

```java
INDArray matrix = Nd4j.create(/* a two dimensions double array */);
```

与上一节一样，我们将创建三个矩阵：两个矩阵用于相乘，一个矩阵是预期结果。

之后，我们想要使用INDArray.mmul()方法实际执行前两个矩阵之间的乘法：

```java
INDArray actual = firstMatrix.mmul(secondMatrix);
```

然后，我们再次检查实际结果是否与预期结果相符，这次我们可以依靠相等性检查：

```java
assertThat(actual).isEqualTo(expected);
```

这演示了如何使用ND4J库进行矩阵计算。

### 3.4 Apache Commons

现在让我们来讨论一下[Apache Commons Math3](https://commons.apache.org/proper/commons-math/)模块，它为我们提供了包括矩阵操作在内的数学计算。

再次，我们必须在pom.xml中指定[依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-math3)：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

设置完成后，**我们可以使用RealMatrix接口及其Array2DRowRealMatrix实现来创建常用矩阵**，该实现类的构造函数以一个二维double数组作为参数：

```java
RealMatrix matrix = new Array2DRowRealMatrix(/* a two dimensions double array */);
```

对于矩阵乘法，**RealMatrix接口提供了一个接收另一个RealMatrix参数的multiply()方法**：

```java
RealMatrix actual = firstMatrix.multiply(secondMatrix);
```

我们最终可以验证结果是否符合我们的预期：

```java
assertThat(actual).isEqualTo(expected);
```

### 3.5 LA4J

这个叫做LA4J，代表[Linear Algebra for Java](http://la4j.org/)。

让我们也添加[依赖](https://mvnrepository.com/artifact/org.la4j/la4j)：

```xml
<dependency>
    <groupId>org.la4j</groupId>
    <artifactId>la4j</artifactId>
    <version>0.6.0</version>
</dependency>
```

现在，LA4J的工作方式与其他库非常相似，**它提供了一个Matrix接口，其中包含一个Basic2DMatrix实现**，该实现接收二维double数组作为输入：

```java
Matrix matrix = new Basic2DMatrix(/* a two dimensions double array */);
```

与Apache Commons Math3模块一样，乘法方法是multiply()并将另一个矩阵作为其参数：

```java
Matrix actual = firstMatrix.multiply(secondMatrix);
```

再次检查结果是否符合我们的预期：

```java
assertThat(actual).isEqualTo(expected);
```

### 3.6 Colt

[Colt](https://dst.lbl.gov/ACSSoftware/colt/)是由CERN开发的一个库，它提供了支持高性能科学和技术计算的功能。

与以前的库一样，我们必须定义正确的[依赖](https://mvnrepository.com/artifact/colt/colt)：

```xml
<dependency>
    <groupId>colt</groupId>
    <artifactId>colt</artifactId>
    <version>1.2.0</version>
</dependency>
```

为了使用Colt创建矩阵，**我们必须使用DoubleFactory2D类**。它带有3个工厂实例：dense、sparse和rowCompressed，每个实例都经过优化，以创建匹配类型的矩阵。

为了达到我们的目的，我们将使用dense实例。这次，要调用的方法是make()，它再次接收一个二维double数组作为参数，并生成一个DoubleMatrix2D对象：

```java
DoubleMatrix2D matrix = doubleFactory2D.make(/* a two dimensions double array */);
```

矩阵实例化后，我们需要将它们相乘。这次，矩阵对象上没有方法可以做到这一点。**我们必须创建一个Algebra类的实例**，该类有一个mult()方法，接收两个矩阵作为参数：

```java
Algebra algebra = new Algebra();
DoubleMatrix2D actual = algebra.mult(firstMatrix, secondMatrix);
```

然后，我们可以将实际结果与预期结果进行比较：

```java
assertThat(actual).isEqualTo(expected);
```

## 4. 基准测试

现在我们已经完成了对矩阵乘法的不同可能性的探索，让我们检查一下哪种方法性能最好。

### 4.1 小矩阵

让我们从小矩阵开始，这里是一个3 × 2矩阵和一个2 × 4矩阵。

**为了实现性能测试，我们将使用[JMH基准测试库](https://www.baeldung.com/java-microbenchmark-harness)**，让我们使用以下选项配置一个基准测试类：

```java
public static void main(String[] args) throws Exception {
    Options opt = new OptionsBuilder()
            .include(MatrixMultiplicationBenchmarking.class.getSimpleName())
            .mode(Mode.AverageTime)
            .forks(2)
            .warmupIterations(5)
            .measurementIterations(10)
            .timeUnit(TimeUnit.MICROSECONDS)
            .build();

    new Runner(opt).run();
}
```

这样，JMH将对每个带有@Benchmark注解的方法进行两次完整运行，每次运行包含5次预热迭代(不计入平均计算)和10次测量迭代。至于测量，它将收集不同库的平均执行时间(以微秒为单位)。

然后我们必须创建一个包含数组的状态对象：

```java
@State(Scope.Benchmark)
public class MatrixProvider {
    private double[][] firstMatrix;
    private double[][] secondMatrix;

    public MatrixProvider() {
        firstMatrix =
                new double[][] {
                        new double[] {1d, 5d},
                        new double[] {2d, 3d},
                        new double[] {1d ,7d}
                };

        secondMatrix =
                new double[][] {
                        new double[] {1d, 2d, 3d, 7d},
                        new double[] {5d, 2d, 8d, 1d}
                };
    }
}
```

这样，我们确保数组初始化不包含在基准测试中。之后，我们仍然需要创建执行矩阵乘法的方法，并使用MatrixProvider对象作为数据源。由于我们之前已经了解过每个库的具体内容，因此这里不再赘述。

最后，我们将使用main方法运行基准测试过程。这将给出以下结果：

```text
Benchmark                                                           Mode  Cnt   Score   Error  Units
MatrixMultiplicationBenchmarking.apacheCommonsMatrixMultiplication  avgt   20   1,008 ± 0,032  us/op
MatrixMultiplicationBenchmarking.coltMatrixMultiplication           avgt   20   0,219 ± 0,014  us/op
MatrixMultiplicationBenchmarking.ejmlMatrixMultiplication           avgt   20   0,226 ± 0,013  us/op
MatrixMultiplicationBenchmarking.homemadeMatrixMultiplication       avgt   20   0,389 ± 0,045  us/op
MatrixMultiplicationBenchmarking.la4jMatrixMultiplication           avgt   20   0,427 ± 0,016  us/op
MatrixMultiplicationBenchmarking.nd4jMatrixMultiplication           avgt   20  12,670 ± 2,582  us/op
```

我们可以看到，**EJML和Colt的性能表现非常出色，每次操作大约需要五分之一微秒，而ND4j的性能稍差一些，每次操作需要十多微秒**，其他库的性能介于两者之间。

另外，值得注意的是，当将预热迭代次数从5次增加到10次时，所有库的性能都会提高。

### 4.2 大矩阵

现在，如果我们计算更大的矩阵，比如3000 × 3000，会发生什么？为了检查会发生什么，我们首先创建另一个状态类，提供该大小的生成矩阵：

```java
@State(Scope.Benchmark)
public class BigMatrixProvider {
    private double[][] firstMatrix;
    private double[][] secondMatrix;

    public BigMatrixProvider() {}

    @Setup
    public void setup(BenchmarkParams parameters) {
        firstMatrix = createMatrix();
        secondMatrix = createMatrix();
    }

    private double[][] createMatrix() {
        Random random = new Random();

        double[][] result = new double[3000][3000];
        for (int row = 0; row < result.length; row++) {
            for (int col = 0; col < result[row].length; col++) {
                result[row][col] = random.nextDouble();
            }
        }
        return result;
    }
}
```

如我们所见，我们将创建3000 × 3000个二维double数组，其中填充随机实数。

现在让我们创建基准测试类：

```java
public class BigMatrixMultiplicationBenchmarking {
    public static void main(String[] args) throws Exception {
        Map<String, String> parameters = parseParameters(args);

        ChainedOptionsBuilder builder = new OptionsBuilder()
                .include(BigMatrixMultiplicationBenchmarking.class.getSimpleName())
                .mode(Mode.AverageTime)
                .forks(2)
                .warmupIterations(10)
                .measurementIterations(10)
                .timeUnit(TimeUnit.SECONDS);

        new Runner(builder.build()).run();
    }

    @Benchmark
    public Object homemadeMatrixMultiplication(BigMatrixProvider matrixProvider) {
        return HomemadeMatrix
                .multiplyMatrices(matrixProvider.getFirstMatrix(), matrixProvider.getSecondMatrix());
    }

    @Benchmark
    public Object ejmlMatrixMultiplication(BigMatrixProvider matrixProvider) {
        SimpleMatrix firstMatrix = new SimpleMatrix(matrixProvider.getFirstMatrix());
        SimpleMatrix secondMatrix = new SimpleMatrix(matrixProvider.getSecondMatrix());

        return firstMatrix.mult(secondMatrix);
    }

    @Benchmark
    public Object apacheCommonsMatrixMultiplication(BigMatrixProvider matrixProvider) {
        RealMatrix firstMatrix = new Array2DRowRealMatrix(matrixProvider.getFirstMatrix());
        RealMatrix secondMatrix = new Array2DRowRealMatrix(matrixProvider.getSecondMatrix());

        return firstMatrix.multiply(secondMatrix);
    }

    @Benchmark
    public Object la4jMatrixMultiplication(BigMatrixProvider matrixProvider) {
        Matrix firstMatrix = new Basic2DMatrix(matrixProvider.getFirstMatrix());
        Matrix secondMatrix = new Basic2DMatrix(matrixProvider.getSecondMatrix());

        return firstMatrix.multiply(secondMatrix);
    }

    @Benchmark
    public Object nd4jMatrixMultiplication(BigMatrixProvider matrixProvider) {
        INDArray firstMatrix = Nd4j.create(matrixProvider.getFirstMatrix());
        INDArray secondMatrix = Nd4j.create(matrixProvider.getSecondMatrix());

        return firstMatrix.mmul(secondMatrix);
    }

    @Benchmark
    public Object coltMatrixMultiplication(BigMatrixProvider matrixProvider) {
        DoubleFactory2D doubleFactory2D = DoubleFactory2D.dense;

        DoubleMatrix2D firstMatrix = doubleFactory2D.make(matrixProvider.getFirstMatrix());
        DoubleMatrix2D secondMatrix = doubleFactory2D.make(matrixProvider.getSecondMatrix());

        Algebra algebra = new Algebra();
        return algebra.mult(firstMatrix, secondMatrix);
    }
}
```

当我们运行这个基准测试时，我们得到了完全不同的结果：

```text
Benchmark                                                              Mode  Cnt    Score    Error  Units
BigMatrixMultiplicationBenchmarking.apacheCommonsMatrixMultiplication  avgt   20  511.140 ± 13.535   s/op
BigMatrixMultiplicationBenchmarking.coltMatrixMultiplication           avgt   20  197.914 ±  2.453   s/op
BigMatrixMultiplicationBenchmarking.ejmlMatrixMultiplication           avgt   20   25.830 ±  0.059   s/op
BigMatrixMultiplicationBenchmarking.homemadeMatrixMultiplication       avgt   20  497.493 ±  2.121   s/op
BigMatrixMultiplicationBenchmarking.la4jMatrixMultiplication           avgt   20   35.523 ±  0.102   s/op
BigMatrixMultiplicationBenchmarking.nd4jMatrixMultiplication           avgt   20    0.548 ±  0.006   s/op
```

我们可以看到，自定义的实现和Apache库现在比以前差多了，需要将近10分钟才能完成两个矩阵的乘法。

Colt耗时略长于3分钟，略有改善，但仍然很长。EJML和LA4J的表现相当不错，运行时间接近30秒。**不过，ND4J在这次基准测试中胜出，在[CPU后](https://deeplearning4j.konduit.ai/v/en-1.0.0-beta7/config/backends/performance-issues)端的测试中，其运行时间不到一秒**。

## 5. 总结

在本文中，我们学习了如何在Java中执行矩阵乘法，无论是自行编写还是使用外部库。在探索了所有解决方案之后，我们对所有方案进行了基准测试，发现除ND4J外，其他方案在小型矩阵上的表现都相当出色。另一方面，在大型矩阵上，ND4J则占据领先地位。