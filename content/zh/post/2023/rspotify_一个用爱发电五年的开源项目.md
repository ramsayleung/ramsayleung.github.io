+++
title = "RSpotify: 一个用爱发电五年的开源项目"
date = 2023-02-07T15:40:00-08:00
lastmod = 2024-12-30T22:28:48-08:00
tags = ["rspotify", "rust"]
categories = ["rspotify"]
draft = false
toc = true
highlighted = true
+++

## <span class="section-num">1</span> 前言 {#前言}

一周前看到个[新闻](https://www.theverge.com/2023/1/31/23577499/spotify-q4-2022-earnings-release-subscriber-growth-layoffs)，Spotify在其第四季度财报中披露，截至2022年12月31日，它的付费订阅用户数达到了2.05亿，同比增长14%；
月活用户4.89亿。Spotify成为第一家订阅用户数突破2亿的音乐流服务。

而我突然意识到，我那个使用Rust, 为Spotify开发，挂在Sptofiy官网的[library](https://developer.spotify.com/documentation/web-api/libraries/)：[RSpotify](https://github.com/ramsayleung/rspotify)，已经维护有五年了：

{{< figure src="/ox-hugo/web_api_wrapper.png" link="/ox-hugo/web_api_wrapper.png" >}}

{{< figure src="/ox-hugo/commit.png" link="/ox-hugo/commit.png" >}}

一件用爱发电的开源项目要坚持维护五年，也是有很多话可以说的。


## <span class="section-num">2</span> 起源 {#起源}

还记得大三暑假时，也就是2017年，当时找好了实习，并且拿到了Return Offer.
在拿到Offer之后，实习还没有离职，就想学习一门新的编程语言。

因为之前用的是都是Java，Python之类的带GC的编程语言，就想学习点硬核，偏底层的编程语言。
本来是想学C++，结果在知乎看了一圈之后，大家都说C++要没落了，推荐学习Rust。（然后我现在靠写C++混饭吃）

搜索了Rust的信息，发现它性能媲美C/C++, 又不需要手动管理内存，还连续几年荣膺Stackoverflow的[ Most Loved Programming Language](https://insights.stackoverflow.com/survey/2017#technology-_-most-loved-dreaded-and-wanted-languages),  就它了。就开始了一边实习一边摸鱼学习了Rust的旅程。

因为大学前三年把学分都已经修满了，所以大四一整个学年都不需要上课了，就有时间折腾。

在学了2-3个月之后，就想拿Rust来写些项目。
但因为我只会写Web应用，又没有想到能写什么，到时就用Rust写了个[博客](https://github.com/ramsayleung/blog)，并将博客从原来的Github Pages迁移到自建的博客上。

很臭屁地在 [V2ex](https://www.v2ex.com/t/394146) 和 [Reddit](https://www.reddit.com/r/rust/comments/72rr96/i_rewrite_my_blog_with_rust_thanks_for_all/) 分享用Rust重写博客的经历，V2ex 一群人问我为什么不用PHP/xxx语言写，Reddit社区就友好很多。
（然后过了5年之后，服务器欠费，又把自建博客迁移回Github Pages。当然，那是后话了。）

在花了2-3个月写完博客之后，觉得自己入门Rust，就想写个开源项目，感受下与其他开发者协作的场景。

当时看到个网易云音乐命令行版本的播放器 [musicbox](https://github.com/darknessomi/musicbox), 当时我在用的是Spotify，就希望可以为Spotify写个类似的播放器。

虽说Spotify API是对外开放，但直接使用HttpClient来请求HTTP API有点太祼，所以就希望使用先封装个library，方便后续的Rust应用直接调用，就不需要自己操心Http请求了。

这就是RSpotify这个库的来源。

这次，我就只在 Reddit和博客 上分享使用Rust来写library 的经历了。

-   Reddit: [My first crate-- rspotify, Spotify API wrapper implemented in Rust](https://www.reddit.com/r/rust/comments/7xn9mh/my_first_crate_rspotify_spotify_api_wrapper/)
-   博客: [RSpotify– 我的第一个Rust crate](https://ramsayleung.github.io/zh/post/2018/rspotify/)


## <span class="section-num">3</span> 演进 {#演进}


### <span class="section-num">3.1</span> 野蛮生长阶段 {#野蛮生长阶段}

刚开始写RSpotify的时候，对于如何设计一个易用，友好的library 完全没有头绪，毕竟设计好用的类库需要相当的经验沉淀。

对于没有设计思路的我而言，当时能想来的解决方案是去Spotify官方列出来的[library](https://developer.spotify.com/documentation/web-api/libraries/)看下，哪个语言的library看得懂，star又多，就把这个library 翻译到Rust上。

就把目光瞄准到Python版本的 [spotipy](https://github.com/spotipy-dev/spotipy) 上。

在2018-01-08 提交了第一个commit, 经过一个多月的日夜施工，终于在[2018-02-18](https://github.com/ramsayleung/rspotify/commit/bb93cc9cc52d5ee62e72b8be1c33a5e3e3ae60ac) 完成了所有的API接口开发，发布了[ 0.1版本](https://crates.io/crates/rspotify/0.1.0)

虽然这是我这个学生写的第一个Rust库，但是开源项目需要的标准配置，我还是都加上了：

-   自动化流水线，Travis（当时Github Action还没有出现）
-   齐全的[文档说明](https://docs.rs/rspotify/0.1.1/rspotify/index.html)
-   完整的单元测试用例
-   使用示例
-   README说明与License

为了吸引其他开发者来协作开发，所有的资料都是英文的。

不过从Rust 包托管网站 [crates.io](https://crates.io/crates/rspotify/0.1.0) 的数据可以看到，0.1版本只有300+的下载量，几乎没有什么人在用。

{{< figure src="/ox-hugo/stats_0.10.png" link="/ox-hugo/stats_0.10.png" >}}


### <span class="section-num">3.2</span> async 阶段 {#async-阶段}

时间来到2019年，对于Rust社区来说，[最激动人心](https://www.reddit.com/r/rust/comments/ct7aus/stabilize_async_await_in_rust_1390/)的应该是Rust 1.39版本，将正式包含 `async/await` 特性，自那天起，Rust正式支持异步编程。

自此之后，Rust社区在做的事情，就是把已有Rust代码疯狂升级到async await，RSpotify虽迟，但也赶上了这波潮流。

当时RSpotify 请求Spotify的API使用的HTTP库是 `reqwest=，在 =reqwest` 支持异步模式之后，开发者 [Alexander](https://github.com/Rigellute)就提了一个超大的[PR](https://github.com/ramsayleung/rspotify/pull/81)，把所有已有的api全部修改成=async=, 我就乐见其成，就把这个PR合并了。

{{< figure src="/ox-hugo/add_async_await_1.png" link="/ox-hugo/add_async_await_1.png" >}}

有社区的同学抱怨说异步模式的代码不好使用，他对性能没有什么要求，能否保留同步模式的接口调用。

后来为了兼顾同步模式和异步模式这两种调用方式，Alexander 又提了一个超大超大的[PR](https://github.com/ramsayleung/rspotify/pull/82/files)，把现有的异步模式代码复制一份，然后把=async= 关键字去掉。

{{< figure src="/ox-hugo/add_async_await_2.png" link="/ox-hugo/add_async_await_2.png" >}}

从此以后，RSpotify就需要同时维护两份几乎相同的代码，每次新增，修改，删除都需要确保同时变更两份代码。
着实痛苦不堪，但我也没有思考出更优解。

这时候，后来和我共同维护RSpotify 的开发者 [Mario](https://github.com/ramsayleung/rspotify/pull/102) 出现了。


### <span class="section-num">3.3</span> maybe_async 阶段 {#maybe-async-阶段}

当时RSpotify最大的问题在于有两份几乎一样，但是使用同步调用和异步调用模式的代码。

而异步调用的代码，返回参数都是一个 `Future<T>` ，将真正的响应结果封装在一个 `Future` 结构里面。

所以当时Mario 提出的第一个[解决思路](https://github.com/ramsayleung/rspotify/issues/112)，是将对异步代码进行封装，使用同步调用的runtime调用异步函数，然后再把响应结果返回回去：

```rust
// 异步代码
async fn original() -> Result<String, reqwest::Error> {
    reqwest::get("https://www.rust-lang.org")
        .await?
        .text()
        .await
}

lazy_static! {
    // Mutex to have mutable access and Arc so that it's thread-safe.
    static ref RT: Arc<Mutex<runtime::Runtime>> = Arc::new(Mutex::new(runtime::Builder::new()
                                                                      .basic_scheduler()
                                                                      .enable_all()
                                                                      .build()
                                                                      .unwrap()));
}

// 同步版本代码
fn with_block_on() -> Result<String, reqwest::Error> {
    RT.lock().unwrap().block_on(async move {
        original().await
    })
}
```

再通过Rust macro来为每个async 函数生成一个block_on 版本的函数。

但实际上，发现编写[macro太复杂](https://github.com/ramsayleung/rspotify/pull/120)，并且这种方案不够灵活，实现起来也相当复杂。

然后Mario 又调研出一种[新方案](https://github.com/ramsayleung/rspotify/pull/129)，通过 [`maybe_async`](https://github.com/fMeow/maybe-async-rs) 这个库在同步和异步模式之间切换。
默认是异步模式，但可以通过 `features = ["is_sync"]` 编译选项来切换到同步模式，=maybe_async= 就会把所有的=async/await= 关键字给去掉。

这个方案简单，可读性高，易于扩展，也不需要维护复杂的 macro 代码。

这也是我们最终采取的方案，重构之后，把 `blocking` 目录的近万行复制粘贴而来的代码删除掉，就非常爽。


### <span class="section-num">3.4</span> 二次重构 阶段 {#二次重构-阶段}

前面提到，RSpotify最开始是直接翻译spotipy的代码。
因为Python是弱类型，而Rust是强类型，直接翻译，难免会有不少代码，写法没有纯正的Rust味道。

社区的Kestrer同学就在一个[issue](https://github.com/ramsayleung/rspotify/issues/127)里面，给RSpotify提了近90条优化建议，指出了RSpotify中设计的各种问题，包括强类型运用不当，使用过多的原始类型，函数出入参设计不够优雅，授权流程设计不够易用等等。

{{< figure src="/ox-hugo/meta-issue.png" link="/ox-hugo/meta-issue.png" >}}

这么多的优化建议，可以看出Kestrer真的花了很多时间来阅读和改善RSpotify的代码，盛情难却（可见原来的代码是+多烂+, 有非常多激发社区同学参与改进的空间）.

别人指出问题，就要好好优化。

所以我和Mario就分别对每个接口返回的数据模型，数据模型与Json间转换的序列化方式，授权流程作了改进。
并对RSpotify这个library作了拆分，按照功能，拆分成 `model`, `http`, `macros` 三个单独的library。

{{< figure src="/ox-hugo/trait_hierarchy.png" link="/ox-hugo/trait_hierarchy.png" >}}

{{< figure src="/ox-hugo/crate_hierarchy.png" link="/ox-hugo/crate_hierarchy.png" >}}

期间提了大概20多个PR，花了超过一年的时间，才处理完Kestrer 提的所有建议。


### <span class="section-num">3.5</span> pre-release 阶级 {#pre-release-阶级}

在开源社区里面，有一个约定俗成的规范：
当一个library 发布1.0 之后，就代表这个库已经处于稳定状态，不会再出现大量breaking change 的情况了。（py2, py3不在此约束内）

而经过4年的开发，RSpotify 已经步入一个相对稳定的开发状态，没有太多的breaking change 或重构了，开始为发布正式的1.0release 版本作准备。

当功能与架构相对稳定后，近一年时间，我和Mario就开始优化RSpotify的易用性，比如

1.  添加更多，针对不同场景的 [examples](https://github.com/ramsayleung/rspotify/tree/master/examples)；
2.  尽可能地去掉 `unsafe` 代码；
3.  为返回列表的API提供同步及异步版本的[Iterator支持](https://github.com/ramsayleung/rspotify/pull/166);
4.  保持向前兼容的情况下，尽量使已有接口更加Rust化；
5.  添加更多的自动化检查，如检查代码中文档的链接是否404；
6.  性能优化，减少不必要的内存分配

目前版本已经去到了 `0.11.6`, 功能也相对稳定, 预计不久后就会正式发布1.0版本。


## <span class="section-num">4</span> 感悟 {#感悟}


### <span class="section-num">4.1</span> 开源协作 {#开源协作}

截至到2023-02-09，RSpotify一共有1673次commit, 但我和Mario都只贡献了1/3的commit，剩下的commit都是社区的其他开发者提交的。

{{< figure src="/ox-hugo/1673.png" link="/ox-hugo/1673.png" >}}

{{< figure src="/ox-hugo/contributor.png" link="/ox-hugo/contributor.png" >}}

从RSpotify的演进历程也可以看出，我只是从0开发了最初版本的RSpotify，后面都是随着Rust的演进，有不同的开发者帮忙优化与迭代，我做的事情就从单纯的creator, developer 变成maintainer, reviewer，负责review其他开发者的PR。

可以说，如果没有其他开发者的贡献与协作，RSpotify不会演进成现在的样子。

如何吸引更多的开发者加入，让他们乐于为项目作贡献，我个人的见解是：

1.  所有文档，注释，commit message, issue, CHANGELOG等材料，都只使用英文。
2.  标准的开源协作流程；
    -   [issue](https://github.com/ramsayleung/rspotify/tree/master/.github/ISSUE_TEMPLATE), [PR](https://github.com/ramsayleung/rspotify/blob/master/.github/pull_request_template.md), [CHANGELOG](https://github.com/ramsayleung/rspotify/blob/master/CHANGELOG.md) 都提供标准模板
    -   要添加新特性，修改已有功能的时候，新建issue讨论动机与可行性
    -   每个PR都需要一个Peer Reviewer review后才能合并
    -   每次发新版本，都需要在 [CHANGELOG](https://github.com/ramsayleung/rspotify/blob/master/CHANGELOG.md) 注明大的特性变更，以及breaking change
3.  文档，示例，开发指引，测试case完备，降低新开发者参与的成本。
4.  be nice，态度友好，针对issue，PR都尽量回复，理性，友善讨论。

开源协作的一个感受就是，在Github讨论问题的时候，可能突然有位大佬也加入群聊。

比如和[Mario讨论](https://github.com/ramsayleung/rspotify/issues/204), 增加更多更严格cargo clippy 的rule，以便让编译器帮我们发现更多潜在问题时，cargo clippy 的maintainer 也加入讨论，就什么rule 更合适，给出自己的建议。


### <span class="section-num">4.2</span> 收获 {#收获}

我用C++已经混了三年的饭吃了，但还只能看到C++的门槛，没法说入了C++的门。

同理，虽然距离我学习Rust已经过去6年了，我依然感觉我还不会Rust，都是编译器教我写代码。

在Review别人代码的过程中，我也学习到非常多「地道」和高级的Rust用法，项目维护的经验.

想到的点：

1.  使用Rust的macro来减少copy-paste的代码（但复杂的 macro，基本不具备可读性。）
2.  使用 serde 自定义序列化函数；
3.  以workspace 模式管理多个crates;
4.  编写 async/await 的异步代码；
5.  使用标准库的Trait, 风格契合标准库；
6.  结合thiserror 和anyhow 处理异常；
7.  通过自动化和模式化，减少项目维护的成本（能用机器做的，就不要用人做）。
8.  规范的开发流程，包括commit message, issue, PR, CHANGELOG, release note 等等

期间把收获与心得写了两篇文章：

-   [The lesson learned from refactoring rspotify](https://ramsayleung.github.io/post/2020/serde_lesson/)
-   [Let's make everything iterable](https://ramsayleung.github.io/post/2021/iterate_through_pagination_api/)


### <span class="section-num">4.3</span> 关于开源 {#关于开源}

维护这个项目5年之后，对于「开源」有了些不一样的理解。

在1970 年代，Richard Stallman发起自由软件运动，旨在推广用户有使用，复制，研究，修改和分发软件的社会运动。
自由软件运动人士认为自由软件的精神应该贯彻到所有软件。

在90年代，又兴起了开源软件运动，则计算机软件的源代码是可以公开，随意获取的。
（自由软件与开源软件不是同一个概念，自由软件定义更为严格）

在那个崇尚黑客精神的年代，开源是「目的」，是为了贯彻自由的精神。

以前听到某某公司内部有好用的工具，组件，框架时，总会问一句，为什么他们不像Google一样把它们开源出来。

现在的想法可能是，为什么要开源出来，价值和收益是什么？

开源一个项目的目的可能是：

1.  我做了个很有用，很有趣的东西，就想分享出来。但是我个人人力有限，大家一起来帮忙做大做好。(Linux, Ruby On Rails等)
2.  我们做了个好东西，我们要抢占市场。我们就开源，搞人海战术，让竞品淹没在人民群众的汪洋大海中，让我们的东西成为事实的标准。（Android，Chromium, Kubernetes, Vscode）
3.  就想开源让你们见识下大佬是怎么样子的。

对于商业公司而言，如果没有收益，为什么要把花钱雇的人写的内部组件开源出来呢？总不成是为了在B站上博取小朋友的称赞吧。

而公司内部的组件，往往是与业务共生，高度适配的，藕断丝连，没有那么容易开源的。

商业公司，只谈收益与预期，如果名声能卖钱，估计也会拿来换取利润。

更重要的是，开源并不是简单把代码公开出来。

软件和生物一样，是有生命的，需要长期维护的，而不是一个commit把所有代码一把push 到Github就完事了，或者然后过了几年又push一把，更新几百个文件。

开源是一个技术与管理结合的决定，需要把开发模式都切换到开源社区，决策过程与动机要对社区可见。

不仅让人能从代码中读懂功能是「什么」，也要从动机讨论中知道「为什么」要这么改。


### <span class="section-num">4.4</span> 些许成果 {#些许成果}

在Github上，被1108个仓库及18个package 所依赖，而其中Alexander的 [spotify-tui](https://github.com/Rigellute/spotify-tui) 就是我期望做的终端版本的Spotify。

开源的好处就是，在开发好基础设施之后，自然就会有其他有相同想法的同学，把应用开发出来。

{{< figure src="/ox-hugo/dependent_packages.png" link="/ox-hugo/dependent_packages.png" >}}

crates.io 的统计，总计被下载23w次，当然包括很多CI的重复下载。

{{< figure src="/ox-hugo/stats.png" link="/ox-hugo/stats.png" >}}

对于有Rust，Spotify，Library等诸多定语的RSpotify来说，目标受众本来就不多，能有现在这样的用户量也算是个挺不错的成果了。


## <span class="section-num">5</span> 总结 {#总结}

虽然我未曾从这个项目上获得到一分物质上的回报，但在创建这个项目的时候，我可能不会想到，我能维护它长达五年。

天上的云，飘来又飘走；开源的项目，挖坑又弃坑。

视线望不穿下一个五年，唯有且行且看。


## <span class="section-num">6</span> 参考 {#参考}

-   [Spotify is first music streaming service to surpass 200M paid subscribers](https://www.theverge.com/2023/1/31/23577499/spotify-q4-2022-earnings-release-subscriber-growth-layoffs)
-   [RSpotify](https://github.com/ramsayleung/rspotify)
