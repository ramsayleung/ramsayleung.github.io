+++
title = "重新造轮子系列(三): HTML Selector"
date = 2025-03-15T10:53:00-07:00
lastmod = 2025-04-04T00:16:01-07:00
tags = ["reinvent"]
categories = ["ReInvent: 重新造轮子系列"]
draft = false
toc = true
showQuote = true
+++

项目 GitHub 地址: [Selector](https://github.com/ramsayleung/reinvent/tree/master/html_selector)&nbsp;[^fn:1]


## <span class="section-num">1</span> 前言 {#前言}

以前写爬虫的时候，必不可少的一个工具就是 HTML selector, 就是用于匹配指定的 HTML 标签。

毕竟爬虫的本质就是找出需要的标签里面的内容，然后解析出来。

而 selector 主要有两个流派，一个是 [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors)&nbsp;[^fn:2], 另外一个是 [XPath selector](https://developer.mozilla.org/en-US/docs/Web/XML/XPath/Guides)&nbsp;[^fn:3] ,本质都是通过某种语法来匹配指定的标签，区别只是一个用的是 CSS 的语法，另外一个是 XML 语法.

这次我们就来写个基于 CSS 语法的 Selector, 来深入理解下 HTML 的 DOM 模型


## <span class="section-num">2</span> DOM {#dom}

写过前端的朋友应该都知道，前端代码主要是由所谓的三剑客组成的：HTML + CSS + JavaScript, 其中的三剑客各司其职，相互配合。

HTML 负责内容展示, CSS 负责布局和样式，而 JavaScript 是负责用户与页面之间的动态交互。

而对于如下的 HTML 代码：

```html
<html>
  <head>
    <title>Example</title>
  </head>
  <body>
    <h1>Title</h1>
    <blockquote id="important">
      <p>Opening</p>
      <p>Explanation</p>
      <p class="highlight">Warning</p>
    </blockquote>
    <p>Closing</p>
  </body>
</html>
```

浏览器会将其进行解析，并生成名为 [Document Object Model](//developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)(DOM) 的数据结构，听着好像很玄乎，但本质就是一棵多叉树 (Multiway Tree):

{{< figure src="/ox-hugo/reinvent_dom_tree.jpg" >}}

知道 `DOM` 是多叉树, 我们就可以写出简化版本 `DOM` 的数据结构了：

```javascript
export interface DomNode {
    type: string;
    name?: string;
    attribs?: {
        id?: string;
        class?: string;
        [key: string]: string | undefined;
    };
    children?: DomNode[];
    data?: string;
    parent?: DomNode;
}
```

一个节点可能有多个子节点 `(children?)` 或者一个父节点 `(parent?)`, 也可能都没有，所以标记成 `?(optional)`;

一个节点可能有多个属性 `attribs`.

而节点的=type= 可能是 `tag`, `text`, `comment`, `script`, `style`, 而对于 `text` 和 `comment` 类型的节点， `name` 也是为空的.

这个 `DOM` 结构只是我们的简化版本，完整的 DOM 还有很多的属性和回调函数，详情可以查看文档： [Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)


## <span class="section-num">3</span> BFS vs DFS {#bfs-vs-dfs}

理解到 `DOM` 的本质是个多叉树之后，我们很快就能意识到， `selector` 本质也就是遍历多叉树，找到符合要求的所有节点, 比如按 `tag` 名来匹配，按 `id` 来匹配，按 `class` 来匹配等等。

而用于遍历多叉树的常用算法就是广度优先搜索(Breadth First Search, BFS)和深度优先搜索(Depth First Search, DFS)

{{< figure src="/ox-hugo/reinvent_dfs_vs_bfs.jpg" >}}

通常来说，BFS 和 DFS 都能完成多叉树遍历，时间复杂度也是相同的，BFS通常使用一个 `queue` 来记录遍历待节点，所以会使用更多的内存，但是能找到最短路径；而 DFS 通常使用递归，如果遇到个循环图，就会 StackOverflow，无法找到结果。

因为我们明确知道 DOM 是个多叉树（有向无环图），所以我们就使用 DFS 来遍历查找。


## <span class="section-num">4</span> Strategy 设计模式 {#strategy-设计模式}

分析好问题之后，我们的实现也差不多能出来了, 按 tag 名来匹配，无非是 `domNode.name === tagName`; 按 `class` 来匹配, 即 `domNode.attribs.class=== class`.

为了解耦和易于扩展，我们可以使用个策略设计模式([Strategy Design Pattern](https://refactoring.guru/design-patterns/strategy)&nbsp;[^fn:4]).

```js
interface Selector {
    match(node: DomNode): boolean;
}

const findByTagName = (tag: string): Selector => ({
    match: (node: DomNode): boolean => {
        return node.name.toLowerCase() === tag.toLowerCase()
    }
});

const findById = (id: string): Selector => ({
    match: (node: DomNode): boolean => {
        return node.attribs.id === id;
    }
})

const findByClass = (clazz: string): Selector => ({
    match: (node: DomNode): boolean => {
        return node.attribs.class === clazz;
    }
});
```

然后遍历节点的时候，只需要判断 `Selector` 是否符合要求，而具体的匹配条件则由 `selector` 决定:

```js
const isMatch = (node: DomNode, selectors: Selector[]): boolean => {
    return selectors.every(selector => selector.match(node));
}
```

这样的话，要增加一个根据属性keyValue值的匹配条件也是非常容易的, 如 `div[align=center]`, 即匹配属性 `align` 和value 为 `center`:

```js
const findByAttributes = (key: string, value: string): Selector => ({
    match: (node: DomNode): boolean => {
        return node.attribs[key] === value;
    }
})
```


## <span class="section-num">5</span> 测试验证 {#测试验证}

DFS + Strategy design pattern 就实现了一个基础的 CSS Selector, 我们自然需要测试验证下是否正确:

```js
describe('HTML selector testsuite', () => {
    const HTML = `<main>
  <p>text of first p</p>
  <p id="id-01">text of p#id-01</p>
  <p id="id-02">text of p#id-02</p>
  <p class="class-03">text of p.class-03</p>
  <div>
    <p>text of div / p</p>
    <p id="id-04">text of div / p#id-04</p>
    <p class="class-05">text of div / p.class-05</p>
    <p class="class-06">should not be found</p>
  </div>
  <div id="id-07">
    <p>text of div#id-07 / p</p>
    <p class="class-06">text of div#id-07 / p.class-06</p>
  </div>
</main>`

    it.each([
        ['p', 'text of first p'],
        ['p#id-01', 'text of p#id-01'],
        ['p#id-02', 'text of p#id-02'],
        ['p.class-03', 'text of p.class-03'],
        ['div p', 'text of div / p'],
        ['div p#id-04', 'text of div / p#id-04'],
        ['div p.class-05', 'text of div / p.class-05'],
        ['div#id-07 p', 'text of div#id-07 / p'],
        ['div#id-07 p.class-06', 'text of div#id-07 / p.class-06']
    ])('test select %s %s', async (selector, expected) => {
        const doc = htmlparser2.parseDOM(HTML)[0];
        const node = select(doc, selector);
        const actual = getText(node);
        expect(actual).toBe(expected);
    })
})
```

使用 Jest 框架编写了如上的单元测试用例， unit test 都通过了，完工.

顺便一提的是，这种相同的验证逻辑, 但是输入多个不同的参数以验证不同case的做法，叫做 `Parameterized Test`

我在《[测试技能进阶系列](https://ramsayleung.github.io/zh/categories/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6/)》的第二篇也曾经介绍过： [Parameterized Tests](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/)


## <span class="section-num">6</span> 总结 {#总结}

这个简单的 CSS Selector 全部代码仅有 **103** 行, 但麻雀虽小，五脏俱全，功能还是齐备的:

```sh
> tokei simple-selectors.ts
===============================================================================
Language            Files        Lines         Code     Comments       Blanks
===============================================================================
TypeScript              1          131          103            9           19
===============================================================================
Total                   1          131          103            9           19
===============================================================================
```

所以标题也可以修改成 100 行代码实现一个简单的 CSS Selector :)

如果细看实现，还是有不少的优化之处的，比如 `parseSelector` 函数可以实现得更优雅些，以便进一步扩展支持其他的语法。

另外就是目前支持的都是所有 selector 完全匹配的情况，即 `and`, 但是目前不支持 `or` 的功能，即类如: `h1,h2,h3`, 可以匹配 `h1`, `h2`, 或者 `h3`.

---

如果想要看下较完整版本的 CSS Selector, 可以看下我六年多前我用 C++ 实现的[版本](https://github.com/ramsayleung/crawler), 实现从字符串解析并生成 `DOM`, 再实现 CSS 解析器，纯正的 OOP 风味。

当时初学 C++, 这个算是我早期写得比较大的 C++17 项目，核心代码大概1000行，还有几百行的单元测试。

现在再翻看自己的代码，会惊讶于当时自己代码写的工整，可谓是有板有眼，像极了书法初学者写的楷书。

> <span class="org-target" id="org-target--Unix----"></span>这本砖头书读过, 其他的C++书籍, 如<span class="org-target" id="org-target--C---Primer"></span>, <span class="org-target" id="org-target--Effective-C--"></span>, <span class="org-target" id="org-target--Modern-C--"></span>也读过, 感觉不把书中的内容实践下, 很容易遗忘。
>
> 但是日常的工作内容并不会涉及底层网络服务, 一切底层细节内容都被框架给包掉了, 开发的主力语言是Java, 也不会使用到C++.
>
> 因此决定创造个机会实践下这些知识，最终决定只用C/C++内置函数库实现。

的确所有工具都是用C/C++内置函数库实现的，甚至测试框架还是自己用宏实现的.

只是我未曾想到的是，写了这段话后不足一年，C++就成为了我下一家公司干活的主力语言; 而现在，我又在重新写 Java, 着实是「白衣苍狗」。

[回到本系列的目录]({{< relref "reinvent_project" >}})

[^fn:1]: <https://github.com/ramsayleung/reinvent/tree/master/html_selector>
[^fn:2]: <https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors>
[^fn:3]: <https://developer.mozilla.org/en-US/docs/Web/XML/XPath/Guides>
[^fn:4]: <https://refactoring.guru/design-patterns/strategy>
