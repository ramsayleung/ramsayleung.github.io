+++
title = "Eshell提示符优化"
description = "Tweak with Emacs shell prompt"
date = 2017-06-07T00:00:00-07:00
lastmod = 2025-01-09T18:06:46-08:00
tags = ["emacs", "eshell", "tool"]
categories = ["Emacs技巧"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 发现帅气的提示符 {#发现帅气的提示符}

近日，我在浏览 [Reddit](https://www.reddit.com/r/emacs/comments/6f0rkz/my_fancy_eshell_prompt/) 的时候，发现了一位 Emacs 用户把他的 Eshell 提示符修改得很帅，如图：

[![](/ox-hugo/eshell_prompt.png)](/ox-hugo/eshell_prompt.png)
本着拿来主义的想法，我就直接把这位小哥的代码添加到了我的配置文件里面：

```elisp
(setq eshell-prompt-function
      (lambda ()
        (concat
         (propertize "┌─[" 'face `(:foreground "green"))
         (propertize (user-login-name) 'face `(:foreground "red"))
         (propertize "@" 'face `(:foreground "green"))
         (propertize (system-name) 'face `(:foreground "blue"))
         (propertize "]──[" 'face `(:foreground "green"))
         (propertize (format-time-string "%H:%M" (current-time)) 'face `(:foreground "yellow"))
         (propertize "]──[" 'face `(:foreground "green"))
         (propertize (concat (eshell/pwd)) 'face `(:foreground "white"))
         (propertize "]\n" 'face `(:foreground "green"))
         (propertize "└─>" 'face `(:foreground "green"))
         (propertize (if (= (user-uid) 0) " # " " $ ") 'face `(:foreground "green"))
         )))
```

效果自然是很 sexy.


## <span class="section-num">2</span> 与原有提示符冲突 {#与原有提示符冲突}

但是我原来使用的 [eshell-prompt-extra](https://github.com/kaihaosw/eshell-prompt-extras) 的效果就被覆盖了。而 `eshell_prompt_extra` 可以提供的额外信息非常多，包括：git, python virtualenv, 以及远程登录时的主机信息，如图：

{{< figure src="/ox-hugo/eshell_extra_prompt.png" link="/ox-hugo/eshell_extra_prompt.png" >}}

如果用上这个 sexy 的提示符，eshell-extra-prompt 的额外的信息就不能显示，感觉好亏:(

鱼和熊掌我都想要，似乎太贪心了？怎么办，自己去修改 `eshell_prompt_extra` 的[源码](https://github.com/kaihaosw/eshell-prompt-extras/blob/master/eshell-prompt-extras.el) :).


## <span class="section-num">3</span> 折腾源码 {#折腾源码}

`eshell_prompt_extra` 这个包注释加上全部代码也只是 400 行，代码也写得很清晰. 其中大部份是辅助函数，而 Eshell 的提示符效果是通过两个 eshell-theme 函数来实现的。use-package 的配置：

```elisp
(use-package eshell-prompt-extras
  :ensure t
  :load-path "~/Code/github/eshell-prompt-extras"
  :config (progn
            (with-eval-after-load "esh-opt"
              (use-package virtualenvwrapper :ensure t)
              (venv-initialize-eshell)
              (autoload 'epe-theme-lambda "eshell-prompt-extras")
              (setq eshell-highlight-prompt nil
                    eshell-prompt-function 'epe-theme-lambda))
            ))
```

而 `epe-theme-lambda` 的代码如下：

```elisp

(defun epe-theme-lambda ()
  "A eshell-prompt lambda theme."
  (setq eshell-prompt-regexp "^[^#\nλ]*[#λ] ")
  (concat
   (when (epe-remote-p)
     (epe-colorize-with-face
      (concat (epe-remote-user) "@" (epe-remote-host) " ")
      'epe-remote-face))
   (when epe-show-python-info
     (when (fboundp 'epe-venv-p)
       (when (and (epe-venv-p) venv-current-name)
         (epe-colorize-with-face
          (concat "(" venv-current-name ") ") 'epe-venv-face))))
   (let ((f (cond ((eq epe-path-style 'fish) 'epe-fish-path)
                  ((eq epe-path-style 'single) 'epe-abbrev-dir-name)
                  ((eq epe-path-style 'full) 'abbreviate-file-name))))
     (epe-colorize-with-face (funcall f (eshell/pwd)) 'epe-dir-face))
   (when (epe-git-p)
     (concat
      (epe-colorize-with-face ":" 'epe-dir-face)
      (epe-colorize-with-face
       (concat (epe-git-branch)
               (epe-git-dirty)
               (epe-git-untracked)
               (let ((unpushed (epe-git-unpushed-number)))
                 (unless (= unpushed 0)
                   (concat ":" (number-to-string unpushed)))))
       'epe-git-face)))
   (epe-colorize-with-face " λ" 'epe-symbol-face)
   (epe-colorize-with-face (if (= (user-uid) 0) "#" "")
                           'epe-sudo-symbol-face)
   " "))
```

代码主要逻辑是调用之前定义的辅助函数，判断是否需要显示 git, python, 远程主机等信息，然后对相应的提示符进行拼接。

而其中出现得比较频繁的 `epe-colorize-with-face` 就是作者定义的一个宏(macro), 用来显示字符串以及对应的 face(其实就是不同的颜色啦). 看懂了代码就好办了，现在就可以自己添加一个 Eshell 主题。


### <span class="section-num">3.1</span> 定义所需的 face {#定义所需的-face}

因为我需要显示的 face(颜色), `eshell-extra-prompt` 并没有定义，所以就只好自己动手啦：

```elisp
(defface epe-pipeline-delimiter-face
  '((t :foreground "green"))
  "Face for pipeline theme delimiter."
  :group 'epe)

(defface epe-pipeline-user-face
  '((t :foreground "red"))
  "Face for user in pipeline theme."
  :group 'epe)

(defface epe-pipeline-host-face
  '((t :foreground "blue"))
  "Face for host in pipeline theme."
  :group 'epe)

(defface epe-pipeline-time-face
  '((t :foreground "yellow"))
  "Face for time in pipeline theme."
  :group 'epe)
```

然后就是按着原有的 Eshell 提示符来组装一个新的 Eshell 主题了，然后把这个主题定义成 pipeline (其实是我自己也没想出比较新颖的名字啦):

```elisp
(defun epe-theme-pipeline ()
  "A eshell-prompt theme with full path, smiliar to oh-my-zsh theme."
  (setq eshell-prompt-regexp "^[^#\nλ]* λ[#]* ")
  (concat
   (if (epe-remote-p)
       (progn
         (concat
          (epe-colorize-with-face "┌─[" 'epe-pipeline-delimiter-face)
          (epe-colorize-with-face (epe-remote-user) 'epe-pipeline-user-face)
          (epe-colorize-with-face "@" 'epe-pipeline-delimiter-face)
          (epe-colorize-with-face (epe-remote-host) 'epe-pipeline-host-face))
         )
     (progn
       (concat
        (epe-colorize-with-face "┌─[" 'epe-pipeline-delimiter-face)
        (epe-colorize-with-face (user-login-name) 'epe-pipeline-user-face)
        (epe-colorize-with-face "@" 'epe-pipeline-delimiter-face)
        (epe-colorize-with-face (system-name) 'epe-pipeline-host-face)))
     )
   (concat
    (epe-colorize-with-face "]──[" 'epe-pipeline-delimiter-face)
    (epe-colorize-with-face (format-time-string "%H:%M" (current-time))
                            'epe-pipeline-time-face)
    (epe-colorize-with-face "]──[" 'epe-pipeline-delimiter-face)
    (epe-colorize-with-face (concat (eshell/pwd)) 'epe-dir-face)
    (epe-colorize-with-face  "]\n" 'epe-pipeline-delimiter-face)
    (epe-colorize-with-face "└─>" 'epe-pipeline-delimiter-face)
    )
   (when epe-show-python-info
     (when (fboundp 'epe-venv-p)
       (when (and (epe-venv-p) venv-current-name)
         (epe-colorize-with-face
          (concat "(" venv-current-name ") ") 'epe-venv-face))))
   (when (epe-git-p)
     (concat
      (epe-colorize-with-face ":" 'epe-dir-face)
      (epe-colorize-with-face
       (concat (epe-git-branch)
               (epe-git-dirty)
               (epe-git-untracked)
               (let ((unpushed (epe-git-unpushed-number)))
                 (unless (= unpushed 0)
                   (concat ":" (number-to-string unpushed)))))
       'epe-git-face)))
   (epe-colorize-with-face " λ" 'epe-symbol-face)
   (epe-colorize-with-face (if (= (user-uid) 0) "#" "")
                           'epe-sudo-symbol-face)
   " "))
```


## <span class="section-num">4</span> 总结 {#总结}

这样一个新的 Eshell 主题就完工了，然后我给 `eshell-extra-prompt` 发了一个[Pull Request](https://github.com/kaihaosw/eshell-prompt-extras/pull/16), 最终效果如下：

{{< figure src="/ox-hugo/epe.png" link="/ox-hugo/epe.png" >}}

Enjoy Emacs, Enjor Tweaking :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

