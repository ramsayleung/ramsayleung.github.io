+++
title = "guava探究系列之四：不可变容器"
date = 2019-09-05T15:36:00-07:00
lastmod = 2025-01-09T18:49:30-08:00
tags = ["java", "guava"]
categories = ["Guava探究"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

先此声明, 个人倾向于将 `Collection` 翻译成容器, 将 `Set` 翻译成集合.

已经许久没有更新Guava研读系列的文章, 今天要介绍的是Guava的不可变容器.


## <span class="section-num">2</span> 关于不可变对象 {#关于不可变对象}

不可变的对象有许多的优点, 如下:

1.  线程安全, 可以在多线程之间使用也不用担心有竞争条件的风险
2.  可以放心地用于不被信任的第三方类库
3.  不用考虑支持可变性, 无需额外的空间和时间消耗.
4.  可用作常量使用

使用对象的不可变拷贝是一项良好的编程防御策略, 为此,
Guava提供了许多简单易用的, 实现了标准库 `Collection` 接口的不可变容器,
当然也包括实现了他们自家 `Collection` 接口的不可变容器.

虽然通过JDK的静态方法 `Collection.unmodifiableXXX` 可以使用内置不可变容器,
但是在Guava团队的同学看来,
它们有若干的不足(又到了喜闻乐见的黑JDK的环节):

1.  笨重; 使用起来很笨重, 不够赏心悦目和优雅.
2.  不安全;
    上述静态方法返回的容器只有在没有对象持有原来容器的情况下才是真正不可变的.
    例如,
    当想要通过可变Map `ids` 来生成一个不可变Map的时候, `Collections.unmodifiableMap(ids)`,
    如果有多个对象持有 `ids` 时, 静态方法返回的对象就不是真正的不可变.
    具体的分析可以参考[StackOverFlow关于unmodifiableMap和ImmutableMap的讨论](https://stackoverflow.com/questions/22636575/unmodifiablemap-java-collections-vs-immutablemap-google/22636674)
3.  低效; 静态方法生成的不可变容器和可变容器有着同样的性能开销,
    包括并发修改, 动态扩容等(对于真正的不可变容器而言,
    这些都是不会出现的操作)

综上所述, 如果你不想修改某个容器, 或者你想把某个容器当作不可变常量,
把这个容器变成一个不可变容器是一个很好的手段(使用Guava的不可变容器).

此外, 在之前的文章中, 我阐述过Guava对于空指针的态度是尽量不要使用空指针,
Guava的类库对于空指针都是快速失败的, Guava的不可变容器也是不例外的,
是拒绝接受空指针的.


## <span class="section-num">3</span> 代码实例 {#代码实例}

前面详细介绍了不可变容器, 现在是时候来看一下Guava不可变容器的代码例子:

```java
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
                                                                       "red",
                                                                       "orange",
                                                                       "purple");

class Foo {
    final ImmutableSet<Bar> bars;
    Foo(Set<Bar> bars) {
        this.bars = ImmutableSet.copyOf(bars); // defensive copy!
    }
}
```

前文提到的, `Collections.unmodifiableXXX(mutableXXX)` ,
Collections方法不能提供真正的不可变容器,
除非没有对象持有可变对象 `mutableXXX` 的引用

那么Guava的不可变容器又是否是真正的不可变呢? 以 `ImmutableSet` 为例,
发现所有可以修改 `ImmutableSet` 对象的操作函数,
包括 `add` , `remove` , `addAll` , `removeAll` 等函数都被重载,
然后标注成 `@Deprecated` ,
重载函数的内容就是抛出 `UnsupportedOperationException` 异常,
所以不可能修改 `ImmutableSet` 对象的内容:

```java
/**
 * Guaranteed to throw an exception and leave the collection unmodified.
 *
 * @throws UnsupportedOperationException always
 * @deprecated Unsupported operation.
 */
@Deprecated
@Override
public final boolean add(E e) {
    throw new UnsupportedOperationException();
}
```

至于持有 `mutableXXX` 对象引用,
修改 `mutableXXX` 对象内容导致不可变内容发生改变的情况也不会发生:

```java
@Test
public void testImmutable() {
    Set<String> colors = Sets.newHashSet();
    colors.add("blue");
    Set<String> modifiableSet = Collections.unmodifiableSet(colors);
    Set<String> unmodifiableSet = Collections.unmodifiableSet(new HashSet<>(colors));
    final ImmutableSet<String> COLOR_NAMES = ImmutableSet.copyOf(colors);
    colors.add("yellow");
    // 不会修改不可变集合的值
    Assert.assertFalse(COLOR_NAMES.contains("yellow"));
    // 修改引用导致集合值发生修改
    Assert.assertTrue(modifiableSet.contains("yellow"));
    // 因为没有对象持有new HashSet<>(colors)的引用, 所以unmodifiableSet是不可变集合, 不能修改
    Assert.assertFalse(unmodifiableSet.contains("yellow"));
    Assert.assertTrue(colors.contains("yellow"));
}
```

查看 `ImmutableSet.copyOf(Set<T>)` 函数的源码,
发现不可变集合的实现逻辑和在构造函数新建对象实现对象引用拷贝的逻辑一致,
即和 `Collections.unmodifiableSet(new HashSet<>(colors))` 的逻辑一样的:

```java
public static <E> ImmutableSet<E> copyOf(Collection<? extends E> elements) {
    /*
     * TODO(lowasser): consider checking for ImmutableAsList here
     * TODO(lowasser): consider checking for Multiset here
     */
    if (elements instanceof ImmutableSet && !(elements instanceof ImmutableSortedSet)) {
        @SuppressWarnings("unchecked") // all supported methods are covariant
            // 新建对象, 拷贝对象引用
            ImmutableSet<E> set = (ImmutableSet<E>) elements;
        if (!set.isPartialView()) {
            return set;
        }
    } else if (elements instanceof EnumSet) {
        return copyOfEnumSet((EnumSet) elements);
    }
    Object[] array = elements.toArray();
    return construct(array.length, array);
}
```


## <span class="section-num">4</span> 具体细节 {#具体细节}

下面我们来讨论一下各种不可变容器的具体使用细节.


### <span class="section-num">4.1</span> 构造不可变容器 {#构造不可变容器}

关于如何构造一个不可变容器, Guava提供的手段是多种多样的:

1.  使用 `copyOf` 静态方法, 例如 `ImmutableSet.copyOf(set)` ,
    这种构造方法与JDK不可变容器的构造方式类似 `Collections.unmodifiableXXX(mutableXXX)`
2.  使用 `of` 静态方法,
    例如 `ImmutableSet.of("a", "b", "c")=或者=ImmutableMap.of("a", 1, "b", 2)`,
    前文已经介绍过, 在此就不赘言
3.  使用 `Builder` 构造不可变容器, 例如:

<!--listend-->

```java
public static final ImmutableSet<Color> GOOGLE_COLORS =
    ImmutableSet.<Color>builder()
    .addAll(WEBSAFE_COLORS)
    .add(new Color(0, 191, 255))
    .build();
```

不过某些不可变容器的builder方法废弃了,
如 `ImmutableSortedSet` 的 `builder` 方法就被替换成了 `naturalOrder`.

此外, 对于有序容器(sorted collections)而言,
容器内的元素的顺序是按照构造时元素的插入顺序排列的, 例如如下代码

```java
final ImmutableSet<String> alphaTable = ImmutableSet.of("a", "b", "c", "a", "d", "b");
alphaTable.forEach(System.out::println);
// 结果为 a b c d
```


### <span class="section-num">4.2</span> `asList` 函数 {#aslist-函数}

所有的不可变容器都提供了一个 `asList` 方法来返回一个不可变列表 `ImmutableList`,
所以即使你把数据存在一个不可变有序集合 `ImmutableSortedSet`,
你也可以通过下标索引获取最小的元素或者第n小的元素, 如:

```java
final ImmutableSet<Integer> numberSet = ImmutableSortedSet.<Integer>naturalOrder()
    .add(2, 3, 1)
    .add(4, 5, 6).build();
numberSet.asList().get(0)
    # 结果为1
```


### <span class="section-num">4.3</span> 智能的 `copyOf` 函数 {#智能的-copyof-函数}

前文提到,
不可变容器都提供了一个 `copyOf` 方法用于从另外一个容器构造出一个不可变容器.
值得指出的是不可变容器的 `copyOf` 方法在不需要拷贝数据的时候就会尽量避免拷贝数据,
但这是什么意思呢? 假如有如下的代码:

```java
ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
thingamajig(foobar);

void thingamajig(Collection<String> collection) {
    ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
    ...
        }
```

在上面的代码调用 `ImmutableList.copyOf(foobar)` 函数的时候,
函数的内部实现不会逐个拷贝,
而会直接通过 `foobar.asList()` 函数返回一个不可变值列表,
这样实现的算法时间复杂度就是 `O(1)` , 而不是 `O(n)` , 实现性能消耗的最小化,
这也就是小标题智能指的意思.

但是需要注意的是,
并不是所有的不可变容器之间的转换都能实现 `O(1)` 时间复杂度,
例如 `ImmutableSet.copyOf(ImmutableList)` 就只能逐个元素拷贝,
时间复杂度退化到 `O(n)` .


## <span class="section-num">5</span> JDK容器与Guava不可变容器 {#jdk容器与guava不可变容器}

对于JDK提供的标准容器, Guava提供了相应的不可变容器实现,
对于Guava自家的容器, Guava也提供了对应的不可变容器, 具体实现对比如下:

| Interface                                                                                                   | JDK or Guava? | Immutable Version                                                                                                                                    |
|-------------------------------------------------------------------------------------------------------------|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Collection`                                                                                                | JDK           | [`ImmutableCollection`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableCollection.html)                 |
| `List`                                                                                                      | JDK           | [`ImmutableList`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableList.html)                             |
| `Set`                                                                                                       | JDK           | [`ImmutableSet`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSet.html)                               |
| `SortedSet=/=NavigableSet`                                                                                  | JDK           | [`ImmutableSortedSet`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSortedSet.html)                   |
| `Map`                                                                                                       | JDK           | [`ImmutableMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMap.html)                               |
| `SortedMap`                                                                                                 | JDK           | [`ImmutableSortedMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSortedMap.html)                   |
| [`Multiset`](https://github.com/google/guava/wiki/NewCollectionTypesExplained#Multiset)                     | Guava         | [`ImmutableMultiset`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultiset.html)                     |
| `SortedMultiset`                                                                                            | Guava         | [`ImmutableSortedMultiset`](http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/ImmutableSortedMultiset.html)             |
| [`Multimap`](https://github.com/google/guava/wiki/NewCollectionTypesExplained#Multimap)                     | Guava         | [`ImmutableMultimap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultimap.html)                     |
| `ListMultimap`                                                                                              | Guava         | [`ImmutableListMultimap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableListMultimap.html)             |
| `SetMultimap`                                                                                               | Guava         | [`ImmutableSetMultimap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSetMultimap.html)               |
| [`BiMap`](https://github.com/google/guava/wiki/NewCollectionTypesExplained#BiMap)                           | Guava         | [`ImmutableBiMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html)                           |
| [`ClassToInstanceMap`](https://github.com/google/guava/wiki/NewCollectionTypesExplained#ClassToInstanceMap) | Guava         | [`ImmutableClassToInstanceMap`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableClassToInstanceMap.html) |
| [`Table`](https://github.com/google/guava/wiki/NewCollectionTypesExplained#Table)                           | Guava         | [`ImmutableTable`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html)                           |


## <span class="section-num">6</span> 总结 {#总结}

因为不可变容器不会在运行时改变他们的内部状态, 所以他们是线程安全和无副作用的.

因为这些属性, 不可变容器在多线程环境就会变得特别有用, 可以安全地传递数据. 总而言之,
生活和工作或许可以多拥抱变化, 对于代码, 最好还是多保持不变地好.


## <span class="section-num">7</span> 参考 {#参考}

-   [Immutable
    Collections](https://github.com/google/guava/wiki/ImmutableCollectionsExplained)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

