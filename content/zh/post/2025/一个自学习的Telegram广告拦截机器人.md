+++
title = "一个自学习的Telegram广告拦截机器人"
date = 2025-08-28T23:45:00-07:00
lastmod = 2025-09-01T21:42:23-07:00
tags = ["telegram", "design", "programming", "rails", "rust"]
categories = ["telegram"]
draft = false
toc = true
showQuote = true
highlighted = true
+++

[English Version](https://ramsayleung.github.io/en/post/2025/a_telegram_spam_blocker_bot_based_on_bayesian/)


## <span class="section-num">1</span> 序言 {#序言}

我花了一周末时间，写了一个自学习的 Telegram 广告拦截机器人 `@BayesSpamSniperBot` (<https://t.me/BayesSpamSniperBot>)，项目开源在：<https://github.com/ramsayleung/bayes_spam_sniper>


### <span class="section-num">1.1</span> Telegram {#telegram}

Telegram 是一个流行的即时通讯软件，类似微信，Whatsapp，已有超过10亿用户，支持许多强大的功能，如聊天记录云存储，支持Linux, Mac, Windows, Android, IOS, Web 多个平台，客户端都是开源，类似微信公众号的频道功能(Channel)，还有我见过的最强大的机器人系统。


## <span class="section-num">2</span> 缘起 {#缘起}

平时我跑步和做饭都习惯会听播客，而《[软件那些事儿](https://podcasts.apple.com/us/podcast/%E8%BD%AF%E4%BB%B6%E9%82%A3%E4%BA%9B%E4%BA%8B%E5%84%BF/id1147186605)》[^fn:1]是我最喜欢的播客之一，主持人是[栋哥](https://liuyandong.com/sample-page)&nbsp;[^fn:2], 我也因为喜欢栋哥的节目，趁机加了栋哥的电报频道。

栋哥的电报频道[汗牛充栋](https://t.me/huruanhuying)&nbsp;[^fn:3]主要是用来发布播客信息，
之前打开过一段时间的留言功能，没有想到引来了一堆的币圈的用户来发广告，因此将评论功能就关了：

{{< figure src="/ox-hugo/spam_concert.jpg" >}}

另外一个我关注的频道 [Ray Tracing](https://t.me/kaedeharakazuha17)&nbsp;[^fn:4]也在吐槽币圈的广告，不堪其忧:

{{< figure src="/ox-hugo/ray_tracing_spam.jpg" >}}


## <span class="section-num">3</span> 黑客与画家 {#黑客与画家}

常见的 Telegram 广告机器人是大多是基于关键字的，通过匹配关键字进行文本拦截，非常容易被发垃圾广告的人绕过。

{{< figure src="/ox-hugo/keyword_based_blocker.jpg" >}}

被绕过的话主要是靠管理员人工删除。

这不禁让我想起了保罗.格雷厄姆在《黑客与画家》一书在2002年介绍的情况：

当时电子邮件兴起，也有非常多的垃圾邮件，常见的垃圾广告拦截方式是关键字匹配+邮件地址黑名单，但是既低效也容易被绕过。

保罗.格雷厄姆就创造性地使用贝叶斯算法(Bayesian Theorem)实现了一个[广告拦截器](https://paulgraham.com/spam.html)&nbsp;[^fn:5], 效果竟然出奇地好。

对于 Telegram 的垃圾广告而言，这不是类似的问题嘛？

那我岂不是可以用类似的解决方案来解决 Telegram 广告的问题嘛


### <span class="section-num">3.1</span> 贝叶斯定理 {#贝叶斯定理}

提起概率算法，最经典的例子莫过于「抛硬币」这一古典概率——每次抛掷都是独立事件，前一次的结果不会影响下一次的概率。

然而，现实中的很多场景并不能像抛硬币那样无限重复，事件之间也往往并非相互独立。

这时候，贝叶斯定理就显示出其独特的价值。

它是一种「由果溯因」的概率方法，用于在已知某些证据的条件下，更新我们对某一假设的置信程度。

换句话说，贝叶斯算法能够根据不断出现的新证据，动态调整对某个事件发生概率的估计。

简单来说，就像人脑的学习过程：我们原本有一个初步认知，在获得新信息之后，会据此修正原有的看法，进而调整下一步的行动。

保罗·格雷厄姆就是通过贝叶斯定理，不断地根据已被标记为垃圾广告或者非垃圾广告的邮件，对新出现的邮件进行分类，判断其是否为垃圾广告。

如果想更直观地理解贝叶斯定理，推荐两个讲解清晰、生动易懂的视频：

-   《[Bayes' Theorem 贝叶斯定理](https://www.youtube.com/watch?v=Pu675cHJ7bg)》[^fn:6](中文)
-   《[Bayes theorem, the geometry of changing beliefs](https://www.youtube.com/watch?v=HZGCoVF3YvM)》[^fn:7](英文)


## <span class="section-num">4</span> 架构设计 {#架构设计}

Telegram Bot 支持两种与 Telegram 服务器交互的模式，分别是：

1.  Webhook: Telegram 服务器会在 Bot 收到新消息时主动回调此前 Bot 注册的地址，Bot Server 只需要处理回调的消息
2.  Long Polling: Bot Server 一直轮询 Telegram 服务器，看是否有新消息，有就处理，本机器人使用的是此模式

    {{< figure src="/ox-hugo/webhook_vs_long_polling.jpg" >}}


#### <span class="section-num">4.0.1</span> 消息分析 {#消息分析}

{{< figure src="/ox-hugo/spam_analyze.jpg" >}}

Bot Server 收到消息之后，会派发到单独的 `telegram_bot_worker` 处理，然后根据预训练的模型判断是否是垃圾广告，如果是，调用 Bot API 删除消息。


#### <span class="section-num">4.0.2</span> 封禁并训练 {#封禁并训练}

{{< figure src="/ox-hugo/mark_spam_and_ban_user.jpg" >}}

Bot Server 收到消息之后，会派发到单独的 `telegram_bot_worker` 处理， `telegram_bot_worker` 会调用 bot API 删除消息并封禁用户，并插入一条训练数据，标记为垃圾广告(spam)

保存训练数据会触发 hook, 创建一个训练消息，投递到消息队列 `training`, 会有另外的 worker `classifier_trainer` 订阅 `training` 消息，并使用新消息重新训练和更新模型

使用队列和后台进程 `classifier_trainer` 来训练任务而非直接使用 `telegram_bot_worker` 主要是为了返回 Bot请求与训练模型解耦，否则随着模型规模的增大，训练时间会越来越长，响应时间会越来越长。

解耦后就易于水平扩展了，在设计上为后续性能优化和扩展预留空间。


## <span class="section-num">5</span> Why Rails {#why-rails}

看了我项目源代码的朋友，难免会浮起疑问，为什么使用 Ruby on Rails 实现的?

因为我工作中会有用到JVM系的编程语言(Java/Kotlin/Scala)和 Rust, 所以我对 Java/Rust 相当熟悉，又觉得模型训练可能对性能要求很高，所以最开始的[原型](https://gist.github.com/ramsayleung/5848af0177a70a01d41f624e361b1b5d)&nbsp;[^fn:8]我是用 Rust 实现的，大概就花了半个多小时。

但是当我想把原型扩展成 Telegram 机器人时，就发现需要处理相当多与机器人交互的逻辑，主要涉及到 API 与数据库操作，其中大部分都是和模型无关的，因此我又想到了 Ruby on Rails。

论单个工程师做产品原型，就我个人而言，实在是没有比 `Ruby on Rails` 更高效的框架了，因此我就切换到 Ruby on Rails 去。

Rails 8 的新特性，把 Rails 向所谓的「一人全栈框架」又推进了不少，通过关系型数据库内置对消息队列 `Solid Queue` 的支持，甚至不再需要类似 Redis 这样的存储来支持队列实现。

架构设计中的队列和后台进程，只需要几行代码就实现了，甚至不需要额外的配置，如果队列不存在，框架会自动创建：

```ruby
class ClassifierTrainerJob < ApplicationJob
  # Job to train classifier asynchronously
  queue_as :training

  def perform(group_id, group_name)
    SpamClassifierService.rebuild_for_group(group_id, group_name)
  end
end
```

得益于 Rails 强大的 ORM 框架，内置各种生命周期的 hook, 对新插入训练数据后触发后台进程重新训练模型的代码也只有寥寥几行：

```ruby
class TrainedMessage < ApplicationRecord
  # Automatically train classifier after creating/updating a message
  after_create :retrain_classifier
  after_destroy :retrain_classifier

  def retrain_classifier
    # For efficiency, we could queue this as a background job
    ClassifierTrainerJob.perform_later(group_id, group_name)
  end
end
```

在 Rails 各种内置强大工具的加持下，我只用了一天时间就把整个机器人的功能给实现出来了。

看到这里，有朋友可能会担心性能，觉得 Ruby 性能不行，并且还是动态语言，不好维护。

我持有的观点还是和之前的博文《[编程十年的感悟](https://ramsayleung.github.io/zh/post/2024/%E7%BC%96%E7%A8%8B%E5%8D%81%E5%B9%B4%E7%9A%84%E6%84%9F%E6%82%9F/)》[^fn:9]一样：

先跑起来再说，先做个原型跑起来，有用户愿意用你的产品再说，
当运行速度成为瓶颈时，你的业务肯定非常大了，肯定有足够的资源招一打程序员把项目优化成 Rust/C++, 甚至是汇编。

没有用户，谈性能只是个伪命题。

至于动态语言一时爽，代码维护火葬场，我也是相当认同的。

因此我在为团队选型时我绝对不会考虑动态语言，只会上编译型的语言，
甚至是Rust这种强类型，但是现在只有我一个人来做原型，我自己是什么顺手就用什么的。


### <span class="section-num">5.1</span> Vibe Coding? {#vibe-coding}

Vibe Coding等AI编程概念可谓是铺天盖地，甚嚣尘上，难免会有朋友好奇我这个项目是否 Vibe Coding生成的。

答案是，我尝试了几个小时之后，直接放弃了， Claude 4 和 Gemini 2.5 Pro 都试过了。

开始是使用 Rust + Cloudflare Worker 的技术栈，Rust + Cloudflare Worker 是个相当小众的领域，训练语料少，Vibe Coding 出来的代码编译无法通过

后面换成 Ruby on Rails, 问题还更严重了，Ruby 是弱类型的动态语言，语法写起来和英语一样，Rails 又还有很多黑魔法，所以到运行时才报错，代码生成省下来的开发时间，debug过程全补回来了。

另外一个是 Vibe Coding 生成的代码很多都是没有设计的，比如把 `Classifier` 和 `TrainedMessage` 的类耦合在一起，在 `Classifier` 里面持久化 `TrainedMessage`

又直接在 `telegram_bot_worker` 进程里面，接收到训练信息马上同步训练新模型，训练完再返回调用命令的结果，完全没考虑解耦接收训练语料和模型训练。

只能说 Vibe Coding 非常适合 Rust 这样的强类型编译型语言，生成的出来的代码起码要编译通过，保证质量的下限。

而对于那些说「一行代码都不用写/改，就能做出一个APP」的言论，此时我脑海不禁升起疑问？

究竟是代码好到一行都不用改？还是开发者看不出症结所在，所以一行都不改？


## <span class="section-num">6</span> 设计理念 {#设计理念}

开发完原型，在机器人整体功能可用之后，脑中又有不少的想法冒出来，当时就马不停蹄地给机器人加上，
因此机器人就支持快十个命令，还支持私聊和群聊的不同模式。

加着加着，连我自己都疑惑起来：这么多的功能，有点像国内的各种大而全的App了，我不禁对此产生疑问：

真的会有用户用这么多功能么？真的有用户会用这些功能嘛？太多功能不是也会有额外的心智负担嘛?

我最喜欢的广告拦截器 [Ublock Origin](https://ublockorigin.com/)&nbsp;[^fn:10]拦截效果非常好，但是使用起来却非常简单，易上手。

想起《[软件设计的哲学](https://ramsayleung.github.io/zh/post/2025/a_philosophy_of_software_design/)》[^fn:11]里面提到的设计理念，接口应该是简单易用的，但是功能可以是复杂丰富的。

因此我只能忍痛把此前新增的，但与核心功能无关的命令都删掉；

此外考虑到可能绝大多数的用户都没有技术背景，也可能不知道命令怎么用，因此将命令尽可能地优化成按钮，用户可以直接点击，改善易用性:

{{< figure src="/ox-hugo/start_zh.jpg" >}}

{{< figure src="/ox-hugo/help_page_zh.jpg" >}}

我还希望可以支持多语言，比如根据用户的系统语言，自动切换到中文或者英文，这个就需要不同语言的文案。

{{< figure src="/ox-hugo/start_en.jpg" >}}

{{< figure src="/ox-hugo/help_en.jpg" >}}

[telegram_botter.rb](https://github.com/ramsayleung/bayes_spam_sniper/blob/master/app/services/telegram_botter.rb) 这个核心服务类里面有超过60%的代码都是为了此类易用性改进而引入的。

简单留给用户，复杂留给开发


### <span class="section-num">6.1</span> 如何使用 {#如何使用}

只需两步，机器人就可以自动工作。

-   将机器人（@BayesSpamSniperBot）[添加到您的群组](https://t.me/BayesSpamSniperBot?startgroup=true)
-   给予机器人管理员权限（删除消息(delete message )，封禁用户权限(ban user )）

完成这两步后，机器人不仅会自动开始工作，自动识别群内广告，然后删除文本消息，如果发送垃圾广告超过3次，将会被封禁；

还会随着社区的使用（通过 `/markspam` 和 `/feedspam` ），变得越来越智能

{{< figure src="/ox-hugo/detect_spam_and_ban_user.jpg" >}}

此机器人的设计理念就是最小化打扰管理员与用户，提供简单的操作命令，并最大可能地自动化，
所以本机器人只提供以下三个命令（支持"/"开头自动补全）:

{{< figure src="/ox-hugo/command_auto_completion.jpg" >}}


#### <span class="section-num">6.1.1</span> `/markspam` {#markspam}

删除垃圾消息并封禁用户, 需要管理员权限。

在某条你想封禁的信息下回复 `/markspam`, 机器人就会自动把该条消息删除被封禁用户.

{{< figure src="/ox-hugo/markspam_2.jpg" >}}

(消息也被删除)
![](/ox-hugo/markspam.jpg)

与常见的群管理机器人不同，这条命令不仅会删除垃圾消息并封禁用户, 因为这条消息还被管理员标记成垃圾广告，有非常高的置信度，所以系统就会以这条垃圾广告为训练数据，对模型进行实时更新。

下次类似的发言不仅会被识别，所有使用本机器人的群组都会受益，也会把类似的文本标记成垃圾广告


#### <span class="section-num">6.1.2</span> `/listspam` {#listspam}

查看封禁账户列表, 需要管理员权限。

{{< figure src="/ox-hugo/listspam.jpg" >}}

查看已封禁的用户列表，并主动解封。


#### <span class="section-num">6.1.3</span> `/feedspam` {#feedspam}

投喂垃圾信息来训练，无任何权限要求，可私聊投喂或在群组内投喂.

私聊投喂:

{{< figure src="/ox-hugo/feedspam.jpg" >}}

群组内投喂:

{{< figure src="/ox-hugo/feedspam2.jpg" >}}


## <span class="section-num">7</span> Eating your own dog food {#eating-your-own-dog-food}

在软件开发领域，有这么一句俗话，Eating your own dog food(吃你自己的狗粮)，大意是你自己的开发的东西，要自己先用起来。

因此我建了一个自己的频道：[菠萝油与天光墟](https://t.me/pipeapplebun)&nbsp;[^fn:12]用于测试，可惜订阅者寥寥，
就吸引不来太多的发垃圾广告的用户，所以欢迎大家订阅或者进来发广告，以吸引更多的发垃圾广告的用户。

在我这个频道，每个人都有自由发言的权利(美中不足只是次数受限)

既然没有人来我的频道发广告，苦于没有训练数据，我只能主动出击，赤膊上阵，割肉喂鹰去加了各种币圈群，黄色群，主动去看各种广告了：

{{< figure src="/ox-hugo/telegram_group1.jpg" >}}

{{< figure src="/ox-hugo/telegram_group2.jpg" >}}

{{< figure src="/ox-hugo/spam_sample.jpg" >}}

自从开发了这个机器人之后，我对广告的看法就变了，以前在别的群看到广告就烦，现在在别的群看到广告就很开心，
这都是宝贵的训练数据，要趁着还没被删，赶紧记录下来。

{{< figure src="/ox-hugo/feedspam3.jpg" >}}


### <span class="section-num">7.1</span> 八仙过海的垃圾广告 {#八仙过海的垃圾广告}

别人故事里的算法效果总是出奇的好，到自己实际运行的时候，总是发现会有这样那样的 case 没有覆盖，总有各种意外惊喜

许多在 Telegram 发广告的用户都是久经考验的反拦截器斗士了。

虽然关键词封禁效率不高，但是那些能让我们见到的广告说明已经是绕过关键词拦截的。

比如:

> 在 币圈 想 赚 钱，那 你 不关 注 这 个 王 牌 社 区，真的太可惜了，真 心 推 荐，每 天 都 有 免 费 策 略

又或者

> 这人简-介挂的 合-约-报单群组挺牛的ETH500点，大饼5200点！ + @BTCETHl6666

前者通过空格分隔来绕过关键词，后者通过添加标点符号来绕过关键词。

与英文等基于拉丁字母的语言天然通过空格分词不同，中文使用贝叶斯算法进行统计时，需要先进行分词

> the fox jumped over the lazy dog
>
> 我们的中文就不一样了

「我们的中文就不一样了」就会被分词成「我们 | 的 | 中文 | 就 | 不 | 一样 | 了」, 然后才能对词频进行统计。

但是像广告 `在 币圈 想 赚 钱，那 你 不关 注 这 个 王 牌 社 区，真的太可惜了，真 心 推 荐，每 天 都 有 免 费 策 略` , 空格除了会影响关键字匹配，也会影响分词，这句话的分词结果就会变成:

`在 | | 币圈 | | 想 | | 赚 | | 钱 | ， | 那 | | 你 | | 不 | 关 | | 注 | | 这 | | 个 | | 王 | | 牌 | | 社 | | 区 | ， | 真的 | 太 | 可惜 | 了 | ， | 真 | | 心 | | 推 | | 荐 | ， | 每 | | 天 | | 都 | | 有 | | 免 | | 费 | | 策 | | 略`

`这人简-介挂的 合-约-报单群组挺牛的ETH500点，大饼5200点！ + @BTCETHl6666` 也会被分词成：

`这人简 | - | 介挂 | 的 | | 合 | - | 约 | - | 报单 | 群组 | 挺 | 牛 | 的 | ETH500 | 点 | ， | 大饼 | 5200 | 点 | ！ | | + | | @ | BTCETHl6666`

未经处理的训练数据就会影响模型的结果，可见训练数据的质量也非常重要，因此我就对训练语料做了相应的预处理：

```ruby
# Step 1: 处理 anti-spam 分隔符
# 把中英文之间的非中英文及数字去掉，即 "合-约" -> "合约"
previous = ""
while previous != cleaned
  previous = cleaned.dup
  cleaned = cleaned.gsub(/([一-龯A-Za-z0-9])[^一-龯A-Za-z0-9\s]+([一-龯A-Za-z0-9])/, '\1\2')
end

# Step 2: 处理中文字符 anti-spam 空格
# 处理 "想 赚 钱" -> "想赚钱" case
previous = ""
while previous != cleaned
  previous = cleaned.dup
  # 匹配中文汉字之间的一个或多个空格，然后删除掉
  cleaned = cleaned.gsub(/([一-龯])(\s+)([一-龯])/, '\1\3')
end

# Step 3: 增加汉字与英文之间的空格
# 以及帮助分词算法如(jieba)更好地分词, e.g., "社区ETH" -> "社区 ETH"
cleaned = cleaned.gsub(/([一-龯])([A-Za-z0-9])/, '\1 \2')
cleaned = cleaned.gsub(/([A-Za-z0-9])([一-龯])/, '\1 \2')

# Step 4: 删除多余的空格(多个空格缩减个一个)
cleaned = cleaned.gsub(/\s+/, ' ').strip
```

预处理之后， `在 币圈 想 赚 钱，那 你 不关 注 这 个 王 牌 社 区，真的太可惜了，真 心 推 荐，每 天 都 有 免 费 策 略` 就会变成 `在币圈想赚钱那你不关注这个王牌社区真的太可惜了真心推荐每天都有免费策略` (这里把合法的逗号也去掉了，我觉得相较过多标点符号对分词的影响，把标点去掉分词结果反而是能接受的), 分词结果是:

`在 | 币圈 | 想 | 赚钱 | 那 | 你 | 不 | 关注 | 这个 | 王牌 | 社区 | 真的 | 太 | 可惜 | 了 | 真心 | 推荐 | 每天 | 都 | 有 | 免费 | 策略`

`这人简-介挂的 合-约-报单群组挺牛的ETH500点，大饼5200点！ + @BTCETHl6666` 就会变成 `这人简介挂的合约报单群组挺牛的 ETH500 点大饼 5200 点！ + @BTCETHl6666` ,分词结果是：

`这 | 人 | 简介 | 挂 | 的 | 合约 | 报单 | 群组 | 挺 | 牛 | 的 | | ETH500 | | 点 | 大饼 | | 5200 | | 点 | ！ | | + | | @ | BTCETHl6666`


#### <span class="section-num">7.1.1</span> 广告新花样 {#广告新花样}

广告看多了，不得不感慨发广告的人的创造力。

因为在消息发垃圾广告会被广告拦截器拦截，他们创新性地玩出了新花样：

{{< figure src="/ox-hugo/spam_by_username.jpg" >}}

消息发的都是正常的文本，但是头像和用户名都是广告，这样广告拦截器就无法工作了，真的是太有创意了。

对手这么有创意，我也因地制宜地建立对用户名的训练模型，检测的时候消息文本的模型和用户名的模型都过一次，
只要有任何一个认为是垃圾广告，那就禁掉。

更进一步的可以对头像做OCR提取文本，再增加一个对头像的训练模型，不过OCR成本挺高的，就先不搞了。


### <span class="section-num">7.2</span> 优化 {#优化}

没有用户的话，做啥优化也没有必要，毕竟过早的优化是万恶之源，
因此我就把想法先做成原型，搞出来再说，但这不意味着这个原型没有优化的空间。

脑海中还是有不少优化的点的：

1.  jieba 分词的效果可能不是最好的，后续可以使用效果更好的分词器进行优化；或者是添加自己的词库。
2.  每次有训练消息都进行重新训练，效率稍低，可以增加 batching 机制：有新消息时，等待5分钟或者等到100条消息再处理
3.  现在整个模型都是在内存中计算，计算完就持久化成 DB, 可以在内存和数据库之间增加一层缓存来优化性能
4.  贝叶斯算法可能效果不够好，换个复杂的机器学习模型

但是这些优化点都算是 Good to have, 不是 Must have, 后面遇到实际问题再进行优化好了。


## <span class="section-num">8</span> 实战效果 {#实战效果}

使用变换之后的垃圾广告词进行发送:

{{< figure src="/ox-hugo/spam_messge_2.jpg" >}}

成功被检测出来，自动删除了:

{{< figure src="/ox-hugo/deleted_spam.jpg" >}}

有朋友可能会说，这只是卖家秀，为什么别人在我群里发的广告还是没有被识别？

因为贝叶斯算法本质是个概率算法，如果它没有见过类似的广告，那么它就没法判断是否垃圾广告 :(

稍安勿躁，你需要做只是使用 `/markspam` 删除消息并封禁用户，就可以帮助训练这个bot, 所有使用这个 bot 的用户都会因此受益


## <span class="section-num">9</span> 结语 {#结语}

我相当享受这种从发现问题、灵光一现，到构建原型，再到最终打磨出一个完整项目的创造过程。

虽然这完全是「用爱发电」——代码开源，还得自掏腰包租服务器，物质上毫无回报。

但每当看到机器人成功拦截广告的那一刻，那种创造的喜悦，就足以令我回味无穷。

-   开源地址：<https://github.com/ramsayleung/bayes_spam_sniper>
-   立即体验：<https://t.me/BayesSpamSniperBot>
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

[^fn:1]: <https://podcasts.apple.com/us/podcast/%E8%BD%AF%E4%BB%B6%E9%82%A3%E4%BA%9B%E4%BA%8B%E5%84%BF/id1147186605>
[^fn:2]: <https://liuyandong.com/sample-page>
[^fn:3]: <https://t.me/huruanhuying>
[^fn:4]: <https://t.me/kaedeharakazuha17>
[^fn:5]: <https://paulgraham.com/spam.html>
[^fn:6]: <https://www.youtube.com/watch?v=Pu675cHJ7bg>
[^fn:7]: <https://www.youtube.com/watch?v=HZGCoVF3YvM>
[^fn:8]: <https://gist.github.com/ramsayleung/5848af0177a70a01d41f624e361b1b5d>
[^fn:9]: <https://ramsayleung.github.io/zh/post/2024/%E7%BC%96%E7%A8%8B%E5%8D%81%E5%B9%B4%E7%9A%84%E6%84%9F%E6%82%9F/>
[^fn:10]: <https://ublockorigin.com/>
[^fn:11]: <https://ramsayleung.github.io/zh/post/2025/a_philosophy_of_software_design/>
[^fn:12]: <https://t.me/pipeapplebun>
