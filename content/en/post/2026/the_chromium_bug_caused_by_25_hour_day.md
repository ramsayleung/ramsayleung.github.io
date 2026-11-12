+++
title = "A Chromium Bug Caused by a 25-Hour Day"
date = 2026-11-09T16:12:00-07:00
lastmod = 2026-11-11T17:46:57-07:00
tags = ["programming"]
categories = ["Software", "Design"]
draft = false
toc = true
+++

{{< figure src="/ox-hugo/written-by-humna-not-ai.png" >}}


## <span class="section-num">1</span> 25 Hours a Day? {#25-hours-a-day}

Seeing the title, readers will immediately find it strange—doesn't a day only have 24 hours? Isn't that common sense? How could 25 hours appear?

But in some countries, like Canada, there is one day a year with 25 hours, and in the half year following that day, there is also a day with only 23 hours.

To understand this hour that suddenly disappears or is added, we need to first learn about daylight saving time and standard time.


## <span class="section-num">2</span> Daylight Saving Time and Standard Time {#daylight-saving-time-and-standard-time}

For those who are familiar with DST and PST, you could skip this section

----

As Confucius said while standing by a river: "It passes on just like this, not ceasing day or night!"

Time flows quietly like this. We use clocks to measure its passage. There is a "master" clock, UTC, which is never fast or slow and does not adjust with the seasons. It is called Coordinated Universal Time (UTC).

It is the "baseline" among clocks. No matter which country you are in or whether it is summer, UTC is always the same.

Wait—why is it called UTC and not CUT?

Because this abbreviation is not a direct translation of the English word order, but a compromise abbreviation between English `CUT` and French TUC (Temps Universel Coordonné).

And the local time of a country is _local time = UTC + time zone offset_

The plus and minus signs in the map below indicate time zone offsets.

{{< figure src="/ox-hugo/timezone_map.jpg" >}}

For example, China's Beijing time zone, East 8, is UTC+8, while São Paulo, Brazil is UTC-3. If the current UTC time is `2026-11-03 04:00:00`, then Beijing's time is `2026-11-03 12:00:00`, and São Paulo's time is `2026-11-03 01:00:00`.

So what are daylight saving time and standard time?

Take Vancouver, Canada, or Seattle, USA, 200 kilometers away, as examples. Under normal circumstances, their time zone (Pacific Standard Time, PST) is UTC-8. That is, when the UTC time is `2026-11-03 04:00:00`, the time in Vancouver and Seattle is `2026-11-02 20:00:00`.

To let people get more sun after work and use less lighting, the government came up with a strange rule: "In the early morning of a certain day in spring, the whole city moves their clocks forward by one hour."

Because _local time = UTC + time zone offset_, moving the clock forward by one hour is equivalent to adding one to the time zone offset, that is, from `UTC-8` to `UTC-7`. This `UTC-7` is called daylight saving time (Pacific Daylight Time, PDT); and `UTC-8` is standard time (Pacific Standard Time, PST).

If Vancouver switches from standard time to daylight saving time at 2 AM on March 8, and switches from daylight saving time to standard time at 2 AM on November 1.

Then the magical phenomenon occurs where March 8 has only 23 hours and November 1 has 25 hours.

{{< figure src="/ox-hugo/2025-03-08-pdt.jpg" >}}

{{< figure src="/ox-hugo/2025-11-01-pst.jpg" >}}

At this point, although I understand that a day with 23 or 25 hours can indeed occur once a year, I am also dizzy. This is too troublesome.

Can't we just pick one time zone and stick with it?

The good news is that some governments have indeed realized this problem and abandoned the return to standard time. The bad news is that only some provincial/state governments have made this decision, while the rest continue to do their own thing and keep switching back to standard time.

For example, the government of British Columbia, where Vancouver is located, [permanently uses daylight saving time](https://archive.news.gov.bc.ca/releases/news_releases_2024-2028/2026AG0013-000209.htm)[^fn:1] and abandoned the return to standard time on March 2nd 2026;

Meanwhile, Ontario, where Toronto is located to the east, and Washington State to the south continue to switch between daylight saving time and standard time,
causing Vancouver and Seattle, only 200 kilometers apart, to have a one-hour time difference.

Time is very error-prone in software development. When I saw that daylight saving time and standard time switching can cause a day to have 23 or 25 hours,
I asserted that any assumption that a day only has 24 hours (developers would feel innocent—this is common sense, after all!) will definitely lead to bugs.

No sooner said than done—here comes the bug.


## <span class="section-num">3</span> The Chromium Bug {#the-chromium-bug}

Chromium is an open-source web browser project led by Google.

It is the underlying implementation of Google Chrome, Microsoft Edge, Opera, Samsung Browser, and many other mainstream browsers.

In its code, there is a bug introduced by the assumption that a day only has 24 hours:

<https://chromium.googlesource.com/chromium/src/+/refs/branch-heads/8078/components/sync/service/device_statistics_scheduler.cc#87>

The core logic of this code is equivalent to:

```cpp
base::Time ComputeEarliestAllowedTimeToRun() {
  if (last_recorded_at.is_null()) {
    return now;
  }

  return (last_recorded_at + base::Days(1)).LocalMidnight();
}
```

The value of this function is used by `ScheduleNextRun()` to start the timer: the timer triggers `Run()` when it reaches `earliest_allowed`. `Run()` has a check roughly like this:

```cpp
if (earliest_allowed > now) {
  ScheduleNextRun();
  return;
 }
// otherwise execute
```

That is to say:

If the "earliest allowed time (`earliest_allowed`)" is still in the future, keep waiting;

If the "earliest allowed time (`earliest_allowed`)" has passed, execute the task, update `last_recorded_at`, and then schedule the next run.

The original intent of the `ComputeEarliestAllowedTimeToRun` function is:

"For some less important tasks, such as data reporting, ensure the task runs at most about once per day, and allow the next run only after local midnight."

The problem is that it interprets "adding one day" as ****adding 24 hours****, but the "day" in the real local calendar is not always ****24 hours****. So the above function uses
`base::Days(1)` to represent exactly 24 hours, that is, 86400 seconds.

But this is physical time, flowing uniformly, and does not represent local clock time.

Take our example above of Vancouver switching from daylight saving time to standard time on November 1. The clock is moved back, resulting in a day with 25 hours.

`last_recorded_at + base::Days(1)`, on November 1, ****adding 24 hours**** does not equal "the same time the next day", but rather falls on a later time on the same day.

{{< figure src="/ox-hugo/chromium_time_bug.jpg" >}}

Suppose `last_recorded_at: 2025-11-01 00:30 PDT`. Because the clock is moved back and the day has 25 hours, `last_recorded_at + base::Day(1)` becomes `2025-11-01 23:30 PST`. The current time now is `2025-11-01 00:30 PDT`, and `earliest_allowed` `(last_recorded_at + base::Day(1)).LocalMidnight()` becomes `2025-11-01 00:00 PST`.

Time comparison does not carry a time zone, so `earliest_allowed < now` becomes `true`, and the timer fires immediately.

After execution, update _last_recorded_at = now_, and then call `ScheduleNextRun()` again.

But at this point `now` is still within the `00:00~01:00` window of the fall-back day, and the new `last_recorded_at` is also still within this window. Calculate again:

```cpp
last_recorded_at + 24h -> still falls on the same day
.LocalMidnight()       -> again gets 00:00 of the same day
```

Again earlier than now, and the timer fires immediately again.

After understanding the cause and effect of the bug, the fix is also easy. The idea is:

Use the local calendar to calculate "midnight of the day after the date of the last record", instead of adding a fixed 24 hours first and then taking midnight.

Chromium fixed the problem two weeks ago, patch: <https://chromium-review.googlesource.com/c/chromium/src/+/8655919>

But the approach is quite "unusual": since the daylight saving/standard time switch causes a one-hour deviation, I'll just add a 4-hour tolerance range—surely the difference won't exceed 4 hours:

```cpp
// The metrics should be recorded once per calendar day, so the next possible
// time is midnight on the following day. Add 4 hours before rounding down to
// midnight to account for DST "fall back" days (which are 25 hours long).
base::Time earliest_allowed =
  last_recorded_at.is_null()
  ? now
  : (last_recorded_at.LocalMidnight() + base::Days(1) + base::Hours(4))
  .LocalMidnight();
```

Well, just tell me—did I fix it or not?

[^fn:1]: <https://archive.news.gov.bc.ca/releases/news_releases_2024-2028/2026AG0013-000209.htm>
