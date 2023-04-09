+++
title = "Emacs技巧分享: 使用eww打开在线org-mode文档"
date = 2023-03-04T21:55:00+08:00
lastmod = 2023-03-04T21:58:52+08:00
tags = ["emacs"]
categories = ["emacs"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 技巧 {#技巧}

对于使用org-mode 格式的文本，例如Emacs官方[ tree-sitter 的使用教程](https://git.savannah.gnu.org/cgit/emacs.git/plain/admin/notes/tree-sitter/starter-guide?h=feature/tree-sitter)

在线阅读不是很易读，相当于人脑解析 `org-mode`. 我的个人习惯是使用 `eww` 浏览器来阅读:

{{< figure src="/ox-hugo/tree-sitter-doc.png" link="/ox-hugo/tree-sitter-doc.png" >}}

1.  复制网页链接
2.  使用 `eww` 打开链接
3.  `major-mode` 切换到 `org-mode`, 就可以愉快地使用 Emacs 来阅读 `org-mode` 文本.

{{< figure src="/ox-hugo/eww.gif" link="/ox-hugo/eww.gif" >}}
