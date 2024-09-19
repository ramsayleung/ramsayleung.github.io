+++
title = "使用Rust的Iterator优雅解决FizzBuzz问题"
date = 2024-09-18T22:46:00-07:00
lastmod = 2024-09-19T00:03:03-07:00
tags = ["rust"]
categories = ["rust"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

按照维基百科的说法，[FizzBuzz问题](https://en.wikipedia.org/wiki/Fizz_buzz) 是一个简单但是常见的面试编程问题（可能以前常见，现在都是考Leetcode了,这种连Easy 都不算了），这个问题的要求如下： <br/>

1.  写一个程序，输出从1到100的数字 <br/>
2.  对于3的倍数，不输出数字，而是输出 "Fizz" <br/>
3.  对于5的倍数，不输出数字，而是输出 "Buzz" <br/>
4.  对于即是3的倍数又是5的倍数的数字（即15的倍数），打印 "FizzBuzz" <br/>


## <span class="section-num">2</span> 常规解法 {#常规解法}

问题非常简单，刚学编程的学生都可以写出符合要求的代码，下面是 Rust 的常规解法： <br/>

```rust
fn main() {
    for i in 0..=100 {
        if i % 3 == 0 && i % 5 == 0 {
            println!("FizzBuzz");
        } else if i % 3 == 0 {
            println!("Fizz");
        } else if i % 5 == 0 {
            println!("Buzz");
        } else {
            println!("{i}");
        }
    }
}
```

这个没有什么太多可说的，就是直接按需求翻译代码了。 <br/>


## <span class="section-num">3</span> Iterator 解法 {#iterator-解法}

如果现在给 FizzBuzz 问题再加一个限制，不能使用乘法，除法，或者取模操作，那么又要怎么实现呢？ <br/>

Rust 标准库中的各式 `Iterator` 可以算是Rust零开销抽象(Zero Cost Abstraction)与表达能力的最佳体现了。 <br/>

最近在读 Programming Rust, 2nd edition, 里面就有使用各种 Iterator 组合，不使用除法或者取模操作来解决 FizzBuzz 问题的实现, 可以说是把 `iterator` 玩得非常花了： <br/>

```rust
use std::iter::{once, repeat};

fn main() {
    let fizzes = repeat("").take(2).chain(once("fizz")).cycle();
    let buzzes = repeat("").take(4).chain(once("buzz")).cycle();
    let fizzes_buzzes = fizzes.zip(buzzes);

    let fizz_buzz = (1..=100).zip(fizzes_buzzes).map(|tuple| match tuple {
        (i, ("", "")) => i.to_string(),
        (_, (fizz, buzz)) => format!("{}{}", fizz, buzz),
    });

    for line in fizz_buzz {
        println!("{line}")
    }
}
```

看起来是否不知道所云呢? 现在可以把每个 `iterator` 的作用逐一拆解。 <br/>


### <span class="section-num">3.1</span> repeat + take {#repeat-plus-take}

`repeat` 的作用就是无限重复某个传入的元素, 例如 `repeat(4)` 就是生成无限个数字4, `repeat("")` 就是生成无限个空白字符. <br/>

虽然 `repeat` 能生成无限个指定的元素，但是我只想要若干个元素，怎么整呢？ `take` 就可以满足这个要求，所以 `repeat(4).take(4)` 就是生成4个数字4的意思，而 `repeat("").take(2)` 就是生成2个空字符 <br/>

```rust
use std::iter;

// that last example was too many fours. Let's only have four fours.
let mut four_fours = iter::repeat(4).take(4);

assert_eq!(Some(4), four_fours.next());
assert_eq!(Some(4), four_fours.next());
assert_eq!(Some(4), four_fours.next());
assert_eq!(Some(4), four_fours.next());

// ... and now we're done
assert_eq!(None, four_fours.next());
```


### <span class="section-num">3.2</span> once {#once}

有生成无限个元素的 `iterator`, 自然就有只生成一个元素的 `iterator`, 那就是 `once()`, 这个 `iterator` 只会返回一个指定的元素。 <br/>

所以 `once("fizz")` 就是创建一个只会返回一个 `"fizz"` 的 `iterator` : <br/>

```rust
use std::iter;

// one is the loneliest number
let mut one = iter::once(1);

assert_eq!(Some(1), one.next());

// just one, that's all we get
assert_eq!(None, one.next());
```


### <span class="section-num">3.3</span> chain {#chain}

顾名思义，就是把两个 iterator 像链子一样串起来, 合并成一个 iterator: <br/>

```rust
use std::iter::chain;

let a = [1, 2, 3];
let b = [4, 5, 6];

let mut iter = chain(a, b);

assert_eq!(iter.next(), Some(1));
assert_eq!(iter.next(), Some(2));
assert_eq!(iter.next(), Some(3));
assert_eq!(iter.next(), Some(4));
assert_eq!(iter.next(), Some(5));
assert_eq!(iter.next(), Some(6));
assert_eq!(iter.next(), None);
```


### <span class="section-num">3.4</span> circle {#circle}

`circle` 就比较有趣了，它的作用是无限循环一个 `iterator`, `repeat` 循环一个元素，而 `circle` 是循环一个 iterator: <br/>

```rust
let dirs = ["North", "East", "South", "West"];
let mut spin = dirs.iter().cycle();
assert_eq!(spin.next(), Some(&"North"));
assert_eq!(spin.next(), Some(&"East"));
assert_eq!(spin.next(), Some(&"South"));
assert_eq!(spin.next(), Some(&"West"));
assert_eq!(spin.next(), Some(&"North"));
assert_eq!(spin.next(), Some(&"East"));
```

把4个 iterator 组合起来的 `repeat("").take(2).chain(once("fizz")).cycle();` 表达式的意思就是: 返回一个 iterator, 这个 iterator 无限循环: `"" "" "fizz" "" "" "fizz" ...` <br/>


### <span class="section-num">3.5</span> zip {#zip}

`zip` iterator 的含义就是 "zips up", 翻译过来就是拉上拉链，它的作用就是把两个 `iterator` 像拉链一样拉起来，返回一个 iterator，用代码来解释会更直观: <br/>

```rust
let a1 = [1, 2, 3];
let a2 = [4, 5, 6];

let mut iter = a1.iter().zip(a2.iter());

assert_eq!(iter.next(), Some((&1, &4)));
assert_eq!(iter.next(), Some((&2, &5)));
assert_eq!(iter.next(), Some((&3, &6)));
assert_eq!(iter.next(), None);
```

`zip` 就是把 `a1` 和 `a2` 两个iterator 「拉起来」了，每次返回一对的元素. 所以 `fizzes.zip(buzzes)` ，就是合并了两个 iterator : <br/>

```rust
// fizzes: "" "" "fizz" "" "" "fizz" "" "" "fizz" ..
// buzzes: "" "" "" "" "buzz" "" "" "" "" "buzz"
// fizzes_buzzes: ("" "") ("" "") ("fizz" "") ("" "") ("" "buzz") ...
```

而 `(1..=100).zip(fizzes_buzzes)` 就是创建一个包含三个元素的 tuple： <br/>

```rust
// (1..=100): 1 2 3 4 5 6 7 ...
// fizzes_buzzes: ("" "") ("" "") ("fizz" "") ("" "") ("" "buzz") ...
// (1..=100).zip(fizzes_buzzes): (1 ("" "")) (2 ("" "")) (3 ("fizz" "")) (4 ("" "")) (5 ("" "buzz")) ..
```


### <span class="section-num">3.6</span> map {#map}

`map` 这个 iterator 在其他语言也有相同的实现，入参是一个闭包函数，然后把每个元素作为入参，调用闭包函数，在新的迭代返回函数的调用结果. <br/>

```rust
.map(|tuple| match tuple {
    (i, ("", "")) => i.to_string(),
    (_, (fizz, buzz)) => format!("{}{}", fizz, buzz),
})
```

最核心的是Rust的 pattern matching, 用来匹配不同的值, `(i, ("", ""))` 就是匹配所有 fizz 和 buzz为 `("", "")` 的值，什么情况下 `fizz` 和 `buzz` 会都为 "" 呢，无法整除3以及无法整除5的时候，那么就直接返回数字 `i`; <br/>

`(_, (fizz,buzz))`, `_` 就是通配符，就是匹配掉所有其他的情况，无论是 fizz = "", fizz = "fizz", buzz = "" 或者 buzz = "buzz", 都把返回 `"{fizz}{buzz}"`, 也就是 `(_, (fizz,buzz))` 匹配了4种情况. <br/>

`map` 迭代器返回的是一个 String, 最后再加 String 打印出来. <br/>

同样是解决问题，这个版本的解法肯定是看起来「高大上」得多，说不定能让面试官眼前一亮，又或者是把自己绕晕。 <br/>


## <span class="section-num">4</span> Zero Cost Abstraction {#zero-cost-abstraction}

所谓的是零开销抽象（Zero Cost Abstraction），用C++之父的话来解释就是: <br/>

> In general, C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better. <br/>

概括来说，就是使用 Iterator 写出来的代码，和你自己 for-loop 手写是性能是一样的，并不会有额外的抽象开销。 <br/>

换个角度讲，你手写的代码也没法实现得比 Iterator 更快，表达力还可能没有那么强。 <br/>

如果看上面的 Iterator 实现觉得着实难以理解，我们可以再来一版兼具优雅与简洁的实现： <br/>

```rust
fn main() {
    for i in 1..=100 {
        match (i % 3, i % 5) {
            (0, 0) => println!("FizzBuzz"),
            (0, _) => println!("Fizz"),
            (_, 0) => println!("Buzz"),
            (_, _) => println!("{}", i),
        }
    }
}
```


## <span class="section-num">5</span> Reference {#reference}

-   Programming Rust, 2nd edition <br/>

