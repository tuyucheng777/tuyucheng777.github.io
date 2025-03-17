---
layout: post
title:  如何在Java中使用自定义字体
category: java
copyright: java
excerpt: Java Swing
---

## 1. 简介

当我们开发Java应用程序时，我们可能需要使用自定义[字体](https://www.baeldung.com/java-add-text-to-image)进行设计，以使GUI中的显示更加清晰。幸运的是，Java默认提供多种字体，使用自定义字体可让设计师发挥创造力，开发出有吸引力的应用程序。

**在本教程中，我们将探讨如何在Java应用程序中使用自定义字体**。

## 2. 配置自定义字体

Java支持[TruеType字体](https://en.wikipedia.org/wiki/TrueType)(TTF)和[OpеnType字体](https://en.wikipedia.org/wiki/OpenType)(OTF)的集成，以实现自定义字体的使用。

**实际上，这些字体并非固有包含在标准Java字体库中，因此我们需要将它们明确地加载到我们的应用程序中**。

让我们深入了解使用以下代码片段在Java中加载自定义字体所需的步骤：

```java
void usingCustomFonts() {
    GraphicsEnvironment GE = GraphicsEnvironment.getLocalGraphicsEnvironment();
    List<String> AVAILABLE_FONT_FAMILY_NAMES = Arrays.asList(GE.getAvailableFontFamilyNames());
    try {
        List<File> LIST = Arrays.asList(
                new File("font/JetBrainsMono/JetBrainsMono-Thin.ttf"),
                new File("font/JetBrainsMono/JetBrainsMono-Light.ttf"),
                new File("font/Roboto/Roboto-Light.ttf"),
                new File("font/Roboto/Roboto-Regular.ttf"),
                new File("font/Roboto/Roboto-Medium.ttf")
        );
        for (File LIST_ITEM : LIST) {
            if (LIST_ITEM.exists()) {
                Font FONT = Font.createFont(Font.TRUETYPE_FONT, LIST_ITEM);
                if (!AVAILABLE_FONT_FAMILY_NAMES.contains(FONT.getFontName())) {
                    GE.registerFont(FONT);
                }
            }
        }
    } catch (FontFormatException | IOException exception) {
        JOptionPane.showMessageDialog(null, exception.getMessage());
    }
}
```

在上述代码段中，我们利用GraphicsEnvironmеnt.gеtLocalGraphicsEnvironmеnt()访问本地图形环境，从而能够访问系统字体。此外，我们使用GE.gеtAvailablеFontFamilyNamеs()方法从系统中获取可用的字体系列名称。

代码还在循环中使用Font.crеatеFont()从指定的字体文件中动态加载指定字体(例如，不同粗细的JеtBrains Mono和Roboto)。此外，使用AVAILABLE_FONT_FAMILY_NAMES.contains(FONT.gеtFontNamе())将这些加载的字体与系统可用的字体进行交叉检查。

## 3. 使用自定义字体

让我们使用Java [Swing](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/swing/package-summary.html)应用程序在GUI中实现这些加载的字体：

```java
JFrame frame = new JFrame("Custom Font Example");
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setLayout(new FlowLayout());

JLabel label1 = new JLabel("TEXT1");
label1.setFont(new Font("Roboto Medium", Font.PLAIN, 17));

JLabel label2 = new JLabel("TEXT2");
label2.setFont(new Font("JetBrainsMono-Thin", Font.PLAIN, 17));

frame.add(label1);
frame.add(label2);

frame.pack();
frame.setVisible(true);
```

此处，GUI代码通过指定字体名称和样式来演示JLabel组件中加载的自定义字体的用法，下图显示了使用默认字体和自定义字体之间的区别：

![](/assets/images/2025/java/javacustomfont.png)

## 4. 总结

总之，在Java应用程序中加入自定义字体可以增强视觉吸引力，并允许我们创建独特的用户界面。

通过遵循概述的步骤并利用提供的代码示例，开发人员可以无缝地将自定义字体集成到他们的Java GUI应用程序中，从而获得更美观和独特的用户体验。