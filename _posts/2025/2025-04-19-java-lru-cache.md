---
layout: post
title:  如何在Java中实现LRU缓存
category: algorithms
copyright: algorithms
excerpt: LRU缓存
---

## 1. 概述

在本教程中，我们将了解LRU[缓存](https://en.wikipedia.org/wiki/Cache_(computing))并研究Java中的实现。

## 2. LRU缓存

最近最少使用(LRU)缓存是一种缓存驱逐算法，它按使用顺序组织元素。顾名思义，在LRU中，最长时间未使用的元素将从缓存中驱逐。

例如，如果我们有一个容量为3个元素的缓存：

![](/assets/images/2025/algorithms/javalrucache01.png)

最初，缓存为空，我们将元素8放入缓存中，元素9和6仍像以前一样被缓存。但是现在，缓存容量已满，为了放入下一个元素，我们必须逐出缓存中最近最少使用的元素。

在我们使用Java实现LRU缓存之前，最好先了解一下缓存的一些方面：

- 所有操作应按O(1)的顺序运行
- 缓存大小有限
- 所有缓存操作必须支持并发
- 如果缓存已满，则添加新元素必须调用LRU策略

### 2.1 LRU缓存的结构

现在，我们来思考一个有助于我们设计缓存的问题。

**我们如何设计一个可以在恒定时间内执行读取、排序(时间排序)和删除元素等操作的数据结构**？

看来，要找到这个问题的答案，我们需要深入思考关于LRU缓存及其特性的讨论：

- 实际上，LRU缓存是一种[队列](https://www.baeldung.com/java-queue)-如果某个元素被重新访问，它将进入驱逐顺序的末尾。
- 由于缓存的大小有限，此队列将具有特定的容量。每当有新元素加入时，它都会被添加到队列的头部。当发生驱逐时，它会从队列的尾部进行。
- 缓存中的数据必须在常量时间内完成，这在Queue中是不可能的。但是，Java的[HashMap](https://www.baeldung.com/java-hashmap)数据结构可以做到这一点。
- 删除最近最少使用的元素必须在恒定时间内完成，这意味着对于Queue的实现，我们将使用[DoublyLinkedList](https://en.wikipedia.org/wiki/Doubly_linked_list)而不是[SingleLinkedList](https://www.baeldung.com/java-linkedlist)或数组。

**因此，LRU缓存只不过是DoublyLinkedList和HashMap的组合，如下所示**：

![](/assets/images/2025/algorithms/javalrucache02.png)

这个想法是将键保留在Map上，以便快速访问Queue内的数据。

### 2.2 LRU算法

LRU算法非常简单，如果键存在于HashMap中，则为缓存命中；否则，则为缓存未命中。

发生缓存未命中后，我们将执行两个步骤：

1. 在列表前面添加一个新元素
2. 在HashMap中添加一个新条目并引用列表的头部

并且，缓存命中后我们将执行两个步骤：

1. 删除命中元素并将其添加到列表前面
2. 使用对列表前面的新引用来更新HashMap

## 3. Java实现

首先，我们定义Cache接口：

```java
public interface Cache<K, V> {
    boolean set(K key, V value);
    Optional<V> get(K key);
    int size();
    boolean isEmpty();
    void clear();
}
```

现在，我们将定义代表缓存的LRUCache类：

```java
public class LRUCache<K, V> implements Cache<K, V> {
    private int size;
    private Map<K, LinkedListNode<CacheElement<K,V>>> linkedListNodeMap;
    private DoublyLinkedList<CacheElement<K,V>> doublyLinkedList;

    public LRUCache(int size) {
        this.size = size;
        this.linkedListNodeMap = new HashMap<>(maxSize);
        this.doublyLinkedList = new DoublyLinkedList<>();
    }
   // rest of the implementation
}
```

我们可以创建一个具有特定大小的LRUCache实例，在此实现中，我们使用HashMap集合来存储对LinkedListNode的所有引用。

现在，让我们讨论一下LRUCache上的操作。

### 3.1 Put操作

第一个是put方法：

```java
public boolean put(K key, V value) {
    CacheElement<K, V> item = new CacheElement<K, V>(key, value);
    LinkedListNode<CacheElement<K, V>> newNode;
    if (this.linkedListNodeMap.containsKey(key)) {
        LinkedListNode<CacheElement<K, V>> node = this.linkedListNodeMap.get(key);
        newNode = doublyLinkedList.updateAndMoveToFront(node, item);
    } else {
        if (this.size() >= this.size) {
            this.evictElement();
        }
        newNode = this.doublyLinkedList.add(item);
    }
    if(newNode.isEmpty()) {
        return false;
    }
    this.linkedListNodeMap.put(key, newNode);
    return true;
}
```

首先，我们在存储所有键/引用的linkedListNodeMap中找到对应的键。如果该键存在，则表示缓存命中，并准备从DoublyLinkedList中检索CacheElement并将其移至前端。

之后，我们用新的引用更新linkedListNodeMap并将其移动到列表的前面：

```java
public LinkedListNode<T> updateAndMoveToFront(LinkedListNode<T> node, T newValue) {
    if (node.isEmpty() || (this != (node.getListReference()))) {
        return dummyNode;
    }
    detach(node);
    add(newValue);
    return head;
}
```

首先，我们检查节点是否为空。此外，节点的引用必须与列表相同。之后，我们将该节点从列表中分离出来，并将newValue添加到列表中。

但如果该键不存在，则发生缓存未命中，我们必须将新的键放入linkedListNodeMap中。在此之前，我们需要检查列表的大小，如果列表已满，则必须从列表中移除最近最少使用的元素。

### 3.2 Get操作

我们来看看获取操作：

```java
public Optional<V> get(K key) {
    LinkedListNode<CacheElement<K, V>> linkedListNode = this.linkedListNodeMap.get(key);
    if(linkedListNode != null && !linkedListNode.isEmpty()) {
        linkedListNodeMap.put(key, this.doublyLinkedList.moveToFront(linkedListNode));
        return Optional.of(linkedListNode.getElement().getValue());
    }
    return Optional.empty();
}
```

如上所示，这个操作很简单。首先，我们从linkedListNodeMap中获取节点，然后检查它是否为null或空。

其余操作与以前相同，只有moveToFront方法有一处区别：

```java
public LinkedListNode<T> moveToFront(LinkedListNode<T> node) {
    return node.isEmpty() ? dummyNode : updateAndMoveToFront(node, node.getElement());
}
```

现在，让我们创建一些测试来验证我们的缓存是否正常工作：

```java
@Test
public void addSomeDataToCache_WhenGetData_ThenIsEqualWithCacheElement(){
    LRUCache<String,String> lruCache = new LRUCache<>(3);
    lruCache.put("1","test1");
    lruCache.put("2","test2");
    lruCache.put("3","test3");
    assertEquals("test1",lruCache.get("1").get());
    assertEquals("test2",lruCache.get("2").get());
    assertEquals("test3",lruCache.get("3").get());
}
```

现在，让我们测试一下驱逐策略：

```java
@Test
public void addDataToCacheToTheNumberOfSize_WhenAddOneMoreData_ThenLeastRecentlyDataWillEvict(){
    LRUCache<String,String> lruCache = new LRUCache<>(3);
    lruCache.put("1","test1");
    lruCache.put("2","test2");
    lruCache.put("3","test3");
    lruCache.put("4","test4");
    assertFalse(lruCache.get("1").isPresent());
 }
```

## 4. 处理并发

到目前为止，我们假设我们的缓存仅在单线程环境中使用。

为了使此容器线程安全，我们需要同步所有公共方法，让我们在之前的实现中添加[ReentrantReadWriteLock](https://www.baeldung.com/java-thread-safety#reentrant-locks)和[ConcurrentHashMap](https://www.baeldung.com/java-concurrent-map)：

```java
public class LRUCache<K, V> implements Cache<K, V> {
    private int size;
    private final Map<K, LinkedListNode<CacheElement<K,V>>> linkedListNodeMap;
    private final DoublyLinkedList<CacheElement<K,V>> doublyLinkedList;
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public LRUCache(int size) {
        this.size = size;
        this.linkedListNodeMap = new ConcurrentHashMap<>(size);
        this.doublyLinkedList = new DoublyLinkedList<>();
    }
    // ...
}
```

**我们更倾向于使用可重入读/写锁，而不是将方法声明为[synchronized](https://www.baeldung.com/java-thread-safety#synchronized-statements)，因为它为我们提供了更大的灵活性来决定何时使用读写锁**。

### 4.1 writeLock

现在，让我们在put方法中添加对writeLock的调用：

```java
public boolean put(K key, V value) {
    this.lock.writeLock().lock();
    try {
        //..
    } finally {
        this.lock.writeLock().unlock();
    }
}
```

当我们对资源使用writeLock时，只有持有锁的线程才能写入或读取该资源。因此，所有其他尝试读取或写入该资源的线程都必须等待，直到当前锁持有者释放该锁。

**这对于防止[死锁](https://www.baeldung.com/cs/os-deadlock)非常重要，如果try块内的任何操作失败，我们仍然会在方法末尾使用finally块在退出函数之前释放锁**。

需要writeLock的其他操作之一是evictElement，我们在put方法中使用了它：

```java
private boolean evictElement() {
    this.lock.writeLock().lock();
    try {
        //...
    } finally {
        this.lock.writeLock().unlock();
    }
}
```

### 4.2 readLock

现在向get方法添加readLock调用：

```java
public Optional<V> get(K key) {
    this.lock.readLock().lock();
    try {
        //...
    } finally {
        this.lock.readLock().unlock();
    }
}
```

这看起来和我们在put方法中所做的完全一样，唯一的区别是我们用了readLock而不是writeLock。因此，读写锁之间的区别使我们能够在缓存未被更新时并行读取缓存。

现在，让我们在并发环境中测试我们的缓存：

```java
@Test
public void runMultiThreadTask_WhenPutDataInConcurrentToCache_ThenNoDataLost() throws Exception {
    final int size = 50;
    final ExecutorService executorService = Executors.newFixedThreadPool(5);
    Cache<Integer, String> cache = new LRUCache<>(size);
    CountDownLatch countDownLatch = new CountDownLatch(size);
    try {
        IntStream.range(0, size).<Runnable>mapToObj(key -> () -> {
            cache.put(key, "value" + key);
            countDownLatch.countDown();
        }).forEach(executorService::submit);
        countDownLatch.await();
    } finally {
        executorService.shutdown();
    }
    assertEquals(cache.size(), size);
    IntStream.range(0, size).forEach(i -> assertEquals("value" + i,cache.get(i).get()));
}
```

## 5. 总结

在本教程中，我们了解了LRU缓存的确切含义，包括它的一些最常用的功能。然后，我们了解了一种用Java实现LRU缓存的方法，并探索了一些最常见的操作。

最后，我们介绍了使用锁机制实现的并发。