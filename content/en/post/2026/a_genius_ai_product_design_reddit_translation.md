+++
title = "A Genius AI Product design: Reddit translation"
date = 2026-01-06T19:46:00-08:00
lastmod = 2026-01-06T20:11:38-08:00
tags = ["ai", "design"]
draft = false
toc = true
+++

## <span class="section-num">1</span> The Universal AI Craze {#the-universal-ai-craze}

Since OpenAI released ChatGPT, there has been a massive rush to jump on the AI bandwagon. Over the past two years, it seems every website and application has been desperate to embrace AI.

Music apps have AI, for example Spotify [launched AI playlists](https://newsroom.spotify.com/2024-04-07/spotify-premium-users-can-now-turn-any-idea-into-a-personalized-playlist-with-ai-playlist-in-beta/)&nbsp;[^fn:1]; the programming Q&amp;A site Stack Overflow introduced [AI Assist](https://stackoverflow.com/ai-assist)&nbsp;[^fn:2]; browsers are integrating AI (Firefox, [Chrome](https://www.google.com/chrome/ai-innovations/)&nbsp;[^fn:3]); note-taking apps like Notion have built [AI workflows](https://www.notion.com/product/ai)&nbsp;[^fn:4]; code hosting platform like GitHub is focusing on AI agent like copilot. Perhaps the most absurd example is the ebook management software Calibre adding an "[Asking AI](https://calibre-ebook.com/whats-new)"&nbsp;[^fn:5] feature. Everyone truly want to be "AI-native".

It would be good if these implementations were useful, but a pile of websites and apps are just shoehorning AI into their products. These so-called AI features are often nothing more than a generic chatbox that lets you talk to an AI, often without even feeding the current page content as context. The user experience is terrible.


## <span class="section-num">2</span> Reddit's AI Translation {#reddit-s-ai-translation}

Today I saw an [article](https://www.theguardian.com/technology/2026/jan/03/reddit-overtakes-tiktok-uk-search-algorithms-gen-z)&nbsp;[^fn:6] stating that Reddit overtaks TikTok to become the fourth most visited social media platform in UK. The platform has undergone huge growth over the last two years, with an 88% increase in the proportion of UK internet users it reaches.

{{< figure src="/ox-hugo/reddit_overtake_tiktok.jpg" >}}

A series of factors are behind its rise. However, a change in Google's search algorithms last year to prioritise helpful content from discussion forums appears to have been a significant driver.
In the AI era, users are increasingly turning towards human-written content, and Reddit is benefiting from this trend.

This article reminded me of a new feature Reddit recently launched. Reddit is utilizing Large Language Models (LLMs) to generate translated versions of its massive archive of posts into multiple languages, such as translating English posts into Chinese, Korean, Japanese, etc.

Now, when I search in Chinese on Google(I was searching "Berserk TV"), the search results display the translated versions of original Reddit posts. When I click on a result, it jumps directly to the translated Chinese version on Reddit:

{{< figure src="/ox-hugo/berserk_tv_in_chinese.jpg" >}}

I can read the translated version directly:

{{< figure src="/ox-hugo/berserk_post_tl_zh_hans.jpg" >}}

Or click "Show Original" to read the source text.

{{< figure src="/ox-hugo/berserk_post_show_original.jpg" >}}


## <span class="section-num">3</span> Genius Product Design {#genius-product-design}

At first glance, it seems unremarkable, but upon closer inspection, it is absolutely brilliant. It might be the most solid AI implementation I've seen to date


### <span class="section-num">3.1</span> The Reddit Perspective {#the-reddit-perspective}

First, this feature allows more users to read posts in different languages. For example, Chinese or Japanese users can read posts originally written in English, French, or Spanish. This significantly increases the universality of Reddit posts. Language is no longer a barrier, which helps Reddit capture user traffic in the AI era and attract new users.

More users mean more content, and richer content attracts even more users. In the AI era, content is the raw material for "training" AI models. You can't train an AI with just NVIDIA GPUs; you need data.

Compared to external translation tools like Google Translate, this is a seamless, page-level translation indexed by search engine. The quality is incredibly high. It is so natural and well-integrated that, at first use, it's hard to even detect that it's an output of machine translation.

The user experience is excellent. Its natural and practical nature perfectly solves the core pain point for non-native speakers accessing high-quality posts.

Crucially, as a user, I was completely unaware of the existence of "AI."

It wasn't until I switched to an engineer's perspective that I realized Reddit used an LLM to translate the posts, and then had Google's search engine index the different language versions, displaying the translated results directly in Google Search.

{{< figure src="/ox-hugo/berserk_tv_in_chinese.jpg" >}}

The best AI features are the ones where you don't even realize AI is there.


### <span class="section-num">3.2</span> The Google Perspective {#the-google-perspective}

For Google, since the vast majority of Reddit posts are written by humans (Reddit mods generally hate bots or AI posting, and many subreddit rules explicitly ban AI-generated content under threat of mutes or bans), increasing the ranking weight of forum content ensures search quality.

Indexing posts in different languages provides Google with more high-quality search data.

So, Google gets to index a massive amount of fresh, high-quality, multilingual human content while improving search result quality, which aligns perfectly with their algorithm adjustment direction.


### <span class="section-num">3.3</span> The Flywheel of Data, Models, and Ecosystem {#the-flywheel-of-data-models-and-ecosystem}

By breaking down language barriers for users and enriching search results for Google, Reddit has kickstarted a grander positive feedback loop.

In February 2024,
Reddit and Google announced an [expanded partnership](https://blog.google/inside-google/company-announcements/expanded-reddit-partnership/)&nbsp;[^fn:7]that includes a multi-million dollar data licensing agreement allowing Google to use Reddit's content for training its artificial intelligence (AI) models.
This deal is reportedly worth about $60 million per year.

In September 2025, reports indicated that Reddit is in talks with Google and OpenAI. They may negotiate new deals.

Therefore, this AI translation design is a powerful tool, achieving a "1+1&gt;3" effect. It even serves as leverage to negotiate with other large model vendors like OpenAI and Anthropic, because you can't train AI without data.

It forms a flywheel: More Users -&gt; More Content -&gt; Better AI Data -&gt; Better Translation/Search/AI Models -&gt; More Users.

This is a design that benefits the users, Reddit, Google, and even the entire AI field. It can truly be called a "quadruple win."


## <span class="section-num">4</span> Conclusion {#conclusion}

Successful AI implementation makes the technology invisible while making the user value prominent.

Compared to other Apps/websites that shoehorn AI into their products, Reddit's translation integrates AI seamlessly into the product experience. It doesn't even tell the user that this is achieved via AI; it doesn't push the AI narrative.

[^fn:1]: <https://newsroom.spotify.com/2024-04-07/spotify-premium-users-can-now-turn-any-idea-into-a-personalized-playlist-with-ai-playlist-in-beta/>
[^fn:2]: <https://stackoverflow.com/ai-assist>
[^fn:3]: <https://www.google.com/chrome/ai-innovations/>
[^fn:4]: <https://www.notion.com/product/ai>
[^fn:5]: <https://calibre-ebook.com/whats-new>
[^fn:6]: <https://www.theguardian.com/technology/2026/jan/03/reddit-overtakes-tiktok-uk-search-algorithms-gen-z>
[^fn:7]: <https://blog.google/inside-google/company-announcements/expanded-reddit-partnership/>
