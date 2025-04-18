---
layout: post
title:  在Spring应用程序中使用Couchbase
category: persistence
copyright: persistence
excerpt: Couchbase
---

## 1. 简介

在我们对[Couchbase介绍](https://www.baeldung.com/java-couchbase-sdk)的后续中，我们创建一组Spring服务，这些服务可以一起使用，为Spring应用程序创建基本的持久层，而无需使用Spring Data。

## 2. 集群服务

为了满足JVM中只能激活一个CouchbaseEnvironment的限制，我们首先编写一个连接到Couchbase集群的服务，并提供对数据存储桶的访问，而无需直接公开Cluster或CouchbaseEnvironment实例。

### 2.1 接口

这是我们的ClusterService接口：

```java
public interface ClusterService {
    Bucket openBucket(String name, String password);
}
```

### 2.2 实现

我们的实现类在Spring上下文初始化期间的@PostConstruct阶段实例化一个DefaultCouchbaseEnvironment并连接到集群。

这确保了集群不为空，并且在将类注入到其他服务类时已连接，从而使它们能够打开一个或多个数据存储桶：

```java
@Service
public class ClusterServiceImpl implements ClusterService {
    private Cluster cluster;

    @PostConstruct
    private void init() {
        CouchbaseEnvironment env = DefaultCouchbaseEnvironment.create();
        cluster = CouchbaseCluster.create(env, "localhost");
    }
    // ...
}
```

接下来，我们提供一个ConcurrentHashMap来包含打开的bucket，并实现openBucket方法：

```java
private Map<String, Bucket> buckets = new ConcurrentHashMap<>();

@Override
synchronized public Bucket openBucket(String name, String password) {
    if(!buckets.containsKey(name)) {
        Bucket bucket = cluster.openBucket(name, password);
        buckets.put(name, bucket);
    }
    return buckets.get(name);
}
```

## 3. 存储桶服务

根据你构建应用程序的方式，可能需要在多个Spring服务中提供对同一数据存储桶的访问。

如果我们在应用程序启动期间仅仅尝试在两个或多个服务中打开同一个存储桶，则尝试此操作的第二个服务可能会遇到ConcurrentTimeoutException。

为了避免这种情况，我们为每个存储桶定义了一个BucketService接口和一个实现类，每个实现类都充当了ClusterService和需要直接访问特定Bucket的类之间的桥梁。

### 3.1 接口

这是我们的BucketService接口：

```java
public interface BucketService {
    Bucket getBucket();
}
```

### 3.2 实现

以下类提供对“tuyucheng-tutorial”存储桶的访问：

```java
@Service
@Qualifier("TutorialBucketService")
public class TutorialBucketService implements BucketService {

    @Autowired
    private ClusterService couchbase;

    private Bucket bucket;

    @PostConstruct
    private void init() {
        bucket = couchbase.openBucket("tuyucheng-tutorial", "");
    }

    @Override
    public Bucket getBucket() {
        return bucket;
    }
}
```

通过在我们的TutorialBucketService实现类中注入ClusterService并在带有@PostConstruct注解的方法中打开存储桶，我们确保当TutorialBucketService注入到其他服务时，存储桶已准备好使用。

## 4. 持久层

现在我们已经有一个服务来获取Bucket实例，我们将创建一个类似Repository的持久层，该层向其他服务提供实体类的CRUD操作，而无需向它们公开Bucket实例。

### 4.1 Person实体

以下是我们希望保存的Person实体类：

```java
public class Person {

    private String id;
    private String type;
    private String name;
    private String homeTown;

    // standard getters and setters
}
```

### 4.2 实体类与JSON之间的相互转换

为了将实体类与Couchbase在其持久化操作中使用的JsonDocument对象相互转换，我们定义了JsonDocumentConverter接口：

```java
public interface JsonDocumentConverter<T> {
    JsonDocument toDocument(T t);
    T fromDocument(JsonDocument doc);
}
```

### 4.3 实现JSON转换器

接下来，我们需要为Person实体实现一个JsonConverter。

```java
@Service
public class PersonDocumentConverter implements JsonDocumentConverter<Person> {
    // ...
}
```

我们可以将Jackson库与JsonObject类的toJson和fromJson方法来序列化和反序列化实体，但是这样做会产生额外的开销。

相反，对于toDocument方法，我们将使用JsonObject类的流畅方法来创建和填充JsonObject，然后将其包装为JsonDocument：

```java
@Override
public JsonDocument toDocument(Person p) {
    JsonObject content = JsonObject.empty()
            .put("type", "Person")
            .put("name", p.getName())
            .put("homeTown", p.getHomeTown());
    return JsonDocument.create(p.getId(), content);
}
```

对于fromDocument方法，我们将在fromDocument方法中使用JsonObject类的getString方法以及Person类中的Setter：

```java
@Override
public Person fromDocument(JsonDocument doc) {
    JsonObject content = doc.content();
    Person p = new Person();
    p.setId(doc.id());
    p.setType("Person");
    p.setName(content.getString("name"));
    p.setHomeTown(content.getString("homeTown"));
    return p;
}
```

### 4.4 CRUD接口

我们现在创建一个通用的CrudService接口，定义实体类的持久化操作：

```java
public interface CrudService<T> {
    void create(T t);
    T read(String id);
    T readFromReplica(String id);
    void update(T t);
    void delete(String id);
    boolean exists(String id);
}
```

### 4.5 实现CRUD服务

有了实体和转换器类，我们现在为Person实体实现CrudService，注入上面显示的存储桶服务和文档转换器，并在初始化期间检索存储桶：

```java
@Service
public class PersonCrudService implements CrudService<Person> {

    @Autowired
    private TutorialBucketService bucketService;

    @Autowired
    private PersonDocumentConverter converter;

    private Bucket bucket;

    @PostConstruct
    private void init() {
        bucket = bucketService.getBucket();
    }

    @Override
    public void create(Person person) {
        if(person.getId() == null) {
            person.setId(UUID.randomUUID().toString());
        }
        JsonDocument document = converter.toDocument(person);
        bucket.insert(document);
    }

    @Override
    public Person read(String id) {
        JsonDocument doc = bucket.get(id);
        return (doc != null ? converter.fromDocument(doc) : null);
    }

    @Override
    public Person readFromReplica(String id) {
        List<JsonDocument> docs = bucket.getFromReplica(id, ReplicaMode.FIRST);
        return (docs.isEmpty() ? null : converter.fromDocument(docs.get(0)));
    }

    @Override
    public void update(Person person) {
        JsonDocument document = converter.toDocument(person);
        bucket.upsert(document);
    }

    @Override
    public void delete(String id) {
        bucket.remove(id);
    }

    @Override
    public boolean exists(String id) {
        return bucket.exists(id);
    }
}
```

## 5. 整合

现在我们已经准备好了持久层的所有部分，下面是一个简单的注册服务示例，该示例使用PersonCrudService来持久化和检索注册Person：

```java
@Service
public class RegistrationService {

    @Autowired
    private PersonCrudService crud;
    
    public void registerNewPerson(String name, String homeTown) {
        Person person = new Person();
        person.setName(name);
        person.setHomeTown(homeTown);
        crud.create(person);
    }
    
    public Person findRegistrant(String id) {
        try{
            return crud.read(id);
        }
        catch(CouchbaseException e) {
            return crud.readFromReplica(id);
        }
    }
}
```

## 6. 总结

通过一些基本的Spring服务，将Couchbase合并到Spring应用程序中并在不使用Spring Data的情况下实现基本的持久层是相当简单的。