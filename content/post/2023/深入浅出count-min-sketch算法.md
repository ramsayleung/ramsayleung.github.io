+++
title = "深入浅出Count-Min Sketch算法"
date = 2023-02-23T14:52:00+08:00
lastmod = 2023-02-24T17:41:47+08:00
tags = ["programming", "algorithm"]
categories = ["programming"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 引言 {#引言}

如果我们需要计算一系列元素的出现频次，应该怎么实现呢？一个简单的方案就是使用hashmap来记录每个元素出现的次数：

```c++
#include <vector>
#include <unordered_map>
#include <string>
std::unordered_map<std::string, int> count_freq(const std::vector<std::string>& elements){
  std::unordered_map<std::string, int> freq;
  for(const auto& e: elements){
    if(freq.find(e) == freq.end()){
      freq[e] = 1;
    }else{
      freq[e] +=1;
    }
  }
  return freq;
}
```

对于数据量不大的场景，这样的方案自然是可行的。

但假如我们要处理的 `elements` 是有几百亿数据量级的数据流，并且大部分的元素都是唯一的。
即使我们只关注最高频的元素，但长尾的情况还是会让 hashmap 的解决方案显得心有余而力不足。

毕竟数据流是无穷尽的，但机器内存却是有上限的，几百亿的key所需的内存，很容易就会让 hashmap 出现 `OutOfMemoryError` 了。

系统设计大部分时候都是在做取舍，采用时间换空间，以空间换时间等策略。

在当前的场景下，我们希望可以换取出空间，又不希望牺牲时间。盘了一下可以换取的属性，我们把目光投向了准确性(accuracy).

我们决定通过牺牲部分的准确性来节省空间，以支持使用固定size的内存，这也是Count-Min Sketch (缩写CMS) 这个概率数据结构(probabilistic data structure)最大的特点。

怎么听起来有点像 [Bloom Filter](https://en.wikipedia.org/wiki/Bloom_filter) 呢？没错，Count-Min Sketch就是从Bloom Filter 演进过来的。


## <span class="section-num">2</span> 通过hash函数来计数 {#通过hash函数来计数}

既然在大数据量下，记录元素需要大量的空间，既然我们希望使用固定 size 的内存空间，那么可否不存储元素本身，只存储记数呢？

回忆下 Bloom Filter 算法，通过哈希函数 `h` 将元素x 映射到一维数组上，然后通过判断一维数组上对应的索引 `h(x)` 是否为1来判断元素是否存在。

我们也可以借鉴下这个思路，通过 hash 函数和一个数组来记录元素的计数，数组的长度为 `w`。

{{< figure src="/ox-hugo/1d_array.png" link="/ox-hugo/1d_array.png" >}}

初始化，将数据每个元素初始化为0:

{{< figure src="/ox-hugo/initialization.png" link="/ox-hugo/initialization.png" >}}

递增元素 x 的计数，只需要将数组下标 `array[h(x)]` 对应的值递增1:

{{< figure src="/ox-hugo/increment.png" link="/ox-hugo/increment.png" >}}

获取元素 x 的计数，只需要获取数据下标 `array[h(x)]` 对应的值：

{{< figure src="/ox-hugo/retrieve.png" link="/ox-hugo/retrieve.png" >}}

设计思路看起来没有问题，但问题在于：所有用到 hash 函数的地方都需要考虑冲突（collision），而冲突就会导致计数比预期的大

{{< figure src="/ox-hugo/collision.png" link="/ox-hugo/collision.png" >}}

既然冲突会导致计数比预期大，那么我们有两个思路来控制这个问题：

1.  降低冲突概率
2.  提高计数的准确性


## <span class="section-num">3</span> 多个hash函数 {#多个hash函数}

让我们先来看下第二个思路，提高计数的准确性。既然使用一个 hash 函数，冲突会导致计数比预期大，那么我们可以采用多个（如 `d` 个） hash 函数，这些 hash 函数应该成对独立的。

对于元素 `x`，每次递增每个 hash 函数对应元素下标的值：  \\(\forall i \in \\{0..,d\\}: array[h\_i(x)] += 1\\)

{{< figure src="/ox-hugo/d_hash_funcs.png" link="/ox-hugo/d_hash_funcs.png" >}}

因为冲突会导到计数比预期大，因此在获取元素的计数值的时候可以取多个 hash 函数对应的最小值：

{{< figure src="/ox-hugo/d_hash_retrieve.png" link="/ox-hugo/d_hash_retrieve.png" >}}

我们已经触及Count-Min Sketch 算法的核心：通过取多个 hash 值中的最小值，来获取某个元素的计数值，这也是 Count-Min 这个算法名字的来源。

但除非我们使用更多空间，不然使用多个 hash 函数又会增加 hash 冲突的概率。

因为size是固定的，更多的 hash 函数就需要更新更多的索引，hash函数之间可能相互影响同一个索引，冲突的概率自然变大。

如果要使用更多空间减少冲突，要怎么增加呢？增大这个一维数组的size 么？


## <span class="section-num">4</span> 增加空间，减少冲突 {#增加空间-减少冲突}

因为在同一个数组中， d个hash 函数可能会相互影响，导致冲突的概念变大，所以增加这个一维数组的size也不能解决 hash 函数相互影响的问题。

那么我们可否给每个hash 函数一个单独的数组，这样就可以相互隔离影响了嘛。

{{< figure src="/ox-hugo/separate_array.png" link="/ox-hugo/separate_array.png" >}}

把 d 个数组合并起来，就会变成一个 _d x w_ 的二维数组：

{{< figure src="/ox-hugo/2d_matrix.png" link="/ox-hugo/2d_matrix.png" >}}

那么，让我们来看下如何通过操作这个二维数组，来记录某个元素的记数：

初始化：将整个矩阵 matrix 都初始化为0

{{< figure src="/ox-hugo/matrix_initialization.png" link="/ox-hugo/matrix_initialization.png" >}}

添加元素x: 使用 d 个 hash 函数对元素x 进行hash, 然后递增 `for i in d: matrix[i][hi(x)]+=1` 的值

{{< figure src="/ox-hugo/add_element_to_matrix.png" link="/ox-hugo/add_element_to_matrix.png" >}}

获取元素x的计数：取多个 hash 函数对应的最小值  `for i in d: min(matrix[i][hi(x)])`

{{< figure src="/ox-hugo/matrix_retrieve_element_x.png" link="/ox-hugo/matrix_retrieve_element_x.png" >}}

这个二维数组matrix 就是所谓的 `sketch` ，Count-Min 和 Sketch 合起来就是所谓的 Count-Min Sketch(CMS) 算法了。


## <span class="section-num">5</span> 模型调优 {#模型调优}

在上文的解释中我们已经提到过，因为哈希冲突的原因，我们获取到的是个近似值。

类似 Bloom Filter，这些概率型数据结构(probabilistic data structure), 得到的都是近似值。

虽说是近似值，但是它们都有确定的上界或下界，以 Bloom Filter 为例，如果某元素不存在 Bloom Filter，那么可以确定元素不存在；如果说元素存在于 Bloom Filter，那么有可能是因为 hash 冲突导致的假阳性，实际不存在。

对于Count-Min Sketch 而言，它只可能因为哈希冲突导致计数比实际的大，也就是说取出来的计数是上限，实际的值只可能小于或等于CMS 中存储的值。

那么问题就来了，如果我们想要设计一个（接近）最优的CMS, hash 函数数量 _d_ 和矩阵宽度 _w_ 应该怎么设置呢？

首先规定一些参数：

-   数据流的大小： _n_
-   元素x的真实计数值: \\(c\_x\\)
-   元素x 的估算计数值：\\(\hat{c\_x}\\)
-   hash 函数的个数 _d_, 以及矩阵的宽度 _w_

并引入两个变量：\\(\varepsilon\\) 和 \\(\delta\\) ,用处后面会提到。

前面提到，估算计数值 \\(\hat{c\_x}\\) 是\\(c\_x\\) 的上限，即 \\(c\_x \le \hat{c\_x} \\)，我们希望可以给 \\(\hat{c\_x}\\) 也设置一个上限，实际计数值与估计计数值的差距, 即误差值不大于 \\(\varepsilon n\\) ，即  \\(\hat{c\_x} \le c\_x + \varepsilon n\\), 可得出： \\(c\_x \le \hat{c\_x} \le c\_x + \varepsilon n\\)

实际情况可能不会100% 如我们所预期，因此我们可以定义结果在这个范围的概率：

\\(P(c\_x \le \hat{c\_x} \le c\_x + \varepsilon n) = 1 - \delta\\)

\\(1 - \delta\\) 表示结果在这个范围里的概率。

那么设计一个最优Count-Min Sketch模型的过程为：

1.  估计数据流的大小 _n_
2.  选择一个合理的 \\(\varepsilon\\) 值使 \\(hat{c\_x} - c\_x \le \varepsilon n\\)
3.  选择一个合理的概率值\\((1 - \delta)\\)
4.  _w_ 和 _d_ 的值可通过以下公式获得（来自论文）：

    \\(
         d = \lceil \frac{e}{\varepsilon} \rceil \\\ w=\lceil \ln(\frac{1}{\delta}) \rceil
         \\)

    可以看出，想要错误范围越小，就要更大的 _w_  ，也就是矩阵的宽度；

同理，想要更高的概率使结果符合预期（更小的 \\(\delta\\)），就要更大的 _d_ ，也就是更多的hash函数。

添加一个新哈希函数可以迅速降低超出边界异常数据的概率，这个降级效果是指数级别的；当然，增加矩阵的宽度也可以增加减少冲突的概率，但这个只是线性级别。

虽说根据常识，我们也可以得出这个结论 :)


### <span class="section-num">5.1</span> 例子 {#例子}

假设我们需要处理的数据量是\\(n = 10^{11}\\), 误差值不大于 \\(\varepsilon n=100000\\), 那么可得出 \\(\varepsilon = 0.000001\\), 由此我们可得出 _w_ = 2718281.

假如我们希望99%的概率落在这个范围内，可得出\\(\delta = 0.01\\)，因此hash函数数量是 \\(d=\ln({\frac{1}{0.01}})=5\\)


## <span class="section-num">6</span> 应用场景 {#应用场景}

Count-Min Sketch的最常用场景是在大数据量下实时计算最高频的若干元素（被称为Heavy Hitter Problem或TopK Problem），具体的业务场景如计算Youtube Trending, Twitter Trending或者DDOS时最高频的攻击ip列表.

对于这种大数据量，是无法使用HashMap 的，因为会导致单机内存不足，因此需要使用Count-Min Sketch 来记录每个元素出现的频次。

我画了下 TopK 问题的系统设计：

{{< figure src="/ox-hugo/topk_problem.png" link="/ox-hugo/topk_problem.png" >}}

为什么要使用Count-Min Sketch 来计算频次，而不使用Hadoop或流处理框架来解决这个问题？

主要是时效性的问题，如果要实现实时热榜，Hadoop和流处理框架可能就不够快；

如果对时效性要求不高，对准确性要求高，那就可以使用Hadoop框架。


## <span class="section-num">7</span> 总结 {#总结}

总而言之，CMS是用来计算频次近似值的概率型数据结构.

如果应用场景是大数据量，且对计数的准确性要求不高，那么就可以考虑使用 Count-Min Sketch.


## <span class="section-num">8</span> 参考 {#参考}

-   [An Improved Data Stream Summary: The Count-Min Sketch and its Applications](https://www.cse.unsw.edu.au/~cs9314/07s1/lectures/Lin_CS9314_References/cm-latin.pdf)
-   [Wikipedia: Count–min sketch](https://en.wikipedia.org/wiki/Count%E2%80%93min_sketch)
-   [Count-Min Sketch](https://florian.github.io/count-min-sketch/)
-   [Count-min Sketch 算法](https://zhuanlan.zhihu.com/p/369981005)
