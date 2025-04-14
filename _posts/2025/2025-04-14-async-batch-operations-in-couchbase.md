---
layout: post
title:  Couchbase中的异步批处理操作
category: persistence
copyright: persistence
excerpt: Couchbase
---

## 1. 简介

在本教程的后续部分，我们将介绍如何[在Spring应用程序中使用Couchbase](https://www.baeldung.com/couchbase-sdk-spring)，探讨Couchbase SDK的异步特性以及如何使用它来批量执行持久化操作，从而使我们的应用程序能够最佳地利用Couchbase资源。

### 1.1 CrudService接口

首先，我们扩充通用的CrudService接口以包含批量操作：

```java
public interface CrudService<T> {
    ...
    
    List<T> readBulk(Iterable<String> ids);

    void createBulk(Iterable<T> items);

    void updateBulk(Iterable<T> items);

    void deleteBulk(Iterable<String> ids);

    boolean exists(String id);
}
```

### 1.2 CouchbaseEntity接口

我们为想要持久化的实体定义一个接口：

```java
public interface CouchbaseEntity {

    String getId();
    
    void setId(String id);
}
```

### 1.3 AbstractCrudService类

然后，我们将在一个通用抽象类中实现这些方法，该类派生自[上一节](https://www.baeldung.com/couchbase-sdk-spring)中使用的PersonCrudService类，其开头如下：

```java
public abstract class AbstractCrudService<T extends CouchbaseEntity> implements CrudService<T> {
    private BucketService bucketService;
    private Bucket bucket;
    private JsonDocumentConverter<T> converter;

    public AbstractCrudService(BucketService bucketService, JsonDocumentConverter<T> converter) {
        this.bucketService = bucketService;
        this.converter = converter;
    }

    protected void loadBucket() {
        bucket = bucketService.getBucket();
    }
    
    // ...
}
```

## 2. AsyncBucket接口

Couchbase SDK提供了AsyncBucket接口来执行异步操作，给定一个Bucket实例，你可以通过async()方法获取其异步版本：

```java
AsyncBucket asyncBucket = bucket.async();
```

## 3. 批量操作

为了使用AsyncBucket接口执行批量操作，我们使用了[RxJava](https://github.com/ReactiveX/RxJava/wiki)库。

### 3.1 批量读取

这里我们实现了readBulk方法，首先，我们使用RxJava中的AsyncBucket和flatMap机制将文档异步检索到Observable<JsonDocument\>中，然后使用RxJava中的toBlocking机制将它们转换为实体列表：

```java
@Override
public List<T> readBulk(Iterable<String> ids) {
    AsyncBucket asyncBucket = bucket.async();
    Observable<JsonDocument> asyncOperation = Observable
            .from(ids)
            .flatMap(new Func1<String, Observable<JsonDocument>>() {
                public Observable<JsonDocument> call(String key) {
                    return asyncBucket.get(key);
                }
            });

    List<T> items = new ArrayList<T>();
    try {
        asyncOperation.toBlocking()
                .forEach(new Action1<JsonDocument>() {
                    public void call(JsonDocument doc) {
                        T item = converter.fromDocument(doc);
                        items.add(item);
                    }
                });
    } catch (Exception e) {
        logger.error("Error during bulk get", e);
    }

    return items;
}
```

### 3.2 批量插入

我们再次使用RxJava的flatMap构造来实现createBulk方法。

由于批量突变请求的生成速度比其响应的生成速度快，有时会导致过载情况，因此每当遇到BackpressureException时，我们都会进行指数延迟的重试：

```java
@Override
public void createBulk(Iterable<T> items) {
    AsyncBucket asyncBucket = bucket.async();
    Observable
            .from(items)
            .flatMap(new Func1<T, Observable<JsonDocument>>() {
                @SuppressWarnings("unchecked")
                @Override
                public Observable<JsonDocument> call(final T t) {
                    if(t.getId() == null) {
                        t.setId(UUID.randomUUID().toString());
                    }
                    JsonDocument doc = converter.toDocument(t);
                    return asyncBucket.insert(doc)
                            .retryWhen(RetryBuilder
                                    .anyOf(BackpressureException.class)
                                    .delay(Delay.exponential(TimeUnit.MILLISECONDS, 100))
                                    .max(10)
                                    .build());
                }
            })
            .last()
            .toBlocking()
            .single();
}
```

### 3.3 批量更新

我们在updateBulk方法中使用类似的机制：

```java
@Override
public void updateBulk(Iterable<T> items) {
    AsyncBucket asyncBucket = bucket.async();
    Observable
            .from(items)
            .flatMap(new Func1<T, Observable<JsonDocument>>() {
                @SuppressWarnings("unchecked")
                @Override
                public Observable<JsonDocument> call(final T t) {
                    JsonDocument doc = converter.toDocument(t);
                    return asyncBucket.upsert(doc)
                            .retryWhen(RetryBuilder
                                    .anyOf(BackpressureException.class)
                                    .delay(Delay.exponential(TimeUnit.MILLISECONDS, 100))
                                    .max(10)
                                    .build());
                }
            })
            .last()
            .toBlocking()
            .single();
}
```

### 3.4 批量删除

而我们deleteBulk方法的编写如下：

```java
@Override
public void deleteBulk(Iterable<String> ids) {
    AsyncBucket asyncBucket = bucket.async();
    Observable
            .from(ids)
            .flatMap(new Func1<String, Observable<JsonDocument>>() {
                @SuppressWarnings("unchecked")
                @Override
                public Observable<JsonDocument> call(String key) {
                    return asyncBucket.remove(key)
                            .retryWhen(RetryBuilder
                                    .anyOf(BackpressureException.class)
                                    .delay(Delay.exponential(TimeUnit.MILLISECONDS, 100))
                                    .max(10)
                                    .build());
                }
            })
            .last()
            .toBlocking()
            .single();
}
```

## 4. PersonCrudService

最后，我们编写一个Spring服务PersonCrudService，它扩展了Person实体的AbstractCrudService。

由于所有Couchbase交互都是在抽象类中实现的，因此实体类的实现很简单，因为我们只需要确保所有依赖都已注入并且存储桶已加载：

```java
@Service
public class PersonCrudService extends AbstractCrudService<Person> {

    @Autowired
    public PersonCrudService(
            @Qualifier("TutorialBucketService") BucketService bucketService,
            PersonDocumentConverter converter) {
        super(bucketService, converter);
    }

    @PostConstruct
    private void init() {
        loadBucket();
    }
}
```

## 5. 总结

你可以在[Couchbase官方开发者文档网站](https://docs.couchbase.com/java-sdk/2.6/start-using-sdk.html)了解有关Couchbase Java SDK的更多信息。