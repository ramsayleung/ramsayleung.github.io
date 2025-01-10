+++
title = "Linux/Unix Shell 二三事之过滤器head+tail"
description = "An introduction about head and tail"
date = 2017-02-17T00:00:00-08:00
lastmod = 2025-01-09T17:36:26-08:00
tags = ["shell", "linux", "command_line"]
categories = ["Linux/Unix Shell 二三事"]
draft = false
toc = true
+++

> head - output the first part of files
> tail - output the last part of files

当拥有的数据太多的时候，使用cat 来展示数据的话，数据量过大，屏幕就只能显示最后一部分的数据了。

所以如果你想选取部分的数据的话，cat 就不是一个好选择了。有两个命令可以满足你的要求，分别是 **head** 和 **tail**.顾名思义，head 选取数据的开头部分tail 是选取数据的结尾部分


## <span class="section-num">1</span> 用法 {#用法}

当把 **head** **tail** 当作过滤器来使用的时候，用法很简单

```shell
$ head data
$ tail data
```

默认情况下 **head** 会选取数据开头的10行数据 **tail** 会选取数据最后的10行数据. 如果你想选取更多的数据的时候，你可以指定行数，例如

```shell
$ head [-n line] data
$ tail [-n line] data
```

其中 line 是希望选取的数据行的数量


## <span class="section-num">2</span> 惊艳点 {#惊艳点}

你可能觉得 **head** **tail** 两个命令很简单，似乎用处不大。

是的，就我一直所介绍的那样，单个unix命令只是完成一个特定的工作，但是当它们组合起来的时候，就很威力无穷了


### <span class="section-num">2.1</span> 场景1 {#场景1}

假如你要生成一串密钥来加密你的某个文件，这是很常见的需求，你会怎么办，用python 或者 java 写一个随机数函数来实现么？无需，你用简单的过滤器加Linux/Unix

内置的设备(dev):

```shell
$ cat /dev/urandom | tr -cd "[[:alnum:]]" |head -c 32;echo
```

在Unix/Linux 的机器下，运行上面的命令就可以生成一个包含数字和字母的32个字符长的密钥了。

/dev/urandom 是一个可以通过收集硬件驱动的环境噪音来产生伪随机数特殊的文件，tr 是转换和删除字符的命令；更多详细的东西，以后我会慢慢介绍滴


### <span class="section-num">2.2</span> 场景2 {#场景2}

在日常的开发或者运行环境中，日志是必不可少滴，但是日志是不断产生新的数据的,所以有时候就会出现用编辑器打开日志的时候，就会出现，编辑器不断提醒你文件已经发生了变化，是否重新加载，但是如果只是用cat,tail 来查看日志，日志又是保持在
打开的那个状态，新产生的日志数据是没办法浏览到，果真如此？

其实不然, tail 可以在查看日志的时候，保持日志一直在更新。关键就在 **-f** 选项

```shell
$ tail -f [-n line] file
```

**-f** 选项告诉tail 当到达文件的末尾不要停止。相反，tail 要一直等下去，并且随着文件的增长，显示更多的输出 (-f -&gt; follow)

你也可以模拟日志不断生成的过程：

```shell
$ tail -f -n 20 something.log
```

然后打开一个新的Shell, 运行：

```shell
cat >> something.log
```

使用 **&gt;&gt;** 追加数据，就可以模拟日志生成的过程了


## <span class="section-num">3</span> 总结 {#总结}

要掌握更多的用法还是要查看文档滴：

```shell
man head
```

Enjoy Shell :)
