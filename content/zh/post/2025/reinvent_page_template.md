+++
title = "重新造轮子系列(五)：模板引擎"
date = 2025-04-14T22:59:00-07:00
lastmod = 2025-04-16T23:10:01-07:00
tags = ["reinvent"]
categories = ["ReInvent: 重新造轮子系列"]
draft = false
toc = true
+++

项目 GitHub 地址: [Page Template](https://github.com/ramsayleung/reinvent/tree/master/page_templates)


## <span class="section-num">1</span> 前言 {#前言}

在现代网站开发里，内容与表现的分离已经成为基本准则(Separation of content and presentation),
比如 HTML 就是负责内容展现，而 CSS 就是负责页面的样式。

而手动更新和编写 HTML 也是一件费时费力并且容易出错的工作，尤其是需要同时修改多个页面的时候，
因此有聪明的程序员就发明了名为静态网页生成器(static site generator)的技术，可以按需生成网页。

事实上，互联网上的大多数页面都是通过某种形式的静态网页生成器生成出来的。

而静态网页生成器的核心就是「模板引擎」，在过去三十年，诞生过无数的模板引擎，
甚至有位加拿大的程序员为了更方便记录谁访问了他的简历，他还发明了一门编程语言来做模板引擎的活，这就是「世界上最好的编程语言：PHP」。

PHP 可以算是 Web时代的王者之一，凭借着 `LAMP(Linux, Apache, MySql, PHP)` 架构不断开疆扩土，攻城掠地，而PHP本身也不断有新的框架被造出来，为谁是最好的「模板引擎」打得头破血流。

虽然关于「模板引擎」的战争至今仍未停歇，但细分下来，「模板引擎」可以分成三个主要的流派：


### <span class="section-num">1.1</span> 嵌入式语法 {#嵌入式语法}

在 Markdown/HTML 这样的标识语言里面嵌入编程语言，使用 `<% %>` 等符号来标记代码与文本内容，其中的代表包括 Javascript 的 [EJS](https://ejs.co/), Ruby 的 [ERB](https://docs.ruby-lang.org/en/2.3.0/ERB.html), 以及 Python 的 [Jinja](https://jinja.palletsprojects.com/en/stable/):

```js
<!-- 用特殊标记混合JavaScript与HTML -->
<% if (user) { %>
  <h1><%= user.name %></h1>
<% } %>
```

其优点就是可以直接使用嵌入的编程语言，功能强大，学习成本低，缺点就是模板很容易变成混杂内容和逻辑的「屎山」代码


### <span class="section-num">1.2</span> 自定义语法 {#自定义语法}

不嵌入现成的编程语言，而是自己开发一套 mini 编程语言，或者叫 DSL(domain specifc language), 代表有 [GitHub Page](https://pages.github.com/) 用到的 [Jekyll](https://jekyllrb.com/), 还有 Golang 开发的著名静态网页生成器 [Hugo](https://gohugo.io/), 都是使用自定义的语法：

```golang
{% comment %} 自创模板语法 {% endcomment %}
{% for post in posts %}
  {{ post.title | truncate: 30 }}
{% endfor %}
```

优点就是语法简洁，缺点就是发展下去，可以又是自己造了一个新的编程语言，功能还不如通用的编程语言强大


### <span class="section-num">1.3</span> HTML指令 {#html指令}

不再在 HTML 中嵌入编程语言或DSL，取而代之的是直接给 HTML 定义特定的属性，不同的属性代表不同的含义，但是使用的还是标准 HTML.

最著名的就是 [Vuejs](https://vuejs.org/):

```html
<!-- 用特殊属性实现逻辑 -->
<div v-if="user">
  <h1>{{ user.name }}</h1>
</div>
```

优点是保持HTML的合法性与简洁，不需要额外的 parser, 缺点就是指令功能受限，不如内嵌编程语言强大，生态工具较少, 灵活性差。

本文的模板引擎就会以这个流派为范式进行开发。


### <span class="section-num">1.4</span> 特例之PHP {#特例之php}

分析完三种流派，就会奇怪 PHP 究竟是属于哪个流派呢？

```php
<h1><?php echo $title; ?></h1>
<ul>
  <?php foreach ($items as $item) { ?>
    <li><?php echo $item; ?></li>
  <?php } ?>
</ul>
```

其实 PHP 本质就是流派二，只是这门专门用于「模板引擎」的 mini 语言，最后演化成了一门专门的编程语言，只是这个编程语言最擅长的还是网页开发，即是做「模板引擎」。

所以 PHP 是从流派二演化成流派一。


## <span class="section-num">2</span> 目标 {#目标}

可能不是所有的朋友都了解 Vue，所以在设计我们的模板引擎之前，先来明确一下需求与目标(scope).

假设我们有如下的 JSON 数据:

```js
{
    names: ['Johnson', 'Vaughan', 'Jackson']
}
```

如果有如下的模板:

```html
<html>
  <body>
    <p>Expect three items</p>
    <ul z-loop="item:names">
      <li><span z-var="item"/></li>
    </ul>
  </body>
</html>
```

那么 `names` 就会被赋值给 `item`, 然后每一个变量都会被展开成 `<span>{item}</span>`, 所以上面的模板就会被展开成:

```html
<html>
  <body>
    <p>Expect three items</p>
    <ul>
      <li><span>Johnson</span></li>
      <li><span>Vaughan</span></li>
      <li><span>Jackson</span></li>
    </ul>
  </body>
</html>
```

而不同的指令会有不同的效果，如上的 `z-loop` 就是遍历一个数组，而 `z-if` 就是判断一个变量是否为 `true`, 为 `true` 则输出，否则则不输出.

如有数据:

```js
{
    "showThis": true,
    "doNotShowThis": false
}
```

和模板:

```html
<html>
  <body>
    <p z-if="showThis">This should be shown.</p>
    <p z-if="doNotShowThis">This should <em>not</em> be shown.</p>
  </body>
</html>
```

就会被渲染成:

```html
<html>
  <body>
    <p>This should be shown.</p>
  </body>
</html>
```

我们可以先支持以下的指令集:

| 指令集   | 含义                         |
|-------|----------------------------|
| `z-loop` | 遍历一个数组的数据，每个数据元素对应一个 HTML 元素 |
| `z-if`   | 在变量为 `True` 时显示 HTML 元素 |
| `z-var`  | 将变量替换成对应的值，并显示在 HTML 元素上 |
| `z-num`  | 将数字显示在对应的 HTML 元素上 |


## <span class="section-num">3</span> 设计思路 {#设计思路}


### <span class="section-num">3.1</span> stack frame {#stack-frame}

模板引擎的核心是将「数据」+「模板」渲染成页面，那么数据要如何保存呢？以什么数据结构和变量形式来处理呢？

最简单的方式肯定就是使用全局变量的 HashMap 来保存所有的变量，但是如果存在两个同名的变量，那么 HashMap 这种数据结构就不适用。

更何况，可变的全局变量可谓是万恶之源，不知道有多少 bug 都是源自可变的全局变量。

在编译原理，保存变量的标准做法就是使用 stack frame, 每次进入一个函数就创建一个新的栈(`stack`), 每次函数调用都有自己的独立的栈，可以理解成每个栈就是一个 `HashMap`, 而每创建一个栈就是向 `List` 里面 `push` 一个新的 `HashMap`, 同一个函数里面不能有同名的变量，那能保证栈里面的值是唯一。

{{< figure src="/ox-hugo/reinvent_stack_frame.jpg" >}}

谈及变量和 stack frame, 编程语言中有个 `作用域(scoping)` 的概念, 定义了变量会怎么被程序访问到。

主要有两种作用域，分别被称为：

词法作用域([Lexical/Static Scoping](https://en.wikipedia.org/wiki/Scope_(computer_science))): 在编译时就将变量给解析确定了下来，大部分编程语言使用的都是语法作用域，比如 Javascript, C/C++, Rust, Golang, Swift, Java 这个名单还可以很长.

因为其性能更优，并且行为是相当明确的，不需要分析运行时代码再来确定，如：

```javascript
let x = 10; // 全局变量

function foo() {
  console.log(x); // 词法作用域，问题绑定全局变量 x
}

function bar() {
  let x = 20; // 局部变量，不会影响 foo 中的 x
  foo(); // 调用 foo(), 仍然需要访问全局变量
}

foo(); // 输出: 10 (全局变量)
bar(); // 输出: 10 (还是全局变量，而非局部变量)
```

另外一种作用域是动态作用域(Dynamic Scoping): 在运行时通过遍历调用栈来确定变量的值，现在已经很少有编程语言使用了，比如是 Perl4, Bash, 或者是 Emacs Lisp:

```bash
#!/bin/bash

x="global"

foo() {
  echo "$x"  # x 的值取决于谁来调用 `foo`, 运行时决定
}

bar() {
  local x="local"  # 动态作用域: 会影响 foo 的值
  foo
}

foo  # 输出: "global" (x 是全局变量)
bar  # 输出: "local"  (x 是 bar 函数的局部变量)
```

也就是 `foo` 中 `x` 的值还取决于调用方的栈，因为在 `bar` 里面调用 `foo` 时， bash 解释器会把 `bar` 的栈一并传给 `foo`, 所以 `foo` 就以最近栈中 `x` 的值为准。

这种作用域实现方式虽然简单，但是对于程序员 debug 来说简直是噩梦，所以在现代编程语言基本绝迹了。

话虽如此，但是对于模板引擎而言，动态作用域却是主流选择，主要是因为：

1.  模板的特性需求：循环/条件语句需要运行时创建临时变量
2.  隔离性要求：避免不同模板间的变量污染
3.  异常处理：未定义变量可返回 `undefined/null` 而非报错

因此我们的模板引擎也会使用动态作用域来保存变量，即 `List<HashMap<String, String>>` 的数据结构.


### <span class="section-num">3.2</span> vistor pattern {#vistor-pattern}

确定好如何保存变量之后，下一个问题就是如何遍历并且生成模板。

解析HTML之后生成的是 DOM(Document Object Model) 结构, 本质是多叉树遍历，按照指令处理栈的变量，然后再把 HTML 输出, 如下:

```javascript
function traverse(node) {
  if (node.type === 'text') console.log(node.data);
  else {
    console.log(`<${node.name}>`);
    node.children.forEach(traverse);
    console.log(`</${node.name}>`);
  }
}
```

实现是很简单，但是我们把「遍历逻辑」和「不同指令对应的逻辑」耦合在一起了，很难维护。

并且我们现在只支持4个指令，或者未来要增加其他指令，只要在 `traverse` 里面再增加 if-else 逻辑，基本没有扩展性。

所以我们需要优化的点就是，把「遍历逻辑」和「指令逻辑」分开，这样就易于我们扩展新指令。

要解耦，想想有啥设计模式合适，遍寻23种设计模式，[访问者(Vistor)模式 ](https://refactoring.guru/design-patterns/visitor)就很合适用来做解耦遍历逻辑和指令逻辑.

{{< figure src="/ox-hugo/reinvent_vistor_pattern.jpg" >}}

不了解 Vistor 模式的同学可以先看下这篇[文章](https://refactoring.guru/design-patterns/visitor), 而Rust 非常著名的序列化框架 [Serde](https://serde.rs) 就通过 [Vistor](https://serde.rs/impl-deserialize.html) 模式可以让用户自定义如何序列化或反序列化某种类型的数据。


### <span class="section-num">3.3</span> 接口设计 {#接口设计}

既然选定了 `Vistor` 模式，那么就让我们来设计具体的接口。

`Vistor` 接口类，接受某个 DOM 元素作为根节点，然后通过 `walk` 函数遍历给定的节点，或者节点为空则遍历根节点:

```js
import { Node, NodeWithChildren } from "domhandler"
export abstract class Visitor {
  private root: Node;
  constructor(root: Node) {
    this.root = root;
  }

  walk(node: Node = null) {
    if (node === null) {
      node = this.root
    }

    if (this.open(node)) {
      node.children.forEach(child => {
        this.walk(child)
    });
    }
    this.close(node);
  }

  // handler to be called when first arrive at a node
  abstract open(node: Node): boolean;

  // handler to be called when finished with a node
  abstract close(node: Node): void;
}
```

其中的 `open` 函数用于在进入一个节点时被调用，相当于是在前序位置被调用，返回值来表现是否需要遍历其子节点；而 `close` 函数在离开一个节点前，即相当于后序位置被调用。

关于二叉树的前序位置和后序位置，可见这篇讲解二叉树算法的[文章](https://labuladong.online/algo/essential-technique/binary-tree-summary/#%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E9%87%8D%E8%A6%81%E6%80%A7)

`Vistor` 算法里面的关键即是实现「遍历逻辑」与「每个节点处理逻辑」的解耦，遍历逻辑我们已经实现在 `Vistor` 基类了，现在就需要实现一个具体的子类来表示节点的处理逻辑:

```js
export enum HandlerType {
  If = 'z-if',
  Loop = 'z-loop',
  Num = 'z-num',
  Var = 'z-var',
}

const HANDLERS: Record<HandlerType, NodeHandler> = {
  [HandlerType.If]: new IfHandler(),
  [HandlerType.Loop]: new LoopHandler(),
  [HandlerType.Num]: new NumHandler(),
  [HandlerType.Var]: new VarHandler(),
}

export class Expander extends Visitor {
  public env: Env;
  private handlers: Record<HandlerType, NodeHandler>
  private result: string[]
  constructor(root: Node, vars: Object) {
    super(root);
    this.env = new Env(vars);
    this.handlers = HANDLERS;
    this.result = [];
  }

  open(node: Node): boolean {
    if (node.type === 'text') {
      const textNode = node as Text;
      this.output(textNode.data);
      return false;
    } else if (this.hasHandler(node as Element)) {
      return this.getHandler(node as Element).open(this, node);
    } else {
      this.showTag(node as Element, false);
      return true;
    }
  }

  close(node: Node): boolean {
    if (node.type === 'text') {
      return;
    }
    if (node.type === 'tag' && this.hasHandler(node as Element)) {
      this.getHandler(node as Element).close(this, node);
    } else {
      this.showTag(node as Element, true);
    }
  }

  // 判断是否有 z-* 属性对应的指令处理器
  hasHandler(node: Element): boolean {
    for (const name in node.attribs) {
      if (name in this.handlers) {
        return true;
      }
    }
    return false;
  }

  getHandler(node: Element) {
    const possible = Object.keys(node.attribs)
      .filter(name => name in this.handlers)
    assert(possible.length === 1, 'Should be exactly one handler');
    return this.handlers[possible[0]];
  }

  // 将 tag 标签及属性输出到 output 去，但排除 `z-` 开头的指令
  showTag(node: Element, closing: boolean) {}

  output(text: string) {
    this.result.push((text === undefined) ? 'UNDEF' : text);
  }
```

`Expander` 的逻辑也并不复杂，每次遍历到一个 `DOM` 元素的时候，通过元素类似执行对应的操作，如果是 `z-` 开头的指令，就看下能否找到对应指令的处理器:

{{< figure src="/ox-hugo/reinvent_expander_design.png" >}}

仔细观察代码会发现，不同的指令对应的处理器实现了 `NodeHandler` 接口，定义在前序位置和后序位置处理节点的逻辑，并按指令名保存在 `HANDLER` 中：

```js
export interface NodeHandler {
  open(expander: Expander, node: Element): boolean;
  close(expander: Expander, node: Element): void;
}
```

这就意味着，如果需要增加一个新的指令，该指令处理器只需要实现 `NodeHandler` 接口，并添加到 `HANDLER` 即可，不需要改动其他的已有代码，我们就实现了「遍历逻辑」与「指令逻辑」的解耦。


## <span class="section-num">4</span> 实现 {#实现}


### <span class="section-num">4.1</span> 支持的指令集 {#支持的指令集}

不同的指令集的差别只是如何实现 `open` 和 `close` 逻辑，我就不一一赘述了，已支持的指令集及实现列表如下：

| 指令          | 实现                                                                                                |
|-------------|---------------------------------------------------------------------------------------------------|
| `z-if`        | [z-if.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-if.ts)               |
| `z-include`   | [z-include.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-include.ts)     |
| `z-iteration` | [z-iteration.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-iteration.ts) |
| `z-literal`   | [z-literal.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-literal.ts)     |
| `z-loop`      | [z-loop.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-loop.ts)           |
| `z-num`       | [z-num.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-num.ts)             |
| `z-snippet`   | [z-snippet.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-snippet.ts)     |
| `z-trace`     | [z-trace.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-trace.ts)         |
| `z-var`       | [z-var.ts](https://github.com/ramsayleung/reinvent/blob/master/page_templates/z-var.ts)             |


### <span class="section-num">4.2</span> 示例 {#示例}

假设有数据如下:

```js
const vars = {
    "firstVariable": "firstValue",
    "secondVariable": "secondValue",
    "variableName": "variableValue",
    "showThis": true,
    "doNotShowThis": false,
    "names": ["Johnson", "Vaughan", "Jackson"]
};
```


#### <span class="section-num">4.2.1</span> z-num {#z-num}

```html
<html>
  <body>
    <p><span z-num="123"/></p>
  </body>
</html>
```

模板展开如下：

```html
<html>
  <body>
    <p><span>123</span></p>
  </body>
</html>
```


#### <span class="section-num">4.2.2</span> z-var {#z-var}

```html
<html>
  <body>
    <p><span z-var="variableName"/></p>
  </body>
</html>
```

模板展开如下：

```html
<html>
  <body>
    <p><span>variableValue</span></p>
  </body>
</html>
```


#### <span class="section-num">4.2.3</span> z-if {#z-if}

```html
<html>
  <body>
    <p z-if="showThis">This should be shown.</p>
    <p z-if="doNotShowThis">This should <em>not</em> be shown.</p>
  </body>
</html>
```

模板展开如下：

```html
<html>
  <body>
    <p>This should be shown.</p>

  </body>
</html>
```


#### <span class="section-num">4.2.4</span> z-loop {#z-loop}

```html
<html>
  <body>
    <p>Expect three items</p>
    <ul z-loop="item:names">
      <li><span z-var="item"/></li>
    </ul>
  </body>
</html>
```

模板展开如下：

```html
<html>
  <body>
    <p>Expect three items</p>
    <ul>
      <li><span>Johnson</span></li>

      <li><span>Vaughan</span></li>

      <li><span>Jackson</span></li>
    </ul>
  </body>
</html>
```


#### <span class="section-num">4.2.5</span> z-include {#z-include}

```html
<html>
  <body>
    <p><span z-var="variableName"/></p>
    <div z-include="simple.html"></div>
  </body>
</html>
```

`simple.html` :

```html
<div>
  <p>First</p>
  <p>Second</p>
</div>
```

模板展开如下：

```html
<html>
  <body>
    <p><span>variableValue</span></p>
    <div>
  <p>First</p>
  <p>Second</p>
</div>
  </body>
  </html>
```

更多的[示例可见](https://github.com/ramsayleung/reinvent/blob/master/__tests__/page_template/expander-test.ts)


## <span class="section-num">5</span> 总结 {#总结}

模板引擎的本质，是帮我们把重复的页面结构抽离出来，而内容与表现的分离(Separation of content and presentation)，可以让我们以数据来填充变化的内容。

这是程序员对「Don't Repeat Yourself」原则最直观的践行。

三十年来，开发者们创造了无数种实现方案，但核心思路始终围绕着前文提到的三种基本模式。

如今即便在最流行的 Vue 或 React 框架中，无论你写的是 `JSX` 或是 `v-*` 指令，背后的思路仍万变不离其宗，本质上仍在沿用模板引擎的思想。

而这种「结构复用，数据驱动」的理念，也早已成为Web开发的根基。

[回到本系列的目录]({{< relref "reinvent_project" >}})


## <span class="section-num">6</span> 参考 {#参考}

-   <https://refactoring.guru/design-patterns/visitor>
-   <https://serde.rs/impl-deserialize.html>
-   <https://third-bit.com/sdxjs/page-templates/>
