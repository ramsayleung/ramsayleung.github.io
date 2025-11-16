+++
title = "写博客的动机"
date = 2017-09-17T10:36:00-07:00
lastmod = 2025-11-16T09:42:24-08:00
tags = ["summary"]
categories = ["得失感悟"]
draft = false
toc = true
highlighted = true
+++

## <span class="section-num">1</span> 为什么要写博客 {#为什么要写博客}

虽说我写博客的时间不长，但是当9月份为止，我已写了不少博文的。那么，为什么要写博文呢？

很明显，这是一个费时费精力的工作，那么为什么我还要写呢？我自己思考过这个问题，我觉得，原因有下：


### <span class="section-num">1.1</span> 技术沉淀 {#技术沉淀}

在平时的工作生活中，我免不了会碰到种种问题，了解这些问题，思考解决方案并最终付诸行动，这本身就是一个学习和进步的过程。

而把其中的想法感悟写成博文会更有益于我的技术沉淀


### <span class="section-num">1.2</span> 提高组织文字的能力 {#提高组织文字的能力}

对于很多工科生来说，不擅文字，不擅言语表达估计是他们撕不掉的标签之一，但是，学会 恰当地表达自己的想法是一项非常重要的技能，写博客可以提高自己的语言组织能力。

此外， 一位前辈曾经告诫我，如果你连文字都没办法组织好，我怎么相信你可以把你的代码组织好呢？


### <span class="section-num">1.3</span> 便于了解自我 {#便于了解自我}

当在编写博客的时候，你会有诸多的想法和思考，然后你会把你的思考一点一滴付诸于笔尖， 你的博文越来越清晰了，你的自己的认识也会越来越清晰，你最终会了解到自己是一个什么样的人，喜欢做的事是什么，想要的又是什么？


### <span class="section-num">1.4</span> 分享与交流 {#分享与交流}

你有一个苹果，我有一个苹果，我们交换了苹果，我们还只是拥有一个苹果.

但是，你有一种想法，我有一种想法，我们交换了想法，我们就有两个想法。

写博客就是一种双向的交流方式，笔者介绍自己的观点，读者发表自己的评论，思想由此而激荡，甚至孕育出新的想法


## <span class="section-num">2</span> 博客历史 {#博客历史}


### <span class="section-num">2.1</span> org-page {#org-page}

我将博客从 [org-page](https://github.com/emacsorphanage/org-page) 搭建的 [Github Page](https://github.com/ramsayleung/samrayleung.github.io/) 迁移到现在的[博客](https://github.com/ramsayleung/blog)上，原来基于 Gtihub Page，使用 Emacs, org-mode 和 org-page 的博客其实也相当好用，只是某一些我想要的功能却缺失，所以我就自己花时间动手写了现在这个博客，并且将原来的博文迁移

我把这个基于 org-page 的博客部署在 Cloudflare Pages 上，感兴趣的读者可以看下：<https://samrayleung-github-io.pages.dev/>


### <span class="section-num">2.2</span> 自建博客 {#自建博客}

当时 [org-page](https://github.com/emacsorphanage/org-page) bug不少，支持不是很及时，虽然org-mode很好用，但最后走到自建的博客的路子上。

使用Rust练手写的博客很不错，但是写作workflow 还不够滑顺，使用orgmode写作，导出为markdown；

对于图片，只能先上传到图床，然后再插入到文章里面。


### <span class="section-num">2.3</span> ox-hugo {#ox-hugo}

<span class="timestamp-wrapper"><span class="timestamp">&lt;2022-02-25 Fri&gt;</span></span>

没想到五年后，我又从自建的[ Blog](https://github.com/ramsayleung/blog/)，又迁移回 [ox-hugo](https://ox-hugo.scripter.co/) 驱动的Github Page。

所以现在切换回Github Page，使用org-mode和ox-hugo, 不需要修改就可以直接把文章发布成博客，又可以愉快地写文章了。

org-mode + Emacs 实在是太舒服了, 关于我现在的写作流，可以参考文章: [《我的写作流》](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E5%86%99%E4%BD%9C%E6%B5%81/)


### <span class="section-num">2.4</span> 迁移历史 {#迁移历史}

-   2016-2017：基于 [org-page](https://github.com/emacsorphanage/org-page) 生成的 [GitHub Pages 博客](https://github.com/ramsayleung/samrayleung.github.io)
-   2017-2021：基于 Rust 自建的 [博客系统](https://github.com/ramsayleung/blog)
-   2022-至今：使用 [ox-hugo](https://ox-hugo.scripter.co/) 生成并部署于 [GitHub Pages](https://github.com/ramsayleung/ramsayleung.github.io)，即[本博客](https://ramsayleung.github.io/zh/)

回顾这段博客迁移历程，不禁让我想起曾读过的一篇文章：[How to Start Your Blog](https://molodtsov.me/2023/02/how-to-start-your-blog-in-2023/)，其中的题图令人印象深刻：

{{< figure src="/ox-hugo/blogging.jpg" >}}

这张图非常有意思：横轴代表配置过的博客数量，纵轴则是实际写下的文章数量。

它生动地描绘出创作表达与博客工具之间的张力 —— 几乎每位博客主都能在图中找到自己的位置。

右下角是那些热衷于「配置与折腾」的选手，他们不断尝试各种工具，却很少真正产出内容；
而左上角则是那些不拘泥于形式、专注于内容本身的选手。

写了十年博客，其间也经历过好几轮工具与样式的更迭，如今我对这些外在配置已渐渐「心如止水」。「表」不再那么重要，我更看重的是「里」。

如果博客外表光彩夺目，内容却贫乏稀疏，我自己难免会产生“金玉其外，空絮其中”的自嘲。

因此，我更希望用扎实的内容去吸引读者，而非仅靠华丽的外观。


## <span class="section-num">3</span> 结语 {#结语}

每个人总会有不同的想法，不同的际遇，如果你想和他人分享而不知从何言起，与何人言？

何不付诸博客呢？

<div class="qr-container" center>

<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" class="qr-container" width="160px" height="160px" center="t" />
公号同步更新，欢迎关注👻

</div>
