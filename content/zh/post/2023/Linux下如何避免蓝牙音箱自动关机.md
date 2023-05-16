+++
title = "Linux下如何避免蓝牙音箱自动关机"
date = 2023-05-15T20:48:00-07:00
lastmod = 2023-05-15T21:44:30-07:00
tags = ["linux"]
categories = ["linux"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 问题 {#问题}

最近方便布线，整理了桌上乱糟糟的线，把原来使用aux 线连接的蓝牙音响换成通过蓝牙连接。 <br/>

然后就发现一个问题，只要音响没有发出声音超过30分钟，蓝牙音响就会断开连接，并且自动关机，即使蓝牙音响连接着电源。 <br/>

一番搜索之后，就在知乎上发现了这个问题：[求问如何避免蓝牙音箱自动关机？](https://www.zhihu.com/question/41682642) <br/>

但里面提到的解决方案，大多只适用于特定平台，例如Windows 或者Macos, 没有提到 Linux 上的解决方案。 <br/>

每过半个小时手动打开蓝牙音响再连接的方式，实在是太蠢了。 <br/>


## <span class="section-num">2</span> 灵感 {#灵感}

但是知乎问题里面的部分回答给了我灵感，让我们想起国内某些APP 为了保活，避免被系统kill 掉，在后台播放无声音频的操作。 <br/>

我可以以低音量循环播放一段音频，以实现保活的作用： <br/>

> mpg123 -f 1000 ~/music/listen_to_the_sea.mp3 --loop -1 <br/>

mpg123 是mp3 播放命令行，=-f 1000= 参数的含义是：100%的音量是32768, 1000 约等于是1000/32768= 3% 的音量，=-loop -1= 就是指无限循环播放。 <br/>

```shell
man mpg123

...

-f factor, --scale factor
Change scale factor (default: 32768).

--loop times
for looping track(s) a certain number of times, < 0 means infinite loop (not with --random!).
```


## <span class="section-num">3</span> 优化 {#优化}

这样就实现了一个可用的版本，只是还要依赖一个 mp3 文件，肯定还有优化的空间。 <br/>

一番调研之后发现，=play/sox= 命令可以播放指定频率和时长的声音，可以播放20 hz以下的声音，这个频率下的声音人耳是听不到的： <br/>

> play -q -n synth 10 sin 20 <br/>

-   `-q`: 不显示播放进度条 <br/>
-   `-n synth 10` 播放10秒的音频 <br/>
-   `sin 20` 频率为20 hz(如果听到了，可以设置成更低) <br/>

执行命令之后，可以使用=pavucontrol= 查看声音输出，应该是类似这样的效果： <br/>

{{< figure src="/ox-hugo/sox_pavucontrol.png" >}} <br/>


## <span class="section-num">4</span> 定时执行 {#定时执行}

一直开着个terminal 窗口运行命令有点麻烦，这种重复性的工作，就可以交给 crontab, 让它每分钟执行一次，每次播放10秒。 <br/>

```shell
* * * * * play -q -n synth 10 sin 20
```

但实际运行，发现声音不能如预期那样播放。一番搜索之后，发现 [StackExchange](https://askubuntu.com/questions/832072/can-i-use-cron-to-chime-at-top-of-hour-like-a-grandfather-clock) 上有个答案提到需要 export 个环境变量，所以最好创建个脚本 `play_beep.sh`: <br/>

```shell
#!/bin/bash
export XDG_RUNTIME_DIR=/run/user/1000
play -q -n synth 10 sin 10
echo $(date) # 打印日期，主要是为了方便排查
```

然后再安装一个 crontab 任务: <br/>

```shell
* * * * * /usr/bin/sh /home/ramsay/code/shell/play_beep.sh >> /tmp/beep.log 2>&1
```

经过验证，一天都没有断开过蓝牙，自动关机了。 <br/>


## <span class="section-num">5</span> 参考 {#参考}

-   [求问如何避免蓝牙音箱自动关机？](https://www.zhihu.com/question/41682642) <br/>
-   [Can I use cron to chime at top of hour like a grandfather clock?](https://askubuntu.com/questions/832072/can-i-use-cron-to-chime-at-top-of-hour-like-a-grandfather-clock/832266#832266) <br/>

