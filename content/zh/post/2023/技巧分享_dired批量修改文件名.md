+++
title = "Emacs 技巧分享：dired-mode 批量修改文件名"
date = 2023-03-04T21:34:00-08:00
lastmod = 2025-01-09T19:18:24-08:00
tags = ["emacs"]
categories = ["Emacs技巧"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 技巧 {#技巧}

分享一下平时使用 `dired-mode` 批量修改文件名的技巧:

1.  `C-x C-f` 指定的文件目录，进入 `dired-mode`
2.  `C-x C-q` `dired-toggle-read-only`: Edit Dired buffer with Wdired.
3.  批量修改，手段有
    -   使用 `query-replace` 批量修改文件名
    -   使用evil的多行编辑模式
    -   使用 `rectangle-command`: `C-x r t` `string-rectangle`
4.  `C-c C-c` 提交修改或 `C-c C-k` 放弃修改

{{< figure src="~/btsync/org/blog/2023/技巧分享-dired批量修改文件名/img/dired_rename_multi_files.gif" caption="<span class=\"figure-number\">Figure 1: </span>使用 `rectangle-command` 进行批量修改" link="/ox-hugo/dired_rename_multi_files.gif" >}}

{{< figure src="~/btsync/org/blog/2023/技巧分享-dired批量修改文件名/img/dired_rename_multi_files_2.gif" caption="<span class=\"figure-number\">Figure 2: </span>使用 evil的多行编辑模式进行批量修改" link="/ox-hugo/dired_rename_multi_files_2.gif" >}}

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

