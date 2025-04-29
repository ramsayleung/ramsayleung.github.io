+++
title = "提高Emacs启动速度"
description = "Tips to reduce emacs startup time"
date = 2016-10-22T00:00:00-07:00
keywords = ["emacs"]
lastmod = 2025-01-09T18:05:22-08:00
tags = ["emacs"]
categories = ["Emacs技巧"]
draft = false
toc = true
showQuote = true
+++

之前看到个笑话：

> -   A: 在等待emacs 加载的时间里，你会干什么？
> -   B: 打开Vim,修改代码，保存，退出

有时候，经常看到社区里面有人吐嘈Emacs 什么都好，就是启动时间太长了，其实是存在一些技巧来缩短加载时间的


## <span class="section-num">1</span> 技巧1 {#技巧1}

在你的 `.emacs` 或者相应的初始化文件里面添加如下代码

```emacs-lisp
# Increase the garbage collection threshold to 128 MB to ease startup
(setq gc-cons-threshold (* 128 1024 1024 ))
# your configuration code
# ......
# Garbage collector-decrease threshold to 5 MB
(add-hook 'after-init-hook (lambda () (setq gc-cons-threshold (* 5 1024 1024))))
# init.el ends here
```

`gc-cons-threshold` 指定了emacs 进行垃圾回收的阀值，默认值是 `800000byte`,实在是太小了，所以Emacs 会在启动期间进行非常多次的垃圾回收，启动时间自然长了。

在加载完以后，再把 `gc-cons-threshold` 的值调低，当然，如果你的内存很大，也可以不改回来


## <span class="section-num">2</span> 技巧2 {#技巧2}

`(let((file-name-hander-alist nil))init.file)` 包裹(wrap)你的初始化文件，即：

```emacs-lisp
(setq gc-cons-threshold (* 500 1024 1024))
(let ((file-name-handler-alist nil))
  ...

  ** your config goes here **

  ...
  )
(add-hook 'after-init-hook (lambda () (setq gc-cons-threshold (* 5 1024 1024))))
(provide 'init)
;;; init.el ends here
```

因为 `file-name-handler-alist` 的默认值是一些正则表达式，也就是说Emacs 在启动过程中加载el和elc 文件都会将文件名和正则表达式进行匹配


## <span class="section-num">3</span> 技巧3 {#技巧3}

Emacs lisp 有一项auto-load 的技术，类似延迟加载，合理运用延迟，让笔者的Emacs启动加载时间减少一半，因为笔者用 `use-package` 这个macro,而 `use-package` 又集成了延迟加载的功能，所以笔者就直接拿自己的代码举例了


### <span class="section-num">3.1</span> :after {#after}

```emacs-lisp
;;; Export to twitter bootstrap
(use-package ox-twbs
  :after org
  :ensure ox-twbs
  )
```

`:after` 关键字的作用基本跟 `with-eval-after-load` 的作用是相同的，所以笔者所
有类似的org-mode 插件包都会在org-mode 加载以后才会加载


### <span class="section-num">3.2</span> :commands {#commands}

```emacs-lisp
(use-package avy
  :commands (avy-goto-char avy-goto-line)
  :ensure t)
```

这里就贴上[use-package](https://github.com/jwiegley/use-package)文档 的说明了

> When you use the :commands keyword, it creates autoloads for those commands
> and defers loading of the module until they are used

也就是 `:commands` 关键字就创建了后面所接的命令的 `autoloads` 机制了


### <span class="section-num">3.3</span> :bind :mode {#bind-mode}

```emacs-lisp
(use-package hi-lock
  :bind (("M-o l" . highlight-lines-matching-regexp)
         ("M-o r" . highlight-regexp)
         ("M-o w" . highlight-phrase)))

(use-package vue-mode
  :ensure t
  :mode ("\\.vue\\'" . vue-mode)
  :config (progn
            (setq mmm-submode-decoration-level 0)
            ))
```

附上文档说明

> In almost all cases you don't need to manually specify :defer t. This is implied
> whenever :bind or :mode or :interpreter is used

也就是说，当你使用了 `:bind` 或者 `:mode` 关键字的时候，不用明确指定 `:defer` 也可以实现延迟加载机制。

当然你也可以，直接使用 `:defer` 关键字来指定延迟加载. 不过前提是，你要明确它加载的时机

> Typically, you only need to specify :defer if you know for a fact that some
> other package will do something to cause your package to load at the appropriate
> time, and thus you would like to defer loading even though use-package isn't
> creating any autoloads for you.

贴上笔者自己的代码，可以更加清晰

```emacs-lisp
(use-package anaconda-mode
  :defer t
  :ensure t
  :init(progn
         (add-hook 'python-mode-hook 'anaconda-mode)
         (add-hook 'python-mode-hook 'anaconda-eldoc-mode)
         ))
```

这样 `anaconda-mode` 就会在 `python-mode` 加载以后被加载

Enjoy Emacs :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

