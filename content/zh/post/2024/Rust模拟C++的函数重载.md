+++
title = "Rust模拟C++的函数重载"
date = 2024-08-30T22:23:00-07:00
lastmod = 2024-08-30T23:00:10-07:00
tags = ["rust", "c++"]
categories = ["rust", "c++"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 函数重载(function overloading) {#函数重载--function-overloading}

所谓的函数重载，指的是某些语言支持创建函数名相同，但函数签名不同的多个函数，所谓的函数签名，既指参数类型，也指参数的数量。 <br/>

如C++，Java都是支持函数重载的，而Rust是不支持函数重载的, 个人猜测可能是Rust最初的设计者认为函数重载可能会导致增加代码理解难度，尤其是在C++里面，隐式类型转换叠加函数重载，可能看代码都看不出实际调用的是哪个版本的函数。 <br/>


## <span class="section-num">2</span> Rust版本的函数重载 {#rust版本的函数重载}

但是我个人觉得函数重载在大部分情况下都是很方便，也不需要为相同的函数想不同的名字，毕竟命名是编程最难的问题之一。 <br/>
今天重读 Programming Rust, 2nd Edition关于 `Into` 这个trait 的功能的时候，突然意识到，可以使用 `Into` 模拟出部分的函数重载功能。 <br/>

为什么说是「部分」呢，因为前文提到，所谓的函数重载是指多个同名但函数签名不一样的函数，而Rust能模拟的就是参数类型不一样，但是参数数量一致的重载函数。 <br/>

假设我们想实现自己的 `ping` 命名, 入参可以是 [`Ipv4Addr`](https://doc.rust-lang.org/std/net/struct.Ipv4Addr.html) 这个 struct, ipv4的地址也可以使用2进制来表示, 又或者可以使用 u32 来表示，毕竟只有32位。 <br/>

如果用 C++, 我们可以写3个重载函数，入参分别是, `Ipv4Addr`, `bitset` 和 `uint32`. 在 Rust, 我们也实现类似的函数： <br/>

```rust
use std::net::Ipv4Addr;
fn ping<A>(address: A) -> std::io::Result<bool>
where A: Into<Ipv4Addr>
{
    let ipv4_address = address.into();
    ...
}
```

需要注意的是，上面函数的入参并不是 `Ipv4Addr`, 而是 `Into<Ipv4Addr>` ，这就是意味着，所有实现了 `Into<Ipv4Addr>` 这个 trait 的类型都可以是 `ping` 的入参，而恰好 `u32` 和 `[u8; 4]` 都实现了 `Into<Ipv4Addr>` ，所以下面的调用都是编译通过的： <br/>

```rust
println!("{:?}", ping(Ipv4Addr::new(23, 21, 68, 141))); // pass an Ipv4Addr
println!("{:?}", ping([66, 146, 219, 98]));             // pass a [u8; 4]
println!("{:?}", ping(0xd076eb94_u32));                 // pass a u32
```

当然，如果你实现了 `impl From<u32> for Ipv4Addr`, Rust 编译器也会贴心地帮你把反向的 `Into<Ipv4Addr>` 也实现掉。 <br/>


## <span class="section-num">3</span> 限制 {#限制}

看完上面的函数实现，有经验的朋友可能就会发现了，Rust版本的函数重载限制比C++的要多。 <br/>

在C++版本的函数重载中： <br/>

```c++
void func1(Type1 foo);

void func1(Type2 bar);
```

参数类型 `Type1` 和 `Type2` 并不需要存在任何关系，但是在 Rust 版本中，需要两个类型之间支持相互转换，所以可以理解成 Rust 的「函数重载」本质就是通过显示类型转换来实现的。 <br/>

毕竟 Rust 设计初衷之一就是支持强类型，就函数重载而言，终归聊胜于无啦。 <br/>


## <span class="section-num">4</span> 参考 {#参考}

-   [Programming Rust, 2nd Edition](https://www.oreilly.com/library/view/programming-rust-2nd/9781492052586/) <br/>

