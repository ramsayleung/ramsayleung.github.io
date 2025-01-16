+++
title = "归并排序算法改进"
description = "An introduction about how to improve merge-sort algorithms"
date = 2017-03-01T00:00:00+08:00
lastmod = 2022-02-23T22:16:39+08:00
tags = ["algorithm", "java"]
categories = ["algorithm"]
draft = false
toc = true
+++

最近笔者在阅读《[算法》](http://algs4.cs.princeton.edu/home/),重温经典数据结构和算法，毕竟一直以来的说法是程序就是数据结构＋算法归并算法所需的时间和N\*logN成正比，所以可以用归并算法处理数百万甚至更大规模的数据。

但是归并算法也是存在不足之处的，需要额外的空间来完成排序，而且空间和N的 大小也是成正比的


## <span class="section-num">1</span> 优化 {#优化}

《算法》中有提到可以通过一些细致的修改实现大幅度缩短归并排序的运行时间


### <span class="section-num">1.1</span> 对小规模子数组使用插入排序 {#对小规模子数组使用插入排序}

因为递归会使小规模问题中的方法被频繁调用，所以改进对它们的处理方法就能改进整个算法。

对于小数组可以使用插入排序或者选择排序来避免递归调用。[完整代码](https://github.com/samrayleung/AlgorithmsCode)


#### <span class="section-num">1.1.1</span> 未改进归并排序 {#未改进归并排序}

```java
public static  void sort(Comparable[] a,Comparable[] aux,int lo,int hi){
    if(hi<=lo){
	return;
    }
    int mid=lo+(hi-lo)/2;
    sort(a,aux,lo,mid);/*将左半边排序*/
    sort(a,aux,mid+1,hi);/*将右半边排序*/
    merge(a,aux,lo,mid,hi);
}
```


#### <span class="section-num">1.1.2</span> 改进后归并排序 {#改进后归并排序}

```java
public static void sort(Comparable[] a,Comparable[] aux,int lo,int hi){
    if(hi<=lo){
	return;
    }else if(hi-lo<15){
	insertionSort(a,lo,hi);
	return;
    }
    else {
	int mid = lo + (hi - lo) / 2;
	sort(a,aux, lo, mid);
	sort(a,aux, mid + 1, hi);
	merge(a,aux, lo, mid, hi);
    }
}
```

{{< figure src="http://algs4.cs.princeton.edu/22mergesort/images/mergesortTD-bars.png" caption="<span class=\"figure-number\">Figure 1: </span>轨迹图如上：(图来源于《算法》)" >}}

《算法》 中提到插入排序处理小规模的子数组(比如长度小于15) 一般可以将归并排序的运行时间缩短10%-15%. 实践出真知，还是要自己来测试一下更佳。


#### <span class="section-num">1.1.3</span> 测试1 {#测试1}

对100\*1000 个字符进行排序，结果如下：

{{< figure src="/ox-hugo/100*1000-1.png" link="/ox-hugo/100*1000-1.png" >}}


#### <span class="section-num">1.1.4</span> 测试2 {#测试2}

对1000\*1000 个字符进行排序，结果如下：

{{< figure src="/ox-hugo/1000*1000-1.png" link="/ox-hugo/1000*1000-1.png" >}}


#### <span class="section-num">1.1.5</span> 测试3 {#测试3}

对10000\*1000 个字符进行排序，结果如下

{{< figure src="/ox-hugo/10000*1000-1.png" link="/ox-hugo/10000*1000-1.png" >}}


#### <span class="section-num">1.1.6</span> 测试4 {#测试4}

对100000\*1000 个字符进行排序，结果如下

{{< figure src="/ox-hugo/1000000*1000-1.png" link="/ox-hugo/1000000*1000-1.png" >}}


#### <span class="section-num">1.1.7</span> 测试5 {#测试5}

对500000\*1000 个字符进行排序，结果如下

{{< figure src="/ox-hugo/500000*1000-1.png" link="/ox-hugo/500000*1000-1.png" >}}


#### <span class="section-num">1.1.8</span> 小结 {#小结}

由于篇幅问题，我无法将所有的测试结果都展示出来，但是从上面的结果，可以看出对于小数组，使用插入排序的确对性能有一定幅度提高(最开始的测试可能因为数据量太小所以结果误差较大，但是这并不妨碍得出一个比较接近的结果).

但是随着数据量的增大改进归并算法性能似乎开始下降 (未经过精确数据验证)


### <span class="section-num">1.2</span> 测试数组是否已经有序 {#测试数组是否已经有序}

可以添加一个判断条件，如果 a[mid]小于等于 a[mid+1],便可认为数组已经有序并跳过 `merge` 方法。这个改动不影响排序的递归调用，但是任意有序的子数组算法的运行时间都变成线性的了


#### <span class="section-num">1.2.1</span> 未改进归并排序 {#未改进归并排序}

```java
public static  void sort(Comparable[] a,Comparable[] aux,int lo,int hi){
    if(hi<=lo){
	return;
    }
    int mid=lo+(hi-lo)/2;
    sort(a,aux,lo,mid);/*将左半边排序*/
    sort(a,aux,mid+1,hi);/*将右半边排序*/
    merge(a,aux,lo,mid,hi);
}
```


#### <span class="section-num">1.2.2</span> 改进后归并排序 {#改进后归并排序}

```java
public static void sort(Comparable[] a,Comparable[] aux,int lo,int hi){
    if(hi<=lo){
	return;
    }else if(hi-lo<15){
	insertionSort(a,lo,hi);
	return;
    }
    else {
	int mid = lo + (hi - lo) / 2;
	sort(a,aux, lo, mid);
	sort(a,aux, mid + 1, hi);
	if(less(a[mid+1],a[mid])){
	    merge(a,aux,lo,mid, hi);
	}
    }
}
```


#### <span class="section-num">1.2.3</span> 测试1 {#测试1}

对100\*1000 个字符进行排序，结果如下：
[![](/ox-hugo/100*1000-2.png)](/ox-hugo/100*1000-2.png)


#### <span class="section-num">1.2.4</span> 测试2 {#测试2}

对1000\*1000 个字符进行排序，结果如下：
[![](/ox-hugo/1000*1000-2.png)](/ox-hugo/1000*1000-2.png)


#### <span class="section-num">1.2.5</span> 测试3 {#测试3}

对10000\*1000 个字符进行排序，结果如下：
[![](/ox-hugo/10000*1000-2.png)](/ox-hugo/10000*1000-2.png)


#### <span class="section-num">1.2.6</span> 测试4 {#测试4}

对100000\*1000 个字符进行排序，结果如下：
[![](/ox-hugo/100000*1000-2.png)](/ox-hugo/100000*1000-2.png)


#### <span class="section-num">1.2.7</span> 测试5 {#测试5}

对500000\*1000 个字符进行排序，结果如下：
[![](/ox-hugo/500000*1000-2.png)](/ox-hugo/500000*1000-2.png)


#### <span class="section-num">1.2.8</span> 小结 {#小结}

从上面的结果可以看出，只是添加了一个判断数组是否已经有序的条件，算法性能就优化了 大概20%左右(未经过精确数据验证), 不得不说真的令人惊讶。

****注意****:
运行结果跟操作系统，电脑配置，以及运行次数都相关，所以我使用的也只是很粗略的数据


### <span class="section-num">1.3</span> 参考 {#参考}

<http://algs4.cs.princeton.edu/home/h>

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

