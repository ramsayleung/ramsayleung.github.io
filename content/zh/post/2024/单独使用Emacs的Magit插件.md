+++
title = "单独使用Emacs的Magit插件"
date = 2024-12-11T16:00:00-08:00
lastmod = 2025-01-09T18:08:11-08:00
tags = ["emacs", "magit"]
categories = ["Emacs技巧"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Emacs 与 Magit {#emacs-与-magit}

不知不觉，我已经使用Emacs 快10年了，在我使用过的编辑器中，Emacs是扩展性最强的编辑器，毕竟Emacs是个披着编辑器外衣的Lisp虚拟机。

在Emacs无所不能的扩展性之下，诞生了非常多强大的插件，
也让Emacs有了「伪装成操作系统的编辑器」的美名，而Emacs公认的杀手锏插件有两个，一个是 [org-mode](https://orgmode.org/)，另一个是 [magit](https://magit.vc/). (我个人觉得还有个 [dired](https://www.gnu.org/software/emacs/manual/html_node/emacs/Dired.html))

Orgmode是类似Markdown，与Emacs深度绑定优化的标记语言，使用Emacs来编写org-mode 文档就有下笔有神，文思泉涌，如丝般顺滑(这篇文章也是用org-mode写的)。

因为org-mode 与Emacs 深度结合，自然无法脱离Emacs单独使用，而其他编辑器模仿org-mode 开发的插件，如 [vim-orgmode](https://github.com/jceb/vim-orgmode), [nvim-orgmode](https://github.com/nvim-orgmode/orgmode) 和 [vscode-orgmode](https://github.com/vscode-org-mode/vscode-org-mode), 难免只得其形，未得其神，还不如用Markdown.

而 Magit 是 Git的Emacs图形化客户端, 也是我用过的最好用的Git 客户端软件，既直观又易用(看看Emacs 道友们夸 Magit 的[帖子](https://emacs-china.org/t/magit/22521/5)):

{{< figure src="/ox-hugo/magit_dashboard.jpg" >}}

虽然我已经用了Emacs很多年，但是已经过了Live with Emacs的境界, 不会用Emacs处理所有事情, 比如用VSCode 写Rust, 用Intellij Idea写Java, 既然 Magit 那么好用，有没可能独立于Emacs使用呢?


## <span class="section-num">2</span> Emacs daemon {#emacs-daemon}

作为无所不能的「操作系统」, Emacs 作为[server](https://www.gnu.org/software/emacs/manual/html_node/emacs/Emacs-Server.html) 一直在后台运行，然后再使用 `emacsclient` 连接 server:

前文提到, Emacs 是批着编辑器外衣的Lisp VM, 而 Magit 本质也只是一个 lisp function, 只要在启动emacsclient的时候，再调用 magit的函数, 那么就可以启动 Magit:

```shell
alias magit="emacsclient -nw -eval '(magit-status)'"
```

{{< figure src="/ox-hugo/magit.gif" >}}

这样就可以在VSCode和Idea里面愉快地使用 magit了.


## <span class="section-num">3</span> 总结 {#总结}

在Emacs-China 论坛搜索Magit的时候, 发现了也有一个帖子讨论把 [magit当作的单独的工具](https://emacs-china.org/t/magit/25527/6), 没想到有个回复的思路和我一样，使用Emacs作为daemon 来启用.

只是没有想到他更evil, 在Nvim 里面使用Magit, 我也学习一下 Nvim+Emacs 的组合 :)


## <span class="section-num">4</span> 参考 {#参考}

-   <https://orgmode.org/>
-   <https://magit.vc/>
-   <https://emacs-china.org/t/magit/22521/5>
-   <https://emacs-china.org/t/magit/25527/6>
-   <https://www.gnu.org/software/emacs/manual/html_node/emacs/Emacs-Server.html>
-   <https://www.gnu.org/software/emacs/manual/html_node/emacs/Dired.html>

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

