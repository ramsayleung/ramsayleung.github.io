+++
title = "Linux/Unix Shell 二三事之过滤器cat"
description = "An introduction about cat"
date = 2017-02-22T00:00:00+08:00
lastmod = 2022-02-23T18:51:24+08:00
tags = ["linux", "shell", "command_line"]
categories = ["linux"]
draft = false
toc = true
+++

> cat - concatenate files and print on the standard output


## <span class="section-num">1</span> 过滤器 {#过滤器}

何谓过滤器呢，例如cat,grep,wl 之类的命令就是过滤器了。这样的命令
读取数据，对数据执行一些操作，然后写入结果。更准确地说，过滤器就是任何能够从标准
输入读取 **文本** 数据，并向标准输出写入 **文本** 数据的命令。又因为Unix 的 **KISS**
设计理念，所以每个程序都被设计成能够出色完成一项特定任务的工具。又因为重定向和
管道的存在，使得可以将这些工具组合起来，发挥无穷威力


## <span class="section-num">2</span> cat {#cat}

在shell 里面运行cat,你会被要求输入文本数据，当你输入一行数据以后，然后按下回
车你输入的数据就会显示在屏幕，当你按下 ^D(&lt;ctrl&gt;+d),发送eof 信号给shell,退出
cat。cat 做的事就是把你输入的字符，复制到标准输出 (一般情况是指你的屏幕).看到
这里有人或许会质疑，这东西有什么用呢？似乎什么都作不了。不，它的用处很大呢，
且容笔者细细禀来


### <span class="section-num">2.1</span> 场景1 {#场景1}

假如你要新建一个文本文件，里面只是很少的文本，你会怎么做呢？一般情况下，都是用
vim/emacs 新建一个文本文件，然后输入几行文字，然后保存退出。这是一般的做法，
看到这里，很自然有人会发问，难道有更优雅的解决方法？有，不用打开文本编辑器写入文本
的hacking方法：

```sh
cat > data
```

输入数据，然后 ^D(&lt;ctrl&gt;+d) 保存。你就新建了一个文本了。当然，如果你已经有一个 **data** 文件
,就会被代替，当然，你也可以也在原来文本末尾添加的方法：

```sh
cat >> data
```


### <span class="section-num">2.2</span> 场景2 {#场景2}

如果你有一个短文件，你想查看一下，同样，你可以使用cat

```sh
cat < data
```

当然，你也可以省略 **&lt;** 这个重定向符号：

```sh
cat data
```

抑或是，你想显示某个大文件的最后一部分，你也可以如上操作。或许你会觉得，这个功能
很多命令也有，最典型的就是 **tail**. 但是如果 **cat** 可以很完美地很其他过滤器结合
充当整套管道线工具流的起始端，这个以后慢慢再阐述


### <span class="section-num">2.3</span> 场景3 {#场景3}

如果你想复制文本文件，你首先会想起什么命令？ **cp**,很自然嘛，我也不例外，但是cat
也可以实现同样的功能，很意外吧：

```sh
cat < file > newfile
```

即把 **file** 复制到标准输出，然后再把 **file** 当作标准输入复制到 **newfile**.hacking!


### <span class="section-num">2.4</span> 场景4 {#场景4}

如果你想把多个文本文件的组合到一个文件，你会怎么做？用编辑器打开所有的文件
然后 select,cut,paste,save.我也会很自然地想到这个方法，但是是否存在着更
优雅的解决方案呢？当然：

```sh
cat file1 file2 file3 >newfile
```


## <span class="section-num">3</span> 总结 {#总结}

上面已经介绍了挺多cat 的使用场景了，你觉得cat 表现滴怎么样呢？相信你的感觉是
还行，但是并没有，我吹嘘的那么令人惊艳。因为这只是cat 最基本的功能，它最大的
用法还没有完全展现出来，笔者先举一例，以后再慢慢详叙：

```sh
cat file |grep "something" |sort -n |tee newfile
```

| 语法                           | 用法           |
|------------------------------|--------------|
| cat &gt; file                  | 读取输入，创建新的文件或替换 |
| cat &gt;&gt;file               | 读取输入，追加新的文件 |
| cat file/cat &lt;file          | 显示一个已有文件 |
| cat &lt;oldfile&gt; newfile    | 复制一个文件   |
| cat file1 file2 file3&gt;file4 | 组合多个文件   |