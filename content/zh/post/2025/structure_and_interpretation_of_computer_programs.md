+++
title = "一本读了八年还没读完的书"
date = 2025-08-04T10:00:00-07:00
lastmod = 2025-11-16T10:13:07-08:00
tags = ["book"]
categories = ["读书感悟", "软件设计"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 缘起 {#缘起}

正如我在之前博客文章《[这些年走过的路：从广州到温哥华](https://ramsayleung.github.io/zh/post/2023/%E8%BF%99%E4%BA%9B%E5%B9%B4%E8%B5%B0%E8%BF%87%E7%9A%84%E8%B7%AF_%E4%BB%8E%E5%B9%BF%E5%B7%9E%E5%88%B0%E6%B8%A9%E5%93%A5%E5%8D%8E/)》[^fn:1]提到的那样，我在大二暑假的时候因缘际会，获得了去一家在深圳的初创公司实习的 Offer。

实习的两个多月时间也快就过去了，我也顺利拿到了 Return Offer，公司也非常有人情味地给实习生办了个欢送典礼。

当时实习的导师，也是这家公司的副总裁，加州州立大学的[刘颖](https://www.linkedin.com/in/yingliu37)教授[^fn:2]，在欢送典礼上给我们几个实习生每人都赠送了一本书作为临别礼物。（可惜换了几次手机，已经找不回当初手捧着书的合照了）

他说这是一本可以帮助我们了解程序本质，以及学习抽象的好书，这本书就叫《[计算机程序的构造和解释](https://book.douban.com/subject/1148282/)》[^fn:3]（Structure and Interpretation of Computer Programs, 简称 SICP，下文使用 SICP 代称）

收到这本书时，我并未料到它会成为一场长达八年的拉锯战。


## <span class="section-num">2</span> 好书不愉悦 {#好书不愉悦}

我在2016年收到这本书，从2017年开始阅读，中间中断了好几次又重新拾起，而时至今日也只读了一半，即五个章节中的前三章。

翻看自己一直以来阅读这本书的日记，总有种看胡适先生一直在打牌的留学日记一样的感受：

```org
** DOING Read The Structure And Interpretation of Computer Programs

   Read SICP everyday, at least 1 hour

   :LOGBOOK:
   CLOCK: [2017-04-03 Mon 20:26]--[2017-04-03 Mon 21:25] =>  0:59
   CLOCK: [2017-03-14 Tue 23:15]--[2017-03-14 Tue 23:40] =>  0:25
   CLOCK: [2017-03-14 Tue 22:45]--[2017-03-14 Tue 23:10] =>  0:25
   CLOCK: [2017-03-12 Sun 15:46]--[2017-03-12 Sun 16:11] =>  0:25
   CLOCK: [2017-03-12 Sun 15:13]--[2017-03-12 Sun 15:38] =>  0:25
   :END:

   <2022-10-08 Sat>
   #+begin_comment
   读了6年了，还是没有读完，重新开始读
   #+end_comment

   <2025-05-25 Sun>
   读了8年，还是没有读完，又开始读
```

看到这里，可能没有读过 SICP 可能会奇怪，为什么读一本书要这么久，如果蜻蜓点水，水过鸭背那样子读完一本书，自然只需要不停地翻页即可。

而 SICP 为了帮助你掌控书中讲解的知识和要点，会有大量的习题，并且把非常多额外的知识点都嵌入到习题中，以练带学。

就数量而言，章节一有46道习题，章节二有97道习题，章节三有82道习题。
如果跳过这些习题，这本书的内容不仅少了一半，而且也失去其精髓，可谓是买椟还珠。

此外，习题不仅数量多，还有相当难度，我每天花一到两个小时阅读，只能完成1-2道习题。

习题完成情况:

-   章节一: 43/46
-   章节二: 88/97
-   章节三: 72/82
-   章节四: TODO
-   章节五: TODO

我把所有的题解都放到了 GitHub 项目: <https://github.com/ramsayleung/sicp_solution>, 并为大部分的题解都配套了单元测试，以验证其正确性，还加上了 GitHub Action 作 CI.

经年累月，我的题解代码和笔记都接近一万行了，这也能侧面说明我为什么读得这么慢了：

```sh
> tokei .
===============================================================================
Language            Files        Lines         Code     Comments       Blanks
===============================================================================
Markdown                1           65            0           46           19
Org                    70         2757         2163            0          594
Racket                 71         3976         3086          256          634
Scheme                 77         2479         1898          110          471
===============================================================================
Total                 219         9277         7147          412         1718
===============================================================================
```

毕竟大部分真正让人进步的阅读，读起来都不是愉悦的：

> 世之奇伟、瑰怪、非常之观，常在于险远，而人之所罕至焉，故非有志者不能至也。
>
> ----王安石《游褒禅山记》


## <span class="section-num">3</span> 主旨 {#主旨}

如果笼统地概括整本书，“无非”是「抽象」，通过使用一门非常简单的语言 `Scheme`, 以及几个非常简单的操作 `cons(构造一个序对)`, `car(取出序对的第一个值)`, `cdr(取出序对的第二个值)`:

```scheme
> (cons 1 2)
'(1 . 2)
> (car (cons 1 2))
1
> (cdr (cons 1 2))
2
```

构造出各种数据结构，如链表，队列，哈希表以及更复杂的组合数据结构; 探寻各种概念，如递归，闭包，高阶函数，赋值，流，程序优化等；

而后两章更进一步，第四章介绍如何实现一个 Scheme 简单的解释器，一个简单的 Prolog 解释器；而第五章介绍计算机体系结构的 CPU 设计，编译器，垃圾回收等。

从括号中的几个简单函数，到最后造出整个计算机体系，有种《道德经》里道生万物的感觉：

> 道生一，一生二，二生三，三生万物。


## <span class="section-num">4</span> 计算机科学与工程 {#计算机科学与工程}

我本科专业读的是软件工程(Software Engineering )，翻看当初的专业培养计划，从计算机导论开始入门，到程序设计基础，面向对象程序设计基础，数据结构，操作系统，计算机组成原理，再到计算机网络，汇编与编译原理，数据库原理到软件工程等。

学习完这些课程，可以成为一个合格的软件工程师，但广义的计算机专业还有一门专业，叫计算机科学(Computer Science)，我一直很疑惑两者之间的差别是什么，就我所见过的不同学校的培养计划里面，两者的课程都非常相似。

而在阅读这本1984年麻省理工就出版的计算机科学的教材时，我找到了我想要的答案。

最初的计算机科学是数学，电子工程和软件设计的交叉学科，计算机科学的学生需要兼备这三者的专业知识。

而三位作者也是高屋建瓴，在数学，电子工程和软件领域旁征博引，各种知识信手拈来，
如练习3.59关于微积分的内容，通过流来处理幂等数积分，练习3.60-3.62都是关于积分的内容。

如3.5章里面，通过流来描述信号处理系统中的「信号」，
练习3.73用程序的流(stream)来表示电流或者电压在时间序列上的值，用以模拟电子线路。

如 3.3章里面，通过程序来建立数字电路的反门，与门，或门，再通过这样的电子元件建立起半加器，
再通过多个半加器实现全加器，实现二进制的加法，从程序到模拟电路，再用模拟电路来构造计算机的处理器。

不同学科的知识在一本书中融会贯通，再配合这个 `eval-apply` 表达式的配图，总有一种太极的感觉，难免让我有种读计算机哲学书的感觉:

{{< figure src="/ox-hugo/apply_eval.jpg" >}}


## <span class="section-num">5</span> 优美的括号 {#优美的括号}

书中那些非常有趣或者优美的代码


### <span class="section-num">5.1</span> 图形构造：从点到面的抽象 {#图形构造-从点到面的抽象}

第二章介绍了复合的数据结构时，就提到了如何去画图，先画点，再画线，然后要求完成习题连线成面，构造出图形。

{{< figure src="/ox-hugo/exercise-2-52-wave.png" >}}

新的习题再要求通过变换，组合图形，构造新的复杂图形:

{{< figure src="/ox-hugo/wave-square-limit.png" >}}

最后把类似的变换应用到图片上:

{{< figure src="/ox-hugo/square-limit-einstein.png" >}}


### <span class="section-num">5.2</span> 蒙特卡罗模拟来计算 π {#蒙特卡罗模拟来计算-π}

所谓的蒙特卡罗模拟（Monte Carlo Simulation）是一种通过随机采样和统计计算来求解的数值方法，通过大量随机实验模拟不确定性，从而估算复杂系统的可能结果。

用人话来说就是不断地试，在试错的过程中逼近确定解，试的次数越多，结果越准确。

书中就介绍了一种通过蒙特卡罗模拟来计算 π 的方法, 就像通过随机撒豆子估算圆的面积，概率统计将抽象的π转化为可计算的实验：

> 举例来说，6/π^2 是随机选取的两个整数之间没有公因子（也就是说，它们的最大公因子是1）的概率。我们可以利用这一事实做出π的近似值。

完全读不懂这段话，没理解是怎么可以算出π的近似值的。

查阅资料后得知:

随机选取两个正整数，它们互质（即最大公约数GCD为1）的概率是 \\(\frac{6}{\pi^2}\\) , 所谓的 互质指两个数没有公共因子（如8和15互质，但8和12不互质，因为公约数为4）。

这一结论源自数论中的经典定理（涉及黎曼ζ函数），我们只需利用其概率公式反推π即可。

而用蒙特卡罗模拟步骤来计算 \\({\pi}\\):

随机实验：重复多次随机选取两个整数，检查它们的GCD是否为1。

例如：

-   (3, 5) → GCD=1（计数+1）
-   (4, 6) → GCD=2（不计数）

统计概率：

若总实验次数为 N，其中 k 次GCD=1，则互质概率的估计值为 \\(\frac{k}{N}\\)

关联π：

根据数论结论 \\(\frac{k}{N} \approx \frac{6}{\pi^2}\\)，解得 \\(\pi \approx \sqrt{\frac{6N}{k}}\\)。

当直接计算π困难时，可通过概率实验间接逼近。

这里利用了数论中的概率规律，将π与随机事件联系起来。(高等数学对于我来说已是雪泥鸿爪，更遑论数论的知识了)

```racket
#lang racket

(define (estimate-pi trials)
  (sqrt (/ 6 (monte-carlo trials cesaro-test))))

(define (cesaro-test)
  (= (gcd (rand) (rand)) 1))

(define (monte-carlo trials experiment)
  (define (iter trials-remaining trial-passed)
    (cond ((= trials-remaining 0)
           (/ trials-passed trials))
          ((experiment)
           (iter (- trials-remaining 1) (+ trials-passed 1)))
          (else
           (iter (- trials-remaining 1) trials-passed))))
  (iter trials 0))
```

这里的蒙特卡罗实现真的是优雅，它将数学理论（互质概率）转化为寥寥数行的递归实验。

而基于流的实现更是优美:

```scheme
(define (monte-carlo experiment-stream passed failed)
  (define (next passed failed)
    (cons-stream
     (/ passed (+ passed failed))
     (monte-carlo
      (stream-cdr experiment-stream)
      passed
      failed)))
  (if (stream-car experiment-stream)
      (next (+ passed 1) failed)
      (next passed (+ failed 1))))
```


## <span class="section-num">6</span> 尾声：括号里的计算机哲学 {#尾声-括号里的计算机哲学}

直到今天，我也只读完前三章。有时我会问自己：这本书究竟给了我什么？

它没有教我实用的编程技巧，也无关面试刷题。

但是我好像又抓住了一些东西，尤如用手拢过一团烟雾，张开手，并未见留下什么，但是手上还残留着它的气味。

如今，MIT 的课程已用 Python 替代 Scheme，但 SICP 的价值从未褪色。

SICP 不是一本教你「如何编程」的书，而是一把钥匙，是一座桥，连接着工程的实用与科学的纯粹。

那些括号中的表达式，最终在我脑中化成了某种朦胧却持久的气味：

****一种对抽象本质的直觉****


### 推荐阅读 {#推荐阅读}

-   旅加经历
    -   [这些年走过的路：从广州到温哥华](https://ramsayleung.github.io/zh/post/2023/%E8%BF%99%E4%BA%9B%E5%B9%B4%E8%B5%B0%E8%BF%87%E7%9A%84%E8%B7%AF_%E4%BB%8E%E5%B9%BF%E5%B7%9E%E5%88%B0%E6%B8%A9%E5%93%A5%E5%8D%8E/)
    -   [加拿大之初体验](https://ramsayleung.github.io/zh/post/2023/%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B9%8B%E5%88%9D%E4%BD%93%E9%AA%8C/)
    -   [登陆加拿大一年后的体会](https://ramsayleung.github.io/zh/post/2024/%E7%99%BB%E9%99%86%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B8%80%E5%B9%B4%E7%9A%84%E4%BD%93%E4%BC%9A/)
-   工具流分享
    -   [简明写作指南](https://ramsayleung.github.io/zh/post/2024/%E7%AE%80%E6%98%8E%E5%86%99%E4%BD%9C%E6%8C%87%E5%8D%97/)
    -   [我的写作流](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E5%86%99%E4%BD%9C%E6%B5%81/)
    -   [我的画图流：画图工具与技巧分享](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E7%94%BB%E5%9B%BE%E6%B5%81/)
    -   [我的搜索流：高效搜索经验分享](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E6%90%9C%E7%B4%A2%E6%B5%81/)
    -   [最好的学习方式：费曼学习法(Feynman Technique)](https://ramsayleung.github.io/zh/post/2022/feynman_technique/)
    -   [系统思考：既见树木，又见森林](https://ramsayleung.github.io/zh/post/2021/%E7%B3%BB%E7%BB%9F%E6%80%9D%E8%80%83/)
-   思考感悟
    -   [编程十年的感悟](https://ramsayleung.github.io/zh/post/2024/%E7%BC%96%E7%A8%8B%E5%8D%81%E5%B9%B4%E7%9A%84%E6%84%9F%E6%82%9F/)
    -   [那些年，我从微信支付学到的东西](https://ramsayleung.github.io/zh/post/2023/%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%98%E7%A6%BB%E7%BA%BF_%E6%88%91%E5%B8%A6%E8%B5%B0%E4%BA%86%E4%BB%80%E4%B9%88/)
    -   [杂谈ai取代程序员](https://ramsayleung.github.io/zh/post/2025/%E6%9D%82%E8%B0%88ai%E5%8F%96%E4%BB%A3%E7%A8%8B%E5%BA%8F%E5%91%98/)
    -   [为什么梦想买不起，故乡回不去](https://ramsayleung.github.io/zh/post/2023/%E7%BD%AE%E8%BA%AB%E4%BA%8B%E5%86%85/)
-   软件工程
    -   [软件设计的哲学](https://ramsayleung.github.io/zh/post/2025/a_philosophy_of_software_design/)
    -   [测试技能进阶(一): 软件质量认知](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%80_%E8%BD%AF%E4%BB%B6%E8%B4%A8%E9%87%8F%E8%AE%A4%E7%9F%A5/)
    -   [测试技能进阶(二): Parameterized Tests](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/)
    -   [测试技能进阶(三): Property Based Testing](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%89_property_based_testing/)

<div class="qr-container" center>

<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" class="qr-container" width="160px" height="160px" center="t" />
公号同步更新，欢迎关注👻

</div>

[^fn:1]: <https://ramsayleung.github.io/zh/post/2023/%E8%BF%99%E4%BA%9B%E5%B9%B4%E8%B5%B0%E8%BF%87%E7%9A%84%E8%B7%AF_%E4%BB%8E%E5%B9%BF%E5%B7%9E%E5%88%B0%E6%B8%A9%E5%93%A5%E5%8D%8E/>
[^fn:2]: <https://www.linkedin.com/in/yingliu37>
[^fn:3]: <https://book.douban.com/subject/1148282/>
