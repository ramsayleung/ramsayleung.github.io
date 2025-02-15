+++
title = "杂谈AI取代程序员"
date = 2025-02-14T21:15:00-08:00
lastmod = 2025-02-15T01:28:56-08:00
tags = ["ai", "programmer"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

最近国产的DeepSeek 模型的爆火，让「AI取代程序员」的说法又甚嚣尘上, 比如在国内著名的程序员社区 V2ex 搜索「[AI取代程序员](https://www.sov2ex.com/?q=AI%E5%8F%96%E4%BB%A3%E7%A8%8B%E5%BA%8F%E5%91%98)」[^fn:1]，可以搜索出 69627 条结果(截止2025-02-15):

{{< figure src="/ox-hugo/v2ex_ai_replace_programmer.jpg" >}}

以至于我的一位非程序员朋友都来关心我，问我是否有被取代的担忧。

在感谢朋友的关心之后，我也开始静下心来思考这个问题，以及形成这种想法背后可能的原因，然后就有了这篇文章。


## <span class="section-num">2</span> AI取代程序员 {#ai取代程序员}

在谈论AI能否真的取代程序员之前，我想先聊一下程序员的工作是什么, 毕竟定义好了问题才会分析.


### <span class="section-num">2.1</span> 研发流程 {#研发流程}

既然是程序员，那工作肯定是写代码嘛, 但是就我个人经验来说，我真正花费在写代码的时间可能连我工作时间30%都占不到.

一个新功能大体的研发流程如下：
当产品经理或者老板提出一个新需求的时候，程序员需要评估需求的可行性，然后撰写设计文档，评审设计方案，而后编码，测试再灰度上线功能，最后对新功能增加新监控告警，提供各项数据指标。

由此可知, 产品经理只会跟你说他/她想要什么(what), 但是产品经理肯定不知道如何实现这个功能的(how), 需要程序员把模糊又朦胧的问题具体化和细致化，然后再根据方案来编码。

遇到不合理的需求时，还需要告知产品经理，你的想法是不切实际的，比如经典的「根据手机壳颜色切换壁纸」需求

但是我们在使用AI的时候，更多是一个具体而明确的任务，并且是我们针对的需求已经拆分好的任务，
所以AI在做的只是将具体的设计方案翻译成代码的步骤。

这不禁让我想起自己当年在微信支付7级晋升8级的答辩项目：一个研发提效的代码生成框架。

我当时可谓信心满满，我的项目可以根据模板直接生成代码，不再需要手写CRUD的代码，以此来给工程师提效。

评委就询问我：

> 你有对项目开发的各个环节的耗时和阻塞时间做量化分析么？
>
> 编码是整个研发流程效能最低的部分么？
>
> 从拿到需求到开发上线，只有编码可以提效么?

评委这么一顿批头盖脸的输出，直接把我的信心给浇没了。

虽然我写的代码生成工具远不如大模型加持的AI强大，但看起来都只局限于解决编码问题，研发流程的其他问题并没有解决。


### <span class="section-num">2.2</span> 功能性与非功能性需求 {#功能性与非功能性需求}

在程序开发中，除了有功能性需求，还需要非功能性需求.

而功能性需求恰恰是其中最简单直接的，是满足「又不是不能用」的最基本要求。

比如我需要实现加法，这个就是功能性需求，AI就需要帮我把加法实现出来。

而所谓的非功能性需求，比较常见的是可扩展性(scalability)，可维护性(maintainability) 和性能(performance).

还是以加法为例, 我可能需要后面扩展到乘法和除法, 或者扩展到复数或者矩阵加法，要怎么易于扩展呢？

这些都是AI生成代码是没有考虑到的因素，AI比较擅长的可能是给它一个需求，它生成一段代码给我们。

但是在软件研发的生命周期中，有大概80%的时间，都是维护已有的系统，谁家没有个已经在跑的系统呢。

而给已有的系统上增加功能，就需要考虑各种奇怪的兼容性，相当于带着锁链来跳舞，并不能像从无到有，什么都不用考虑的那般洒脱。

而这样的限制，又是AI生成代码时未曾考虑的。所以从零开始的日抛型，不需要考虑维护成本的项目，很适合由AI来生成.

但即使功能性的需求，有时候AI也会做不好。

我之前在工作中需要根据已有的Schema, 使用Rust 写入Parquet数据，Rust本来就新，加之操作Parquet的库就更少了, 所以ChatGPT, Claude或者Gemini 都没有给出我满意能跑的结果，最后还是靠自己去读 Parquet库的源码找出的解决办法。

无论模型怎么变，AI现阶段还需要预训练数据的投喂，所以没有相关的预处理数据，AI也只能胡扯。

所以总结下来，在真实项目中，AI还没有办法取代程序员。


### <span class="section-num">2.3</span> AI取代论背后的动机 {#ai取代论背后的动机}

既然你说AI现阶段还没有办法取代程序员，为什么我看到各种各样程序员要被AI取代的新闻，甚至有公司用AI来替代员工了？

除去真的担心自己被AI取代的从业者外，我觉得看到两种扯着AI大旗的人。

一种就是懂程序开发，装不懂程序开发，然后贩卖焦虑的人，最近非常火的就是，「我用Cursor从0 xx 天开发了一个App」 (Cursor是最近一个很火的AI编辑器，可以通过输入的指令生成代码).

大体都是说自己完全不懂程序开发，借助AI/Cursor, xx天内开发出了一个App, 并且冲到了苹果App Store热门榜第一.

普通程序员看到这种说法，一个小白从无到有都能开发出一个App, 难免会焦虑,不得不说，这种反向装弱再变强真的令人防不胜防， 想着不能被落下，就斥资加入各种付费群或者购买各种课程。

神智清明的你可能在心平气和下很难做出这样的决定嘛。

另外一种就是要用AI来取代程序的公司，说要通过AI来增效，裁撤工程师，比如国外的 [Workday 裁员说要再招人做 AI 的新闻](https://apnews.com/article/workday-layoffs-job-cuts-ai-investments-437581ad79d6e1cef2de7b300015dfbb?utm_source=t.me/mtfront)&nbsp;[^fn:2], 还有2023年 [Google 因为AI裁员而30000 名员工的新闻 ](https://www.bwpeople.in/article/google-contemplates-30000-layoffs-due-to-ai-job-impact-503670)[^fn:3]

怎么说呢，AI这块牌子太好用了, 什么都可以往里面套，裁撤工程师降本是真，AI增效大概是假。

裁员就裁员呗，为什么还要说是用AI来取代程序员呢？

因为说是AI取代了程序员，被毕业的员工们和待毕业的员工们都只会把裁员的原因归罪到AI上，
管理层的无所作为，墨守成规，决策失误，无序扩张这些失误都不需要承诺后果了，反正背锅有AI。

苦一苦员工，骂名AI来担。
![](/ox-hugo/ai_bear_the_infamy.jpg)


## <span class="section-num">3</span> AI一无用处 {#ai一无用处}

当然凡事都有例外，有人觉得AI会取代程序员，也有人觉得AI一无用处，不值一提。

我对此也是持反对态度了，虽然AI无法取代程序员，但是它的确在很多方面可以给我提升相当大的效率，其效率跃升堪比计算机连上互联网。

所以我也藉此来分享下我的AI用法


## <span class="section-num">4</span> 我的AI用法 {#我的ai用法}

我使用对AI的定位把它当作一位非常耐心，能力全面的助理，所以我给它分配的都是明确而繁琐的任务，并且我有能力对其输出进行评判的情景。
(也就是让它干脏活累活)

先声明一下我使用的AI 模型，日常事务我会使用ChatGPT, 响应速度快，量大够饱； 编码我会使用Claude 3.5 Sonnet，目前是我使用过效果最好的; 中文相关会使用DeepSeek, 但是经常服务器繁忙，体验不好。


### <span class="section-num">4.1</span> 生成测试用例 {#生成测试用例}

测试用例太适合让AI来写了，毕竟任务是非常明确的，需要被测的函数也是非常明确的，你只需要使用把测试的场景说清楚。

我觉得都可以把这个内容扩展写成 [测试技能进阶系列](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%80_%E8%BD%AF%E4%BB%B6%E8%B4%A8%E9%87%8F%E8%AE%A4%E7%9F%A5/) 的第四篇了

假设我需要测试一个计算字符串中元音数量的函数 `count_vowels(s: str) -> int`, 我的指示是:

> 请为以下函数 \`count_vowels(s: str) -&gt; int\` 生成pytest测试用例：
>
> 1.  正常输入包含大小写元音的情况
> 2.  空字符串边界条件
> 3.  全元音字符的特殊情况
> 4.  不含元音的字符串
> 5.  非法输入类型校验

AI生成结果：

```python
import pytest

def test_count_vowels_normal_cases():
    # 混合大小写元音测试
    assert count_vowels("Hello World") == 3
    assert count_vowels("AEiou") == 5

def test_count_vowels_edge_cases():
    # 空字符串/全元音测试
    assert count_vowels("") == 0
    assert count_vowels("aei") == 3

def test_count_vowels_special_cases():
    # 无元音字符串测试
    assert count_vowels("rhythm") == 0
    assert count_vowels("12345!@#") == 0

def test_count_vowels_invalid_input():
    # 类型校验测试
    with pytest.raises(TypeError):
        count_vowels(123)
    with pytest.raises(TypeError):
        count_vowels(["hello"])
```


### <span class="section-num">4.2</span> 语言学习 {#语言学习}

AI的多语言翻译功能对比其他传统基于单词的翻译软件简直是降维打击bb，毕竟Google的科学家最初做 Transform 模型就是为了做翻译的。

记得我之前考雅思的时候，口语和写作考前复习就是把题库的复制给AI，然后让他来给我答题，我就把思路背下来，做题这方面，AI实在是太擅长了。

> Given the following IELTS Speaking Part1, you are an examinee, could you give me some band 9 answer:
>
> Ielts Speaking 3 Practice 5 (Change)
>
> 1.  Do you think change is good?
> 2.  What are some of the major changes that occur to people throughout their lives?
> 3.  Is your country changing rapidly?
> 4.  In what ways have changes in technology changed people's lives?
> 5.  Why do old people not accept change?

<!--quoteend-->

> Here are some Band 9 answers for the IELTS Speaking Part 1 topic "Change", with a natural, fluent style and advanced vocabulary.
>
> 1.  Do you think change is good?
>
> Absolutely. Change is an inevitable part of life, and in most cases, it drives progress and personal growth. While change can be daunting at first, it often leads to new opportunities and valuable experiences. Without change, we would stagnate, both individually and as a society.
> ...

让AI做词典, 帮我解释和翻译我不懂的单词，或者做中译英方便交流:

> You are a professional translator. Translate all input Chinese paragraphs into English and all input English into Chinese, and help me explain its meaning

而后我输入中文，它就会给我英文翻译；输入英文，它会给我中文翻译:

> "爱哭的孩子有奶喝"
>
> ----
>
> "The crying baby gets the milk."
> Meaning:
>
> This Chinese proverb "爱哭的孩子有奶喝" (ài kū de háizi yǒu nǎi hē) means that those who complain, express their needs, or fight for their interests are more likely to get attention and benefits. It suggests that speaking up and advocating for oneself can lead to better outcomes, similar to how a baby who cries gets fed first.

<!--quoteend-->

> illiterate
>
> ----
>
> "Illiterate" 的中文翻译是 “文盲” 或 “不识字”。
> 含义：
>
> 1.  基本含义：指一个人 不识字或无法阅读和书写。
>     -   例如：Many people in the past were illiterate because they had no access to education.（过去，许多人因为无法接受教育而是文盲。）
>
> 2.  引申含义：在某个领域缺乏基本知识或技能。
>     -   例如："He is financially illiterate."（他在财务方面一窍不通。）
>     -   在科技、法律、文化等方面，也可以用 illiterate 来形容缺乏基础知识的人，例如 "technologically illiterate"（科技文盲）。

还有一个就是让AI 给我写的内容纠错，这个也比传统的语法工具类如 Grammarly 好用.

> You are a professional and seasoned translator, correct me if I am wrong.


### <span class="section-num">4.3</span> 代码审计 {#代码审计}

有时候我会查看一些智能合约的代码或者TG bot的代码，在实际交互前，我一般会把代码丢给AI，让它检查一下是否有可疑的代码片段，类如窃取用户的私钥，把资金转到可疑账户等等。

或者把我自己写的代码丢给AI，让它看下是否有安全漏洞:

> Given the following code, help me spot if there is any vulnerability here:

```python
@app.route('/search')
def search():
    query = request.args.get('q')
    cursor = db.execute(f"SELECT * FROM products WHERE name LIKE '%{query}%'")
    return render_template('results.html', data=cursor.fetchall())
```

> Yes, there are several serious security vulnerabilities in this code. Let me break them down:
>
> -   SQL Injection Vulnerability (Critical):
>
> The most severe issue is that the code uses string formatting (f-string) to directly insert user input into the SQL query. This allows SQL injection attacks. For example, a malicious user could input:
>
> Here's a secure version of the code:

```python
@app.route('/search')
def search():
    query = request.args.get('q')
    if query is None:
        return render_template('results.html', data=[])

    # Using parameterized query with placeholders
    cursor = db.execute(
        "SELECT * FROM products WHERE name LIKE ?",
        ('%' + query + '%',)
    )
    return render_template('results.html', data=cursor.fetchall())
```


### <span class="section-num">4.4</span> 反编译 {#反编译}

现代前端基本都会使用类似webpack 的打包工具进行打包压缩, 甚至还会进行代码混淆，所以F12 查看源码，并不能查出端倪.

因此我一般都是让AI 来帮我反编译和还原, 这样就免去了复杂的人工转译的过程:

> Given the obfuscated and compressed code, could you reverse it back to the original source code?

```javascript
var _0x3f2a = ['\x48\x65\x6c\x6c\x6f', '\x6c\x6f\x67'];
(function(_0x12d3f0, _0x3f2a8c) {
    var _0x553e8f = function(_0x5e7a28) {
        while (--_0x5e7a28) {
            _0x12d3f0['push'](_0x12d3f0['shift']());
        }
    };
    _0x553e8f(++_0x3f2a8c);
}(_0x3f2a, 0x1a4));
var _0x553e = function(_0x12d3f0, _0x3f2a8c) {
    _0x12d3f0 = _0x12d3f0 - 0x0;
    var _0x553e8f = _0x3f2a[_0x12d3f0];
    return _0x553e8f;
};
function _ax342() {
    var _0x5e7a28 = _0x553e('0x0');
    console[_0x553e('0x1')](_0x5e7a28 + ' World!');
}
_ax342();
```

> Let me help you deobfuscate this code. Based on the patterns and structure, this appears to be JavaScript code that has been obfuscated using common techniques like array-based string storage and variable name mangling.
> The original source code would be:

```javascript
function greet() {
    console.log("Hello World!");
}
greet();
```

着实有点东西


### <span class="section-num">4.5</span> 排查问题 {#排查问题}

在出现大模型AI之前，当遇到某个工具或者库的错误信息时，我一般的做法是：

1.  复制错误信息
2.  粘贴到Google
3.  点击排名最靠前的Stackoverflow 链接
4.  扫一下是否是我想问的问题，如果不是就再回到2
5.  按照最高赞的解决方案尝试下
6.  如果还不行，再回到2

有了AI之后，我一般的做法是:

1.  复制错误信息
2.  问AI "How to fix this erro", 粘贴上错误信息
3.  按照AI的建议一步步尝试
4.  如果不行，把最新的错误信息复制粘贴给AI, 告诉它这不管用.

与Google 的方案相比，我现在有了个再反馈的过程, 可以把最新的结果再告知AI, 这样更方便AI基于上下文做出解答.


### <span class="section-num">4.6</span> 数据处理 {#数据处理}

非结构化数据转换成结构化数据，如JSON, 相当于让AI做了数据清洗的脏活:

> Given the following output of ls command, could you convert it to JSON?

```bash

-> % ls -alrt
total 828
-rw-rw----+   1 ramsay ramsay   4672 Mar 21  2019 abbrev_defs
-rw-rw----+   1 ramsay ramsay  10079 Mar 21  2019 recentf~
drwxr-xr-x    2 ramsay ramsay   4096 Apr 16  2021 images
drwxrw----+  44 ramsay ramsay   4096 Apr 16  2021 snippets
-rw-rw----+   1 ramsay ramsay   3740 Dec 10  2021 projectile-bookmarks.eld
-rw-rw----+   1 ramsay ramsay 241501 May 27  2022 company-statistics-cache.el

```

> Here is the JSON representation of the given ls -alrt output:
> Explanation:
>
> -   Each file/directory is represented as an object inside the "files" array.
> -   "name" is the file/directory name.
> -   "permissions" is the file permissions.
> -   "links" is the number of hard links.
> -   "owner" is the owner of the file.
> -   "group" is the group the file belongs to.
> -   "size" is the size of the file in bytes.
> -   "modified" is the date of the last modification.

```json
     {
  "files": [
    {
      "name": "abbrev_defs",
      "permissions": "-rw-rw----+",
      "links": 1,
      "owner": "ramsay",
      "group": "ramsay",
      "size": 4672,
      "modified": "Mar 21 2019"
    },
    {
      "name": "recentf~",
      "permissions": "-rw-rw----+",
      "links": 1,
      "owner": "ramsay",
      "group": "ramsay",
      "size": 10079,
      "modified": "Mar 21 2019"
    },
    {
      "name": "images",
      "permissions": "drwxr-xr-x",
      "links": 2,
      "owner": "ramsay",
      "group": "ramsay",
      "size": 4096,
      "modified": "Apr 16 2021"
    },
    {
      "name": "snippets",
      "permissions": "drwxrw----+",
      "links": 44,
      "owner": "ramsay",
      "group": "ramsay",
      "size": 4096,
      "modified": "Apr 16 2021"
    },
    {
      "name": "projectile-bookmarks.eld",
      "permissions": "-rw-rw----+",
      "links": 1,
      "owner": "ramsay",
      "group": "ramsay",
      "size": 3740,
      "modified": "Dec 10 2021"
    },
    {
      "name": "company-statistics-cache.el",
      "permissions": "-rw-rw----+",
      "links": 1,
      "owner": "ramsay",
      "group": "ramsay",
      "size": 241501,
      "modified": "May 27 2022"
    }
  ]
}
```

对于非结构化的数据，或者缺失格式的数据, AI都能处理.


## <span class="section-num">5</span> 总结 {#总结}

这是我用即梦AI生成的一张街舞海报，我注明了地址，联系电话，以及价格，生成出来的结果是这样的

{{< figure src="/ox-hugo/genai_cover_image.jpg" >}}

远看是那么一回事！细看是什么回事?

AI现阶段还是无法取代程序员的，未来AI我也觉得很难取代程序员(当然，取代把需求翻译成代码的人肉翻译器还是有可能的)，因为程序员虽然是和机器打交道，但是本质上还是解决人的问题。

担忧程序员被AI取代的人可能只看到了测试场景里面的 happy path, 也就是AI能生成老板们想要的代码，没有看到unhappy path, 也就是老板们想要的代码, AI生成不出来要怎么办?

毕竟人可以根据已有的经验进行归纳总结，把已有的经验应用在未踏足的领域，且持续学习，但是AI还是受限于预训练数据，做不出来就是做不出来。

但是对程序员而言，AI不失为一个好用且提效的工具，好用的工具就应该多学多用多玩，而不是担忧被其取代。

AI无法取代持续精进者, 傲慢自满方为淘汰之源

(什么，你问我当初的答辩结果？自然是

---

挂了)


### 推荐阅读 {#推荐阅读}

-   旅加经历
    -   [这些年走过的路：从广州到温哥华](https://ramsayleung.github.io/zh/post/2023/%E8%BF%99%E4%BA%9B%E5%B9%B4%E8%B5%B0%E8%BF%87%E7%9A%84%E8%B7%AF_%E4%BB%8E%E5%B9%BF%E5%B7%9E%E5%88%B0%E6%B8%A9%E5%93%A5%E5%8D%8E/)
    -   [加拿大之初体验](https://ramsayleung.github.io/zh/post/2023/%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B9%8B%E5%88%9D%E4%BD%93%E9%AA%8C/)
    -   [登陆加拿大一年后的体会](https://ramsayleung.github.io/zh/post/2024/%E7%99%BB%E9%99%86%E5%8A%A0%E6%8B%BF%E5%A4%A7%E4%B8%80%E5%B9%B4%E7%9A%84%E4%BD%93%E4%BC%9A/)
    -   [夏日捕蟹记](https://ramsayleung.github.io/zh/post/2024/%E5%A4%8F%E6%97%A5%E6%8D%95%E8%9F%B9%E8%AE%B0/)
-   历史思考
    -   [为什么梦想买不起，故乡回不去](https://ramsayleung.github.io/zh/post/2023/%E7%BD%AE%E8%BA%AB%E4%BA%8B%E5%86%85/)
    -   [闲聊写作的好处](https://ramsayleung.github.io/zh/post/2023/%E9%97%B2%E8%81%8A%E5%86%99%E4%BD%9C/)
    -   [金榜题名之后](https://ramsayleung.github.io/zh/post/2024/%E9%87%91%E6%A6%9C%E9%A2%98%E5%90%8D%E4%B9%8B%E5%90%8E/)
-   工具流分享
    -   [简明写作指南](https://ramsayleung.github.io/zh/post/2024/%E7%AE%80%E6%98%8E%E5%86%99%E4%BD%9C%E6%8C%87%E5%8D%97/)
    -   [我的写作流](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E5%86%99%E4%BD%9C%E6%B5%81/)
    -   [我的画图流：画图工具与技巧分享](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E7%94%BB%E5%9B%BE%E6%B5%81/)
    -   [我的搜索流：高效搜索经验分享](https://ramsayleung.github.io/zh/post/2023/%E6%88%91%E7%9A%84%E6%90%9C%E7%B4%A2%E6%B5%81/)
    -   [最好的学习方式：费曼学习法(Feynman Technique)](https://ramsayleung.github.io/zh/post/2022/feynman_technique/)
    -   [系统思考：既见树木，又见森林](https://ramsayleung.github.io/zh/post/2021/%E7%B3%BB%E7%BB%9F%E6%80%9D%E8%80%83/)
-   职场思考
    -   [编程十年的感悟](https://ramsayleung.github.io/zh/post/2024/%E7%BC%96%E7%A8%8B%E5%8D%81%E5%B9%B4%E7%9A%84%E6%84%9F%E6%82%9F/)
    -   [那些年，我从微信支付学到的东西](https://ramsayleung.github.io/zh/post/2023/%E4%BB%8E%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%98%E7%A6%BB%E7%BA%BF_%E6%88%91%E5%B8%A6%E8%B5%B0%E4%BA%86%E4%BB%80%E4%B9%88/)
-   软件工程
    -   [测试技能进阶(一): 软件质量认知](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%80_%E8%BD%AF%E4%BB%B6%E8%B4%A8%E9%87%8F%E8%AE%A4%E7%9F%A5/)
    -   [测试技能进阶(二): Parameterized Tests](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/)
    -   [测试技能进阶(三): Property Based Testing](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%89_property_based_testing/)

<div center class="qr-container">

<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻

</div>

[^fn:1]: <https://www.sov2ex.com/?q=AI%E5%8F%96%E4%BB%A3%E7%A8%8B%E5%BA%8F%E5%91%98>
[^fn:2]: <https://apnews.com/article/workday-layoffs-job-cuts-ai-investments-437581ad79d6e1cef2de7b300015dfbb?utm_source=t.me/mtfront>
[^fn:3]: <https://www.bwpeople.in/article/google-contemplates-30000-layoffs-due-to-ai-job-impact-503670>
