+++
title = "Spock 一个优雅的Groovy/Java测试框架"
description = "an introduction about spock"
date = 2017-04-11T00:00:00+08:00
lastmod = 2022-02-23T17:51:48+08:00
tags = ["spock", "groovy", "test", "java"]
categories = ["java"]
draft = false
toc = true
+++

因为需要编写 RESTful api 测试的缘故，重拾了 Spock 这个适用于 Groovy/Java 的测试
框架，顺便把以前写的一篇旧文整理了一下，权当重温。


## <span class="section-num">1</span> 关于 Spock {#关于-spock}

Spock 是一个适用于 Java(Groovy) 的一个优雅并且全面的测试框架, 说 Spock 全面，是
因为 Spock 集成了现有的 Java 测试库；至于为什么赞美 Spock 优雅，阅读完全文你就会
有体会的了

{{< figure src="http://ww3.sinaimg.cn/large/cd613764jw1f71jlmu3hpj20i70eemy7.jpg" >}}

因为基于 Groovy, 使得 Spock 可以更容易地写出表达能力更强的测试用例。又因为它内置
了 Junit Runner, 所以 Spock 兼容大部分的 IDE，测试工具，和持续集成服务器。接下来
就介绍一下 Spock 的特性


## <span class="section-num">2</span> Spock 特性 {#spock-特性}

1.  内置支持 mocking stubbing，可以很容易地模拟复杂的类的行为
2.  Spock 实现了 BDD 范式(behavior-driven development)
3.  与现有的 Build 工具集成，可以用来测试后端代码，Web 页面等等
4.  兼容性强，内置 Junit Runner, 可以像运行 Junit 那样运行 Spock，甚至可以在同一个项
    目里面同时使用两种测试框架
5.  取长补短，吸收了现有框架的优点，并加以改进
6.  Spock 代码风格简短，易读，表达性强，扩展性强，还有更清晰显示 bug


## <span class="section-num">3</span> 为什么是 Spock {#为什么是-spock}

Spock 似乎有很多不错的特性，但是为什么有 Junit 这个那么强大的测试框架, 还要去
使用 Spock 呢? 甚至可以用 Spock 来代替 Junit 呢? 下面就用一些简单的例子来诠释
一下Spock 的强大. 以一个简单的加法为例：

{{< figure src="http://ww1.sinaimg.cn/large/cd613764jw1f71jm5dserj20ek04kmxc.jpg" >}}

Junit 的测试用例

{{< figure src="http://ww4.sinaimg.cn/large/cd613764jw1f71jmipwsmj20hy07xabc.jpg" >}}

Spock 的测试用例

{{< figure src="http://ww2.sinaimg.cn/large/cd613764gw1f71jn4vlzoj20nn09vmym.jpg" >}}

是否觉得耳目一新呢? 因为 Spock 支持以类人类语言的形式来定义方法名, 所以对比
Junit 的测试用例, 你会发现 Spock 的测试用例, 只需函数名, 就可以清晰了解这个测
试的用途

接下来, 再写一个乘法的类, 然后人为地加入一个 Bug, 再看看 Junit 和 Spock 的表现

{{< figure src="http://ww1.sinaimg.cn/large/cd613764gw1f71jnieycnj20lz047dfz.jpg" >}}

如果测试 fail, 会出现什么情况呢?

{{< figure src="http://ww1.sinaimg.cn/large/cd613764gw1f71jnxh6wtj20sq0gpdic.jpg" >}}

{{< figure src="http://ww2.sinaimg.cn/large/cd613764jw1f71job6us9j20p308iq42.jpg" >}}

显而易见，Junit 只是显示了结果不等，却没办法究竟判断是加法还是乘法出现了 bug，
但是 Spock 就很清晰地给出了答案。不难看出 Spock 的语法更加简洁, 优雅; 此外, 得
益于 Spock 独特的命名方式，只需查看函数名字便可以了解测试用例的目的，无需额外
的注释。而这只是 Spock 和 Junit 的一部分差异，其他的差异，接下来会继续说明。


## <span class="section-num">4</span> Spock 语法 {#spock-语法}


### <span class="section-num">4.1</span> Specification {#specification}

```java
class MyFirstSpecification extend Specification{
    //fields
    //fixture methods
    //feature methods
    //helper methods
}
```

Specification 是指一个继承于 **spock.lang.Specification** 的一个 Groovy 类. 而
Specification 的名字一般是跟系统或者业务逻辑有关的组合词，例如之前的AdderSpec


### <span class="section-num">4.2</span> Fields {#fields}

实例化一个类

```groovy
def obj = new ClassUnderSpecification()
def coll = new Collaborator()
```


### <span class="section-num">4.3</span> Feature Methods {#feature-methods}

Feature Methods 指具体的测试用例方法

```groovy
def "pushing an element on the stack"() {
    // blocks go here
}
```


### <span class="section-num">4.4</span> Fixture Methods {#fixture-methods}

```groovy
def setup() {}          // run before every feature method
def cleanup() {}        // run after every feature method
def setupSpec() {}     // run before the first feature method
def cleanupSpec() {}   // run after the last feature method
```

关于 Fixture Methods 的作用，笔者引用一下官方文档的一段话

> Fixture methods are responsible for setting up and cleaning up the environment in
> which feature methods are run. Usually it’s a good idea to use a fresh fixture for
> every feature method, which is what the setup() and cleanup() methods are for.
> All fixture methods are optional.

简而言之， **Fixture methodr** 是进行初始化或者收尾工作的。为了更好地理解 Spock
的特性，可以用 Spock 和 Junit 进行比较，(图截自官网)

{{< figure src="http://ww4.sinaimg.cn/large/cd613764jw1f71josjimij20sl0g7jsr.jpg" >}}

以上就是 Spock 的基本用法， 也只能说是中规中矩，难言惊艳。那么，接下来介绍的
就是 Spock **killer** 级别的特性了


### <span class="section-num">4.5</span> Blocks {#blocks}

关于 Blocks 的用法， 这里引用官网的一段话

> Spock has built-in support for implementing each of the conceptual phases of a
> feature method. To this end, feature methods are structured into so-called blocks.
> Blocks start with a label, and extend to the beginning of the next block, or the
> end of the method. There are six kinds of blocks: setup, when, then, expect,
> cleanup, and where blocks

简而言之， 这些内置的功能强大的 blocks, 就是帮助开发者编写单元测试的语法糖

{{< figure src="http://ww2.sinaimg.cn/large/cd613764jw1f71jpaxv52j20uv095q40.jpg" >}}

下面就了解一下不同 Block 的功能


#### <span class="section-num">4.5.1</span> The **given**: block {#the-given-block}

**given**: 应该包含所有的初始化条件或者初始化类，例如你可以把要测试的类的实例化放在
**given**. 总而言之， **given** 就是放置所有单元测试开始前的准备工作的地方


#### <span class="section-num">4.5.2</span> The **setup**: block {#the-setup-block}

**setup**: 笔者个人理解功能跟 **given** 很相似，所以初始化的时候可以二选一(笔者
个人推荐用 given，因为这样更符合 BDD 范式)


#### <span class="section-num">4.5.3</span> The **when**: blcok {#the-when-blcok}

**when**: 是 Spock 测试中最重要的一部分，这里放置的就是你要测试的代码，和你如
何测试的用例，这里的测试代码应该尽可能地短。有经验的 Spock 用户可以直接看
**when**: block 就了解测试流程了


#### <span class="section-num">4.5.4</span> The **then**: block {#the-then-block}

**then**: block 包含隐式的断言, 补充一下，Spock 是没有 assert 这个断言函数的，
Spock 使用的是 assertion, 笔者个人理解成这是一种隐式的断言。概括来说, **then**
就是放置你预期测试结果的地方。

现在已经把 given-when-then 粗略地解释了一下, 现在就通过代码阐述具体的用法.
首先确定一下需求; 假设现在要测试一个通过网站来销售电脑的电商平台, 如下图 (图
截自 java_test_with_spock 一书)

{{< figure src="http://ww1.sinaimg.cn/large/cd613764gw1f71jyfgad2j20t90alq3n.jpg" >}}

然通过模拟用户添加商品到购物车, 以展示 Spock 的用法

```java
public class Product{
    private String name;
    private int price;
    private int weight;
}
public class Basket{
    public void addProduct(Product product){
	addProduct(product,1)
	    }
    public void addProduct(Product product,int times){
	//some code about business
    }
    public int getCurrentWeight(){
	//
    }
    public int getProductTypesCount(){
	//
    }
}
```

然后编写 Spock 的测试用例

```groovy
def "A basket with one product has equal weight"(){

    given: "an empty basket and a Tv"
    Product tv=new Product(name:"bravia",price:1200,weight:18)
    Basket basket=new Basket()

    when:"user wants to buy the TV"
    basket.addProduct(tv)

    then:"basket weight is equal to the TV"
    basket.currentWeight==tv.weight
}
```

现在对 Spock 有一个初步的认识了。也可以使用 given-when-then 这 "三板斧" 来写
一些逻辑不是非常复杂的测试用例了。


#### <span class="section-num">4.5.5</span> The **and**: block {#the-and-block}

**and**: 它的用法有点像语法糖，它自己本身是没有什么功能，它只是拿来扩展其他的
功能的. 用上面的例子来解释一下用法:

```groovy
def "A basket with one product has equal weight"(){

    given: "an empty basket "
    Basket basket=new Basket()

    and: "several products"
    Product tv=new Product(name:"bravia",price:1200,weight:18)
    Product camera=new Product(name:"panasonic",price:350,weight:2)
    Product hifi=new Product(name:"jvc",price:600,weight:5)

    when:"user wants to buy the TV abd the camera and the hifi"
    basket.addProduct(tv)
    basket.addProduct(camera)
    basket.addProduct(hifi)

    then:"basket weight is equal to all product weight"
    basket.currentWeight==(tv.weight+camera.weight+hifi.weight）
}
```

从上面的代码可以看出，given 和 and 都用来进行类初始化，只是根据 Basket 和
Product 类型进行了细分。如下图

{{< figure src="http://ww1.sinaimg.cn/large/cd613764jw1f71jq4hwjlj20wt0e10uv.jpg" >}}

使用 **and** block 可以代码结构更简洁优雅. 此外, 如果 **and** 是紧跟在 **when** 后
面, 那么 **and** 就据有和 **when** block 一样的功能，依此类推


#### <span class="section-num">4.5.6</span> The **expect**: block {#the-expect-block}

**expect** 是一个很强大的特性，它用很多种用法，最常用的用法就是把
given-when-then 都结合起来

```groovy
def "An empty basket has no weight"(){

    expect:"zero weight when nothing is added"
    new Basket().currentWeight==0
}
```

或者是以下这种形式

```groovy
def "An empty basket has no weight(alternative)"(){

    given:"an empty basket"
    Basket basket=new Basket()

    expect:"that the weight is 0"
    basket.currentWeight==0
}
```

又或者用 **expect** 提前进行条件判断

```groovy
def "A basket with two products weights as their sum (precondition)"() {

    given: "an empty basket, a TV and a camera"
    Product tv = new Product(name:"bravia",price:1200,weight:18)
    Product camera = new Product(name:"panasonic",price:350,weight:2)
    Basket basket = new Basket()

    expect:"that nothing should be inside"
    basket.currentWeight == 0
    basket.productTypesCount == 0

    /* expect: block performs
     intermediate assertions*/

    when: "user wants to buy the TV and the camera"
    basket.addProduct tv
    basket.addProduct camera

    then: "basket weight is equal to both camera and tv"
    basket.currentWeight == (tv.weight + camera.weight)
    /* then: block examines
     the final result*/
}
```

上面那个例子是在添加产品之前检查初始化条件，这种情况下，能更容易看出是哪里测试 fail


#### <span class="section-num">4.5.7</span> The **clean**: block {#the-clean-block}

clean 就相当于在所有的测试结束以后执行的操作，例如，如果在测试中新建了 IO 流，
就可以在 clean 里面关闭 IO 流，那样就可以保证代码的正确性了


### <span class="section-num">4.6</span> Spock killer future {#spock-killer-future}

确定需求:（例子来自 Java_test_with_spock 一书），假设有一个核反应堆，这个反应
堆的系统组成：

-   多个烟雾感应器(输入)
-   3 个辐射感应器(输入)
-   现在的压力值(输入
-   报警器(输出)
-   疏散命令(输出)
-   通知操作员关闭反应堆(输出)
    系统如图

    {{< figure src="http://ww1.sinaimg.cn/large/cd613764jw1f71jsj8fqqj20vt0jlmzt.jpg" >}}

    系统相关设定:
-   如果压力值超过 150，报警器报警
-   如果 2 个或者更多的烟雾感应器被触发，那么报警器报警，通知操作员关闭反应堆
-   如果辐射值超过 100，警报器报警，通知操作员关闭反应堆，并马上疏散人群

输入输出对应关系

{{< figure src="http://ww4.sinaimg.cn/large/cd613764jw1f71jtabh47j20tm0k976p.jpg" >}}

现在，假如用 Junit 来写测试用例

```java
@RunWith(Parameterized.class)
public class NuclearReactorTest {
    private final int triggeredFireSensors;
    private final List<Float> radiationDataReadings;
    private final int pressure;

    private final boolean expectedAlarmStatus;
    private final boolean expectedShutdownCommand;
    private final int expectedMinutesToEvacuate;

    public NuclearReactorTest(int pressure, int triggeredFireSensors,
			      List<Float> radiationDataReadings, boolean expectedAlarmStatus,
			      boolean expectedShutdownCommand, int expectedMinutesToEvacuate) {

	this.triggeredFireSensors = triggeredFireSensors;
	this.radiationDataReadings = radiationDataReadings;
	this.pressure = pressure;
	this.expectedAlarmStatus = expectedAlarmStatus;
	this.expectedShutdownCommand = expectedShutdownCommand;
	this.expectedMinutesToEvacuate = expectedMinutesToEvacuate;

    }

    @Test
    public void nuclearReactorScenario() {
	NuclearReactorMonitor nuclearReactorMonitor = new NuclearReactorMonitor();

	nuclearReactorMonitor.feedFireSensorData(triggeredFireSensors);
	nuclearReactorMonitor.feedRadiationSensorData(radiationDataReadings);
	nuclearReactorMonitor.feedPressureInBar(pressure);
	NuclearReactorStatus status = nuclearReactorMonitor.getCurrentStatus();

	assertEquals("Expected no alarm", expectedAlarmStatus,
		     status.isAlarmActive());
	assertEquals("No notifications", expectedShutdownCommand,
		     status.isShutDownNeeded());
	assertEquals("No notifications", expectedMinutesToEvacuate,
		     status.getEvacuationMinutes());
    }

    @Parameters
    public static Collection<Object[]> data() {
	return Arrays
	    .asList(new Object[][] {
		    { 150, 0, new ArrayList<Float>(), false, false, -1 },
		    { 150, 1, new ArrayList<Float>(), true, false, -1 },
		    { 150, 3, new ArrayList<Float>(), true, true, -1 },
		    { 150, 0, Arrays.asList(110.4f, 0.3f, 0.0f), true,
		      true, 1 },
		    { 150, 0, Arrays.asList(45.3f, 10.3f, 47.7f), false,
		      false, -1 },
		    { 155, 0, Arrays.asList(0.0f, 0.0f, 0.0f), true, false,
		      -1 },
		    { 170, 0, Arrays.asList(0.0f, 0.0f, 0.0f), true, true,
		      3 },
		    { 180, 0, Arrays.asList(110.4f, 0.3f, 0.0f), true,
		      true, 1 },
		    { 500, 0, Arrays.asList(110.4f, 300f, 0.0f), true,
		      true, 1 },
		    { 30, 0, Arrays.asList(110.4f, 1000f, 0.0f), true,
		      true, 1 },
		    { 155, 4, Arrays.asList(0.0f, 0.0f, 0.0f), true, true,
		      -1 },
		    { 170, 1, Arrays.asList(45.3f, 10.3f, 47.7f), true,
		      true, 3 }, });

    }
```

各种输入输出数据以及 getter setter 耦合在一起，代码变得难读起来. 此外，除了可
读性， 还有更严重的问题，假如需求要增加一个输入或者增加一个输出呢， 就只能改
变数据结构， 这样的代码真的难以维护。不知道 Spock 的表现又如何呢？

```groovy
class NuclearReactorSpec extends spock.lang.Specification{

    def "Complete test of all nuclear scenarios"() {

	given: "a nuclear reactor and sensor data"
	NuclearReactorMonitor nuclearReactorMonitor =new NuclearReactorMonitor()

	when: "we examine the sensor data"
	nuclearReactorMonitor.feedFireSensorData(fireSensors)
	nuclearReactorMonitor.feedRadiationSensorData(radiation)
	nuclearReactorMonitor.feedPressureInBar(pressure)
	NuclearReactorStatus status = nuclearReactorMonitor.getCurrentStatus()

	then: "we act according to safety requirements"
	status.alarmActive == alarm
	status.shutDownNeeded == shutDown
	status.evacuationMinutes == evacuation

	where: "possible nuclear incidents are:"
	pressure | fireSensors | radiation             || alarm | shutDown | evacuation
	150      | 0           | []                    || false | false    | -1
	150      | 1           | []                    || true  | false    | -1
	150      | 3           | []                    || true  | true     | -1
	150      | 0           | [110.4f ,0.3f, 0.0f]  || true  | true     | 1
	150      | 0           | [45.3f ,10.3f, 47.7f] || false | false    | -1
	155      | 0           | [0.0f ,0.0f, 0.0f]    || true  | false    | -1
	170      | 0           | [0.0f ,0.0f, 0.0f]    || true  | true     | 3
	180      | 0           | [110.4f ,0.3f, 0.0f]  || true  | true     | 1
	500      | 0           | [110.4f ,300f, 0.0f]  || true  | true     | 1
	30       | 0           | [110.4f ,1000f, 0.0f] || true  | true     | 1
	155      | 4           | [0.0f ,0.0f, 0.0f]    || true  | true     | -1
	170      | 1           | [45.3f ,10.3f, 47.7f] || true  | true     | 3
    }

}
```

除了上面提及的 given-when-then 范式外，还多了一个之前没见过的 where block。现
在就来认识一下 Spock 的 killer 特性. 可以看到 Spock 的输入输出参数都保存在类
似表格的数据结构，其实这是 Spock 的 Parameterized tests，而在 **||** 符号左边的
是输入，右边的输出，每一列开始都是该参数的属性名，这样就可以很便捷地在 **then**
判断输出结果是否符合预期结果. 而数据添加或者减少输入参数或者输出结果的操作，
只需在 **where** block 里面对应地添加或者减少具体的参数，整个操作一目了然. 参数
的新增或者移除也很容易地实现


## <span class="section-num">5</span> 结语 {#结语}

笔者在项目中正是使用 Spock 编写测试， 或许对比 Junit, Spock 在流行度方面还难而
望其项背, 但是综合多方考虑，Spock 真的值得一试，兼之 Groovy 语言的语法加成，就
有一种在使用脚本编写 Java 的感觉 (好吧，笔者知道 Groovy 就是基于 jvm 的脚本)，
无需再为 Java 啰嗦的语法而烦恼。此外 Spock还有很多很强大的功能，例如内置的
Mocking Stubbing (Junit 需要第三方库支持), 还有支持企业级应用，Spring, Spring
boot, 和 Restful service 测试等。更多的用法，就要查阅官方文档了


## <span class="section-num">6</span> 参考 {#参考}

-   [Java Testing with Spock](https://www.amazon.com/Java-Testing-Spock-Konstantinos-Kapelonis/dp/1617292532)
-   [Spock Framework Reference Documentation](http://spockframework.org/spock/docs/1.1-rc-3/index.html)