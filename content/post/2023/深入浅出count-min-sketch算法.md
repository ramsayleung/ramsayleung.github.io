+++
title = "深入浅出Count-Min Sketch算法"
date = 2023-02-23T14:52:00+08:00
lastmod = 2023-02-23T22:46:35+08:00
tags = ["programming", "algorithm"]
categories = ["programming"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

如果我们需要计算一系列元素的出现频次，应该怎么实现呢？可以使用hashmap来计算，元素作为 key，每次递增：

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

对于数据量不大的场景，这样的方案自然是可行的。假如我们要处理的 `elements` 是有几百亿数据量级的数据流，并且大部分的元素都是唯一的，hashmap 的解决方案就显得心有余而力不足。

毕竟数据流是无穷尽的，但机器内存却是有上限的，几百亿的key所需的内存，很容易就会让 hashmap 出现 `OutOfMemoryError` 了。

系统设计大部分时候都是在做取舍，采用时间换空间，以空间换时间等策略。

在当前的场景下，我们希望可以换取出空间，又不希望牺牲时间。盘了一下可以换取的属性，我们把目光投向了准确性(accuracy).

我们决定通过牺牲部分的准确性来节省空间，以支持使用固定size的内存，这也是Count-Min Sketch (缩写CMS) 这个概率数据结构(probabilistic data structure)最大的特点。

怎么听起来有点像 [Bloom Filter](https://en.wikipedia.org/wiki/Bloom_filter) 呢？没错，Count-Min Sketch就是从Bloom Filter 演进过来的，虽说它们有着不同的应用场景。


## <span class="section-num">2</span> 通过hash函数来计数 {#通过hash函数来计数}


## <span class="section-num">3</span> 多个hash函数 {#多个hash函数}


## <span class="section-num">4</span> 减少hash冲突 {#减少hash冲突}


## <span class="section-num">5</span> 错误率 {#错误率}


## <span class="section-num">6</span> 应用场景 {#应用场景}


## <span class="section-num">7</span> 参考 {#参考}

-   [Wikipedia: Count–min sketch](https://en.wikipedia.org/wiki/Count%E2%80%93min_sketch)
-   [Count-Min Sketch](https://florian.github.io/count-min-sketch/)
