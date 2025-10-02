+++
title = "Telegram频道精华贴检索机器人"
date = 2025-10-01T21:55:00-07:00
lastmod = 2025-10-01T23:17:02-07:00
tags = ["telegram", "rust"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 引言 {#引言}

我有时总会觉得自己是个「奇葩」：
在现在各种算法推荐大行其道的当下，我却偏偏不喜欢算法推荐，因为这种「千人千面」的模式，总让我担心自己会陷入「信息茧房」的焦虑之中，此外我也希望可以看到一些我兴趣之外的内容。

我算是很多年的 Twitter 老用户了，使用 Twitter 已经超过十年了，只是在马斯克收购 Twitter,新增了一个 "For You" 的推荐流之后，越来越没有兴趣看，不是吵架就是借极端观点/话题引流。

于是，我转向了Telegram频道，并为此打造了一个专属的「挖掘好帖」工具。

在最近的一年时间，我订阅了越来越多的 Telegram 频道，频道主可以向所有的订阅者广播内容，因为 Telegram 频道都是由频道主发布的，这给了我一种主动发现、主动阅读，而非被动接受算法推荐的感觉。

有频道主分享 twitter 上的内容时，也可以算是某种意义的「编辑精选」了。

当我订阅越来越多的频道之后，发现有些频道主太能聊了，一天能发几十个帖子，有些只是碎碎念，有些却是思考和精华。

所以我就在想，是否有个好用的挖宝藏（挖坟）工具，可以把频道的热贴，精华帖都列出来呢？


## <span class="section-num">2</span> 灵感 {#灵感}

另外一个我高频使用的网站就是 [Reddit](https://old.reddit.com), 不了解的 Reddit 的朋友可以理解成它是网站的百度贴吧，可以有不同的子版（subreddit），相当于不同的吧, 然后每个子版都有一个 `top` 的功能，可以按照过去一小时，过去24小时，过去一周，过去一年，历史全时段按帖子的顶贴数进行排序。

类似 `rust` 这个子版的历史精华帖：<https://old.reddit.com/r/rust/top/>

那么，我是否也可以对 Telegram 频道也实现类似的功能呢？

我可以把某个频道所有的帖子都爬下来，然后按点赞数，转发数，点击数按不同的时间段进行排序，

支持不同的时间段：包括一周内，一个月内，一年内，和历史所有帖子

相当于根据频道的订阅者的用手投票，挑选出不同时段最精华的帖子


## <span class="section-num">3</span> 技术挑战 {#技术挑战}

仔细分析之后，我发现检索频道并构建热榜是个相当有趣的技术问题，也与频道帖子本身的产品属性有关。

在我爬取某个频道的所有帖子后，如何与频道的最新动态保持同步更新呢？

毕竟像阅读数，转发数这些数据也是一直会更新的。

但是大部分历史的帖子的阅读数据都是不怎么会变化的，比如半年前的帖子可能就不会有人去翻看，订阅者大多只会关注最新的帖子。

每次都所有帖子都重新爬取一次固然可行，但是效率太低，毕竟绝大部分的帖子的数据都是不怎么变动的，怎么平衡性能与数据的及时性呢？

有点像超简化版本的搜索引擎问题了。

查询的时候要支持不同的时间段，按不同的指标进行排序，如何保证查询的性能呢？

总不能每次查询都重新计算一次吧，这样在帖子非常多的热点频道，就会出现查询的性能瓶颈。


## <span class="section-num">4</span> 解决方案 {#解决方案}

在仔细考量，权衡利弊之后，我取了个折衷的方案：

1.  第一次爬取某个频道时，全量爬取并索引
2.  每天定时重新爬取最近30天的帖子数据，作增量更新
3.  每周再爬取和索引一次全量数据

就这样兼顾性能，成本以及数据及时性，又不会造成过多的重复爬取。

而对于查询性能，我选择了建立物化视图(material view)的策略, 每次爬取成功之后就重新计算，更新一下 material view, 耗时可能比较长，每个频道更新视图大概需要个十几秒。

但此后所有的查询都直接指向这个预计算好的视图，数据库还可以利用缓存优化，以空间换时间，查询性能因此得到保障


## <span class="section-num">5</span> 使用示例 {#使用示例}

使用方法非常简单：

1.  点击链接打开机器人：<https://t.me/tele_ranker_bot>
2.  在对话框中输入 `/rank <频道链接>` ，例如 `/rank https://t.me/pipeapplebun`
3.  首次检索一个频道时，机器人需要一些时间来建立索引（如下图所示），完成后会通知用户：

{{< figure src="/ox-hugo/first_index.jpg" >}}

完成之后就会发消息通知用户

{{< figure src="/ox-hugo/index_finish.jpg" >}}

然后用户就可以按照点赞数，转发数，点击数查看帖子

{{< figure src="/ox-hugo/rank1.jpg" >}}

{{< figure src="/ox-hugo/rank2.jpg" >}}

通过这个工具，我甚至发现了一些有趣的现象：

某些频道看似有十几万订阅者，但近期帖子的点击数仅有一千多，还不到百分之一，数据真实性令人存疑。


## <span class="section-num">6</span> 结语 {#结语}

有了这个频道检索工具，我终于能在一个频道里高效「挖坟」了，再也不用担心错过沉淀在时间线里的精华。

不管是想回顾精华，还是挖掘隐藏好帖，甚至是做简单的数据挖掘，这个机器人都能搞定，举个例子：

-   找最近一个月点赞最多的帖子
-   看看历史上点击最高的内容有哪些

这次，我总算为自己、也为可能有同样需求的你们，打造了一个称手的工具。

-   立即体验：<https://t.me/tele_ranker_bot>
-   我的频道（菠萝油与天光墟）: <https://t.me/pipeapplebun>


### 推荐阅读 {#推荐阅读}

-   旅加经历
    -   [这些年走过的路：从广州到温哥华](https://ramsayleung.github.io/zh/post/2023/%E8%BF%99%E4%BA%9B%E5%B9%B4%E8%B5%B0%E8%BF%87%E7%9A%84%E8%B7%AF_%E4%BB%8E%E5%B9%BF%E5%B7%9E%E5%88%B0%E6%B8%A9%E5%93%A5%E5%8D%8E/)
    -   [加拿大之初体验](https://ramsayleung.github.io/zh/post/2023/%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B9%8B%E5%88%9D%E4%BD%93%E9%AA%8C/)
    -   [登陆加拿大一年后的体会](https://ramsayleung.github.io/zh/post/2024/%E7%99%BB%E9%99%86%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B8%80%E5%B9%B4%E7%9A%84%E4%BD%93%E4%BC%9A/)
-   工具流分享
    -   [简明写作指南](https://ramsayleung.github.io/zh/post/2024/%E7%AE%80%E6%98%8E%E5%86%99%E4%BD%9C%E6%8C%87%E5%8D%97/)
    -   [我的写作流](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E5%86%99%E4%BD%9C%E6%B5%81/)
    -   [我的画图流：画图工具与技巧分享](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E7%94%BB%E5%9B%BE%E6%B5%81/)
    -   [我的搜索流：高效搜索经验分享](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E6%90%9C%E7%B4%A2%E6%B5%81/)
    -   [最好的学习方式：费曼学习法(Feynman Technique)](https://ramsayleung.github.io/zh/post/2022/feynman_technique/)
    -   [系统思考：既见树木，又见森林](https://ramsayleung.github.io/zh/post/2021/%E7%B3%BB%E7%BB%9F%E6%80%9D%E8%80%83/)
-   思考感悟
    -   [编程十年的感悟](https://ramsayleung.github.io/zh/post/2024/%E7%BC%96%E7%A8%8B%E5%8D%81%E5%B9%B4%E7%9A%84%E6%84%9F%E6%82%9F/)
    -   [那些年，我从微信支付学到的东西](https://ramsayleung.github.io/zh/post/2023/%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%98%E7%A6%BB%E7%BA%BF_%E6%88%91%E5%B8%A6%E8%B5%B0%E4%BA%86%E4%BB%80%E4%B9%88/)
    -   [杂谈ai取代程序员](https://ramsayleung.github.io/zh/post/2025/%E6%9D%82%E8%B0%88ai%E5%8F%96%E4%BB%A3%E7%A8%8B%E5%BA%8F%E5%91%98/)
    -   [为什么梦想买不起，故乡回不去](https://ramsayleung.github.io/zh/post/2023/%E7%BD%AE%E8%BA%AB%E4%BA%8B%E5%86%85/)
-   软件工程
    -   [软件设计的哲学](https://ramsayleung.github.io/zh/post/2025/a_philosophy_of_software_design/)
    -   [一本读了八年还没读完的书](https://ramsayleung.github.io/zh/post/2025/structure_and_interpretation_of_computer_programs/)
    -   [测试技能进阶(一): 软件质量认知](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%80_%E8%BD%AF%E4%BB%B6%E8%B4%A8%E9%87%8F%E8%AE%A4%E7%9F%A5/)
    -   [测试技能进阶(二): Parameterized Tests](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/)
    -   [测试技能进阶(三): Property Based Testing](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%89_property_based_testing/)

<div class="qr-container" center>

<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" class="qr-container" width="160px" height="160px" center="t" />
公号同步更新，欢迎关注👻

</div>
