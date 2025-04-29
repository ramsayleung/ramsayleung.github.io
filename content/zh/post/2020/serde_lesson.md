+++
title = "Serde Tricks"
date = 2020-12-13T22:29:00+08:00
lastmod = 2022-02-24T14:56:34+08:00
tags = ["rust", "rspotify", "serde"]
categories = ["rust"]
draft = false
toc = true
showQuote = true
+++

The lesson learned from refactoring rspotify


## <span class="section-num">1</span> Preface {#preface}

Recently, I and [Mario](https://github.com/marioortizmanero) are working on refactoring [`rspotify`](https://github.com/ramsayleung/rspotify), trying to improve performance, documentation, error-handling, data model and reduce compile time, to make it easier to use. (For those who has never heard about `rspotify`, it is a Spotify HTTP SDK implemented in Rust).

I am partly focusing on polishing the data model, based on the [issue](https://github.com/ramsayleung/rspotify/issues/127) created by [Koxiaet](https://github.com/Koxiaet).

Since `rspotify` is API client for Spotify, it has to handle the request and response from Spotify HTTP API.

Generally speaking, the data model is something about how to structure the response data, and used [`Serde`](http://serde.rs/) to parse JSON response from HTTP API to Rust `struct`, and I have learnt a lot Serde tricks from refactoring.


## <span class="section-num">2</span> Serde Lesson {#serde-lesson}


### <span class="section-num">2.1</span> Deserialize JSON map to Vec based on its value. {#deserialize-json-map-to-vec-based-on-its-value-dot}

An actions object which contains a `disallows` object, allows to update the user interface based on which playback actions are available within the current context.

The response JSON data from HTTP API:

```json
{
    ...
	"disallows": {
	    "resuming": true
	}
    ...
}
```

The original model representing actions was:

```rust
#[derive(Clone, Debug, Serialize, PartialEq, Eq)]
pub struct Actions {
    pub disallows: HashMap<DisallowKey, bool>
}

#[derive(Clone, Serialize, Deserialize, Copy, PartialEq, Eq, Debug, Hash, ToString)]
#[serde(rename_all = "snake_case")]
#[strum(serialize_all = "snake_case")]
pub enum DisallowKey {
    InterruptingPlayback,
    Pausing,
    Resuming,
    ...
}
```

And Koxiaet gave great advice about how to polish `Actions`:

> `Actions::disallows` can be replaced with a `Vec<DisallowKey>` or `HashSet<DisallowKey>` by removing all entires whose value is false, which will result in a simpler API.

To be honest, I was not that familiar with `Serde` before, after digging in its official documentation for a while, it seems there is now a built-in way to convert JSON map to `Vec<T>` base on map's value.

After reading the [Custom serialization](https://serde.rs/custom-serialization.html) from documentation, there was a simple solution came to my mind, so I wrote my first customized deserialize function.

I created a dumb `Actions` struct inside the `deserialize` function, and converted `HashMap` to `Vec` by filtering its value.

```rust
#[derive(Clone, Debug, Serialize, PartialEq, Eq)]
pub struct Actions {
    pub disallows: Vec<DisallowKey>,
}

impl<'de> Deserialize<'de> for Actions {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
	D: Deserializer<'de>,
    {
	#[derive(Deserialize)]
	struct OriginalActions {
	    pub disallows: HashMap<DisallowKey, bool>,
	}

	let orignal_actions = OriginalActions::deserialize(deserializer)?;
	Ok(Actions {
	    disallows: orignal_actions
		.disallows
		.into_iter()
		.filter(|(_, value)| *value)
		.map(|(key, _)| key)
		.collect(),
	})
    }
}
```

The types should be familiar if you've used `Serde` before.

If you're not used to Rust then the function signature will likely look a little strange. What it's trying to tell is that d will be something that implements `Serde`'s `Deserializer` trait, and that any references to memory will live for the `'de` lifetime.


### <span class="section-num">2.2</span> Deserialize Unix milliseconds timestamp to Datetime {#deserialize-unix-milliseconds-timestamp-to-datetime}

A currently playing object which contains information about currently playing item, and the `timestamp` field is an integer, representing the Unix millisecond timestamp when data was fetched.

The response JSON data from HTTP API:

```json
{
    ...
	"timestamp": 1490252122574,
    "progress_ms": 44272,
    "is_playing": true,
    "currently_playing_type": "track",
    "actions": {
	"disallows": {
	    "resuming": true
	}
    }
    ...
}
```

The original model was:

```rust
/// Currently playing object
///
/// [Reference](https://developer.spotify.com/documentation/web-api/reference/player/get-the-users-currently-playing-track/)
#[derive(Clone, Debug, Serialize, Deserialize, PartialEq, Eq)]
pub struct CurrentlyPlayingContext {
    pub timestamp: u64,
    pub progress_ms: Option<u32>,
    pub is_playing: bool,
    pub item: Option<PlayingItem>,
    pub currently_playing_type: CurrentlyPlayingType,
    pub actions: Actions,
}
```

As before, Koxiaet made a great point about `timestamp` and =progress_ms=(I will talk about it later):

> `CurrentlyPlayingContext::timestamp` should be a `chrono::DateTime<Utc>`, which could be easier to use.

The polished struct looks like:

```rust
#[derive(Clone, Debug, Serialize, Deserialize, PartialEq, Eq)]
pub struct CurrentlyPlayingContext {
    pub context: Option<Context>,
    #[serde(
	deserialize_with = "from_millisecond_timestamp",
	serialize_with = "to_millisecond_timestamp"
    )]
    pub timestamp: DateTime<Utc>,
    pub progress_ms: Option<u32>,
    pub is_playing: bool,
    pub item: Option<PlayingItem>,
    pub currently_playing_type: CurrentlyPlayingType,
    pub actions: Actions,
}
```

Using the `deserialize_with` attribute tells `Serde` to use custom deserialization code for the `timestamp` field. The
`from_millisecond_timestamp` code is:

```rust
/// Deserialize Unix millisecond timestamp to `DateTime<Utc>`
pub(in crate) fn from_millisecond_timestamp<'de, D>(d: D) -> Result<DateTime<Utc>, D::Error>
where
    D: de::Deserializer<'de>,
{
    d.deserialize_u64(DateTimeVisitor)
}
```

The code calls `d.deserialize_u64` passing in a struct. The passed in struct implements `Serde`'s `Visitor`, and look like:

```rust
// Vistor to help deserialize unix millisecond timestamp to `chrono::DateTime`
struct DateTimeVisitor;

impl<'de> de::Visitor<'de> for DateTimeVisitor {
    type Value = DateTime<Utc>;
    fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
	write!(
	    formatter,
	    "an unix millisecond timestamp represents DataTime<UTC>"
	)
    }
    fn visit_u64<E>(self, v: u64) -> Result<Self::Value, E>
    where
	E: de::Error,
    {
	...
    }
}
```

The struct `DateTimeVisitor` doesn't have any fields, it just a type implemented the custom visitor which delegates to parse the `u64`.

Since there is no way to construct `DataTime` directly from Unix millisecond timestamp, I have to figure out how to handle the
construction. And it turns out that there is a way to construct `DateTime` from seconds and nanoseconds:

```rust
use chrono::{DateTime, TimeZone, NaiveDateTime, Utc};

let dt = DateTime::<Utc>::from_utc(NaiveDateTime::from_timestamp(61, 0), Utc);
```

Thus, what I need to do is just convert millisecond to second and nanosecond:

```rust
fn visit_u64<E>(self, v: u64) -> Result<Self::Value, E>
where
    E: de::Error,
{
    let second = (v - v % 1000) / 1000;
    let nanosecond = ((v % 1000) * 1000000) as u32;
    // The maximum value of i64 is large enough to hold millisecond, so it would be safe to convert it i64
    let dt = DateTime::<Utc>::from_utc(
	NaiveDateTime::from_timestamp(second as i64, nanosecond),
	Utc,
    );
    Ok(dt)
}
```

The `to_millisecond_timestamp` function is similar to `from_millisecond_timestamp`, but it's eaiser to implement, check
[this PR](https://github.com/ramsayleung/rspotify/pull/157/files#) for more detail.


### <span class="section-num">2.3</span> Deserialize milliseconds to Duration {#deserialize-milliseconds-to-duration}

The simplified episode object contains the simplified episode information, and the `duration_ms` field is an integer, which represents the episode length in milliseconds.

The response JSON data from HTTP API:

```json
{
    ...
	"audio_preview_url" : "https://p.scdn.co/mp3-preview/83bc7f2d40e850582a4ca118b33c256358de06ff",
    "description" : "Följ med Tobias Svanelid till Sveriges äldsta tegelkyrka"
    "duration_ms" : 2685023,
    "explicit" : false,
    ...
}
```

The original model was

```rust
#[derive(Clone, Debug, Serialize, Deserialize, PartialEq, Eq)]
pub struct SimplifiedEpisode {
    pub audio_preview_url: Option<String>,
    pub description: String,
    pub duration_ms: u32,
    ...
}
```

As before without saying, Koxiaet pointed out that

> `SimplifiedEpisode::duration_ms` should be replaced with a `duration` of type `Duration`, since a built-in `Duration` type works better than primitive type.

Since I have worked with `Serde`'s custome deserialization, it's not a hard job for me any more. I easily figure out how to deserialize `u64` to `Duration`:

```rust
#[derive(Clone, Debug, Serialize, Deserialize, PartialEq, Eq)]
pub struct SimplifiedEpisode {
    pub audio_preview_url: Option<String>,
    pub description: String,
    #[serde(
	deserialize_with = "from_duration_ms",
	serialize_with = "to_duration_ms",
	rename = "duration_ms"
    )]
    pub duration: Duration,
    ...
}

/// Vistor to help deserialize duration represented as millisecond to `std::time::Duration`
struct DurationVisitor;
impl<'de> de::Visitor<'de> for DurationVisitor {
    type Value = Duration;
    fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
	write!(formatter, "a milliseconds represents std::time::Duration")
    }
    fn visit_u64<E>(self, v: u64) -> Result<Self::Value, E>
    where
	E: de::Error,
    {
	Ok(Duration::from_millis(v))
    }
}

/// Deserialize `std::time::Duration` from millisecond(represented as u64)
pub(in crate) fn from_duration_ms<'de, D>(d: D) -> Result<Duration, D::Error>
where
    D: de::Deserializer<'de>,
{
    d.deserialize_u64(DurationVisitor)
}
```

Now, the life is easier than before.


### <span class="section-num">2.4</span> Deserialize milliseconds to Option {#deserialize-milliseconds-to-option}

Let's go back to `CurrentlyPlayingContext` model, since we have replaced millisecond (represents as `u32`) with `Duration`, it makes sense to replace all millisecond fields to `Duration`.

But hold on, it seems `progress_ms` field is a bit different.

The `progress_ms` field is either not present or a millisecond, the `u32` handles the milliseconds, as its value might not be present in the response, it's an `Option<u32>`, so it won't work with `from_duration_ms`.

Thus, it's necessary to figure out how to handle the `Option` type, and the answer is in the documentation, the `deserialize_option` function:

> Hint that the `Deserialize` type is expecting an optional value.

<!--quoteend-->

> This allows deserializers that encode an optional value as a nullable value to convert the null value into `None` and a regular value into `Some(value)`.

```rust
#[derive(Clone, Debug, Serialize, Deserialize, PartialEq, Eq)]
pub struct CurrentlyPlayingContext {
    pub context: Option<Context>,
    #[serde(
	deserialize_with = "from_millisecond_timestamp",
	serialize_with = "to_millisecond_timestamp"
    )]
    pub timestamp: DateTime<Utc>,
    #[serde(default)]
    #[serde(
	deserialize_with = "from_option_duration_ms",
	serialize_with = "to_option_duration_ms",
	rename = "progress_ms"
    )]
    pub progress: Option<Duration>,
}

/// Deserialize `Option<std::time::Duration>` from millisecond(represented as u64)
pub(in crate) fn from_option_duration_ms<'de, D>(d: D) -> Result<Option<Duration>, D::Error>
where
    D: de::Deserializer<'de>,
{
    d.deserialize_option(OptionDurationVisitor)
}
```

As before, the `OptionDurationVisitor` is an empty struct implemented `Visitor` trait, but key point is in order to work with
`deserialize_option`, the `OptionDurationVisitor` has to implement the `visit_none` and `visit_some` method:

```rust
impl<'de> de::Visitor<'de> for OptionDurationVisitor {
    type Value = Option<Duration>;
    fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
	write!(
	    formatter,
	    "a optional milliseconds represents std::time::Duration"
	)
    }
    fn visit_none<E>(self) -> Result<Self::Value, E>
    where
	E: de::Error,
    {
	Ok(None)
    }

    fn visit_some<D>(self, deserializer: D) -> Result<Self::Value, D::Error>
    where
	D: de::Deserializer<'de>,
    {
	Ok(Some(deserializer.deserialize_u64(DurationVisitor)?))
    }
}
```

The `visit_none` method return `Ok(None)` so the `progress` value in the struct will be None, and the `visit_some` delegates the parsing logic to `DurationVisitor` via the `deserialize_u64` call, so deserializing `Some(u64)` works like the `u64`.


### <span class="section-num">2.5</span> Deserialize enum from number {#deserialize-enum-from-number}

An `AudioAnalysisSection` model contains a `mode` field, which indicates the modality(major or minor) of a track, the type of scle from which its melodic content is derived. This field will contain a 0 for `minor`, a 1 for `major`, or a -1 for no result.

The response JSON data from HTTP API:

```json
{
    ...
	"mode": 0,
    "mode_confidence": 0.414,
    ...
}
```

The original struct representing `AudioAnalysisSection` was like this, since `mode` field was stored into a `f32=(=f8` was a better choice for this case):

```rust
#[derive(Clone, Debug, Serialize, Deserialize, PartialEq)]
pub struct AudioAnalysisSection {
    ...
	pub mode: f32,
    pub mode_confidence: f32,
    ...
}
```

Koxiaet made a great point about `mode` field:

> `AudioAnalysisSection::mode` and `AudioFeatures::mode` are `f32=s but should be =Option<Mode>=s where =enum Mode { Major, Minor }` as it is more useful.

In this case, we don't need the `Opiton` type and in order to deserialize enum from number, we firstly need to define a C-like enum:

```rust
pub enum Modality {
    #[serde(rename = "0")]
    Minor = 0,
    #[serde(rename = "1")]
    Major = 1,
    #[serde(rename = "1")]
    NoResult = -1,
}

pub struct AudioAnalysisSection {
    ...
	pub mode: Modality,
    pub mode_confidence: f32,
    ...
}
```

And then, what's the next step? It seems serde doesn't allow C-like enums to be formatted as integers rather that strings in JSON natively:

```json
working version:
{
    ...
	"mode": "0",
    "mode_confidence": 0.414,
    ...
}

failed version:
{
    ...
	"mode": 0,
    "mode_confidence": 0.414,
    ...
}
```

Then the failed version is exactly what we want. I know that the serde's official documentation has a solution for this case, the [serde_repr](https://github.com/dtolnay/serde-repr) crate provides alternative derive macros that derive the same Serialize and Deserialize traits but delegate to the underlying representation of a C-like enum.

Since we are trying to reduce the compiled time of rspotify, so we are cautious about introducing new dependencies. So a custom-made serialize function would be a better choice, it just needs to `match` the number, and convert to a related enum value.

```rust
/// Deserialize/Serialize `Modality` to integer(0, 1, -1).
pub(in crate) mod modality {
    use super::enums::Modality;
    use serde::{de, Deserialize, Serializer};

    pub fn deserialize<'de, D>(d: D) -> Result<Modality, D::Error>
    where
	D: de::Deserializer<'de>,
    {
	let v = i8::deserialize(d)?;
	match v {
	    0 => Ok(Modality::Minor),
	    1 => Ok(Modality::Major),
	    -1 => Ok(Modality::NoResult),
	    _ => Err(de::Error::invalid_value(
		de::Unexpected::Signed(v.into()),
		&"valid value: 0, 1, -1",
	    )),
	}
    }

    pub fn serialize<S>(x: &Modality, s: S) -> Result<S::Ok, S::Error>
    where
	S: Serializer,
    {
	match x {
	    Modality::Minor => s.serialize_i8(0),
	    Modality::Major => s.serialize_i8(1),
	    Modality::NoResult => s.serialize_i8(-1),
	}
    }
}
```


## <span class="section-num">3</span> Move into module {#move-into-module}

Update:

2021-01-15

-   `from(to)_millisecond_timestamp` have been moved into its module `millisecond_timestamp` and rename them to `deserialize` &amp; `serialize`
-   `from(to)_duration_ms` have been moved into its module `duration_ms` and rename them to `deserialize` &amp; `serialize`
-   `from(to)_option_duration_ms` have been moved into its module `option_duration_ms` and rename them to `deserialize` &amp; `serialize`


## <span class="section-num">4</span> Summary {#summary}

To be honest, it's the first time I have needed some customized works, which took me some time to understand how does `Serde` works. Finally, all investments paid off, it works great now.

Serde is such an awesome deserialize/serialize framework which I have learnt a lot of from and still have a lot of to learn from.


## <span class="section-num">5</span> Reference {#reference}

-   [Deserializing optional datetimes with serde](https://chrismcg.com/2019/04/30/deserializing-optional-datetimes-with-serde/)
-   [PR: Keep polishing the models](https://github.com/ramsayleung/rspotify/pull/157)
-   [PR: Refactor model](https://github.com/ramsayleung/rspotify/pull/145)
-   [PR: Deserialize enum from number](https://github.com/ramsayleung/rspotify/pull/177)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

