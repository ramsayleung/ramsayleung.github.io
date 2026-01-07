+++
title = "迄今所见最扎实的AI落地实践: Reddit的AI翻译"
date = 2026-01-05T22:18:00-08:00
lastmod = 2026-01-06T20:15:09-08:00
tags = ["ai", "design"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 全民AI热潮 {#全民ai热潮}

自OpenAI 发布 ChatGPT 以来，国内外都掀起了大干快上AI的各种项目，这两年真的是各种网站/应用都追求落地AI.

音乐软件有AI, 比如QQ音乐 [搞AI编曲](https://y.qq.com/vemus/index.html)&nbsp;[^fn:1], Spotify [搞AI歌单](https://newsroom.spotify.com/2024-04-07/spotify-premium-users-can-now-turn-any-idea-into-a-personalized-playlist-with-ai-playlist-in-beta/)&nbsp;[^fn:2]； 编程问答网站 Stackoverflow搞 [AI 问答](https://stackoverflow.com/ai-assist)&nbsp;[^fn:3], 浏览器搞AI(Firefox, [Chrome](https://www.google.com/chrome/ai-innovations/)&nbsp;[^fn:4]), 笔记软件如 Notion 搞 [AI workflow](https://www.notion.com/product/ai)&nbsp;[^fn:5]; 代码托管网站 GitHub 搞AI 编程和代码Review, 甚至最离谱的是连电子书管理软件(Calibre)也搞AI, 增加了一个 "[Asking AI](https://calibre-ebook.com/whats-new)"&nbsp;[^fn:6]的功能，真的是全民「炼」AI.

你做得好用就还好，但是一堆的网站或者应用就硬堆AI，所谓的AI功能无非是加个AI 对话框，让你可以打开聊天框和AI 对话，但是连当前网页的内容都没有当作上下文喂给AI, 体验非常差。


## <span class="section-num">2</span> Reddit 的AI翻译 {#reddit-的ai翻译}

今天看到一篇[文章](https://www.theguardian.com/technology/2026/jan/03/reddit-overtakes-tiktok-uk-search-algorithms-gen-z)&nbsp;[^fn:7], 说 Reddit 在英国超过 TikTok 成为访问量第四大的社媒平台，过去两年英国用户人数增长了 88%，三分之二的英国网民会访问 Reddit，而 2023 年仅是三分之一。而18-24 岁英国用户中 Reddit 是访问量第六大的网站，而一年前这一数字是第十。

{{< figure src="/ox-hugo/reddit_overtake_tiktok.jpg" >}}

不熟悉 Reddit 的朋友可以把 Reddit 理解成海外版本的「贴吧」.

Reddit 的崛起背后的因素包括了 Google 调整了算法增加了论坛类内容的权重，而 Reddit 就是论坛形式的社媒。在 AI 时代，用户也越来越多的转向人工撰写的内容，Reddit 也受益于这一趋势。

这篇文章让我想起最近Reddit 做的新功能，Reddit 利用大语言模型，为其海量帖子生成了多种语言的翻译版本，比如把英文帖子翻译成了中文, 韩文，日文等等。

然后我用中文在Google 搜索的时候，Google 搜索结果中就会展示原帖子的翻译版本，当我点击搜索结果时，就会跳转到Reddit 翻译后的中文版本:

{{< figure src="/ox-hugo/berserk_tv_in_chinese.jpg" >}}

我可以直接阅读翻译后的版本:

{{< figure src="/ox-hugo/berserk_post_tl_zh_hans.jpg" >}}

又或者点击「Show Original」阅读原版。

{{< figure src="/ox-hugo/berserk_post_show_original.jpg" >}}


## <span class="section-num">3</span> 天才产品设计 {#天才产品设计}

初看之下似乎平平无奇，细品之下真的是拍案叫绝，我愿称之为「迄今所见最扎实的AI落地实践」


### <span class="section-num">3.1</span> Reddit 角度 {#reddit-角度}

首先这个功能让更多用户阅读不同语言的帖子，比如中文，日文用户可以阅读原文是英语，法语，西班牙语的帖子，大大增加了Reddit 帖子的普适性，语言不再是个障碍，可以为Reddit 在AI时代增加用户流量，为新用户引流。

更多的用户就意味着更多的内容，更丰富的内容就能吸引来更多的用户，在AI时代，
内容就是「炼」AI大模型的原材料，光有英伟达的显卡，没有原材料，肯定没法炼出AI。

对比谷歌翻译这样的外置的翻译软件，这个是实时、无缝的页面级翻译，翻译质量相当高，其翻译质量之高、
与页面整合之自然，初用时甚至令人难以察觉这是机器翻译的产物。

用户体验非常好，其自然和实用的特点，完美解决了非母语用户访问高质量 UGC 的核心痛点。

更关键的是，作为用户，我完全没有意识到「AI」的存在。

我是在切换到工程师的视角之后，我才意识到 Reddit 使用大模型把帖子给翻译了，然后让谷歌的搜索引擎把不同语言的版本都索引, 直接把翻译结果在谷歌的搜索结果中展示了

{{< figure src="/ox-hugo/berserk_tv_in_chinese.jpg" >}}

最好的AI特性就是让你根本意识不到AI的存在。


### <span class="section-num">3.2</span> Google 角度 {#google-角度}

对Google 来说，因为Reddit 绝大部分的帖子都是人发的(Reddit版主大多都非常讨厌机器人或者AI 发帖，有相当多的版规说明不能使用AI生成内容，否则非常容易被禁言或者在某个子版封号), 提高论坛类内容的权重, 即可以保证搜索质量.

而索引不同语言版本的帖子又给谷歌提供了更多的高质量的搜索数据。

所以谷歌既索引了大量新鲜、高质量、多语言的人类内容，又提升了搜索结果质量，正好契合其算法调整方向


### <span class="section-num">3.3</span> 数据、模型与生态的飞轮 {#数据-模型与生态的飞轮}

正是通过为用户打破语言壁垒，为谷歌丰富搜索结果，Reddit得以启动一个更宏大的正向循环

在 2024年2月, Reddit 和 谷歌 达成深化合作的协议，Reddit 允许谷歌使用它的数据训练 AI 模型, 据报道，初始合同是每年六千万美元。

在 2025年9月，传出了 Reddit 和谷歌以及 OpenAI 谈新的合同，因为他们相信他们的用户数据对于 AI 训练非常重要，所以他们想谈更好的合同。

所以这个AI翻译的设计，就是一个强强联合, 打出 `1+1>3` 的效果, 甚至能以此为助力和其他的大模型厂商聊合作，无论是OpenAI, 还是 Anthropic, 没有数据也练不出 AI来。

形成了一个正向循环：更多用户 -&gt; 更多内容 -&gt; 更好AI数据 -&gt; 更好翻译/搜索/AI模型 -&gt; 更多用户

这是一个对用户，对 Reddit, 对谷歌，甚至对整个AI领域都有益处的设计，甚至可以说是「四赢」


## <span class="section-num">4</span> 结语 {#结语}

成功的AI落地，是让技术隐形，让用户价值凸显.

比对其他「强加AI」的产品，Reddit 的翻译功能是真正将AI融入产品，而非牵强附会，它甚至都没有告诉用户，这是用AI来实现的, 不强求AI叙事。

写到这里，不禁让我想起国内最早 all in AI, 还手握搜索引擎和中国最大论坛贴吧的某公司，真的是一手王炸的牌打得稀烂。


### 推荐阅读 {#推荐阅读}

-   旅加经历
    -   [这些年走过的路：从广州到温哥华](https://ramsayleung.github.io/zh/post/2023/%E8%BF%99%E4%BA%9B%E5%B9%B4%E8%B5%B0%E8%BF%87%E7%9A%84%E8%B7%AF_%E4%BB%8E%E5%B9%BF%E5%B7%9E%E5%88%B0%E6%B8%A9%E5%93%A5%E5%8D%8E/)
    -   [加拿大之初体验](https://ramsayleung.github.io/zh/post/2023/%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B9%8B%E5%88%9D%E4%BD%93%E9%AA%8C/)
    -   [登陆加拿大一年后的体会](https://ramsayleung.github.io/zh/post/2024/%E7%99%BB%E9%99%86%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B8%80%E5%B9%B4%E7%9A%84%E4%BD%93%E4%BC%9A/)
    -   [加国旅居纪–开篇](https://ramsayleung.github.io/zh/post/2025/%E5%8A%A0%E5%9B%BD%E6%97%85%E5%B1%85%E7%BA%AA--%E5%BC%80%E7%AF%87/)
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

[^fn:1]: <https://y.qq.com/vemus/index.html>
[^fn:2]: <https://newsroom.spotify.com/2024-04-07/spotify-premium-users-can-now-turn-any-idea-into-a-personalized-playlist-with-ai-playlist-in-beta/>
[^fn:3]: <https://stackoverflow.com/ai-assist>
[^fn:4]: <https://www.google.com/chrome/ai-innovations/>
[^fn:5]: <https://www.notion.com/product/ai>
[^fn:6]: <https://calibre-ebook.com/whats-new>
[^fn:7]: <https://www.theguardian.com/technology/2026/jan/03/reddit-overtakes-tiktok-uk-search-algorithms-gen-z>
