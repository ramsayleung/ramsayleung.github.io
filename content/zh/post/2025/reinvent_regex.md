+++
title = "重新造轮子系列(四)：正则表达式引擎"
date = 2025-03-15T11:01:00-07:00
lastmod = 2025-04-04T00:13:14-07:00
tags = ["reinvent"]
categories = ["ReInvent: 重新造轮子系列"]
draft = false
toc = true
showQuote = true
+++

项目 GitHub 地址: [Regex](https://github.com/ramsayleung/reinvent/tree/master/regular_expression)


## <span class="section-num">1</span> 前言 {#前言}

所谓的正则表达式，指的是由一系列字符和特殊字符组成的模式，用于描述要匹配的文本。

最开始是一位叫 [Stephen Cole Kleene](https://en.wikipedia.org/wiki/Stephen_Cole_Kleene) 的数学家用被他称为 Regular Events 的数学表达式来描述这一模型，在 1968 年，由C语言之父 Ken Tompson 将这个表达式引入到行编辑器 [QED](https://en.wikipedia.org/wiki/QED_(text_editor)), 随后是 Unix 上的编辑器 [ed](https://en.wikipedia.org/wiki/Ed_(software)) (vi 的前身) ，并最终引入到 [grep](https://en.wikipedia.org/wiki/Grep).

我一直很好奇正则表达式 (regular expression, 即 `Regex` ) 是怎么实现的，自正则表达式被引入编程语言之后 之后，可谓说有字符串的地方就基本有正则表达式。

想起个关于 `Regex` 的经典笑话:

> 程序员A：我有个问题，想用正则表达式解决。
>
> 程序员B：现在你有两个问题了。


## <span class="section-num">2</span> 需求 {#需求}

完整版本的正则表达式非常复杂，我们的实现不会覆盖所有的规则，所以先来看下我们要支持的正则表达式规则：

| 含义       | 字符 |
|----------|----|
| 任意的字符 `c` | `c` |
| 任意的单个字符 | `.` |
| 匹配开头的字符 | `^` |
| 匹配结尾的字符 | `$` |
| 匹配零个或多个的字符 | `*` |

虽然这五条原则看起来不是很多，但是已经覆盖日常开发绝大多数的场景了。

比如 `^ab*c` 就意味着匹配以 `a` 开头，并且0到无数个的 `b`, 再接一个字符 `c`, 所以它能匹配:
`ac`, `abc` 以及 `abbbbbc`


## <span class="section-num">3</span> 初始版本 {#初始版本}

根据上面的需求，可以使用40行不到的代码就实现一个简单的递归版本的正则表达式引擎：

```js
export const match = (pattern: string, text: string): boolean => {
  // '^' at start of pattern matches start of next.
  if (pattern[0] === '^') {
    return matchHere(pattern, 1, text, 0);
  }

  // Try all possible starting points for pattern.
  let iText = 0;
  do {
    if (matchHere(pattern, 0, text, iText)) {
      return true;
    }
    iText += 1;
  } while (iText < text.length);

  // Nothing worked.
  return false;
}

const matchHere = (pattern: string, patternIndex: number, text: string, textIndex: number) => {
  // There is no more pattern to match.
  if (patternIndex === pattern.length) {
    return true;
  }

  // '$' at end of pattern matches end of text.
  if ((patternIndex === (pattern.length - 1)) && (pattern[patternIndex] === '$') && (textIndex === text.length)) {
    return true;
  }

  // '*' following current character means zero or more.
  if (((pattern.length - patternIndex) > 1) && (pattern[patternIndex + 1] === '*')) {
    // Try matching zero occurences(skip the current char and the '*')
    if (matchHere(pattern, patternIndex + 2, text, textIndex)) {
      return true;
    }

    // Try matching one or more occurences
    while ((textIndex < text.length) && (pattern[patternIndex] === '.' || text[textIndex] === pattern[patternIndex])) {
      // Try to match the rest of pattern after consuming this
      // character
      if (matchHere(pattern, patternIndex + 2, text, textIndex + 1)) {
        return true;
      }
      textIndex += 1;
    }
    // if there is any match, it will return early in the while loop,
    // so when reach this statement, it means nothing found.
    return false;
  }

  // Match a single chacater.
  if (textIndex < text.length && (pattern[patternIndex] === '.') || (pattern[patternIndex] === text[textIndex])) {
    return matchHere(pattern, patternIndex + 1, text, textIndex + 1);
  }

  // Nothing worked.
  return false;
}
```

实现思路如下图:

{{< figure src="/ox-hugo/reinvent_simple_regex_design.png" >}}

好的，我们的正则表达式引擎完工了，正则表达式看起来也没有那么难嘛。

只是用是能用的，但是看起来不同含义的字符都耦合在 `matchHere` 函数了，想要支持新的字符匹配(例如 `+`, 或者 `|` )很难扩展。


## <span class="section-num">4</span> 面向对象版本 {#面向对象版本}


### <span class="section-num">4.1</span> 接口 {#接口}

再来思考一下版本1的问题:

我们把不同模式的符号都耦合在同一个函数中。

在讨论解耦方式之前，先来观察下每个模式的共同点，以便我们抽象接口。

以最简单的 `^c` 模式为例，我们需要将 `c` 与给定的文本 `abc` 和 `cde` 作比较，首先匹配第一个字符，如果匹配失败(如 `abc`)，则直接结束； 如果匹配第一个字符成功（=cde=）, 那么就匹配剩余的其他字符, 直到模式匹配结束.

那么对于精确匹配字符的模式 `Literal` 而言，入参就是字符 `c` 和文本 `text`, 返回结果就是true/false, 用来表示是否匹配成功.

```javascript
const literal_match = (pattern: string, text: string): boolean => {}
```

如果不同的模式匹配都使用这个函数签名的话，每次匹配之后，都需要把剩下需要匹配的文本给复制出来，频繁拷贝字符串可能会导致性能开销很大。

我们可以做个小优化, 通过下标 `start` 来指定需要匹配的文本, 就可以在不同的模式中都只使用同一份的字符串，避免了多次拷贝的开销。

而返回结果也不再是 boolean, 而是下一个模式需要匹配的下标。

比如 `^c` 来匹配 `cde` ，匹配成功之后就返回 `1`, 就意味着下个模式从 `1`, 也就是 `d` 开始匹配.

那匹配失败要怎么表示？这个也很简单，返回一个不合法的下标，比如 `-1` 即可，那么我们的模式的函数接口就变成:

```js
const literal_match = (pattern: string, text: string, index: number): number => {}
```


### <span class="section-num">4.2</span> 模板设计模式 {#模板设计模式}

既然版本一提到了 `matchHere` 实现耦合在一起，那么有什么方式可以实现解耦呢？

其中的一个经典解决方式就是面向对象编程(Object Oriented Programming)，这也是面向对象编程的设计初衷。

既然前面实现的缺点是不同的模式耦合在一起，那么我们可以把每种模式实现成一个函数或者一个类，然后再通过某种模式给组合起来。

既然用到 OOP, 那么自然少不了设计模式了。如果使用一种模式表示成一个类，那么会是哪种设计模式呢？

要不就是[策略模式(strategy)](https://refactoring.guru/design-patterns/strategy):

```c++
class ConcreteAlgorithm : IAlgorithm
{
    void DoAlgorithm(int datum) {...}
}

class Strategy
{
    Strategy(IAlgorithm algo) {...}

    void run(int datum) { this->algo.DoAlgorithm(datum); }
}
```

要么就是[模板方法(template method)](https://refactoring.guru/design-patterns/template-method):

```c++
class ConcreteAlgorithm : AbstractTemplate
{
    void DoAlgorithm(int datum) {...}
}

class AbstractTemplate
{
    void run(int datum) { DoAlgorithm(datum); }

    virtual void DoAlgorithm() = 0; // abstract
}
```

看起来好像都可以，那不如就使用模板方式吧。


### <span class="section-num">4.3</span> 单向链表 {#单向链表}

那么就让我们来定义个基类 `RegexBase` :

```js
export const INVALID_INDEX = -1;
export abstract class RegexBase {
  // index to continue matching at or -1 indicating that matching failed
  abstract _match(text: string, start: number): number;
  abstract rest: RegexBase;

  match(text: string): boolean {
    // check if the pattern matches at the start of the string
    if (this._match(text, 0) !== INVALID_INDEX) {
      return true;
    }
    for (let i = 1; i < text.length; i += 1) {
      if (this._match(text, i) !== undefined) {
        return true;
      }
    }
    return false;
  }
}
```

细看之下, 函数签名与我们上文讨论的有所不同，那是因为我们把模式 `pattern` 作为每个模式类的成员变量了，就不需要显式定义在 `_match` 函数中了。

再来看下我们精确匹配字符的 `Lit` 模式类的实现:

```js
class RegexLit extends RegexBase {
  private chars: string;
  rest: RegexBase
  constructor(chars: string, rest: RegexBase | null = null) {
    super()
    this.chars = chars;
    this.rest = rest;
  }

  _match(text: string, start: number): number {
    const nextIndex = start + this.chars.length;
    if (nextIndex > text.length) {
      return INVALID_INDEX;
    }

    if (text.slice(start, nextIndex) !== this.chars) {
      return INVALID_INDEX;
    }

    if (this.rest === null) {
      return nextIndex;
    }

    return this.rest._match(text, nextIndex);
  }
}
```

实现很简单, 但 `rest` 又是什么呢?

还是以 `^c` 为例, 现在改复杂一点, 模式变成 `^cd` 来匹配 `cde` ，模式 `^c` 匹配完 `c` 之后, 就要使用剩下的模式(`rest`) `d` 来匹配剩下的文本 `de`, 剩下的模式可能也会再包含剩下的模式，用来匹配再被剩下的文本，依此类推.

{{< figure src="/ox-hugo/reinvent_regex_rest_pointer.jpg" >}}

相当于 `rest` 就是指向下一个模式类的单向指针，用来表示下一个模式需要匹配剩余的文本，直到所有的模式匹配完成，即 `rest` 指针指向 `null`

所以模式 `cde` 就可以表示成 `Lit('c', Lit('d', Lit('e')))`

而所有的模式组合在一起，本质就是一条单向链条，而正则表达式就是判断是否存在依次匹配链表中所有模式的文本。


### <span class="section-num">4.4</span> Any 模式 {#any-模式}

Any 模式即 `*` 匹配 0到任意个前一个字符，与其类似的还有 Plus 模式，即 `+` 匹配1到任意个前一个字符字符；以及 `?` 表示匹配0到1个前一个字符，Any算是最有代表性和最难实现的模式。

即 `a*b` 表示可以匹配0到任意个 `a` ，再匹配一个 `b` , 所以 `b`, `ab`, `aaaaaab` 它都可以匹配上。

那么问题就来了，既然它可以匹配0到任意个字符，那么匹配的时候我要匹配几个字符呢？

理论上有 `N` 个的可能性, N = 待匹配文本 `text` 的长度。

既然不知道要匹配几个字符，那不如我们把所有可能性都穷举一次呗，而这种穷举算法，则被称为是[回溯算法](https://labuladong.online/algo/essential-technique/backtrack-framework/#%E5%85%A8%E6%8E%92%E5%88%97%E9%97%AE%E9%A2%98%E8%A7%A3%E6%9E%90)([backtracking](https://en.wikipedia.org/wiki/Backtracking))

我们知道穷举的上界是 N(`N=len(text)`), 下界是 0, 那么是从 0 穷举到 `N`, 还是从 `N` 穷举到 `0` 呢？

两种方法都可以解决问题，计算机科学家们还给这两种做法起了个洋气的名字， `N` -&gt; `0`, 因为是先开始匹配所有的字符，所以就被称为贪婪匹配 greedy(eager) matching.

而从 `0` -&gt; `N`, 因为是从0开始，所以又被称为是惰性匹配 lazy matching。

从性能的角度来说，是 `lazy matching` 更优，因为它尽可能地去掉了不必要的匹配了。

我们可以先来看下贪婪匹配的实现，再看下惰性匹配：

```js
class RegexAny extends RegexBase {
  private child: RegexBase;
  private rest: RegexBase;

  constructor(child: RegexBase, rest: RegexBase | null) {
    super();
    this.child = child;
    this.rest = rest;
  }

  _match(text: string, start: number): number | null {
    const maxPossible = text.length - start;
    for (let num = maxPossible; num >= 0; num -= 1) {
      const afterMany = this._matchMany(text, start, num);
      if (afterMany !== undefined) {
        return afterMany;
      }
    }
    return undefined;
  }

  _matchMany(text: string, start: number, num: number) {
    for (let i = 0; i < num; i += 1) {
      start = this.child._match(text, start);
      if (start === undefined) {
        return undefined;
      }
    }

    if (this.rest !== null) {
      return this.rest._match(text, start);
    }
    return start;
  }
}
```

`a*b` 会被解析成, `Any(Lit('a'), Lit('b'))`, 因为 `*` 表示匹配0到任意个前一个字符，前一个字符还可能另外一种模式，所以我们可以把前一个字符也解析成模式，作为 `child` 传入到 `Any`.

`_matchMany` 是从 `start` 匹配到 `start+num` 位置，看是否匹配，而 `maxPossible` 表示当前剩余文本中可能的最大匹配次数.

以 `text = "aab"`, `start = 0`, `pattern = a*b` 为例， `maxPossible = len(text) = 3`,

1.  第一轮尝试(`num=3`):
    -   尝试匹配 3 个 `a` -&gt; 失败(只有 2 个 `a`)
2.  第二轮尝试(`num=2`):

    -   匹配 2 个 =a=(位置 `0->1->2`)
    -   然后匹配 rest(b 在位置 `2->3`): 成功！
    -   返回 3

    {{< figure src="/ox-hugo/reinvent_regex_match_aab.png" >}}

以及使用模式 `a*ab` 来匹配文本 `ab` 的过程:
![](/ox-hugo/reinvent_regex_match_ab.jpg)


### <span class="section-num">4.5</span> 支持的模式 {#支持的模式}

每种模式对应一个单独的类之后，再通过 `rest` 指针进行关联，现在的实现就非常易于扩展了，我们可以很容易地支持其他的模式，具体列表如下：

| 含义          | 字符 | 例子                    | 对应实现                                                                                               |
|-------------|----|-----------------------|----------------------------------------------------------------------------------------------------|
| 任意的字符 `c` | `c`  | `c` 匹配字符c           | [Lit](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-lit.ts)             |
| 任意的单个字符 | `.`  | `.` 匹配任意字符        |                                                                                                        |
| 匹配开头的字符 | `^`  | `^c` 匹配以 `c` 开头的字符串 | [Start](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-start.ts)         |
| 匹配结尾的字符 | `$`  | `c$` 匹配以 `c` 结尾的字符串 | [End](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-end.ts)             |
| 匹配零个或多个的字符 | `*`  | `a*` 匹配0-任意个a的字符串, 贪婪匹配 | [GreedyAny](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-any.ts)       |
| 匹配零个或多个的字符 | `*`  | `a*` 匹配0-任意个a的字符串, 惰性匹配 | [LazyAny](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-lazy-any.ts)    |
| 匹配一个或多个的字符 | `+`  | `a+` 匹配1-任意个a的字符串 | [Plus](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-plus.ts)           |
| 匹配零个或一个的字符 | `?`  | `a?` 匹配0-1个a的字符串 | [Opt](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-opt.ts)             |
| 多选一匹配    | `❘`  | `a❘b` 匹配a或b的字符串  | [Alt](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-alt.ts)             |
| 序列匹配      | `()` | `(ab)` 匹配 ab 的字符串 | [Group](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-group.ts)         |
| 匹配方括号内的任意单个字符 | `[]` | `[abcd]` 匹配a或b或c或d的字符串 | [CharClass](https://github.com/ramsayleung/reinvent/blob/master/regular_expression/regex-charclass.ts) |

```js
describe('Regex testsuite', () => {
    it.each([
        ['a', 'a', true, Lit('a')],
        ['b', 'a', false, Lit('b')],
        ['ab', 'ba', false, Lit('ab')],
        ['^a', 'ab', true, Start(Lit('a'))],
        ['^b', 'ab', false, Start(Lit('b'))],
        ['a$', 'ab', false, Lit('a', End())],
        ['a$', 'ba', true, Lit('a', End())],
        ['a*', '', true, Any(Lit('a'))],
        ['a*', 'baac', true, Any(Lit('a'))],
        ['ab*c', 'ac', true, Lit('a', Any(Lit('b'), Lit('c')))],
        ['ab*c', 'acc', true, Lit('a', Any(Lit('b'), Lit('c')))],
        ['ab*c', 'abc', true, Lit('a', Any(Lit('b'), Lit('c')))],
        ['ab*c', 'abbbc', true, Lit('a', Any(Lit('b'), Lit('c')))],
        ['ab*c', 'abxc', false, Lit('a', Any(Lit('b'), Lit('c')))],
        ['ab*c', 'ac', true, Lit('a', LazyAny(Lit('b'), Lit('c')))],
        ['ab*c', 'acc', true, Lit('a', LazyAny(Lit('b'), Lit('c')))],
        ['ab*', 'ab', true, Lit('a', LazyAny(Lit('b')))],
        ['ab+c', 'ac', false, Lit('a', Plus(Lit('b'), Lit('c')))],
        ['ab+c', 'abc', true, Lit('a', Plus(Lit('b'), Lit('c')))],
        ['a(b|c)d', 'xabdy', true, Lit('a', Alt(Lit('b'), Lit('c'), Lit('d')))],
        ['a(b|c)d', 'xabady', false, Lit('a', Alt(Lit('b'), Lit('c'), Lit('d')))],
        ['ab?c', 'abc', true, Lit('a', Opt(Lit('b'), Lit('c')))],
        ['ab?c', 'acc', true, Lit('a', Opt(Lit('b'), Lit('c')))],
        ['ab?c', 'a', false, Lit('a', Opt(Lit('b'), Lit('c')))],
        ["[abcd]", 'a', true, CharClass([Lit('a'), Lit('b'), Lit('c'), Lit('d')])],
        ["[abcd]", 'ab', true, CharClass([Lit('a'), Lit('b'), Lit('c'), Lit('d')])],
        ["[abcd]", 'xhy', false, CharClass([Lit('a'), Lit('b'), Lit('c'), Lit('d')])],
        ["c[abcd]", 'c', false, Lit('c', CharClass([Lit('a'), Lit('b'), Lit('c'), Lit('d')]))],
    ])('Regex base test ("%s" "%s" "%p")', (_pattern, text, expected, matcher) => {
        const actual = matcher.match(text);
        expect(actual).toBe(expected);
    })
});
```

顺便一提的是，这种相同的验证逻辑, 但是输入多个不同的参数以验证不同case的做法，叫做 `Parameterized Test`

我在《[测试技能进阶系列](https://ramsayleung.github.io/zh/categories/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6/)》的第二篇也曾经介绍过： [Parameterized Tests](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/)

这样我们就完成了一个功能较完整的正则表达式引擎了。


## <span class="section-num">5</span> 表达式解析 {#表达式解析}

虽然我们已经完成了一个正则表达式引擎，只不过我们平时用表达式是 `a*bc` ，现在要写成 `Any(Lit('a'), Lib('b', Lib('c')))` 多个类的实例也太烦琐了。

让我们再来分析下正则表达式，以 `^(a|b|$)*z$` 为例，以任意数量的 `a`, `b`, 或 `$` 开头, 再紧接一个 `z`, 然后结束。

我们可以创建一个树来表达这个表达式:

{{< figure src="/ox-hugo/reinvent_regex_express_as_tree.jpg" >}}

在考虑如何把表达式变成上面那棵树之前，我们可以先从最简单的步骤开始：分割字符串

正如物理学家给不可再分的元素称之为「原子」(atom), 计算机科学家也给不可再分割的文本起了一个名字，称之为 **token**, 类似 `a`, `b`, `$`, `*` 这些都是 token，而把文本切分成 token 的过程，即为 _tokenize_ 。

不同的token可能代表不同的含义，像 `a`, `b`, `c` 这类，所以它们的值不同，但是它们都可以被称为字面量(Literal), 而像 `*`, `+`, `|`, `(`, `)` 这样的字符又各种其代表的含义, 如:

```js
const SYMBOL_TOKEN_TYPE_MAP = {
  '*': TokenKind.Any,
  '|': TokenKind.Alt,
  '(': TokenKind.GroupStart,
  ')': TokenKind.GroupEnd,
}
```

定义好 token 类型之后， `tokenize` 跃然纸上了：

直接按照字符作匹配，如果能匹配上的就是特殊类型的 `Token` ，不然就是字面量:

```js
export interface Token {
  kind: TokenKind,
  location: number
  value?: string,
}

export const tokenize = (text: string) => {
  const result: Token[] = [];
  for (let i = 0; i < text.length; i += 1) {
    const c = text[i]
    if (c in SIMPLE) {
      result.push({ kind: SIMPLE[c], location: i });
    } else if ((c === '^') && (i === 0)) {
      result.push({ kind: TokenKind.Start, location: i });
    } else if ((c === '$') && (i === (text.length - 1))) {
      result.push({ kind: TokenKind.End, location: i });
    } else {
      result.push({ kind: TokenKind.Lit, location: i, value: c });
    }
  }
  return result;
}
```

`^(a|b|$)*z$` 就会被解析成如下的结果:

```js
[
  {
    "kind": "Start",
    "location": 0
  },
  {
    "kind": "GroupStart",
    "location": 1
  },
  {
    "kind": "Lit",
    "location": 2,
    "value": "a"
  },
  {
    "kind": "Alt",
    "location": 3
  },
  {
    "kind": "Lit",
    "location": 4,
    "value": "b"
  },
  {
    "kind": "Alt",
    "location": 5
  },
  {
    "kind": "Lit",
    "location": 6,
    "value": "$"
  },
  {
    "kind": "GroupEnd",
    "location": 7
  },
  {
    "kind": "Any",
    "location": 8
  },
  {
    "kind": "Lit",
    "location": 9,
    "value": "z"
  },
  {
    "kind": "End",
    "location": 10
  }
]
```


## <span class="section-num">6</span> 组装抽象语法树 {#组装抽象语法树}

`tokenize` 的结果是一个包含 Token 的列表，我们要如何组装成树状数据结构呢？

顺带一提，这树状数据结构全称是抽象语法树(Abstract syntax tree, AST), 是一种用来表示程序结构的数据结构，如:

{{< figure src="/ox-hugo/Abstract_syntax_tree_for_Euclidean_algorithm.svg.png" >}}

我们可以分情况来讨论，因为不同的模式有不同的组装方式，组装完之后的 AST 的输出是一个 `output`, 包含组装后的 `token` 列表:

对于表达式 `a`, 我们可以创建一个 `Lit` 类型的 `token` (为了便于理解，「创建」指创建一个 `token`, 然后插入到 `output`.)

对于表达式 `a*` 呢？我们可以先创建一个 `Lit('a')` 的 `token`, 当遇到 `*` 时，因为 `*` 表示匹配0至任意的前一个字符, 所以我们可以创建一个 `Any` 类型的 token, 然后把 `output` 最后一个元素 `pop` 出来，作为 `Any` 的 `child` 元素.

{{< figure src="/ox-hugo/reinvent_regex_construct_ast_any.jpg" >}}

对于表达式 `(ab)`, 情况就变得复杂一些:
当遇到 `(` 括号的时候，我们可以创建一个 `Group` ，但是问题在于，我们不知道这个 `Group` 什么时候结束，即不知道什么时候才会遇上 `)`.

所以我们需要换种解决思路：当遇到 `(`, 创建一个 `GroupStart` 类型的 `token`, 然后再继续处理 `a`, `b`, 当遇到 `)` 时，创建一个 `Group` 类型的 `token`, 然后一直调用 `pop` 函数直到把 `GroupStart` 也 `pop` 出来, 然后把过程中 `pop` 出来的 `token` 都当作是 `Group` 的 `children` 列表，而 `GroupStart` 相当于起到一个标记符的作用。

这种思路就自动处理了 `(a*)` 和 `(a(b*)c)` 的差异:

{{< figure src="/ox-hugo/reinvent_regex_ast_group.jpg" >}}

对于表达式 `a|b`, 我们是否可以参考 `Any` 的做法呢?

遇到 `a` 的时候先创建一个 `Lit('a')`, 遇到 `|` 时再创建一个 `Alt`, 然后把 `Lit('a')` 从 `output` pop 出来作为 `left` 节点， 再遇到下一个字符 `b` 的时候，再把 `Alt` 从 output pop 出来，把 `b` 作为 `right` 节点。

听起来没问题，但是上面的算法无法正确解析 `a|b*`, 它表示匹配一个 `a` 或者是任意数量的 `b`, 但是我们的做法会把它解析成 `(a|b)*`, 即任意数量的 `a` 或 `b`.

更合理的做法是先部分组装 Alt 的 `left` 节点，等解析完所有字符之后，再重新解析一次，把 `right` 节点给组装上。

以 `a|b*` 为例子:

1.  创建一个 `Lit('a')` token
2.  当遇到 `|` 的时候，创建一个 `Alt`, 并将 `Lit('a')` pop 出来作为 `left` 节点
3.  创建一个 `Lit('b')` token
4.  创建一个 `Any` token, 并将 `Lit('b')` pop 出来作为 `child` 节点.
5.  当解析完所有字符后, 再遍历一次 `output`, 如果遇到 `Alt` token, 那么就把它的下一个 `token` (即 `Any`) 作为它的 `right` 节点.

{{< figure src="/ox-hugo/reinvent_regex_ast_alt.jpg" >}}

实现代码如下:

```js
export const parse = (text: string) => {
  const result: Token[] = [];
  const allTokens = tokenize(text);
  for (let i = 0; i < allTokens.length; i += 1) {
    const token = allTokens[i];
    const isLast = i === allTokens.length - 1;
    handle(result, token, isLast);
  }
  return compress(result);
}

const handle = (result: Token[], token: Token, isLast: boolean) => {
  if (token.kind === TokenKind.Lit) {
    result.push(token);
  } else if (token.kind === TokenKind.Start) {
    assert(result.length === 0, 'Should not have start token after other tokens');
    result.push(token);
  } else if (token.kind === TokenKind.End) {
    assert(isLast, `Should not have end token before other tokens`);
    result.push(token);
  } else if (token.kind === TokenKind.GroupStart) {
    result.push(token);
  } else if (token.kind === TokenKind.GroupEnd) {
    result.push(groupEnd(result, token));
  } else if (token.kind === TokenKind.Any) {
    assert(result.length > 0, `No Operand for '*' (location ${token.location})`);
    token.child = result.pop();
    result.push(token)
  } else if (token.kind === TokenKind.Alt) {
    assert(result.length > 0, `No Operand for '|' (location ${token.location})`);
    token.left = result.pop();
    token.right = null;
    result.push(token)
  } else {
    assert(false, `UNIMPLEMENTED`);
  }
}

const groupEnd = (result: Token[], token: Token): Token => {
  const group: Token = {
    kind: TokenKind.Group,
    location: null,
    end: token.location,
    children: []
  };

  while (true) {
    assert(result.length > 0, `Unmatched end parenthesis (location ${token.location})`);
    const child = result.pop();
    if (child.kind === TokenKind.GroupStart) {
      group.location = child.location;
      break;
    }
    group.children.unshift(child);
  }
  return group;
}

// go through the output list to fill in the right side of Alts:
const compress = (raw: Token[]) => {
  const cooked: Token[] = [];
  while (raw.length > 0) {
    const token = raw.pop();
    if (token.kind === TokenKind.Alt) {
      assert(cooked.length > 0, `No right operand for alt (location ${token.location})`);
      token.right = cooked.shift();
    }
    cooked.unshift(token);
  }
  return cooked;
}
```

对于表达式 `a|(bc)`, 输出的 AST 如下:

```js
[
    {
        kind: TokenKind.Alt, location: 1, left: {
            kind: TokenKind.Lit, location: 0, value: 'a'
        },
        right: {
            kind: TokenKind.Group, location: 2, end: 5,
            children: [
                { kind: TokenKind.Lit, location: 3, value: 'b' },
                { kind: TokenKind.Lit, location: 4, value: 'c' },
            ]
        }
    },
]
```


## <span class="section-num">7</span> 实例化 {#实例化}

既然抽象语法树 AST 已经就绪了，我们就差最后一步了，把 AST 转变为我们的类实例.

还记得上文提到过, 不同的模式对应不同的类，然后通过 `rest` 指针指向下一个模式类，以此串成一个链表。

那么我们对于 `output` 这个包含多个 token 的列表，我们可以抽象成两个 token, 当前 token 和下一个 token:

假如我们有函数 `f` 可以把当前 `token` 初始化对应的模式类，我们只需要再把剩下的 token 列表初始化成 `rest`, 那么 `rest` 要怎么初始化呢？

只需要再调用 `f` 即可.

这不就是递归嘛! 是的，通过递归就很简单地把实例化也实现出来了:

```js
export const compile = (text: string): RegexBase => {
  const tokens: Token[] = parse(text);
  return createObjectByAST(tokens);
}

// return instances of classes derived from RegexBase by abstract syntax tree
const createObjectByAST = (tokens: Token[]): RegexBase | null => {
  if (tokens.length === 0) {
    return null;
  }
  const token = tokens.shift();
  if (token.kind === TokenKind.Lit) {
    return Lit(token.value, createObjectByAST(tokens));
  } else if (token.kind === TokenKind.Start) {
    return Start(createObjectByAST(tokens));
  } else if (token.kind === TokenKind.End) {
    assert(tokens.length === 0, `Should not have end token before other tokens`);
    return End();
  } else if (token.kind === TokenKind.Alt) {
    return Alt(createObjectByAST([token.left]), createObjectByAST([token.right]), createObjectByAST(tokens));
  } else if (token.kind === TokenKind.Group) {
    return Group(token.children.map((childToken) => createObjectByAST([childToken])), createObjectByAST(tokens));
  } else if (token.kind === TokenKind.Any) {
    return Any(createObjectByAST([token.child]), createObjectByAST(tokens));
  } else {
    assert(false, `UNKNOWN token type ${token.kind}`);
  }
}
```


## <span class="section-num">8</span> 总结 {#总结}

```js
it.each([
  ['a', 'a', true, Lit('a')],
  ['^a', 'ab', true, Start(Lit('a'))],
  ['a$', 'ab', false, Lit('a', End())],
  ['a*', 'baac', true, Any(Lit('a'))],
  ['ab+c', 'abc', true, Lit('a', Plus(Lit('b'), Lit('c')))],
  ['ab+c', 'abxc', false, Lit('a', Plus(Lit('b'), Lit('c')))],
  ['(ab)|(cd)', 'xaby', true, Alt(Group([Lit('a'), Lit('b')]), Group([Lit('c'), Lit('d')]))],
  ['a(b|c)d', 'xabdy', true, Lit('a', Group([Alt(Lit('b'), Lit('c'))], Lit('d')))],
  ['ab?c', 'ac', true, Lit('a', Opt(Lit('b'), Lit('c')))],
  ['ab?c', 'acc', true, Lit('a', Opt(Lit('b'), Lit('c')))],
  ["[abcd]c", 'ac', true, CharClass([Lit('a'), Lit('b'), Lit('c'), Lit('d')], Lit('c'))],
  ["c[abcd]", 'c', false, Lit('c', CharClass([Lit('a'), Lit('b'), Lit('c'), Lit('d')]))],
])('parse, compile and matcher test ("%s" "%s" "%p")', (pattern, text, expected, expectedMatcher) => {
  const actualMatcher = compile(pattern);
  expect(actualMatcher).toStrictEqual(expectedMatcher);
  const actual = actualMatcher.match(text);
  expect(actual).toBe(expected);
})
```

大功告成，终于将所有的功能都组装起来实现这个正则表达式引擎了, 除去前文提到的功能之外，还实现了 `\*` 转义特殊字符， `[xya]` 匹配 `x`, `y`, `z` 其中任意字符，以及 `*?` 实现惰性匹配的功能。

完整功能集的测试 case 可见 [parser_test.ts](https://github.com/ramsayleung/reinvent/blob/master/__tests__/regular_expression/parser-test.ts)

日常使用正则表达式的场景非常多，因为其强大的功能和表达能力，总会下意识觉得很难实现（当然，高性能的完整版本的确是非常有挑战性的）。

但是当自己把正则表达式引擎这个轮子拆开，再重新造一个出来之后，才感悟到：

「没有启程的路才会遥不可及」，很多时候，困难只是我们给自己设下的心理障碍。

[回到本系列的目录]({{< relref "reinvent_project" >}})


## <span class="section-num">9</span> 参考 {#参考}

-   <https://en.wikipedia.org/wiki/Stephen_Cole_Kleene>
-   <https://en.wikipedia.org/wiki/Ed_(software)>
-   <https://en.wikipedia.org/wiki/Grep>
-   <https://third-bit.com/sdxjs/regex-parser/>
-   <https://third-bit.com/sdxjs/pattern-matching/>
