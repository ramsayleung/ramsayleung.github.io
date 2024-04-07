+++
title = "TIL: Git Conditional Configs"
date = 2024-04-07T12:38:00-07:00
lastmod = 2024-04-07T14:23:13-07:00
tags = ["til", "git"]
categories = ["til", "git"]
draft = false
toc = true
+++

Every Git user will have probably been asked to set up their Git at the first time: <br/>

```sh
git config --global user.name "Ramsay Leung"
git config --global user.email ramsayleung@gmail.com
```

The above command will simply add the `user.name` and `user.email` value into your `~/.gitconfig` file <br/>

```sh
> cat ~/.gitconfig
[user]
    name = Ramsay Leung
    email = ramsayleung@gmail.com
[core]
    quotepath = false
[init]
    defaultBranch = master
```

You could also specify `--local` argument to writes the config values to `.git/config` in whatever project you're currently in. <br/>

If you need to simultaneously contribute to your work and open source project on the same laptop, with different Git config values, e.g.(company email address for work-specific projects, personal email address for open source project), what should you do? <br/>

You could definitely set up work-specific config as global config, then set up personal config with `--local` for every personal project separately. It works, but tedious and easy to mess-up. <br/>

Fortunately, starting from Git version 2.13, Git supports conditional configuration includes, you are capable of setting up different configs for different repositories. <br/>

If you add the following config to your global config file: <br/>

```toml
[includeIf "gitdir:~/projects/oss/"]
    path = ~/.gitconfig-oss

[includeIf "gitdir:~/projects/work/"]
    path = ~/.gitconfig-work
```

Then Git will look in the `~/.gitconfig-oss` files for values only if the project you are currently working on matches `~/projects/oss/`. <br/>

****Caution****: If you forget to specify the "/" at the end of the git dir, e.g. "~/projects/oss", Conditional Config won't work! <br/>

Therefore, you could have a "work" directory and work-specific config here and an "oss" directory with values for your open source projects, etc. <br/>

{{< figure src="/ox-hugo/conditional_config.png" >}} <br/>

Git also supports other filters more than `gitdir`, you could specify a branch name as an include filter with `onbranch` <br/>

```toml
  ; include only if we are in a worktree where foo-branch is
; currently checked out
[includeIf "onbranch:foo-branch"]
        path = foo.inc
```

Check out [the Git docs](https://git-scm.com/docs/git-config?ref=blog.gitbutler.com#_includes) for more details <br/>

