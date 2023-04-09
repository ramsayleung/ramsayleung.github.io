+++
title = "Linux/Unix Shell 二三事之过滤器diff"
description = "An introduction about diff"
date = 2017-02-28T00:00:00+08:00
lastmod = 2022-02-23T18:56:31+08:00
tags = ["linux", "shell", "command_line"]
categories = ["linux"]
draft = false
toc = true
+++

> diff - compare files line by line

如果你有使用过git,那么你一定不会对diff 陌生，因为对你源文件和修改后的文件进行比较的就是 **diff** 这个大名鼎鼎的家伙了。

多年以来， **diff** 都一直是非常重要的工具，上古大神 都是使用 **diff** 和 **patch** 对程序进行差分和打补丁滴(现在有git了，但是diff同样发挥着重要作用)


## <span class="section-num">1</span> 语法 {#语法}

diff 的语法如下

```shell
diff [OPTION].... file1 file2
```

_OPTION_ 指不同的选项参数，file1,file2 是文本文件的名字，如果比较的两个文件相同 diff 将不输出任何东西。如果两个文件有差异，diff 会显示一系列的指示，让你可以把第一个文件修改为与第二个文件一致


## <span class="section-num">2</span> 例子 {#例子}


### <span class="section-num">2.1</span> 用法一 {#用法一}

现在有两个文件，分别保存着不同的地址。
address1 包含：

```text
guangdong
shanghai
beijing
chengdu
```

address2 包含：

```text
guangdong
shanghai
beijin
chengdu
```

你可以注意到两个文件的区别就是第三行的 _beijing_.然后运行 **diff**

```shell
diff address1 address2
```

输出结果：

```text
3c3
< beijing
---
> beijin
```

似乎有点难以理解，输出结果描述了什么呢？其实diff 是在指导如何修改不同的文件使之一致 **&lt;** 后接的是文件1中与文件2不同的部分， **&gt;** 后接的是文件2中与文件1不同的部分

diff 的输出使用3个不同的单字符指导：a(append,追加),c(change,修改),d(delete,删除). 在上面的例子，只是看到一个 _c_,意味着，如果想把 _address1_ 修改成 _address2_ 只需将 _address1_ 的第三行修改成 _address2_ 的第三行


### <span class="section-num">2.2</span> 用法2 {#用法2}

现在把 _address2_ 的最后一行删除，看看运行 _diff_ 结果如何：
address1 包含：

```text
guangdong
shanghai
beijing
chengdu
```

address2 包含：

```text
guangdong
shanghai
beijing
```

```shell
diff address1 address2
```

输出结果：

```text
4d3
< chengdu
```

在该例子中，为了将 _address1_ 变成 _address2_ 只需删除 _address1_ 的第四行


### <span class="section-num">2.3</span> 用法3 {#用法3}

现在把 _address1_ 的最后一行删除，看看运行 _diff_ 结果如何：
address1 包含：

```text
guangdong
shanghai
beijing
```

address2 包含：

```text
guangdong
shanghai
beijing
chengdu
```

```shell
diff address1 address2
```

输出结果：

```text
3a4
> chengdu
```

想将第一个文件转换成第二个文件，只需在第一个文件追加第二个文件的第四行(即在第一个文件的第 **3** 行之后追加第二个文件的第 **4** 行)


## <span class="section-num">3</span> diff 选项 {#diff-选项}

因为diff 是一个相当强大也是一个相当复杂的命令，所以我没办法将所有的用法一一道
尽所以笔者将比较常用的选项列举出来

-   **-b**:忽略制表符(不忽略所有的空白符，指忽略空白符数量的差异),例如下面的两行是相同的

<!--listend-->

```nil
a    a
a a
```

-   **-B(blank lines)**:忽略所有的空白行
-   **-c(context)**:以上下文的形式显示差异内容，对比默认输出更加容易理解(但是也更加繁杂)
-   **-q(quiet)**: diff 静默设置，即如果文件file1和file2有差异，diff 也只会显示 _File file1 and file2 differ_
-   **-w(whitespace)**:忽略所有的空白符
-   **-u(unified output)**: 上下文形式显示的改进，不会输出重复行
-   **-y**:将文件分成两列或多列并排进行输出(非常直观，但是输出很繁杂)

还是老话，更多的用法就需要：

```shell
man diff
```