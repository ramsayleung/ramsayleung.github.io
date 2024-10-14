+++
title = "测试技能进阶(三): Property Based Testing"
date = 2024-10-14T09:37:00-07:00
lastmod = 2024-10-14T14:41:38-07:00
tags = ["testing", "rust"]
categories = ["testing", "rust"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}


### <span class="section-num">1.1</span> test case的局限 {#test-case的局限}

想要更好地理解什么是 Property based testing, 就来先看下已有 test case 的局限，再来观察它解决了什么问题。 <br/>

用之前[《测试技能进阶(二): Parameterized Tests》](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/)中计算折扣的函数为例： <br/>

```python
def calculate_discount(price, discount_percentage):
    if price < 0:
        raise ValueError(f"Price must be greater than zero: {price}")
    if discount_percentage < 0:
        raise ValueError(f"Discount_percentage must be greater than zero: {discount_percentage}")
    if price > 50000:
        return price - (price * (discount_percentage * 1.15) / 100) 
    elif price > 100000:
        return price - (price * (discount_percentage * 1.18) / 100) 
    else:
        return price - (price * discount_percentage / 100)
```

即使我们使用了 Parameterized Test, 把测试逻辑和测试数据集作了分离，但是还是有两个缺点： <br/>

1.  我们的测试数据集还是要手工构造，即使现在不需要写新的 test case, 手工构造数据集还是很麻烦 <br/>
2.  第二个问题更严重，就是我们的构建的数据集可能不是完备的，如果数据集没有办法覆盖所有的条件分支，那我们仍然可能发现不了代码中的Bug <br/>


## <span class="section-num">2</span> Property Based Testing {#property-based-testing}

而 Property Based Testing 就是想解决这个问题，它希望可以结合人脑对特定问题域的理解和机器的运算能力，使用更少的时间来生成更优的测试case. <br/>

Property Based Testing 这个概念是由 Haskell 项目 [QuickCheck](https://en.wikipedia.org/wiki/QuickCheck) 在1999年引入的，它的理念是，程序员应该只定义某个测试case, 参数需要满足的标准(specification), 然后程序就会自动生成大量满足这个标准的随机数，用这些随机数来测试这个 test case。 <br/>

而因为测试数据是随机生成的，所以你意料之内的数据，或者意料之外的数据都会被用来测试， <br/>
既省去了费时费力构造不同数据作数据集来测试的烦恼，又能保证数据集的完备性, 经常可以帮助你发现意想不到的bug. <br/>

这就是声明式定义的一种，你只需要声明你想干什么(用什么样的数据测试什么函数)，而非命令式定义（你需要定义你要怎么做）. <br/>

人力应该是很珍贵，而机器的计算资源却是很便宜，应该让机器代替人去做生成数据的事。 <br/>

举例来说, 以上面的 `calculate_discount` 函数为例，如果我们告诉程序, `price` 和 `discount_percentage` 应该是整数（specification）, 那么 Quickcheck 就会生成各种整数, 从 Integer.Min 到 Integer.Max 不等，用来测试我们的程序. <br/>

如果还是觉得这个概念比较抽象，可以来看下具体的例子： <br/>


## <span class="section-num">3</span> Hypothesis {#hypothesis}

Python Property Based Testing的测试框架叫 [Hypothesis](https://hypothesis.works/)(假想)，这个项目名字也是起得非常有水平，结合Property Based Testing的哲学，可谓信雅达. <br/>

假设我们现在要实现一个简单的数据压缩的算法： [Run-length Encoding](https://en.wikipedia.org/wiki/Run-length_encoding)(RLE)，通常用于压缩包含连续重复数据的序列, 这种编码方法特别适用于那些有大量重复字符或值的数据. <br/>

它的基本原理是： <br/>

1.  统计连续重复的数据元素的数量。 <br/>
2.  用一个计数值和数据值的组合来替代这些重复的数据。 <br/>

比如字符串: `AABBBCCCC`, RLE 编码后: `2A3B4C`. `2A` 表示两个连续的 `A`, `3B` 表示三个连续的 `B`, `4C` 表示四个连续的 `C` 。 <br/>

Python实现如下: <br/>

```python
def encode(input_string):
    count = 1
    prev = ""
    lst = []
    for character in input_string:
        if character != prev:
            if prev:
                entry = (prev, count)
                lst.append(entry)
            count = 1
            prev = character
        else:
            count += 1
    entry = (character, count)
    lst.append(entry)
    return lst


def decode(lst):
    q = ""
    for character, count in lst:
        q += character * count
    return q
```

如果我们的代码实现没有问题的话，对于任意的字符串，编码后的字符串，解码后的结果应该和原来的字符串一致的，这个就是我们的测试逻辑: <br/>

```python
from hypothesis import given
from hypothesis.strategies import text


@given(text()) # 入参的标准是：任意的字符串，hypothesis 框架就会自动生成随机数，并调用test_decode_inverts_encode
def test_decode_inverts_encode(s):
    assert decode(encode(s)) == s
```

使用 `pytest` 运行上面的用例，结果如下: <br/>

```shell
> pytest property_based_testing.py
=================================== test session starts ====================================
platform darwin -- Python 3.12.6, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/ramsayleung/code/python/test_technique
plugins: hypothesis-6.115.0
collected 1 item                                                                           

property_based_testing.py F                                                          [100%]

========================================= FAILURES =========================================
________________________________ test_decode_inverts_encode ________________________________

@given(text())
>   def test_decode_inverts_encode(s):

property_based_testing.py:29: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
property_based_testing.py:30: in test_decode_inverts_encode
assert decode(encode(s)) == s
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

input_string = ''

def encode(input_string):
count = 1
prev = ""
lst = []
for character in input_string:
                 if character != prev:
                    if prev:
                       entry = (prev, count)
                       lst.append(entry)
                       count = 1
                       prev = character
                       else:
                       count += 1
                       >       entry = (character, count)
                       E       UnboundLocalError: cannot access local variable 'character' where it is not associated with a value
                       E       Falsifying example: test_decode_inverts_encode(
                           E           s='',
                           E       )

                       property_based_testing.py:17: UnboundLocalError
                       ================================= short test summary info ==================================
                       FAILED property_based_testing.py::test_decode_inverts_encode - UnboundLocalError: cannot access local variable 'character' where it is not associated ...
                       ==================================== 1 failed in 0.14s =====================================
```

可以看到，当 `input_string =''` 是空字符串的时候， `encode` 函数抛出异常了，说 `character` 变量未定义。原来是 `encode` 函数没有对空字符串这个 corner case 作处理，那么就加个判断条件，修复一下: <br/>

```python
def encode(input_string):
    if not input_string:
        return []
    count = 1
    prev = ""
    lst = []
    for character in input_string:
        if character != prev:
            if prev:
                entry = (prev, count)
                lst.append(entry)
            count = 1
            prev = character
        else:
            count += 1
    entry = (character, count)
    lst.append(entry)
    return lst
```

既然我们知道空字符串是个特殊的 case, 因为 hypothesis 生成的都是任意的随机数，不一定每次都会测到空字符串，那我们就自己指定一个 case: <br/>

```python
from hypothesis import example, given, strategies as st


@given(st.text())
@example("") # 手工指定空字符串这个 corner case 
def test_decode_inverts_encode(s):
    assert decode(encode(s)) == s
```

`pytest` 重新运行，测试就通过了。但是，对 `hypothesis` 框架还没有建立信心的你我就不确定，它是否真的生成很多随机来运行这个 test case 呢？ <br/>

有两个方法可以验证： <br/>

方法一：最简单粗暴的方式，把 `s` 变量给打印出来，毕竟眼见为实: <br/>

```python
@given(st.text())
@example("")
def test_decode_inverts_encode(s):
    print(s)
    assert decode(encode(s)) == s
```

然后通过 `pytest -s` 参数要求 `pytest` 将写入到 `stdout` 的内容给打印出来 <br/>

```shell
> pytest property_based_testing.py -s
======================================= test session starts =======================================
platform darwin -- Python 3.12.6, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/ramsayleung/code/python/test_technique
plugins: hypothesis-6.115.0
collected 1 item                                                                                  

property_based_testing.py 

O


¶
\å񢄏«
𥛗Îbó
𜆮å
񰘰9
gah󭾔𛧁

i򼯜+ó»򮩸b񝕨
S!ÕTå&𰵩í¤ýäó÷F
øôyµ
Äª
sLz$ï
_𠵈
Ü
A

R󃝷{©¾

   ìõ
   æ􂐛BÝ1*􅄢ëóg𮎈¼ ?𩓁
   Òör @PP􎾂ö񳱊ûÁ½¬HÈ6#
   a𣽗¶󿅌𧑁x~󗜬韹ûð󴯮#Z󅖫\©𳖅ûf>
   i
   ....
   ======================================== 1 passed in 0.15s ========================================
```

这一堆都是什么字符呢, 都乱码了。 <br/>

毕竟我们告诉 `hypothesis` 框架的是，我们参数接受的标准是任意的字符串， `hypothesis` 就非常尽职地帮我们生成了各种字符串，这个测试数据集可比我们自己手工构建的范围大得多，这就是 property based testing 的优势所在. <br/>

第二种方法是使用 `hypothesis` 框架提供的命令行参数 =--hypothesis-show-statistics=，用于打印统计信息: <br/>

```shell
> pytest property_based_testing.py --hypothesis-show-statistics
======================================= test session starts =======================================
platform darwin -- Python 3.12.6, pytest-8.3.3, pluggy-1.5.0
rootdir: /Users/ramsayleung/code/python/test_technique
plugins: hypothesis-6.115.0
collected 1 item                                                                                  

property_based_testing.py .                                                                 [100%]
====================================== Hypothesis Statistics ======================================

property_based_testing.py::test_decode_inverts_encode:

- during generate phase (0.03 seconds):
- Typical runtimes: < 1ms, of which < 1ms in data generation
- 100 passing examples, 0 failing examples, 0 invalid examples

- Stopped because settings.max_examples=100


======================================== 1 passed in 0.05s ========================================
```

上面运行了 100 条数据，如果你觉得还想跑更多，可以通过 `settings` 装饰器指定更多: <br/>

```python
@settings(max_examples=500)
```


## <span class="section-num">4</span> Quickcheck &amp; Proptest {#quickcheck-and-proptest}

而在Rust生态，就有两个 Property Based Testing 的库，一个是由Rust社区知名开发者，[ripgrep](https://github.com/BurntSushi/ripgrep)和 regex 库作者移植自 Haskell Quickcheck 库的 [quickcheck](https://github.com/BurntSushi/quickcheck) (名字也一并移植了), 另外一个是思路继承自 Python Hypothesis 的 [Proptest](https://github.com/proptest-rs/proptest)(这位直接用property based testing技术来命名了，不得不说，命名真的是门艺术) <br/>

两者的社区接受度都相差无几(star, 使用者数量), 而在公司内部，我也发现 quickcheck 和 proptest 都有人用，坐我旁边的Principle Engineer 用的是 proptest, 而另外一个现在和我共事的同事，她的之前团队用的就是 quickcheck，看到都势均力敌嘛。 <br/>

翻开 quickcheck 和 proptest 的API 文档之后，我发现我更喜欢 quickcheck 的接口风格，虽说它的活跃度更低一些，我最后还是选择了使用 quickcheck. <br/>

下面就来介绍一下我在Rust上使用 quickcheck 的心得: <br/>

假设我们现在有一个可以反转列表的函数 `reverse`: <br/>

```rust
fn reverse<T: Clone>(xs: &[T]) -> Vec<T> {
    let mut rev = vec!();
    for x in xs {
        rev.insert(0, x.clone())
    }
    rev
}
```

对于任意类型的列表，反转之后现反转的结果，肯定是和原结果一样的，那么我们就可以开始声明我们的标准(specification), 那就是任意的列表，可以是字符串列表，整型列表或者是其他的结构体列表: <br/>

```rust
#[cfg(test)]
mod tests{
    use quickcheck_macros::quickcheck;
    use crate::reverse;

    #[quickcheck]
    fn double_reversal_is_identity_isize(xs: Vec<isize>) -> bool {
        xs == reverse(&reverse(&xs))
    }

    #[quickcheck]
    fn double_reversal_is_identity_string(xs: Vec<String>) -> bool {
        xs == reverse(&reverse(&xs))
    }
}
```

Rust 的unit test 是不支持带参数的，=#[quickcheck]= 这个宏就会自动将 `double_reversal_is_identity_isize` 转换成 property based test case, 而得益于Rust的类型系统, `quickcheck` 就能推断出入参就是我们声明的标准 `Vec<isze>`, 任意 `isize` 类型的数组. <br/>


### <span class="section-num">4.1</span> Struct with quickcheck {#struct-with-quickcheck}

如果上面的例子觉得过于简单的话，现在就让我们看个复杂一点的例子, 一个简单的图书管理系统，支持会员，借书，还书功能： <br/>

```rust
use chrono::{Duration, NaiveDate};
use std::collections::HashMap;

#[derive(Debug, Clone, PartialEq)]
struct Book {
    isbn: String,
    title: String,
    author: String,
    publication_year: u16,
}

#[derive(Debug, Clone, PartialEq)]
struct Member {
    id: u32,
    name: String,
    email: String,
}

#[derive(Debug, Clone)]
struct Loan {
    book_isbn: String,
    member_id: u32,
    due_date: NaiveDate,
}

#[derive(Debug, Clone)]
struct Library {
    books: HashMap<String, Book>,
    members: HashMap<u32, Member>,
    loans: Vec<Loan>,
    current_date: NaiveDate,
}

impl Library {
    fn new(current_date: NaiveDate) -> Self {
        Library {
            books: HashMap::new(),
            members: HashMap::new(),
            loans: Vec::new(),
            current_date,
        }
    }

    fn add_book(&mut self, book: Book) -> Result<(), String> {
        if self.books.contains_key(&book.isbn) {
            Err("Book with this ISBN already exists".to_string())
        } else {
            self.books.insert(book.isbn.clone(), book);
            Ok(())
        }
    }

    fn add_member(&mut self, member: Member) -> Result<(), String> {
        if self.members.contains_key(&member.id) {
            Err("Member with this ID already exists".to_string())
        } else {
            self.members.insert(member.id, member);
            Ok(())
        }
    }

    fn loan_book(&mut self, book_isbn: &str, member_id: u32) -> Result<(), String> {
        if !self.books.contains_key(book_isbn) {
            return Err("Book not found".to_string());
        }
        if !self.members.contains_key(&member_id) {
            return Err("Member not found".to_string());
        }
        if self.loans.iter().any(|loan| loan.book_isbn == book_isbn) {
            return Err("Book is already on loan".to_string());
        }
        let due_date = self.current_date + Duration::days(14);
        self.loans.push(Loan {
            book_isbn: book_isbn.to_string(),
            member_id,
            due_date,
        });
        Ok(())
    }

    fn return_book(&mut self, book_isbn: &str) -> Result<(), String> {
        if let Some(index) = self
            .loans
            .iter()
            .position(|loan| loan.book_isbn == book_isbn)
        {
            self.loans.remove(index);
            Ok(())
        } else {
            Err("Book is not currently on loan".to_string())
        }
    }
}
```

通过上面的简单代码，就实现了新增图书，新增会员，借书，和还书功能。现在就让我们来结合 `quickcheck` 的 `Arbitrary` 接口，实现生成任意的图书和会员，以便用于测试: <br/>

```rust
use quickcheck::{Arbitrary, Gen};
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

impl Arbitrary for Member {
    fn arbitrary(g: &mut Gen) -> Self {
        Member {
            id: u32::arbitrary(g), // 任意大于0，小于uint32.max_value的整型
            name: String::arbitrary(g), // 任意字符串
            // 任意字符开头, 以@example.com 结尾的字符
            email: format!("{}@example.com", String::arbitrary(g)), 
        }
    }
}
```

现在就让我们来看下借助 quickcheck 编写的 test case, 注意参数为 `Book` 和 `Member` 类型的 case, quickcheck 就会以我们上面定义的标准，自动给我们生成符合规定的 `Book` 和 `Member` 参数. <br/>

```rust
#[cfg(test)]
mod tests {
    use chrono::NaiveDate;
    use quickcheck_macros::quickcheck;

    use crate::book::Member;

    use super::{Book, Library};

    #[quickcheck]
    fn adding_book_increases_book_count(book: Book) -> bool {
        let mut library = Library::new(NaiveDate::from_ymd_opt(2024, 10, 14).unwrap());
        let initial_count = library.books.len();
        library.add_book(book.clone()).unwrap();
        library.books.len() == initial_count + 1 && library.books.contains_key(&book.isbn)
    }

    #[quickcheck]
    fn cannot_loan_nonexistent_book(book_isbn: String, member_id: u32) -> bool {
        let mut library = Library::new(NaiveDate::from_ymd_opt(2024, 10, 14).unwrap());
        library.loan_book(&book_isbn, member_id).is_err()
    }

    #[quickcheck]
    fn can_return_loaned_book(book: Book, member: Member) -> bool {
        let mut library = Library::new(NaiveDate::from_ymd_opt(2024, 10, 14).unwrap());
        library.add_book(book.clone()).unwrap();
        library.add_member(member.clone()).unwrap();
        library.loan_book(&book.isbn, member.id).unwrap();
        library.return_book(&book.isbn).is_ok()
    }
}
```

通过 `quickcheck` 我们就可以只专注测试逻辑，可以假定测试数据集是完备的了。可能看到 `Book` 和 `Member`, 你会觉得 quickcheck 并没有做太多事情，你手工也可以构造。 <br/>

但是我在的实际工作中，我就需要构造一个超过23个成员变量的 struct, 大部分还是 optional, 然后需要将这个 struct 写入到 parquet 文件，然后再测试读取逻辑。 <br/>
不同成员变量的值可取的范围实在太多了，再叠加上 optional 的可能性，构造数据的代码写得相当恶心. <br/>

所以有了 quickcheck 之后，我只需要为这个 struct 实现 `Arbitrary` 接口，剩下的就由 `quickcheck` 替我生成，所以我直接和PE大佬说: <br/>

> property test saves me life, now I couldn't live without it. <br/>


## <span class="section-num">5</span> 总结 {#总结}

本来想抒发感想写点结语，但是看到 Hypothesis 作者写的 [The purpose of Hypothesis](https://hypothesis.readthedocs.io/en/latest/manifesto.html) 来说明他开发的 Hypothesis 的动机，他的文章甚至用来给这个《测试技能进阶》系列总结都相当妥当。 <br/>

我就试翻译下他文章的部分段落, 更推荐阅读原文，可谓是用心良苦，字字珠玑: <br/>

> 请容我狂妄一下，Hypothesis 的目标是希望可以让这个世界迈进到一个全新，由高质量软件打造的新世代。 <br/>
> 
> 正如人们所说，软件正在吞噬整个世界。但软件本身却很烂，它充满bug，又不安全，还经常被设计得很烂，这样的软件可谓是万恶之源. <br/>
> 
> 而软件测试的状况甚至更糟糕，虽然大家都认同应该对代码进行测试，但是你能问心无愧地说，你经手过的代码都有被充分测试么？ <br/>
> 
> 问题在于，实在是太难写出好的测试了， <br/>
> ****你写测试用例的时候，通常持有和你写代码时一样的假设与误区，你写的测试用例自然无法发现你当初埋下的bug**** (精辟) <br/>
> 
> 与此同时，有各种各样让测试变成更好的工具却基本无人使用，最初的 Quickcheck 是1999年推出的，但是大多数开发者甚至从未听说过它，更别提使用了（开山始祖的Quickcheck在GitHub只有700多个Star，就知道作者所言不虚）。 <br/>
> 虽然其他语言有些半成品的实现，但是大部分都不值得一试。 <br/>
> 
> 而 Hypothesis 的目标正是正本清源，把先进的测试技术传递给大众，并提供一个高质量的实现，让人们可以接纳它。 <br/>
> 
> 希望可以集百家之所长，附以个人微薄之力，让软件测试变得更好。 <br/>

系列文章: <br/>

-   [测试技能进阶(一): 软件质量认知](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%B8%80_%E8%BD%AF%E4%BB%B6%E8%B4%A8%E9%87%8F%E8%AE%A4%E7%9F%A5/) <br/>
-   [测试技能进阶(二): Parameterized Tests](https://ramsayleung.github.io/zh/post/2024/%E6%B5%8B%E8%AF%95%E6%8A%80%E8%83%BD%E8%BF%9B%E9%98%B6%E4%BA%8C_parameterized_tests/) <br/>


## <span class="section-num">6</span> 参考 {#参考}

-   [Haskell Quickcheck](https://hackage.haskell.org/package/QuickCheck) <br/>
-   [Hypothesis](https://hypothesis.works/) <br/>
-   [Proptest](https://github.com/proptest-rs/proptest) <br/>
-   [Rust quickcheck](https://github.com/BurntSushi/quickcheck) <br/>

