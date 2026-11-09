+++
title = "一天25小时导致的Chromium bug"
date = 2026-11-09T13:46:00-07:00
lastmod = 2026-11-09T16:20:03-07:00
tags = ["programming"]
categories = ["软件设计"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 1天怎么会有25小时？ {#1天怎么会有25小时}

看到标题，读者第一时间会很奇怪，一天不只有24个小时么？这不是常识嘛，怎么会出现25小时。

但是某些国家，比如加拿大，一年中会有1天有25小时，那天之后的半年，也会出现一天只有23小时。

要明白这个突然消失或者增加的一小时，就需要先了解一下夏令时和冬令时。


## <span class="section-num">2</span> 夏令时与冬令时 {#夏令时与冬令时}

正如孔夫子临河感慨那般：
「逝者如斯夫，不舍昼夜」

时间就这么静静地流淌着，我们使用时钟来衡量时间的流逝，有一个总钟表 UTC，它永远不快不慢、不跟着季节调整，它叫 Coordinated Universal Time, 协调世界时间。

它就是时钟里的「基准」，不管你在哪个国家、是不是夏天，UTC 都一样。

慢着，为什么叫 UTC 而不是 CUT?

因为这个缩写不是英文词序的直译，而是英文 `CUT` 和法文 TUC（Temps Universel Coordonné）之间的折中缩写

而某个国家的本地时间就是 _本地时间 = UTC + 时区偏移_

下图的加减标识就是时区的偏移

{{< figure src="/ox-hugo/timezone_map.jpg" >}}

比如中国的北京时区, 东8区就是UTC+8, 而巴西圣保罗就是UTC-3, 假如当前 UTC 时间是 `2026-11-03 04:00:00`, 那么北京的时间就是
`2026-11-03 12:00:00`, 巴西圣保罗的时间就是 `2026-11-03 01:00:00`

那什么是夏令时/冬令时呢?

以加拿大的温哥华或者200公里之外的美国的西雅图为例，正常情况下，他们的时区(太平洋标准时间 Pacific Standard Time, PST)是 UTC-8, 也就是当 UTC 时间是 `2026-11-03 04:00:00`, 温哥华和西雅图的时间是 `2026-11-02 20:00:00`

为了让大家下班后还能多晒到太阳、少开灯，政府就出了个奇怪的规定：「春天某天凌晨，全城把钟表统一拨快一小时」

因为 _本地时间 = UTC + 时区偏移_ , 拨快一小时相当于把时区偏移加一，即从 `UTC-8` 变成 `UTC-7`, 这个 `UTC-7` 就被称为夏令时（Pacific Day Time PDT）；而 `UTC-8` 是标准时间，俗称冬令时(Pacific Standard Time PST).

假如温哥华在3月8日凌晨2点从冬令时切换到夏令时，在11月1号凌晨2点从夏令时切换到冬令时。

那么就会出现3月8日这一天只有23小时，11月1号这一天有25小时的神奇现象。

{{< figure src="/ox-hugo/2025-03-08-pdt.jpg" >}}

{{< figure src="/ox-hugo/2025-11-01-pst.jpg" >}}

看到这里，虽然明白了一年中的确可能会出现一天有23小时/25小时的情况，但是也被搞晕了，这个也太麻烦了。

就不能只选定某个时区嘛？

好消息是某些政府的确意识到这个问题了，所以把冬令时摒弃掉了。坏消息是，只有某些省/州的政府这么决定了，剩下的政府依旧我行我素，继续调回冬令时。

比如温哥华所在的英属哥伦比亚省政府在2026年3月2号宣布： [永久使用夏令时](https://archive.news.gov.bc.ca/releases/news_releases_2024-2028/2026AG0013-000209.htm)[^fn:1]，摒弃了冬令时；

而东边的多伦多所在的安大略省和南边的华盛顿州就继续在夏令时和冬令时之间切换，
导致温哥华和西雅图只相差了200公里（大概广州和香港的距离），却有了一个小时的时差。

时间在软件开发中是非常容易出错的，当我看到这个夏令时和冬令时切换导致的，一天可能出现23小时或者25小时的情况，
我就断言只要假设了一天只会有24小时这个前提（开发者也会觉得很无辜，这明明是常识嘛），那么就一定会出现 bug。

话音未落，这 bug 不就来了嘛.


## <span class="section-num">3</span> Chromium 的 bug {#chromium-的-bug}

Chromium 是由 Google 主导开发的一个开源网页浏览器项目。

它是 Google Chrome 浏览器以及微软 Edge、Opera、三星浏览器等众多主流浏览器的底层实现。

它的代码里面就有一段假设一天只会有24小时引入的bug:

<https://chromium.googlesource.com/chromium/src/+/refs/branch-heads/8078/components/sync/service/device_statistics_scheduler.cc#87>

这段代码的核心逻辑等价于:

```cpp
base::Time ComputeEarliestAllowedTimeToRun() {
  if (last_recorded_at.is_null()) {
    return now;
  }

  return (last_recorded_at + base::Days(1)).LocalMidnight();
}
```

这个函数的值会被 `ScheduleNextRun()` 用来启动定时器：定时器到 `earliest_allowed` 时才触发 `Run()`, `Run()` 里大概有这样的判断

```cpp
if (earliest_allowed > now) {
  ScheduleNextRun();
  return;
 }
// otherwise execute
```

也就是说：

如果「最早允许时间（=earliest_allowed=）」还在未来，就继续等；

如果「最早允许时间（=earliest_allowed=）」已经过去，就执行任务，并更新 =last_recorded_at=，然后再安排下一次。

这个 `ComputeEarliestAllowedTimeToRun` 函数的本意是：

「有些不大重要的任务，比如数据上报，保证任务最多大约每天跑一次，并且尽量在本地午夜之后才允许下一次运行。」

问题在于，它把「加一天」理解成了 ****加 24 小时**** ，但现实中的本地日历 「一天」 不总是 ****24 小时****, 所以上面的函数使用了
`base::Days(1)` 表示精确的 24 小时，也就是 86400 秒。

但是这是物理时间，均匀流逝，并不代表本地的时钟时间.

以我们上面的11月1号温哥华夏令时换冬令时为例，时钟回拨，就会出现一天有25小时的情况.

`last_recorded_at + base::Days(1)`, 在11月1号那天， ****加24小时**** 并不等于是「第二天的相同时间」，而是会落在同一天更晚的时间。

{{< figure src="/ox-hugo/chromium_time_bug.jpg" >}}

假如 `last_recorded_at: 2025-11-01 00:30 PDT`, 因为回拨导致当天有25小时, 所以 `last_recorded_at + base::Day(1)` 就会变成 `2025-11-01 23:30 PST`, 当前时间 now 是 `2025-11-01 00:30 PDT`, 而 `earliest_allowed` `(last_recorded_at + base::Day(1)).LocalMidnight()` 就会变成 `2025-11-01 00:00 PST`

时间比较是没有带时区的，所以 `earliest_allowed < now` 就会变成 `true`, 那么定时器就会被执行。

执行完更新 _last_recorded_at = now_ , 然后再次 `ScheduleNextRun()`

但此时 `now` 仍然在回拨日的 `00:00~01:00` 窗口内，新的 `last_recorded_at` 也还在这个窗口内。再次计算：

```cpp
last_recorded_at + 24h -> 仍然落在同一天
.LocalMidnight()       -> 又得到当天 00:00
```

又早于 now，定时器又立即触发。

了解完 bug 形成的前因后果之后，修复起来也很容易，思路是：

使用本地日历计算「上次记录所在日期的下一天午夜」，而不是先加固定 24 小时再取午夜

Chromium 已经在两周前把问题修复了, patch: <https://chromium-review.googlesource.com/c/chromium/src/+/8655919>

但是思路却是相当「清奇」： 因为夏/冬令时的切换导致会有1个小时的偏差，那么我就再加4个小时的可容纳误差范围，总不能调出4个小时的差异吧:

```cpp
// The metrics should be recorded once per calendar day, so the next possible
// time is midnight on the following day. Add 4 hours before rounding down to
// midnight to account for DST "fall back" days (which are 25 hours long).
base::Time earliest_allowed =
  last_recorded_at.is_null()
  ? now
  : (last_recorded_at.LocalMidnight() + base::Days(1) + base::Hours(4))
  .LocalMidnight();
```

well, 你就说我修没修复吧。

[^fn:1]: <https://archive.news.gov.bc.ca/releases/news_releases_2024-2028/2026AG0013-000209.htm>
