---
layout: post
title:  仅根据类名构造Java对象
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在本教程中，我们将探索使用类名创建Java对象的过程。[Java反射API](https://www.baeldung.com/java-reflection)提供了多种方法来完成此任务，但是，确定最适合当前上下文的方法可能具有挑战性。

为了解决这个问题，让我们从一种简单的方法开始，并逐步完善它以获得更有效的解决方案。

## 2. 使用类名创建对象

让我们想象一个汽车服务中心，该中心负责机动车的维护和维修，使用工作卡对服务请求进行分类和管理，我们可以将其表示为类图：

![](/assets/images/2025/javareflect/javaobjectsmakeusingclassname01.png)

让我们看一下MaintenanceJob和RepairJob类：

```java
public class MaintenanceJob {
    public String getJobType() {
        return "Maintenance Job";
    }
}

public class RepairJob {
    public String getJobType() {
        return "Repair Job";
    }
}
```

现在，让我们实现BronzeJobCard：

```java
public class BronzeJobCard {
    private Object jobType;
    public void setJobType(String jobType) throws ClassNotFoundException,
            NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class jobTypeClass = Class.forName(jobType);
        this.jobType = jobTypeClass.getDeclaredConstructor().newInstance();
    }

    public String startJob() {
        if(this.jobType instanceof RepairJob) {
            return "Start Bronze " + ((RepairJob) this.jobType).getJobType();
        }
        if(this.jobType instanceof MaintenanceJob) {
            return "Start Bronze " + ((MaintenanceJob) this.jobType).getJobType();
        }
        return "Bronze Job Failed";
    }
}
```

在BronzeJobCard中，**[Class.forName()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html#forName(java.lang.String))接收类的完全限定名称来返回原始作业对象**。稍后，startJob()对原始对象进行[类型转换](https://www.baeldung.com/java-type-casting)以获取正确的作业类型。除了这些缺点之外，还有处理异常的开销。

让我们看看它的实际效果：

```java
@Test
public void givenBronzeJobCard_whenJobTypeRepairAndMaintenance_thenStartJob() throws ClassNotFoundException, InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
    BronzeJobCard bronzeJobCard1 = new BronzeJobCard();
    bronzeJobCard1.setJobType("cn.tuyucheng.taketoday.reflection.createobject.basic.RepairJob");
    assertEquals("Start Bronze Repair Job", bronzeJobCard1.startJob());

    BronzeJobCard bronzeJobCard2 = new BronzeJobCard();
    bronzeJobCard2.setJobType("cn.tuyucheng.taketoday.reflection.createobject.basic.MaintenanceJob");
    assertEquals("Start Bronze Maintenance Job", bronzeJobCard2.startJob());
}
```

所以，上述方法启动了两个作业，一个修复作业和一个维护作业。

几个月后，服务中心也决定开始喷漆工作。因此，我们创建了一个新的类PaintJob，但是BronzeJobCard可以容纳这个新添加的内容吗？让我们看看：

```java
@Test
public void givenBronzeJobCard_whenJobTypePaint_thenFailJob() throws ClassNotFoundException, InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
    BronzeJobCard bronzeJobCard = new BronzeJobCard();
    bronzeJobCard.setJobType("cn.tuyucheng.taketoday.reflection.createobject.basic.PaintJob");
    assertEquals("Bronze Job Failed", bronzeJobCard.startJob());
}
```

彻底失败了！由于使用原始对象，BronzeJobCard无法处理新的PaintJob。

## 3. 使用原始类对象创建对象

在本节中，我们将升级作业卡，使用java.lang.Class而不是类名来创建作业。首先，看一下类图：

![](/assets/images/2025/javareflect/javaobjectsmakeusingclassname02.png)

让我们看看SilverJobCard与BronzeJobCard有何不同：

```java
public class SilverJobCard {
    private Object jobType;

    public void setJobType(Class jobTypeClass) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        this.jobType = jobTypeClass.getDeclaredConstructor().newInstance();
    }

    public String startJob() {
        if (this.jobType instanceof RepairJob) {
            return "Start Silver " + ((RepairJob) this.jobType).getJobType();
        }
        if (this.jobType instanceof MaintenanceJob) {
            return "Start Silver " + ((MaintenanceJob) this.jobType).getJobType();
        }
        return "Silver Job Failed";
    }
}
```

**它不再依赖作业类的完全限定名称来创建对象**。但是，原始对象和异常的问题保持不变。

如下所示，它还可以处理创建作业然后启动它们：

```java
@Test
public void givenSilverJobCard_whenJobTypeRepairAndMaintenance_thenStartJob() throws InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
    SilverJobCard silverJobCard1 = new SilverJobCard();
    silverJobCard1.setJobType(RepairJob.class);
    assertEquals("Start Silver Repair Job", silverJobCard1.startJob());

    SilverJobCard silverJobCard2 = new SilverJobCard();
    silverJobCard2.setJobType(MaintenanceJob.class);
    assertEquals("Start Silver Maintenance Job", silverJobCard2.startJob());
}
```

但是，与BronzeJobCard一样，SilverJobCard也无法容纳新的PaintJob：

```java
@Test
public void givenSilverJobCard_whenJobTypePaint_thenFailJob() throws ClassNotFoundException, InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
    SilverJobCard silverJobCard = new SilverJobCard();
    silverJobCard.setJobType(PaintJob.class);
    assertEquals("Silver Job Failed", silverJobCard.startJob());
}
```

此外，setJobType()方法不限制传递除RepairJob和MaintenanceJob之外的任何对象，这可能会导致开发阶段出现错误代码。

## 4. 使用Class对象和泛型创建对象

之前，我们看到了原始对象如何影响代码的质量。在本节中，我们将解决这个问题。但首先，让我们看一下类图：

![](/assets/images/2025/javareflect/javaobjectsmakeusingclassname03.png)

**这次，我们摆脱了原始对象。GoldJobCard接收类型参数，并在方法setJobType()中使用[泛型](https://www.baeldung.com/java-generics)**，让我们检查一下实现：

```java
public class GoldJobCard<T> {
    private T jobType;

    public void setJobType(Class<T> jobTypeClass) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        this.jobType = jobTypeClass.getDeclaredConstructor().newInstance();
    }

    public String startJob() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        return "Start Gold " + this.jobType.getClass().getMethod("getJobType", null)
                .invoke(this.jobType).toString();
    }
}
```

有趣的是，startJob() 现在使用[反射API](https://www.baeldung.com/java-reflection)调用对象上的方法。最后，我们还摆脱了类型转换的需要。让我们看看它的表现如何：

```java
@Test
public void givenGoldJobCard_whenJobTypeRepairMaintenanceAndPaint_thenStartJob() throws InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
    GoldJobCard<RepairJob> goldJobCard1 = new GoldJobCard();
    goldJobCard1.setJobType(RepairJob.class);
    assertEquals("Start Gold Repair Job", goldJobCard1.startJob());

    GoldJobCard<MaintenanceJob> goldJobCard2 = new GoldJobCard();
    goldJobCard2.setJobType(MaintenanceJob.class);
    assertEquals("Start Gold Maintenance Job", goldJobCard2.startJob());

    GoldJobCard<PaintJob> goldJobCard3 = new GoldJobCard();
    goldJobCard3.setJobType(PaintJob.class);
    assertEquals("Start Gold Paint Job", goldJobCard3.startJob());
}
```

在这里，它也处理PaintJob。

但是，**我们仍然无法在开发阶段限制传入startJob()方法的对象**。因此，对于没有getJobType()方法的对象(如MaintenanceJob、RepairJob和PaintJob)，该方法将失败。

## 5. 使用类型参数扩展创建对象

现在该解决之前提出的问题了。让我们从习惯的类图开始：

![](/assets/images/2025/javareflect/javaobjectsmakeusingclassname04.png)

**我们引入了Job接口，所有Job对象都必须实现该接口**。此外，PlatinumJobCard现在仅接收Job对象，由T extends Job参数指示。

实际上，这种方法与[工厂设计模式](https://www.baeldung.com/java-factory-pattern)非常相似。我们可以引入一个JobCardFactory来处理Job对象的创建。

接下来，我们可以看一下具体实现：

```java
public class PlatinumJobCard<T extends Job> {
    private T jobType;

    public void setJobType(Class<T> jobTypeClass) throws
            NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        this.jobType = jobTypeClass.getDeclaredConstructor().newInstance();
    }

    public String startJob() {
        return "Start Platinum " + this.jobType.getJobType();
    }
}
```

**我们通过引入Job接口摆脱了反射API和startJob()方法中的类型转换**，值得庆幸的是，现在PlatinumJobCard将能够处理未来的Job类型而无需对其进行任何修改。让我们看看它的实际效果：

```java
@Test
public void givenPlatinumJobCard_whenJobTypeRepairMaintenanceAndPaint_thenStartJob() throws InvocationTargetException, NoSuchMethodException, InstantiationException, IllegalAccessException {
    PlatinumJobCard<RepairJob> platinumJobCard1 = new PlatinumJobCard();
    platinumJobCard1.setJobType(RepairJob.class);
    assertEquals("Start Platinum Repair Job", platinumJobCard1.startJob());

    PlatinumJobCard<MaintenanceJob> platinumJobCard2 = new PlatinumJobCard();
    platinumJobCard2.setJobType(MaintenanceJob.class);
    assertEquals("Start Platinum Maintenance Job", platinumJobCard2.startJob());

    PlatinumJobCard<PaintJob> platinumJobCard3 = new PlatinumJobCard();
    platinumJobCard3.setJobType(PaintJob.class);
    assertEquals("Start Platinum Paint Job", platinumJobCard3.startJob());
}
```

## 6. 总结

在本文中，我们探讨了使用类名和Class对象创建对象的各种方法。我们展示了相关对象如何实现基本接口，然后，可以进一步使用它来简化对象创建过程。使用这种方法，无需进行类型转换，而且它确保了Job接口的使用，在开发过程中强制进行类型检查。