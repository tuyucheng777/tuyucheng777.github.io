---
layout: post
title:  核心Java中的行为型模式
category: designpattern
copyright: designpattern
excerpt: 行为型模式
---

## 1. 简介

之前我们学习了[创建型设计模式](https://www.baeldung.com/java-creational-design-patterns)，以及它们在JVM和其他核心库中的应用。现在我们将学习[行为型设计模式](https://www.baeldung.com/design-patterns-series)，**它们关注的是对象之间如何交互，或者我们如何与对象交互**。

## 2. 责任链

[责任链](https://www.baeldung.com/chain-of-responsibility-pattern)模式允许对象实现一个通用接口，并在适当的情况下将每个实现委托给下一个实现。**这样，我们就可以构建一个实现链，每个实现在调用链中下一个元素之前或之后执行一些操作**：

```java
interface ChainOfResponsibility {
    void perform();
}
```

```java
class LoggingChain {
    private ChainOfResponsibility delegate;

    public void perform() {
        System.out.println("Starting chain");
        delegate.perform();
        System.out.println("Ending chain");
    }
}
```

这里我们可以看到一个例子，其中我们的实现在委托调用之前和之后打印出来。

我们不需要调用委托,我们可以决定不这样做，而是提前终止链。例如，如果有一些输入参数，我们可以验证它们，如果它们无效，则提前终止。

### 2.1 JVM中的示例

[Servlet过滤器](https://www.oracle.com/java/technologies/filters.html)是JEE生态系统中一个以这种方式工作的示例，一个实例接收Servlet请求和响应，一个FilterChain实例代表整个过滤器链。**每个过滤器都应该执行其工作，然后终止过滤器链，或者调用chain.doFilter()将控制权传递给下一个过滤器**：

```java
public class AuthenticatingFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        if (!"MyAuthToken".equals(httpRequest.getHeader("X-Auth-Token"))) {
            return;
        }
        chain.doFilter(request, response);
    }
}
```

## 3. 命令

[命令](https://www.baeldung.com/java-command-pattern)模式允许我们将一些具体的行为(或命令)封装在一个通用接口后面，以便它们可以在运行时正确触发。

通常，我们会有一个Command接口、一个接收命令实例的Receiver实例，以及一个负责调用正确命令实例的Invoker实例。**我们可以定义不同的Command接口实例，对接收者执行不同的操作**：

```java
interface DoorCommand {
    perform(Door door);
}
```

```java
class OpenDoorCommand implements DoorCommand {
    public void perform(Door door) {
        door.setState("open");
    }
}
```

这里，我们有一个命令实现，它将Door作为接收器，并导致门“open”。我们的调用者可以在需要打开指定门时调用此命令，该命令封装了如何执行此操作。

将来，我们可能需要修改OpenDoorCommand函数，使其先检查门是否锁好。**此修改将完全在命令内部进行，接收方和调用方类无需进行任何修改**。

### 3.1 JVM中的示例

这种模式的一个非常常见的例子是Swing中的Action类：

```java
Action saveAction = new SaveAction();
button = new JButton(saveAction)
```

这里，SaveAction是命令，使用此类的Swing JButton组件是调用者，并使用ActionEvent作为接收器来调用Action实现。

## 4. 迭代器

迭代器模式允许我们遍历集合中的元素并依次与每个元素进行交互，**我们利用它编写函数，接收任意迭代器对某些元素进行迭代，而无需考虑这些元素来自何处**。迭代器源可以是有序列表、无序集合或无限流：

```java
void printAll<T>(Iterator<T> iter) {
    while (iter.hasNext()) {
        System.out.println(iter.next());
    }
}
```

### 4.1 JVM中的示例

**所有JVM标准集合都通过暴露iterator()方法实现了[迭代器模式](https://www.baeldung.com/java-iterator)**，该方法返回一个针对集合中元素的Iterator<T\>对象。Stream也实现了相同的方法，只不过在这种情况下，它可能是无限流，因此迭代器可能永远不会终止。

## 5. 备忘录

**[备忘录](https://www.baeldung.com/java-memento-design-pattern)模式允许我们编写能够改变状态并恢复到先前状态的对象**，本质上是一个对象状态的“撤销”函数。

这可以通过在每次调用Setter时存储先前的状态来相对容易地实现：

```java
class Undoable {
    private String value;
    private String previous;

    public void setValue(String newValue) {
        this.previous = this.value;
        this.value = newValue;
    }

    public void restoreState() {
        if (this.previous != null) {
            this.value = this.previous;
            this.previous = null;
        }
    }
}
```

这样就可以撤消对对象所做的最后更改。

这通常是通过将整个对象状态包装在一个称为备忘录(Memento)的对象中来实现的，这样就可以在一次操作中保存和恢复整个状态，而不必单独保存每个字段。

### 5.1 JVM中的示例

**[JavaServer Faces](https://www.baeldung.com/spring-jsf)提供了一个名为StateHolder的接口，允许实现者保存和恢复其状态**。有多个标准组件实现了该接口，包括独立组件(例如HtmlInputFile、HtmlInputText或HtmlSelectManyCheckbox)以及复合组件(例如HtmlForm)。

## 6. 观察者

[观察者](https://www.baeldung.com/java-observer-pattern)模式允许一个对象向其他对象指示发生了变化，通常，我们会有一个主体(Subject)-发出事件的对象，以及一系列观察者(Observers)-接收这些事件的对象。观察者会向主体注册，表示它们希望收到变化的通知。**一旦发生变化，主体中发生的任何变化都会通知观察者**：

```java
class Observable {
    private String state;
    private Set<Consumer<String>> listeners = new HashSet<>;

    public void addListener(Consumer<String> listener) {
        this.listeners.add(listener);
    }

    public void setState(String newState) {
        this.state = state;
        for (Consumer<String> listener : listeners) {
            listener.accept(newState);
        }
    }
}
```

这需要一组事件监听器，并在每次状态随着新的状态值改变时调用每个事件监听器。

### 6.1 JVM中的示例

**Java有一对标准类可以让我们做到这一点-java.beans.PropertyChangeSupport和java.beans.PropertyChangeListener**。

PropertyChangeSupport是一个类，可以添加和删除观察者，并通知它们所有状态变化。PropertyChangeListener是一个接口，我们的代码可以实现它来接收发生的任何变化：

```java
PropertyChangeSupport observable = new PropertyChangeSupport();

// Add some observers to be notified when the value changes
observable.addPropertyChangeListener(evt -> System.out.println("Value changed: " + evt));

// Indicate that the value has changed and notify observers of the new value
observable.firePropertyChange("field", "old value", "new value");
```

请注意，还有另外一对类似乎更合适-java.util.Observer和java.util.Observable。但是，由于不灵活且不可靠，它们在Java 9中已被弃用。

## 7. 策略

[策略模式](https://www.baeldung.com/java-strategy-pattern)允许我们编写通用代码，然后将特定的策略插入其中，以便为我们提供具体情况所需的特定行为。

这通常通过一个表示策略的接口来实现，**客户端代码可以根据具体情况编写实现此接口的具体类**。例如，我们可能有一个系统需要通知最终用户，并将通知机制实现为可插拔策略：

```java
interface NotificationStrategy {
    void notify(User user, Message message);
}
```

```java
class EmailNotificationStrategy implements NotificationStrategy {
    // ...
}
```

```java
class SMSNotificationStrategy implements NotificationStrategy {
    // ...
}
```

然后，我们可以在运行时决定使用哪种策略来向该用户发送消息。我们还可以编写新的策略，以最大程度地减少对系统其他部分的影响。

### 7.1 JVM中的示例

**标准Java库广泛使用了这种模式，其使用方式乍一看可能并不明显**。例如，Java 8中引入的[Streams API](https://www.baeldung.com/java-streams)就广泛使用了这种模式。提供给map()、filter()和其他方法的Lambda表达式都是提供给泛型方法的可插拔策略。

不过，例子可以追溯到更早的时期。Java 1.2中引入的Comparator接口是一种策略，可以根据需要对集合中的元素进行排序。我们可以提供Comparator的不同实例，以便根据需要以不同的方式对同一列表进行排序：

```java
// Sort by name
Collections.sort(users, new UsersNameComparator());

// Sort by ID
Collections.sort(users, new UsersIdComparator());
```

## 8. 模板方法

当我们想要协调多个不同的方法协同工作时，可以使用[模板方法模式](https://www.baeldung.com/java-template-method-pattern)。**我们将定义一个包含模板方法的基类，以及一组一个或多个抽象方法**-这些抽象方法要么未实现，要么已实现并带有一些默认行为。然后，**模板方法会以固定的模式调用这些抽象方法**。之后，我们的代码会实现此类的子类，并根据需要实现这些抽象方法：

```java
class Component {
    public void render() {
        doRender();
        addEventListeners();
        syncData();
    }

    protected abstract void doRender();

    protected void addEventListeners() {}

    protected void syncData() {}
}
```

这里，我们有一些任意的UI组件，我们的子类将实现doRender()方法来实际渲染组件。我们还可以选择实现addEventListeners()和syncData()方法，当我们的UI框架渲染此组件时，它将保证这3个方法都按正确的顺序被调用。

### 8.1 JVM中的示例

**Java集合类使用的AbstractList、 AbstractSet和AbstractMap中有很多这种模式的例子**。例如，indexOf()和lastIndexOf()方法都基于listIterator()方法工作，该方法具有默认实现，但在某些子类中会被重写。同样，add(T)和addAll(int,T)方法都基于add(int, T)方法工作，该方法没有默认实现，需要由子类实现。

Java IO也在InputStream、OutputStream、Reader和Writer中使用了这种模式，例如，InputStream类有几个方法可以用于read(byte[], int, int)，这需要子类来实现。

## 9. 访问者

**[访问者](https://www.baeldung.com/java-visitor-pattern)模式允许我们的代码以类型安全的方式处理各种子类，而无需进行instanceof检查**。我们将为每个需要支持的具体子类创建一个访问者接口，其中包含一个方法。我们的基类将包含一个accept(Visitor)方法，子类将分别调用此访问者接口上的相应方法，并将自身作为参数传入。这样，我们就可以在每个方法中实现具体的行为，并且每个方法都知道它将与具体类型交互：

```java
interface UserVisitor<T> {
    T visitStandardUser(StandardUser user);
    T visitAdminUser(AdminUser user);
    T visitSuperuser(Superuser user);
}
```

```java
class StandardUser {
    public <T> T accept(UserVisitor<T> visitor) {
        return visitor.visitStandardUser(this);
    }
}
```

这里我们有一个UserVisitor接口，其中包含3个不同的访问者方法。我们的例子StandardUser调用了相应的方法，AdminUser和Superuser中也会执行相同的操作，然后我们可以根据需要编写访问者来使用这些方法：

```java
class AuthenticatingVisitor {
    public Boolean visitStandardUser(StandardUser user) {
        return false;
    }
    public Boolean visitAdminUser(AdminUser user) {
        return user.hasPermission("write");
    }
    public Boolean visitSuperuser(Superuser user) {
        return true;
    }
}
```

我们的StandardUser从来没有权限，我们的Superuser总是有权限，而我们的AdminUser可能有权限，但这需要在用户本身中查找。

### 9.1 JVM中的示例

Java NIO2框架通过[Files.walkFileTree()](https://www.baeldung.com/java-nio2-file-visitor)使用了这种模式，它接收FileVisitor的一个实现，其中包含处理遍历文件树各个方面的方法。**我们的代码可以用它来搜索文件、打印匹配的文件、处理目录中的多个文件，或者执行许多其他需要在目录中进行的操作**：

```java
Files.walkFileTree(startingDir, new SimpleFileVisitor() {
    public FileVisitResult visitFile(Path file, BasicFileAttributes attr) {
        System.out.println("Found file: " + file);
    }

    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
        System.out.println("Found directory: " + dir);
    }
});
```

## 10. 总结

在本文中，我们了解了用于对象行为的各种设计模式。我们还研究了这些模式在核心JVM中的应用示例，以便我们了解它们如何帮助许多应用程序从中受益。