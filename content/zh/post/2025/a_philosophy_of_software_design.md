+++
title = "软件设计的哲学"
date = 2025-05-30T00:39:00-07:00
lastmod = 2025-09-01T09:55:35-07:00
tags = ["book", "design", "programming"]
categories = ["读书感悟", "软件设计"]
draft = false
toc = true
showQuote = true
highlighted = true
+++

## <span class="section-num">1</span> 前言 {#前言}

知道这本书是因为在 Hacker News 上有人提问：[你读过最好的技术书是什么](https://news.ycombinator.com/item?id=31713756)&nbsp;[^fn:1]?

最高赞的书是 Design Data Intensive Application(DDIA, 即《[数据密集型应用系统设计](https://book.douban.com/subject/30329536/)》[^fn:2]), 我觉得 DDIA 也担得起这个赞誉，然后最高赞的回答顺势提到了 [A Philosophy Of Software Design](https://book.douban.com/subject/30218046/)&nbsp;[^fn:3], 想来能与 DDIA 齐名的书，肯定不会差得哪里去。

作者是 John Ousterhout, 斯坦福大学的教授，TCL 编程语言的创造者(Redis 的初始化版本就是用 TCL 写的)，共识算法 Raft 的作者之一.

这本书并不厚，全书只有200多页，读起来也并不费劲。

而这本书的主旨，开篇就点出来了:

> This book is about how to design software systems to minimize their complexity.
>
> 本书讲述如何设计软件系统以最小化其复杂度

而软件工程的本质就是如何管理复杂度，全书围绕如何降低软件复杂性提出的思考和解决方案，
主要围绕抽象，异常，文档，一致性，设计原则这五个方向。

许多原则我看着都深有共鸣，尤其在设计过相当多的系统之后，犯过许多错误之后，才会意识到这些原则的重要之处。

很多原则看上去说的和没说一样，但只有踩过坑，实践起来都知道是金科玉律, 除了道出「软件设计」的真谛之外, 这本书其他论点也可谓字字珠玑.

关于谨慎暴露过多的配置给用户，尽量让程序动态计算各种参数值，尽量提供默认参数。

> 开发软件时，开发者主动承担一些额外痛苦，从而减少用户的痛苦。
>
> When developing a module, look for opportunities to take a little bit of extra suffering upon yourself in order to reduce the suffering of your users.

关于接口设计的原则:

> 模块拥有简单的接口比简单的实现更重要。
>
> it's more important for a module to have simple interface than a simple implementation

关于异常处理的洞见:

> 解决问题的最好方式是避免出现问题。
>
> The best way to eliminate exception handling complexity is to define your APIs so that there are no exceptions to handle: ****define errors out of existence****
>
> 归根结底，减少 Bug 的最佳方法是让软件更简单(少即是多)
>
> ****Overall, the best way to reduce bugs is to make software simpler.****


## <span class="section-num">2</span> 抽象 {#抽象}

所谓的抽象，用我自己的话来说的就是把复杂的东西简单地呈现出来。


### <span class="section-num">2.1</span> 模块深度 {#模块深度}

为了直观地感受一个模块设计是否足够抽象，作者提出一个模块深度的概念:

{{< figure src="/ox-hugo/deep_module.jpg" >}}

矩形的表层长度即是接口的复杂程度，而矩形的面积代表模块实现的功能，好的模块应该是深的(deep), 这意味着它有简单的接口，但是内部有复杂且丰富的实现.

例如 Unix 的文件读写接口:

```c
int open(const char* path, int flags, mode_t permissions);

ssize_t read(int fd, void* buffer, size_t count);

ssize_t write(int fd, const void* buffer, size_t count);

off_t lseek(int fd, off_t offset, int referencePosition);

int close(int fd);
```

接口非常简单，但是其内部的实现可能需要成千上万行的代码，
需要支持文件目录的读写，文件权限，读写缓冲区，磁盘读写等等功能，这就是「深的」模块。

与其相反的就是浅的模块(shallow), 接口很复杂，但是功能却很简单。


### <span class="section-num">2.2</span> 信息的漏与藏 {#信息的漏与藏}

实现抽象的关键手段就是辨别出信息的重要程度，对于不重要的信息，就要对用户隐藏起来，关键的信息，就要暴露给用户, 实现「去粗存精，开箱即用」。

一个典型的例子就是参数配置，把参数暴露给用户，除非用户非常熟悉这个系统，不然他也不知道怎么算，
不需要用户关注的参数就提供默认值，能程序动态计算就由程序自己来算.

我很反感的一种设计就是引入一个配置系统，系统的运行参数都要由工程师配置，美其名是提供灵活度。

但这不仅引入额外的系统依赖（须知复杂度的根源就来自依赖与不明确），还大大增加了的运维成本，
更何况这样的配置还无法自适应，换种机型又要重新配置，导致配置越来越复杂。

除非是业务的黑名单或者白名单，系统的运行参数能用默认的就用默认，能动态计算就动态计算。

想想TCP/IP 的重试延迟时长如果不是动态计算，那么配置什么值比较合适，网络畅通和网络延迟又该是什么值，
开始恢复时和开始堵塞时又应该是什么值的呢?


## <span class="section-num">3</span> 异常 {#异常}

异常处理是系统复杂度的关键来源之一，异常就是一种特殊的分支，系统为了处理特殊 case难免需要写很多额外的逻辑。

而作者提出的降低异常处理来系统复杂度影响的方法，就是优化设计，减少必须处理异常的地方。

解决一个问题最好的方法是避免其发生，听起来很空洞或者是很不可思议，作者举出来的例子就是 Java 的 `substring(int beginIndex, int endIndex)` 用于截取子字符串的接口, 如果 `endIndex` 超出字符长度，Java 就会抛出一个 `IndexOutOfBoundException`, 调用方就是需要考虑越界的问题。

但是如果 Java 的 `substring` 接口本身可以像 Python 那样支持越界，返回一个空字符串，那么调用方就完全不需要考虑越界导致的异常

另外一个例子是作者设计的TCL脚本中的 `unset` 指令，原意是用来删除一个变量，因为他最初的设想是变量如果不存在，用户不可能调用 `unset` 的，那么当 `unset` 操作的变量不存在，那么就会抛出异常。

但是很多用户就是用 `unset` 来清理可能被初始化或者未初始化的变量，现在的设计就意味用户还需要包一层 `try/catch` 才能使用 `unset`.

意识到这个设计错误之后，作者对 `unset` 的语义作了稍微的修正，用 `unset` 来确保指定的变量不再存在(如果变量本身不存在，那么它什么都不需要做)

更经典的例子就是 Windows 下面删除一个文件，相信使用过 Windows 的朋友尝试删除文件时都会遇到这样的弹窗：「文件已被打开，无法删除，请重试」

{{< figure src="/ox-hugo/windows_delete_opening_file.png" >}}

用户只能费尽心思去找打开这个文件的进程，然后把它杀掉再尝试删除，甚至只能通过关机重启来尝试删除文件。

但是 Unix 的处理方式就更优雅，它允许用户删除已经被其他进程打开的文件，它会对该文件做标记，让用户看来它已经被删除了，但是在打开它的进程结束前文件对应的数据都会一直存在。

只有在进程结束后，文件数据才会被删除掉，这样用户在删除文件时就不需要担心文件是否被使用。

通过优化以上的设计，减少需要用户处理的异常，这也是一个「去粗留精」的过程, 减少用户需要感知的内容。


## <span class="section-num">4</span> 注释 {#注释}

本书用了好几个章节来介绍文档与注释的重要性，命名的重要性，如何写好注释和起好名字。

好的文档可以大幅改善一个系统的设计，因为文档的作用就是把「对用户重要的，但是无法直接从代码中得知的关键信息告知用户」, 相当于帮用户把一个系统的关键信息给找出来。

不是有这么一句话： ****程序员都讨厌写文档，但是更痛恨其他程序员不写文档。****

****而注释就是离源码最近的文档.****

程序员不写注释的借口大概有这么几个（可惜它们都是不成立的）, 常见的借口与它们不成立的原因可见:


### <span class="section-num">4.1</span> 好的代码是自解释的 {#好的代码是自解释的}

如果用户必须阅读方法源码才能使用它，那就没有抽象，你相当于把实现的所有复杂度都直接暴露给用户。

若想通过抽象隐藏复杂性，注释必不可少


### <span class="section-num">4.2</span> 我没有时间写注释 {#我没有时间写注释}

如果你一直把写代码的优先级置于写注释之上，那么你会一直没有时间写注释，
因为一个项目结束之后总会有新的项目到来，如果你一直把写注释的优先级放在代码之后，那么你永远都不会去写注释。

写注释实际并不需要那么多的时间


### <span class="section-num">4.3</span> 注释都会过期的啦 {#注释都会过期的啦}

注释虽然难免会过期，但是保持与代码一致也并不会花费太多时间。

只有大幅需要修改代码时才需要更新注释，更何况，只有每次都不更新注释，注释才会难免过期


### <span class="section-num">4.4</span> 我见过的注释都很烂，我为啥还要写 {#我见过的注释都很烂-我为啥还要写}

别人的注释写得不好，那不正说明你可以写出好的注释嘛。

不能用别人的低标准来要求自己嘛。


### <span class="section-num">4.5</span> 注释的原则 {#注释的原则}

说起接口注释和文档，我一直觉得我描述下接口功能和使用场景，已经比绝大多数的同行做得好了。

在和现在的 L7 大佬一起工作之后，着实被他的文档所震撼。

不知道是因为其对代码质量和文档都有非常高的要求，还是读博士时训练出来的写作能力，
其对接口的功能，使用场景以及异常的描述都非常详尽，甚至包括代码使用示例，质量与 JDK 源码的注释不相上下, 原来真的有程序员花这么多精力写代码注释的。


#### <span class="section-num">4.5.1</span> 注释应当描述代码中不明显的内容 {#注释应当描述代码中不明显的内容}

****注释应当描述代码中不明显的内容****,

简单来说，就是要描述代码为什么要这么做，而不是描述代码是怎么做的，这相当于是把代码换成注释再写一次。


#### <span class="section-num">4.5.2</span> 注释先行 {#注释先行}

很多程序员都习惯在写完代码之后才写注释，作者反其道而行， 作者推荐在定义完函数或者模块接口之后，不要马上动手写实现，
而是在这个时候在接口上把接口注释写下来，这相当于是在脑海把模块的设计再过一次。

写完代码再写注释，设计思路已经记不大清了，脑中更多的是实现细节，既容易把实现写成注释，又容易陷入「写完代码就不写注释」的陷阱。


## <span class="section-num">5</span> 一致性 {#一致性}

前文提到，系统的复杂度来自于两个方面「依赖」与「不明确」，
而「一致性」就是让系统的行为更加清晰明确。

****它意味着相似的事情以相似的方式处理，不同的事情以不同的方式处理。****

即所谓的「规圆矩方」，通过规范约束降低随意性，以及「一法通，万法通」，统一模式提升可维护性，让行为可预期。

一个系统的一致性一般体现在以下方面：

1.  命名(驼峰还是下划线)
2.  代码风格(缩进，空格还是tab)
3.  设计模式(使用特定的设计模式解决特定的问题)

当然，还有通过「一致性」降低系统复杂度，走得比较极端的:

之前还在微信支付的时候，除上述的要求外，还要求后端只能使用一种语言(C++, Golang/JavaScript就别想了), 存储组件只能使用微信内部研发的KV(使用MySql需要向总经理申请)等等的要求.


## <span class="section-num">6</span> 设计原则 {#设计原则}


### <span class="section-num">6.1</span> 通用设计 {#通用设计}

好的设计应该是通用的，优先采用通用设计而非特殊场景的定制化方案，这个是减少复杂度和改善软件系统的根本原则。

过度定制通常是成为软件复杂度增加的首要诱因。

通用设计可以降低系统的整体复杂度(更少处理特殊分支的逻辑), 更深的模块(接口简单，功能丰富), 隐藏非关键信息.

文中提到的例子就是文本编辑器的文字插入与删除操作:

```java
// 反例：过度定制（绑定特殊场景）, 实现删除键功能
class TextEditor {
    void handleBackspaceKey() { // 耦合UI事件
        if (cursorPosition > 0) {
            text.deleteCharAt(cursorPosition - 1);
            cursorPosition--;
        }
    }
}

// 正例：通用设计（解耦核心逻辑）
class Text {
    void delete(int start, int end) { // 纯文本操作
        content.delete(start, end);
    }
}

class UI {
    void onBackspacePressed() {
        text.delete(cursor.position(), cursor.position() + 1); // 调用通用API
        cursor.moveLeft();
    }
}
```

通过 `delete(int start, int end)` 既可以实现删除键功能，也可以实现选中并删除的功能。


### <span class="section-num">6.2</span> 性能 {#性能}

在设计系统的时候，一般不需要太多地考虑性能的问题，因为简单，通用的系统要做性能优化通常都是比较容易；
相反而言，深度定制的系统因为耦合了定义逻辑，要优化性能并没有那么容易。


### <span class="section-num">6.3</span> 设计两次 {#设计两次}

Design it twice

因为很难一次就把事情做到极致, 那就再来一次, 设计时把能想到的选项都列下来.

反直觉的是，第一直觉通常不是最优的, 所以不要只考虑一种设计方案，无论它看起来多么合理，多对比下其他方案总没有害处的。

只用第一直觉的方案，其实你是在低估自己的潜力，你错失了找到更好方案的机会。

这也是我在写设计方案时候的做法，把自己能想到的，和同事讨论出来的所有方案都写上，然后分析各种方案的优劣, 最好的方案可能并不在原有方案列表里面，而是其中几个方案的合体。


### <span class="section-num">6.4</span> 大局观 {#大局观}

做任何事都要有大局观, 编程也不例外，战略编程优于战术编程(Strategic Programming over Tactical Programming);

虽然我们一直说「又不是不能跑」，但是我们对代码的要求，不能是「能跑就行啦」.

再者就是要和扁鹊他大哥治病一样，把功夫都做在前期，防范于未然，修补错误成本往往也越往后越高，病入膏肓之后，扁鹊来了也要提桶跑路:

> 治不了，等死吧，告辞


## <span class="section-num">7</span> 代码整洁之道vs软件设计哲学 {#代码整洁之道vs软件设计哲学}

本书的作者对[《代码整洁之道》](https://book.douban.com/subject/34986245/)(Clean Code)[^fn:4] 的作者(Robert C. Martin, 即 Uncle Bob)的诸多观点作了反驳


### <span class="section-num">7.1</span> 函数拆分 {#函数拆分}

比如关于什么时候应该拆分一个函数，Uncle Bob 的观点是，基于函数的代码行数，一个函数需要相当短，甚至10行都有太长了。

Uncle Bob 原话:

> In the book Clean Code1, Robert Martin argues that functions should be broken up on length alone. He says that functions should be extremely short, and that even 10 lines is too long.

而本书作者 John 的观点是: ****每个函数应只做一件事，并完整地做好****

函数的接口应当简洁，这样调用者无需记住大量信息就能正确使用它。

函数应当具备深度：其接口应远比实现更简单。如果一个函数满足以上所有特性，那么它的长度通常并不重要。

****除非能让整个系统更简单，否则不应拆分函数****


### <span class="section-num">7.2</span> 文档注释 {#文档注释}

Uncle Bob 认为需要给函数「注释始终是一种失败(****Comments are always failures****)」

如果我们的编程语言足够富有表现力，或者如果我们有能力用好这些语言来传达意图，那么我们就不太需要注释——甚至可能完全不需要.

****注释的正确用途，是弥补我们无法用代码清晰表达的缺陷……注释始终是一种失败****

> If our programming languages were expressive enough, or if we had the talent to subtly wield those languages to express our intent, we would not need comments very much — perhaps not at all.
>
> he proper use of comments is to compensate for our failure to express ourselves in code.... Comments are always failures.

而 John 的观点是

但注释并非失败的表现。

****它们提供的信息与代码截然不同，而这些信息目前无法通过代码本身来表达。****

****注释的作用之一，正是让人无需阅读代码即可理解其含义****

甚至直接反驳其观点:

> I worry that Martin’s philosophy encourages a bad attitude in programmers, where they avoid comments so as not to seem like failures.


### <span class="section-num">7.3</span> 网上对线 {#网上对线}

所以也难怪 Uncle Bob 和 John Ousterhout 几个月前直接在网上论坛来了一次 ~~对线~~ ([辩论)](https://news.ycombinator.com/item?id=43166362)&nbsp;[^fn:5]

然后有看热闹不嫌事大的播主，把两人邀请到直播上，让他们直接面对面再来了一次对线

对应的Youtube视频: <https://www.youtube.com/watch?v=3Vlk6hCWBw0>

两位的书我都看过，我个人的感觉是《代码整洁之道》更适合入门的工程师，它可以教你如何写出好的「代码片段」；
而《软件设计的哲学》更适合需要做系统设计的工程师，它指导你如何设计好的「软件」。

考虑到两位作者的背景和作品，我可以说两位的差别可以说是 ****以编程为生的人与以写编程相关的东西为生的人****


## <span class="section-num">8</span> 总结 {#总结}

全书读完，我觉得《软件设计的哲学》绝对是配得上最好的技术书籍之一的赞誉。

但是不同的人读起来可能会有不同的感觉，其中的许多原则真的是做过设计，踩过坑才会有所共鸣, 否则会觉得其泛泛其谈。

当然，我也不是完全同意书中的所有观点的。

比如书中提到的会导致代码意图不「明显」的其中一种做法是声明的类型与初始化的类型不一致的情况:

```java
private List<Message> incomingMessageList;

...

incomingMessageList = new ArrayList<Message>();
```

上面声明的是 `List<Message>`, 实际使用的 `ArrayList<Message>`, 这可能会误导用户，因为意图不清晰，阅读代码的人可能不确定是否需要使用 `List` 或者 `ArrayList`, 最好是声明和初始化都换成相同的类型。

但是 `List` 是接口, `ArrayList` 是接口的具体实现，这个就是非常标准的面向对象编程中的多态，这并不什么问题。

但瑕不掩瑜，全书读完，把书盖上后，我有种齿颊留香, 余音绕梁的感觉，书里有很多「熟悉的味道」，总是让我想起经手过的项目中种种的好代码和「坏」代码.


## 推荐阅读 {#推荐阅读}

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
    -   [一本读了八年还没读完的书](https://ramsayleung.github.io/zh/post/2025/structure_and_interpretation_of_computer_programs/)
    -   [测试技能进阶(一): 软件质量认知](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%80_%E8%BD%AF%E4%BB%B6%E8%B4%A8%E9%87%8F%E8%AE%A4%E7%9F%A5/)
    -   [测试技能进阶(二): Parameterized Tests](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/)
    -   [测试技能进阶(三): Property Based Testing](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%89_property_based_testing/)

<div class="qr-container" center>

<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" class="qr-container" width="160px" height="160px" center="t" />
公号同步更新，欢迎关注👻

</div>

[^fn:1]: <https://news.ycombinator.com/item?id=31713756>
[^fn:2]: <https://book.douban.com/subject/30329536/>
[^fn:3]: <https://book.douban.com/subject/30218046/>
[^fn:4]: <https://book.douban.com/subject/34986245/>
[^fn:5]: <https://news.ycombinator.com/item?id=43166362>
