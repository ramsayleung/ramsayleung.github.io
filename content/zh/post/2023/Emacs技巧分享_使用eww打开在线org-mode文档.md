+++
title = "Emacs技巧分享: 使用eww打开在线org-mode文档"
date = 2023-03-04T21:55:00-08:00
lastmod = 2025-01-09T18:07:56-08:00
tags = ["emacs"]
categories = ["Emacs技巧"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 技巧 {#技巧}

对于使用org-mode 格式的文本，例如Emacs官方[ tree-sitter 的使用教程](https://git.savannah.gnu.org/cgit/emacs.git/plain/admin/notes/tree-sitter/starter-guide?h=feature/tree-sitter)

在线阅读不是很易读，相当于人脑解析 `org-mode`. 我的个人习惯是使用 `eww` 浏览器来阅读:

{{< figure src="/ox-hugo/tree-sitter-doc.png" link="/ox-hugo/tree-sitter-doc.png" >}}

1.  复制网页链接
2.  使用 `eww` 打开链接
3.  `major-mode` 切换到 `org-mode`, 就可以愉快地使用 Emacs 来阅读 `org-mode` 文本.

{{< figure src="/ox-hugo/eww.gif" link="/ox-hugo/eww.gif" >}}

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

