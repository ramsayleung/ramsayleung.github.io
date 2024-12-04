+++
title = "Rust通过Trait扩展已有类型"
date = 2024-12-04T18:04:00+08:00
lastmod = 2024-12-04T19:21:59+08:00
tags = ["rust", "programming"]
categories = ["rust", "programming"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Swift extension {#swift-extension}

可扩展性是一个语言非常关键的特性，以Swift 为例，它有一个相当好用的特性，名为 [extension](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions/), 它可以非常便利地扩展已有的类型, 例如给已有类型增加 computed property, 实例方法, 新增构造器又或是实现新的 Protocol.

已有的类型既可以是你自己的代码，或者是第三方的代码，甚至是标准库的代码, 以标准库的 `String` 类型为例:

```swift
extension String {
    var isPalindrome: Bool {
        let reversed = String(self.reversed())
        return self == reversed
    }
    func greet() -> Void {
        print("Hello \(self)")
    }
}

let word = "racecar"
print(word.isPalindrome) // Outputs: true
word.greet() // Outputs: Hello racecar
```

又或者让 `String` 实现新的 Protocol, 如:

```swift
extension String: YourOwnProtocol {

}
```

换言之，如果你对已有的类型不满意，你可以直接扩展已有的类型，添加上你想要的属性，方法或者实现你期望的接口。


## <span class="section-num">2</span> Rust 的扩展能力 {#rust-的扩展能力}

Rust 也部分支持Swift extension 特性，如让已有的类型实现新的Trait.

还是以 `String` 为例子, 我们希望给 `String` 实现一个 `Greet` 的接口:

```rust
// Define a trait with the desired functionality
trait Greet {
    fn greet(&self) -> String;
}

// Implement the trait for an existing type
impl Greet for String {
    fn greet(&self) -> String {
        format!("Hello, {}!", self)
    }
}

fn main() {
    let name = String::from("Rust");
    println!("{}", name.greet()); // Outputs: "Hello, Rust!"
}
```

这样我们就给 `String` 添加上 `greet` 方法，不足之处在于，需要定义一个额外的 `trait=，没有像 Swift 那样的 =extension` 语法糖可以用.


### <span class="section-num">2.1</span> 实际例子 {#实际例子}

上面的 `Greet` 接口可能过于简单，让我们来看下实际项目的例子, 在[测试技能进阶(三): Property Based Testing](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%89_property_based_testing/) 一文中，我提到了使用 [Quickcheck](https://github.com/BurntSushi/quickcheck) 库在Rust实现 Property Based Testing.

假如有 Book struct, 我们只要实现 quickcheck 的 Arbitrary 接口，quickcheck 就会按照我们指定的规则来生成随机测试数据:

```rust
use quickcheck::{Arbitrary, Gen};

#[derive(Debug, Clone, PartialEq)]
struct Book {
    isbn: String,
    title: String,
    author: String,
    publication_year: u16,
}

impl Arbitrary for Book {
    fn arbitrary(g: &mut Gen) -> Self {
        Book {
            // isbn必须以`ISBN` 开头，后接任意的大于等于0，小于uint32.max_value
            // 的整型
            isbn: format!("ISBN-{}", u32::arbitrary(g)),
            title: String::arbitrary(g), // 任意的字符串
            author: String::arbitrary(g), // 任意的字符串
            publication_year: *g.choose(&[2014_u16, 2022_u16, 2025_u16]).unwrap(), // 2014,2022或2025年出版的书
        }
    }
}
```

quickcheck 的 `Gen` 结构体有一个非常顺手的函数 `gen_range` ，用于生成指定的范围的数据, 但是作者在[1.0](https://github.com/BurntSushi/quickcheck/blob/aa968a94650b5d4d572c4ef581a7f5eb259aa0d2/src/arbitrary.rs#L72)之后，就不向外暴露这个接口了，不然我们就可以通过 `g.gen_range(b'a'...b'z') as char)` 来指定我们想要的数据.

既然这么好用的函数没有了，我们可以通过 `Trait` 的扩展能力，把这个 `gen_range` 函数带回来.

思路很简单，就是定义一个 `GenRange` Trait, 然后再让 `Gen` 实现这个 `Trait`.

```rust
use core::ops::Range;

use num_traits::sign::Unsigned;
pub use quickcheck::*;

pub trait GenRange {
    fn gen_range<T: Unsigned + Arbitrary + Copy>(&mut self, _range: Range<T>) -> T;
}

impl GenRange for Gen {
    fn gen_range<T: Unsigned + Arbitrary + Copy>(&mut self, range: Range<T>) -> T {
        <T as Arbitrary>::arbitrary(self) % (range.end - range.start) + range.start
    }
}
```

通过上面的代码, 我们就可以在 `Book` 的 `arbitrary` 函数中使用 `gen_range` 了:

```rust
impl Arbitrary for Book {
    fn arbitrary(g: &mut Gen) -> Self {
        Book {
            // isbn必须以`ISBN` 开头，后接任意的大于等于0，小于uint32.max_value
            // 的整型
            isbn: format!("ISBN-{}", u32::arbitrary(g)),
            title: String::arbitrary(g), // 任意的字符串
            author: String::arbitrary(g), // 任意的字符串
            publication_year: g.gen_range(2014...2026), // 2014-2025年出版的书
        }
    }
}
```


### <span class="section-num">2.2</span> 使用已有Trait扩展已有类型 {#使用已有trait扩展已有类型}

上面提到的例子都是通过定义一个新的 `Trait`, 然后让已有类型实现这个新Trait, 那么是否可以让已有类型实现已有的Trait 呢?

事实上, 由于Orphan Rule的限制, Rust 并不允许已有类型实现已有接口, 以下的代码是无法编译通过的:

```rust
use std::fmt;

// Implement the external trait for the wrapper
impl fmt::Display for String {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "String: {}", self)
    }
}
```

所谓的 `Orphan Rule` 限制指的是，如果允许已有类型实现已有接口, 那么 `lib1` 和 `lib2` 都实现了 `impl fmt::Display for String`, 编译器并不知道应该使用哪个lib的实现.

对此，Rust 官方也提供了[指引](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types)，我们可以通过定义一个 `Wrapper` 类来实现我们的诉求：

```rust
use std::fmt;

// Define a newtype wrapper
struct MyString(String);

// Implement the external trait for the wrapper
impl fmt::Display for MyString {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "MyString: {}", self.0)
    }
}

fn main() {
    let s = MyString("Hello".to_string());
    println!("{}", s); // Outputs: MyString: Hello
}
```


## <span class="section-num">3</span> 参考 {#参考}

-   [Using the Newtype Pattern to Implement External Traits on External Types](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types)
-   [add back a way to put a bound on numbers generated](//github.com/BurntSushi/quickcheck/issues/267)
