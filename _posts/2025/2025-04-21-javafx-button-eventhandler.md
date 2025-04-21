---
layout: post
title:  向JavaFX按钮添加事件处理程序
category: javafx
copyright: javafx
excerpt: JavaFX
---

## 1. 简介

### 1.1 概述

在这个简短的教程中，我们将**介绍[JavaFX](https://www.baeldung.com/javafx) Button组件并了解如何处理用户交互**。

### 1.2 JavaFX API

在Java 8、9和10中，无需进行其他设置即可开始使用JavaFX库。从JDK 11开始，该项目将从JDK中移除，并且应将以下依赖添加到pom.xml中：

```xml
<dependencies>
    <dependency>
        <groupId>org.openjfx</groupId>
        <artifactId>javafx-controls</artifactId>
        <version>${javafx.version}</version>
    </dependency>
    <dependency>
        <groupId>org.openjfx</groupId>
        <artifactId>javafx-fxml</artifactId>
        <version>${javafx.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.openjfx</groupId>
            <artifactId>javafx-maven-plugin</artifactId>
            <version>${javafx-maven-plugin.version}</version>
            <configuration>
                <mainClass>Main</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

## 2. 应用程序设置

首先，让我们创建一个小型应用程序，从一个包含按钮的简单[FXML布局](https://www.baeldung.com/javafx#fxml)开始：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>
<BorderPane xmlns:fx="http://javafx.com/fxml"
            xmlns="http://javafx.com/javafx"
            fx:controller="cn.tuyucheng.taketoday.button.eventhandler.ButtonEventHandlerController"
            prefHeight="200.0" prefWidth="300.0">
    <center>
        <Button fx:id="button" HBox.hgrow="ALWAYS"/>
    </center>

    <bottom>
        <Label fx:id="label" text="Test label"/>
    </bottom>
</BorderPane>
```

让我们创建ButtonEventHandlerController类，**它负责连接UI元素和应用程序逻辑**，我们将在initialize方法中设置按钮的标签：

```java
public class ButtonEventHandlerController {

    private static final Logger logger = LoggerFactory.getLogger(ButtonEventHandlerController.class);

    @FXML
    private Button button;

    @FXML
    private Label label;

    @FXML
    private void initialize() {
        button.setText("Click me");
    }
}
```

让我们启动应用程序，我们应该在中心看到一个名为“Click me”的按钮，并在窗口底部看到一个测试标签：

![](/assets/images/2025/javafx/javafxbuttoneventhandler01.png)

## 3. 点击事件

让我们从处理简单的点击事件开始，并向initialize方法添加事件处理程序：

```java
button.setOnAction(new EventHandler<ActionEvent>() {
    @Override
    public void handle(ActionEvent event) {
        logger.info("OnAction {}", event);
    }
});
```

当我们点击按钮时，会出现一条新的日志消息：

```text
INFO c.t.t.b.e.ButtonEventHandlerController - OnAction javafx.event.ActionEvent[source=Button[id=searchButton, styleClass=button]'Click me']
```

**因为事件处理程序接口只有一个方法，所以我们可以将其视为一个[函数接口](https://www.baeldung.com/java-8-functional-interfaces)，并用单个Lambda表达式替换这些行，以使我们的代码更易于阅读**：

```java
searchButton.setOnAction(event -> logger.info("OnAction {}", event));
```

让我们尝试添加另一个点击事件处理程序，我们可以简单地复制此行并更改日志消息，以便在测试应用程序时看到差异：

```java
button.setOnAction(event -> logger.info("OnAction {}", event));
button.setOnAction(event -> logger.info("OnAction2 {}", event));
```

现在，当我们点击按钮时，我们只会看到“OnAction 2”消息，这是因为第二个setOnAction方法调用将第一个事件处理程序替换成了第二个。

## 4. 不同的事件

**我们还可以处理其他事件类型，例如鼠标按下/释放、拖动和键盘事件**。

让我们为按钮添加悬停效果，当光标开始悬停在按钮上时，我们将显示阴影；当光标离开时，我们将删除阴影：

```java
Effect shadow = new DropShadow();
searchButton.setOnMouseEntered(e -> searchButton.setEffect(shadow));
searchButton.setOnMouseExited(e -> searchButton.setEffect(null));
```

## 5. 重用事件处理程序

在某些情况下，我们可能需要多次使用同一个事件处理程序。让我们创建一个事件处理程序，当我们点击鼠标的辅助按钮时，该事件处理程序将增加按钮的字体大小：

```java
EventHandler<MouseEvent> rightClickHandler = event -> {
    if (MouseButton.SECONDARY.equals(event.getButton())) {
        button.setFont(new Font(button.getFont().getSize() + 1));
    }
};
```

但是，由于我们没有将其与任何事件关联，它没有任何功能，让我们将此事件处理程序用于按钮和标签的鼠标按下事件：

```java
button.setOnMousePressed(rightClickHandler);
label.setOnMousePressed(rightClickHandler);
```

现在，当我们测试应用程序并使用辅助鼠标按钮单击标签或按钮时，我们会看到字体大小增加了。

## 6. 总结

我们学习了如何向JavaFX按钮添加事件处理程序并根据事件类型执行不同的操作。