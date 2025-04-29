+++
title = "OpenAI-translator: 基于ChatGPT的划词翻译及润色应用"
date = 2023-03-12T10:20:00-07:00
lastmod = 2025-01-09T20:50:53-08:00
tags = ["tool"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 前言 {#前言}

因为ChatGPT 的爆红，最近基于在 ChatGPT 的工具如雨后春笋般冒出来，在 Twitter 上，基本每周都可以看到开发者发布基于 ChatGPT  的新应用（这些人不用上班的么？）。

而使用过好多的 ChatGPT 应用后，最惊艳的是 [@yetone](https://github.com/yetone) 开发的是 [openai-translator](https://github.com/yetone/openai-translator) 这款应用，支持「翻译」，「润色」，「总结」三种功能。


### <span class="section-num">1.1</span> 开发历程 {#开发历程}

因为在Twitter 上关注了 [@yetone](https://twitter.com/yetone), 所以能从推文看到@yetone 的开发历程：

最开始 yetone 是为 [Bob](https://bobtranslate.com/guide/#%E5%AE%89%E8%A3%85) 开发了基于 ChatGPT-api 的 [openai-translator](https://github.com/yetone/bob-plugin-openai-translator) 插件，广受Bob用户的好评。后面yetone 码力全开，又乘胜追击，为 Bob 开发了基于ChatGPT-api 的润色和语法纠错 [openai-polisher](https://github.com/yetone/bob-plugin-openai-polisher) 插件，完美替代了 Grammarly.

因为这两个插件的出色表现，很多非Bob 用户和 Mac 用户也希望可以尝鲜，因此 yetone 就徇众要求, 「糊」(yetone的原话)了一个浏览器插件，这就是 [openai-translator](https://github.com/yetone/openai-translator) ，而后就一发不可收拾了。

因其出色的表现，在Github 和 Hacker News 上爆火了。

然后 yetone 又将openai-translator 浏览器插件进行打包，做成跨平台的桌面端应用。


## <span class="section-num">2</span> 效果 {#效果}

测试文章来自Github Blog：[Raising the bar for software security: GitHub 2FA begins](https://github.blog/2023-03-09-raising-the-bar-for-software-security-github-2fa-begins-march-13/)


### <span class="section-num">2.1</span> 翻译(translate) {#翻译--translate}

翻译的用户体验，对比我之前一直在使用的沙拉划词([ext-saladict](https://github.com/crimx/ext-saladict)):

沙拉划词:

{{< figure src="/ox-hugo/ext-translator_translating.gif" >}}

openai-translator:

{{< figure src="/ox-hugo/openai-translator_translating.gif" >}}

翻译的效果（最核心的功能）

测试文章片段：

> We want enrolling your GitHub account in 2FA to be as easy as possible, using methods that are reliable and secure so you always have access to your account (and no one else does!). To prepare for this program we’ve been busy enhancing that experience. Here are a few of the highlights:

有道翻译与Google翻译：

{{< figure src="/ox-hugo/ext-translator.png" >}}

> 我们希望在2FA中注册您的GitHub帐户尽可能简单，使用可靠和安全的方法，以便您始终可以访问您的帐户(没有其他人可以!)。为了准备这个节目，我们一直在忙着提高这种体验。以下是其中的一些亮点

<!--quoteend-->

> 我们希望使用可靠且安全的方法在2FA中注册您的GitHub帐户尽可能容易，因此您始终可以访问您的帐户（而且没有其他人可以！）。为了准备该计划，我们一直在忙于增强这种体验。以下是一些亮点：

DeepL翻译：

{{< figure src="/ox-hugo/deepl.png" >}}

> 我们希望为您的GitHub账户注册2FA时尽可能简单，使用可靠和安全的方法，这样您就可以始终访问您的账户（而没有其他人可以访问！）。为了准备这个项目，我们一直在忙着增强这种体验。以下是其中的几个亮点。

openai-translator 翻译：

{{< figure src="/ox-hugo/openai-translator.png" >}}

> 我们希望让您的GitHub账户启用双重身份验证变得尽可能简单，使用可靠和安全的方法，以便您始终可以访问自己的账户（而别人则不能！）。为了准备这个计划，我们一直在不断改进用户体验。以下是其中的亮点：


### <span class="section-num">2.2</span> 润色(polish) {#润色--polish}

语法纠错及词句润色的效果：

测试文章片段：

> We want enrolling your GitHub account in 2FA to be as easy as possible, using methods that are reliable and secure so you always have access to your account (and no one else does!). To prepare for this program we’ve been busy enhancing that experience. Here are a few of the highlights:

对比我之前一直使用的 Language Tool:

{{< figure src="/ox-hugo/language_tool.png" >}}

估值 100 亿刀的 Grammarly:

{{< figure src="/ox-hugo/grammarly.png" >}}

DeepL 家新出的基于AI的写作助手DeepL Write：

{{< figure src="/ox-hugo/deepl_write.png" >}}

openai-translator:

{{< figure src="/ox-hugo/polished_result.png" >}}

因为openai-translator没有给出润色前后的比对，我们可以通过 diff 工具查看下：

{{< figure src="/ox-hugo/polished_result_diff.png" >}}


### <span class="section-num">2.3</span> 总结(summarize) {#总结--summarize}

这个应该是openai-translator 特有的功能，可针对长文给出总结与概要：

{{< figure src="/ox-hugo/summarizing.gif" >}}

原文片段：

> Over the course of the next year, we’ll be reaching out to groups of developers and administrators, starting with smaller groups on March 13, to notify them of their 2FA enrollment requirement. This gradual rollout will let us make sure developers are able to successfully onboard, and make adjustments as needed before we scale to larger groups as the year progresses.
>
> If your account is selected for enrollment, you will be notified via email and see a banner on GitHub.com, asking you to enroll. You’ll have 45 days to configure 2FA on your account—before that date nothing will change about using GitHub except for the reminders. We’ll let you know when your enablement deadline is getting close, and once it has passed you will be required to enable 2FA the first time you access GitHub.com. You’ll have the ability to snooze this notification for up to a week, but after that your ability to access your account will be limited. Don’t worry: this snooze period only starts once you’ve signed in after the deadline, so if you’re on vacation or out of office, you’ll still get that one week period to set up 2FA when you’re back at your desk.
>
> So, what if you’re not in an early enrollment group but you want to get started? Click here and follow a few easy steps to enroll in 2FA.

openai-translator 总结后的片段：

> GitHub will gradually roll out 2FA enrollment requirements to developers and administrators over the next year, starting with smaller groups on March 13.
>
> Those selected for enrollment will be notified via email and a banner on GitHub.com, with 45 days to configure 2FA before being required to enable it upon accessing the site. A snooze period of up to one week is available after the deadline has passed.
>
> Anyone can enroll in 2FA by following a few easy steps provided by GitHub.


## <span class="section-num">3</span> 使用方式 {#使用方式}

所谓 action is louder than words, 不看广告看疗效。

评价一个产品好不好用，只有用过才知道。体验openai-translator 最简单快捷的方式就是使用浏览器插件，免于安装桌面应用。

而你所需要的只是一个chatgpt 账号+ 一个浏览器插件：

1.注册安装 [openai](https://platform.openai.com/), 此处省去指引，有非常多的教程。

2.获取 [openai api key](https://platform.openai.com/account/api-keys)，并复制此key

{{< figure src="/ox-hugo/api-key.png" >}}

3.打开 [Chrome web store](https://chrome.google.com/webstore/detail/openai-translator/ogjibjphoadhljaoicdnjnmgokohngcc), 搜索 OpenAi Translator, 并点击安装

{{< figure src="/ox-hugo/chrome_web_store.png" >}}

4.点击搜件，粘贴刚刚复制的api-key:

{{< figure src="/ox-hugo/input_key.png" >}}

5.划词，并点击 openai-translator 图标进行体验。

{{< figure src="/ox-hugo/taste_it.gif" >}}


## <span class="section-num">4</span> 总结 {#总结}

ChatGPT 向我们展示了 GPT 模型的伟大之处。但模型虽强，阳春白雪，终究是离普通用户太远。

是无数个像yetone 这样的开发者，用产品展示给用户看，GPT 模型是如何的伟大。

向 yetone 致敬。


## <span class="section-num">5</span> 参考 {#参考}

-   [bob-plugin-openai-polisher](https://github.com/yetone/bob-plugin-openai-polisher)
-   [bob-plugin-openai-translator](https://github.com/yetone/bob-plugin-openai-translator)
-   [openai-translator](https://github.com/yetone/openai-translator)
-   [@yetone](https://twitter.com/yetone)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

