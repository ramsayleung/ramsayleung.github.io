+++
title = "guava 探究系列之五：新类型容器"
date = 2019-12-12T15:45:00-08:00
lastmod = 2025-01-09T18:44:55-08:00
tags = ["java", "guava"]
categories = ["Guava探究"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

JDK 提供了各种功能强大的工具类, 宛如装备齐全的军火库,
而容器就是其中一项内置的利器, 提供了包括诸多常用的数据结构, 下图对 JDK
已有容器进行了概括:

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20191122171101.png" caption="<span class=\"figure-number\">Figure 1: </span>JDK 容器" >}}

不过, 虽然 JDK 的容器类已经五花八门, 琳琅满目, 但是某些很有用的容器类
JDK 依然欠缺, 而 Guava 恰如其分地填补了这些空缺, 开发了 JDK
所欠缺的容器类, "造福大众".

此外, 虽然引入了新的容器类, 但 Guava 实现了
JDK 的 `Collection` 接口, 保证 Guava 的容器类能够与 JDK
的容器类”和谐共处”, 避免不必要的”纷争”.


## <span class="section-num">2</span> Multiset {#multiset}

假设你是个书店的老板, 你想统计下书店里不同书籍的存货量,
你可能写下这样的实现:

```java
Map<String, Integer> counts = new HashMap<String, Integer>();
for (String book : bookNames) {
    Integer count = counts.get(book);
    if (count == null) {
        counts.put(book, 1);
    } else {
        counts.put(book, count + 1);
    }
}
```

嗯, 我现在想改需求, 我想知道书店里共有多少本书? 怎么办呢? 把 `counts` 的
value 都加起来?

对于这样的要求, Guava 提供了一个更好的解决方案: 一个新类型容器
`Multiset` , 它支持新增多个相同类型的元素并统计. 维基百科给出的关于
`Multiset`
的[解释](https://en.wikipedia.org/wiki/Set_(abstract_data_type)#Multiset):

> 这是个成员可以出现多次的集合(Set), 也被称为背包(bag)

大名鼎鼎的《算法/Algorithm》也给出过 [bag](https://algs4.cs.princeton.edu/13stacks/) 的解释和实现.

需要注意的是, `multisets` 的成员是无序的, `{a,a,b}` 和 `{a,b,a}`
这两个集合在 `multisets` 看来是相等.

我们可以从两个角度来分析 `multisets` :

-   `multisets` 就好像一个=ArrayList&lt;E&gt;=, 只不过是无序的.
    当把它当作=ArrayList&lt;E&gt;=时:
    -   调用=add(E)=函数, 增加给定元素的出现次数
    -   调用=iterator()=函数, 获取一个 `multisets` 的迭代器,
        用来迭代每个元素
    -   调用=size()=函数, 获取所有元素出现次数之和

-   `multisets` 就好象一个=Map&lt;E, Integer&gt;=, 包含元素和对应的数量,
    只不过数量只能为正数. 当把它当作=Map&lt;E, Integer&gt;=的时候:
    -   调用=count(Object)=函数获取某个特定元素的出现次数.
    -   调用=entrySet()=函数返回一个=Set&lt;Multiset.Entry&lt;E&gt;&gt;=, 大概类似一个
        `Map` 返回 `entrySet` .
    -   调用 `elementSet` 函数返回一个=Set&lt;E&gt;=对象,
        返回所有的元素(去掉重复的元素)


### <span class="section-num">2.1</span> Multiset 的例子 {#multiset-的例子}

粗略介绍完 `Multiset` 之后, 现在就让我们用它重新实现原来的需求:

```java
@Test
public void testMultiset() {
    final String POTTER = "Potter";
    Multiset<String> bookstore = HashMultiset.create();
    bookstore.add(POTTER);
    bookstore.add(POTTER);
    bookstore.add("四体");
    bookstore.add("五体");
    Assert.assertTrue(bookstore.contains(POTTER));
    Assert.assertEquals(2, bookstore.count(POTTER));
    Assert.assertEquals(4, bookstore.size());

    bookstore.remove(POTTER);
    Assert.assertTrue(bookstore.contains(POTTER));
    Assert.assertEquals(1, bookstore.count(POTTER));
}
```

`multisets` 完美满足了我们的需求.


### <span class="section-num">2.2</span> Multiset 并不是一个 Map {#multiset-并不是一个-map}

需要注意的是, `Multiset` 虽然与=Map&lt;E, Integer&gt;=类似, 但 `Multiset`
并不是一个=Map&lt;E, Integer&gt;=, 请不要混淆它们两个.

最大的差别是, `Multiset` 实现了 `Collection` 接口, 完全遵守 `Collection`
接口需要满足的协议, 而 `Map` 和 `Collection` 是完全不同的接口,
这点需要牢记于心. 还有其他的差别, 诸如:

1.  =Multiset&lt;E&gt;=出现的次数只能是正数, 没有任何元素的出现次数会是负数的,
    出现次数为 0 的元素会被认为不存在,
    这样的元素是不会出现在=elementSet()=和=entrySet()=的返回结果中的.
    而=Map&lt;E, Integer&gt;=肯定不会有这样的限制.
2.  `multiset.size()=返回所有元素出现次数之和,
          如果想要知道有多少个不重复的元素, 可以使用=elementSet().size()`,
    例如={a,a,b}=, =elementSet.size()=返回结果是 2,
    =multiset.size()=返回结果是 3.
3.  =multiset.iterator()=用于迭代每个出现的元素,
    所以迭代次数和=multiset.size()=的值一样的.
4.  `Multiset<E>=支持增加元素, 删减元素, 或者通过 =setCount`
    函数直接设置元素的出现次数, `setCount(a, 0)=的意思等于将删除所有的
          =a` 元素.
5.  `multiset.count(elem)`: 如果元素 `elem` 不存在, 那么返回值总是 0. 而
    `Map` 对于不存在的元素, 返回的是 `null` .


### <span class="section-num">2.3</span> Multiset 实现 {#multiset-实现}

鉴于 `Multiset` 只是个接口, Guava 提供许多的接口实现, 大致可以与 Java
中的容器对应上:

| Map               | MultiSet                                                                                                                     | 支持 `null` 元素 |
|-------------------|------------------------------------------------------------------------------------------------------------------------------|--------------|
| HashMap           | [HashMultiset](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/HashMultiset.html)                     | Yes          |
| TreeMap           | [TreeMultiset](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/TreeMultiset.html)                     | Yes          |
| LinkedHashMap     | [LinkedHashMultiset](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/LinkedHashMultiset.html)         | Yes          |
| ConcurrentHashMap | [ConcurrentHashMultiset](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ConcurrentHashMultiset.html) | No           |
| ImmutableMap      | [ImmutableMultiset](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultiset.html)           | No           |


## <span class="section-num">3</span> Multimap {#multimap}

又来假设, 你是个班主任, 刚刚考完试, 你想记录下班里所有同学的成绩,
你可能写下这样的实现:

```java

Integer studentScore = 60;
String studentName = "Alan";
// 每个学生的成绩单
Map<String, List<Integer>> studentScoresMap = new HashMap<>();

// 如果 Alan 还没记录各科成绩的列表, 就新建一个列表
List<Integer> studentScores = studentScoresMap.get(studentName);
if (studentScores == null) {
    studentScores = new ArrayList<>();
    studentScoresMap.put(studentName, studentScores);
}

// 然后将某个科目的成绩加进去
studentScores.add(studentScore);
```

一个学生考试要考多个科目, 自然就会有多个学科成绩, 也就出现了一个 key
需要对应多个 value 的情况.
使用=Map&lt;K, List&lt;V&gt;&gt;=或者=Map&lt;K, Set&lt;V&gt;&gt;=这样的方式构建 key-values
自然可以, 只不过显得不甚优雅.

为此, Guava 提供了新的容器类型来应对一个
key 对应多个 values 的场景: `Multimap` . 同样的,
我们也可以从两个角度来理解 `Multimap` :

-   一个 `key` 对应一个 `value` , 同样的 `key` 可以存在多个:

<!--listend-->

```text
a -> 1
a -> 2
a -> 4
b -> 3
c -> 5
```

-   或者一个 `key` 对应一个列表的 `value` :

<!--listend-->

```text
a -> [1, 2, 4]
b -> [3]
c -> [5]
```

通常来说, 最好以第一种方式来理解 `Multimap` 接口,
不过你也可以以第二种方式来获取数据: `asMap()=函数, 返回一个
  =Map<K, Collection<V>>` 对象.

需要注意的是, 不存在 1 个 `key` 对应 0 个 `value` 的情况, 不会有空的值列表这样的说法, 要不一个 `key` 对应至少一个
`value` , 要不就是这个 `key` 不存在于这个 `Multimap` .

一般来说, 我们不会直接使用 `Multimap` 接口, 使用的是它的子接口; `Multimap`
接口提供了两个子接口: `ListMultimap` 和 `SetMultimap` , 大致类似于
`Map<K, List<V>>=和 =Map<K, Set<V>>`.


### <span class="section-num">3.1</span> Multimap 的例子 {#multimap-的例子}

现在让我们用 `Multimap` 重新实现一次学生不同科目的成绩单:

```java
String alan = "Alan";
String turing = "Turing";
// 创建一个 ListMultimap
ListMultimap<String, Integer> studentScoresMap = MultimapBuilder.hashKeys().arrayListValues().build();

studentScoresMap.put(alan, 95);
studentScoresMap.put(alan, 88);
studentScoresMap.put(turing, 100);

Assert.assertEquals(3, studentScoresMap.size());

List<Integer> alanScores = studentScoresMap.get(alan);
Assert.assertEquals(2, alanScores.size());
alanScores.clear();
Assert.assertEquals(0, studentScoresMap.get(alan).size());
```


#### <span class="section-num">3.1.1</span> 构造 {#构造}

细心的同学可能会发现, 上面创建 `ListMultimap`
的方式不是直接调用实现类的=.create()=函数, 而是使用 `MultimapBuilder` .

并不是 `Multimap` 的实现没有提供=.create()=方法, 是通过
`MultimapBuilder` 创建 `Multimap` 实现会更加便利一点,
使用=hashKeys()=函数创建的就是一个 `HashMap` ,
使用=treeKeys()=函数创建的就是一个 `TreeMap` .


#### <span class="section-num">3.1.2</span> 修改 {#修改}

`Multimap.get(key)=返回的就是特定 =key` 关联的集合, 对于一个
`ListMultimap` , 返回的就是一个 `List` ; 对于一个 `SetMultimap` ,
返回的就是一个 `Set` .

实际返回的是集合的引用, 所以对这个返回集合的操作,
将直接反馈在 `Multimap` 实例上. 如上面的例子所示, 把学生 `alan`
返回列表的数据清空, 在 `ListMultimap` 的数据也相应地被清空了.


### <span class="section-num">3.2</span> 视图 {#视图}

所谓的视图(Views), 我理解就是看待事物的方式和角度,
称为视图(或者视角'perspective').

`Multimap` 提供了若干个有用的视图:

1.  `asMap` 把 `Multimap<K,V>` 看作一个 `Map<K, Collection<V>>` ,
    返回一个的 map 对象支持 `remove` 操作, 但不支持 `put` 和 `putAll`
    操作.

    值得关注的是: 当对应的 key 不存在的时候, multiMap
    返回的是一个新构造的, 为空的集合, 如果你想在对应的 key
    不存在的时候返回空指针(就好像 HashMap 那样), 你可以通过
    `asMap().get(key)` 实现这样的效果

<!--listend-->

```java
studentScoresMap.asMap().remove(alan);
// 抛出 UnsupportedOperationException 异常
studentScoresMap.asMap().put("key", Lists.newArrayList());
// 抛出 UnsupportedOperationException 异常, 除非 anotherScores 是个空的 Map
studentScoresMap.asMap().putAll(anotherScores);
// 返回空的集合
Collection<Integer> Elons = studentScoresMap.get("Elon");
// 返回空指针
studentScoresMap.asMap().get("Elon");
```

-   `entries` 把 `Multimap` 内所有的记录(entry)看作
    `Collection<Map.Entry<K,V>>`, 如前文的 `studentScoresMap.entries()`
    返回的就是: `[{"Alan": 95}, {"Alan": 88}, {"Turing": 100}]`.
-   `keySets` 把 `Multimap` 内所有的不重复的 `key` 看作一个 `Set` .
    如前文的 `studentScoresMap.keySets()` 返回的就是:
    `Set(["Alan","Turing"])`.
-   `keys` 把 `Multimap` 内所有的 key 看作一个前文提到的 `Multiset` ,
    可以从这个 `Multiset` 删除元素, 但不能新增元素, 如前文的
    `studentScoresMap.keys()` 返回的就是:
    `Multiset(["Alan","Alan", "Turing"])`.
-   `values()` 把 `Multimap` 内所有的 value 看作一个集合, 相当于把所有 key
    对应的 value 集合串联起来, 如前文的 `studentScoresMap.values()`
    返回的就是: `[95, 88, 100]`


### <span class="section-num">3.3</span> Multimap 也不是一个 Map {#multimap-也不是一个-map}

严格来说, 即使 `Multimap` 名字中带有 map, 甚至 `map` 可能用来实现
`Multimap` , 但一个 `Multimap<K,V>` 终究不是一个
`Map<K, Collection<V>>`. 它们之间的差异包括:

1.  `Multimap.get(key)` 返回的对象总是不为空指针的, 即使查询的 key
    不存在, 返回的是个空的集合. 而 `Map.get(key)` 查询的 key 不存在,
    返回的就是空指针. 前文提到过, 如果想要让 `Multimap` 在查询 key
    不存在的时候返回空指针, 可以使用 `Multimap.asMap().get(key)`.
2.  `Multimap.containsKey(key)` 在 values 集合为空的时候就会返回 false,
    例如
    `studentScoresMap.putAll("elon", Lists.newArrayList()); Assert.assertFalse(studentScoresMap.containsKey("elon"))`,
    但对于 `Map<K, Collection<V>>` 而言, 返回的就会是 true, 因为 value
    不为 null.
3.  `Multimap.size()` 返回的是所有记录的总数的, 即把所有的 value
    的数量累加起来, 而 `Map<K, Colleciton<V>>` 返回的就是 key 对应的数量.


### <span class="section-num">3.4</span> 实现 {#实现}

`Multimap` 提供了若干个不同类型的实现, 你可以使用对应的实现来取代原来
`Map<K, Collection<V>>` 的地方:

| 实现                                                                                                                       | key 表现得类似... | value 表现得类似... |
|--------------------------------------------------------------------------------------------------------------------------|--------------|----------------|
| [ArrayListMultimap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ArrayListMultimap.html)         | HashMap       | ArrayList      |
| [HashMultimap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/HashMultimap.html)                   | HashMap       | HashSet        |
| [LinkedListMultimap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/LinkedListMultimap.html)       | LinkedHashMap | LinkedList     |
| [LinkedHashMultimap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/LinkedHashMultimap.html)       | LinkedHashMap | LinkedHashSet  |
| [TreeMultimap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/TreeMultimap.html)                   | TreeMap       | TreeSet        |
| [ImmutableListMultimap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableListMultimap.html) | ImmutableMap  | ImmutableList  |
| [ImmutableSetMultimap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableSetMultimap.html)   | ImmutableMap  | ImmutableSet   |

上述的实现, 除了不可变的实现之外, 其他都支持 `null key` 与 `null value`.
并非所有的实现底层用的都是 `Map<K, Collection<V>>`,
有好几个实现出于性能的考虑, 实现了自定义的 hash 表.

`Multimap` 还支持自定义 value 的集合形式, 如 List 形式或者 Set 形式, 详情可见
[`Multimaps.newMultimap(Map, Supplier<Collection>)`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newMultimap-java.util.Map-com.google.common.base.Supplier-)


## <span class="section-num">4</span> BiMap {#bimap}

继续假设, 你是个班主任, 你有个学生名字与学号的名单,
你有时会通过名字查询对应学号, 有时又会根据学号反查询学生名字, 通常来说,
你会这么实现这个名单:

```java
Map<String, String> nameToId = Maps.newHashMap();
Map<String, String> idToName = Maps.newHashMap();
nameToId.put("Linus", "0001");
idToName.put("0001", "Linus");
// 如果 0001 这个学号, 或者 Linus 这个名字已经存在了, 会发生什么事情呢?
// 会出现很微妙的 bug, 为了避免出现这种情况, 你需要手动维护这种限制
```

不得不说, 通过两个 Map 和实现 `value` 反查 `key` 的传统做法并不优雅,
即增加了心理负担, 又容易出 bug.

幸运的是, Guava 有一个名为 `BiMap` 类库, 提供了通过 `value` 也反查 `key` 的特性.
一个=BiMap&lt;K,V&gt;=是一个=Map&lt;K,V&gt;=, 提供了如下功能:

1.  允许通过 `inverse()` 函数调转 key-value, 从 `Map<K,V>` 变成
    `Map<V,K>`
2.  保证所有的 `value` 都是唯一的, `values()` 函数返回一个包含所有
    `value` 的 Set
3.  如果 `value` 已经存在, 那么 `BiMap.put(key,value)` 会抛出一个
    `IllegalArgumentException` 异常, 如果想强制删除掉原来的 `value` ,
    并插入一对新的 key-value, 可以使用
    [`Bimap.forcePut(key,value)`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/BiMap.html#forcePut-java.lang.Object-java.lang.Object-)


### <span class="section-num">4.1</span> BiMap 例子 {#bimap-例子}

让我们用 BiMap 来重新实现学生名字和学号的名单:

```java
BiMap<String, String> userId = HashBiMap.create();
userId.put("Linus", "0001");
String user = userId.get("Linus");
// 反向查询, 通过学号查询名字.
String idForUser = userId.inverse().get("0001");
// 抛出异常: java.lang.IllegalArgumentException: value already present: 0001
userId.put("RMS", "0001");
```


### <span class="section-num">4.2</span> BiMap 实现 {#bimap-实现}

| key-value map 实现 | value-key map 实现 | 对应的 BiMap                                                                                                   |
|------------------|------------------|-------------------------------------------------------------------------------------------------------------|
| HashMap          | HashMap          | [`HashBiMap`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/HashBiMap.html)           |
| ImmutableMap     | ImmutableMap     | [`ImmutableBiMap`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html) |
| EnumMap          | EnumMap          | [`EnumBiMap`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/EnumBiMap.html)           |
| EnumMap          | HashMap          | [`EnumHashBiMap`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/EnumHashBiMap.html)   |


## <span class="section-num">5</span> Table {#table}

假设还是个班主任, 现在你需要制作一个包含学号, 姓名与成绩的名单,
然后可以通过姓名或者学号进搜索, 你会怎么实现呢? 什么? 用 excel?
你好幽默啊!

```java
// key: 学号, value: {姓名: 成绩}
Map<String, Map<String, Integer>> studentScores = Maps.newHashMap();
Map<String, Integer> linus = Maps.newHashMap();
linus.put("Linus", 99);
studentScores.put("0001", linus);

// 通过学号获取成绩
Assert.assertEquals(1, studentScores.get("0001").size());
for (Map.Entry<String, Integer> element : studentScores.get("0001").entrySet()) {
    String name = element.getKey();
    Integer scores = element.getValue();
}

// 通过姓名获取成绩
for (Map.Entry<String, Map<String, Integer>> element : studentScores.entrySet()) {
    String id = element.getKey();
    Map<String, Integer> nameScores = element.getValue();
    if (nameScores.containsKey("Linus")) {
        Integer score = nameScores.get("Linus");
    }
}
```

不得不说, 用 `Map<R, Map<C, V>>` 的形式来实现多 key 搜索非常难受,
算法效率变为 O(n), 线性时间复杂度, 不但不优雅, 还容易出错,
如果我是班主任, 我就辞职了, 给我个 excel 不行么?

excel 是没有的了, 但是 Guava 提供了一个类 excel 的多 key 存储/搜索的容器: `Table`,
它支持以行和列维度搜索.


### <span class="section-num">5.1</span> Table 例子 {#table-例子}

让我们用 `Table` 重新实现一次可根据姓名与学号进行搜索的成绩单:

```java
// 从左到右各列分别是: 学号, 姓名, 成绩
Table<String, String, Integer> idNameScoreTranscript = HashBasedTable.create();
idNameScoreTranscript.put("0001", "Linus", 99);
idNameScoreTranscript.put("0002", "Aaron", 100);
idNameScoreTranscript.put("0001", "RMS", 98);
idNameScoreTranscript.put("0004", "RMS", 97);

/// 返回结果:
/// Linus: 99
/// RMS: 98
idNameScoreTranscript.row("0001");

/// 返回结果:
/// 0001: 98
/// 0004: 97
idNameScoreTranscript.column("RMS");
```

正常来说, 不会有两个学号一样的学生, 只是为了展示 `Table`
用法而这样造数据. `row`, `column` 函数可能让人比较迷惑,
这两个函数是怎么搜索的?

其实很简单, `row` 是以第一个 key 来搜索, 而 `column` 以第二个 key 来搜索, 如图:

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/guava_table_row.png" caption="<span class=\"figure-number\">Figure 2: </span>row: 以第一个 key 来搜索" >}}

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/guava_table_column.png" caption="<span class=\"figure-number\">Figure 3: </span>column: 以第二个 key 来搜索" >}}


### <span class="section-num">5.2</span> Table 视图 {#table-视图}

一往常, `Table` 也提供了若干个视图:

1.  `rowMap()`, 把 `Table<R, C, V>` 看作一个 `Map<R, Map<C, V>>`, 同样的,
    `rowKeySet()=返回一个=Set<R>`.
2.  `row(r)` 返回一个非空的 `Map<C, V>` 的引用, 对返回的 Map
    的修改也会反馈给持有该引用 `Table`.
3.  类似地, `column(c)` 返回一个非空的 `Map<R, V>` 的引用, 对返回的 Map
    的修改也会反馈给持有该引用 `Table`.
4.  `cellSet()` 把=Table&lt;R, C, V&gt;=看作一个 `Table.Cell<R, C, V>`, `Cell`
    与 `Map.Entry` 十分类似, 只不过它有两个 key, 形式是 `(r,c)=v`, 而
    `Map.Entry` 是 `key = value`.


### <span class="section-num">5.3</span> Table 实现 {#table-实现}

Table 依旧提供了若干个实现, 列表如下:

| Table&lt;R, C, V&gt;                                                                                           | 类似的 Map&lt;R, Map&lt;C, V&gt;&gt;                   |
|----------------------------------------------------------------------------------------------------------------|-----------------------------------------------------|
| [`HashBasedTable`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/HashBasedTable.html) | HashMap&lt;R, HashMap&lt;C, V&gt;&gt;                  |
| [`TreeBasedTable`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/TreeBasedTable.html) | TreeMap&lt;R, TreeMap&lt;C, V&gt;&gt;                  |
| [`ImmutableTable`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html) | ImmutableMap&lt;R, ImmutableMap&lt;C, V&gt;&gt;        |
| [`ArrayTable`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ArrayTable.html)         | ImmutableMap&lt;R, ImmutableMap&lt;C, V&gt;&gt;, 特别的一个 |


## <span class="section-num">6</span> ClassToInstanceMap {#classtoinstancemap}

目前, 我们介绍过的 Map, 无论是原生 Jdk 的 Map, 抑或是 Guava 的 Map,
`key` 都是同一个类型的.

这是因为 Map 的签名是 `Map<K,V>`, 实例的时候, 只能实例成某具体一个类型的参数. 所谓凡事都有例外, 有没有支持 `key`
是不同类型的 map 呢? 自然是有的, Guava 的
[`ClassToInstanceMap`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ClassToInstanceMap.html)
就可以支持多个类型的 key.

为什么它可以实现多个类型的 `key` 呢? 因为 `ClassToInstanceMap` 的签名声明为 `Map<Class<? extends B>, B>`,
通过传入不同类型的 `Class` 对象, 实现类型不同的 =key=(如果你要说,
即使传入不同类型的 Class 对象, 它只有一个 Class, 没有实现多个不同类型的
key 阿! 你也可以这样理解, well, 咬文嚼字就没有什么意义了)


### <span class="section-num">6.1</span> ClassToInstanceMap 例子 {#classtoinstancemap-例子}

```java
ClassToInstanceMap<Number> numberDefault = MutableClassToInstanceMap.create();
numberDefault.put(Integer.class, 10);
numberDefault.put(Long.class, 20L);
// 编译失败
//numberDefault.put(String.class, "string");
Assert.assertEquals(Long.valueOf(20L), numberDefault.getInstance(Long.class));
```

如果查看源码, 可以发现, `ClassToInstanceMap<B>` 只有一个类型参数 `B`:

```java
public interface ClassToInstanceMap<B> extends Map<Class<? extends B>, B>
```

很明显的, 类型 `B` 限制了 `key` 与 `value` 的类型。

对于 `value` 的限制,
就和常规的 `map` 一样; 而对于 `key` 而言, 泛型实例化时的参数类型只能是
`B`, 或者是 `B` 的子类, 例如: `ClassToInstanceMap<Number>`, 那么这个 map
的 `key` 类型必须是 `Number` 或 `Number` 的子类, 而传入的 `Integer` 和
`Long` 都是 `Number` 子类, 因此能编译通过。

如果传入的是 `String,` 不符合声明, 编译就报错了.

**需要注意的是, 和 `Map<Class, Object>` 一样, 一个 `ClassToInstanceMap` 可以包含着是原始类型的 `value`,
而原始类型与它对应的包装类型并不是同一种类型, 不要混淆了哦**


### <span class="section-num">6.2</span> ClassToInstanceMap 实现 {#classtoinstancemap-实现}

`ClassToInstanceMap` 提供了两个实现:

| ClassToInstanceMap                                                                                                                               | 类似的 Map                       |
|--------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------|
| [`MutableClassToInstanceMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MutableClassToInstanceMap.html) | Map&lt;Class, Object&gt;         |
| [`ImmutableClassToInstanceMap`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableClassToInstanceMap.html)         | ImmutableMap&lt;Class,Object&gt; |


## <span class="section-num">7</span> RangeSet {#rangeset}

目前为止, 我们介绍过的新类型容器都是常见的 Map/Set/Table,
现在我们就来介绍一个表示区间的容器: `RangeSet`. 一个 `RangeSet`,
表示一个包含无连接的, 不为空的区间的集合, 例如包含一个整数区间的
`RangeSet`: `{[1,5], [7,9)}`.

在 `RangeSet` 中, 区间是由类 `Range` 来表示的, 当把一个区间加入到一个可变的 `RangeSet` 时,
任何有交集的区间都会被合并, 为空的区间就会被忽略, 例如将区间 `[3,5]`
加入到已有的 `RangeSet` `{[2,4]}`, 就会被合并成 `{[2,5]}`,
这个也符合我们日常的生活经验.


### <span class="section-num">7.1</span> RangeSet 例子 {#rangeset-例子}

让我们现在来看一下 `RangeSet` 的两个例子, 一个是整数的 `RangeSet`

```java
RangeSet<Integer> rangeSet = TreeRangeSet.create();
// {[2,4]}
rangeSet.add(Range.closed(2, 4));
// {[2,5]}
rangeSet.add(Range.closed(3, 5));
// {[1,10]}
rangeSet.add(Range.closed(1, 10));
// 无连接上的区间: {[1,10], [11,15)}
rangeSet.add(Range.closedOpen(11, 15));
// 连接上的区间; {[1,10], [11,20)}
rangeSet.add(Range.closedOpen(15, 20));
// 空区间, 被忽略; {[1,10],[11,20)}
rangeSet.add(Range.openClosed(0, 0));
// 分割区间 [1,10]; {[1,5],[10,10],[11,20)}
rangeSet.remove(Range.open(5, 10));
```

在上面的例子中, `[2,4]` 和 `[3,5]` 这两个区间有交集,
所以它们被自动合并到一起了, 而对于区间 `[1,10]` 和 `[11,15)`, 10
相邻的整数就是 11, 但两个区间也没有合并起来, 因为它们没有相交,
如果想要他们合并起来, 可以手动调用 `Range.canonical(DiscreteDomain)`,
即:

```java
// {[1,10]}
rangeSet.add(Range.closed(1, 10).canonical(DiscreteDomain.integers()));
// 连接上的区间: {[1,15)}
rangeSet.add(Range.closedOpen(11, 15));
```

另外一个例子是日期的 `RangeSet`:

```java
RangeSet<LocalDate> rangeSet = TreeRangeSet.create();
// {[2019-10-10, 2019-12-25]}
rangeSet.add(Range.closed(LocalDate.parse("2019-10-10"), LocalDate.parse("2019-12-25")));
// {[2019-10-10, 2019-12-30)}
rangeSet.add(Range.closedOpen(LocalDate.parse("2019-12-24"), LocalDate.parse("2019-12-30")));
Assert.assertTrue(rangeSet.contains(LocalDate.parse("2019-10-20")));
// {[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)}
rangeSet.remove(Range.closed(LocalDate.parse("2019-10-20"), LocalDate.parse("2019-10-30")));
Assert.assertFalse(rangeSet.contains(LocalDate.parse("2019-10-20")));
// [2019-11-10,2019-11-25] 在 `{[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)}`的区间包围内
Assert.assertTrue(rangeSet.encloses(Range.closed(LocalDate.parse("2019-11-11"), LocalDate.parse("2019-11-20"))));
Assert.assertEquals(Range.closedOpen(LocalDate.parse("2019-10-10"), LocalDate.parse("2019-10-20")),
                    rangeSet.rangeContaining(LocalDate.parse("2019-10-19")));
// {[2019-10-10, 2019-12-30)}
Range<LocalDate> span = rangeSet.span();
Assert.assertEquals(LocalDate.parse("2019-10-10"), span.lowerEndpoint());
Assert.assertEquals(LocalDate.parse("2019-12-30"), span.upperEndpoint());
```

`RangeSet` 提供了若干个查询函数, 用法在上面的代码已经展示了,
查询函数列表:

-   `contains(C)`: `RangeSet` 最基础的查询操作, 判断任意的元素是否在
    `RangeSet` 内.
-   `rangeContaining(C)`: 与 `contains(C)` 类似, 判断任意的元素是否在
    `RangeSet` 内, 如果在的话返回一个对应的区间, 否则返回空指针. 如上代码,
    有 `RangeSet`: `{[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)}`,
    而元素 `2019-10-19` 在区间 `[2019-10-10, 2019-10-20)` 内, 因此
    `rangeContaining(C)` 函数返回的就是 `[2019-10-10, 2019-10-20)`.
-   `encloses(Range<C>)`: 判断任意的区间是否在 `RangeSet` 的包围中.
-   `span`: 返回一个最小区间, 包含 `RangeSet` 中的所有区间, 如有:
    `RangeSet`: `{[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)}`,
    `span` 函数返回的区间就是 `{[2019-10-10, 2019-12-30)}`.


### <span class="section-num">7.2</span> RangeSet 视图 {#rangeset-视图}

依照惯例, `RangeSet` 也提供了若干个视图:

-   `complement()`: 返回某个 `RangeSet` 的补集, 返回结果也是个 `RangeSet`,
    如有 `RangeSet`:
    `{[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)}`, 那它的补集就是:
    `RangeSet`:
    `{(-∞,2019-10-10), [2019-10-20,2019-10-30], [2019-12-30,+∞)}`,
    分别是三个区间: 负无穷到 `2019-10-10`, `2019-10-20` 到 `2019-10-30`,
    以及 `2019-12-30` 到正无穷.
-   `subRangeSet(Range<C>)`: 返回某个 `RangeSet` 相交的子区间, 如有
    `RangeSet`: `{[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)}`,
    取子区间 `[2019-11-10,2019-11-20]`, 那么返回结果就是
    `{[2019-11-10, 2019-11-20]}`; 如果取子区间 `[2019-10-15, 2019-11-20]`,
    那么返回结果就是
    `{[2019-10-10, 2019-10-20), (2019-10-30, 2019-11-20]}`
-   `asRanges()`: 把 `RangeSet` 当作一个 `Set<Range<C>>`, 如有 `RangeSet`:
    `{[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)}`, 返回结果就是:
    `Set({[2019-10-10,2019-10-20), (2019-10-30, 2019-12-30)})`


### <span class="section-num">7.3</span> RangeSet 实现 {#rangeset-实现}

`RangeSet` 提供了两个实现:

| `RangeSet`                                                                                                           | 类似的 Set&lt;Range&gt;   |
|----------------------------------------------------------------------------------------------------------------------|------------------------|
| [`TreeRangeSet`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/TreeRangeSet.html)           | TreeSet&lt;Range&gt;      |
| [`ImmutableRangeSet`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableRangeSet.html) | ImmutableSet&lt;Range&gt; |


## <span class="section-num">8</span> RangeMap {#rangemap}

既然能以区间集作为容器, 那么能否把区间当作 Map 的 `key` 呢?
答案是当然可以, Guava 就提供了一个这样的容器: [`RangeMap`](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/RangeMap.html).

需要注意的是, 不像 `RangeSet` 那样, 相邻或者相交的区间不能连接起来的,
即使毗邻的区间映射的是同一个 `value`.


## <span class="section-num">9</span> RangeMap 例子 {#rangemap-例子}

```java
RangeMap<Integer, String> rangeMap = TreeRangeMap.create();
// {[1,10] => "foo"}
rangeMap.put(Range.closed(1, 10), "foo");
// {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo"}
rangeMap.put(Range.open(3, 6), "bar");
// {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo", (10,20) => "foo"}
rangeMap.put(Range.open(10, 20), "foo");
// {[1, 3] => "foo", (3, 5) => "bar", (11, 20) => "foo"
rangeMap.remove(Range.closed(5, 11));
Assert.assertSame("foo", rangeMap.get(3));
Range<Integer> span = rangeMap.span();
Assert.assertEquals(span.lowerEndpoint(), Integer.valueOf(1));
Assert.assertEquals(span.upperEndpoint(), Integer.valueOf(20));
// {[12, 15]} => "foo"
RangeMap<Integer, String> subRangeMap = rangeMap.subRangeMap(Range.closed(12, 15));
Assert.assertEquals(subRangeMap.span().lowerEndpoint(), Integer.valueOf(12));
Assert.assertEquals(subRangeMap.span().upperEndpoint(), Integer.valueOf(15));
Assert.assertEquals("foo", subRangeMap.get(12));
```

`RangeMap` 提供的查询函数不多, 满打满算也只有 `get(K)` 和 `span`
这两个函数.


### <span class="section-num">9.1</span> RangeMap 视图 {#rangemap-视图}

`RangeMap` 提供的视图也不多, 只有两个:

-   `asMapOfRanges()`, 把 `RangeMap` 看作一个 `Map<Range<K>, V>`,
    可以用来遍历 `RangeMap`
-   `subRangeMap(Range<K>)`, 返回某个 `RangeMap`
    相关区间的子区间以及对应的 `value`, 如有 `RangeMap`:
    `{[1, 3] => "foo", (3, 5) => "bar", (11, 20) => "foo"`, 取子区间
    `[12,15]`, 返回结果就是 `{[12, 15]} => "foo"`; 如果取子区间=[4,12]=,
    返回结果就是: `{[4,5) => bar, (11, 12] => foo}`


### <span class="section-num">9.2</span> RangeMap 实现 {#rangemap-实现}

RangeMap 提供了两个实现:

| RangeMap                                                                                                           | 类似的 Map&lt;Range, V&gt;            |
|--------------------------------------------------------------------------------------------------------------------|------------------------------------|
| [TreeRangeMap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/TreeRangeMap.html)           | TreeMap&lt;Range, V&gt;               |
| [ImmutableRangeMap](https://guava.dev/releases/snapshot/api/docs/com/google/common/collect/ImmutableRangeMap.html) | ImmutableMap&lt;Range&lt;K, V&gt;&gt; |
|                                                                                                                    |                                       |


## <span class="section-num">10</span> 总结 {#总结}

介绍完新类型的容器之后, 希望大家对这些新类型容器熟悉起来, 应对需求来也能得心应手 :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

