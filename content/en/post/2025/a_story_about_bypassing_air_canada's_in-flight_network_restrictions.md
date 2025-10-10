+++
title = "A Story About Bypassing Air Canada's In-flight Network Restrictions"
date = 2025-10-10T15:29:00+08:00
lastmod = 2025-10-10T15:36:43+08:00
tags = ["network", "hacking"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Prologue {#prologue}

A while ago, I took a flight from Canada back to Hong Kong - about 12 hours in total with Air Canada.

Interestingly, the plane actually had WiFi:

{{< figure src="/ox-hugo/acwifi-connect-2.png" >}}

However, the WiFi had restrictions. For Aeroplan members who hadn't paid, it only offered [Free Texting](https://www.aircanada.com/ca/en/aco/home/fly/onboard/in-flight-entertainment-and-connectivity.html#/), meaning you could only use messaging apps like WhatsApp, Snapchat, and WeChat to send text messages, but couldn't access other websites.

If you wanted unlimited access to other websites, it would cost CAD $30.75:

{{< figure src="/ox-hugo/acwifi.jpg" >}}

And if you wanted to watch videos on the plane, that would be CAD $39:

{{< figure src="/ox-hugo/acwifi_plan.jpg" >}}

I started wondering: for the Free Texting service, could I bypass the messaging app restriction and access other websites freely?

Essentially, could I enjoy the benefits of the $30.75 paid service without actually paying the fee? After all, with such a long journey ahead, I needed something interesting to pass the 12 hours.

Since I could use WeChat in flight, I could also call for help from the sky.

Coincidentally, my roommate happens to be a security and networking expert who was on vacation at home. When I mentioned this idea, he thought it sounded fun and immediately agreed to collaborate. So we started working on it together across the Pacific.


## <span class="section-num">2</span> The Process {#the-process}

After selecting the only available WiFi network `acwifi.com` on the plane, just like other login-required WiFi networks, it popped up a webpage from `acwifi.com` asking me to verify my Aeroplan membership. Once verified, I could access the internet.

{{< figure src="/ox-hugo/onboard_success.jpg" >}}

There's a classic software development interview question: what happens after you type a URL into the browser and press enter?

For example, if you type `https://acwifi.com` and only focus on the network request part, the general process is: DNS query -&gt; TCP connection -&gt; TLS handshake -&gt; HTTP request and response.

{{< figure src="/ox-hugo/network_request_sequence_en.png" >}}

Let's consider `github.com` as our target website we want to access. Now let's see how we can break through the network restrictions and successfully access `github.com`.


## <span class="section-num">3</span> Approach 1: Domain Self-Signing {#approach-1-domain-self-signing}

Since `acwifi.com` is accessible but `github.com` is not, could I disguise my server as `acwifi.com` and route all request traffic through my server to access the target website (`github.com`)?

{{< figure src="/ox-hugo/self-sign-certificate-en.png" >}}

The idea was roughly: I modify DNS records to bind our proxy server's IP `137.184.231.87` to `acwifi.com`, then use a self-signed certificate to tell the browser that this IP and this domain are bound together, and it should trust it.

Let me first test this idea:

```shell
> ping 137.184.231.87
PING 137.184.231.87 (137.184.231.87): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
Request timeout for icmp_seq 4
^C
--- 137.184.231.87 ping statistics ---
6 packets transmitted, 0 packets received, 100.0% packet loss
```

Unexpectedly, the IP was completely unreachable via `ping`, meaning the IP was likely blocked entirely.

I tried other well-known IPs, like Cloudflare's CDN IP, and they were also unreachable:

```shell
> ping 172.67.133.121
PING 172.67.133.121 (172.67.133.121): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
Request timeout for icmp_seq 4
^C
--- 172.67.133.121 ping statistics ---
6 packets transmitted, 0 packets received, 100.0% packet loss
```

It seems this approach won't work. After all, if the IPs are directly blocked, no amount of disguise will help. This network likely maintains some IP whitelist (such as WhatsApp and WeChat's egress IPs), and only IPs on the whitelist can be accessed.


## <span class="section-num">4</span> Approach 2: DNS Port Masquerading {#approach-2-dns-port-masquerading}

When the first approach failed, my roommate suggested a second approach: try using DNS service as a breakthrough:

```shell
> dig http418.org

; <<>> DiG 9.10.6 <<>> http418.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64160
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;http418.org.			IN	A

;; ANSWER SECTION:
http418.org.		300	IN	A	172.67.133.121
http418.org.		300	IN	A	104.21.5.131

;; Query time: 3288 msec
;; SERVER: 172.19.207.1#53(172.19.207.1)
;; WHEN: Sat Oct 04 14:18:24 PDT 2025
;; MSG SIZE  rcvd: 94
```

This is good news! It means there are still ways to reach external networks, and DNS is one of them.

Looking at the record above, it shows our DNS query for `http418.org` was successful, meaning DNS requests work.


### <span class="section-num">4.1</span> Arbitrary DNS Servers {#arbitrary-dns-servers}

My roommate then randomly picked another DNS server to see if the network had a whitelist for DNS servers:

```shell
> dig @40.115.144.198 http418.org

; <<>> DiG 9.10.6 <<>> @40.115.144.198 http418.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58958
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1224
;; QUESTION SECTION:
;http418.org.			IN	A

;; ANSWER SECTION:
http418.org.		275	IN	A	104.21.5.131
http418.org.		275	IN	A	172.67.133.121

;; Query time: 1169 msec
;; SERVER: 40.115.144.198#53(40.115.144.198)
;; WHEN: Sat Oct 04 14:24:25 PDT 2025
;; MSG SIZE  rcvd: 72
```

We can actually use arbitrary DNS servers - even better!


### <span class="section-num">4.2</span> TCP Queries {#tcp-queries}

The fact that arbitrary DNS servers can be queried successfully is excellent news. DNS typically uses UDP protocol, but would TCP-based DNS requests be blocked?

```shell
> dig @40.115.144.198 http418.org +tcp

; <<>> DiG 9.10.6 <<>> @40.115.144.198 http418.org +tcp
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30355
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1224
;; QUESTION SECTION:
;http418.org.			IN	A

;; ANSWER SECTION:
http418.org.		36	IN	A	172.67.133.121
http418.org.		36	IN	A	104.21.5.131

;; Query time: 4679 msec
;; SERVER: 40.115.144.198#53(40.115.144.198)
;; WHEN: Sat Oct 04 14:28:24 PDT 2025
;; MSG SIZE  rcvd: 72
```

DNS TCP queries also work! This indicates the plane network's filtering policy is relatively lenient, standing a chance of our subsequent DNS tunneling approach.


### <span class="section-num">4.3</span> Proxy Service on Port 53 {#proxy-service-on-port-53}

It seems the plane network restrictions aren't completely airtight - we've found a "backdoor" in this wall.

So we had a clever idea: since the plane gateway doesn't block DNS requests, theoretically we could disguise our proxy server as a DNS server, expose port 53 for DNS service, route all requests through the proxy server disguised as DNS requests, and thus bypass the restrictions.

My roommate spent about an hour setting up a proxy server exposing port 53 using [xray](https://github.com/XTLS/Xray-core)&nbsp;[^fn:1], and sent me the configuration via WeChat:

The proxy server configuration my roommate set up with Xray included the following sample configuration:

```json
{
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "our-proxy-server-domain",
            "port": 53,
            "users": [
              {
                "id": "some-uuid",
                "flow": "xtls-rprx-vision",
                "encryption": "none",
                "level": 0
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tlsSettings": {
          "allowInsecure": false,
          "allowInsecureCiphers": false,
          "alpn": [
            "h2"
          ]
        }
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom"
    },
    {
      "tag": "block",
      "protocol": "blackhole"
    }
  ]
}
```

And I already had an xray client on my computer, so no additional software was needed to establish the connection.

{{< figure src="/ox-hugo/dns-server-proxy-en.png" >}}

Everything was ready. The exciting moment arrived - pressing enter to access `github.com`:

```shell
/Users/ramsayleung [ramsayleung@ramsayleungs-Laptop] [18:28]
> curl -v github.com -x socks5://127.0.0.1:10810
*   Trying 127.0.0.1:10810...
* Connected to 127.0.0.1 (127.0.0.1) port 10810
* SOCKS5 connect to 172.19.1.1:80 (locally resolved)
* SOCKS5 request granted.
* Connected to 127.0.0.1 (127.0.0.1) port 10810
> GET / HTTP/1.1
> Host: github.com
> User-Agent: curl/8.4.0
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Content-Length: 0
< Location: https://github.com/
<
* Connection #0 to host 127.0.0.1 left intact

/Users/ramsayleung [ramsayleung@ramsayleungs-Laptop] [18:28]
> curl -v github.com -x socks5://127.0.0.1:10810
*   Trying 127.0.0.1:10810...
* Connected to 127.0.0.1 (127.0.0.1) port 10810
* SOCKS5 connect to 172.19.1.1:80 (locally resolved)
* SOCKS5 request granted.
* Connected to 127.0.0.1 (127.0.0.1) port 10810
> GET / HTTP/1.1
> Host: github.com
> User-Agent: curl/8.4.0
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Content-Length: 0
< Location: https://github.com/
<
* Connection #0 to host 127.0.0.1 left intact
```

The request actually succeeded! github.com returned a successful result!

This means we've truly broken through the network restrictions and can access any website!

We hadn't realized before that xray could be used in this clever way :)

Here we exploited a simple cognitive bias: not all services using port 53 are DNS query requests.


## <span class="section-num">5</span> Ultimate Approach: DNS Tunnel {#ultimate-approach-dns-tunnel}

If Approach 2 still didn't work, we had one final trick up our sleeves.

Currently, the gateway only checks whether the port is 53 to determine if it's a DNS request.
But if the gateway were stricter and inspected the content of DNS request packets, it would discover that our requests are "disguised" as DNS queries rather than genuine DNS queries:

{{< figure src="/ox-hugo/intercept-dns-request-en.png" >}}

Since disguised DNS requests would be blocked, we could embed all requests inside genuine DNS request packets, making them DNS TXT queries. We'd genuinely be querying DNS, just with some extra content inside:

{{< figure src="/ox-hugo/dns-tunnel-en.png" >}}

However, this ultimate approach requires a DNS Tunnel client to encapsulate all requests. I didn't have such software on my computer, so this remained a theoretical ultimate solution that couldn't be practically verified.


## <span class="section-num">6</span> Conclusion {#conclusion}

With the long journey ahead, my roommate and I spent about 4 hours remotely breaking through the network restrictions, having great fun in the process, proving that our problem-solving approach was indeed feasible.

The successful implementation of the solution was mainly thanks to my roommate, the networking expert, who provided remote technical and conceptual support.

The only downside was that although we broke through the network restrictions and could access any website, the plane's bandwidth was extremely limited, making web browsing quite painful. So I didn't spend much time surfing the web.

For the remaining hours, I rewatched the classic 80s time-travel sci-fi movie trilogy: `"Back to the Future"`&nbsp;[^fn:2], which was absolutely fantastic.

Finally, I hereby solemnly declare:

This technical exploration is for learning and research purposes only. We strictly adhere to relevant regulations and service terms.

[^fn:1]: <https://github.com/XTLS/Xray-core>
[^fn:2]: <https://movie.douban.com/subject/1300555/>
