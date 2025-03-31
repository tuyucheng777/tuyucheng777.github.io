---
layout: post
title:  如何使用Deeplearning4j实现CNN
category: libraries
copyright: libraries
excerpt: Deeplearning4j
---

## 1. 概述

在本教程中，我们将使用Java中的Deeplearning4j库构建和训练卷积神经网络模型。

有关如何设置库的更多信息，请参阅我们的[Deeplearning4j指南](https://www.baeldung.com/deeplearning4j)。

## 2. 图像分类

### 2.1 问题陈述

假设我们有一组图像，每幅图像代表一个特定类别的对象。此外，图像上的对象属于唯一已知的类别。因此，**问题陈述是构建一个能够识别给定图像上对象类别的模型**。

例如，假设我们有一组包含10个手势的图像，我们建立一个模型并训练它对它们进行分类。然后在训练之后，我们可以传递其他图像并对它们上的手势进行分类。当然，给定的手势应该属于已知的类别。

### 2.2 图像表示

在计算机内存中，图像可以表示为数字矩阵，每个数字都是一个像素值，范围从0到255。

灰度图像是二维矩阵。同样，RGB图像是具有宽度、高度和深度维度的三维矩阵。

我们看到，图像是一组数字，因此，我们可以建立多层网络模型，训练它们对图像进行分类。

## 3. 卷积神经网络

卷积神经网络(CNN)是一种具有特定结构的多层网络模型，**CNN的结构可分为两部分：卷积层和全连接层(或密集层)**，让我们分别看一下。

### 3.1 卷积层

**每个卷积层都是一组方阵，称为核**。首先，我们需要它们对输入图像进行卷积，它们的数量和大小可能有所不同，具体取决于给定的数据集。我们大多使用3 × 3或5 × 5核，很少使用7 × 7核。确切的大小和数量是通过反复试验选择的。

另外，我们在训练开始时随机选择核矩阵的变量，它们是网络的权重。

为了执行卷积，我们可以使用核作为滑动窗口。我们将核权重乘以相应的图像像素并计算总和，然后我们可以使用步幅(向右移动)和填充(向下移动)移动核以覆盖图像的下一个块。结果，我们将获得将用于进一步计算的值。

简而言之，通过这一层，我们得到了一个卷积图像。一些变量可能小于0，这通常意味着这些变量不如其他变量重要，这就是为什么应用[ReLU](https://en.wikipedia.org/wiki/Rectifier_(neural_networks))函数是一种减少计算的好方法。

### 3.2 子采样层

子采样(或池化)层是网络的一层，通常用在卷积层之后。**卷积之后，我们得到很多计算变量。然而，我们的任务是从中选择最有价值的变量**。

该方法是对卷积图像应用[滑动窗口算法](https://www.baeldung.com/cs/sliding-window-algorithm)。在每个步骤中，我们将在预定义大小的方形窗口中选择最大值，该窗口通常介于2 × 2和5 × 5像素之间。因此，我们需要计算的参数会更少。因此，这将减少计算量。

### 3.3 密集层

密集层(或全连接层)由多个神经元组成，我们需要这一层来执行分类。此外，可能有两个或更多这样的后续层。重要的是，最后一层的大小应等于分类的类别数。

**网络的输出是图像属于每个类别的概率**，为了预测概率，我们将使用[Softmax](https://en.wikipedia.org/wiki/Softmax_function)激活函数。

### 3.4 优化技术

为了进行训练，我们需要优化权重。记住，我们最初随机选择这些变量。神经网络是一个很大的函数，而且，它有很多未知参数，也就是我们的权重。

**当我们将图像传递给网络时，它会给出答案**。然后，我们可以**建立一个损失函数，它将取决于这个答案**。在监督学习方面，我们还有一个实际的答案-真正的类别。我们的任务是最小化这个损失函数，如果我们成功了，那么我们的模型就是训练有素的。

**为了最小化函数，我们必须更新网络的权重**。为了做到这一点，我们可以计算损失函数对每个未知参数的导数，然后，我们可以更新每个权重。

由于我们知道斜率，我们可以增加或减少权重值来找到损失函数的局部最小值。**此外，这个过程是迭代的，称为[梯度下降](https://www.baeldung.com/java-gradient-descent)**。反向传播使用梯度下降将权重更新从网络的末端传播到开头。

在本教程中，我们将使用[随机梯度下降](https://en.wikipedia.org/wiki/Stochastic_gradient_descent)(SGD)优化算法。主要思想是我们在每一步随机选择一批训练图像，然后我们应用反向传播。

### 3.5 评估指标

最后，在训练网络之后，我们需要获取有关模型运行情况的信息。

**最常用的指标是准确率，这是正确分类的图像占所有图像的比例。同时，召回率、精确率和F1分数也是[图像分类非常重要的指标](https://medium.com/analytics-vidhya/confusion-matrix-accuracy-precision-recall-f1-score-ade299cf63cd)**。

## 4. 数据集准备

在本节中，我们将准备图像，让我们在本教程中使用嵌入的[CIFAR10](https://en.wikipedia.org/wiki/CIFAR-10)数据集。我们将创建迭代器来访问图像：

```java
public class CifarDatasetService implements IDataSetService {

    private CifarDataSetIterator trainIterator;
    private CifarDataSetIterator testIterator;

    public CifarDatasetService() {
        trainIterator = new CifarDataSetIterator(trainBatch, trainImagesNum, true);
        testIterator = new CifarDataSetIterator(testBatch, testImagesNum, false);
    }

    // other methods and fields declaration
}
```

我们可以自行选择一些参数。TrainBatch和testBatch分别是每次训练和评估步骤的图像数量，TrainImagesNum和testImagesNum是用于训练和测试的图像数量，**一个epoch持续trainImagesNum / trainBatch步骤**。因此，如果批次大小为32，则2048张训练图像将导致每个epoch有2048 / 32 = 64个步骤。

## 5. Deeplearning4j中的卷积神经网络

### 5.1 构建模型

接下来，让我们从头开始构建CNN模型。为此，**我们将使用卷积、子采样(池化)和全连接(密集)层**。

```java
MultiLayerConfiguration configuration = new NeuralNetConfiguration.Builder()
        .seed(1611)
        .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
        .learningRate(properties.getLearningRate())
        .regularization(true)
        .updater(properties.getOptimizer())
        .list()
        .layer(0, conv5x5())
        .layer(1, pooling2x2Stride2())
        .layer(2, conv3x3Stride1Padding2())
        .layer(3, pooling2x2Stride1())
        .layer(4, conv3x3Stride1Padding1())
        .layer(5, pooling2x2Stride1())
        .layer(6, dense())
        .pretrain(false)
        .backprop(true)
        .setInputType(dataSetService.inputType())
        .build();

network = new MultiLayerNetwork(configuration);
```

**在这里，我们指定学习率、更新算法、模型的输入类型和分层架构**。我们可以对这些配置进行实验，因此，我们可以训练具有不同架构和训练参数的许多模型。此外，我们可以比较结果并选择最佳模型。

### 5.2 训练模型

然后，我们将训练构建的模型，这可以用几行代码完成：

```java
public void train() {
    network.init();
    IntStream.range(1, epochsNum + 1).forEach(epoch -> network.fit(dataSetService.trainIterator()));
}
```

**epoch的数量是我们可以自己指定的参数**。我们的数据集很小，因此，几百个epoch就足够了。

### 5.3 评估模型

最后，我们可以评估现在训练好的模型，Deeplearning4j库提供了轻松完成此操作的功能：

```java
public Evaluation evaluate() {
   return network.evaluate(dataSetService.testIterator());
}
```

Evaluation是一个对象，其中包含训练模型后计算出的指标。这些指标包括准确率、精确率、召回率和F1分数。此外，它还有一个友好的可打印界面：

```text
==========================Scores=====================
# of classes: 11
Accuracy: 0,8406
Precision: 0,7303
Recall: 0,6820
F1 Score: 0,6466
=====================================================
```

## 6. 总结

在本教程中，我们了解了CNN模型的架构、优化技术和评估指标。此外，我们还使用Java中的Deeplearning4j库实现了该模型。