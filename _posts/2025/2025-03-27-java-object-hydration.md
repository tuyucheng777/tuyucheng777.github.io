---
layout: post
title:  水化对象是什么意思
category: libraries
copyright: libraries
excerpt: Hydrate
---

## 1. 简介

在本教程中，我们将在编程背景下讨论“水合”这个术语，并深入探讨水合Java对象的含义。 

## 2. 对象水化

### 2.1 延迟初始化

对象的[延迟加载](https://www.baeldung.com/hibernate-lazy-eager-loading)或延迟初始化是软件应用程序中常用的模式，Java中的对象是使用new关键字创建的类的实例，对象是程序的构建块，对象与其他对象交互以实现所需的功能。 

在面向对象编程范式中，对象通常用于表示现实世界的实体，因此对象具有多个相关属性。**对象[初始化](https://www.baeldung.com/java-initialization)是指使用真实数据填充对象属性的过程**，这通常是通过调用类构造函数并传入数据作为参数来完成的。**初始化也可以从数据源(例如网络、数据库或文件系统)完成**。 

对象初始化有时可能是一项资源密集型操作，尤其是当数据来自不同的数据源时。此外，对象并不总是在创建后立即被程序使用。 

**在这种情况下，最好尽可能推迟对象初始化，直到需要它为止**。这种模式称为延迟初始化，因为我们一次使用空数据创建对象，并在将来用相关数据延迟填充对象。**有意识地延迟数据初始化有助于提高代码性能和内存利用率**。

让我们创建一个具有多个属性的User类：

```java
public class User {
    private String uId;
    private String firstName;
    private String lastName;
    private String alias;
    // constructor, getter-setters
}
```

我们可以创建一个User对象并将其保存在内存中，而无需用有意义的数据填充其属性：

```java
User iamUser = new User();
```

### 2.2 什么是水合？

**使用延迟初始化，我们故意延迟已创建并存在于内存中的对象的初始化过程，将数据填充到现有空对象的过程称为“水合”**。

我们创建的User实例是一个虚拟实例，该对象没有任何相关的数据属性，因为直到此时才需要它。**为了使对象有用，我们应该用相关的域数据填充对象，这可以通过用来自网络、数据库或文件系统等源的数据填充它来实现**。 

我们按照以下步骤执行User实例的补充。首先，我们将补充逻辑编写为类级方法，该方法使用类设置器来设置相关数据。在我们的示例中，我们将使用我们的数据。但是，我们也可以从文件或类似来源获取数据：

```java
public void generateMyUser() {
    this.setAlias("007");
    this.setFirstName("James");
    this.setLastName("Bond");
    this.setuId("JB");
}
```

**现在我们创建一个空的User实例，并在需要时通过调用generateMyUser()方法来填充同一个实例**：

```java
User jamesBond = new User();
// performing hydration
jamesBond.generateMyUser();
```

我们可以通过断言其属性的状态来验证水合的结果：

```java
User jamesBond = new User();
Assert.assertNull(jamesBond.getAlias());

jamesBond.generateMyUser();
Assert.assertEquals("007", jamesBond.getAlias());
```

## 3. 水合和反序列化

水化和反序列化不是同义词，我们不应该互换使用它们。**[反序列化](https://www.baeldung.com/jackson-deserialization)是编程中用于从序列化形式恢复或重新创建对象的过程，我们经常通过网络存储或传输对象**。在此过程中，序列化(将对象转换为字节流)和反序列化(恢复对象的逆过程)非常有用。 

我们可以将我们的User实例序列化为文件或等效文件：

```java
try {
    FileOutputStream fileOut = new FileOutputStream(outputName);
    ObjectOutputStream out = new ObjectOutputStream(fileOut);
    out.writeObject(user);
    out.close();
    fileOut.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

类似地，当需要时，我们可以从其序列化形式重新创建User实例：

```java
try {
    FileInputStream fileIn = new FileInputStream(serialisedFile);
    ObjectInputStream in = new ObjectInputStream(fileIn);
    deserializedUser = (User) in.readObject();
    in.close();
    fileIn.close();
} catch (IOException | ClassNotFoundException e) {
    e.printStackTrace();
}
```

显然，水化和反序列化都涉及处理对象并用数据填充。**但是，两者之间的重要区别在于反序列化主要是创建实例和填充属性的单步过程**。

**另一方面，水化只关心将数据添加到预先形成的对象的属性中。因此，反序列化是同一步骤中的对象实例化和对象水化**。 

## 4. ORM框架中的水化

[ORM(对象关系映射)](https://www.baeldung.com/cs/object-relational-mapping)框架是一种软件开发范例，使我们能够将面向对象编程与关系型数据库相结合。ORM框架有助于实现应用程序代码中的对象与关系数据库中的表之间的映射，并允许开发人员将数据库实体作为原生对象进行交互。 

对象水化的思想在ORM框架中更为流行，例如Hibernate或[JPA](https://www.baeldung.com/learn-jpa-hibernate)。

让我们考虑一个JPA实体类Book及其对应的Repository类，如下所示：

```java
@Entity
@Table(name = "books")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "name")
    private String name;
    // other columns and methods
}
```

```java
public interface BookRepository extends JpaRepository<Book, Long> {
}
```

根据ORM原则，实体Book表示关系数据库中的一张表。实体与数据库的交互以我们上面定义的BookRepository接口的形式抽象出来，该类的一个实例表示表中的一行。 

当我们使用众多内置find()方法之一或使用自定义查询从数据库加载Book实例时，ORM框架会执行几个步骤：

```java
Book aTaleOfTwoCities = bookRepository.findOne(1L);
```

**框架会初始化一个空对象，通常是通过调用类的默认构造函数。对象准备就绪后，框架会尝试从缓存存储中加载属性数据**。如果此时缓存未命中，框架会与数据库建立连接并查询表以获取行。

**一旦获得了ResultSet，框架就会将前面提到的对象aTaleOfTwoCities与结果集对象进行混合，并最终返回实例**。 

## 5. 总结

在本文中，我们讨论了编程上下文中“水合”一词的含义，我们了解了水合与反序列化的区别。最后，我们探讨了ORM框架和普通对象模型中对象水合的示例。 