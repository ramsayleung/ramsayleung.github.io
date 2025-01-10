+++
title = "(翻译)从Helm到Ivy"
description = "An Translated Post about switch from helm to ivy"
date = 2017-03-05T00:00:00-08:00
keywords = ["emacs", "helm", "ivy"]
lastmod = 2025-01-09T19:18:56-08:00
tags = ["emacs", "translation"]
categories = ["翻译"]
draft = false
toc = true
+++

最近，我发现很多Emacs 用户对Ivy 很感兴趣；而且大部份用户都是已经了解过Helm 或者Ido的
当有人在Reddit 上面问 [选择Helm 还是Ido](https://www.reddit.com/r/emacs/comments/51lqn9/helm_or_ivy/)这类问题的时候，我觉得我会给出我自己的选择：
**Ivy**,即使我是一个前Helm 的狂热用户
![](http://www.feer-mcqueen.com/blog/wp-content/uploads/2015/11/miimalism-vs.-maximilasim-700x334.jpg)
_最大或者最小_

**Helm 和Ivy 都是补全框架**.这意味着它们都是Emacs生态系统中用来在用户输入后缩窄可供选择选项的范围的工具。
很自然而然想起的通用例子就是搜索文件。Helm 和Ivy 都可以帮助用户快速搜索文件

它们两者都是框架，这意味着它们都可以用在那些需要补全或者缩窄范围的复杂命令。

例如Helm 有一个命令(**helm-google-suggest**)可以模拟Goole 的搜索框，并在你输入时给出相应的google 提示

Ivy 和Helm 都有相同的目标，但是它们实现的方法却是迥然不同

现在我想站在用户的角度来比较一下这两个工具。我这里指的用户观点是我在不需要了解Helm 和Ivy 的内部工作原理的前提下对这两个工具进行比较。

其实，因为我对 \*elisp\*还谈不上精通，所以也没办法就两者实现细节来进行比较。但是这两个工具我都使用过，所以我可以从用户的角度，跟你分享我使用它们的不同感受。最后，我从Helm 切换到了Ivy

我想先谈Helm.当我使用Spacemacs 的时候，我学会了怎么使用Helm,以Helm 的方式思考, 如何自定义Helm,怎么把Helm 配置得称心如意。

我想我应该算得上是一个中级的Helm 用户吧。我有读过[这篇文章](http://tuhdo.github.io/helm-intro.html) 还有[这篇文章](http://tuhdo.github.io/helm-projectile.html) 以及[Wiki](https://github.com/emacs-helm/helm/wiki) 此外，在长达一年的时间里，我每天都是使用Helm的

**Helm 是一个非常成熟的工具**.根据git 的提交历史，Helm 的开发工作是在2009年左右开始的。
在写这篇文章的时候，Helm 官方的git 仓库有超过26000行elisp 代码

```shell
git clone https://github.com/emacs-helm/helm.git
cd helm
cat *.el | wc -l
# => 26431
```

这还是没有把在MELPA 上查询到跟Helm 有关的包有142个的情况考虑在内的呢。

你可以用Helm来完成任何事情它主要的强大之处在于你可以把Helm 和很多Emacs 的行为整合在一起。你可以以Helm 为中心构造接口，就像Spacemacs 做的那样。Helm 支持非常一致的接口，你可以通过Helm 来做任何事

你可以搜索文件，搜索缓冲区，搜索颜色，搜索项目，搜索你最近编辑过的文件，搜索系统进程, 搜索音乐，搜索网络资源，搜索补全，搜索代码片段，搜索正则表达式，搜索命令，文档
相关描述，手册....

你可以用Helm-projectile(一个Helm 对projectile 非常好的包装)来管理你的项目。你可以用[gitignore.io](https://www.gitignore.io/)来生成gitignore文件，你可以用Helm-bibtex来管理你的参考书目，你可以浏览你的火狐书签

你可以用Helm 来完成任何事。

基于 **tuhdo** 对我在[Reddit](https://www.reddit.com/r/emacs/comments/52lnad/from_helm_to_ivy_a_user_perspective/d7lypeu/) 上面问题的回复，我想指出的一个特性就是Helm 是不使用 _minibuffer_,但是Ivy 是使用的。

所以它可以被配置成总是在当前打开的窗口展示。对于那些大屏幕显示器的用户而言，这个特性真的非常有用，因为你的目光不用在 _minibuffer_ 来回切换：

{{< figure src="http://i.imgur.com/g1Oz9JY.png" caption="<span class=\"figure-number\">Figure 1: </span>补全结果总是显示在同一个窗口" >}}

最终的比较结果是Helm 是非常便利的工具，相信会有数量非常多的Spacemacs 用户告诉你同样的看法。

而Helm 主要的缺点就是它的代码量太大了。我想虽然Helm 的代码量很大，但是它的开发者利用 **elisp** 成功把它打造成了一个相当快的工具了

而且有些时候，Helm 似乎把简单的问题复杂化了;它配置起来也感觉相当臃肿；有时它也会有一些很奇怪的表现，然后导致卡顿，或者让Emacs 过载，即使你做的只是很简单的查询。

或许那些Helm 的高手用户看到这里，会觉得如果我也是个 **elisp** 高手，就不会出现上述问题了。虽然我已经使用Helm 超过一年了，我还是没有找到方法让可以Helm更加稳定。我觉得Helm 在用自己做例子来讲述了什么是化简为繁吧

你可以用Helm 来做任何事；但事实上你并不需要。你可以这样做并不意味着你应该这样做。

在使用Helm 一年以后，我可以告诉你我只是使用了Helm 三分之一或者更小的功能。有些功能我觉得真的很棒，昨天在读了[这篇文章](http://tuhdo.github.io/helm-intro.html) 之后，我又发现了一些新的东西。大部分时间，我都是使用简单的命令来切换缓冲区，或者列举文件

> Helm 只是一个用来补全的包，就好像Ido或者Ivy.它可能很容易使用，一旦有人经历过配置它的困难，就会发现它很难做到让你随心所欲。
>
> 有些人觉得只要可以让他们使用好的工具，即使他们完全不了解这些工具也无所谓。
>
> 但是我就做不到
>
> --abo-abo,Ivy 的开发者，回答["为什么不选择Helm"](https://github.com/abo-abo/swiper/issues/3) 这个问题

**Ivy 为实现最小化，简单化，可定制化，可发现化而努力**.这四个形容词告诉我们很多Helm
和Ivy 这两个工具间不同的设计理念。阅读[Ivy介绍](http://oremacs.com/swiper/) 以便更好了解Ivy的理念。

在写这篇文章的时候，Ivy 只有大概3400行代码，为Ivy 所打造的生态系统：即Swipter 和
Counsel 也只有7500 行代码

```shell
git clone https://github.com/abo-abo/swiper.git
cd swiper
## Only ivy ?
cat ivy.el | wc -l
# => 3442

## count lines of code into the whole swiper ecosystem
cat *.el | wc -l
# => 7526
```

Ivy 真的是很容易上手，下面就是我的全部配置：

```emacs-lisp
(use-package ivy :ensure t
  :diminish (ivy-mode . "")
  :bind
  (:map ivy-mode-map
        ("C-'" . ivy-avy))
  :config
  (ivy-mode 1)
  ;; add ‘recentf-mode’ and bookmarks to ‘ivy-switch-buffer’.
  (setq ivy-use-virtual-buffers t)
  ;; number of result lines to display
  (setq ivy-height 10)
  ;; does not count candidates
  (setq ivy-count-format "")
  ;; no regexp by default
  (setq ivy-initial-inputs-alist nil)
  ;; configure regexp engine.
  (setq ivy-re-builders-alist
        ;; allow input not in order
        '((t   . ivy--regex-ignore-order))))
```

Ivy 是很低调的；它不想让你把一切都整合到Ivy去。它仅仅是提供你必需的补全。你不能像Helm 那样用Ivy 来做任何事；那为什么我还要切换到Ivy 去呢？

虽然Ivy 已经最小化，但是我依然可以用Ivy 来代替我绝大部分日常使用的Helm命令。

因为Ivy是如此简洁， _abo-abo_ 在它上开发了一个叫 **Counsel** 的包； **Counsel** 可以为你提供非常非常多像你在Helm使用的命令

你可以切换缓冲区，搜索文件，在项目级别进行搜索和替换，与Projectile 整合，搜索你最近
编辑过的文件，搜索Emacs 命令，搜索文档，搜索按键绑定，浏览 kill-ring

让我向你介绍我是怎样用Ivy 代替Helm 的。下面是我对那些我需要使用Ivy 来代替Helm的最常用命令的总结。

这些基本是我一直以来最常用的方法。我每分钟会使用三次的 **ivy-switch-buffer** ,我一天会使用五次的 **helm-swoop**, **swiper** 跟 **helm-swoop** 不分伯仲；

对于那些大文件， **Counsel** 有 **counsel-grep-or-swiper**.

我已经用一些非常非常大的标记语言的文件(一百万行左右)来测试过了，一点问题也没有。

| Helm                  | Ivy                | What ?                                        |
|-----------------------|--------------------|-----------------------------------------------|
| helm-mini             | ivy-switch-buffer  | search for currently opened buffers           |
| helm-recentf          | counsel-recentf    | search for recently edited files              |
| helm-find-files       | counsel-find-files | search files starting from ./                 |
| helm-ag               | counsel-ag         | search regexp occurence in current project    |
| helm-grep-do-git-grep | counsel-git-grep   | search regexp in current project              |
| helm-swoop            | swiper             | search string interactively in current buffer |
| helm-show-kill-ring   | counsel-yank-pop   | search copy-paste history                     |
| helm-projectile       | counsel-projectile | search project and file in it                 |
| helm-ls-git-ls        | counsel-git        | search file in current git project            |
| helm-themes           | counsel-load-theme | switch themes                                 |
| helm-descbinds        | counsel-descbinds  | describe keybindings and associated functions |
| helm-M-x              | counsel-M-x        | enhanced M-x command                          |

我觉得你可以看到Ivy 基本的命令对比Helm 的命令也是毫不逊色的。它们可以代替你日常使用的每一条Helm命令。我不是说你可以像Helm 那样用Ivy 来做任何事，但是它已经足够好用了，正如我说的那样，你也不需要任何事都使用Helm 来完成。

说到补全理念这个话题上，Helm 和Ivy 之间的差异并没有那么大。作为一个用户，我可以告诉你的是：Ivy 会让你感觉到更少的臃肿，更加的直观，更加地容易理解。每一次的补全都是可以预见的。

最后，这真的跟个人的品味有关。对于我自己来说，"Ivy 还是Helm" 这样的争论跟"Emacs 还是Spacemacs" "Emacs 还是Ide" "C 还是Java" "简洁还是全能" "Thelonious 还是 Duke"(译者注，两者都是爵士乐作曲家),"Van Der Rohe 还是 Gaudi."(译者注：前者是德国美国
的建筑风格，后者是西班牙加泰罗尼亚的建筑风格)这样的争论是非常相似的。

你选择Helm呢，你会得到一个巨型的包，一系列你不会用到的特性，一堆你可能只是偶尔用一下的功能，一些你会一个小时使用50次的特性。如果你选择Ivy,你会得到一个只拥有那些让你顺心的必要特性的精简的包，你可以很容易地通过 **Counsel** 或者简单的函数对它进行扩展

```emacs-lisp
(ivy-read "Pick:" (mapcar #'number-to-string (number-sequence 1 10)))
```

如果你想要通过Helm 来扩展：

```emacs-lisp
(helm
 :sources
 (helm-build-sync-source "one-to-ten"
                         :candidates
                         (mapcar #'number-to-string (number-sequence 1 10))
                         :fuzzy-match t)
 :buffer
 "*helm one-to-ten*")

```

或者简单的列表：

```emacs-lisp
(helm-comp-read "Pick:" (mapcar #'number-to-string (number-sequence 1 10)))
```

Helm 为用户作了非常多的决定，Ivy 让用户按需求进行定制；Helm 通过耗费非常多的内存来变得快速，Ivy 通过保持简洁来实现快速；Helm 很成熟，Ivy 很青涩；Helm 为Emacs 提供一致性，Ivy 为Emacs 提供简洁性和可预见性；Helm 需要你进行一定的配置，Ivy 开箱即用

我自己是稍偏向Ivy 的，因为我正在使用它；
它更符合我的口味。但是作为一个用户，Helm和Ivy并没有那么大的差别。它们都是非常优秀的包，只是以不用的方式去实现相同的目标

原文地址 <https://sam217pa.github.io/2016/09/13/from-helm-to-ivy/>

在下翻译水平有限，如有错误，还请指出
