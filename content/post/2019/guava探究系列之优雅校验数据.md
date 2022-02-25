+++
title = "Guava探究系列之二: 优雅校验数据"
date = 2019-07-04T10:16:00+08:00
lastmod = 2022-02-25T22:20:19+08:00
tags = ["java", "guava"]
categories = ["guava"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

根据防御式编程的要求, 在日常的开发中, 总少不了对函数的各种入参做校验,
以便保证函数能按照预期的流程执行下去.

比如各种费率的值就没可能是负数, 如果费率出现负数, 所以数据有问题,
我们需要做的事情就是把这些有问题的数据挑出来.
自己手写这些校验函数未免过于繁琐, 所幸的是我们需要的函数已经有现成的:

Guava 提供了一系列的静态方法用于校验函数和类的构造器是否符合预期,
并称其为前置条件(preconditions). 如果前置条件校验失败,
就会抛出一个指定的异常.


## <span class="section-num">2</span> 前置函数特征 {#前置函数特征}

目前的前置校验方法有如下特征: 须需要, 下面例子中的`checkArgument`函数可以替换成任何一个前置条件校验函数

1.  这些前置方法一般接受一个布尔表达式作为入参，并判断表达是否为`true`,
    格式如:

<!--listend-->

```java
Preconditions.checkArgument(a>1)
// 如果表达式为false, 抛出IllegalArgumentException
```

2.  除了用于判断的布尔表达式之外,
    前置方法可以接受一个额外的`Object`作为入参, 在抛出异常的时候,
    把`Object.toString()`作为异常信息, 如:

<!--listend-->

```java
public enum ErrorDetail {
    SC_NOT_FOUND("404", "Resource could not be fount");
    // 省略部分内容
    @Override
    public String toString() {
	return "ErrorDetail{" + "code='" + code + '\'' + ", description='" + description + '\'' + '}';
    }
}

@Test
public void testCheckArgument() {
    Preconditions.checkArgument(1 > 2, ErrorDetail.SC_NOT_FOUND);
}

// 结果如下:
// java.lang.IllegalArgumentException: ErrorDetail{code='404', description='Resource could not be fount'}
```

3.  Guava的前置表达式还支持类似`printf`函数那样的格式化输出错误信息,
    只不过出于兼容性和性能的考虑, 只支持使用`%s`指示符格式化字符串,
    不支持其他类型. 如:

<!--listend-->

```java
int i=-1;
checkArgument(i >= 0, "Argument was %s but expected nonnegative", i);
// 结果如下:
// java.lang.IllegalArgumentException: Argument was -1 but expected nonnegative
```


## <span class="section-num">3</span> 前置条件函数介绍 {#前置条件函数介绍}

须注意的是, 下面介绍的`checkArgument`, `checkArgument`,
`checkState`函数都有三个对应的重载函数，分别对应前文所述的三种特征,
下文不会三种函数都介绍, 只介绍标准格式的前置条件函数.
以`checkArgument`函数为例, 三个重载函数分别是(忽略函数体):

```java
public static void checkArgument(boolean expression);
public static void checkArgument(boolean expression, @Nullable Object errorMessage);
public static void checkArgument(boolean expression,@Nullable String errorMessageTemplate,@Nullable Object... errorMessageArgs)
```


### <span class="section-num">3.1</span> checkArgument {#checkargument}

函数的签名如下:

```java
public static void checkArgument(boolean expression);
```

入参是一个布尔表达式, 函数校验这个表达式是否为`true`, 如果为`false`,
抛出`IllegalArgumentException`. 例子如下:

```java
@Test
public void testCheckArgument() {
    Preconditions.checkArgument(1 > 2);
}
```


### <span class="section-num">3.2</span> checkNotNull {#checknotnull}

这是个泛型函数, 函数签名如下:

```java
public static <T> T checkNotNull(T reference);
```

入参是个任意类型的对象, 函数校验这个对象是否为`null`, 如果为空,
抛出`NullPointerException`, 否则直接返回该对象,
所以`checkNotNull`的用法就比较有趣, 可以在调用`setter`方法前作前置校验.
例子如下:

```java
PreconditionTest caller = new PreconditionTest();
caller.setErrorDetail(Preconditions.checkNotNull(ErrorDetail.SC_INTERNAL_SERVER_ERROR));
```


### <span class="section-num">3.3</span> checkState {#checkstate}

函数签名如下:

```java
public static void checkState(boolean expression);
```

看着这个函数, 我个人感觉很奇怪:
这个函数和`checkNotNull`函数功能非常相似, 实现也基本一样,
都是判断表达式是否为`true`, 只是抛出的异常不一样而已,
是否有必要开发这个函数. 两个函数的实现如下:

```java
public static void checkArgument(boolean expression) {
    if (!expression) {
	throw new IllegalArgumentException();
    }
}

public static void checkState(boolean expression) {
    if (!expression) {
	throw new IllegalStateException();
    }
}
```

此外, 因为这两个函数相当类似, 就不展示相应例子了.


### <span class="section-num">3.4</span> checkElementIndex {#checkelementindex}

函数签名如下:

```java
public static int checkElementIndex(int index, int size);
```

这个函数用于判断指定数组, 列表, 字符串的下标是否越界, `index`是下标,
`size`是数组, 列表或字符串的长度, 下标的有效范围是`[0,数组长度)` 即
`0<=index<size`. 如果数组下标越界(即`index=<0 或者 =index=>==size`),
那么抛出`IndexOutOfBoundsException`异常, 否则返回数组的下标,
也就是`index`. 例子如下:

```java
Preconditions.checkElementIndex("test".length(), "test".length());
// 运行结果:
// 抛出异常: java.lang.IndexOutOfBoundsException: index (4) must be less than size (4)

Assert.assertEquals(3, Preconditions.checkElementIndex("test".length() - 1, "test".length()));
// 运行结果:
// 通过
```


## <span class="section-num">4</span> checkPositionIndex {#checkpositionindex}

函数的签名如下:

```java
public static int checkPositionIndex(int index, int size);
```

这个函数和`checkElementIndex`非常类似, 连Guava wiki的说明也基本一致(只有一个单词不同).

除了一点, `checkElementIndex`函数的下标有效范围是`[0, 数组长度)`, 而`checkPositionIndex`函数的下标有有效范围是`[0, 数组长度]`,
即`0<=index<=size`. 例子如下:

```java
Preconditions.checkPositionIndex("test".length() + 1, "test".length());
// 运行结果:
// 抛出异常: java.lang.IndexOutOfBoundsException: index (5) must be less than size (4)

Assert.assertEquals(4, Preconditions.checkPositionIndex("test".length(), "test".length()));
// 运行结果:
// 通过
```


### <span class="section-num">4.1</span> checkPositionIndexes {#checkpositionindexes}

函数的签名如下:

```java
public static void checkPositionIndexes(int start, int end, int size);
```

这个函数是用于判断`[start,end]`这个范围是否是个有效范围, 即`[start, end]` 是否在`[0, size]` 范围内(如果`[start, end]`
和`[0, size]`相同, 也认为在范围内), 如果不在, 则抛出`IndexOutOfBoundsException`异常. 例子如下:

```java
Preconditions.checkPositionIndexes(1, 3, 2);
// 运行结果:
// 抛出异常: java.lang.IndexOutOfBoundsException: end index (3) must not be greater than size (2)

Preconditions.checkPositionIndexes(0, 2, 2);
// 运行结果:
// 校验通过
```


## <span class="section-num">5</span> 前置条件在实际项目的应用 {#前置条件在实际项目的应用}

前置条件在检验条件不成交的时候抛的异常类型虽说是合情合理(比如,
`checkArgument`函数抛出`IllegalArgumentException`),

但是对于业务系统来说, 你抛出个`IllegalArgumentException`或者`NullPointerException`, 接口调用方对于这个异常摸不着头脑, 虽说只是正常的数据问题,
还是很容易觉得接口提供方服务出了问题, 甚至还会被质疑技术不过硬.

咱们又不是底层组件, 抛个`NPE`, 着实是不成体统. 基于各种有的没的的原因,
我们的业务系统在使用前置条件的时候进行了封装,
将前置条件抛出的异常进行了转换, 换成正常的业务异常, 提供完整的异常信息,
代码如下:

```java
// 封装代码:
public final class AssertUtils {
    /**
     * 检查条件表达式是否为真
     *
     * @param expression 条件表达式
     * @param errDetailEnum 错误码
     * @param msgTemplate 错误消息模板
     * @param vars 占位符对应变量
     * @throws BkmpException 条件表达式结果为假
     */
    public static void checkArgument(boolean expression, ErrDetailEnum errDetailEnum, String msgTemplate,
				     Object... vars) {
	try {
	    Preconditions.checkArgument(expression);
	} catch (IllegalArgumentException e) {
	    throw new BkmpException(errDetailEnum, msgTemplate, vars);
	}
    }

    /**
     * 检查条件表达式是否为假
     *
     * @param expression 条件表达式
     * @param errDetailEnum 错误码
     * @param msgTemplate 错误消息模板
     * @param vars 占位符对应变量
     * @throws BkmpException 条件表达式结果为假
     */
    public static void checkArgumentNotTrue(boolean expression, ErrDetailEnum errDetailEnum, String msgTemplate,
					    Object... vars) {
	try {
	    Preconditions.checkArgument(!expression);
	} catch (IllegalArgumentException e) {
	    throw new BkmpException(errDetailEnum, msgTemplate, vars);
	}
    }
}
// 省略其他部分的封装

// 调用例子:
AssertUtils.checkArgument(merchantEntity.exist(), ErrDetailEnum.DATA_NOT_EXIT, "商户不存在");
```


## <span class="section-num">6</span> Guava Precondition vs Apache Common Validate {#guava-precondition-vs-apache-common-validate}

自古文无第一, 武无第二, 文人之间的口水战总是少不了的.

没想到这不是国人的专利, 原来国外也有文人相轻的风气: Guava wiki
在介绍完preconditions之后, 还踩了一波竞品Apache Common Validate,
认为Guava的preconditions 比Apache Common 更加清晰明了, 也更加美观,

我个人对Apache Common Validate 了解不深, 也不好随意置喙. 除了踩竞品之外,
Guava wiki 还提了两点最佳实践(best practice):

1.  使用前置条件校验的时候, 推荐每个校验条件单独一行, 这样即更了然,
    出问题也更方便调试.
2.  使用前置条件校验的时候, 尽量提供有用的错误信息,
    这样可以更快地定位问题.


## <span class="section-num">7</span> 总结 {#总结}

代码大全一书有一章是关于防御式编程的, 用于提高程序的健壮性, 主要思想是子程序应该不因传入错误数据而被破坏，要保护程序免遭非法输入数据的破坏.

而Guava的preconditions 就是实现防御式编程的有力工具呢. oh yeah!


## <span class="section-num">8</span> 参考 {#参考}

-   [PreconditionsExplained](https://github.com/google/guava/wiki/PreconditionsExplained)
