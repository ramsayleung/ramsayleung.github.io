+++
title = "TamperMonkey userscript在 Single Page Application 跳转链接后不运行问题分析"
date = 2023-08-26T09:29:00-07:00
lastmod = 2025-01-09T20:05:21-08:00
tags = ["javascript", "userscript", "debug"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 背景 {#背景}

我习惯使用浏览器匿名模式来打开 Youtube 视频，避免 Youtube 的推荐算法给我总是推荐同一类的视频。 <br/>

但有个问题: 匿名模式下，Youtube的播放器是默认自动播放的。 <br/>

虽然我可以登录 Google 账号并关闭自动播放，但是每次我使用匿名模式来浏览，关闭窗口之后，所有的操作记录都会被清除了，自动播放设置也不会被保存。 <br/>

所以我使用 TamperMonkey 给 Youtube写了一个关闭自动播放的脚本，打开 Youtube 播放器时，把自动播放按钮给关闭掉，避免我使用浏览器的匿名窗口打开 Youtube 之后，Youtube自动播放导致一直有声音。 <br/>

脚本逻辑很简单： <br/>

```js
// ==UserScript==
// @name         Disable YouTube Autoplay
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  Automatically turn off YouTube autoplay
// @author       ramsayliang
// @match        https://*.youtube.com/*
// @grant        none
// @icon         https://www.youtube.com/favicon.ico
// ==/UserScript==

(function() {
    'use strict';

    // Function to disable autoplay
    function disableAutoplay() {
        console.log("disableAutoplay");
        const autoplayToggle = document.querySelector('div[class="ytp-autonav-toggle-button"]');
        if (autoplayToggle && autoplayToggle.getAttribute('aria-checked') === 'true') {
            autoplayToggle.click();
            console.log("Disable YouTube Autoplay");
        }
    }

    console.log("Before loading");
    // Run the function when the page loads
    window.addEventListener('load', () => {
        // Wait for a short delay to ensure the page fully loads
        // setTimeout(disableAutoplay, 8000);
        console.log("Loading page..");
        disableAutoplay();
    }, false);
})();
```


## <span class="section-num">2</span> 问题 {#问题}

但是我发现这个脚本时灵不灵，甚至有一天晚上，睡着之后被自动播放的声音吵醒了。 <br/>

我就在找能稳定复现这个问题的场景，花费半个小时，终于能稳定复现问题了。 <br/>

1.  打开 Chrome 的匿名模式 <br/>
2.  打开 Youtube 首页, 地址是: youtube.com, 脚本运行. <br/>
3.  随意点击一个视频，进行播放：<https://www.youtube.com/watch?v=-pKGaxoVhok> ，脚本就不会运行了。 <br/>
4.  如果我在视频播放页刷新，脚本又会重新运行。 <br/>

但分析了1个小时，都没有找到原因，我甚至怀疑是 Tampermonkey 有Bug(虽然主观感觉这个可能性较小) <br/>


## <span class="section-num">3</span> 分析 {#分析}

结合 Tampermonkey 的表现，我觉得可能是 Tampermonkey 的执行机制有问题，可能是判断 youtube.com 和 youtube.com/watch?v=xxxx 是同一个页面，就不会运行两次。 <br/>

在Stackoverflow上搜索了一下，发现果然如此： <br/>

-   <https://stackoverflow.com/questions/65017670/tampermonkey-match-not-working-when-visit-target-link-through-redirection> <br/>

原来这个是feature, 不是bug. <br/>

对于 Single Page Application, Tampermonkey 无法判断页面的 DOM 是否发生变化，是否访问到新的页面了，所以不会重复执行。 <br/>

分析下来，之前脚本能直接生效的原因是, 我在Chrome正常模式下打开 Youtube 首页, 右键对想要看的视频，点击"Open link in incognito window", 因为页面是首次打开，所以就能正常运行。 <br/>

{{< figure src="/ox-hugo/youtube_open_link_in_incognito_window.png" >}} <br/>

但当通过Chrome 匿名模式打开 Youtube 首页，然后再点击视频播放，无法运行脚本。 <br/>

因为在打开首页的时候，脚本已经运行过了，当点击跳转到指定的视频时，且 Youtube 是个 Single Page Application, 对脚本来说，页面就没有发生过变化，所以不会再运行。 <br/>

如果手动刷新，页面重新加载，脚本就又会被加载。 <br/>


## <span class="section-num">4</span> 解决方案 {#解决方案}

最后通过 [Stackoverflow](https://stackoverflow.com/a/39508954) 的建议，增加一个对 DOM 事件变化的监听来解决： <br/>

```javascript
// https://stackoverflow.com/questions/2844565/is-there-a-javascript-jquery-dom-change-listener/39508954#39508954
// Detect the url change for programmatic navigation
let lastUrl = location.href;
new MutationObserver(() => {
    const url = location.href;
    if (url !== lastUrl) {
        lastUrl = url;
        onUrlChange();
    }
}).observe(document, {subtree: true, childList: true});

// callback when url change
function onUrlChange() {
    console.log('URL changed!', location.href);
    if(isYouTubeVideoURL(location.href)){
        disableAutoplay();
    }
}
```

这样就可以正常运行了。 <br/>

