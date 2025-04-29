+++
title = "rspotify has come to async/await"
date = 2020-02-28T01:27:00+08:00
lastmod = 2022-02-25T20:54:09+08:00
tags = ["rust", "rspotify"]
categories = ["rust"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> Preface {#preface}

Today, I am exited to introduce you the [v0.9](https://github.com/ramsayleung/rspotify/releases/tag/v0.9) release I have been continued to work on it for the past few weeks that
adds `async/await` support now!


## <span class="section-num">2</span> The road to async/await {#the-road-to-async-await}

What is rspotify: &gt; For those who has never heared about rspotify
before, [rspotify](https://github.com/ramsayleung/rspotify) is a
Spotify web Api wrapper implemented in Rust.

With async/await's forthcoming stabilization and reqwest adds
`async/await` support now, I think it's time to let rspotify leverage
power from `async/await`. To be honest, I was not familiar with
`async/await` before, because of my Java background from where I just
get used to multiple thread and sync stuff(Yes, I know Java has `future`
either).

After reading some good learning resources, such as [Async book](https://rust-lang.github.io/async-book/), [Zero-cost Async IO](https://www.youtube.com/watch?v=skos4B5x7qE), I
started to step into the world of `async/await`. `async/await` is a way
to write functions that can "pause", return control to the runtime, ant
then pick up from where they left off.

I think perhaps the most important part of `async/await` is runtime, which defines how to
schedule the functions.

Now, by leveraging the `async/await` power of `reqwest`, rspotify could
send HTTP request and handle response asynchronously.

Futhermore, not only do I refactor the old blocking endpoint functions to `async/await`
version, but also keep the old blocking endpoint functions with a new
additional feature `blocking`, then other developers could choose API to
their taste.


## <span class="section-num">3</span> Overview {#overview}

`album` example:

```rust

use rspotify::client::Spotify;
use rspotify::oauth2::SpotifyClientCredentials;

#[tokio::main]
async fn main() {
    // Set client_id and client_secret in .env file or
    // export CLIENT_ID="your client_id"
    // export CLIENT_SECRET="secret"
    let client_credential = SpotifyClientCredentials::default().build();

    // Or set client_id and client_secret explictly
    // let client_credential = SpotifyClientCredentials::default()
    //     .client_id("this-is-my-client-id")
    //     .client_secret("this-is-my-client-secret")
    //     .build();
    let spotify = Spotify::default()
	.client_credentials_manager(client_credential)
	.build();
    let birdy_uri = "spotify:album:0sNOF9WDwhWunNAHPD3Baj";
    let albums = spotify.album(birdy_uri).await;
    println!("{:?}", albums);
}
```

Just change the default API to async, and moving the previous
synchronous API to `blocking` module.

Notes that I think the v0.9 release of rspotify is going to be a huge
break change because of the support for `async/await`, which definitely
breaks backward compatibility.

So I decide to make an other break change
into the next release, just refactoring the project structure to shorten
the import path:

before:

```rust
use rspotify::spotify::client::Spotify;
use rspotify::spotify::oauth2::SpotifyClientCredentials;
```

after:

```rust
use rspotify::client::Spotify;
use rspotify::oauth2::SpotifyClientCredentials;
```

the `spotify` module is unnecessary and inelegant, so I just remove it.


## <span class="section-num">4</span> Conclusion {#conclusion}

[rspotify v0.9](https://github.com/ramsayleung/rspotify/releases/tag/v0.9) is now available! There is [documentation](https://docs.rs/crate/rspotify/), [examples](https://github.com/ramsayleung/rspotify/tree/master/examples) and an [issue](https://github.com/ramsayleung/rspotify/issues/new)
tracker!

Please provide any feedback, as I would love to improve this library any way I can! Thanks [@Alexander](<https://github.com/Rigellute>) so much for actively participate in the refactor work for support
`async/await`.

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

