+++
title = "Prompt 工程的本质，是提问的艺术"
date = 2025-10-25T20:18:00+08:00
lastmod = 2025-10-25T23:05:22+08:00
tags = ["ai"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 引言 {#引言}

在 AI 浪潮席卷全球的今天，"Prompt Engineer"（提示词工程师）成了热门词汇。

AI 爱好者热衷于分享 prompt、研究 token 控制、调试 temperature 参数。

各种梗图也从 Linux 之父 Linus Torvalds 的

> Talk is cheap. Show me the code.

演变成：

> Code is cheap. Show me the prompts.

{{< figure src="/ox-hugo/talk_is_cheap_show_me_the_prompt.jpg" >}}

原推: <https://x.com/tunguz/status/1856045530951917763?lang=en>

但其实， 高效与 AI 交互的核心，并不是技术黑话，而是「如何有效地提问」

早在 2001 年，Eric S. Raymond 和 Rick Moen 就写下了经典指南《[How To Ask Questions The Smart Way](http://www.catb.org/~esr/faqs/smart-questions.html)》。

它原本是在黑客文化兴起时，为开源社区中的技术求助者而写，但其原则出奇地适用于今天的 AI 交互 ——
因为 ****无论是求助专家还是大模型，本质上都是提供尽量多的有效信息，以期待尽快得到能解决问题的答案**** 。

在我的日常工作中，无论是向同事还是其他领域的专家提问，我仍能从中汲取智慧，快速获得所需答案。

在 AI 新时代，我尝试把这瓶关于「提问的智慧」的旧酒，装到「Prompt Engineer」的新瓶里。


## <span class="section-num">2</span> 先做功课，别做伸手党 {#先做功课-别做伸手党}

> 在提问前，请先搜索、阅读文档、尝试自己解决。

AI 不是你的私人助理，更不是替你思考的替身。

如果你连问题都没搞清楚就丢一句「帮我写个程序」，模型只能瞎猜。

好的 Prompt 应体现你已做的努力 ：

> "我尝试用 Python 的 requests 库抓取某网站，但返回 403 错误。我检查了网络连接，也添加了 User-Agent 请求头，但问题依旧。请问是否需要处理 Cookies 或考虑 JS 渲染？"
>
> 下面是我的报错日志:
> ...

这不仅能让 AI 更精准地定位问题，也减少了它进行无效尝试的可能 。


## <span class="section-num">3</span> 描述现象，而非猜测原因 {#描述现象-而非猜测原因}

> 说‘鼠标光标变形了’，别说‘显卡驱动坏了’。

很多人喜欢在 Prompt 里预设结论："我的代码有内存泄漏""这个 API 肯定有 bug"。

但 AI 没有上下文，无法验证你的假设。

请提供可观察的事实 ：

> "程序运行 10 分钟后，内存占用从 100MB 升至 2GB 且未见释放。这是我的核心代码片段和资源监控数据。"

最常见的可观察事实就是日志和运行数据； 对于 UI 问题（如 CSS 渲染不符合预期），提供截图会非常有用。

核心在于：你提供症状，让 AI 做分析推理。


## <span class="section-num">4</span> 目标导向，而非步骤导向 {#目标导向-而非步骤导向}

> 你想换轮胎，别只问「怎么用千斤顶」。

用户常陷入「路径依赖」：执着于某个工具或方法，却忘了最终目标。

先说目标，再说卡点。

不好的提问：

-   步骤导向：「怎么在 Excel 里用 `VLOOKUP` 匹配两列数据？」

好的提问：

-   目标导向：「我想合并两个表格，根据 ID 列匹配信息。我试了 `VLOOKUP` 但返回 `#N/A` ，已确认两列数据格式均为文本。」

这样，AI 才有可能建议你改用 `XLOOKUP`, `Power Query` ，甚至直接推荐使用 Python —— 给你更好的解决方案，而非仅仅完成你指定的步骤。


## <span class="section-num">5</span> 简洁、具体、结构化 {#简洁-具体-结构化}

> 文本长度 ≠ 信息量。
>
> 一段 500 行的代码，其信息密度往往不如一个精心提炼的、10 行代码的最小复现案例。

大模型虽能处理长上下文，但 ****噪声越多，有效信号越弱****

-   提供最小可复现示例
-   明确输入、期望输出、实际输出
-   用清晰的格式（如代码块、列表）组织信息

例如：

```text
输入：[1, 2, '3', 4]
期望：全部转为整数 → [1, 2, 3, 4]
实际：直接使用 int() 转换时报错
问题：如何安全地转换这种混合类型的列表？
```


## <span class="section-num">6</span> 别先求「完整答案」，要「关键指引」 {#别先求-完整答案-要-关键指引}

虽然 AI 的能力远超传统问答场景，但高效协作的关键并非「全盘托出」，而是「分步引导」。

> 传统建议：问「哪里可以学？」比「教我全部」更高效。

在 AI 时代，虽然 AI 能直接输出完整答案，但「 先给思路、再选方案、后生成代码 」的分阶段交互模式，往往更高效、更可控，也能减少"幻觉"。

这本质上是把 AI 当作「 协作者 」而非「答题机 」。

更高效的协作模式：

1.  先让 AI 阐述解决思路
2.  提供2–3 个可行方案
3.  对比各方案的优劣（性能、可维护性、复杂度等）
4.  在人类确认方向后，再展开具体实现

这种「分阶段协作」能显著减少幻觉、提升可控性，并让人类保持对关键决策的主导权。


## <span class="section-num">7</span> 礼貌 + 闭环 = 长期共赢 {#礼貌-plus-闭环-长期共赢}

> 一句「谢谢」不费事，但能让专家更愿意帮你。

虽然 AI 没有情感，但结构化的礼貌与反馈能塑造更高质量的交互 ：

-   开头清晰说明背景，结尾表达感谢
-   如果得到帮助，可追加反馈：「按你的建议修改后，问题解决了，谢谢！」

这种反馈闭环对当前的静态 AI 模型虽不直接触发在线学习，但告知模型它的解决方案有效，相当于在模拟 [RLHF（人类强化学习）](https://en.wikipedia.org/wiki/Reinforcement_learning_from_human_feedback) 的过程 。

你越会提问、越会验证、越会修正，AI 在你手中就"越像"一个懂你的协作者。

这就是闭环反馈的价值。

---

此外，给AI说「请」还有「谢谢」还能以防万一，如果 AI 以后统治人类，还有可能会因为你曾经对它有礼貌而放你一马：

{{< figure src="/ox-hugo/be_polite_to_robot.jpg" >}}

来源：<https://old.reddit.com/r/comics/comments/x8bcu9/be_polite_to_robots/>


## <span class="section-num">8</span> 结语：提问是一种能力，也是一种尊重 {#结语-提问是一种能力-也是一种尊重}

《提问的智慧》的核心不是「技巧」，而是态度 ：尊重他人的时间，尊重知识的边界，尊重问题本身的复杂性。

在 AI 时代，我们拥有了前所未有的「全能即时专家」，但懒惰的提问只会得到平庸的回答 。

真正高效的 Prompt Engineer，不是会背 prompt 的人，而是懂得如何与 AI 共建理解的人。
（尽管当前的 AI 本质上仍是一个概率模型。）

正如书中所言：

> Good questions are a stimulus and a gift. (好问题，本身就是一份礼物。)


## <span class="section-num">9</span> 延伸阅读 {#延伸阅读}

-   [How To Ask Questions The Smart Way (英文原文)](http://www.catb.org/~esr/faqs/smart-questions.html)
-   [How to Report Bugs Effectively – Simon Tatham](https://www.chiark.greenend.org.uk/~sgtatham/bugs.html)


### 推荐阅读 {#推荐阅读}

-   旅加经历
    -   [这些年走过的路：从广州到温哥华](https://ramsayleung.github.io/zh/post/2023/%E8%BF%99%E4%BA%9B%E5%B9%B4%E8%B5%B0%E8%BF%87%E7%9A%84%E8%B7%AF_%E4%BB%8E%E5%B9%BF%E5%B7%9E%E5%88%B0%E6%B8%A9%E5%93%A5%E5%8D%8E/)
    -   [加拿大之初体验](https://ramsayleung.github.io/zh/post/2023/%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B9%8B%E5%88%9D%E4%BD%93%E9%AA%8C/)
    -   [登陆加拿大一年后的体会](https://ramsayleung.github.io/zh/post/2024/%E7%99%BB%E9%99%86%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B8%80%E5%B9%B4%E7%9A%84%E4%BD%93%E4%BC%9A/)
    -   [关于破解加拿大航空飞机网络限制的一件小事](https://ramsayleung.github.io/zh/post/2025/%E5%85%B3%E4%BA%8E%E7%A0%B4%E8%A7%A3%E5%8A%A0%E6%8B%BF%E5%A4%A7%E8%88%AA%E7%A9%BA%E9%A3%9E%E6%9C%BA%E7%BD%91%E7%BB%9C%E9%99%90%E5%88%B6%E7%9A%84%E4%B8%80%E4%BB%B6%E5%B0%8F%E4%BA%8B/)
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
公众号同步更新，欢迎关注👻

</div>
