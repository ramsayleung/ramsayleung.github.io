+++
title = "Linux/Unix Shell 二三事之过滤器grep"
description = "an introduction about grep"
date = 2017-03-13T00:00:00-07:00
keywords = ["shell", "linux", "grep"]
lastmod = 2025-01-09T17:37:12-08:00
tags = ["shell", "linux", "command_line"]
categories = ["Linux/Unix Shell 二三事"]
draft = false
toc = true
+++

文本三剑客之 Grep

> grep - print  lines matching a pattern

今天我想聊聊 **grep** 这个命令；据说，有Unix/Linux 的地方就会有 **grep**, 这个可能是安装得最广泛的命令之一；那么 **grep** 是用来干什么的呢？

grep 其实是用来在文件中搜索特定内容或者模式的工具(配合正则表达式“食用”，味道更佳 :))现在就来一起看看\*grep\* 的用法


## <span class="section-num">1</span> 基本用法 {#基本用法}


### <span class="section-num">1.1</span> 基础用法 {#基础用法}

现在假设有一个简单的文本文件(双城记开头)tinytale.txt,内容如下

> it was the best of times it was the worst of times
> it was the age of wisdom it was age of foolishness
> it was the epoch of belief it was the epoch of incredulity
> it was the season of light it was the season of darkness
> IT WAS THE SPRING OF HOPE IT WAS THE WINTER OF DESPAIRE

现在开始介绍 **grep** 的基本用法： **grep** 的基本用法很简单的，假设我想要搜索单词 **darkness**

```shell
grep darkness /tmp/tinytale.txt
```

输出如下：

```text
it was the season of light it was the season of darkness
```


### <span class="section-num">1.2</span> 结合正则表达式 {#结合正则表达式}

默认情况下， **grep** 是开启正则表达式的模式的，所以你可以直接在文件搜索中使用
正则表达式。现在在文件中搜索以字母 **e** 开头后接三个字符，然后以 **h** 结尾的单词：

```shell
grep "e...h" /tmp/tinytale.txt
```

输出如下：

> ```text
> it was the epoch of belief it was the epoch of incredulity
> ```

可以看到，正则表达式匹配了 **epoch** 这个单词。正则表达式的威力无与伦比的，把 **grep\*和正则表达式结合起来可以更好地发挥 \*grep** 这个工具的潜力；而本文主要是介绍 **grep**, 更多有关正则表达式的用法不细讲了


### <span class="section-num">1.3</span> 统计出现的次数 {#统计出现的次数}

有时，如果你需要统计某种模式或者某个单词出现的个数，你会发现 **grep** 非常有用；

要实现该功能，只需给 **grep** 添加 **-c** 参数；例如统计单词 **the** 出现的个数：

````shell
grep -c the /tmp/tinytale.txt
````

结果输出如下：

````text
4
````

文本中包含4个 **the**


### <span class="section-num">1.4</span> 忽略大小写 {#忽略大小写}

前面提到， **grep** 默认是使用正则表达式来搜索文件的，所以 **grep** 是区分大小写的；

如果你想修改 **grep** 的默认行为来忽略大小写，你可以添加 **-i** 参数

````shell
grep -i the /tmp/tinytale.txt
````

输出结果如下：

````text
it was the best of times it was the worst of times
it was the age of wisdom it was age of foolishness
it was the epoch of belief it was the epoch of incredulity
it was the season of light it was the season of darkness
IT WAS THE SPRING OF HOPE IT WAS THE WINTER OF DESPAIRE
````

可以发现 **THE** 也是可以被 grep 搜索到的；但是如果没有添加 **-i** ,你只会看到4行输出。

当然你可以在正则表达式里面添加忽略大小写的模式，只是直接添加 **-i** 会简单很多。


## <span class="section-num">2</span> 搜索多个文件 {#搜索多个文件}

上面搜索的都只是单个文件，而 grep 可以让你同时搜索多个文件；现在就来看看怎么搜索多个文件吧。

下面两种写法结果都是一样的，但是我个人推崇第一种，因为可以输入更少一些内容 :)

````shell
grep belief /tmp/{tinytale.txt,tale.txt}
````

````shell
grep belief /tmp/tinytale.txt /tmp/tale.txt
````

输出结果如下：

````text
tinytale.txt:it was the epoch of belief it was the epoch of incredulity
tale.txt:it was the epoch of belief it was the epoch of incredulity
tale.txt:pains of by rearing her in the belief that her father was dead
tale.txt:this was no passive belief but an active weapon which they flashed
tale.txt:belief in solomon deducting a mere trifle for this slight mistake
tale.txt:you will bear testimony to what i have said and to your belief in it
tale.txt:herself into the show of a belief that they would soon be reunited
````

可以看到， **grep** 把匹配到单词的那一行内容和对应的文件都显示出来了，你就可以很方便地看到搜索结果，并知道匹配单词的来源。

如果你也像我这样，不想输入那么多的内容，你可以使用正则表达式匹配所有的文本文件，如下：

````shell
grep belief /tmp/*.txt
````

输出结果也会跟上面一致 (假设你 **_tmp_** 目录下只有两个文本文件); 我告诉\*grep\* 搜索\*/tmp\* 下所有的 **.txt** 文件。


### <span class="section-num">2.1</span> 递归搜索 {#递归搜索}

你也可以使用 **grep** 递归搜索目录；你只需在指定目录后，添加 **-R** , **grep** 就会
递归搜索指定目录的所有子目录。我已经把当前目录切换到 **/tmp**:

````shell
grep -R "belief" .
````

输出结果如下：

````nil
./tale.txt:it was the epoch of belief it was the epoch of incredulity
./tale.txt:pains of by rearing her in the belief that her father was dead
./tale.txt:this was no passive belief but an active weapon which they flashed
./tale.txt:belief in solomon deducting a mere trifle for this slight mistake
./tale.txt:you will bear testimony to what i have said and to your belief in it
./tale.txt:herself into the show of a belief that they would soon be reunited
./tinytale.txt:it was the epoch of belief it was the epoch of incredulity
````

结果展示了一系列在当前目录和子目录匹配 **belief** 的文件。此外你也可以排除掉某些你
不需要搜索的文件，例如有一个 **foo.xml** 的文件，里面也可能会有 **belief** 这个单词，
但是你就是不想搜索这个文件，或者全部的 **.xml** 文件，你可以这么玩：

````shell
grep -R --exclude="*.xml" "belief" .
````


## <span class="section-num">3</span> 在标准输入搜索 {#在标准输入搜索}

**grep** 也是过滤器，所以 **grep** 自然而然具有处理标准输入输出的能力了；处理其他命令的输出结果也是 **grep** 非常常用的场景之一。假设你现在的 **vim** 突然卡顿，挂了:),你想要 **kill** 掉 **vim** 的进程，你可以：

````shell
ps -e|grep vim
````

结果输出如下：

````nil
samray   21939     1  0 19:42 ?        00:00:00 gvim
````

其中第一条记录就是你想要搜索的进程了，你运行 **kill 21939** 就可以杀掉 vim 的进程了；因为我系统的是图型化界面的 vim, 所以是 gvim.

---

正如我之前的文章提到的那样，单纯的过滤器的用处似乎不大，但是如果结合起来就会威力无穷至于，如何结合，就需要慢慢探索了。


## <span class="section-num">4</span> 反向搜索 {#反向搜索}

现在执行的搜索都是匹配搜索，即将匹配的内容显示出来，而 **grep** 还有反向搜索的功能 (invert Searches)就是将不包含有指定模式的内容显示出来。

该功能在用来修改有很多注释的配置文件时特别有用；例如常用的服务器软件 **nginx**  的配置文件是默认是含有很多注释的，如下

````cfg
#user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
worker_connections 1024;
# multi_accept on;
}

http {

##
# Basic Settings
##

sendfile on;
tcp_nopush on;
tcp_nodelay on;
keepalive_timeout 65;
types_hash_max_size 2048;
# server_tokens off;

# server_names_hash_bucket_size 64;
# server_name_in_redirect off;

include /etc/nginx/mime.types;
default_type application/octet-stream;

##
# SSL Settings
##

ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
ssl_prefer_server_ciphers on;

##
# Logging Settings
##
log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $bytes_sent "$http_referer" "$http_user_agent" "$gzip_ratio"';
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;

##
# Gzip Settings
##

gzip on;
gzip_disable "msie6";

# gzip_vary on;
# gzip_proxied any;
# gzip_comp_level 6;
# gzip_buffers 16 8k;
# gzip_http_version 1.1;
# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

##
# Virtual Host Configs
##

include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;

ignore_invalid_headers on;
client_header_timeout 240;
client_body_timeout 240;
send_timeout 240;
client_max_body_size 100m;
proxy_buffer_size 128k;
proxy_buffers 8 128k;

upstream tomcat_server{
server 127.0.0.1:8080 fail_timeout=0;
}

upstream gunicorn_server{
server 127.0.0.1:5000 fail_timeout=0;
}
server{
server_name 127.0.0.1;
listen 443;
# ssl on;
# ssl_certificate /etc/letsencrypt/live/samray.ren/fullchain.pem;
# ssl_certificate_key /etc/letsencrypt/live/samray.ren/privkey.pem;
location / {

# Forward SSL so that Tomcat knows what to do
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-Server $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_pass http://tomcat_server;
proxy_set_header X-Forwarded-Proto https;

proxy_redirect off;
proxy_connect_timeout      240;
proxy_send_timeout         240;
proxy_read_timeout         240;

}

location /test{
return 402;
}

location /weixin {
# try_files $uri @proxy_to_app;
return 402;
}
location @proxy_to_app {
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $http_host;
proxy_redirect off;
proxy_pass http://gunicorn_server;

}
}

}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
````

里面实在有太多的注释了，虽说是很好的参考，但是看多了会感觉很碍眼，所以你希望可以有一份没有注释的配置文件，你就可以使用 **grep** 和参数 **-v**:

````shell
egrep -v "#|^$" /etc/nginx/nginx.conf >/tmp/nging.conf
````

结果如下：

````cfg
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
events {
worker_connections 1024;
}
http {
sendfile on;
tcp_nopush on;
tcp_nodelay on;
keepalive_timeout 65;
types_hash_max_size 2048;
include /etc/nginx/mime.types;
default_type application/octet-stream;
ssl_prefer_server_ciphers on;
log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $bytes_sent "$http_referer" "$http_user_agent" "$gzip_ratio"';
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
gzip on;
gzip_disable "msie6";
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
ignore_invalid_headers on;
client_header_timeout 240;
client_body_timeout 240;
send_timeout 240;
client_max_body_size 100m;
proxy_buffer_size 128k;
proxy_buffers 8 128k;

upstream tomcat_server{
server 127.0.0.1:8080 fail_timeout=0;
}

upstream gunicorn_server{
server 127.0.0.1:5000 fail_timeout=0;
}
server{
server_name 127.0.0.1;
listen 443;
location / {

proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-Server $host;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_pass http://tomcat_server;
proxy_set_header X-Forwarded-Proto https;
proxy_redirect off;
proxy_connect_timeout      240;
proxy_send_timeout         240;
proxy_read_timeout         240;
}
location /test{
return 402;
}
location /weixin {
return 402;
}
location @proxy_to_app {
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $http_host;
proxy_redirect off;
proxy_pass http://gunicorn_server;

}
}
}
````

**egrep** 是 **grep** 的扩展，你也可以通过 **-E** 使用扩展功能。就这样，你就可以得到一份很“干净”的配置文件了。


## <span class="section-num">5</span> 小结 {#小结}

在以前 **grep** 是 **hacker** 工具箱里面审查源代码必不可少的工具之一，但是随着技术的发展，似乎对比其他同类型的工具， **grep** 的性能已经难尽人意，特别是对比 **ag** 这个搜索神器；

虽说很多人都已经转移到了 **ag** 阵营，但是因为 **grep** 被广泛预装到各类的Linux/Unix 机器，所以 **grep** 还是使用得很广泛滴。

更多 **grep** 的用法就需要查询手册了：

````shell
man grep
````

Enjoy Shell :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

