+++
title = "ReInvent: 重新造轮子系列(序言)"
date = 2025-02-16T22:10:00-08:00
lastmod = 2025-03-15T14:31:09-07:00
tags = ["reinvent"]
categories = ["ReInvent: 重新造轮子系列"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 起因与动机 {#起因与动机}

最近在看 [System Design By Example](https://third-bit.com/sdxjs/unit-test/) 这本书，主旨是通过设计和实现各种日常开发中常见的软件，以理解和提高系统设计(System Design)的能力。

每个章节都会实现一个软件，然后还会有大量的习题来完善这个软件，以练带学。

而我最推崇，并且认为最好的学习方法就是：[费曼学习法(Feynman Technique)](https://ramsayleung.github.io/zh/post/2022/feynman_technique/), 其核心理念就是:

**学习一种新事物最好的方法是，用你的话讲给别人听。**

**通过向别人清楚的解说某一事物，来确认自己是否真的弄懂了这件事。**

所以说，学习最好的方式，是把你学到的东西教给别人。

因此，这个项目就是我在学习和理解 System Design By Example 这本书后，结合参考的论文和个人经验内化出来的学习成果。

也希望其他人也可以从中受益。


## <span class="section-num">2</span> 项目 {#项目}

GitHub: <https://github.com/ramsayleung/reinvent>

原书是用 node + Javascript 编写的，部分代码因API变更而变得不可用，因此本项目也做了对应修改，并将 Javascript 替换成 Typescript, 通过类型系统来降低维护成本。

1.  [单元测试框架]({{< relref "reinvent_unit_test" >}})
2.  [文件备份]({{< relref "reinvent_file_backup" >}})
3.  [HTML Selector]({{< relref "reinvent_selector" >}})
