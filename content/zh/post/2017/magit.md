+++
title = "(翻译)An Introduction to Magit"
description = "An introduction to Magit"
date = 2017-03-03T00:00:00-08:00
lastmod = 2025-01-09T19:20:08-08:00
tags = ["emacs"]
categories = ["翻译"]
draft = false
toc = true
showQuote = true
+++

如果你足够幸运(或者不幸运，取决于你怎么看待了)可以使用 `git` 作为你工作流的一部分。

你可能已经 **邂逅** 过 `magit` 这个Emacs 的git接口了。 `magit` 是Emacs 上的非常优秀的git 接口，它假定你是了解你正在对 `magit` 或者 `git` 做的种种操作的 **注意** ：该文章是针对 magit 1.x 的，对 magit 2.x 并不适用 Magit 有非常完整的文档，包含了Magit 的各种操作，但是和大多数的文档一样，Magit 的文档并没有介绍如何将Magit 和你的工作流结合；

Magit 假定你是熟悉Magit 并且了解如何合理地使用Magit(但大多数情况并不是这样).

Magit 现在还是处于活跃的开发中。在2013年12月，增加了很多很多新的很有用特性的一次 release, 让Magit 变得比以前更强大了，所以本教程是基于比较新的Magit 版本的，并且 假定你也已经安装了新的版本

如果想安装处于 `master` 分支的Magit,我建议你使用 `Melpa` 来安装；此外你也可以选择直接拉取[magit github](https://github.com/magit/magit) 仓库的最新版本，然后按照README 上面的指导来构建 `magit`

这是我Magit 教程的第一部分。这部分会介绍状态窗口(status window); 已暂存和未暂存的项目 (staging and unstagin item);已经提交的更改 (committing changes) 和查看历史提交 (history view)


## <span class="section-num">1</span> Getting Started {#getting-started}

首先：Magit 并没有隐藏git 的复杂性，所以，如果你想高效地使用Magit ,你最好清楚了解git 究竟做了哪些工作。

事实上，我更原意把Magit 当作一个取代了git 枯燥的纯命令行操作的工具，它也表现得出乎意料地好。

你可以通过 `M-x magit-status` 来使用Magit,该命令会打开一个窗口(如果你的缓冲区 所在的目录不是一个git 项目，Magit 会提示你进入一个git项目的目录),然后展示Magit 当前的状态。

你是通过 `magit-status` 这个接口来使用Magit 的。此外，如果你是使用 Emacs VC的，你需要知道的是，Magit是没有集成到Emacs VC(Version Control)的 抽象层。

虽然你没法在VC 使用Magit,不过你还是可以在大多数版本控制工具使用Magit 的，Magit 为这些版本控制工具都提供了统一的接口；你如果想调用Magit,你只需要 `M-x magit-status`


## <span class="section-num">2</span> The Magit Status {#the-magit-status}

{{< figure src="https://www.masteringemacs.org/static/uploads/Screenshot-from-2013-12-06-114511.png" >}}

你首先会注意到关于Magit的事情应该就是当你打开Magit 的状态窗口时，Magit 的状态窗口
是可以与你的Emacs 窗口配置配合工作的，你也可以像在其他Emacs 窗口那样，通过按下 `q`
来关闭窗口。

几乎你在Magit 执行的所有操作都是通过在底部打开一个 `command console` 窗口，然后按下对应的单字符指令执行；你也可以重新定义你自己的指令。

这种交互的方式真的非常好用，可能正是这么强大的特性让Magit变得如此优秀吧。

我个人真的非常喜欢这种交互的方式，我甚至把这部分特性的代码复制到了我自己的Emacs项目上，因为这真的真的非常好用。

Magit 之前的稳定版本在帮助用户更好使用Magit这方面做得略有不足，所以在最新的版本有了相应的改进，你可以通过按下 ~?&lt;/kbd&gt; 来显示一系列带注解说明的操作。我觉得在最开始的时候，Magit 真的很难用，因为我总是在「茫茫」的菜单选项中迷失，好不容易才能找到我想要的操作。

即使是现在，也并不是所有的操作都有注解说明了；有一些命令
(对于我的工作流来说很重要的命令)依然是没有说明的，特别是用来重新定位的 `E`.


## <span class="section-num">3</span> Staging and Unstaging Items {#staging-and-unstaging-items}

把你的文件放到git下面是你经常需要完成工作之一，Magit 有一系列的按键绑定和工具
可以帮助你更好地完成工作。

Magit 操作不仅可以暂存文件，还可以暂存在 **diff** 中选定 的代码块。

Magit"杀手级"特性之一就是它使用不同的等级来显示相关的信息。Magit 可以
让你通过 `tab` 展开或者折叠已经暂存或者未暂存文件。

如果你想更加细颗粒度地暂存或者未暂存文件，你可以使用 **M-1** 到 **M-4** 来操作所有的文件；此外，也可以使用 **1** 到 **4**
操作选定的文件

{{< figure src="https://www.masteringemacs.org/static/uploads/diff-hunk-refined-level-4.png" >}}

1.  等级1会把所有的东西隐藏到一个分类里面(即已暂存的文件);
2.  等级 2在一个分类里面只是显示文件名 (这是默认的等级);
3.  等级3会显示git代码块的头部；等级4 会显示所有做出了修改的代码块。

    我使用最多的是等级2和等级4, 如果你使用按键 `TAB`,Magit会完成你想要的等级操作的。你拥有一系列可以让你的

生活变得更加美好的按键绑定，例如： `n` 和 `p` 可以在你前一个单元和后一个单元(通常以代码块为单元)
之间移动；~M-n~ 和 `M-p` 可以在相邻单元之间移动，例如在等级4中的每个文件间移动。

你也可以使用 `+` 或者 `-` 放大或者缩小每段代码，使用 `0` 可以恢复默认设置。此外你也可以按下 `H` 给代码块 添加额外的代码高亮。

最后，你按下回车 `RET` 可以打开你修改的文件，代码块或者文件都适用该操作。

---

你可以通过按下 `s` 或者 `u` 来暂存或者撤销暂存文件(或者代码块),此外，奉送一个很有用的小提示：
如果你选定某部分的代码，然后按下暂存(撤销暂存)按键，Magit 会自动暂存(撤销)你选定的那部分代码。

当你发现 `diff` 选定的代码块不符合你的要求的时候，你会发现这种细颗粒度的操作真的非常有用

---

有时候你对某些修改并不在意，你也不关心这部分修改是否已经提交；你可以像上面的暂存(撤销暂存)操作一样，通过按下按键 `K` 来忽略选定的代码块和文件，并且从你的电脑删除未提交到暂存区(untracked)的文件；

这个命令可以比暂存(撤销暂存)命令完成更多的操作，例如，删除已保存的文件或目录(stash)


## <span class="section-num">4</span> Committing Changes {#committing-changes}

如果你想打开提交菜单，只需按下 `c`,然后你就会看到琳罗满目的选项，不过大部份选项 你都是用不上的了。你真正有用的操作，不仅可以让你提交已暂存的修改，还可以完成更多的任务：

-   你可以扩展(extend `e`) `HEAD` 所指向的提交
-   你可以修改(amend `a`) 有关的提交信息
-   如果你不喜欢现在的提交信息，你可以重写(reword `r`)提交信息
-   你同时也可以修整(fixup `f`)和压缩(squash `s`)当前这次的提交。如果你之前用 `.` 标记了
    一次提交，那么今次使用的就是被标记的那次提交。

扩展一次提交其实就是在当前提交上附加修改，所以，如果你忘记了提交本属于此次提交的东西
你可以使用 `扩展` 选项。

如果你想修改当前的提交信息，那就使用 `修改` 选项吧

重写可以重写你的提交信息但是无需提交你已暂存的修改；如果你不小心按错了选项，想重写你的提交信息，=重写= 选项就是你最好的选择

如果你想在最新一次提交下创建一个 `fixup` 或者 `squash` 提交的话，使用修整或压缩命令 可以重整或者 `--(自动压缩)autosquash` 最新一次提交。

如果你不会去重写你的git历史或者你未使用过重整，你可能觉得这两个命令不是很有用


## <span class="section-num">5</span> Logging {#logging}

{{< figure src="https://www.masteringemacs.org/static/uploads/Screenshot-from-2013-12-06-142317.png" >}}

我觉得Magit非常强大的特性之一就是它有不计其数的选项可以用来对你的git 历史进行 过滤，排序，查找。

Magit 不仅可以展示你的git信息，还可以让你执行交互操作。如果你想打开日志的菜单，你只需按下 `l`.

你应该知道的第一个有用的按键就是 `l l`, 这个 按键会为你展示缩略的日志信息：你会看到单行的提交信息, 作者的名字, 修改提交距今的时间, 树状结构的git 日志, 各种的标签信息，例如 `HEAD` 指针的位置或者分支标记的位置

如果你不小心玩坏了git 的提交信息，命令 `git reflog` 会是你的救星；此外，对于magit 的引用日志(reflog)机制(`l h`)，它也有很友好稳定的UI界面支持。

---

引用日志和普通的日志都有非常丰富的按键绑定。在日志里，你对单个的提交可以进行
非常多的操作：

-   `.`: 为此次提交作标记以便进行后续的操作例如提交修整 (`c f`)或者提交压缩 (`c s`)
-   `x`: 重置你的 `HEAD` 指针到选定的提交
-   `v`: 撤销提交
-   `d`: 将你的工作区与选定的提交进行比较
-   `a`: 将选定的提交作用在你的工作区
-   `A`: 选择位于你工作区顶部的提交
-   `E`: 以交互的方式重置你的 `HEAD` 指针到选定的工作区。如果你想重写历史，该命令会非常有用
-   `C-w`: 复制你此次提交的hash值
-   `SPC`: 展示完整的提交历史

需要注意的是：即使你关闭了日志的窗口，标记的命令还是会继续作用的；标记是非常有用的工具，但是
你很容易忘记你是否曾经作过标记。如果你在magit 使用 `M-n` 或者 `N-p` 向上或者向下浏览日志，
maigt 会自动为你在另外一个窗口显示提交信息


## <span class="section-num">6</span> Conclusion {#conclusion}

对于有经验的Git 用户来说，Magit 是一个非常好的工具；此外，如果你是Git 的新手， Magit可以帮助你了解Git 是怎么工作的，但是它永远不会教你使用Git.

在我看来，阻碍 你使用Magit 最大的障碍就是Magit缺乏对选项的描述说明；即使Magit 包含了成千上万 Git的选项，参数和操作，但是它并没有教你如何找到并使用这些命令。

我发现Git 的命令行真的无可替代(不是因为我喜欢git 的命令行我才这么说，事实是它真的很棒)因为我 想要完成的操作真的隐藏得很深，没有那么容易在Magit找到。

不过最新版本的改进真的很好，你可以通过按下 `?` 查看一系列带有注解说明的命令(但不是全部命令，不过这也
已经是一个很大的改进了).

如果你曾被Magit 的学习曲线所吓倒，抑或者你已经尝试Magit, 却无奈放弃；我建议你再试一次。我打算写更多关于Magit 的博文

---

原文地址 <https://www.masteringemacs.org/article/introduction-magit-emacs-mode-git>
在下翻译水平有限，如有错误，还请指出

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

