+++
title = "Shell神器fasd与Eshell的不期而遇"
description = "An introduction about tweaking eshell with fasd"
date = 2017-03-02T00:00:00-08:00
keywords = ["eshell", "fasd"]
lastmod = 2025-01-09T18:05:49-08:00
tags = ["emacs", "shell", "eshell"]
categories = ["Emacs技巧"]
draft = false
toc = true
+++

> fasd - quick access to files and directory

之前一位 Windows 用户看到我在 Shell 下面的操作，他很奇怪，觉得明明已经有图形化界面，为什么还要用这种命令行呢，直接用鼠标点击不就很好了么。

我觉得很难直接跟他解释，因为他没有用过Linux/Unix,完全不熟悉命令行，不知道其强大之处，其高效率是图形化界面完全无法比拟的(当然，命令行的学习成本和学习曲线肯定比图型化界面高), So I live in terminal.

而今天我要介绍的神器 fasd 就是可以让命令行操作变得更加高效


## <span class="section-num">1</span> Fasd {#fasd}

在 Shell 下面有非常多的命令操作是与文件和目录相关的，如果你要进入到另外一个目 录你可以使用相对或者绝对路径来访问该目录，但是如果这是一个与当前目录不相关的目 录你就只能通过绝对路径来访问。

以我自己的目录为例，当前目录是 **_home/samray_.emacs.d/elisp/** ,我希望访问 **Document** 目录下一个的子目录 **Python**, 我可以通过下面的命令来访问：

```shell
cd ~/Document/Programming/Python
cd /home/samray/Document/Programming/Python
```

这就是我需要的命令，虽然可以通过 **tab** 进行目录名的补全，但是我还是觉得要输入的东西太多了(正如 Larry Wall 所说，懒惰是程序员的美德). 然后，我发现了 [Fasd ](https://github.com/clvv/fasd)这个神器。它可以让我只输入 **Python** 就进入到我想访问的 **Python** 目录，

神奇吧！:)

---

Fasd以访问的频繁程度和最近是否有访问对文件和目录分配优先级，然后通过判断已访问的文件以及其优先级来切换目录或者打开文件，所以如果你之前已经访问过某个目录.

那么 你很容易就可以切换到那个目录


### <span class="section-num">1.1</span> 常用选项 {#常用选项}

-   **-a(any)**: 匹配文件和目录
-   **-i(interactive)**: 以交互的方式选择文件或者目录
-   **-s(show/search)**: 按照优先级展示文件或者目录
-   **-e &lt;cmd&gt;**:对匹配的文件调用命令&lt;cmd&gt;
-   **-d**:只匹配目录
-   **-f**:只匹配文件
    Fasd 文档还建议你为 fasd的命令选项设置别名
    ```shell
    alias a='fasd -a'        # any
    alias s='fasd -si'       # show / search / select
    alias d='fasd -d'        # directory
    alias f='fasd -f'        # file
    alias sd='fasd -sid'     # interactive directory selection
    alias sf='fasd -sif'     # interactive file selection
    alias z='fasd_cd -d'     # cd, same functionality as j in autojump
    alias zz='fasd_cd -d -i' # cd with interactive selection
    ```
    这样你就可以通过 **z some-dir** 直接进入到某个目录或者 **zz some-dir** 选择进入有多个匹配的特定目录。

    Fasd 还会判断应该显示所有的匹配选项或者是直接选择最佳匹配. 例如你也可以将fasd配合 _subshell_ 使用，例如打开 **foo**
    ```shell
    vim `f foo`
    ```
    又或者打开  **/etc/rc.conf**
    ```shell
    vim `f rc conf`
    ```


### <span class="section-num">1.2</span> 例子 {#例子}

你可以将fasd 配合正则表达式使用，例如列举以 _py_ 结尾的最近访问的文件：

```shell
f py$
```

又或者使用Emacs 打开最近频繁访问的文件 _bar_

```shell
f -e emacs bar
```


## <span class="section-num">2</span> Fasd +Eshell {#fasd-plus-eshell}

fasd 真的可以大幅度提高效率，但是我有点不太满意的是，我是个 Emacser, 我的操作基本是在 Emacs 里完成的，而我在 Emacs里面使用的 shell 是 Eshell,Eshell 似乎不能与 fasd 无缝结合，似乎可以折腾一下。

---

**z** 和 **zz** 命令是无法在Eshell 里面运行，因为 **z** 是 **fasd_cd** 的别名，而\*fasd_cd\* 是一个shell script 函数，Eshell无法运行该函数，代码如下：

```shell
fasd_cd () {
    if [ $# -le 1 ]
    then
        fasd "$@"
    else
        local _fasd_ret="$(fasd -e 'printf %s' "$@")"
        [ -z "$_fasd_ret" ] && return
        [ -d "$_fasd_ret" ] && cd "$_fasd_ret" || printf %s\n "$_fasd_ret"
    fi
}
```

Eshell无法运行该函数，因为Eshell文档的匮乏，我也不知道如何编写跟上面代码等价的 "Eshell script",所以就用 elisp 写一段同样功能的函数好了。

```emacs-lisp
(defun samray/eshell-fasd-z (&rest args)
  "Use fasd to change directory more effectively by passing ARGS."
  (setq args (eshell-flatten-list args))
  (let* ((fasd (concat "fasd " (car args)))
         (fasd-result (shell-command-to-string fasd))
         (path (replace-regexp-in-string "\n$" "" fasd-result))
         )
    (eshell/cd path)
    (eshell/echo path)
    ))
```

函数功能很快就写好了，实现了 **z** 的功能，但是原来的代码一直不能正常运行，折腾了一个多小时都没解决，输出什么都正常，最后 debug 发现是因为显示的路径后面多了一个换行符即 **/home/samray** 变成了 **/home/samray\n**,而输出换行符又不会显示，真
的坑。

最后为命令赋予别名就可以像在 **zsh** 下那样工作了：

```shell
alias z 'samray/shell-fasd-z $1'
```

---

更多的用法就要查阅官方文档了

```shell
man fasd
```

Enjoy Emacs and Shell :)

参考：
<https://github.com/clvv/fasd>

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

