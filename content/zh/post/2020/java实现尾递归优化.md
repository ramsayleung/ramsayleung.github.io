+++
title = "java8基于堆实现尾递归优化"
date = 2020-07-05T16:51:00+08:00
lastmod = 2022-02-26T10:11:32+08:00
tags = ["java"]
categories = ["java"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 前言 {#前言}

尾调用消除(tail call elimination, TCE)是函数式编程的重要概念, 有时也被称为尾调用优化(tail call optimization, TCO),
作用是将尾递归函数转化成循环, 避免创建许多栈帧, 减少开销.

遗憾的是, Java不支持TCE, 所以本文主要是介绍, 如何使用java8特性, 基于堆来实现尾递归优化.

---

一个有趣的事，这篇文章是我在阿里ATA上发的最后一篇文章。发在内网的第二天，也就是我的last day，有位P8的同事在钉钉上夸我文章写得好，只回复了一句，还未来得及多交流几句，我的离职流程就走完，钉钉被强制下线了，甚至没看到这位同事的回复。


## <span class="section-num">2</span> 尾调用与尾递归 {#尾调用与尾递归}

想要了解尾递归优化, 首先要了解下什么是尾调用.

尾调用的概念非常简单,
一言以蔽之, 指函数的最后一步是调用另一个函数. 以斐波那契数列为例:

```java
public int fac(int n) {
    if (n < 2) {
	return 1;
    }
    return n * fac(n - 1);
}
```

虽说上面的函数看起来像是尾调用函数, 但实际上它只是普通的递归函数,
因为它最后一步不是调用函数, 它只是作了加法计算, 上面的逻辑等同于:

```java
public int fac(int n){
    if(n < 2){
	return 1;
    }
    int accumulator = fac(n - 1);
    return n * accumulator;
}
```

既然调用 `fac(n-1)`函数的目的是为了获取累加值,
那么我们自然将累加值抽出来,
然后把上面的斐波那契数列函数改成尾调用函数呢:

```java
public int fac(int n) {
    return facTailCall(1, n);
}

public int facTailCall(int accumulator, int n) {
    if (n < 2) {
	return accumulator;
    }
    return facTailCall(n * accumulator, n - 1);
}
```

函数调用自身, 称为递归函数. 如果尾调用函数自身, 就称为尾递归函数.
那尾递归函数有什么用呢? 仅仅是将斐波那契数列的累加值抽了出来么?

要回答这个问题, 让我们先把目光投回到递归版本的斐波那契数列, 当调用
`fac(6)`时发生了什么事情:

```java
6 * fac(5)
    6 * (5 * fac(4))
    6 * (5 * (4 * fac(3)))
// N次展开之后
    6 * (5 * (4 * (3 * (2 * (1 * 1))))) // <= 最终的展开
// 到这里为止, 程序做的仅仅还只是展开而已, 并没有运算真正运运算, 接下来才是运算

    6 * (5 * (4 * (3 * (2 * 1))))
    6 * (5 * (4 * (3 * 2)))
    6 * (5 * (4 * 6))
// N次调用之后
    720 // <= 最终的结果

    fac(10000) // => java.lang.StackOverflowError
```

从上面的例子可以看出, 普通递归的问题在于展开的时候会需要非常大的空间,
这些空间指的就是函数调用的栈帧, 每一次递归的调用都需要创建新的栈帧,
递归调用有对应的深度限制, 这个限制就是栈的大小.

默认栈空间从32kb到1024kb不等, 具体取决于Java版本和所用的系统,
对于64位的java8程序而言, 递归的最大次数约为8000.

我们也没法通过增加栈的大小来增加递归的次数,
栈的大小相当于是一个全局配置, 所有的线程都会使用相同的栈,
增加栈的大小只是浪费资源而言.

那有没有方法可以避免上述的 `StackOverflowError` 呢? 那当然是有的,
答案就是上文提到的尾递归.

让我们来观察下尾递归版本的斐波那契数列, 看看调用 `facTailCall(1, 6)` 会发生什么事情?

```java
facTailCall(1, 6) // 1 是 fac(0) 的值
    facTailCall(6, 5)
    facTailCall(30, 4)
    facTailCall(120, 3)
    facTailCall(360, 2)
    facTailCall(720, 1)

    720 // <= 最终的结果

    facTailCall(1, 15000) // java.lang.StackOverflowError
```

与上方的普通递归函数相比, 尾递归函数在展开的过程中计算并且缓存了结果,
使得并不会像普通递归函数那样展开出非常庞大的中间结果,
**但是尾递归函数还是递归函数, 如果不作尾递归优化(TCO), 依然会出现
StackOverflowError**.

所谓的尾递归优化, 可以简单理解成将尾递归函数优化成循环; 在函数式编程中,
是鼓励大家使用递归, 而不是循环来解决问题.

这是因为循环会引入变量, 而变量是函数式编程中被视为洪水猛兽一样的存在.

但如果递归调用的深度比较大, 栈帧会开辟很多, 一来是浪费空间,
二来性能也必然会下降(有很多读写内存操作);

相反, 如果使用循环, 则只在一个函数栈空间里, 不会开辟更多的空间, 所以使用循环,
性能要好于递归.

所以在函数式编程语言中, 如Scheme, Haskell, Scala, 尾递归优化是标配, 所以不会出现 **StackOverflowError**

```lisp
(define (fact x)
    (define (fact-tail x accum)
	(if (= x 0) accum
	    (fact-tail (- x 1) (* x accum))))
  (fact-tail x 1))

(fact 1000000),  ;;; 返回一个很大很大的数, 使用的空间与(fact 3)相当
```

遗憾的是, Java并不支持尾递归优化.


## <span class="section-num">3</span> 基于堆的尾递归 {#基于堆的尾递归}

尾递归优化的一大用处是维持常数级空间, 保证不会爆栈.

既然爆栈的原因是栈空间不足, 又无法扩大栈的空间,
那么只能把函数存在其他地方, 比如堆(heap). 使用堆来抽象递归,
那么需要做的事情如下:

1.  表示一个函数的调用
2.  把函数调用存储在栈式结构中, 直到条件终止
3.  以后进先出(LIFO)的顺序调用函数

为此我们可以定义一个名为`TailCall`的抽象类, 它有两个子类:
其一表示挂起一个函数以再次调用该函数对下一步求值, 如下,
先暂停`f()`的调用, 先调用出`g()`的结果, 再对`f()`进行求值,
此子类名为`Suspend`:

```python
def f():
    return g() + 1
```

而一个函数的调用可以通过java8引入的`Supplier<T>`类来表示,
以此来存储函数, T为TailCall, 表示下一个递归调用.

这样一来, 就可以通过每个尾调用引用下一个调用的方式来构造一个隐式链表,
完成栈式数据结构存储的要求.

另一个子类表示返回一个调用, 它应该返回结果,
不会持有到一个TailCall的引用, 因为已经没有下一个TailCall了,
所以其名为`Return`.

其外, 还需要几个额外的抽象方法: 返回一个调用,
返回结果, 以及判断是否判断`TailCall`是`Suspend`还是`Result`,
接口及子类实现如下:

```java
/**
 * @author Ramsay/Ramsayleung@gmail.com
 * Create on 7/5/20
 */
public abstract class TailCall<T> {
    public abstract TailCall<T> resume();

    public abstract T eval();

    public abstract boolean isSuspend();

    public static class Return<T> extends TailCall<T> {
	private final T t;

	private Return(T t) {
	    this.t = t;
	}

	@Override
	public TailCall<T> resume() {
	    throw new IllegalStateException("Return has no more TailCall");
	}

	@Override
	public T eval() {
	    return t;
	}

	@Override
	public boolean isSuspend() {
	    return false;
	}
    }

    public static class Suppend<T> extends TailCall<T> {
	private final Supplier<TailCall<T>> resume;

	private Suppend(Supplier<TailCall<T>> resume) {
	    this.resume = resume;
	}

	@Override
	public TailCall<T> resume() {
	    return resume.get();
	}

	@Override
	public T eval() {
	    TailCall<T> tailCall = this;
	    while (tailCall.isSuspend()) {
		tailCall = tailCall.resume();
	    }
	    return tailCall.eval();
	}

	@Override
	public boolean isSuspend() {
	    return true;
	}
    }
    public static <T> Return<T> tReturn(T t){
	return new Return<>(t);
    }
    public static <T> Suppend<T> suppend(Supplier<TailCall<T>> supplier){
	return new Suppend<>(supplier);
    }
}
```

`Return`并没有实现`resume`方法, 只是简单地抛出了异常, 因为前文提到过,
`Return`表示最后一个调用, 没有下一个调用了, 自然无法实现`resume`方法;

同理, 只要不是最后一个调用, 就没法实现`eval()`方法,
因为最后的一个调用才能返回结果.

那为啥`Suspend`还实现了`eval`方法呢? 主要是不让用户感知函数调用并返回结果的逻辑, 将其内敛到`Suspend`内.
现在让我们来看看效果:

```java
/**
 * @author Ramsay/Ramsayleung@gmail.com
 * Create on 7/5/20
 */
public class TailCallTest {
    /**
     * 尾递归版本斐波那契数列
     */
    public int fac(int accumulator, int n) {
	return facTailCall(accumulator, n).eval();
    }

    public TailCall<Integer> facTailCall(int accumulator, int n) {
	if (n < 2) {
	    return TailCall.tReturn(accumulator);
	}
	return TailCall.suppend(() -> facTailCall(accumulator * n, n - 1));
    }

    /**
     * 递归版本的两数相加
     */
    public int addRecur(int x, int y) {
	return y == 0 ? x :
	    addRecur(++x, --y);
    }

    /**
     * 尾递归优化版本的两数相加
     */
    public int addTCO(int x, int y) {
	return addTailCall(x, y).eval();
    }

    public TailCall<Integer> addTailCall(int x, int y) {
	int _x_plus_one = x + 1;
	int _y_minus_one = y - 1;
	return y == 0 ? TailCall.tReturn(x) :
	    TailCall.suppend(() -> addTailCall(_x_plus_one, _y_minus_one));
    }

    @Test
    public void addTest() {
	addRecur(10, 10); // => 20
	addRecur(10, 10000); // StackoverFlowError
	addTCO(3, 100000); // => 100003
    }

    @Test
    public void test() {
	fac(1, 6); // => 720
	fac(1, 600000); // 数字过大溢出, 返回0, 且没有出现 StackOverflowError
    }

}
```


## <span class="section-num">4</span> 总结 {#总结}

至此, 我们通过java8的lambda, `Supplier`接口实现了基于堆的尾递归优化,
虽说没有优化成常数空间, 但终归解决了递归过深时, 栈空间不足导致
`StackOverflowError`的问题.

而按照[Stackoverflow问题的说法](https://stackoverflow.com/questions/53354898/tail-call-optimisation-in-java), java不支持尾调用的原因如下:

> In jdk classes there are a number of security sensitive methods that
> rely on counting stack frames between jdk library code and calling
> code to figure out who's calling them.

后续java版本也暂无支持尾递归优化的计划, 无奈摊手.jpg


## <span class="section-num">5</span> 参考 {#参考}

-   <https://en.wikipedia.org/wiki/Tail_call>
-   [Functional Programming in Java](https://book.douban.com/subject/26981273/)
-   [NightHacking with Venkat Subramaniam](https://youtu.be/4tEi86h8-TM?t=32m30s)
-   [Designing tail recursion using java 8](https://stackoverflow.com/questions/43937160/designing-tail-recursion-using-java-8)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

