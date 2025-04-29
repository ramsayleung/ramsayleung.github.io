+++
title = "Linux/Unix Shell 二三事之神器percol"
description = "An introduction about percol"
date = 2017-02-13T00:00:00-08:00
lastmod = 2025-01-09T17:36:15-08:00
tags = ["linux", "shell"]
categories = ["Linux/Unix Shell 二三事"]
draft = false
toc = true
showQuote = true
+++

[Percol](https://github.com/mooz/percol) 是Emacs 的一个非常优秀package, js2-mode作者mooz 的又一力作得益于Unix Shell的管道和重定向设计理念，percol 所有的输入输出变得可交互 percol 给我一种很熟悉的感觉，就是 Eamcs 中helm 增量补全 (incremental completion)的感觉，真的可以10倍提高工作效率。


## <span class="section-num">1</span> 例子 {#例子}

假如你要用git 切换分支，但是分支很多，你不能记住你要切换的分支的名字。那么有percol 你可以：

```shell
$ git checkout $(git branch|percol)
```

那样，你就可以，选择要切换的分支了

平时在Linux/Unix 下，如果要kill 掉某个进程的话，我一般是用 `htop` 或者是ps 找出要kill 掉的进程的pid, 然后在 `kill pid`. 但是现在有了percol, 可以一步搞定所有的步骤。

官网给出的例子函数：

```shell
function ppgrep() {
    if [[ $1 == "" ]]; then
        PERCOL=percol
    else
        PERCOL="percol --query $1"
    fi
    ps aux | eval $PERCOL | awk '{ print $2 }'
}

function ppkill() {
    if [[ $1 =~ "^-" ]]; then
        QUERY=""            # options only
    else
        QUERY=$1            # with a query
        [[ $# > 0 ]] && shift
    fi
    ppgrep $QUERY | xargs kill $*
}
```

又或者是更好地进行查找历史命令：

```shell
function exists { which $1 &> /dev/null }

if exists percol; then
    function percol_select_history() {
        local tac
        exists gtac && tac="gtac" || { exists tac && tac="tac" || { tac="tail -r" } }
        BUFFER=$(fc -l -n 1 | eval $tac | percol --query "$LBUFFER")
        CURSOR=$#BUFFER         # move cursor
        zle -R -c               # refresh
    }

    zle -N percol_select_history
    bindkey '^R' percol_select_history
fi
```


### <span class="section-num">1.1</span> 运行截图 {#运行截图}

[![](/ox-hugo/percol1.png)](/ox-hugo/percol1.png)
有时候，我需要复制当前目录下，某个文件的路径，但是无论是文件管理器，还是shell都要用鼠标来复制指定文件的路径，效率不高且很不方便。

在 [陈斌](http://blog.binchen.org/posts/how-to-use-git-effectively.html) 代码的启发下，我自己写了一个函数来复制当前文件夹某个特定目录的路径，很方便地解决了问题：

```shell
OS_NAME=`uname`
function pclip() {
    if [ $OS_NAME = "CYGWIN" ]; then
        putclip "$@";
    elif [ $OS_NAME = "Darwin" ]; then
        pbcopy "$@";
    else
        if [ -x /usr/bin/xsel ]; then
            xsel -ib "$@";
        else
            if [ -x /usr/bin/xclip ]; then
                xclip -selection c "$@";
            else
                echo "Neither xsel or xclip is installed!"
            fi
        fi
    fi
}

function pwdf()
{
    local current_dir=`pwd`
    local copied_file=`find $current_dir -type f -print |percol`
    echo -n $copied_file |pclip;
}
```

更多的用法就要查看官方文档[ percol](https://github.com/mooz/percol)

Enjoy Shell :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

