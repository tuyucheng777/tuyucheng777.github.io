---
layout: post
title:  Java中的K-Means聚类算法
category: algorithms
copyright: algorithms
excerpt: K-Means
---

## 1. 概述

聚类是一类无监督算法的总称，用于发现彼此密切相关的事物、人或想法的群体。

在这个看似简单的单行定义中，我们看到了一些流行语。聚类到底是什么？无监督算法又是什么？

在本教程中，我们首先会阐明这些概念。然后，我们将了解它们如何在Java中体现。

## 2. 无监督算法

在使用大多数学习算法之前，我们应该以某种方式向其提供一些样本数据，并让算法从这些数据中学习。在机器学习术语中，**我们将该样本数据集称为训练数据。此外，整个过程称为训练过程**。

无论如何，**我们可以根据训练过程中所需的监督量对[学习算法](https://www.baeldung.com/cs/examples-supervised-unsupervised-learning)进行分类**，此类别中的两种主要学习算法是：

- **监督学习**：在监督算法中，训练数据应该包含每个点的实际解。例如，如果我们要训练垃圾邮件过滤算法，我们会将样本电子邮件及其标签(即垃圾邮件或非垃圾邮件)提供给算法。从数学上讲，我们将从包含xs和ys的训练集中推断出f(x)。
- **无监督学习**：当训练数据中没有标签时，算法就是无监督的。例如，我们有大量关于音乐家的数据，并且打算从中发现相似的音乐家群体。

## 3. 聚类

聚类是一种无监督算法，用于发现相似的事物、想法或人群。与监督算法不同，我们不会使用已知标签的示例来训练聚类算法。相反，聚类会尝试在训练集中寻找数据中没有标签的结构。

### 3.1 K均值聚类

[K-Means](https://www.baeldung.com/cs/k-means-for-classification)是一种聚类算法，其基本特性是：**聚类的数量是预先定义的**。除了K-Means之外，还有其他类型的聚类算法，如层次聚类、亲和传播或谱聚类。

### 3.2 K-Means的工作原理

假设我们的目标是在数据集中找到一些相似的组，例如：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm01.png)

K-Means从k个随机放置的质心开始，**顾名思义，质心是聚类的中心点**。例如，这里我们添加了4个随机质心：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm02.png)

然后我们将每个现有数据点分配给其最近的质心：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm03.png)

分配后，我们将质心移动到分配给它的点的平均位置。记住，质心应该是聚类的中心点：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm04.png)

每次我们完成重定位质心后，当前迭代就结束。**我们重复这些迭代，直到多个连续迭代之间的分配不再变化**：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm05.png)

当算法终止时，正如预期的那样，找到了这四个聚类。现在我们知道了K-Means的工作原理，让我们用Java来实现它。

### 3.3 特征表示

在对不同的训练数据集进行建模时，我们需要一个数据结构来表示模型属性及其对应的值。例如，音乐家可以有一个流派属性，其值可以是“摇滚”，**我们通常使用“特征”来指代属性及其值的组合**。

为了为特定的学习算法准备数据集，我们通常会使用一组通用的数值属性，用于比较不同的项目。例如，如果我们让用户为每位艺术家标记一个流派，那么最终我们就可以计算出每位艺术家被标记特定流派的次数：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm06.png)

像Linkin Park这样的艺术家，其特征向量是[rock -> 7890, nu-metal -> 700, alternative -> 520, pop -> 3\]。因此，如果我们能找到一种方法将属性表示为数值，那么我们就可以通过比较它们对应的向量条目来简单地比较两个不同的项目(例如艺术家)。

由于数字向量是一种用途广泛的数据结构，我们将使用它们来表示特征，以下是我们在Java中实现特征向量的方法：
```java
public class Record {
    private final String description;
    private final Map<String, Double> features;

    // constructor, getter, toString, equals and hashcode
}
```

### 3.4 查找类似项目

在K-Means的每次迭代中，我们都需要一种方法来找到数据集中每个项目的最近质心。计算两个特征向量之间距离的最简单方法之一是使用[欧几里得距离](https://en.wikipedia.org/wiki/Euclidean_distance)，两个向量(如[p1,q1\]和[p2,q2\])之间的欧几里得距离等于：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm07.png)

让我们用Java实现这个函数，首先是抽象：
```java
public interface Distance {
    double calculate(Map<String, Double> f1, Map<String, Double> f2);
}
```

除了欧氏距离，**还有其他方法可以计算不同项目之间的距离或相似度，例如皮尔逊[相关系数](https://www.baeldung.com/cs/correlation-coefficient)**，这种抽象使得在不同的距离度量之间切换变得容易。

让我们看看欧几里得距离的实现：
```java
public class EuclideanDistance implements Distance {

    @Override
    public double calculate(Map<String, Double> f1, Map<String, Double> f2) {
        double sum = 0;
        for (String key : f1.keySet()) {
            Double v1 = f1.get(key);
            Double v2 = f2.get(key);

            if (v1 != null && v2 != null) {
                sum += Math.pow(v1 - v2, 2);
            }
        }

        return Math.sqrt(sum);
    }
}
```

首先，我们计算相应条目之间的平方差之和。然后，通过应用sqrt函数，我们计算实际的欧几里得距离。

### 3.5 质心表示

质心与普通特征位于同一空间，因此我们可以用类似于特征的方式表示它们：
```java
public class Centroid {

    private final Map<String, Double> coordinates;

    // constructors, getter, toString, equals and hashcode
}
```

现在我们已经完成了一些必要的抽象，是时候编写我们的K-Means实现，以下是我们的方法签名：
```java
public class KMeans {

    private static final Random random = new Random();

    public static Map<Centroid, List<Record>> fit(List<Record> records,
                                                  int k,
                                                  Distance distance,
                                                  int maxIterations) {
        // omitted
    }
}
```

让我们分解一下这个方法签名：

- dataset是一组特征向量，由于每个特征向量都是一个记录，因此数据集类型为List<Record\>
- k参数决定了聚类的数量，我们应该提前提供
- distance概括了我们计算两个特征之间的差异的方式
- 当赋值在连续几次迭代中停止变化时，K-Means就会终止。除了这个终止条件之外，我们还可以设置迭代次数的上限，maxIterations参数决定了该上限
- 当K-Means终止时，每个质心应该会分配一些特征，因此我们使用Map<Centroid,List<Record\>\>作为返回类型；基本上，每个Map条目对应一个聚类

### 3.6 质心生成

第一步是生成k个随机放置的质心。

虽然每个质心可以包含完全随机的坐标，但最好**在每个属性的最小和最大可能值之间生成随机坐标**，在不考虑可能值范围的情况下生成随机质心会导致算法收敛速度更慢。

首先，我们应该计算每个属性的最小值和最大值，然后生成每对之间的随机值：
```java
private static List<Centroid> randomCentroids(List<Record> records, int k) {
    List<Centroid> centroids = new ArrayList<>();
    Map<String, Double> maxs = new HashMap<>();
    Map<String, Double> mins = new HashMap<>();

    for (Record record : records) {
        record.getFeatures().forEach((key, value) -> {
            // compares the value with the current max and choose the bigger value between them
            maxs.compute(key, (k1, max) -> max == null || value > max ? value : max);

            // compare the value with the current min and choose the smaller value between them
            mins.compute(key, (k1, min) -> min == null || value < min ? value : min);
        });
    }

    Set<String> attributes = records.stream()
            .flatMap(e -> e.getFeatures().keySet().stream())
            .collect(toSet());
    for (int i = 0; i < k; i++) {
        Map<String, Double> coordinates = new HashMap<>();
        for (String attribute : attributes) {
            double max = maxs.get(attribute);
            double min = mins.get(attribute);
            coordinates.put(attribute, random.nextDouble() * (max - min) + min);
        }

        centroids.add(new Centroid(coordinates));
    }

    return centroids;
}
```

现在，我们可以将每条记录分配给这些随机质心之一。

### 3.7 分配

首先，给定一个Record，我们应该找到最接近它的质心：
```java
private static Centroid nearestCentroid(Record record, List<Centroid> centroids, Distance distance) {
    double minimumDistance = Double.MAX_VALUE;
    Centroid nearest = null;

    for (Centroid centroid : centroids) {
        double currentDistance = distance.calculate(record.getFeatures(), centroid.getCoordinates());

        if (currentDistance < minimumDistance) {
            minimumDistance = currentDistance;
            nearest = centroid;
        }
    }

    return nearest;
}
```

每条记录属于其最近的质心聚类：
```java
private static void assignToCluster(Map<Centroid, List<Record>> clusters,
                                    Record record,
                                    Centroid centroid) {
    clusters.compute(centroid, (key, list) -> {
        if (list == null) {
            list = new ArrayList<>();
        }

        list.add(record);
        return list;
    });
}
```

### 3.8 质心重定位

如果经过一次迭代后，质心不包含任何分配，则我们不会重新定位它。否则，我们应该将每个属性的质心坐标重新定位到所有已分配记录的平均位置：
```java
private static Centroid average(Centroid centroid, List<Record> records) {
    if (records == null || records.isEmpty()) {
        return centroid;
    }

    Map<String, Double> average = centroid.getCoordinates();
    records.stream().flatMap(e -> e.getFeatures().keySet().stream())
            .forEach(k -> average.put(k, 0.0));

    for (Record record : records) {
        record.getFeatures().forEach(
                (k, v) -> average.compute(k, (k1, currentValue) -> v + currentValue)
        );
    }

    average.forEach((k, v) -> average.put(k, v / records.size()));

    return new Centroid(average);
}
```

由于我们可以重新定位单个质心，现在可以实现relocateCentroids方法：
```java
private static List<Centroid> relocateCentroids(Map<Centroid, List<Record>> clusters) {
    return clusters.entrySet().stream().map(e -> average(e.getKey(), e.getValue())).collect(toList());
}
```

这个简单的单行代码遍历所有质心，重新定位它们，并返回新的质心。

### 3.9 整合

在每次迭代中，将所有记录分配到它们最近的质心后，首先，我们应该将当前分配与上一次迭代进行比较。

如果分配结果相同，则算法终止。否则，在跳到下一次迭代之前，我们应该重新定位质心：
```java
public static Map<Centroid, List<Record>> fit(List<Record> records,
                                              int k,
                                              Distance distance,
                                              int maxIterations) {

    List<Centroid> centroids = randomCentroids(records, k);
    Map<Centroid, List<Record>> clusters = new HashMap<>();
    Map<Centroid, List<Record>> lastState = new HashMap<>();

    // iterate for a pre-defined number of times
    for (int i = 0; i < maxIterations; i++) {
        boolean isLastIteration = i == maxIterations - 1;

        // in each iteration we should find the nearest centroid for each record
        for (Record record : records) {
            Centroid centroid = nearestCentroid(record, centroids, distance);
            assignToCluster(clusters, record, centroid);
        }

        // if the assignments do not change, then the algorithm terminates
        boolean shouldTerminate = isLastIteration || clusters.equals(lastState);
        lastState = clusters;
        if (shouldTerminate) {
            break;
        }

        // at the end of each iteration we should relocate the centroids
        centroids = relocateCentroids(clusters);
        clusters = new HashMap<>();
    }

    return lastState;
}
```

## 4. 示例：在Last.fm上发现类似的艺术家

Last.fm通过记录用户收听的内容来构建每个用户音乐品味的详细资料，在本节中，我们将找到相似艺术家的聚类。为了构建适合此任务的数据集，我们将使用Last.fm的3个API：

1. 用于获取Last.fm上[顶级艺术家集合的API](https://www.last.fm/api/show/chart.getTopArtists)。
2. 另一个用于查找[热门标签](https://www.last.fm/api/show/chart.getTopTags)的API，每个用户都可以为某位艺术家添加标签，例如rock。因此，Last.fm维护着一个包含这些标签及其频率的数据库。
3. 还有一个API用于[获取艺术家的热门标签](https://www.last.fm/api/show/artist.getTopTags)(按受欢迎程度排序)。由于此类标签数量众多，我们只会保留那些位列全局热门标签的标签。

### 4.1 Last.fm的API

要使用这些API，我们应该[从Last.fm获取API密钥](https://www.last.fm/api/authentication)，并在每个HTTP请求中发送它。我们将使用以下[Retrofit](https://www.baeldung.com/retrofit)服务来调用这些API：
```java
public interface LastFmService {

    @GET("/2.0/?method=chart.gettopartists&format=json&limit=50")
    Call<Artists> topArtists(@Query("page") int page);

    @GET("/2.0/?method=artist.gettoptags&format=json&limit=20&autocorrect=1")
    Call<Tags> topTagsFor(@Query("artist") String artist);

    @GET("/2.0/?method=chart.gettoptags&format=json&limit=100")
    Call<TopTags> topTags();

    // A few DTOs and one interceptor
}
```

现在，让我们找出Last.fm上最受欢迎的艺术家：
```java
// setting up the Retrofit service

private static List<String> getTop100Artists() throws IOException {
    List<String> artists = new ArrayList<>();
    // Fetching the first two pages, each containing 50 records.
    for (int i = 1; i <= 2; i++) {
        artists.addAll(lastFm.topArtists(i).execute().body().all());
    }

    return artists;
}
```

类似地，我们可以获取热门标签：
```java
private static Set<String> getTop100Tags() throws IOException {
    return lastFm.topTags().execute().body().all();
}
```

最后，我们可以建立一个艺术家及其标签频率的数据集：
```java
private static List<Record> datasetWithTaggedArtists(List<String> artists,
                                                     Set<String> topTags) throws IOException {
    List<Record> records = new ArrayList<>();
    for (String artist : artists) {
        Map<String, Double> tags = lastFm.topTagsFor(artist).execute().body().all();

        // Only keep popular tags.
        tags.entrySet().removeIf(e -> !topTags.contains(e.getKey()));

        records.add(new Record(artist, tags));
    }

    return records;
}
```

### 4.2 形成艺术家聚类

现在，我们可以将准备好的数据集提供给我们的K-Means实现：
```java
List<String> artists = getTop100Artists();
Set<String> topTags = getTop100Tags();
List<Record> records = datasetWithTaggedArtists(artists, topTags);

Map<Centroid, List<Record>> clusters = KMeans.fit(records, 7, new EuclideanDistance(), 1000);
// Printing the cluster configuration
clusters.forEach((key, value) -> {
    System.out.println("-------------------------- CLUSTER ----------------------------");

    // Sorting the coordinates to see the most significant tags first.
    System.out.println(sortedCentroid(key)); 
    String members = String.join(", ", value.stream().map(Record::getDescription).collect(toSet()));
    System.out.print(members);

    System.out.println();
    System.out.println();
});
```

如果我们运行此代码，那么它会将聚类可视化为文本输出：
```text
------------------------------ CLUSTER -----------------------------------
Centroid {classic rock=65.58333333333333, rock=64.41666666666667, british=20.333333333333332, ... }
David Bowie, Led Zeppelin, Pink Floyd, System of a Down, Queen, blink-182, The Rolling Stones, Metallica, 
Fleetwood Mac, The Beatles, Elton John, The Clash

------------------------------ CLUSTER -----------------------------------
Centroid {Hip-Hop=97.21428571428571, rap=64.85714285714286, hip hop=29.285714285714285, ... }
Kanye West, Post Malone, Childish Gambino, Lil Nas X, A$AP Rocky, Lizzo, xxxtentacion, 
Travi$ Scott, Tyler, the Creator, Eminem, Frank Ocean, Kendrick Lamar, Nicki Minaj, Drake

------------------------------ CLUSTER -----------------------------------
Centroid {indie rock=54.0, rock=52.0, Psychedelic Rock=51.0, psychedelic=47.0, ... }
Tame Impala, The Black Keys

------------------------------ CLUSTER -----------------------------------
Centroid {pop=81.96428571428571, female vocalists=41.285714285714285, indie=22.785714285714285, ... }
Ed Sheeran, Taylor Swift, Rihanna, Miley Cyrus, Billie Eilish, Lorde, Ellie Goulding, Bruno Mars, 
Katy Perry, Khalid, Ariana Grande, Bon Iver, Dua Lipa, Beyoncé, Sia, P!nk, Sam Smith, Shawn Mendes, 
Mark Ronson, Michael Jackson, Halsey, Lana Del Rey, Carly Rae Jepsen, Britney Spears, Madonna, 
Adele, Lady Gaga, Jonas Brothers

------------------------------ CLUSTER -----------------------------------
Centroid {indie=95.23076923076923, alternative=70.61538461538461, indie rock=64.46153846153847, ... }
Twenty One Pilots, The Smiths, Florence + the Machine, Two Door Cinema Club, The 1975, Imagine Dragons, 
The Killers, Vampire Weekend, Foster the People, The Strokes, Cage the Elephant, Arcade Fire, 
Arctic Monkeys

------------------------------ CLUSTER -----------------------------------
Centroid {electronic=91.6923076923077, House=39.46153846153846, dance=38.0, ... }
Charli XCX, The Weeknd, Daft Punk, Calvin Harris, MGMT, Martin Garrix, Depeche Mode, The Chainsmokers, 
Avicii, Kygo, Marshmello, David Guetta, Major Lazer

------------------------------ CLUSTER -----------------------------------
Centroid {rock=87.38888888888889, alternative=72.11111111111111, alternative rock=49.16666666, ... }
Weezer, The White Stripes, Nirvana, Foo Fighters, Maroon 5, Oasis, Panic! at the Disco, Gorillaz, 
Green Day, The Cure, Fall Out Boy, OneRepublic, Paramore, Coldplay, Radiohead, Linkin Park, 
Red Hot Chili Peppers, Muse
```

由于质心坐标是按平均标签频率排序的，我们可以轻松地识别出每个聚类中的主导类型。例如，最后一个聚类是由一群优秀的老摇滚乐队组成的，而第二个聚类则充满了说唱明星。

虽然这种聚类很有意义，但在大多数情况下，它并不完美，因为数据仅仅是从用户行为中收集的。

## 5. 可视化

刚才，我们的算法以终端友好的方式可视化了艺术家聚类，如果我们将聚类配置转换为JSON并将其提供给D3.js，那么只需几行JavaScript，我们就能得到一个美观易用、易于使用的[Radial Tidy-Tree](https://observablehq.com/@d3/radial-tidy-tree?collection=@d3/d3-hierarchy)图表：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm08.png)

我们必须将Map<Centroid,List<Record\>\>转换为具有类似模式的JSON，如这个[d3.js示例](https://raw.githubusercontent.com/d3/d3-hierarchy/v1.1.8/test/data/flare.json)所示。

## 6. 聚类数量

K-Means的一个基本特性是，我们应该提前定义聚类的数量。到目前为止，我们对k使用了静态值，但确定这个值可能是一个具有挑战性的问题。计算聚类数量有两种常用方法：

1. 领域知识
2. 数学启发法

如果我们足够幸运，对这个领域了解足够多，那么我们或许能够轻松猜出正确的数字。否则，我们可以应用一些启发式方法，如肘部方法或轮廓方法，以了解聚类的数量。

在继续之前，我们应该知道，这些启发式方法虽然有用，但仅仅是启发式方法，可能无法提供明确的答案。

### 6.1 肘部法

要使用肘部法，我们首先应该计算每个聚类质心与其所有成员之间的差异。随着我们将更多不相关的成员分组到聚类中，质心与其成员之间的距离会增大，因此聚类质量会下降。

执行此距离计算的一种方法是使用平方误差和，平方误差和或SSE等于质心与其所有成员之间的平方差和：
```java
public static double sse(Map<Centroid, List<Record>> clustered, Distance distance) {
    double sum = 0;
    for (Map.Entry<Centroid, List<Record>> entry : clustered.entrySet()) {
        Centroid centroid = entry.getKey();
        for (Record record : entry.getValue()) {
            double d = distance.calculate(centroid.getCoordinates(), record.getFeatures());
            sum += Math.pow(d, 2);
        }
    }

    return sum;
}
```

然后，我们可以对不同的k值运行K-Means算法，并计算每个值的SSE：
```java
List<Record> records = // the dataset;
Distance distance = new EuclideanDistance();
List<Double> sumOfSquaredErrors = new ArrayList<>();
for (int k = 2; k <= 16; k++) {
    Map<Centroid, List<Record>> clusters = KMeans.fit(records, k, distance, 1000);
    double sse = Errors.sse(clusters, distance);
    sumOfSquaredErrors.add(sse);
}
```

最终，可以通过绘制聚类数量与SSE的关系来找到合适的k：

![](/assets/images/2025/algorithms/javakmeansclusteringalgorithm09.png)

通常，随着聚类数量的增加，聚类成员之间的距离会减小。但是，我们不能为k选择任意大的值，因为如果多个聚类只有一个成员，那么聚类的初衷就完全背离了。

**肘部法背后的理念是找到一个合适的k值，使得SSE在该值附近急剧下降**。例如，k = 9可能是一个不错的选择。

## 7. 总结

在本教程中，我们首先介绍了机器学习中的一些重要概念。然后，我们熟悉了K-Means聚类算法的原理。最后，我们为K-Means编写了一个简单的实现，使用来自Last.fm的真实数据集测试了我们的算法，并以漂亮的图形方式可视化了聚类结果。