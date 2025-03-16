---
layout: post
title:  使用Java截屏
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 简介

在本教程中，我们将了解使用Java截屏的几种不同方法。

## 2. 使用Robot截屏

在我们的第一个例子中，我们将截取主屏幕的屏幕截图。

为此，我们将使用Robot类中的createScreenCapture()方法。它以Rectangle作为参数，设置屏幕截图的边界并返回BufferedImage对象，BufferedImage可进一步用于创建图像文件：

```java
@Test
public void givenMainScreen_whenTakeScreenshot_thenSaveToFile() throws Exception {
    Rectangle screenRect = new Rectangle(Toolkit.getDefaultToolkit().getScreenSize());
    BufferedImage capture = new Robot().createScreenCapture(screenRect);

    File imageFile = new File("single-screen.bmp");
    ImageIO.write(capture, "bmp", imageFile );
    assertTrue(imageFile .exists());
}
```

屏幕的尺寸可通过Toolkit类的getScreenSize()方法获取，在具有多个屏幕的系统中，默认使用主显示屏。

将屏幕捕获到BufferedImage后，我们可以使用ImageIO.write()将其写入文件。为此，我们需要两个附加参数，图像格式和图像文件本身。在我们的示例中，**我们使用.bmp格式，但也可以使用.png、.jpg或.gif等其他格式**。

## 3. 截取多个屏幕的屏幕截图

**也可以一次截取多个显示器的屏幕截图**，与前面的示例一样，我们可以使用Robot类中的createScreenCapture()方法，但这次屏幕截图的边界需要覆盖所有需要的屏幕。

为了获取所有显示，我们将使用GraphicsEnvironment类及其getScreenDevices()方法。

接下来，我们将获取每个单独屏幕的边界并创建一个适合所有屏幕的Rectangle：

```java
@Test
public void givenMultipleScreens_whenTakeScreenshot_thenSaveToFile() throws Exception {
    GraphicsEnvironment ge = GraphicsEnvironment.getLocalGraphicsEnvironment();
    GraphicsDevice[] screens = ge.getScreenDevices();

    Rectangle allScreenBounds = new Rectangle();
    for (GraphicsDevice screen : screens) {
        Rectangle screenBounds = screen.getDefaultConfiguration().getBounds();
        allScreenBounds.width += screenBounds.width;
        allScreenBounds.height = Math.max(allScreenBounds.height, screenBounds.height);
    }

    BufferedImage capture = new Robot().createScreenCapture(allScreenBounds);
    File imageFile = new File("all-screens.bmp");
    ImageIO.write(capture, "bmp", imageFile);
    assertTrue(imageFile.exists());
}
```

**在迭代显示器时，我们总是将宽度相加并选择一个最大高度，因为屏幕将水平拼接**。

接下来我们需要保存截图图像，和前面的例子一样，我们可以使用ImageIO.write()方法。

## 4. 对给定的GUI组件进行截图

我们还可以对给定的UI组件进行截图。

**由于每个组件都知道其大小和位置，因此可以通过getBounds()方法轻松访问尺寸**。

在本例中，我们不会使用Robot API。相反，我们将使用Component类中的paint()方法，该方法会将内容直接绘制到BufferedImage中：

```java
@Test
public void givenComponent_whenTakeScreenshot_thenSaveToFile(Component component) throws Exception {
    Rectangle componentRect = component.getBounds();
    BufferedImage bufferedImage = new BufferedImage(componentRect.width, componentRect.height, BufferedImage.TYPE_INT_ARGB);
    component.paint(bufferedImage.getGraphics());

    File imageFile = new File("component-screenshot.bmp");
    ImageIO.write(bufferedImage, "bmp", imageFile );
    assertTrue(imageFile.exists());
}
```

获取组件的边界后，我们需要创建BufferedImage。为此，我们需要宽度、高度和图像类型。在本例中，我们使用BufferedImage.TYPE_INT_ARGB，它指的是8位彩色图像。

然后我们继续调用paint()方法来填充BufferedImage，与前面的示例相同，我们使用ImageIO.write()方法将其保存到文件中。

## 5. 总结

在本教程中，我们学习了几种使用Java截屏的方法。