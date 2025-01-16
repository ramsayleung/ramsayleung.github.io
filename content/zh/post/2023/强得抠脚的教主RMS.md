+++
title = "黑客列传：强得抠脚的教主RMS"
date = 2023-07-16T22:58:00-07:00
lastmod = 2025-01-09T19:21:34-08:00
tags = ["biography", "history"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

前段时间流行一种关于程序员效率的说法，叫「10x程序员」，即一个好的程序员的工作效率是普通程序员的10倍。 <br/>

但是，在编程界，有这么一群人，他们的工作效率，可以说是百倍，甚至千倍于普通程序员； <br/>

更令人叹服的是，他们创造了普通程序员即使百倍努力也无法写出的作品。 <br/>

使用「程序员」这个职业来称呼他们，未免流于平凡，无法展现出他们竖立起的丰碑；而使用「计算机科学家」，又未免过于学术，不接地气； <br/>

那么，就回到最初，用「黑客(hacker)」这个称谓来称呼他们吧。 <br/>


## <span class="section-num">2</span> 关于黑客 {#关于黑客}

可能大家对「黑客 (hacker)」的印象多来自于电影，比如《黑客帝国》，就是那种在电脑面前，使用各种看不懂的工具入侵别人电脑的人。 <br/>

但是这种看法大多是对于「黑客」的误解，称之为「骇客(cracker)」可能更加合适，即恶意入侵他人电脑的人。 <br/>

`hacker` 一词又是从 `hacking` 衍生而来的，将 `hacking` 翻译成成中文网络语中的「整，搞，开干」可能会更贴切，  而最初的「黑客」指的就是一群富有创造力和兴趣的爱好者，只是比较具有代表性的是在计算机领域。 <br/>

国外有个很有名的科技相关的聚合网站，叫做「Hacker News」, 其中的「Hacker」, 也是沿用黑客最初的含义。 <br/>

既然提到黑客，那么有一个无法绕过去的人物，那就是今天的主角，黑客文化的领军人物：Richard Stallman <br/>

{{< figure src="/ox-hugo/rms_avatar.jpg" >}} <br/>


## <span class="section-num">3</span> UNIX {#unix}


### <span class="section-num">3.1</span> 分时系统 {#分时系统}

相信今天的我们，对操作系统这个概念不会陌生，在电脑上有 Windows 10, Windows 11, Windows 7 或者苹果的 MacOS操作系统，在手机上有 Android 和 IOS操作系统。 <br/>

所谓的操作系统，即是一套管理硬件，发挥硬件性能的软件，避免应用程序直接和硬件打交道，省去普通程序员大量的开发成本和心智。 <br/>

与今天直接在手机操作系统上，一边聊微信，一边放音乐不同，远古时候(二十世纪六十年代)的操作系统只支持批处理模式： <br/>

即用户同时提交多个任务，任务1运行完才能运行任务2，相当于你只能把音乐听完，然后关掉音乐软件，然后才能打开微信，发送聊天消息。 <br/>
（请忽略远古时代还没有微信这个问题） <br/>

你可能会想，这也太挫了吧。 <br/>

没错，当时的计算机科学家也这么认为的。 <br/>

因此1964年，通用电气和麻省理工大学就打算合作开发一个多任务操作系统，支持多个用户，运行多个任务，名为 `MULTICS` <br/>

后来，AT&amp;T公司的贝尔实验室也加入到这个操作系统的研发中，但是项目目标过于庞大，特性太多，性能又很低, AT&amp;T见项目前景不妙，就把资源都撤了，退出了这个项目。 <br/>


### <span class="section-num">3.2</span> 玩游戏玩出来的UNIX {#玩游戏玩出来的unix}

贝尔实验室的一位工程师，名叫Ken Thompson, 刚加入 MULTICS 项目不久，公司就准备退出了，但是通用公司为了项目而准备的机器 GE-645 就还保留在贝尔实验室，Ken 就打算用这些机器写个太空旅行的游戏。 <br/>

{{< figure src="/ox-hugo/ge-645.jpg" >}} <br/>

然而，Ken 写出来的游戏跑得很慢，每次运行还要75美刀，更难受的是，GE-645 这批机器，不久后就被搬回去通用公司了。 <br/>

所以Ken 只好在实验室角落找了几台没人用的PDP-7, 在同事 Dennis Ritchie 的帮助下，再重写了一次游戏。 <br/>

{{< figure src="/ox-hugo/pdp-7.jpeg" >}}  <br/>

这次的游戏开发经历，加上之前的 MULTICS 项目经验，让Ken 开始研究如何使用 PDP-7 开发一个分时多任务操作系统。 <br/>

然后他花费了一年的时间，和 Dennis 一起，在PDP-7上开发了一个分时多任务系统，名为UnICS，这就是第一版的 UNIX。 <br/>

因为PDP-7的性能不佳，最多支持两个用户, Ken 和 Dennis 又把第一版的 UNIX迁移到 PDP-11上，为了方便迁移，还顺便发明了一门编程语言，名为 C语言，并将UnICS 改名为 UNIX. <br/>

(这两位也是神) <br/>

影响后世无数操作系统的 UNIX 操作系统就此诞生，并迅速风靡各大研究机构，政府机关，企业与大学，成为70-80年代，操作系统事实上的标准 <br/>


### <span class="section-num">3.3</span> 商业版本与闭源 {#商业版本与闭源}

原来的软件只是买硬件时的赠品，到七十年代未，人们开始发现，原来软件也可以卖钱，很快，制作与销售商业软件成为一门热门生意。 <br/>

最开始的UNIX 版本是开放源代码供使用者的，也就是使用者不但可以安装 UNIX 系统，还可以阅读，并修改UNIX 系统的源代码。 <br/>

但是贝尔实验室的母公司 AT&amp;T毕竟是商业公司，把自己的源代码授权出去，后面还怎么赚钱呢？ <br/>

所以在20世纪80年代相继发布的UNIX 商业版本，只发行二进制，不再包含源代码。 <br/>

对于黑客来说，就是你能看到这个操作系统是怎么跑的，但是你再也无法知道他是怎么实现的了。 <br/>


## <span class="section-num">4</span> RMS {#rms}

Richard Matthew Stallman, 1953年出生于纽约的一个犹太家庭, 1974年毕业于哈佛大学，1975年在 MIT 攻读博士，后来退学在 MIT AI 实验室写代码。 <br/>

他的名字首字母为 RMS, 早期在黑客社区混的时候，以 RMS为用户名，所以大家都叫他 RMS(后面就以RMS来称呼他了). <br/>

当时的「黑客文化」崇尚开放，分享与交流，认为分享才能促进社会进步，在这样的文化熏陶下，RMS 自然对闭源软件痛恨不已。 <br/>

1980年，还在 MIT AI 实验室工作的时候，因为激光打印机和大部分工作人员都不在同一层楼，总是跑上跑下去查看打印结果和进度就很麻烦。 <br/>

RMS 就给实验室的激光打印机写了一个程序： <br/>

可以在打印任务完成时，发消息通知用户；或者当打印任务卡住的时候，也发消息通知用户； <br/>

然而，因为最新版本的打印机源码不再开放，RMS写的程序就无法再适配，让他相当恼火。 <br/>

以小见大，整个软件行业都在发生变化，甚至连UNIX 这样的基石软件都开始不再开放源代码授权，RMS感觉，他要站出来做些什么了。 <br/>


## <span class="section-num">5</span> GNU {#gnu}


### <span class="section-num">5.1</span> 荜路蓝缕 {#荜路蓝缕}

在1983年, RMS 宣布了GNU 操作系统计划，计划开发出一个兼容 Unix的源码开放的操作系统，让 Unix用户可以无缝切换到 GNU 操作系统上. <br/>

GNU 就是 "GNU is Not Unix"的缩写(那开头的GNU又是什么意思呢? 按照程序员的行话来说，这个叫递归) <br/>

经过十多年的发展，Unix 已经成为操作系统事实上的标准，重新开发一个新的操作系统几近天方夜谭。 <br/>

想象一下，有人跟你说要开发一个 Android 操作系统，用来替换掉 Google 的Android 系统，这工作量和难度可想而知，这就是现实中的想要移山的愚公，大战风车的堂吉诃德。 <br/>

但是 RMS 并未被眼前的困难所吓退，而是一步一步，从0开始构建他心中的类Unix操作系统. <br/>

1984年, RMS 开发并发布GNU Emacs 这个著名的文本编辑器, 方便程序员进行代码开发; <br/>

{{< figure src="/ox-hugo/gnu_emacs.png" >}} <br/>

1986年, RMS 开发并发布GNU Debugger(gdb) 调试器, 方便程序员来调试程序; Emacs + gdb 就是他那个时代的IDE <br/>

{{< figure src="/ox-hugo/gdb_screenshot.png" >}} <br/>

1987年, RMS 开发并发布GNU Compiler Collection(gcc) 编译器套件; 所谓的编译器，即将人写的代码，转换成机器可以运行的二进制代码。 <br/>

{{< figure src="/ox-hugo/gcc_logo.svg" >}} <br/>

开发出一个这样的软件就足以在计算机史上留名，RMS 在这3年间，还一口气开发出了3个，这样的技术水平和生产效率，只能让人叹服，影响力堪比盗火的普罗米修斯。 <br/>

何况这些软件至今仍在迭代，被无数程序员所依赖，所使用。 <br/>

比如微信的所有后台代码，都是使用GCC 编译出来的，也就是你现在也在间接使用着 RMS 当初编写的软件。 <br/>

近40年过去了，市面上被广泛使用的C/C++编译器就只有三个: 微软家的 MSVC, 苹果支持开发的 Clang, 还有GNU 项目的 GCC. <br/>

除此之外，GNU 项目还开发了许多的基础设施，如GNU make, GNU grep, bash，以及志在替换掉 PS的 GIMP 等等. <br/>

除了基础设施外，GNU项目还希望类似通过美国宪法保证言论自由一样，通过法律和版权，确保软件开放源代码。 <br/>

因此, 在1989年, RMS 发布了 GNU General Public License(GPL)授权, 主要内容是: 用户可以自由使用，复制，修改GPL软件, 派生的软件也必须使用GPL, 不能转换成闭源软件. <br/>

从法律层面保证了GPL软件不会被有心人直接拿去闭源赚钱。 <br/>

此外, RMS 还将自由软件发展成社会运动，将软件开发这个程序员小圈子的活动，成为扩展到整个社会的思想运动。 <br/>

在人类没有进化到「无私」的精神境界前（可能永远都达不到），通过GPL法律条款来保证「自由」的权利， <br/>
不得不说是一种创举，从而让世界上每个人都有机会享受到软件发展所带来的好处。 <br/>


### <span class="section-num">5.2</span> 开花结果 {#开花结果}

时间来到90年代, 经过近10年的耕耘, 在基础组件和配套设施相继完善之后，GNU 项目终于来到最关键的节点，开发出可以替换Unix 系统的内核(kernel). <br/>

如果电脑硬件来比喻操作系统的话，就是内存，硬盘，主板，显示器，电源全部都就绪，就差最后的CPU, 画龙最后的点睛. <br/>

但是GNU 的内核 Hurd 却迟迟未能发布, 而天下可谓苦闭源 Unix 久矣。 <br/>

在1991年, 一个叫Linus的芬兰学生在社区上发布了他自己的业余项目：一个类Unix 的操作系统内核。 <br/>

他把GNU 项目的相关组件(bash和gcc)移植到这个系统，也能正常运行起来了, 这个系统就是Linux(完整的名称应该是 GNU/Linux) <br/>

自此, GNU 项目的最后一块拼图完整了, 十年磨一剑, GNU的基础组件加 Linus 的Linux内核, 一个志在替换 Unix 的操作系统终于完成了, 这就是 GNU/Linux. <br/>

苦Unix久矣的社区的开发者云集而来，为 GNU/Linux 添砖加瓦, 让GNU/Linux 成为今天的参天大树(连微软家的服务器也在运行 Linux) <br/>

--- <br/>

只见新人笑，哪闻旧人哭. <br/>

有点离谱的是, GNU Hurd 已经开发超过30年了，还没有发布1.0(稳定可用版本). <br/>

更离谱的是，最近还有更新: <br/>

2023年6月份，还发布了2023年 [版本更新](https://lists.gnu.org/archive/html/bug-hurd/2023-06/msg00038.html): <br/>

{{< figure src="/ox-hugo/gnu_hurd_2023_release.png" >}} <br/>


## <span class="section-num">6</span> 轶事 {#轶事}


### <span class="section-num">6.1</span> 教主 {#教主}

为什么称RMS 为教主呢？ <br/>

因为RMS 创建了 Emacs 这个神的编辑器，自其诞生以来，与编辑器之神 Vi/Vim 的圣战就从未停息。 <br/>

使用Emacs 的程序员与使用Vi/Vim 的程序员，一直在争论，究竟哪个才是更好的编辑器？ <br/>

既然 RMS 是Emacs 的创始人，自然被使用 Emacs的人尊称为「教主」。 <br/>

而这场争论已经持续近四十年，依旧没有分出胜负。 <br/>

像 Google 这样浓眉大眼的家伙，还在不时地给这场战争拱火: <br/>

{{< figure src="/ox-hugo/google_emacs.png" >}}  <br/>

{{< figure src="/ox-hugo/google_vi.png" >}} <br/>


### <span class="section-num">6.2</span> 教主与教主 {#教主与教主}

乔布斯被「果迷」尊称为「教主」，大家可能不知道的是，这两位「教主」曾经有过一场交锋。 <br/>

1993年, 当时乔布斯还在 NeXT公司, 买下了 `Objective-C` 语言来开发应用程序(后来的IOS用的也是 `Objective-C`), 使用的编译器也是 GCC. <br/>

NeXT 修改了 GCC的源码，以便增加对 `Objective-C` 的支持，而GCC 使用的又是GPL 授权，而根据GPL 的授权，任何对GPL软件的修改，也必须要开放源代码。 <br/>

所以乔布斯就问RMS, 他能否把 GCC 拆分成两部分，一部分是原来GCC, 继续开放源代码；另外一部分是增加 `Objective-C` 的GCC 编译器前端，闭源收费商用。 <br/>

RMS 回复，当然是不可以。我估计老爷子心想，防的就是你这种人。 <br/>

乔布斯只好将 `Objective-C` 编译器的前端也以GPL 授权开放出源代码。 <br/>

--- <br/>

若干年后，苹果计划开发自己的编译器，因为设计以及授权的原因，在谋求与 GCC的合作未果后，转而支持 LLVM 的clang, 那也是后话了. <br/>


### <span class="section-num">6.3</span> 中国芯 {#中国芯}

根据 RMS [自述](https://usesthis.com/interviews/richard.stallman/), 他之前用的一直是中国科学院设计的龙芯处理器的龙梦电脑, 虽然这台电脑的性能，显示尺寸(只有9英寸)都无法让RMS 满意，但是这台电脑的是完全自由的，包括硬件, bios, 软件: <br/>

> What hardware do you use? <br/>
> 
> I am using a Lemote Yeelong, a netbook with a Loongson chip and a 9-inch display. This is my only computer, and I use it all the time. I chose it because I can run it with 100% free software even at the BIOS level. <br/>

在性能和自由之间，他一如既往地选择了「自由」 <br/>

根据RMS [官网的描述](https://stallman.org/intel.html), 他不用intel 或者 amd 的芯片，是因为他们都有后门: <br/>

> Reasons not to use Intel <br/>
> 
> Don't use Intel processors newer than Core2, because they have the "management engine" back door. <br/>
> 
> Recent AMD processors have a similar problem, but we do not yet have an article about it. <br/>

不过，据闻他的龙梦电脑被偷了之后，他也就换到 [ThinkPad 上了](https://stallman.org/stallman-computing.html): <br/>

> As of 2022 I use a Thinkpad x200 computer, which has a free initialization program (Libreboot) and a free operating system (Trisquel GNU/Linux). <br/>


### <span class="section-num">6.4</span> 抠脚 {#抠脚}

菜的抠脚就听说过，强得抠脚又是什么呢？ <br/>

因为他真的抠脚（字面意思），还吃回去了。 <br/>

{{< figure src="/ox-hugo/koujiao_1.png" >}} <br/>

{{< figure src="/ox-hugo/koujiao2.png" >}} <br/>


## <span class="section-num">7</span> 总结 {#总结}

他是天才黑客，是自由软件的精神领袖，是知行合一的孤勇者，更是个凡人堆里的理想主义者. <br/>

当然，还是我大 Emacs 神教的教主. <br/>


## <span class="section-num">8</span> 参考 {#参考}

-   <https://en.wikipedia.org/wiki/GNU_Compiler_Collection> <br/>
-   <https://en.wikipedia.org/wiki/Richard_Stallman> <br/>
-   <https://en.wikipedia.org/wiki/GNU_Emacs> <br/>
-   <https://en.wikipedia.org/wiki/GNU_Debugger> <br/>
-   <https://stallman.org/> <br/>
-   <https://usesthis.com/interviews/richard.stallman/> <br/>


<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

