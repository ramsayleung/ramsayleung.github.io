+++
title = "RSpotify– 我的第一个Rust crate"
date = 2018-02-28T20:44:00-08:00
lastmod = 2024-12-30T22:28:21-08:00
tags = ["rust", "rspotify"]
categories = ["rust"]
draft = false
toc = true
showQuote = true
+++

开发第一个Rust crate 的感受和踩到的坑

最近写了人生第一个 Rust crate -- [RSpotify](https://github.com/ramsayleung/rspotify).

虽说并不是什么惊天地，泣鬼神的大作，但是也是我花费了近两个月实现的。

现在就来聊聊这个开发过程的感悟和踩到的坑


## <span class="section-num">1</span> 感悟 {#感悟}


### <span class="section-num">1.1</span> 函数的缺省值 {#函数的缺省值}

因为我是参考着 Python 版本的 Spotify API SDK 来写 rspotify的，Spotify 某些API 需要请求的时候附加上默认值，例如在获取一个歌手最热的10首歌的时候需要指定country.

因为Python 的函数是有缺省参数的，所以用 python 来实现就很方便

```python
def artist_top_tracks(self, artist_id, country='US'):
    """ Get Spotify catalog information about an artist's top 10 tracks
        by country.

        Parameters:
            - artist_id - the artist ID, URI or URL
            - country - limit the response to one particular country.
    """

    trid = self._get_id('artist', artist_id)
    return self._get('artists/' + trid + '/top-tracks', country=country)
```

但是用 Rust 来实现的时候，问题就来了，因为Rust 是没有缺省参数的。而Rust 处理缺省
参数的策略一般是=Builder Pattern=:

```rust
struct Part1 {
    points: u32,
    tf: f64,
    dt: f64
}

impl Part1 {
    fn new() -> Part1 {
        Part1 { points: 30_u32, tf: 3_f64, dt: 0.1_f64 }
    }

    fn tf(mut self, tf: f64) -> Self {
        self.tf = tf;
        self
    }
    fn points(mut self, points: u32) -> Self {
        self.points = points;
        self
    }
    fn dt(mut self, dt: f64) -> Self {
        self.dt = dt;
        self
    }
    fn run(self)  {
        // code here
        println!("{:?}", self);
    }
}

//调用函数
Part1::new().points(10_u32).run();
Part1::new().tf(7_f64).dt(15_f64).run();
```

具体情况具体分析，就 rspotify 而言， `Builder Pattern` 并不适用，因为 rspotify
有很多函数都需要缺省参数，而不同函数的缺省值可能又不一样。

例如，有些函数的 `offset=参数是 0, 而另外一些函数的 =offset` 参数是1. 为此，我还在 [Reddit](https://www.reddit.com/r/rust/comments/7u1zjl/question_about_default_values_for_function/) 发贴询问意见，[PM_ME_WALLPAPER](https://www.reddit.com/user/PM_ME_WALLPAPER) 建议我用=Into&lt;Option&lt;T&gt;&gt;=:

```rust
fn foo<T: Into<Option<usize>>>(limit: T) {
    let limit = limit.into().unwrap_or(10);
    …
}
```

在他的建议下，我把 `artist_top_tracks()` 修改成：

```rust
pub fn artist_top_tracks(
    &self,
    artist_id: &mut str,
    country: impl Into<Option<String>>,
) -> Option<FullTracks> {
    let mut params: HashMap<&str, String> = HashMap::new();
    params.insert("country", country.into().unwrap_or("US".to_owned()));
    let trid = self.get_id(Type::Artist, artist_id);
    let mut url = String::from("artists/");
    url.push_str(&trid);
    url.push_str("/top-tracks");
    match self.get(&mut url, &mut params) {
        Some(result) => {
            // let mut albums: Albums = ;
            match serde_json::from_str::<FullTracks>(&result) {
                Ok(_tracks) => Some(_tracks),
                Err(why) => {
                    eprintln!("convert albums from String to Albums failed {:?}", why);
                    None
                }
            }
        }
        None => None,
    }
}
```

虽说不如Python 那样优雅，但是看起来还是不错滴


### <span class="section-num">1.2</span> 错误处理 {#错误处理}

对于一个 library 而言，错误处理是设计的重要一环。

因为我之前只有开发应用的经验， 而开发应用的错误处理和开发类库的错误处理显然需要考虑的东西不一样，所以我还谨慎思
考过这个问题。后来，我决定不处理调用Spotify API 或者其他操作导致的错误，将错误进行一次包装(wrap), 然后再返回给library 的调用者。

最开始的时候，我是自己定义错误类型的，后来觉得过于累赘，就用上=error_chain=. 用上 error_chain 之后， =errors.rs=这个文件也非常简单：

```rust
///The kind of spotify error.
use serde_json;

error_chain! {
    errors {}

    foreign_links {
        Json(serde_json::Error) #[doc = "An error happened while serializing JSON"];
    }
}
```

而刚刚我提到了只对错误作简单的包装，得益于 =error_chain=的设计，这个特性也很容易实现：

```rust
pub fn convert_result<'a, T: Deserialize<'a>>(&self, input: &'a str) -> Result<T> {
    let result = serde_json::from_str::<T>(input)
        .chain_err(|| format!("convert result failed, content {:?}",input))?;
    Ok(result)
}
```

这个函数是将 Spotify 的响应体映射成对应的 object(例如 playlist, album 等). 如果转换过程出错了，那么就返回=convert result failed, content {:?}=错误信息之后，返回 `serde_json` 转换时出现的错误信息。


### <span class="section-num">1.3</span> Reddit+clippy {#reddit-plus-clippy}

剩下的是在纠结定义一个函数传参的时候是传值，参数是 mutable 还是 immutable, 以及
其他类似的考虑。

或许 Effective Rust 和 More Effective Rust 出现之后，我读完就知道什么样的设计才是 best practice. 因为有诸多设计的不确定，所以在完成rspotify 90% 的代码量之后，我在 [Reddit](https://www.reddit.com/r/rust/comments/7xn9mh/my_first_crate_rspotify_spotify_api_wrapper/)
上发贴，邀请社区的同学来 review code 以帮我完善代码。

他们的确给了我很多建议，我也根据他们的建议修改 rspotify. 在经过人肉 code review 之后，是时候祭出
[clippy](https://github.com/rust-lang-nursery/rust-clippy) 这个大杀器， clippy 就代码的编写给出了非常多的建议，比如将函数 `Vec<String>` 的参数类型修改成
`&[String]`, 因为函数并没有使用(consume) 这个参数，所以传引用比传值更合适，类似 的建议不胜枚举。

最后在 clippy 的建议下， 我几乎将所有的 clippy warning 都消除掉。 邀请别人经常帮你 review code 有点不实际，但是 clippy 确是不会因为帮你审查代码而感到厌烦的，真的是非常强大的工具


## <span class="section-num">2</span> 坑 {#坑}


### <span class="section-num">2.1</span> Debugger {#debugger}

虽说 Rust 也有Debugger-- gdb-rust. gdb 我以前写c 的时候用过，gdb 熟悉程度虽然谈 不上精通，但是也能熟练使用。但是用gdb-rust 调试并不是非常便利，比如在使用 [Rocket](https://rocket.rs/) 这个Web框架的时候，就很难使用gdb来调试Web程序。

虽说 [intellij-rust](https://intellij-rust.github.io/) 这个 Intellij Idea 的插件也支 持Debugger, 但是只有配合[Clion](https://www.jetbrains.com/clion/)才能使用。因为 只有 Clion 才能调用 gdb, 无奈。所以在开发 rspotify 的时候，我用得都是 =println!()=调试大法。


### <span class="section-num">2.2</span> 编译器Bug {#编译器bug}

战战兢兢地开发着，终于到发布到 [crates.io](https://crates.io/) 的大喜日子了，怎知在发布之后一直没办法看到生成的文档，本地不是一切正常么？

后来在[社区](https://www.reddit.com/r/rust/comments/7yrofg/rspotify_is_published_to_cratesio/) 同学的提醒下， 我才发现我踩到了 Rust 编译器的一个bug, 最后我就顺手提交了一个 [issue](https://github.com/rust-lang/rust/issues/48368), 虽说这个问题已经在 `nightly` 里面修复了。


## <span class="section-num">3</span> 结语 {#结语}

前后两个月的时间，终于发布了 rspotify. 项目不大，但是也是我花费时间，精力去开发的，也得到其他同学的肯定，喔耶 :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

