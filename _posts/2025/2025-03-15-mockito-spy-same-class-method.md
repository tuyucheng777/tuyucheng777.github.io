---
layout: post
title:  使用Mockito Spy模拟同一测试类中的方法
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

Mock是单元测试中非常有用的功能，尤其是在隔离我们想要在方法中测试的单一行为时。在本教程中，我们将了解如何使用[Mockito](https://www.baeldung.com/mockito-series) Spy模拟同一测试类中的方法。

## 2. 理解问题

**单元测试是测试的第一级，也是我们针对错误的第一道防线**。但是，有时一个方法太复杂，有多个调用，特别是在遗留代码中，我们并不总是有重构的可能性。为了简化单元测试，我们将使用一个简单的示例来演示Mockito Spy的强大功能。

为了举个例子，我们来介绍一个CatTantrum类：

```java
public class CatTantrum {
    public enum CatAction {
        BITE,
        MEOW,
        VOMIT_ON_CARPET,
        EAT_DOGS_FOOD,
        KNOCK_THING_OFF_TABLE
    }

    public enum HumanReaction {
        SCREAM,
        CRY,
        CLEAN,
        PET_ON_HEAD,
        BITE_BACK,
    }

    public HumanReaction whatIsHumanReaction(CatAction action){
        return switch (action) {
            case MEOW -> HumanReaction.PET_ON_HEAD;
            case VOMIT_ON_CARPET -> HumanReaction.CLEAN;
            case EAT_DOGS_FOOD -> HumanReaction.SCREAM;
            case KNOCK_THING_OFF_TABLE -> HumanReaction.CRY;
            case BITE -> biteCatBack();
        };
    }

    public HumanReaction biteCatBack() {
        // Some logic
        return HumanReaction.BITE_BACK;
    }
}
```

方法whatIsHumanReaction()包含一些逻辑和对方法biteCatBack()的调用。我们将重点关注方法whatIsHumanReaction()的单元测试。

## 3. 使用Mockito Spy的解决方案

让我们使用Mockito Spy编写单元测试：

```java
@Test
public void givenMockMethodHumanReactions_whenCatActionBite_thenHumanReactionsBiteBack(){
    // Given
    CatTantrum catTantrum = new CatTantrum();
    CatTantrum catTantrum1 = Mockito.spy(catTantrum);
    Mockito.doReturn(HumanReaction.BITE_BACK).when(catTantrum1).biteCatBack();

    // When
    HumanReaction humanReaction1 = catTantrum1.whatIsHumanReaction(CatAction.BITE);

    // Then
    assertEquals(humanReaction1, HumanReaction.BITE_BACK);
}
```

第一步是使用Mockito.spy(catTantrum)为我们的测试对象创建一个Spy，此行允许我们Mock由whatIsHumanReaction()调用的方法biteCatBack()的返回值。

值得注意的是，**当我们调用想要测试的方法时，我们需要在Spy对象catTantrum1上调用它**，而不是原始对象catTantrum。

**Mockito Spy允许我们直接从类中使用某些方法，并存根其他方法**。如果我们使用Mock，则需要存根所有方法调用。另一方面，如果我们使用对象的真实实例，则无法Mock该类中的任何方法。

在上面的例子中，Mockito Spy允许我们在同一个对象catTantrum1上调用真实方法whatIsHumanReaction()，并调用方法biteCatBack()的存根。

## 4. 总结

在本文中，我们学习了如何使用Mockito Spy解决在同一个测试类中Mock方法的问题。

我们应该记住，编写更简单、单一用途的方法通常更好。但如果这不可能，使用Mockito Spy可以帮助绕过这种复杂性。