+++
title = "Eshell实现fzf的历史命令搜索功能"
date = 2017-12-17T15:46:00-08:00
lastmod = 2025-01-09T18:07:24-08:00
tags = ["emacs", "eshell", "linux"]
categories = ["Emacs技巧"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 前言 {#前言}

目标: 在=Eshell=中像在bash/zsh中使用=fzf=那般搜索历史命令


## <span class="section-num">2</span> fzf {#fzf}

我的主力Shell 是Eshell, 但是平时我也会用Zsh, 而[fzf](https://github.com/junegunn/fzf) 是一个非常好用的命令行工具，用了=fzf=搜索历史命令:

{{< figure src="https://i.imgur.com/pPMxauw.gif" caption="<span class=\"figure-number\">Figure 1: </span>fzf" >}}


## <span class="section-num">3</span> Eshell {#eshell}

我日常的操作基本都是在 Eshell 上面进行的，不过 Eshell 是没办法直接像 Bash 那样调用 =fzf=来查找命令历史的，所以我希望把这个功能迁移到到Eshell 上面来。

我在 Emacs 使用的补全框架是 `Ivy/Counsel`,它有一个 `counsel-esh-history=的命令可以使用 =Ivy` 来搜索命令，但是没办法使用用户已经输入的内容来过滤命令，所以我就在自己折腾了一个

`counsel-esh-history` 命令。效果如下：

{{< figure src="https://i.imgur.com/3tvGDzW.gif" caption="<span class=\"figure-number\">Figure 2: </span>感觉很不错嘛 :)" >}}


## <span class="section-num">4</span> 源代码 {#源代码}

得益于 =Ivy=强大的内置函数, 功能实现起来相当便利，完整代码如下：

```lisp
(defun samray/esh-history ()
  "Interactive search eshell history."
  (interactive)
  (require 'em-hist)
  (save-excursion
    (let* ((start-pos (eshell-bol))
           (end-pos (point-at-eol))
           (input (buffer-substring-no-properties start-pos end-pos)))
      (let* ((command (ivy-read "Command: "
                                (delete-dups
                                 (when (> (ring-size eshell-history-ring) 0)
                                   (ring-elements eshell-history-ring)))
                                :preselect input
                                :action #'ivy-completion-in-region-action))
             (cursor-move (length command)))
        (kill-region (+ start-pos cursor-move) (+ end-pos cursor-move))
        )))
  ;; move cursor to eol
  (end-of-line)
  )
```

代码不是很复杂, 主要功能是获取用户输入的命令, 然后把所有的历史命令读取出来,最后使用=ivy-read=内置的=ivy-completion-in-region-action=功能, 用用户的输入的命令与历史命令进行匹配, 由用户选择最终的命令.

=ivy-read=是Emacs内置=completing-read=的函数的强化, 关于=ivy-read=具体用法可以参考文档[ivy-read](https://oremacs.com/swiper/#getting-started).


## <span class="section-num">5</span> 总结 {#总结}

最后, 我也顺便把代码分享到 [Emacs社区](https://www.reddit.com/r/emacs/comments/7k54px/snippet_share_make_eshell_search_command_history/), 而 [manateelazycat](https://github.com/manateelazycat)也把这段代码的功能加入到[aweshell](https://github.com/manateelazycat/aweshell/commit/ecaddac98b87f881910dbee8b51a98f00b6d9d5d), Oh yeah !

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

