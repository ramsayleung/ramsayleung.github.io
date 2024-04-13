+++
title = "TIL: Git Blame with Following"
date = 2024-04-13T12:46:00-07:00
lastmod = 2024-04-13T13:52:07-07:00
tags = ["til", "git"]
categories = ["til", "git"]
draft = false
toc = true
+++

Developers usually use `git blame` in GUI tools like GitHub Blame <br/>

{{< figure src="/ox-hugo/github_blame.png" >}} <br/>

or using GitLens blame in VSCode: <br/>

{{< figure src="/ox-hugo/git_blame_git_lens_vscode.png" >}} <br/>

Even though GUI tools is intuitive, but the Git CLI has much more powerful tooling for finding something closer to the real story behind your code. <br/>

There are many scenarios that CLI is valuable, the first is ignoring the whitespace changes. <br/>

For example, if you formatted your C++ codebase with `clang-format` or Javascript codebase with `prettier`, you haven't actually changed the codebase, but you're the owner of tons of lines of code. <br/>

The `git blame -w` option will ignore these type of whitespace changes. <br/>

The other great option is `-C` which will look for code movement between files in a commit. <br/>

For example, if you refactor a function from one file to another, the normal `git` blame will simply show you as the author in the new file, but the `-C` option will follow that movement and show the last person who actually change those lines of code. <br/>

`-C` is extremely helpful when I need to find out the original author of some lines of code after file renames or refactors, to know more about the background and context behind this code <br/>

According to the `git blame` doc, you could pass `-C` up to three times to ask Git try even harder: <br/>

```shell
-C[<num>]
           In addition to -M, detect lines moved or copied from other files that were modified in the same commit.
           This is useful when you reorganize your program and move code around across files.
           When this option is given twice, the command additionally looks for copies from other files in the commit that creates the file.
           When this option is given three times, the command additionally looks for copies from other files in any commit.
```

(it's a bit of odd design) <br/>

Let's take [the access.rb file of ActiveModel module in Rails framework](https://github.com/rails/rails/blob/main/activemodel/lib/active_model/access.rb) for example: <br/>

```shell
git blame activemodel/lib/active_model/access.rb
```

{{< figure src="/ox-hugo/normal_git_blame.png" caption="<span class=\"figure-number\">Figure 1: </span>Vanilla git blame" >}} <br/>

Ok, it looks like Jonathan Hefner wrote all of this code it appears, let's look at the same code with `git blame -w -C -C -C activemodel/lib/active_model/access.rb` <br/>

{{< figure src="/ox-hugo/git_blame_-w_-C_-C_-C.png" caption="<span class=\"figure-number\">Figure 2: </span>git blame -w -C -C -C" >}} <br/>

Now we can see that Git has followed this code from file to file over the course of multiple renames, it turns out Jonathan Hefner is the most recent file renamer, Guillermo Iguaran is the original author. <br/>

If we want to know the history about this file, it's much better to ask Guillermo rather than Jonathan, which is beyond what the GUI blame or normal Git blame tool reveals <br/>

