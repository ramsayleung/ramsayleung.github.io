+++
title = "A Telegram Spam Blocker Bot Based On Bayesian Algorithm"
date = 2025-08-30T10:34:00-07:00
lastmod = 2025-09-01T10:06:38-07:00
tags = ["telegram", "bot", "design"]
categories = ["design"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Preface {#preface}

I spent a weekend building a Telegram spam blocker bot based on Bayesian Algorithm. The project is open-sourced at: <https://github.com/ramsayleung/bayes_spam_sniper>


### <span class="section-num">1.1</span> Telegram {#telegram}

Telegram is a popular instant messaging application, similar to Snapchat and WhatsApp, with over 1 billion users.

It supports many powerful features like cloud chat history storage, clients for Linux, Mac, Windows, Android, IOS, and Web (all open-source), Channel, and arguably the most powerful bot system I've ever seen.


## <span class="section-num">2</span> Origin {#origin}

I usually listen to podcasts while running and cooking. 《[软件那些事儿(A podcast in Chinese about history and story behind software)](https://podcasts.apple.com/us/podcast/%E8%BD%AF%E4%BB%B6%E9%82%A3%E4%BA%9B%E4%BA%8B%E5%84%BF/id1147186605)》[^fn:1] is one of my favorites, hosted by [栋哥](https://liuyandong.com/sample-page)&nbsp;[^fn:2]. Because I enjoyed 栋哥's show, I took the chance to join his Telegram channel.

栋哥's Telegram channel [汗牛充栋](https://t.me/huruanhuying)&nbsp;[^fn:3] is primarily used for releasing podcast information.
He once opened the comment function, but it unexpectedly attracted a flood of crypto-related users posting spam, leading him to disable comments:

{{< figure src="/ox-hugo/spam_concert.jpg" >}}

Another channel I subscribe, [Ray Tracing](https://t.me/kaedeharakazuha17)&nbsp;[^fn:4], also complained about the crypto spam:

{{< figure src="/ox-hugo/ray_tracing_spam.jpg" >}}


## <span class="section-num">3</span> Hackers &amp; Painters {#hackers-and-painters}

Most common Telegram spam blocker bots are keyword-based, blocking messages by matching keywords, which can be easily bypassed by spammers.

{{< figure src="/ox-hugo/keyword_based_blocker.jpg" >}}

If the messages get bypassed, it could only be deleted manually by administrator.

This reminded me of the situation Paul Graham described in his 2002 essay within "Hackers &amp; Painters":

When email became popular, there was also a lot of spam. Common spam blockers were keyword matching + email address blacklists, but these were inefficient and easily circumvented.

Paul Graham creatively used Bayesian Theorem to implement a [spam blocker](https://paulgraham.com/spam.html)&nbsp;[^fn:5], and the results were surprisingly effective.

Isn't this a similar problem for Telegram spam?

Couldn't I use a similar solution to tackle Telegram spam?


### <span class="section-num">3.1</span> Bayes' Theorem {#bayes-theorem}

When it comes to probabilistic algorithms, the most classic example is the "coin toss" – a case of classical probability where each toss is an independent event, and the previous outcome doesn't affect the next probability.

However, many real-world scenarios aren't infinitely repeatable like coin tosses, and events are often not independent.

This is where Bayes Theorem shows its unique value.

It used to update our degree of belief in a hypothesis given certain evidence.

In other words, the Bayes algorithm can dynamically adjust the estimated probability of an event occurring based on continuously emerging new evidence.

Simply put, it's like the human brain's learning process: we start with a preliminary understanding, then revise our original view based on new information, thereby adjusting our next actions.

Paul Graham used Bayes theorem to continuously classify new emails as spam or not based on emails already identified as spam or non-spam (ham).

To understand Bayes Theorem more intuitively, I recommend this clear and easy-to-understand videos:

-   《[Bayes theorem, the geometry of changing beliefs](https://www.youtube.com/watch?v=HZGCoVF3YvM)》[^fn:6]


## <span class="section-num">4</span> Architecture Design {#architecture-design}

Telegram Bot supports two modes of interacting with Telegram servers:

1.  Webhook: Telegram servers actively callback a URL previously registered by the Bot whenever the Bot receives a new message. The Bot Server only needs to handle the callback messages.
2.  Long Polling: The Bot Server continuously polls the Telegram servers to check for new messages and processes them if any. This bot uses this mode.

    {{< figure src="/ox-hugo/webhook_vs_long_polling.jpg" >}}


#### <span class="section-num">4.0.1</span> Message Analysis {#message-analysis}

{{< figure src="/ox-hugo/spam_analyze.jpg" >}}

After the Bot Server receives a message, it dispatches it to a separate `telegram_bot_worker` for processing. Based on the pre-trained model, it judges whether it's a spam. If it is, it calls the Bot API to delete the message.


#### <span class="section-num">4.0.2</span> Ban and Train {#ban-and-train}

{{< figure src="/ox-hugo/mark_spam_and_ban_user.jpg" >}}

After the Bot Server receives a message, it dispatches it to a separate `telegram_bot_worker` for processing. The `telegram_bot_worker` calls the bot API to delete the message and ban the user, and inserts a training data record marked as spam.

Saving the training data triggers a hook, creating a training message delivered to the `training` message queue. Another worker, `classifier_trainer`, subscribes to `training` messages and uses the new messages to retrain and update the model.

Using a queue and a background process (`classifier_trainer`) for training tasks, instead of directly using the `telegram_bot_worker`, primarily decouples the Bot request handling from model training. Otherwise, as the model size increases, training time would get longer and longer, leading to increasingly long response times.

Decoupling makes it easy to scale.


## <span class="section-num">5</span> Why Rails {#why-rails}

whoever have seen my project source code might wonder, why was it implemented using Ruby on Rails?

Because I work with JVM languages (Java/Kotlin/Scala) and Rust, I'm quite familiar with Java/Rust. Initially, thinking model training might require high performance, my first [prototype](https://gist.github.com/ramsayleung/5848af0177a70a01d41f624e361b1b5d)&nbsp;[^fn:7] was implemented in Rust, taking about half an hour.

But when I wanted to expand the prototype into a Telegram bot, I found I needed to handle a lot of logic related to bot interaction, mainly involving API and database operations, most of which were unrelated to the model. So, I thought of Ruby on Rails again.

For a single engineer building a product prototype, in my personal opinion, there's really no framework more efficient than `Ruby on Rails`, so I switched to Ruby on Rails.

New features in Rails 8 move it further towards being a so-called "one-person full-stack framework," with built-in support for `Solid Queue` via the relational database.

The queue and background process from the architecture design were implemented with just a few lines of code, without even needing extra configuration. If the queue doesn't exist, the framework creates it automatically:

```ruby
class ClassifierTrainerJob < ApplicationJob
  # Job to train classifier asynchronously
  queue_as :training

  def perform(group_id, group_name)
    SpamClassifierService.rebuild_for_group(group_id, group_name)
  end
end
```

Thanks to Rails' powerful ORM framework and its built-in lifecycle hooks, the code to trigger the background process for retraining the model after inserting new training data is also just a few lines:

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

Empowered by Rails' various built-in powerful tools, I implemented the entire bot's functionality in just one day.

Seeing this, some friends might worry about performance, thinking Ruby's performance isn't great, and it's a dynamic language, making it hard to maintain.

My view remains the same as in my previous blog post 《[Thoughts on a Decade of Programming(In Chinese)](https://ramsayleung.github.io/zh/post/2024/%E7%BC%96%E7%A8%8B%E5%8D%81%E5%B9%B4%E7%9A%84%E6%84%9F%E6%82%9F/)》[^fn:8]:

****Get it running first****.

Build a prototype and get it running. See if users are willing to use your product first.

When running speed becomes a bottleneck, your business must be very successful, and you'll surely have enough resources to hire a team of programmers to optimize the project into Rust/C++, or even assembly.

Without users, discussing performance is a pseudo-proposition.

As for the saying "dynamic languages are fast in the moment, but maintaining the code is a nightmare" I quite agree with that too.

Therefore, I won't consider dynamic languages when choosing a tech stack for a team; I would only use compiled languages, even strong-typed ones like Rust.

But right now, it's just me building a prototype, so I use whatever I'm most proficient with.


### <span class="section-num">5.1</span> Vibe Coding? {#vibe-coding}

Concepts like Vibe Coding and AI programming are everywhere, overwhelming the discourse. You might naturally wonder if this project was generated by Vibe Coding.

The answer is, I tried for a few hours and then gave up entirely. I tried both Claude 4 and Gemini 2.5 Pro.

I started with a Rust + Cloudflare Worker tech stack. Rust + Cloudflare Worker is a relatively niche field with limited training data. The code generated by Vibe Coding failed to compile.

Later, I switched to Ruby on Rails, and the problems became even worse. Ruby is a dynamic language; its syntax is almost like English, and Rails has many "black magic" metaprogramming features.

So errors only appeared at runtime. The development time saved by code generation was entirely consumed by the debugging process.

Another issue is that code generated by Vibe Coding often lacks design. For example, it tightly coupled the `Classifier` and `TrainedMessage` classes, having the `Classifier` persist `TrainedMessage` instances.

It also directly performed synchronous model training within the `telegram_bot_worker` process upon receiving training data, waiting for training to finish before returning the command result, completely neglecting to decouple receiving training data from model training.

One can only say that Vibe Coding is quite suitable for strongly-typed, compiled languages like Rust – at least the generated code has to compile.

As for those claims of "making an APP without writing/changing a single line of code," I can't help but wonder:

Is the code so good that it doesn't need a single change? Or can the developer not identify the crux of the problem, and thus doesn't change anything?


## <span class="section-num">6</span> Design Philosophy {#design-philosophy}

After developing the prototype and making the bot's core functionality usable, many ideas popped into my head.

I immediately rushed to add them to the bot,
resulting in support for nearly ten commands, plus different modes for private chats and group chats.

But I felt something was off. Adding so many features felt like those all-in-one super apps common in China. I began to question:

Would users really use all these features? Would **any** users use these features? Don't too many features also create extra cognitive burden?

My favorite ad blocker, [Ublock Origin](https://ublockorigin.com/)&nbsp;[^fn:9], is powerful and extremely effective at blocking, yet very simple and easy to use.

Recalling the design philosophy mentioned in 《[A Philosophy of Software Design(My thought about the book in Chinese)](https://ramsayleung.github.io/zh/post/2025/a_philosophy_of_software_design/)》[^fn:10], the interface should be simple and easy to use, even if the functionality underneath is complex and rich.

So, I first removed all commands I considered unrelated to the core functionality.

Furthermore, considering that most users might not have a technical background and might not know how to use commands, I optimized the interface to use buttons as much as possible, allowing users to click directly, improving usability:

{{< figure src="/ox-hugo/start_en.jpg" >}}

{{< figure src="/ox-hugo/help_en.jpg" >}}

I also wanted to support multiple languages (e.g., automatically switching to Chinese or English based on the user's system language). This required decent Internationalization support.

{{< figure src="/ox-hugo/start_zh.jpg" >}}

{{< figure src="/ox-hugo/help_page_zh.jpg" >}}

Over 60% of the code in the core service class [telegram_botter.rb](https://github.com/ramsayleung/bayes_spam_sniper/blob/master/app/services/telegram_botter.rb) was introduced for such usability improvements.

Simplicity for the user, complexity for the developer.


### <span class="section-num">6.1</span> How to Use {#how-to-use}

Just two steps, and the bot works automatically.

-   [Add the bot (@BayesSpamSniperBot) to your group](https://t.me/BayesSpamSniperBot?startgroup=true)
-   Grant the bot admin permissions (Delete messages, Ban users)

After these two steps, the bot will not only start working automatically, identifying spam within the group, deleting text messages, and banning users who send spam more than 3 times;

It will also become smarter with community usage (via `/markspam` and `/feedspam`).

{{< figure src="/ox-hugo/detect_spam_and_ban_user.jpg" >}}

The design philosophy of this bot is to minimize disruption to admins and users, provide simple operation commands, and maximize automation.
Therefore, this bot only offers the following three commands (supporting auto-completion with "/"):

{{< figure src="/ox-hugo/command_auto_completion.jpg" >}}


#### <span class="section-num">6.1.1</span> `/markspam` {#markspam}

Delete spam messages and ban the user. Requires admin permissions.

Reply `/markspam` to the message you want to ban, and the bot will automatically delete that message and ban the user.

{{< figure src="/ox-hugo/markspam_2.jpg" >}}

(Message has also been deleted)
![](/ox-hugo/markspam.jpg)

Unlike common group management bots, this command not only deletes the spam and bans the user but also, because this message is marked as spam by an admin with very high confidence, uses this spam ad as training data to update the model in real-time.

Similar messages will not only be identified next time, but all groups using this bot will benefit, as similar texts will also be marked as spam.


#### <span class="section-num">6.1.2</span> `/listspam` {#listspam}

View the list of banned accounts. Requires admin permissions.

{{< figure src="/ox-hugo/listspam.jpg" >}}

View the list of banned users and proactively unban them.


#### <span class="section-num">6.1.3</span> `/feedspam` {#feedspam}

Feed spam messages for training. No permissions required. Can be used in private chat or in-group.

Feeding in private chat:

{{< figure src="/ox-hugo/feedspam.jpg" >}}

Feeding in-group:

{{< figure src="/ox-hugo/feedspam2.jpg" >}}


## <span class="section-num">7</span> Eating your own dog food {#eating-your-own-dog-food}

In the software development field, there's a saying: "Eating your own dog food," which means you should use the things you develop yourself.

So I created my own channel for testing: [菠萝油与天光墟](https://t.me/pipeapplebun)&nbsp;[^fn:11]. Unfortunately, it has very few subscribers,
which fails to attract many spammers. So everyone is welcome to subscribe or come in to post spam, to attract more spammers.

In my channel, everyone has the right to speak freely :-) (the only slight drawback is a limit on frequency ).

Since no one was posting spam in my channel, and suffering from a lack of training data, I had to do it in the hard way: to join various crypto groups, NSFW groups, and actively seek out spam:

The screenshots are Chinese Telegram group contains a lot of spammers and spam:

{{< figure src="/ox-hugo/telegram_group1.jpg" >}}

{{< figure src="/ox-hugo/telegram_group2.jpg" >}}

{{< figure src="/ox-hugo/spam_sample.jpg" >}}

Since developing this bot, my perspective on spam has changed. I used to find spam annoying in other groups, but now I'm happy to see them in other groups,
as they are valuable training data that I need to record quickly before they get deleted.


### <span class="section-num">7.1</span> The Ingenuity of Spam {#the-ingenuity-of-spam}

The algorithm's effectiveness in other people's stories is always surprisingly good, but when I run it myself, I always find various uncovered cases and unexpected surprises, just like life.

Although keyword blocker is inefficient, the spam we actually see are those that have already bypassed keyword filters.

For example:

> 在 币圈 想 赚 钱，那 你 不关 注 这 个 王 牌 社 区，真的太可惜了，真 心 推 荐，每 天 都 有 免 费 策 略
> (Want to make money in the crypto circle? It's a real pity if you don't follow this ace community. Sincerely recommended, free strategies every day)

Or

> 这人简-介挂的 合-约-报单群组挺牛的ETH500点，大饼5200点！ + @BTCETHl6666
> (The contract reporting group linked in this person's bio is pretty awesome. ETH to $500, BTC to $5200! + @BTCETHl6666)

The former uses spaces to bypass keywords, the latter uses punctuation.

Unlike languages based on the Latin alphabet which naturally use spaces for word separation, Chinese requires word segmentation before statistical analysis with Bayes' theorem.

> the fox jumped over the lazy dog
>
> 我们的中文就不一样了(Our Chinese is different)

"我们的中文就不一样了" (Our Chinese is different) would be segmented into "我们 | 的 | 中文 | 就 | 不 | 一样 | 了" before word frequency can be counted.

But for the spam `在 币圈 想 赚 钱，那 你 不关 注 这 个 王 牌 社 区，真的太可惜了，真 心 推 荐，每 天 都 有 免 费 策 略`, the spaces not only affect keyword matching but also affect segmentation. The segmentation result for this sentence becomes(incorrect):

`在 | | 币圈 | | 想 | | 赚 | | 钱 | ， | 那 | | 你 | | 不 | 关 | | 注 | | 这 | | 个 | | 王 | | 牌 | | 社 | | 区 | ， | 真的 | 太 | 可惜 | 了 | ， | 真 | | 心 | | 推 | | 荐 | ， | 每 | | 天 | | 都 | | 有 | | 免 | | 费 | | 策 | | 略`

`这人简-介挂的 合-约-报单群组挺牛的ETH500点，大饼5200点！ + @BTCETHl6666` would be segmented into:

`这人简 | - | 介挂 | 的 | | 合 | - | 约 | - | 报单 | 群组 | 挺 | 牛 | 的 | ETH500 | 点 | ， | 大饼 | 5200 | 点 | ！ | | + | | @ | BTCETHl6666`

Unsanitized training data would affect the model's results, showing the importance of training data quality. Therefore, I performed corresponding preprocessing on the training data:

```ruby
# Step 1: Handle anti-spam separators
# This still handles the cases like "合-约" -> "合约"
previous = ""
while previous != cleaned
  previous = cleaned.dup
  cleaned = cleaned.gsub(/([一-龯A-Za-z0-9])[^一-龯A-Za-z0-9]+([一-龯A-Za-z0-9])/, '\1\2')
end

# Step 2: Handle anti-spam SPACES between Chinese characters
# This specifically targets the "想 赚 钱" -> "想赚钱" case.
# We run it in a loop to handle multiple spaces, e.g., "社 区" -> "社区"
previous = ""
while previous != cleaned
  previous = cleaned.dup
  # Find a Chinese char, followed by one or more spaces, then another Chinese char
  cleaned = cleaned.gsub(/([一-龯])(\s+)([一-龯])/, '\1\3')
end

# Step 3: Add strategic spaces
# This helps jieba segment properly, e.g., "社区ETH" -> "社区 ETH"
cleaned = cleaned.gsub(/([一-龯])([A-Za-z0-9])/, '\1 \2')
cleaned = cleaned.gsub(/([A-Za-z0-9])([一-龯])/, '\1 \2')

# Step 4: Remove excessive space
cleaned = cleaned.gsub(/\s+/, ' ').strip
```

After preprocessing, `在 币圈 想 赚 钱，那 你 不关 注 这 个 王 牌 社 区，真的太可惜了，真 心 推 荐，每 天 都 有 免 费 策 略` becomes `在币圈想赚钱那你不关注这个王牌社区真的太可惜了真心推荐每天都有免费策略` (Note: legitimate commas are also removed here. I find the segmentation result acceptable compared to the impact of excessive punctuation). Its segmentation is:

`在 | 币圈 | 想 | 赚钱 | 那 | 你 | 不 | 关注 | 这个 | 王牌 | 社区 | 真的 | 太 | 可惜 | 了 | 真心 | 推荐 | 每天 | 都 | 有 | 免费 | 策略`

`这人简-介挂的 合-约-报单群组挺牛的ETH500点，大饼5200点！ + @BTCETHl6666` becomes `这人简介挂的合约报单群组挺牛的 ETH500 点大饼 5200 点！ + @BTCETHl6666`. Its segmentation is:

`这 | 人 | 简介 | 挂 | 的 | 合约 | 报单 | 群组 | 挺 | 牛 | 的 | | ETH500 | | 点 | 大饼 | | 5200 | | 点 | ！ | | + | | @ | BTCETHl6666`


#### <span class="section-num">7.1.1</span> Genius New Tricks {#genius-new-tricks}

Seeing many spam, I has to admire the creativity of spammers.

Because sending spam in messages gets caught by blockers, they innovatively came up with new tricks:

{{< figure src="/ox-hugo/spam_by_username.jpg" >}}

The messages contain normal text, but the profile picture and username are spam. This way, the spam blocker can't work. Truly ingenious.

Faced with such creative opponents, I adapted by building a training model for usernames. During detection, both the message text model and the username model are checked.
If either one considers it spam, it gets blocked.

One could go further by doing OCR on profile pictures to extract text and add another model for profile pictures, but OCR is quite costly, so I'll hold off for now.


### <span class="section-num">7.2</span> Optimization {#optimization}

Without users, any optimization is unnecessary, as premature optimization is the root of all evil.
Therefore, I focus on building the prototype first. But this doesn't mean the prototype has no room for optimization.

I have several optimization ideas in mind:

1.  jieba's segmentation might not be the best; other NLP algorithms could be used later for optimization.
2.  Retraining on every single training message is inefficient. A batching mechanism could be added, waiting for 5 minutes or accumulating 100 messages before processing.
3.  Currently, the entire model is computed in memory and persisted to the DB after calculation. A cache layer between memory and the database could optimize performance.
4.  The Bayesian algorithm might not be effective enough; a more complex machine learning model could be used.

But these optimization points are all Good to have, not Must have. I'll optimize them when actual problems arise later.


## <span class="section-num">8</span> Does it capture spam? {#does-it-capture-spam}

Sending a message using transformed spam words:

{{< figure src="/ox-hugo/spam_messge_2.jpg" >}}

Successfully detected and automatically deleted:

{{< figure src="/ox-hugo/deleted_spam.jpg" >}}

Some you might say this is just a demo showcase. Why are spam posted by others in my group still not being detected?

Because the Bayesian algorithm is fundamentally a probability-based algorithm. If it hasn't encountered similar spam before, it cannot determine whether they are spam :(

Be patient. All you need to do is use the `/markspam` command to delete the message and ban the user. This helps train the bot, and all users of this bot will benefit.


## <span class="section-num">9</span> Conclusion {#conclusion}

I thoroughly enjoyed this creative process: discovering a problem, having a spark of inspiration, building a prototype, and finally polishing it into a complete project.

Although this is purely powered by passion – the code is open source, and I have to pay for the server out of my own pocket, with no material return.

But every time I see the bot successfully block an spam, that joy of creation and see it's working is the best reward.

-   Project repository: <https://github.com/ramsayleung/bayes_spam_sniper>
-   Try it now: <https://t.me/BayesSpamSniperBot>

[^fn:1]: <https://podcasts.apple.com/us/podcast/%E8%BD%AF%E4%BB%B6%E9%82%A3%E4%BA%9B%E4%BA%8B%E5%84%BF/id1147186605>
[^fn:2]: <https://liuyandong.com/sample-page>
[^fn:3]: <https://t.me/huruanhuying>
[^fn:4]: <https://t.me/kaedeharakazuha17>
[^fn:5]: <https://paulgraham.com/spam.html>
[^fn:6]: <https://www.youtube.com/watch?v=HZGCoVF3YvM>
[^fn:7]: <https://gist.github.com/ramsayleung/5848af0177a70a01d41f624e361b1b5d>
[^fn:8]: <https://ramsayleung.github.io/zh/post/2024/%E7%BC%96%E7%A8%8B%E5%8D%81%E5%B9%B4%E7%9A%84%E6%84%9F%E6%82%9F/>
[^fn:9]: <https://ublockorigin.com/>
[^fn:10]: <https://ramsayleung.github.io/zh/post/2025/a_philosophy_of_software_design/>
[^fn:11]: <https://t.me/pipeapplebun>
