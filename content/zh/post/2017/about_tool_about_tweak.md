+++
title = "关于工具,关于折腾"
description = "An discussion about tool and tweak"
date = 2017-03-24T00:00:00-07:00
lastmod = 2024-12-30T22:36:15-08:00
tags = ["linux", "tool", "tweak"]
categories = ["linux", "tool"]
draft = false
+++

我最近一直在思考，关于工具，关于折腾，关于其中的付出与收获


## <span class="section-num">1</span> 乐趣 {#乐趣}


### <span class="section-num">1.1</span> Linux {#linux}

回顾我的大学，从大一开始就是一个不停折腾的过程，在其他的同学还在用Windows玩游戏的时候，我已经把系统换成Linux了.

记得最开始装的第一个发行版本是 [Kali Linux](https://www.kali.org/)一个黑客和安全专家使用的发行版本，上面有不计其数的渗透工具；毕竟每一个学 计算机的孩子心中都是有个 hacker dream的嘛，我也不例外:)。

只是我最开始并没有能力去使用Kali Linux; 甚至连基本的命令都完全不了解；我相当沮丧，因为 hacker并不是想象中的那么容易的. 我后来就把自己的系统重装，装了个[ Ubuntu](https://www.ubuntu.com/global), 买 了一本《鸟哥的Linux私房菜》，一边学，一边用，就这样进了Linux的坑了。

《鸟哥的私房菜》大概看了两年，翻过好几次了，后来也看了《服务器篇》，前后共看了近十本Linux的书籍吧，整个大学大概在自己电脑上前后装了10种的发行版本吧


### <span class="section-num">1.2</span> Vim/Emacs {#vim-emacs}

《鸟哥Linux私房菜》一书中，鸟哥强推Vim, 其他的Linux论坛也对 Vim 推崇备至，我 很自然就随大流去学习Vim了，开始的时候，真的非常不习惯，编辑个文本还要分那么多 的模式，真的是反人类，连个单词都不能输入.

后来，好不容易输完数据之后，又不知道怎么保存 (Ctrl-S? 想多了), 然后直接关闭，重新打开又有什么提示说是否恢复数据。 觉得为何有这样异类难用的编辑器, 真不知道为什么那么多人推崇。

但当我坚持这种煎熬半个月以后，就发现其他的编辑器都非常低效，没错，就是非常低效，又要鼠标， 又要键盘，不断地切换，效率实在太低了。

就这样，我糊里糊涂就进入了Vim的阵营，直到遇到 [Vim实用技巧 ](https://book.douban.com/subject/25869486/)这本神书，它跟你讲述了如何实现 Vim "Edit Text at the speed of thought" 的理念，的确是神书. 自然，我对Vim就更 "坚贞不渝" 了；

---

直到有一天，在浏览Linux/Unix历史的时候掀开了 Editor War(Vim与Emacs之战)一章， 那些 Emacser 竟敢宣称 Emacs 比 Vim 好用，我对此并不服气，不相信有比Vim强的编辑器，这可是编辑器之神阿，而我是一个很实在的人，没用过 Emacs 是不会随便发
言的，所以就跑去折腾Emacs ,打算折腾回来再跟 Emacser 论道，结果嘛，我就 "叛
逃" 到了 Emacs 了 :)。

作为一个曾经的 Vim 粉丝，我就抛开 "宗教因素" 比较一下 Vim 跟 Emacs:

-   Vim的 modal edit 是最好的，真的难有敌手，所以这也是为什么在各种的 IDE/editor 都有 Vim 插件的原因；
-   但 Emacs 的扩展性也是无可匹敌 (毕竟是伪 装成编辑器的操作系统，只缺一个好用的编辑器)，又因为 Emacs lisp这种真正的编程 语言(对比之下 viml真的很弱)的存在, Emacs 就有了无限可能，这也是 Emacs 上面有非常多高质量的插件的原因之一，其中最典型的例子就是 Org-mode ,无愧神器之名，我现在的博文也是在 Emacs 里面利用 Org-mode 编写，然后发布的。

而至于选择神之编辑器还是编辑器之神，那就是信仰的抉择了。我选择了在 Emacs 里面使用 Vim 的编
辑模式 Evil :)


### <span class="section-num">1.3</span> Misc {#misc}

除了折腾编辑器之外，我还折腾了各种的命令行，Shell 脚本，还有 Firefox, Chrome浏览器。当初那些 Windows 用户一直说 Linux 的桌面丑，我就去了折腾各种 的桌面环境 (window manager)这种折腾可不是 Windows 上面的切换壁纸哦，后来把桌 面折腾得非常炫，以至同学看到我的电脑就说我装了黑苹果，然而事实并非如此。

如果你也好奇那些炫酷的 Linux/Unix 桌面，可以查看 <https://reddit.com/r/unixporn> 上面有各种 Linuxer/Unixer 分享的炫酷桌面


## <span class="section-num">2</span> 投入产出比 {#投入产出比}


### <span class="section-num">2.1</span> 值得否？ {#值得否}

我的大学基本都是在学习并折腾各种的工具或者技术，并且乐在其中.

但是有一天当笔 者又在跟朋友推荐 Vim/Emacs, 或许是我喋喋不休实在太多次了，朋友回了我一句 " notepad++, sublime text 不一样可以写代码，你为什么还要花那么多时间去折腾这些东西呢，你写脚本都可以直接用IDE,为什么还要自己折腾呢，把时间花到其他地方不更好么？".

我难以反驳，我之前一直是玩得很开心，从未曾考虑过这个问题，所以那个时候开始询问自己，这是否值得，自己是否要把时间用到其他地方？

在之后的一段 时间，我都难掩沮丧，因为觉得自己浪费了很多的时间来完成一些无用功！


### <span class="section-num">2.2</span> 长期投资 {#长期投资}

但是最终我还是解答了自己的疑问! 我之前付出是绝对值得的，先不说我在其中获得的乐趣，乐趣是无价的嘛 :)

我在折腾的过程中也学到很多新的东西: 为了用好我配置的 Emacs, 我使用 Emacs 写了很多不同的脚本，这种感觉就好像，侠士为了展示手中利刃之威力，苦练武艺; 而在折腾 Emacs lisp 的过程中，也学习很多函数式编程的思想，甚至掌握了一门新的语言 -- elisp, 虽说它的语法很奇怪。

其实我的付出是长期投资，学会了 Vim 的 moral edit, 也可以在其他 IDE使用嘛，这并不矛盾的，无鼠标操作是非常高效的，也是所谓的 modern editor 无法比拟的。

最重要的是，在折腾过程中所培养的解决困难的动手能力，也是可以受益终生的，我知道如何去google，如何去查找文档，如何去提问; 而且在不停的折腾过程中，你对某样技术的理解是单纯的理论学习无法比拟的；

在大学的操作系统课，我基本是没听老师讲解课程的，因为老师讲的，我基本都知道，甚至实践过。


## <span class="section-num">3</span> 工具集 {#工具集}

在经历大学的折腾后，我现在很多的工具集都基本确定下来了；这些也是对我而言，
最高效的工具集


### <span class="section-num">3.1</span> 编辑器 {#编辑器}

-   Emacs 神之编辑器，主力编辑器 [个人配置](https://github.com/ramsayleung/emacs.d)
-   Vim 编辑器之神，一般在服务器改改配置的时候用


### <span class="section-num">3.2</span> 浏览器 {#浏览器}

-   Chrome 不常用，特定情况下使用
-   Firefox 日常浏览器，我也折腾过非常久，所以即使 Chrome 很强，我只为
    Firefox 倾心


### <span class="section-num">3.3</span> FireFox扩展 {#firefox扩展}

因为 FireFox 对插件的限制相对宽松，所以社区开发出了非常多非常强的插件，我就
列举一下自己使用的扩展集吧

-   bitwarden -免费的密码管理器，比LastPass强
-   Bluhell Firewall-轻量级的广告拦截器，和隐私保护
-   Clear Cache -更方便清除缓存
-   FalshGot -下载扩展器，配合axel或者aria2使用更佳
-   FoxyProxy -类似Chrome SwitchOmega,但是略有不如，配合Shadowsocks翻墙，必备
-   Ghostery -隐私保护
-   Greasemonkey -用户自定义插件管理器，神器
-   HttpRequester -类似Chrome Postman,发送Http请求
-   HTTPtoHTTPS -尽可能使用Https,提高安全性
-   KeySnail -把Firefox快捷键设置为Emacs快捷键，无鼠标操作，你也可以为该插件编
    写插件.神器，这个是我无法切换回Chrome的原因
-   Octotree -以树状目录来浏览Github代码，非常方便
-   uBlock Origin -广告blocker,低资源要求，感觉比Adblock plus好用
-   User Agent Switcher -切换User Agent,写爬虫时非常有用
-   Xpath checker -直接获取Dom节点的Xpath,配合Lxml解析网页非常高效
-   Firebug -神器，但是已经停止开发了。


### <span class="section-num">3.4</span> 桌面 {#桌面}

i3wm, 在折腾过炫酷的 KDE, Gnome, xfce, 而我最后选择的是 i3这个平铺桌面，可
以实现无鼠标操作，非常轻量。


### <span class="section-num">3.5</span> 命令行 {#命令行}


#### <span class="section-num">3.5.1</span> Shell {#shell}

-   zsh -配合oh-my-zsh,可以非常高效，但是使用频率不高
-   Eshell -与Emacs集成，是我的主力Shell,不过某些Eshell不支持的操作，只好在
    zsh完成


#### <span class="section-num">3.5.2</span> 过滤器 {#过滤器}

-   [ag](https://github.com/ggreer/the_silver_searcher)   grep的加强版，速度快
-   [ripgrep ](https://github.com/BurntSushi/ripgrep)最快的命令搜索工具
-   [percol](https://github.com/mooz/percol) 过滤文本，神器
-   [fasd](https://github.com/clvv/fasd) 目录跳转，文件查找，高效


#### <span class="section-num">3.5.3</span> misc {#misc}

-   [httpie](https://github.com/jakubroztocil/httpie) http客户端，发送http请求
-   htop top的改进版，信息更详细
-   [glances](https://github.com/nicolargo/glances) 一个好用的系统监控工具
-   ncdu Linux最好用的磁盘分析工具
-   git Linus又一神作

其它就是常用的内置命令了


### <span class="section-num">3.6</span> 影音 {#影音}

-   VLC Linux最好用的播放器
-   网易云音乐 国产良心音乐软件
-   [musicbox](https://github.com/darknessomi/musicbox) 网易云音乐的社区命令行版本


### <span class="section-num">3.7</span> 其它 {#其它}

-   Fcitx -中文输入
-   VirtualBox -开源虚拟机
-   Shadowsocks 翻墙必备
-   Zeal 类似Mac 上的Dash,查看各种文档
-   Intellij Idea Java IDE(写Java 我是不会使用Emacs 的:) )
-   Datagrip SQL IDE

使用最频繁的就是 I3+Firefox+Emacs,实现无鼠标操作，因为使用鼠标太慢了，效率太
低。我也不是一个疯子，所以只会用Emacs 做力所能及的事情，煮咖啡就算了。


## <span class="section-num">4</span> 结语 {#结语}

如果让我的大学重来一遍，估计我还是会这样折腾，因为自己动手的感觉还是很美好，充满成就感，这也是玩游戏所不能给予我的感觉，毕竟 **hacker** 不是想出来的嘛，是做出来的。

---

更新 2017-4-21

附上一篇关于折腾的文章 (需翻墙) [The importance of ZheTeng](https://program-think.blogspot.com/2017/04/The-Importance-of-Zheteng.html)

-   Enjoy tweaking;Enjoy Linux :)
