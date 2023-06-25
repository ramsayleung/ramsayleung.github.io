+++
title = "两个鲜为人知的Gmail地址技巧"
date = 2023-06-24T20:15:00-07:00
lastmod = 2023-06-24T21:07:58-07:00
tags = ["tool", "productivity", "gmail"]
categories = ["tool", "productivity", "gmail"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

分享两个鲜为人知，但是却相当有用的 Gmail 地址技巧。「鲜为人知」并非是标题党，而是引用Gmail 博客原话： <br/>

> I recently discovered some ****little-known**** ways to use your Gmail address that can give you greater control over your inbox and save you some time and headache. <br/>


## <span class="section-num">2</span> 技巧 {#技巧}

假设你的Gmail 地址是 `xiaoming@gmail.com`: <br/>


### <span class="section-num">2.1</span> 加号 {#加号}

你可以将在用户名后面增加一个加号 `+`, 并在加号后面增加任意数量的字符，比如 `xiaoming+happy@gmail.com`, `xiaoming+upset@gmail.com`, Gmail 都会把这些地址当作成 `xiaoming@gmail.com`, 发送到你的地址邮箱中。 <br/>


### <span class="section-num">2.2</span> 点号 {#点号}

你也可以在地址的任意地方插入任意数量的点号: `.`, 比如 `x.i.a..o.ming@gmail.com`, `xiao...mi..ng@gmail.com`, Gmail 都会把点号忽略掉，解析成 `xiaoming@gmail.com` <br/>


## <span class="section-num">3</span> 用途 {#用途}

技巧比较简单，寥寥数语就说完了，好像也没有什么大不了，有什么用处么？ <br/>

这个就要发挥想象力了。 <br/>


### <span class="section-num">3.1</span> 用途一：重复注册用户 {#用途一-重复注册用户}

这个主要是针对能使用邮箱注册的网站，可能大多数是国外网站。 <br/>

如果网站的邮箱地址校验正则写得不好，允许加号和点号，不知道Gmail的这两个规则，那么 `xiaoming+user1@gmail.com`, `xiaoming+user2@gmail.com`, `xi..aoming@gmail.com` 就会被认为是三个不同的邮箱地址，就可以重复注册。 <br/>

在薅羊毛等需要重复注册用户的场景就比较有用了。 <br/>


### <span class="section-num">3.2</span> 用途二：溯源 {#用途二-溯源}

个人邮箱难免会收到一些奇怪的邮件，例如：猎头的招聘邮件，钓鱼邮件等等。 <br/>

收到这些邮件的第一反应肯定是把邮件删掉，之后就会思考，究竟是哪里泄漏了个人邮箱。 <br/>

而通过 Gmail 加号的技巧，我就可以做到垃圾邮件溯源. <br/>

首先，在注册每个网站的时候，都给他们加上一个tag, 例如注册Twitter, 那就用 `xiaoming+twitter@gmail.com`, 如果注册Github, 那就用 `xiaoming+github@gmail.com`, 依此类推。 <br/>

只要有垃圾邮件，我就能通过加号的后缀，知道是哪个浓眉大眼的网站把我的信息给泄漏出去了。 <br/>

比如下面这个垃圾邮件，我就知道它是通过爬虫爬取我Github 公开邮件群发的. <br/>

{{< figure src="/ox-hugo/gmail_plus_sign_example.png" >}} <br/>

我就可以选择不公开 Github 邮箱，来避免后续收到类似的邮件。 <br/>


## <span class="section-num">4</span> 参考 {#参考}

-   [Google Gmail Blog: 2 hidden ways to get more from your Gmail address](https://gmail.googleblog.com/2008/03/2-hidden-ways-to-get-more-from-your.html) <br/>

