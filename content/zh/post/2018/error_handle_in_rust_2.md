+++
title = "Rust的错误处理(二)"
date = 2018-02-08T22:01:00+08:00
lastmod = 2022-02-24T16:52:24+08:00
tags = ["rust", "java"]
categories = ["rust"]
draft = false
toc = true
showQuote = true
+++

自定义错误和`error_chain` 库


## <span class="section-num">1</span> 前言 {#前言}

上一篇文章聊到 Rust 的错误处理机制，以及和 Java 的简单比较，现在就来聊一下如何在 Rust 自定义错误，以及引入 `error_chain`这个库来优雅地进行错误处理。

还有，少不了用 Java 来做对比咯:)


### <span class="section-num">1.1</span> Java 自定义异常 {#java-自定义异常}

前文简单提到 Java 的错误和异常但是继承自一个 `Throwable`的父类，既然异常是继承自异常父类的，我们自定义异常的时候，
也可以模仿JDK, 继承一个异常类：

```java
public class MyException extends Exception {
    public MyException(String message) {
	super(message);
    }
}
```

这样就定义了属于自己的异常. 只需要继承 `Exception`,然后调用父类的构造方法。

不过 对于那些复杂的项目，这样的例子未免过于简单。现在就来看一个我项目的中的一个异常类：

```rust
public final class MyError extends RuntimeException {
    /**
     *
     */
    private static final long serialVersionUID = 1L;

    private static boolean isFillStack = true;

    private Integer httpStatusCode;
    private Integer code;
    private String message;

    private MyError(int httpStatusCode, int code, String message) {
	super(message, null, isFillStack, isFillStack);
	this.httpStatusCode = httpStatusCode;
	this.code = code;
	this.message = message;
    }

    public MyError(MyErrorCode myErrorCode, Object... messageArgs) {
	this(myErrorCode.getHttpStatusCode(), myErrorCode.getErrorCode(),
	     MessageFormat.format(myErrorCode.getMessagePattern(), messageArgs));
    }

    public static MyError throwError(MyErrorCode myErrorCode, Object... messageArgs) {
	throw new MyError(myErrorCode, messageArgs);
    }

    public static MyError internalServerError(String logId) {
	throw new MyError(MyErrorCode.INTERNAL_SERVER_ERROR, logId);
    }
    public static MyError DataError(String logId) {
	throw new MyError(MyErrorCode.DATA_ERROR, logId);
    }

    public static MyError BadParameterError(String logId) {
	throw new MyError(MyErrorCode.BAD_PARAMETER_ERROR, logId);
    }
}
```

这是我去掉了多余方法和变量的简化版，但是也足以一叶知秋了。

`MyError`这个异常类是 继承于 `RuntimeException`的，并调用了 `RuntimeException`的构造方法。

因为我的项目是 WEB 服务的业务层，要处理大量的逻辑，难免会出现异常.

比如说可能调用方调用接口 的时候，入参不符合规范，我就抛出一个经过包装的 `BadParameterError` 异常，对于接 口调用方，这样会比一个单纯的 400 错误要友好，其他的异常也是同理。


### <span class="section-num">1.2</span> Rust 自定义错误 {#rust-自定义错误}

对于习惯了 OOP 编程的同学来说，Java 的异常是很容易理解，但是回到 Rust 身上，Rust是没有父类一说的，显然，Rust 是没可能套用 Java 的自定义异常的方式的。

Rust 用的是 `trait`, `trait`就有点类似 Java 的 =interface=(只是类似，不是等同!).

按照 Rust 的规范，Rust 允许开发者定义自己的错误，设计良好的错误应该包含以下的特性：

1.  使用相同的类型(type)来表示不同的错误
2.  错误中包含对用户友好的提示(我也在上面提到的)
3.  能便捷地与其他类型比较，例如：
    -   Good: `Err(EmptyVec)`
    -   Bad: `Err("Please use a vector with at least one element".to_owned())`
4.  包含与错误相关的信息，例如：
    -   Good: `Err(BadChar(c, position))`
    -   Bad: `Err("+ cannot be used here".to_owned())`
5.  可以很方便地与其他错误结合

<!--listend-->

```rust
use std::error;
use std::fmt;
use std::num::ParseIntError;

type Result<T> = std::result::Result<T, DoubleError>;

#[derive(Debug, Clone)]
// 自定义错误类型。
struct DoubleError;

// 不同的错误需要展示的信息也不一样，这个就要视情况而定，因为 DoubleError 没有定义额外的字段来保存错误信息
// 所以就现在就简单打印错误信息
impl fmt::Display for DoubleError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
	write!(f, "invalid first item to double")
    }
}

// 实现 error::Error 这个 trait, 对DoubleError 进行包装
impl error::Error for DoubleError {
    fn description(&self) -> &str {
	"invalid first item to double"
    }

    fn cause(&self) -> Option<&error::Error> {
	// Generic error, underlying cause isn't tracked.
	None
    }
}

fn double_first(vec: Vec<&str>) -> Result<i32> {
    vec.first()
    // Change the error to our new type.
	.ok_or(DoubleError)
	.and_then(|s| s.parse::<i32>()
		  // Update to the new error type here also.
		  .map_err(|_| DoubleError)
		  .map(|i| 2 * i))
}

fn print(result: Result<i32>) {
    match result {
	Ok(n)  => println!("The first doubled is {}", n),
	Err(e) => println!("Error: {}", e),
    }
}

fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    print(double_first(numbers));
    print(double_first(empty));
    print(double_first(strings));
}
```

这段代码的运行结果如下：

```text
The first doubled is 84
Error: invalid first item to double
Error: invalid first item to double
```


### <span class="section-num">1.3</span> error_chain {#error-chain}

虽说 Rust 自定义错误很灵活和方便，但是如果每次定义异常都需要实现 `Display` 和 `Error`, 未免过于繁琐，现在来介绍
[error_chain](https://github.com/rust-lang-nursery/error-chain) 这个类库。

`error_chain` 是由 Rust 项目组的 leader--[Brian Anderson](https://brson.github.io/) 编写的异常处理库，可以让你更舒心~~简单不粗 暴~~地定义错误。

---


#### <span class="section-num">1.3.1</span> error_chain示例 {#error-chain示例}

以上面的 `DoubleError`为例，并改写 error_chain 的官方[例子](https://github.com/rust-lang-nursery/error-chain/blob/master/examples/quickstart.rs) 以实现相同的效果，代码如下：

```rust
// `error_chain!` 的递归深度
#![recursion_limit = "1024"]

//引出 error_chain 和相应的宏
#[macro_use]
extern crate error_chain;

//将跟错误有关的内容放入 errors module, 其他需要用到这个错误module 的模块就通过
// use errors::* 来引入所有内容
mod errors {
    // Create the Error, ErrorKind, ResultExt, and Result types
    error_chain! {
	errors{Double{
	    description("invalid first item to double")
		display("invalid first item to double")
	}}
    }
}

use errors::*;
pub type Result<T> = ::std::result::Result<T, ErrorKind>;
fn main() {
    let numbers = vec!["42", "93", "18"];
    let empty = vec![];
    let strings = vec!["tofu", "93", "18"];

    print(double_first(numbers));
    print(double_first(empty));
    print(double_first(strings));

}
fn double_first(vec: Vec<&str>) -> Result<i32> {
    vec.first()
    // Change the error to our new type.
	.ok_or(ErrorKind::Double)
	.and_then(|s| s.parse::<i32>()
		  // Update to the new error type here also.
		  .map_err(|_| ErrorKind::Double)
		  .map(|i| 2 * i))
}

fn print(result: Result<i32>) {
    match result {
	Ok(n) => println!("The first doubled is {}", n),
	Err(e) => println!("Error: {}", e),
    }
}
```

运行这代码可以得到和上面小节同样的输出。


#### <span class="section-num">1.3.2</span> error_chain 详解 {#error-chain-详解}

刚刚就先目睹了一下 `error_chain` 的芳容了，现在是时候来解剖一下 `error_chain`, 这次就以 `error_chain`的 [example](https://github.com/rust-lang-nursery/error-chain/blob/master/examples/quickstart.rs)来解释

```rust
// Simple and robust error handling with error-chain!
// Use this as a template for new projects.

// `error_chain!` can recurse deeply
#![recursion_limit = "1024"]

// Import the macro. Don't forget to add `error-chain` in your
// `Cargo.toml`!
#[macro_use]
extern crate error_chain;

// We'll put our errors in an `errors` module, and other modules in
// this crate will `use errors::*;` to get access to everything
// `error_chain!` creates.
mod errors {
    // Create the Error, ErrorKind, ResultExt, and Result types
    error_chain! { }
}

use errors::*;

fn main() {
    if let Err(ref e) = run() {
	println!("error: {}", e);

	for e in e.iter().skip(1) {
	    println!("caused by: {}", e);
	}

	// The backtrace is not always generated. Try to run this example
	// with `RUST_BACKTRACE=1`.
	if let Some(backtrace) = e.backtrace() {
	    println!("backtrace: {:?}", backtrace);
	}

	::std::process::exit(1);
    }
}

// Most functions will return the `Result` type, imported from the
// `errors` module. It is a typedef of the standard `Result` type
// for which the error type is always our own `Error`.
fn run() -> Result<()> {
    use std::fs::File;

    // This operation will fail
    File::open("contacts")
	.chain_err(|| "unable to open contacts file")?;

    Ok(())
}
```

重要的信息例子已经作了注释，现在就来看看用法。首先来看看 main 函数

```rust
fn main() {
    if let Err(ref e) = run() {
	println!("error: {}", e);

	for e in e.iter().skip(1) {
	    println!("caused by: {}", e);
	}

	// The backtrace is not always generated. Try to run this example
	// with `RUST_BACKTRACE=1`.
	if let Some(backtrace) = e.backtrace() {
	    println!("backtrace: {:?}", backtrace);
	}

	::std::process::exit(1);
    }
}
```

可以看出，这个函数的大部份逻辑是进行错误处理，例如返回自定义的 `Result`和 `Error`, 然后处理这些错误。上面的处理流程显示了 `error_chain` 从某个错误继承而来的三样信息：最近出现的错误(即`e`)，导致错误的调用链，原来错误的堆栈信息 (`e.backtrace()`)


## <span class="section-num">2</span> 小结 {#小结}

刚刚的例子只是 `error_chain` 小试了一波牛刀，如果想要了解更多关于 Rust 异常处理 的细节，就需要看看 Rust 的文档咯


## <span class="section-num">3</span> 参考 {#参考}

-   [rust by example](https://rustbyexample.com/error.html)
-   [starting with error chain](https://brson.github.io/2016/11/30/starting-with-error-chain)
-   [24-days-rust-error_chain/](http://siciarz.net/24-days-rust-error_chain/)
-   [handling errors in rust](https://vincent.is/handling-errors-in-rust/)
-   [error chain](https://docs.rs/error-chain)
-   [error handle](https://doc.rust-lang.org/book/second-edition/ch09-00-error-handling.html)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

