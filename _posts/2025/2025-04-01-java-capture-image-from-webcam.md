---
layout: post
title:  使用Java从网络摄像头捕获图像
category: libraries
copyright: libraries
excerpt: JavaCV、Marvin
---

## 1. 概述

通常，Java无法轻松访问计算机硬件，这就是为什么我们可能很难使用Java访问网络摄像头。

在本教程中，我们将探索一些允许我们通过访问网络摄像头来捕获图像的Java库。

## 2. JavaCV

首先，我们来看一下[javacv](https://github.com/bytedeco/javacv)库。这是[Bytedeco](http://bytedeco.org/)的[OpenCV](https://www.baeldung.com/java-opencv)计算机视觉库的Java实现。

让我们将最新的[javacv-platform](https://mvnrepository.com/artifact/org.bytedeco/javacv-platform) Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv-platform</artifactId>
    <version>1.5.5</version>
</dependency>
```

类似地，在使用Gradle时，我们可以在build.gradle文件中添加javacv-platform依赖：

```groovy
compile group: 'org.bytedeco', name: 'javacv-platform', version: '1.5.5'
```

现在我们已经完成设置，让我们**使用[OpenCVFrameGrabber](http://bytedeco.org/javacv/apidocs/org/bytedeco/javacv/OpenCVFrameGrabber.html)类来访问网络摄像头并捕获帧**：

```java
FrameGrabber grabber = new OpenCVFrameGrabber(0);
grabber.start();
Frame frame = grabber.grab();
```

这里，**我们传递了设备编号0，指向系统默认的网络摄像头**。但是，如果我们有多个可用的摄像头，那么第二个摄像头的编号为1，第三个摄像头的编号为2，依此类推。

然后，我们可以使用OpenCVFrameConverter将捕获的帧转换为图像。此外，我们将**使用opencv_imgcodecs类的cvSaveImage方法保存图像**：

```java
OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();
IplImage img = converter.convert(frame);
opencv_imgcodecs.cvSaveImage("selfie.jpg", img);
```

最后，我们可以使用[CanvasFrame](http://bytedeco.org/javacv/apidocs/org/bytedeco/javacv/CanvasFrame.html)类来显示捕获的帧：

```java
CanvasFrame canvas = new CanvasFrame("Web Cam");
canvas.showImage(frame);
```

让我们研究一个完整的解决方案，该解决方案访问网络摄像头、捕获图像、在窗口中显示图像并在两秒后自动关闭窗口：

```java
CanvasFrame canvas = new CanvasFrame("Web Cam");
canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

FrameGrabber grabber = new OpenCVFrameGrabber(0);
OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();

grabber.start();
Frame frame = grabber.grab();

IplImage img = converter.convert(frame);
cvSaveImage("selfie.jpg", img);

canvas.showImage(frame);

Thread.sleep(2000);

canvas.dispatchEvent(new WindowEvent(canvas, WindowEvent.WINDOW_CLOSING));
```

## 3. webcam-capture

接下来，我们将研究通过支持多种捕获框架来允许使用网络摄像头的webcam-capture库。

首先，让我们将最新的[webcam-capture](https://mvnrepository.com/artifact/com.github.sarxos/webcam-capture) Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.github.sarxos</groupId>
    <artifactId>webcam-capture</artifactId>
    <version>0.3.12</version>
</dependency>
```

或者，我们可以在Gradle项目的build.gradle中添加webcam-capture：

```text
compile group: 'com.github.sarxos', name: 'webcam-capture', version: '0.3.12'
```

然后，我们来编写一个简单的示例，使用[Webcam](https://javadoc.io/static/com.github.sarxos/webcam-capture/0.3.12/com/github/sarxos/webcam/Webcam.html)类来捕获图像：

```java
Webcam webcam = Webcam.getDefault();
webcam.open();

BufferedImage image = webcam.getImage();

ImageIO.write(image, ImageUtils.FORMAT_JPG, new File("selfie.jpg"));
```

在这里，我们访问默认的网络摄像头来捕获图像，然后将图像保存到文件中。

或者，我们可以使用[WebcamUtils](https://javadoc.io/static/com.github.sarxos/webcam-capture/0.3.12/com/github/sarxos/webcam/WebcamUtils.html)类来捕获图像：

```java
WebcamUtils.capture(webcam, "selfie.jpg");
```

另外，**我们可以使用[WebcamPanel](https://javadoc.io/static/com.github.sarxos/webcam-capture/0.3.12/com/github/sarxos/webcam/WebcamPanel.html)类[在窗口中显示捕获的图像](https://www.baeldung.com/java-images#3-displaying-an-image)**：

```java
Webcam webcam = Webcam.getDefault();
webcam.setViewSize(WebcamResolution.VGA.getSize());

WebcamPanel panel = new WebcamPanel(webcam);
panel.setImageSizeDisplayed(true);

JFrame window = new JFrame("Webcam");
window.add(panel);
window.setResizable(true);
window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
window.pack();
window.setVisible(true);
```

在这里，**我们将VGA设置为网络摄像头的视图大小，创建了[JFrame](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/JFrame.html)对象，并将WebcamPanel组件添加到窗口中**。

## 4. Marvin框架

最后，我们将探索Marvin框架来访问网络摄像头并捕获图像。

与往常一样，我们将最新的[marvin](https://mvnrepository.com/artifact/com.github.downgoon/marvin)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.github.downgoon</groupId>
    <artifactId>marvin</artifactId>
    <version>1.5.5</version>
</dependency>
```

或者，对于Gradle项目，我们将在build.gradle文件中添加marvin依赖：

```groovy
compile group: 'com.github.downgoon', name: 'marvin', version: '1.5.5'
```

现在设置已准备就绪，让我们**使用[MarvinJavaCVAdapter](http://marvinproject.sourceforge.net/javadoc/marvin/video/MarvinJavaCVAdapter.html)类通过提供设备编号0来连接到默认网络摄像头**：

```java
MarvinVideoInterface videoAdapter = new MarvinJavaCVAdapter();
videoAdapter.connect(0);
```

接下来，我们可以使用getFrame方法来捕获帧，然后**使用[MarvinImageIO](http://marvinproject.sourceforge.net/javadoc/marvin/io/MarvinImageIO.html)类的saveImage方法保存图像**：

```java
MarvinImage image = videoAdapter.getFrame();
MarvinImageIO.saveImage(image, "selfie.jpg");
```

另外，我们可以使用[MarvinImagePanel](http://marvinproject.sourceforge.net/javadoc/marvin/gui/MarvinImagePanel.html)类在窗口中显示图像：

```java
MarvinImagePanel imagePanel = new MarvinImagePanel();
imagePanel.setImage(image);

imagePanel.setSize(800, 600);
imagePanel.setVisible(true);
```

## 5. 总结

在这篇短文中，我们研究了一些可轻松访问网络摄像头的Java库。

首先，我们探索了提供OpenCV项目的Java实现的javacv-platform库。然后，我们看到了使用网络摄像头捕获图像的webcam-capture库的示例实现。最后，我们查看了使用Marvin框架捕获图像的简单示例。