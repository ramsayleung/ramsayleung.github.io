+++
title = "(翻译)Mastering Eshell"
description = "An complete instroduction about Eshell"
date = 2017-02-28T00:00:00-08:00
keywords = ["eshell", [",", "emacs"]]
lastmod = 2024-12-30T23:10:56-08:00
tags = ["eshell", "emacs"]
categories = ["emacs"]
draft = false
+++

Emacs 支持若干种shell,但是就功能丰富度，以及与Emacs 的集成程度而言，
无一能望Eshell项背。

Eshell 是一个完全由Emacs Lisp 编写的shell,但是不要以为这样Eshell在功能上就会先天不足，它能代替绝大部分GNU核心功能集和Bourne shell家族的命令以及相关特性。

此外，通过用Emacs Lisp 重写了类似 `ls` 和 `cp` 等常用命令，使Eshell 可以成为真正的跨平台Shell.

但是，Eshell 有一个很大的不足，那就是Eshell 严重缺乏文档与之形成鲜明对比的是，Emacs 及其生态都拥有丰富的文档


## Overview {#overview}

与Emacs 中的其他shell 不一样的是，Eshell 不是继承默认与的子进程进行交互的 `comint-mode`
因为Eshell 不是一个子进程，所以它就没有必要使用 `comint-mode` 了；不过似乎这是件
好事，因为这意味着 `comint-mode` 的例程和钩子是不会作用于Eshell 的。

此外，几乎所有 `comint-mode` 的常用命令，都在Eshell 中重新实现了，并且使用相同的按键绑定.

当然，也有为数不多的按键绑定没有迁移到Eshell,例如在Eshll 中搜索历史命令的=comint-history-isearch-backward-regexp= 是绑定到了 `M-r`,
而原来shell中相同功能的命令的按键就是 `C-r` Eshell 可以在所有的平台都正常工作.

其实和Eshell 真正进行交互的是一个通用的中间件(即 Emacs Lisp/C 的一个库),而该中间件又会跟你的操作系统进行通信，处理例如复制文件 等相关操作。

此外，这个中间件甚至支持在Eshell 中使用 `Tramp` 的特性, 由于Emacs 是血统纯正的Unix 产物，Eshell 是可以重现例如bash 等传统Unix shell 以及 其他GNU 工具链的同样功能的。

此外，如果你使用的是Windows,你应该感到庆幸，因为你再也不用折腾 `cygwin` ,也不需要为跨平台移植Emacs 而担忧依赖问题。

事实上，对比=cygwin= 的bash,在很多方面，Eshell 对Windows 的支持是做得更好滴。

例如，你不再需要 `/cygdrive/c` 来扩展相应的功能，因为Eshell 自身就支持Windows/MS-DOS 的路径
(所以 `cd D:` 或者 `D:` 同样都可以进入D 盘)

虽然Eshell 有很多好处，但是我还是要列出那些让人们很困惑的观点(或者说是误解):

-   Eshell 不是一个终端模拟器，它不是跟shell 进行通信，事实上，它就是一个shell. 它做的所有工作，无论是在屏幕展示数据，还是获取目录里面的信息，它都是通过Emacs 实现的，然后Emacs会再跟你的操作系统进行通信
-   由于Eshell 跟其他进程通信的方式 (特别是异步通信),所以可能导致它的缓冲区(Buffers) 和其他中断操作出现问题
-   Eshell 无法直接支持交互命令 (按照Eshell 的说法，叫做“可视化命令”),例如top ,所以你一定要告诉Eshll应在单独的 `ansi-term` 实例运行此类命令
-   它不是 `bash` 或者 `zsh`,更不是 `csh`,所以不要像操作它们那样操作Eshell 即使Eshell 跟它们真的很像。因此，如果你想更加高效地使用Eshell,你最好把它当作一个不一样的shell


## Commands {#commands}

Eshell 是可以调用几乎所有的已加载的elisp 函数的；这种灵活性是其它shell 无法想象的也是它们力所不能及的。

事实上，这种在shell 里面跟elisp 函数结合的玩法理应得到更多人的支持和推崇，因为它真的很酷(当然也很有用).假如你想在Emacs 里面打开 `foobar.txt` ,你只需调用 `find-file foobar.txt`,Eshell 就会调用对应的 `(find-file "foobar.txt")` ,并为你打开文件


## Technical Details {#technical-details}

所有被Eshell 执行的命令都有一个执行的顺序，这是必需传递给Emacs 的有序列表，
因为这个列表决定了Eshell 的哪一部分应该处理该命令的。

如果该列表中没有找到可以执行你的操作的函数或者对应命令，你会被告知，你输入的是无效命令

假设 你想调用 `cp` 命令，调用顺序如下：

1.  完整的路径 (即 `/bin/cp`),在 `/bin` 目录下执行 `cp`
2.  寻找命令的前缀， _eshell-explict-command-char_ (默认值是 "="),如果有前缀的话,那么，在搜索路径寻找对应的命令
3.  寻找shell 命令的别名 (`alias` 命令)
4.  在搜索路径寻找 `cp` ,即 `$PATH` (或者 `eshell-path-env`) 定义的路径
5.  寻找叫做 `cp` 的Lisp 函数 或者 叫 `eshell/cp` 的elisp 函数

变量 `eshell-prefer-lisp-functions` 让内部的elisp 函数调用要先于外部调用，这意味着，当该变量值为 `t` 的时候，Eshell 会 `最先` 调用elisp 函数，而不是 =最后=才调用；

但是，当命令前缀(即 _eshll-explicit-command-char_)被指定，该变量会被忽略


## Built-in Commands {#built-in-commands}

Eshell 有很多很好用的通过Emacs-Lisp 重写的命令，这些命令实现了绝大部分GNU 核心工具集
或者你所钟爱的shell 的特性，所以这些命令被称为"Alias functions"(别名函数).

但是Eshell 并不是全盘模拟其他Shell 的功能特性，如果你传递了一些参数，试图调用Eshell未实现的功能时，Eshell 会自动调用外部的对应的命令来实现你想要的功能 (当然前提是你已经安装该命令)
下面列出Eshell 已经重新实现的命令：
`cat` , `cp` , `ls` , `cd` , `export` , `dirs` , `du` , `echo` , `env` , `kill`
`ln` , `mkdir` , `mv` , `alias` , `popd` , `pushd` , `pwd` , `rm` , `rmdir`
`time` , `umask`.
Eshell 注重跟原有的GNU工具功能同步，所以你不用担心因Eshell 命令跟其他原生Shell 命令不一致而导致的问题


## Command Interception {#command-interception}

Eshell 有一个很cool 的特性，那就是某些命令会被拦截并且传递给Emacs.

这种机制允许你调用一个命令例如 `man ls`,但是真正调用处理的是Emacs 内置的 `man`.

此外，对于之前提及的交互式命令而言，这种特性是很重要的，因为Eshell 是没有能力处理该命令的。

但真正展现该特性威力的还是那么复杂的命令，例如 `grep` `diff`,因为Emacs 本身就内置了更加强大的 _grep_ 和 _diff_ 工具。

这种特性真真实实展示了Eshell 对比其他shell 的强大之处

下列的命令都会被重定向到Emacs内置的功能去：
`agrep` , `diff` , `egrep` , `fgrep` , `glimpse` , `grep` , `info` , `jobs`
`locate` , `man` , `occur` , `su` , `sudo` , `whoami`
`su` `sudo` `whoami` 是与 `TRAMP` 相关的命令，所以如果你是连接到远程shell 的
这些命令也是可以正常工作的


## Subshells {#subshells}

你可以使用 `$()` 来调用命令，并且把命令对应的输出当作接下来命令的参数，就好像你在bash 那样使用。

但是你要谨记的一样事情就是你是无法使用反引号 =\`\`=来生成一个subshell的。

虽然你也可以使用像调用subshell 的语法来调用标准的elisp form: `(form ....)` 注意没有了 `$`,不过我并不推荐这种用法，因为很多情况，这种用法都是不行的


## Useful Elisp Commands {#useful-elisp-commands}

Eshell 有一套可以让你每天的生活变得更美好的帮助函数(helper function), 此外你可以在Eshell调用几乎所有的elisp 函数，这就意味着，你拥有无上的能力来控制你的shell.

接下来，我会列举那些为Eshell 专门编写的命令和一些我觉得很有用的命令。

我也编写了挺多的elisp 函数了 (部分是专门写给Eshell,其他的就不是了)


### listify ARGS {#listify-args}

将字符串参数解析成 elisp 列表符号，然后打印到屏幕。

该函数不仅可以解析 POSIX 类型的参数，也可以解析 MS-DOS/Windows 类型参数


### addpath PATH {#addpath-path}

将参数 (必须是文件路径) 添加到环境变量 `$PATH`,如果没有参数被指定的话，那么将原有的变量值输出到屏幕


### unset ENV-VAR {#unset-env-var}

移除已有的环境变量


### find-file FILE {#find-file-file}

搜索文件FILE,然后在Emacs 中打开该文件。这个函数与 `TRAMP` 相关，所以也可以远程工作


### dired DIRECTORY {#dired-directory}

在目录 `DIRECTORY` 下打开一个 dired 缓冲区


### calc-eval EXPR {#calc-eval-expr}

在Emacs calculator 执行该表达式 `EXPR`


### upcase STR /downcase STR {#upcase-str-downcase-str}

字符串 STR 大小写转换


### vc-dir DIRECTORY {#vc-dir-directory}

展示在版本控制下的目录 `DIRECTORY` 的状态，跟大多数版本控制工具的 `status` 命令相同


### ediff-files FILE1 FILE2 {#ediff-files-file1-file2}

使用Emacs 的比较引擎 (diff engine) ediff,对文件 `FILE1` `FILE2` 进行比较


## Aliasing {#aliasing}

你可以像在其它主流的shell 那样给Eshell命令赋予别名，操作是一样滴，此外，你甚至可以混合使用elisp 函数和Eshell 命令。

`alias` 命令的格式是 `alias alias-name definition` `definition` 必须由一对单引号 `''` 包围。

你也可以使用其它shell 的参数引用形式：

例如 `$1` 指第一个参数， `$2` 指第二个参数，依此类推，或者 `$=` 指所有的参数。
当参数没有在 `definition` 被引用，Eshell 会自动把参数添加到命令的末尾，并把参数忽略

如果想移除一条命令的别名，只需不对变量 `definition` 赋值 (即 `alias alias-name`) 别名就会被自动移除，如果想列出所有的别名，只需输入 `alias`

Eshell 会把命令的别名及其定义写入到变量 `eshell-aliases-file` 然后统一被变量 `Eshell-directory-name` 管理；然后别名默认会被统一写入到 `~/.Eshell/alias`.

每次你更改一个命令别名，都会重复上面的流程, 另外一个很有用的特性就是别名自动修正 (_auto-correcting aliasing_),
如果你输入 一个无效的命令太多次 (变量 `eshll-bad-command-tolerance` 表示触发自动更正的最低 次数，默认值为3),Eshell会为你真正想执行的命令提供别名.

例如你想输入的是 `cp` 但是输入了太多次的 `co`,所以下次你输入 `co` 的时候，Eshell 就会自动执行 `cp`.

当然，如果你不喜欢这种特性的话，你可以把最低次数设得很大


### Useful Examples {#useful-examples}

让我们把长长的 `find-file` 命令映射到更顺手的别名 `ff`:

```shell
alias ff 'find-file $1'
```

把 `dired` 映射到 `d`:

```shell
alias d 'dired $1'
```


## Visual Commands {#visual-commands}

有一些对Eshell 而言是太复杂的命令，Eshell 是无法直接显示的，所以需要特殊的处理

例如 `top` ,是无法与一些哑终端(dumb terminal)一起正常工作的。

为了使这些命令正常工作，Eshell 会运行一个终端模拟器 `term` 来执行这些的命令 (即被称为可视化的命令)

如果你想修改可视化命令的列表，你可以修改变量 `eshell-visual-commands`


## Command History {#command-history}

Eshell 有功能丰富的命令行历史机制，但是因为Eshell 不是继承 `comint-mode` 的

所以 `comint-mode` 与历史相关的功能，Eshell 是没法用的，不过它绝大部份的功能都已经在Eshell 重新实现了


### M-r /M-s {#m-r-m-s}

向前或者向后搜索命令，支持正则表达式


### M-p/M-n {#m-p-m-n}

在历史命令列表中前进或者后退


### C-p/C-n {#c-p-c-n}

Eshell上一条命令或者下一条命令


### C-c M-r /C-c M-s {#c-c-m-r-c-c-m-s}

回到上一条/下一条历史命令，历史命令必须与现在的命令输入一致。

例如现在的输入是：
`ls` ,那么回到的上一条 /下一条历史命令必须是 `ls`,或者以 `ls` 开头的命令，如 `lsmod`

不足的是，新的经过修改的命令 `comint-history-isearch-backward-regexp` (在 `comint` 键绑定是 `M-r`)在Eshll 是无法使用的，因为Eshell 不是继承于 `comint` (所以在升级中被忽略了)


## History Interaction {#history-interaction}

像bash 和其它shell 那样，Eshell 也支持历史的修改和交互。

如果想要知道历史交互是怎么操作的，你就需要回去翻一下 bash 的手册了。接下来我会总结一下Eshell 大部份的历史交互用法


### !! {#ef9fcd}

重复上一条命令


### !ls {#ls}

重复上一条以 `ls` 开头的命令


### !?ls {#ls}

重复上一条包含 `ls` 的命令


### !ls:n {#ls-n}

从上一条以 `ls` 开头的命令截取第n个参数


### !ls&lt;tab&gt; {#ls-tab}

使用命令补全，显示补全结果中包含 `ls` 的命令


### ^old^new {#old-new}

快速替换，对于上一条命令，使用 `old` 来代替命令中的 `new` (备注：似乎有Bug)


### $\_ {#792e5b}

返回上一条执行的命令的最后一个参数

Eshll 也支持bash 历史修改(例如 !!:s/old/new/),如果你想了解更多的信息，
[the bash reference on history interaction](https://www.gnu.org/software/bash/manual/bash.html#History-Interaction) 可以告诉你你想知道的东西


## Commandline Interaction {#commandline-interaction}


## The Eshell Prompt {#the-eshell-prompt}

你可以通过修改变量 `eshell-prompt-function` 来自定义Eshell 的提示符；该变量
有一个函数定义了Eshell 命令行提示符应该包含的内容。

通过用elisp 来管理Eshell 命令行提示符的配置，你就可以实现你想要的任何特性。

你需要注意的事情就是：你需要告诉Eshell,命令行提示符长什么样子，所以你必须修改变量 `eshell-prompt-regexp`
,那样 Eshell 就会知道你想要的提示符长什么样子了


## The Commandline {#the-commandline}

Eshell 可以使用反斜杠 `\` 来转义新行，以及基本的多行输入。

另外一个输入多行的文学字符串 (literal string)的方法就是使用单引号：输入一个单引号，然后回车，
接着你就可以输入你想输入的内容，最后用另外一个单引号结束输入。

如果你使用双引号的话，Eshell 会自动展开 subshell 命令并且展开相应的变量得益于Eshell 的调用机制，你甚至可以回去继续修改引号里面的文本。

当你想回去修改你不喜欢的内容，让Eshell像你预期那样工作的时候，你就会觉得这种特性真的相当
有用


## Useful Keybindings {#useful-keybindings}

Eshell 做了很多与Eamcs 进行交互的功能的改进，而且，这些改进足以影响你的生活
质量，让我为你一一道来：


### C-c M-b {#c-c-m-b}

将已经某个缓冲区的名字插入到当前光标


### C-c M-i {#c-c-m-i}

将已经某个进程的名字插入到当前光标


### C-c M-v {#c-c-m-v}

将一个环境变量的名字插入到当前光标


### C-c M-d {#c-c-m-d}

在直接输入和延迟输入(回车确认)之间切换 (对不能与来源于其他缓冲区的输入正常工作
的命令来说就很有用了)


## Argument Predicates {#argument-predicates}

参数谓词是一个很擅长过滤文件，甚至elisp列表的工具。

Eshell的谓词语法是参照zsh 的，所以如果你熟悉zsh的参数谓词，你也可以以同样的方式来使用Eshell.

与Eshell 绝大部分迥异的是，参数谓词是有详细的文档的。你可以通过输入 `eshell-display-predicate-help`

或者 `eshell-display-modifier-help` 来查看帮助文档参数谓词用来过滤有相同模式的文件是很有用，你不需再花费额外的时间来使用诸如 `find` `ls` 此类命令。

虽然有帮助手册，但是手册还是很简单，不尽人意，所以我自己总结了一些用法来帮助读者了解相关特性。

但是最好的学习方法还是多尝试，多出错，多总结


### Syntax Reference {#syntax-reference}

我就不把那么多的谓词和修饰符一一列出来了，因为Eshell 的手册已经作了很详细的解释了，你需要做的就是自己查看


### Globbing {#globbing}

Eshell 的匹配模式和其他常用shell 的是基本一致滴：shell 会扩展文件和路径的匹配 模式，然后将匹配后的列表当作参数传递给相应的命令，例如 `ls`.

这就是为什么你一起使用 `find` 和 `xargs` 命令的时候，最好要把 `-print0` 传递给 `find` 并且把 `-0`
传递给 `xargs`.

因为如果你不这样做的话，文件名或者路径名中的特殊字符或者空格就会 让 `xargs` 不知道如何正确地处理。通过使用 `NUL` 字符作为分隔符，保证字符可以被正确地标记，并且文件中紧跟着 `/` 的 `NUL` 字符会被标记为无效字符


### Elisp Lists {#elisp-lists}

如果你把Eshell 的列表理解成输出的 `form` 的elisp列表，你会发现理解起来变得容易因为事实上Eshell 是可以通过Elisp 来处理列表的，而处理列表恰恰是Lisp 擅长的东西最简单的模式扩展就是 `echo *`,该命令会把当前文件夹下所有匹配的文件以列表的形式 打印出来。

因为，正如我先前提及的那样，通配符扩展是同步一致进行的，所以我可以在 在使用 `*` 的同时再使用另外一个修饰符。

例如: 我们把当前文件夹下的所有文件名变成大写的形式，并以列表的形式打印出来：

```shell
/ $ echo *(:U)
("BAR" "BIN/" "DEV/" "ETC/" "FOO" "HOME/" "LIB/" "TMP/" "USR/" "VAR/")
```

请注意，我是怎样在使用模式扩展的同时使用 `()`.这对括号可以让你使用参数修饰符或者 是谓词。

修饰符是可以修饰你的结果列表的(很惊讶吧).

修饰符总是以冒号 `:` 开头滴， 而谓词却不一样。

我会展示另外一个例子，这次这个例子我会使用谓词来过滤目录：

```shell
/ $ echo *(^/)
("bar" "foo")
```

这个 `^` 在上面的命令的作用，是跟在正则表达式中一样，用作取反，而斜杠的作用 `/`
是只代表目录，所以上面的作用就是打印所有文件

对于修饰符和谓词，我也可以不使用模式扩展

```shell
/ $ echo ("foo" "bar" "baz" "foo")(:gs/foo/blarg)
("blarg" "bar" "baz" "blarg")
```

这次我是把所有的 `foo` 代替为 `blarg`.

你可以发现语法是相同的，只是这次我不是使用模式匹配来获取文件列表，而是直接输入文件的列表.

使用参数谓词和修饰符的好处是你大大减少了输入的命令行数量，因为用谓词可以处理权限 ，属主，文件属性，甚至更多方面的问题


### Adding New Modifiers and Predicates {#adding-new-modifiers-and-predicates}

你也可以添加自己的谓词 (`eshell-predicate-alist`)或者修饰符 (`eshell-modifier-alist`):

```emacs-lisp
(add-to-list 'eshell-modifier-alist '(?X . '(lambda(lst)(mapcar 'rot13 lst))))
```

我已经将 `rot13` 绑定到 `X` 了，替换结果如下：

```shell
/ $ echo ("foo" "bar" "baz")(:X)
("sbb" "one" "onm")
```


## Plan 9 Smart Shell {#plan-9-smart-shell}

Eshell 有一个 `Plan 9` 终端的弱化版，叫做 **the Eshell smart display**.

Eshell 的智能展示(smart display)意味着它改进了所有黑客所习惯的 **输入－运行－修改** 工作流程。

智能展示特别之处在于，Eshell 的光标不会像普通的shell那样，落在你运行
的命令的输出后面；

相反，光标的位置会保持在你输入命令的位置，让你可以通过 `M-p` `M-n` 或者其他修改历史的命令更容易地修改你输入的命令.

如果你启用了 `smart display` 模式，你还可以使用 `SPC` 向下翻页，或者使用 `BACKSPACE`
向上翻页来查看那些长时间运行的命令的输出。

如果你按下了任何其它的按键，光标会直接跳到你缓冲区的结尾，就好像你没有启用 `smart display` 运行命令时那样

值得注意的是，如果Eshell 检测到你想回顾最后一条执行的命令时，Eshell 会很贴心地帮你回顾的，但是，如果你没有这样的行为，Eshell 的光标会直接跳转到缓冲区的结尾.

这么看来，Eshell真的很智能，而且它也有一些设置可以让你微调相关的行为。

你会发现智能显示 (smart display)真的非常有用，特别是你可以通过移动按键就能修改

刚刚执行过的命令；例如修改拼写错误的命令或者是给相应的命令添加参数, 智能显示还可以被设置成当命令成功执行时，不使用扩展的 `edit mode`;并且隐藏命令输出, 就好像你执行 `chown` 那样。

这也是我喜欢的玩法，如果你也想试试这种玩法，你可以把下面的elisp 代码添加到你的 `.emacs` 文件：

```emacs-lisp
(require 'eshell)
(require 'em-smart)
(setq eshell-where-to-jump 'begin)
(setq eshell-review-quick-commands nil)
(setq eshell-smart-space-goes-to-end t)
```

如果Eshell 已经被初始化(即你已经在Emacs运行了一个Eshell实例),那样的话，运行
上面的代码是不会起作用的。

你必须在Eshell 里面按下 `M-:` 然后输入 `(shell-smart-initialize)` ,或者直接重启Emacs
智能显示真的是非常有用的特性，但是你一时半刻是很难完全领会其全部的精妙之处滴。

你直接输入一个命令，Eshell的光标就会跳转到缓冲区的结尾，所以你会觉得光标似乎本来就在那里


## Redirection {#redirection}

Eshell 的重定向跟其它shell 的工作方式基本是一样的，但是，有一项非常重要的差异就是Eshell 必须模拟可能不存在的伪设备，例如Windows 平台上的 `/dev/null` 其实是 `NUL`

另外一个值得注意的地方就是：虽然Eshell 支持重定向，但是只是支持输出重定向，是 不支持输入重定向的。

为了避免跳进输入重定向这个坑，你最好使用管道。

重定向到标准输入 标准输出，标准错误都是可以正常工作的，此外，你也可以重定向到多个目标，很不错的特性吧


## To Emacs {#to-emacs}

因为Eshell 在内部用Elisp重新实现了各种伪设备，所以也就无需跟Unix 的设备文件打交道了，甚至，可以用Elisp实现自己的伪设备。

一个很好的例子就是，你可以把重定向到一个你选择的缓冲区，用下面的命令就能实现：

```shell
/ $ cat mylog.log >> #<buffer *scratch*>
```

我之前提到的快捷键 `C-c M-b` 就是可以把一个选定的缓冲区的名字插入到光标前

此外，你也可以把输出直接重定向到Elisp 的符号(不过注意，不要执行错误的设置)

```shell
/ $ echo foo bar baz > #'myvar
/ $ echo $(cadr myvar)
bar
```

如果你将变量 `eshell-buffer-shorthand` 设置为 `t` 的话。

你就可以使用缓冲区的速记名, 例如 `#'*scratch*'`,但是你就不能直接重定向到Elisp 的符号了


## To Pseudo-Devices {#to-pseudo-devices}

Eshell 重新实现了以下的伪设备：


### /dev/eshell {#dev-eshell}

以交互的方式，把结果输出到Eshell


### /dev/null {#dev-null}

把结果输出到 `NULL` 设备


### /dev/clip {#dev-clip}

把结果输出到剪切板


### /dev/kill {#dev-kill}

把结果输出到 `kill ring`

跟通用的shell 一样，使用 `>` 代表覆盖(或者新建);使用 `>>` 代表追加


### To custom virtual target {#to-custom-virtual-target}

你通过修改变量 `eshell-virtual-targets` 创建自己的可视化目标(即存储你想创建的 伪设备的名字的一个列表),以及修改代表重定向行为(即覆盖或追加或插入)的函数 `mode`


## TRAMP {#tramp}

Eshell 可以很好地支持TRAMP,这意味着如果Eshell 所在的目录是在远程服务器的话， 像 `su` `sudo` `whoami` 这样的命令会自动作用在远程服务器

想直接使用TRAMP,你可以像使用 `C-x C-f` 寻找文件那样输入TRAMP的命令符，然后你就可以使用TRAMP 了。

虽然你会觉得Eshell里面使用TRAMP有点奇怪，但是你的确得到了 一个TRAMP的远程shell,不是么？

此外，你不应把TRAMP局限在使用远程shell, 你可以在本地 使用 `sudo` 和 `su` 命令的

有关TRAMP 的更详细的用法，我总结在了另外一篇文章，不过如果你迫不及待想了解更多
有关TRAMP的用法，[官方手册](http://www.gnu.org/software/tramp/) 是一个很好的选择


## Startup Scripts {#startup-scripts}

跟其它的shell 一样，Eshell 也支持 `login` 和 `profile` 的配置文件。

`login` 和 `profile` 配置文件的绝对路径分别保存在变量 `eshell-login-script` 和 `eshell-rc-script`
不过默认情况下，上述两个配置文件都保存在 `~/.eshell/`.

顺便说一下，Eshell的配置文件也是使用 `#` 来注释变量和语句的


## More Customization {#more-customization}

如果你想折腾的话，Eshell 有成百上千的选项供你选择。

如果你想配置Eshell 的话，按下 `M-x` 然后输入 `customize-group` 回车，然后输入 `eshell` 回车确认


## Conclusion {#conclusion}

额，我觉得我已经总结了Eshell 的大部份用法了，希望你可以在其中发现乐趣。

因为与Emacs 的紧密结合，Eshell 有了各种各样突出好用的特性，但是你需要理解的是，Eshell 的诞生不是
为了全盘取代bash 或者其它你喜欢的终端模拟器，它只是希望在Emacs 里面就可以完成我们
日常必需的命令行操作。

如果你要运行很多交互式的命令，Eshell 就可能不是很有用了 因为为了运行你输入的每一条可视化命令，Eshell 都会在Emacs 里面启动一个新的终端模拟器。

Eshell 有TRAMP支持，自定义伪设备，袖珍的elisp REPL和很多非常有用的命令，例如对你打开的文件或者目录，调用 `find-file` 或者 `dired`.正是这种种有用的特性， 让Eshell 成为我工具箱里面一个非常可靠的工具。

原文地址 [Mastering Eshell](https://www.masteringemacs.org/article/complete-guide-mastering-eshell) ,在下翻译水平有限，如有错误，还望指出
