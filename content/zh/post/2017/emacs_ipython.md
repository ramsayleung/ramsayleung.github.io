+++
title = "在Emacs中使用Ipython"
description = "Use Ipython in Emasc"
date = 2017-02-22T00:00:00+08:00
keywords = ["emacs", "ipython"]
lastmod = 2022-02-23T19:16:07+08:00
tags = ["emacs", "python"]
categories = ["emacs"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Emacs Ipython 输出错误 {#emacs-ipython-输出错误}

在Emacs 运行 **run-python** 的时候，报错了，如下

```emacs-lisp
[?12l[?25h2+2
      [J[?7h[?12l[?25h[?2004l[?7hOut[1]: 4
```

因为我的版本时Ipython5,查阅文档[http://ipython.readthedocs.io/en/stable/whatsnew/version5.html#id1](http://ipython.readthedocs.io/en/stable/whatsnew/version5.html#id1)
之后，发现Ipython5 有了新的terminal 接口，和Emacs 继承的shell 不兼容，所以
会出现上述的错误，只要给Ipython 加上运行参数就能解决了，所以只要在 **.emacs**
或者对应的初始化文件加上下面语句

```emacs-lisp
(setq python-shell-interpreter "ipython"
      python-shell-interpreter-args "--simple-prompt -i")
```

---


### <span class="section-num">1.1</span> Update 2017-3-15 {#update-2017-3-15}

在添加了 **--simple-promp -i** 参数以后，虽说乱码的问题解决了，但是新的问题又出现了
在Ipython 里面是没法无法输入多行内容的，即使是一个简单的循环，详情查看这条issue
<https://github.com/ipython/ipython/issues/9816>. 现在Ipython 开发社区还没有解决这个
问题，所以现在的权宜之计就是使用 Ipython4,等到社区解决了这个问题在升级为 Ipython5

```shell
pip install --force-reinstall ipython==4.2.1
```


## <span class="section-num">2</span> Emacs Ipython 的使用优化 {#emacs-ipython-的使用优化}


### <span class="section-num">2.1</span> python-pop {#python-pop}

因为我之前使用Emacs的时候，是使用Spacemacs的配置的，但是后来觉得还是自己的
配置用的更舒服，所以又切换回自己的配置，但是我还是很想念Spacemacs的一些绑定
例如shell在底下弹出，或者是关闭，然后找到了[Shell-pop](https://github.com/kyagi/shell-pop-el) 这package,就可以用回
Spacemacs的shell使用习惯。然后我觉得，Ipython shell也可以这样配置，只不过
我没有发现类似的package,又因为Emacs Lisp的强大，所以我自己写了一段小函数实现
shell-pop 的功能

```emacs-lisp

(defun samray/python-pop ()
  "Run python and switch to the python buffer.
similar to shell-pop"
  (interactive)
  (if (get-buffer "*Python*")
      (if (string= (buffer-name) "*Python*")
	  (if (not (one-window-p))
	      (progn (bury-buffer)
		     (delete-window))
	    )
	(progn (switch-to-buffer-other-window "*Python*")
	       (end-of-buffer)
	       (evil-insert-state)))
    (progn
      (run-python)
      (switch-to-buffer-other-window "*Python*")
      (end-of-buffer)
      (evil-insert-state))))
```

如果没有使用Evil,可以把 **(evil-insert-state)**去掉


### <span class="section-num">2.2</span> Ipython History {#ipython-history}

我在普通的Shell使用Ipython的时候，很自然地使用上下方向键翻到上一条/下一条
执行的命令，因为shell的使用习惯就是这样滴，但是在Emacs里面使用Ipython,上下
方向键是去到上一行/下一行，就好像 vim 的 **j** **k**,如果要翻到上一条命令，快捷键
是 **M-p**,实在很不习惯，所以在查了一下Emacs manual 后，我改了一下按键绑定就实现了
我想要的效果

```emacs-lisp
(define-key comint-mode-map (kbd "<up>") 'comint-previous-input)
(define-key comint-mode-map (kbd "<down>") 'comint-next-input)
```

Enjoy Emacs :)