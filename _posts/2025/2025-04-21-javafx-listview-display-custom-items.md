---
layout: post
title:  在JavaFX ListView中显示自定义元素
category: javafx
copyright: javafx
excerpt: JavaFX
---

## 1. 简介

### 1.1 概述

[JavaFX](https://www.baeldung.com/javafx)是一款功能强大的工具，旨在为不同平台构建应用程序UI。它不仅提供UI组件，还提供各种实用工具，例如属性和可观察集合。

ListView组件方便管理集合，也就是说，我们无需定义DataModel或显式更新ListView元素。一旦ObservableList发生更改，它就会反映在ListView小部件中。

但是，这种方法需要一种在JavaFX ListView中显示自定义元素的方法，本教程介绍了一种设置域对象在ListView中的外观的方法。

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

## 2. 列工厂

### 2.1 默认行为

默认情况下，JavaFX中的ListView使用toString()方法来显示对象。

因此，显而易见的方法是重写它：

```java
public class Person {
    String firstName;
    String lastName;

    @Override
    public String toString() {
        return firstName + " " + lastName;
    }
}
```

这种方法对于学习和概念性示例来说还行，但并非最佳方法。

首先，我们的域类承担了显示实现。因此，这种方法违背了单一职责原则。

其次，其他子系统可能会使用toString()。例如，我们使用toString()方法来记录对象的状态。日志可能需要比ListView的item更多的字段。因此，在这种情况下，单个toString()实现无法满足所有模块的需求。

### 2.2 Cell Factory在ListView中显示自定义对象

让我们考虑一种更好的方法来在JavaFX ListView中显示我们的自定义对象。

ListView中的每个元素都以ListCell类的一个实例显示，ListCell有一个叫做text的属性，单元格显示其文本值。

因此，要自定义ListCell实例中的文本，我们应该更新其text属性。在哪里可以做到这一点？ListCell有一个名为updateItem的方法，当元素的单元格出现时，它会调用updateItem。当单元格发生变化时，updateItem方法也会运行。因此，我们应该从默认的ListCell类继承我们自己的实现，在这个实现中，我们需要重写updateItem。

但是我们如何让ListView使用我们的自定义实现而不是默认实现呢？

ListView可以有一个单元格工厂，单元格工厂默认为null，我们应该设置它来自定义ListView显示对象的方式。

让我们用一个例子来说明单元格工厂：

```java
public class PersonCellFactory implements Callback<ListView<Person>, ListCell<Person>> {
    @Override
    public ListCell<Person> call(ListView<Person> param) {
        return new ListCell<>(){
            @Override
            public void updateItem(Person person, boolean empty) {
                super.updateItem(person, empty);
                if (empty || person == null) {
                    setText(null);
                } else {
                    setText(person.getFirstName() + " " + person.getLastName());
                }
            }
        };
    }
}
```

CellFactory应该实现JavaFX回调，JavaFX中的Callback接口类似于标准Java Function接口，但是，由于历史原因，JavaFX使用的是Callback接口。

我们应该调用updateItem方法的默认实现，此实现会触发默认操作，例如将单元格连接到对象，并在空列表中显示一行。

方法updateItem的默认实现也调用setText，它设置将在单元格中显示的文本。

### 2.3 使用自定义小部件在JavaFX ListView中显示自定义元素

ListCell为我们提供了将自定义窗口小部件设置为内容的机会，为了在自定义窗口小部件中显示域对象，我们只需使用setGraphics()而不是setCell()。

假设我们需要将每一行显示为CheckBox，我们来看看相应的单元格工厂：

```java
public class CheckboxCellFactory implements Callback<ListView<Person>, ListCell<Person>> {
    @Override
    public ListCell<Person> call(ListView<Person> param) {
        return new ListCell<>(){
            @Override
            public void updateItem(Person person, boolean empty) {
                super.updateItem(person, empty);
                if (empty) {
                    setText(null);
                    setGraphic(null);
                } else if (person != null) {
                    setText(null);
                    setGraphic(new CheckBox(person.getFirstName() + " " + person.getLastName()));
                } else {
                    setText("null");
                    setGraphic(null);
                }
            }
        };
    }
}
```

在这个例子中，我们将text属性设置为null，如果text和graphic属性都存在，则文本将显示在小部件旁边。

当然，我们可以基于自定义元素数据设置CheckBox回调逻辑和其他属性，这需要一些代码，就像设置小部件文本一样。

## 3. 总结

在本文中，我们探讨了在JavaFX ListView中显示自定义元素的方法，可以看到ListView提供了相当灵活的设置方式，甚至可以在ListView单元格中显示自定义小部件。