+++
title = "Rust的错误处理(一)"
date = 2018-02-05T20:35:00+08:00
lastmod = 2022-02-24T16:38:50+08:00
tags = ["rust", "java"]
categories = ["rust"]
draft = false
toc = true
+++

拉上Java 来谈谈 Rust的错误处理


## <span class="section-num">1</span> 前言 {#前言}

每个语言都会有异常处理机制（没有异常处理机制的语言估计也没有人会用了），Rust 自然也不例外，所以今天我就来谈Rust 的异常处理，因为 Rust 的异常处理跟常见的语言 （Java/Python 等）的处理机制差异略大，所以打算拉个上个语言，对比着解释. 没错，这 个光荣的任务就落到了 Java 身上


## <span class="section-num">2</span> Java 的异常处理 {#java-的异常处理}

在谈 Rust 的异常处理之前，为了把它们之前的差异讲清楚，先来聊一下 Java 的异常处理。

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/java_exception_hierarchy.png" caption="<span class=\"figure-number\">Figure 1: </span>Java exception hierarchy" >}}

如上面的简易图所示， Java 的异常都是继承于 `Throwable` 这个分类的，而异常又是分 成不同的类型： `Error`, `Exception`;
`Exception` 又分成 `Checked Exception` 和 `RuntimeException`.

`Error` 一般都是了出现严重的问题，按照JDK 注释的说法，都是不应该 `try-catch`的：

> An {() Error} is a subclass of {() Throwable}
> that indicates serious problems that a reasonable application should
> not try to catch. Most such errors are abnormal conditions.

比如虚拟机挂了，或者JRE 出了问题就可能是 `Error`，前几天我就遇到一个JRE 的 Bug, 整个项目都挂 了：

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20191104104407.png" caption="<span class=\"figure-number\">Figure 2: </span>JRE fatal error" >}}

我还顺便给 Oracle 报了个[Bug](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=JDK-8196769) :)

至于`RuntimeException` 就是类似数组越界，空指针这些异常，即无法在程序编译时发现，只有在运行的时候才会出 现的问题，所以叫做运行时异常(`RuntimeException`).


## <span class="section-num">3</span> Checked Exception {#checked-exception}

Java的`Checked Exception`, 也就是Java 要求你必须在函数的类型里面声明或处理它可能抛出的异常。比如，你的函数如果是这样：

```java
void foo(string filename) throws IOException
{
    File file = new File(filename);

    BufferedReader br = new BufferedReader(new FileReader(file));

    String st;
    while ((st = br.readLine()) != null)
	System.out.println(st);
}
}
```

Java 要求你必须在函数头部写上 `throws IOException` 或者是必须用 `try-catch`处理这个异常，因为`readline()` 的方法签名是：

```java
String readLine(boolean ignoreLF) throws IOException {
}
```

所以编译器要求必须要处理这个异常，否则它就不能编译。

同理，在使用 `foo()`这个函数 的时候，可能会抛出 `IOException` 这个异常，由于编译器看到了这个声明，它会严格检 查你对 `foo`
函数的用法。

在我看来，`CheckedException`是Java 优良的设计之一，正因 为`Checked Exception`的存在，会更容易编写出正确处理错误的程序，更健壮的程序


## <span class="section-num">4</span> Rust 的异常处理 {#rust-的异常处理}

Rust 是一个注重安全（Safety）的语言，而错误处理也是 Rust关注的要点之一。

Rust 主要是将错误划分成两种类型，分别是可恢复的错误(recoverable error) 和不可恢复错误 (unrecoverable error).

出现可恢复的错误的原因多种多样，例如打开文件的时候，文件找不到或者没有读权限等，开发者就应该对这种可能出现的错误进行处理；

而不可恢复的错误就可能是Bug 引起的，比如数组越界等。而其他常见的语言一般是没有没有区分 `recoverable error`和 `unrecoverable error`的. 比如 Python, 用的就是 `Exception`.

而Rust 是没有 `Exception`, Rust 用 `Result<T, E>` 表示可恢复错误， 用 `panic!()` 来表示出现错误，并且中断程序的执行并退出(不可恢复错误)。

`Result` 是Rust 标准库的枚举：

```rust
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T`和`E`都是泛型，`T`表示程序执行正常的时候的返回值，那`E`自然是程序出错时的返回 值。以标准库的打开文件的函数为例， `std::io::File` 的 `open()` 函数的签名如下：

```rust
pub fn open<P: AsRef<Path>>(path: P) -> io::Result<File> {
    OpenOptions::new().read(true).open(path.as_ref())
}
```

忽略这个方法的参数，只看返回值类型：`io::Result<File>`, 又因为有 `type Result<T>` Result&lt;T, Error&gt;;=

这个 `typedef` 语句，所以返回值的完整版本时`io::Result<File,io::Error>`, 即调用 `open` 这个函数的时候，可能出现错误，出现错误时候返回一个 `io::Error`, 如果调用`open`没有问题的话，就会返回一个 `File` 的结构体，所以这个就类似 Java 的`CheckedException`,

只要声明了函数可能出现问题，在调用函数的时候就必须处理可能出现的错误，不然编译器就不会让你通过(Rust 的编译器就像位父亲那样对开发者耳提面命), 例如：

```rust
match File::open(&self.cache_path) {
    Ok(file) => println!("{:?}",file),
    Err(why) => {
	panic!("couldn't open {:?}", why.description())
    }
};
```


## <span class="section-num">5</span> Java 的异常传递 {#java-的异常传递}

在程序中，总会有一些错误需要处理，但是却不应该在错误出现的函数进行处理的情况(或者是，你很懒惰，只想应付一下编译器，不想处理出现的异常 :)

比如你正在编写一个类 库，里面有很多的IO 操作，有IO 操作的地方就有可能出现`IOException`. 如果出现异常，
你不应该自己在类库把异常给 `try-catch`了，如果这样，使用你类库的开发者就没办法知 道程序出现了异常，异常的堆栈也丢了。

比较合理的做法是，把`IOException`捕捉了，然后对 `IOException` 做一层包装，然后再抛给类库的调用者，例如：

```java
public void doSomething() throws WrappingException{
    try{
	doSomethingThatCanThrowException();
    } catch (SomeException e){
	e.addContextInformation("there is something happen in doSomething() function, `Some Exception` is raised, balabala");
	//throw e;  //throw e, or wrap it  see next line.
	throw new WrappingException(e, more information about Some Exception, balabala);
    } finally {
	//clean up close open resources etc.
    }
}
```

当然，你也可以在添加了额外的信息之后，直接把原来的异常抛出来


## <span class="section-num">6</span> Rust 的异常传递 {#rust-的异常传递}

刚刚谈了 Java 的异常传递，现在轮到 Rust 的异常传递了，既然Rust 没有 `Exception`一说，那 Rust 传递的自然也是 `Result<T,E>`
这个枚举类型(这里针对的是 可恢复错误，不可恢复错误出现错误的时候，会返回错误并弹出程序，自然不存在异常传递).

先来看看 Rust 的异常传递的例子：

```nil
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
	Ok(file) => file,
	Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
	Ok(_) => Ok(s),
	Err(e) => Err(e),
    }
}
```

例子来自 Rust Book

先来看看函数的返回值 `Result<String,io::Error>`, 也就是说， `read_username_from_file` 正确执行的时候返回是 `String`,
错误的时候，返回的是 `io::Error`. 这里的异常传递是在出现 `io::Error`的时候，将错误原样返回，不然就是返
回函数执行成功的结果。

就异常传递的方式而言，Rust 和 Java 是大同小异：声明可能抛出的异常和成功时返回的结果，然后在遇到错误的时候，直接（或者包装一下）返回错误。


### <span class="section-num">6.1</span> ? 关键字 {#关键字}

虽说 Rust 的异常处理很清晰，但是每次都要 `match` 然后返回未免太繁琐了，所以 Rust 提供了一个语法糖来显示繁琐的异常传递：用
"?" 关键字进行异常传递：

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

同样的功能，但是模板代码却减少了很多 :)


### <span class="section-num">6.2</span> unwrap 和 expect {#unwrap-和-expect}

虽说 Rust 的可恢复错误设计得很优雅，但是每次遇到可能出现错误得地方都要显示地进行 处理，不免让人觉得繁琐.

Rust 也考虑到这种情况了，提供了 `unwrap()` 和 `expect()`让你舒心~~简单粗暴~~地处理错误：在函数调用成功的时候返回正确的结果，在 出现错误地时候直接 `panic!()`,并退出程序


#### <span class="section-num">6.2.1</span> unwrap {#unwrap}

```rust
fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

打开 `hello.txt`这个文件，能打开就返回文件 `f`,不能打开就 `panic!()` 然后退出程序。

```log
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```


#### <span class="section-num">6.2.2</span> expect {#expect}

`expect()`和 `unwrap()`类似，只不过 `expect()`可以加上额外的信息：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

出现错误的时候，除了显示应有的错误信息之外，还会显示你自定义的错误信息：

```log
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

以上代码来自 [Rust book](https://doc.rust-lang.org/book/second-edition/ch09-02-recoverable-errors-with-result.html)


## <span class="section-num">7</span> 结语 {#结语}

以上只是浅谈了 Rust 的错误处理，以及和 Java 的异常处理机制的简单比较，接下来我会 谈谈如何自定义`Error`以及使用
`erro_chain` 这个库来优雅地进行错误处理 :)

如果想了解更多关于 Rust 异常处理的内容，可以查阅 Rust book [Error handle](https://doc.rust-lang.org/book/second-edition/ch09-00-error-handling.html)


## <span class="section-num">8</span> 参考 {#参考}

-   [propagating exceptions](http://tutorials.jenkov.com/exception-handling-strategies/propagating-exceptions.html)
-   [Rust book](https://doc.rust-lang.org/book/second-edition/)
-   [IO Result](https://doc.rust-lang.org/stable/std/io/type.Result.html)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

