+++
title = "Python多线程端口扫描器"
description = "An introduction about port scan"
date = 2017-03-19T00:00:00-07:00
lastmod = 2025-01-09T20:02:08-08:00
tags = ["python", "network", "tool"]
draft = false
toc = true
+++

近两日，闲来无事，就写了些端口扫描器，重温TCP/IP协议栈的部分原理。


## <span class="section-num">1</span> 端口扫描器 {#端口扫描器}

所谓的端口扫描器，其实是用来检测目标服务器有哪些端口开放所使用的工具，一般是管理员用来进行安全加固，检测是否有无意开放的端口；或者是恶意攻击的人员在进行攻击前的准备工作。

所以综述上下，端口扫描器是用来确定目标机器 (本地机器或者远程机器)的特定服务的可用性


## <span class="section-num">2</span> 端口扫描原理 {#端口扫描原理}

上面提到过，端口扫描器是用来确定目标机器的服务的可用性的；那么具体是怎么确定的呢？如果还没有答案的话，可以换个角度来思考这个问题。

假如你想确定邻居家的妹子是否在家，你会怎么办？这不简单么，问一下不就清楚了么？对阿，对于服务器的端口也可以适用这样的方法嘛。端口扫描的原理都是“问一下”，只是问的方法不一样而已，就好像你是决定直接过去敲邻居门，还是打电话过去一样，殊途同归，方法是没有对错的之分，差异只是方法的优劣。


### <span class="section-num">2.1</span> TCP连接扫描 {#tcp连接扫描}

这是最简单的一种方法，一般被称为连接扫描，即利用 `socket` 对目标机器进行连接尝试，如果能够成功建立三次握手连接，那就说明你用 `socket` 连接的端口是开放的；然后你就可以断开连接，扫描下一个目标端口了 (如果不断开连接，这就是一种 DDOS攻击了).

只不过TCP连接扫描不是很常用，不仅是因为容易被发现，而且你的IP地址也可能会被目标地址记录下来的(对于攻击者来说，隐藏身份是很重要的)


#### <span class="section-num">2.1.1</span> 代码解析： {#代码解析}

```python
def scan(self, args):
    host, port = args
    try:
        # Create a TCP socket and try to connect
        # AF_INET for ipv4,AF_INET6 for ipv6
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((host, port))
        sock.close()
        return host, port, True
    except (socket.timeout, socket.error):
        return host, port, False
```

因为原理很简单，所以核心代码也是很简洁的，只是建立 `socket` 然后进行连接，如果连接不上，就很大几率说明端口是关闭的 (并不是绝对的，例如socket超时的异常可能就是因为网络异常，不一定是目标机器的缘故)


### <span class="section-num">2.2</span> SYN扫描 {#syn扫描}

再回顾一下TCP的三次握手：


#### <span class="section-num">2.2.1</span> TCP三次握手 {#tcp三次握手}

1.  TCP建立连接时，首先客户端和服务器处于close状态。
2.  然后客户端发送SYN同步位，此时客户端处于SYN-SEND状态，服务器处于lISTEN状态，当服务器收到SYN以后，向客户端发送同步位SYN和确认码ACK，然后服务器变为SYN-RCVD，客户端收到服务器发来的SYN和ACK 后，客户端的状态变成ESTABLISHED(已建立连接)，
3.  客户端再向服务器发送ACK确认码，服务器接收到以后也变成ESTABLISHED。然后服务器客户端开始数据传输

如图：

{{< figure src="https://sites.google.com/a/javainterview.net/question/_/rsrc/1425457816649/misc/tcp-ip/3-way-handshake-Intro-to-transport-layer-The-internetworking-Part2.gif" caption="<span class=\"figure-number\">Figure 1: </span>图来源于Google" >}}


#### <span class="section-num">2.2.2</span> SYN扫描原理 {#syn扫描原理}

<!--list-separator-->

1.  SYN+ACK

    那么现在再回到SYN扫描上来.如果在发送第一次握手的 `SYN` flag 时，目标机器回复了=SYN+ACK=,这不就说明笔者发送的TCP包中的目标端口是开放的么！如果不开放，服务器就不会期待第三次握手了，也不会给笔者发送 `SYN+ACK` 了；如图：

    {{< figure src="http://2we26u4fam7n16rz3a44uhbe1bq2.wpengine.netdna-cdn.com/wp-content/uploads/101613_1123_PortScannin3.jpg" >}}

    图来自 <http://resources.infosecinstitute.com/port-scanning-using-scapy/>

<!--list-separator-->

2.  RST

    如果第二次握手的时候，目标机器回复的不是 `SYN+ACK`, 而是 `RST`, 就说明TCP包中的目标端口在目标机器上是关闭的；如图

    {{< figure src="http://2we26u4fam7n16rz3a44uhbe1bq2.wpengine.netdna-cdn.com/wp-content/uploads/101613_1123_PortScannin4.jpg" >}}

    图来自 <http://resources.infosecinstitute.com/port-scanning-using-scapy/>

<!--list-separator-->

3.  Filtered

    上面提及了目标端口的开放和关闭两种状态，那么，还有没有其他状态呢？什么，还有其他状态？

    如果就SYN扫描而言，就还有 filtered被过滤之一说，如果还有加上其他扫描技术， 就还有其他状态了。

    回到SYN扫描，当返回的不是服务器想建立第二次握手的包，而是ICMP的包就有可能被过滤，例如响应信息是ICMP错误信息类型3代码3(无法到达目标：端口不可达)这里出现的端口不可达，可能就是被防火墙过滤了，如果是类型3代码13(无法到达目标：通信被管理员禁止),那也是被过滤了。

    更多信息就要查询[ICMP的官方文档](https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml) 了


#### <span class="section-num">2.2.3</span> 代码解释 {#代码解释}

```python
def scan(self, args):
    dst_ip, dst_port = args
    src_port = RandShort()
    answered, unanswered = sr(IP(dst=dst_ip) / TCP(sport=src_port,
                                                   dport=dst_port, flags="S"),
                          timeout=self.timeout, verbose=False)
    for packet in unanswered:
        return packet.dst, packet.dport, "Filtered"

    for (send, recv) in answered:
        if(recv.haslayer(TCP)):
            flags = recv.getlayer(TCP).sprintf("%flags%")
            if(flags == "SA"):
                # set RST to server in case of ddos attack
                send_rst = sr(IP(dst=dst_ip) / TCP(sport=src_port,
                                                   dport=dst_port, flags="R"),
                          timeout=self.timeout, verbose=True)
                return dst_ip, dst_port, "Open"
            elif (flags == "RA" or flags == "R"):
                return dst_ip, dst_port, "Closed"
        elif(recv.haslayer(ICMP)):
            icmp_type = recv.getlayer(ICMP).type
            icmp_code = recv.getlayer(ICMP).code
            if(icmp_type == ICMP_TYPE_DESTINATION_UNREACHABLE and icmp_code in ICMP_CODE):
                return dst_ip, dst_port, "Filtered"
        else:
            return dst_ip, dst_port, "CHECK"
```

核心代码很简单，就是发送建立连接的握手请求，然后根据不同的返回结果判断不同的状态。

如果端口确定是开放，那就发送 `R` flag给目标机器结束握手 (如果不结束握手的话，那就是DDOS,这也是DDOS最常用的手段); 因为这次不是使用操作系统原生的 `socket`, 而是自行构造发送 IP数据包，所以需要使用一个很强大的构造 操作各种数据包的工具 -- [scapy](https://github.com/phaethon/scapy)

(顺便说一下，如果在Windows下安装 scapy,需要非常多的步骤，如果是Unix/Linux,只需几行命令:) )


## <span class="section-num">3</span> 后话 {#后话}

简单的扫描器就已经完成了，加上多线程的功能提高性能。

很想吐嘈一下，真的对Python 的多线程恨铁不成钢，只好换成多进程；也给 Python2 Python3 API的改变折腾得够呛，不禁让笔者怀念起Java:(

其实正如笔者开头所言的，你确定隔壁家妹子是否在家的方法有很多，你扫描端口的方法也有很多：例如 XMAS scan(TCP圣诞树扫描), FIN scan,Null scan, ACK scan, Window scan, UDP scan等。

当然你如果不想针对各种扫描都写一个扫描器，你可以使用 [nmap](https://nmap.org/) 这个地球最强大的扫描器 (没有之一). 在Python也已经有与nmap整合的强大的包 [python-nmap](http://xael.org/pages/python-nmap-en.html)

扫描器完整代码地址 <https://github.com/ramsayleung/PortScanner>

---

参考

-   <http://resources.infosecinstitute.com/port-scanning-using-scapy/>
