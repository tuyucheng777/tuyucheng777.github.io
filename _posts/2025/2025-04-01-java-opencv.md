---
layout: post
title:  使用Java介绍OpenCV
category: libraries
copyright: libraries
excerpt: OpenCV
---

## 1. 简介

在本教程中，我们将学习**如何安装和使用OpenCV[计算机视觉](https://www.baeldung.com/cs/computer-vision)库并将其应用于实时[人脸检测](https://www.baeldung.com/cs/cv-face-recognition-mechanism)**。

## 2. 安装

为了在我们的项目中使用OpenCV库，我们需要将[opencv](https://mvnrepository.com/artifact/org.openpnp/opencv) Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.openpnp</groupId>
    <artifactId>opencv</artifactId>
    <version>4.9.0-0</version>
</dependency>
```

对于Gradle用户，我们需要将依赖添加到build.gradle文件中：

```groovy
compile group: 'org.openpnp', name: 'opencv', version: '3.4.2-0'
```

添加依赖后，我们可以使用OpenCV提供的功能。

## 3. 使用库

要开始使用OpenCV，我们需要初始化库，我们可以在main方法中执行此操作：

```java
OpenCV.loadShared();
```

**OpenCV是一个类，它包含与加载各种平台和架构的OpenCV库所需的原生包相关的方法**。

值得注意的是，[文档](https://opencv-java-tutorials.readthedocs.io/)中的说法略有不同：

```java
System.loadLibrary(Core.NATIVE_LIBRARY_NAME)
```

这两种方法调用实际上都会加载所需的原生库。

此处的区别在于后者需要安装原生库，而前者则可以将库安装到临时文件夹中(如果给定计算机上没有库)。由于这种差异，loadShared方法通常是最佳选择。

或者，我们可以按照[官方文档](https://opencv-java-tutorials.readthedocs.io/en/latest/01-installing-opencv-for-java.html)下载并编译该库，并在使用OpenCV相关类之前加载该库：

```java
System.load("/opencv/build/lib/libopencv_java4100.dylib");
```

现在我们已经初始化了库，让我们看看可以用它做什么。

## 4. 加载图像

首先，**让我们使用OpenCV从磁盘加载示例图像**：

```java
public static Mat loadImage(String imagePath) {
    return new Imgcodecs(imagePath);
}
```

**此方法将给定的图像加载为Mat对象，它是一个矩阵表示**。

要保存之前加载的图像，我们可以使用Imgcodecs类的imwrite()方法：

```java
public static void saveImage(Mat imageMatrix, String targetPath) {
    Imgcodecs.imwrite(targetPath, imageMatrix);
}
```

## 5. Haar级联分类器

在深入研究面部识别之前，让我们先了解实现这一目标的核心概念。

简而言之，**分类器是一种程序，它试图根据经验将新的观察结果归入一个组**。级联分类器试图使用多个分类器的串联来实现这一点，每个后续分类器都使用前一个分类器的输出作为附加信息，从而显著改善分类效果。

### 5.1 Haar特征

在我们的例子中，我们将使用基于Haar特征的级联分类器在OpenCV中进行人脸检测。

**[Haar特征](https://docs.opencv.org/3.4/db/d28/tutorial_cascade_classifier.html)是用于检测图像上的边缘和线条的过滤器**，过滤器看起来像是黑色和白色的方块：

![](/assets/images/2025/libraries/javaopencv01.png)

这些过滤器会多次应用于图像，逐个像素地应用，并将结果收集为单个值。该值是黑色方块下像素总和与白色方块下像素总和之间的差值。

## 6. 人脸检测

一般来说，**级联分类器需要经过预先训练才能检测到任何东西**。

由于训练过程可能很长并且需要大量数据集，我们将使用OpenCV提供的[预训练模型](https://github.com/opencv/opencv/tree/master/data/haarcascades)之一，我们将此XML文件放在resources文件夹中以方便访问。

**让我们逐步了解一下检测人脸的过程**：

![](/assets/images/2025/libraries/javaopencv02.png)

我们将尝试通过使用红色矩形勾勒出脸部轮廓来检测脸部。

首先，我们需要从源路径加载Mat格式的图像：

```java
Mat loadedImage = loadImage(sourceImagePath);
```

然后，我们将声明一个MatOfRect对象来存储我们找到的脸：

```java
MatOfRect facesDetected = new MatOfRect();
```

接下来我们需要初始化CascadeClassifier来进行识别：

```java
CascadeClassifier cascadeClassifier = new CascadeClassifier(); 
int minFaceSize = Math.round(loadedImage.rows() * 0.1f); 
String filename = FaceDetection.class.getClassLoader().getResource("haarcascades/haarcascade_frontalface_alt.xml").getFile();
cascadeClassifier.load(filename); 
cascadeClassifier.detectMultiScale(loadedImage, 
    facesDetected, 
    1.1, 
    3, 
    Objdetect.CASCADE_SCALE_IMAGE, 
    new Size(minFaceSize, minFaceSize), 
    new Size() 
);
```

上面的参数1.1表示我们要使用的比例因子，指定在每个图像比例上图像大小缩小多少。下面的参数3是minNeighbors，这是候选矩形应具有的邻居数量，以便保留它。

最后，我们循环遍历所有脸并保存结果：

```java
Rect[] facesArray = facesDetected.toArray(); 
for(Rect face : facesArray) { 
    Imgproc.rectangle(loadedImage, face.tl(), face.br(), new Scalar(0, 0, 255), 10); 
} 
saveImage(loadedImage, targetImagePath);
```

当我们输入源图像时，我们现在应该收到所有面部都用红色矩形标记的输出图像。

这简要描述了我们的detectFace()方法的内容，让我们使用它来测试一切是否正常工作：

```java
public static void main(String[] args) {
    // Load the native library.
    System.load("/opencv/build/lib/libopencv_java4100.dylib");
    detectFace(Paths.get("portrait.jpg"),"./processed.jpg");
}
```

简而言之，我们将加载一张图片，并使用输入文件路径和输出文件路径调用detectFace()方法。结果，我们将得到一个名为processed.jpg的文件，其中包含一个围绕图片中人物脸部的红色矩形：

![](/assets/images/2025/libraries/javaopencv03.png)

## 7. 使用OpenCV访问相机

到目前为止，我们已经了解了如何在加载的图像上执行人脸检测。但大多数时候，我们希望实时执行此操作，为此，我们需要访问相机。

但是，要显示来自相机的图像，除了显而易见的东西(相机)之外，我们还需要一些东西。为了显示图像，我们将使用JavaFX。

因为我们将使用ImageView来显示相机拍摄的照片，所以我们需要一种方法将OpenCV Mat转换为JavaFX Image：

```java
public Image mat2Img(Mat mat) {
    MatOfByte bytes = new MatOfByte();
    Imgcodecs.imencode("img", mat, bytes);
    InputStream inputStream = new ByteArrayInputStream(bytes.toArray());
    return new Image(inputStream);
}
```

在这里，我们将Mat转换为字节，然后将字节转换为Image对象。

我们首先将摄像机视图传输到JavaFX Stage。

现在，让我们使用loadShared方法初始化库：

```java
OpenCV.loadShared();
```

接下来，我们将创建带有VideoCapture和ImageView的舞台来显示图像：

```java
VideoCapture capture = new VideoCapture(0); 
ImageView imageView = new ImageView(); 
HBox hbox = new HBox(imageView); 
Scene scene = new Scene(hbox);
stage.setScene(scene); 
stage.show();
```

这里，0是我们要使用的相机的ID。我们还需要创建一个AnimationTimer来处理图像的设置：

```java
new AnimationTimer() { 
    @Override public void handle(long l) { 
        imageView.setImage(getCapture()); 
    } 
}.start();
```

最后，我们的getCapture方法负责将Mat转换为Image：

```java
public Image getCapture() { 
    Mat mat = new Mat(); 
    capture.read(mat); 
    return mat2Img(mat); 
}
```

**应用程序现在应该创建一个窗口，然后将视图从摄像头实时传输到imageView窗口**。

## 8. 实时人脸检测

最后，我们可以将所有点连接起来，创建一个可以实时检测人脸的应用程序。

上一节中的代码负责从相机抓取图像并将其显示给用户。现在，我们要做的就是使用CascadeClassifier类处理抓取的图像，然后将其显示在屏幕上。

**让我们修改getCapture方法来执行人脸检测**：

```java
public Image getCaptureWithFaceDetection() {
    Mat mat = new Mat();
    capture.read(mat);
    Mat haarClassifiedImg = detectFace(mat);
    return mat2Img(haarClassifiedImg);
}
```

现在，如果运行我们的应用程序，脸部应该被一个红色矩形标记。

我们还可以看到级联分类器的一个缺点，如果我们将脸部向任何方向转动太多，红色矩形就会消失。**这是因为我们使用了一个专门训练的分类器，它只用于检测脸部的正面**。

## 9. 总结

在本教程中，我们学习了如何在Java中使用OpenCV。

我们使用预先训练的级联分类器来检测图像上的人脸，借助JavaFX，我们让分类器使用来自相机的图像实时检测人脸。