+++
title = "Guava探究系列之一: 使用和避免使用空指针"
date = 2019-07-02T19:25:00-07:00
lastmod = 2025-01-09T18:45:12-08:00
tags = ["java", "guava"]
categories = ["Guava探究"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

> To be, or not to be, that is the question:

先来看看奆佬们关于空指针的看法:

> Null sucks - [Doug Lea](https://en.wikipedia.org/wiki/Doug_Lea)(JCP,Java并发编程实战作者, Java巨佬)
>
> I call it my billion-dollar mistake. - [Sir C. A. R. Hoare](https://en.wikipedia.org/wiki/C._A._R._Hoare), 空指针的发明者

按照Guava wiki的说法,
大部分的Google代码都是不支持使用空指针(下文用=null=表示空指针)的,

如接近95%的集合类都不支持使用=null=作为集合元素.
像Google这样的大公司明确不建议使用=null=自然是有其原由的, 不会无的放矢.
那具体原因是什么呢？待下文为你细细道来;


## <span class="section-num">2</span> 空指针的问题 {#空指针的问题}


### <span class="section-num">2.1</span> 空指针语意隐晦不明 {#空指针语意隐晦不明}

`null=的语意并不了然明确, 即当一个函数返回=null`, 我们并不知道=null=的意思是指返回结果理应为空? 还是指函数没有达到预期结果, 返回=null=表示失败?

举个常见的例子, 当调用=Map.get(key)=获取key对应的value的时候, 返回结果为=null=; `null=是指找不到这个key对应的value?
   还是说这个key对应的value本身就是=null`, 原来是通过=Map.put(key,value)=赋值的呢? =null=甚至可以是代表其他东西!

老实说, 当我们获得一个=null=, 我们并不清楚它究竟指的是啥,
除非有对应的javadoc 进行了说明.


### <span class="section-num">2.2</span> 空指针”暗藏杀机” {#空指针-暗藏杀机}

=null=除了语意不明外, 还非常容易在不经意间挖坑坑人. 例如有下面的代码:

```java
private String testNull(String input) {
    if (random.nextInt() % 2 == 0) {
        return input;
    } else {
        return null;
    }
}

@Test
public void useNull() {
    String foo = testNull("foo#bar").split("#")[0];
    String bar = testNull("foo#bar").split("#")[1];
}
```

可能你会说，这样的明显有坑的代码, 程序员理所当然会注意,
并对=null=指针进行校验的.

但事实并非如此, 因为=null=是一个特殊类型,
它可以表示一切的类型, 所以上面的代码是肯定可以编译通过的.
没有了编译器的约束, 只要使用=testNull=函数的时候没有查看源码,
或者源码非常复杂, 一下子理不清思路, 防御式编程落实不到,
就会忽略了=null=, 运行时就有可能抛出=NullPointException=, 导致程序crash.
这种情况真的防不胜防.


## <span class="section-num">3</span> Guava对于空指针的态度 {#guava对于空指针的态度}

因为上文提到或者隐藏但没提到的种种问题,
Guava的诸多类库在设计时就不支持=null=.

如果检测到=null=的存在, Guava的类库就会快速失败(fail fast),一般的处理策略是抛出异常.
虽说=null=存在种种的坑, 但=null=依旧是Java的一项关键特性,
因此Guava的类库也不能将=null=彻底拒之门外.

此外, Guava秉承既然不能消灭=null=, 那就把=null=建设得更好用的理念,
除了提供了一些工具可以让开发者避免使用=null=,
还提供了可以让开发者更易于使用=null=的工具.


## <span class="section-num">4</span> Optional {#optional}

在很多情况下, 程序员使用=null=是为了表示有些值可能存在或者不存在.
我们又可以用熟悉的=Map.get(key)=函数来举例,
如果规定=null=不能作为=value=值使用(但事实并非如此),
那么当这个函数返回=null=时就代表没有找到这个=key=对应的=value=.

为了应对这种使用=null=的情况, Guava团队参考其他语言(例如Scala)应对=null=的实践, 开发了=Optional&lt;T&gt;=类.
`Optional=类表示那些可能为空的值, 一个=Optional=类要不包含一个非空的=T=类型的对象引用(这种情况下,
  我们称引用对象是存在的-"present"), 要不什么东西都不包含(这种情况被, 我们说引用对象是不存在的-"absent"), 除此之外, =Optional=不存在其他情况, 更没有可能是=null`.


### <span class="section-num">4.1</span> Java8的Optional {#java8的optional}

鉴于我对=Optional=类的兴趣,
我用下面这条命令找了一个Guava库=Optional=开发的最初提交历史:

```sh
find guava/ -name "Optional.java" -print | xargs -I '{}' git log --pretty=tformat:%cd-%aN-%s --date=iso |tail -n2
# 结果如下
# 2009-09-15 19:50:59 +0000-kevinb@google.com-Initial code dump: version 9.09.15
# 2009-06-18 18:11:55 +0000-(no author)-Initial directory structure.
```

从Guava的commit历史中, 我们可以知道=Optional=最开始是在2009年开始开发的,
而10年前还是Java6的时代, Java7都尚未发布.

在那个”远古年代”, 是Guava的=Optional=一直引领着Java的抗击=null=重任,
为众多的蒙受”空指针之苦”的Java的程序员带来希望之光.

而当时光的脚步终于来到2014年3月18号, 在这一天, Java程序员迎来了Java8,
这是自Java5发布以来最激动人心的发布. 这天之后, 尘埃落定, `Optional`,
`Stream`, =Lambda=等诸多令人期待已久的特性终于成为Java的标准库的一部分,
而这也意味, Guava的=Optional=已经完成了自己的使命, 成为历史.

Guava的=Optional=类与JDK的=Optional=功能类似,
既然JDK的=Optional=已成为正统,
那么下面我就不再介绍Guava的=Optional=(Guava的wiki本来是有较大篇幅介绍自家的=Optional=,
个人感觉已经意义不大), 转而介绍JDK的=Optional=(下文通称为=Optional=).


### <span class="section-num">4.2</span> Optional构造方式 {#optional构造方式}

在使用=Optional=之前, 首先需要了解如果构造=Optional=对象,
方式有如下几种:


#### <span class="section-num">4.2.1</span> 声明一个空的=Optional=对象 {#声明一个空的-optional-对象}

可以通过静态工厂方法=Optional.empty=, 创建一个空的=Optional=对象:

```java
Optional<T> optional = Optional.empty();
```


#### <span class="section-num">4.2.2</span> 根据一个非空值创建Optional {#根据一个非空值创建optional}

还可以使用静态工厂方法=Optional.of=,
依据一个非空值创建一个=Optional=对象:

```java
Optional<T> optional = Optional.of(objectT);
```

需要注意的是, 按照=Optional=的源码声明, 如果传入的=objectT=为=null=,
那么=Optional=就会立刻抛出=NullPointException=(这就是快速失败-fail
fast), 而还是等到访问=optional=属性时才返回一个错误.

```java
/**
 * Returns an {@code Optional} with the specified present non-null value.
 *
 * @param <T> the class of the value
 * @param value the value to be present, which must be non-null
 * @return an {@code Optional} with the value present
 * @throws NullPointerException if value is null
 */
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}
```


#### <span class="section-num">4.2.3</span> 可接受null的Optional {#可接受null的optional}

最后, 使用静态工厂方法=Optional.ofNullable=,
我们可以创建一个允许=null=的=Optional=的对象:

```java
Optional<T> optional = Optional.ofNullable(objectT);
```

如果=objectT=为=null=, 那么得到的=Optional=对象就是个空对象.


### <span class="section-num">4.3</span> Optional的消费方式 {#optional的消费方式}


#### <span class="section-num">4.3.1</span> Optional与Stream的邂逅 {#optional与stream的邂逅}

既然=Optional=在Oracle的文档中被定性为一个容器(container),

那么对于一个容器,
我们关注的点无非是这个容器如何\*存\*(对于=Optional=来说是构造)和如何\*取\*这两件事而已(也就是消费).
在谈=Optional=的消费接口之前,
先来回顾一下Java8引进的=Stream=操作(关于Java8
=Stream=操作的说明已经汗牛充栋了, 既然珠玉在前, 我就不赘言了),
常用的=Stream=操作函数有如下几个:

-   filter
-   map
-   flatmap
-   peek
-   reduce
-   更多的函数可以参考[Oracle文档](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)

因为前文已经说过=Optional=是容器类, 那么按理来说,
正常容器类支持的=Stream=操作, =Optional=也支持.

只不过在Java8的时候, `Optional=只支持=filter`,=map=和=flatmap=这三个=Stream=操作.

可能是因为Java委员会的奆佬们也觉得=Optional=身为一个容器类只支持三个=Stream=操作有点丢人,
所以在Java9,
=Optional=增加了一个=Optional.stream()=这样一个可以返回=Stream=对象的函数,
让=Optional=拥有了容器类操作=Stream=的所有能力,
重振了身为一个容器的荣光. =Optional=与=Stream=结合使用的示例如下:

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
        .filter(car->car.getName().equals("Spaceship"))
        .flatMap(Car::getInsurance)
        .map(Insurance::getName)
        .orElse("Unknown");
```


#### <span class="section-num">4.3.2</span> 默认行为及解引用Optional对象 {#默认行为及解引用optional对象}

除了使用=Stream=来消费=Optional=对象,
还可以使用解引用读取=Optional=实例中的变量值以及定义默认行为,
具体函数说明如下:

1.  =get()=是这些方法中最简单但又最不安全的方法. 如果变量存在,
    它直接返回封闭的变量值. 否则就抛出一个=NoSuchElementException=异常.
    所以, 除非是非常确定=Optional=变量一定包含值,
    否则使用这个函数就相当容易踩坑. 此外,
    使用这个函数和直接进行=null=检查差别并不大.
2.  `orElse(T other)`
    该函数允许在=Optional=对象不存在的时候提供一个默认值(也是我个人最常用的使用方式之一)
3.  =orElseGet(Supplier&lt;? extends T&gt; other)=是=orElse=函数的延迟调用版,
    =Supplier=方法只有在=Optional=对象不含值的时候才执行.
    如果创建默认值是件耗时操作, 那么可以使用这种方式来提升性能,
    又或者某个函数仅在=Optional=为空的时候才调用, 也可以使用这种方式
4.  `orElseThrow(Supplier<? extends X> exceptionSupplier)`
    和=get=方法非常类似, 这两个函数都会在=Optional=对象为空时, 抛出异常,
    但差别在于=orElseThrow=可以指定抛出的异常类型
5.  =ifPresent(Consumer&lt;? super T&gt;)=和=orElseGet=函数类似,
    可以在变量存在的时候执行传入的函数, 否则就不进行任何操作.


#### <span class="section-num">4.3.3</span> Optional 实战示例 {#optional-实战示例}

在啰啰嗦嗦介绍了一系列=Optional=的概念之后,
是时候来看一下=Option=的实例了. 现存的Java
API几乎都是通过返回一个null的方式表示所需的值的缺失,
或者由于某些原因计算无法得到所需的值.

在上文, 我们已经给=null=盖棺定论了, `null=是有坑的, 甚至是有害的,
    所以要尽量少用=null`. 而现存的海量Java API都已经使用=null=作为返回结果,
我们没可能把这些API都重构成返回一个=Optional=对象的, 但眼看着=Optional=这样一个设计更完善无法在已有的Java API中使用未免令人心有不甘.

现实中, 可能我们无法修改这些API的签名, 但是我们却可以很轻易地用=Optional=对象对这些API的返回值进行封装.
现在还是用熟悉的=Map=举例, 假设有一个=Map&lt;String, Object&gt;=的对象, 在查询=key=对应的=value=时, 如果=value=不存在,
那么调用=Map.get(key)=就会返回一个=null=:

```java
Object value = map.get(key);
```

现在, 每次使用=value=都需要进行空指针判断, 着实是太繁琐.
为了解决这个问题, 可以使用=Optional.ofNullable=函数进行优化:

```java
Map<String, Object> map = new HashMap<>();
map.put("foo", "bar");
String value = Optional.ofNullable(map.get("foo")).map(Object::toString).orElse("helloworld");
```

这样, 每次使用=value=都不会再有=NullPointException=的忧虑.


## <span class="section-num">5</span> 结语 {#结语}

本文最开始只是想阐述Guava类库使用空指针和避免使用空指针的设计理念,
只是因为Guava大部分类库都是不支持=null=,
因此使用Guava自家的=Optional=类来代替=null=的大部分应用场景,
而Guava自家的=Optional=无可避免地被JDK的=Optional=取代,

所以本文大部份的内容也变成对JDK的=Optional=的探讨.
相信下篇文章会有所改观, 总不可能Guava所有的工具类, 都有JDK对应的竞品,
如果真是这样的话, JDK应该改名为GDK :)


## <span class="section-num">6</span> 参考 {#参考}

-   [Using and avoiding](https://github.com/google/guava/wiki/UsingAndAvoidingNullExplained)
-   [Java8 in Action](https://book.douban.com/subject/25912747/)
-   [Oracle java doc about Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
