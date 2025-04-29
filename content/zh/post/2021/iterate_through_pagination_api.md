+++
title = "Let's make everything iterable"
date = 2021-04-29T11:48:00+08:00
lastmod = 2022-02-24T14:38:19+08:00
tags = ["rust", "rspotify"]
categories = ["rust"]
draft = false
toc = true
showQuote = true
+++

Iterate through pagination in the Rest API


## <span class="section-num">1</span> Preface {#preface}

About 4 months ago, [icewind1991](https://github.com/icewind1991) created an exciting [PR](https://github.com/ramsayleung/rspotify/pull/166) that adding `Stream/Iterator` based versions of methods with paginated results, which makes enpoints in [Rspotify](https://github.com/ramsayleung/rspotify) more much ergonomic to use, and [Mario](https://github.com/marioortizmanero) completed this PR.

In order to know what this PR brought to us, we have to go back to the orignal story, the paginated results in Spotify's Rest API.


## <span class="section-num">2</span> Orignal Story {#orignal-story}

Taking the `artist_albums` as example, it gets Spotify catalog information about an artist's albums.

The HTTP response body for this endpoint contains an array of simplified [album object ](https://developer.spotify.com/documentation/web-api/reference/#object-simplifiedalbumobject)wrapped in a [paging object](https://developer.spotify.com/documentation/web-api/reference/#object-pagingobject) and use `limit` field to control the number of album objects to return and `offset` field to set the index of the first album to return.

So designed endpoint in `Rspotify` looks like this:

```rust
/// Paging object
///
/// [Reference](https://developer.spotify.com/documentation/web-api/reference/#object-pagingobject)
#[derive(Clone, Debug, Serialize, Deserialize, PartialEq, Eq)]
pub struct Page<T> {
    pub href: String,
    pub items: Vec<T>,
    pub limit: u32,
    pub next: Option<String>,
    pub offset: u32,
    pub previous: Option<String>,
    pub total: u32,
}

/// Get Spotify catalog information about an artist's albums.
///
/// Parameters:
/// - artist_id - the artist ID, URI or URL
/// - album_type - 'album', 'single', 'appears_on', 'compilation'
/// - market - limit the response to one particular country.
/// - limit  - the number of albums to return
/// - offset - the index of the first album to return
/// [Reference](https://developer.spotify.com/documentation/web-api/reference/#endpoint-get-an-artists-albums)
pub fn artist_albums<'a>(
    &'a self,
    artist_id: &'a ArtistId,
    album_type: Option<&'a AlbumType>,
    market: Option<&'a Market>,
) -> ClientResult<Page<SimplifiedAlbum>>;
```

Supposing that you fetched the first page of an artist's ablums, then
you would to get the data of the next page, you have to parse a URL:

```json
{
    "next": "https://api.spotify.com/v1/browse/categories?offset=2&limit=20"
}
```

You have to parse the URL and extract `limit` and `offset` parameters, and recall the `artist_albums` endpoint with setting `limit` to 20 and `offset` to 2.

We have to manually fetch the data again and again until all datas have been consumed. It is not elegant, but works.

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/20210429172938.png" >}}


## <span class="section-num">3</span> Iterator Story {#iterator-story}

Since we have the basic knowledge about the background, let's jump to the iterator version of pagination endpoints.

First of all, the iterator pattern allows us to perform some tasks on a sequence of items in turn. An iterator is responsible for the logic of itreating over each item and determining when the sequence has finished.

If you want to know about about `Iterator`, Jon Gjengset has covered a brilliant [tutorial](https://www.youtube.com/watch?v=yozQ9C69pNs) to demonstrate `Iterators` in Rust.

All iterators implement a trait named `Iterator` that is defined in the standard library. The definition of the trait looks like this:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

By implementing the `Iterator` trait on our own types, we could have iterators that do anything we want. Then working mechanism we want to iterate over paginated result will look like this:

{{< figure src="https://raw.githubusercontent.com/ramsayleung/images/master/sync_iterator_iterate.png" >}}

Now let's dive deep into the code, we need to implement `Iterator` for our own types, the pseudocode looks like:

```rust
impl<T> Iterator for PageIterator<Request>
{
    type Item = ClientResult<Page<T>>;

    fn next(&mut self) -> Option<Self::Item> {
	match call endpoints with offset and limit {
	    Ok(page) if page.items.is_empty() => {
		we are done here
		None
	    }
	    Ok(page) => {
		offset += page.items.len() as u32;
		Some(Ok(page))
	    }
	    Err(e) => Some(Err(e)),
	}
    }
}
```

In order to iterate paginated result from different endpoints, we need a generic type to represent different endpoints. The
[`Fn`](https://doc.rust-lang.org/std/ops/trait.Fn.html) trait comes to our mind, the function pointer that points to code, not data.

Then the next version of pseudocode looks like:

```rust
impl<T, Request> Iterator for PageIterator<Request>
where
    Request: Fn(u32, u32) -> ClientResult<Page<T>>,
{
    type Item = ClientResult<Page<T>>;

    fn next(&mut self) -> Option<Self::Item> {

	match (function_pointer)(offset and limit) {
	    Ok(page) if page.items.is_empty() => {
		we are done here
		None
	    }
	    Ok(page) => {
		offset += page.items.len() as u32;
		Some(Ok(page))
	    }
	    Err(e) => Some(Err(e)),
	}
    }
}
```

Now, our iterator story has iterated to the end, the next item is that
current full version code is
[here](https://github.com/ramsayleung/rspotify/blob/master/src/pagination/iter.rs),
check it if you are interested in :)


## <span class="section-num">4</span> Stream Story {#stream-story}

Are we done? Not yet. Let's move our eyes to stream story.

The stream story is mostly similar with iterator story, except that
iterator is synchronous, stream is asynchronous.

The `Stream` trait can yield multiple values before completing, similiar
to the `Iterator` trait.

```rust
trait Stream {
    /// The type of the value yielded by the stream.
    type Item;

    /// Attempt to resolve the next item in the stream.
    /// Returns `Poll::Pending` if not ready, `Poll::Ready(Some(x))` if a value
    /// is ready, and `Poll::Ready(None)` if the stream has completed.
    fn poll_next(self: Pin<&mut Self>, cx: &mut Context<'_>)
	-> Poll<Option<Self::Item>>;
}
```

Since we have already known the `iterator`, let make the stream story short. We leverage the
[`async-stream`](https://github.com/tokio-rs/async-stream) for using macro as Syntactic sugar to avoid clumsy type declaration and notation.

We use `stream!` macro to generate an anonymous type implementing the `Stream` trait, and the `Item` associated type is the type of the values yielded from the stream, which is `ClientResult<T>` in this case.

The stream [full version](https://github.com/ramsayleung/rspotify/blob/master/src/pagination/stream.rs) is shorter and clearer:

```rust
/// This is used to handle paginated requests automatically.
pub fn paginate<T, Fut, Request>(
    req: Request,
    page_size: u32,
) -> impl Stream<Item = ClientResult<T>>
where
    T: Unpin,
    Fut: Future<Output = ClientResult<Page<T>>>,
    Request: Fn(u32, u32) -> Fut,
{
    use async_stream::stream;
    let mut offset = 0;
    stream! {
	loop {
	    let page = req(page_size, offset).await?;
	    offset += page.items.len() as u32;
	    for item in page.items {
		yield Ok(item);
	    }
	    if page.next.is_none() {
		break;
	    }
	}
    }
}
```


## <span class="section-num">5</span> Appendix {#appendix}

Whew! It took more than I expected. Since iterators is the Rust features inspired by functional programming language ideas, which contributes to Rust's capability to clearly express high-level ideas at low-level performance.

It's good to leverage iterators wherever possible, now we can be thrilled to say that all endpoints don't need to manuallly loop over anymore, they are all iterable and rusty.

Thanks Mario and icewind1991 again for their works :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

