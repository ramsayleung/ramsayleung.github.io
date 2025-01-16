+++
title = "Hugo评论系统自适应博客主题: 支持dark与light theme"
date = 2024-12-04T13:25:00-08:00
lastmod = 2025-01-09T20:12:43-08:00
tags = ["blog", "hugo"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 问题 {#问题}

评论系统是博客的关键组件，Hugo 支持若干个评论系统，包括流行的 [Disqus](https://disqus.com/), 基于 GitHub 的 [Giscus](https://giscus.app) 和 [Utteranc](https://utteranc.es/), 以及其他[评论系统](https://gohugo.io/content-management/comments/)

我[博客](https://ramsayleung.github.io/)使用的评论系统是 Utteranc, 主题是 [PaperMod](https://github.com/adityatelange/hugo-PaperMod/), PaperMod 支持 dark 和 light 两种主题, 在初始化 Utteranc 时可以指定 theme, 如 `Github-Light`

```html
<script src="https://utteranc.es/client.js"
        repo="ramsayleung/comment"
        issue-term="title"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

但是 theme 在初始化时就指定好了，那么在博客切换到 dark theme 的时候， Utteranc 也不会自适应 dark theme，博客的theme与评论theme就不一致：

{{< figure src="/ox-hugo/responsive_comment_theme.jpg" >}}


## <span class="section-num">2</span> 解决思路 {#解决思路}

解决思路其实很简单，就获取当前的 theme, 然后再初始化 Utteranc 对应的 theme; 再在用户切换 theme 之后，再重新初始化 `Utteranc`.


### <span class="section-num">2.1</span> 获取当前Theme {#获取当前theme}

Hugo 并没有提供标准接口来获取当前主题， 虽然可以通过以下的方式来获取的 theme:

```js
const isDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
console.log(isDarkMode ? 'dark' : 'light');
```

但是这个是通过编译出来的CSS来判断当前theme，用户一旦手动切换了 theme, 上面的代码就无法生效了.

每个Hugo主题定义的方式可能还不一样, 以 PaperMod 为例，观察之后发现，light theme的时候body的html 为:

```css
<body id = "top" class="">
</body>
```

dark theme 的时候 `class` 就变为了 `class="dark"`:

```css
<body id = "top" class="">
</body>
```

所以可以通过判断 `body` 是否包含 `dark` 的 class 来判断当前是否为 dark theme.

```js
// 不同的Hugo theme可能会需要不同的判断方式
function getCurrentTheme() {
    return document.body.classList.contains('dark') ? 'dark' : 'light';
}
```


### <span class="section-num">2.2</span> 初始化评论系统 {#初始化评论系统}

[Utteranc](https://utteranc.es/) 文档提供的启用评价系统代码如下：

```html
<script src="https://utteranc.es/client.js"
        repo="[ENTER REPO HERE]"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

因为我们需要在切换 Theme 时重新加载 Utteranc, 所以就需要通过 Javascript 来实现上面的HTML功能:

```js
function loadUtterances(darkMode=false) {
    const commentContainer = document.getElementById("comments-utteranc");
    if (commentContainer !== null) {
        commentContainer.innerHTML = ''
        const commentScript = document.createElement("script");
        commentScript.setAttribute("id", "utteranc");
        commentScript.setAttribute("src", "https://utteranc.es/client.js");
        commentScript.setAttribute("data-repo", "ramsayleung/comment");
        commentScript.setAttribute("data-theme", darkMode ? "github-dark" : "github-light");
        commentScript.setAttribute("data-issue-term", "title");
        commentScript.setAttribute("crossorigin", "anonymous");
        commentScript.setAttribute("async", "true");
        commentContainer.appendChild(commentScript);
    }
}
```


### <span class="section-num">2.3</span> 监听主题变动 {#监听主题变动}

用户可以在博客界面手动选择他们喜欢的主题，可以从 `dark` -&gt; `light`, `light` -&gt; `dark`, 我们需要做的就是监听主题的变动，在切换主题之后，重新加载评论系统。

```js
// Watch for theme changes
const themeObserver = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
        if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
            const isDarkMode = getCurrentTheme() === 'dark';
            loadUtterances(isDarkMode);
            console.log(`changing theme`);
        }
    });
});

// Start observing the body element for class changes
themeObserver.observe(document.body, {
    attributes: true,
    attributeFilter: ['class']
});
```


## <span class="section-num">3</span> 总结 {#总结}

自适应评论系统的代码[在这里](https://github.com/ramsayleung/ramsayleung.github.io/blob/master/layouts/partials/comments.html), 实现的效果如下:

{{< figure src="/ox-hugo/responsive_comment_theme_2.jpg" >}}

{{< figure src="/ox-hugo/responsive_comment_theme_3.jpg" >}}

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

