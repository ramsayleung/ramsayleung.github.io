+++
title = "flask牛刀小试之微信公众号开发"
description = "An flask project"
date = 2017-02-23T00:00:00-08:00
lastmod = 2024-12-30T22:32:24-08:00
tags = ["python", "flask"]
categories = ["python"]
draft = false
toc = true
+++

[flask](http://flask.pocoo.org/) 是一个轻量级的python 框架(官网称为微型框架),很容易上手，之前因为笔者跟朋友开发小程序的时候使用过 flask,过后就遗忘了。

为了重拾flask, 笔者决定写点小东西，之前开发小程序，不如现在再玩玩公众号开发


## <span class="section-num">1</span> 验证服务器 {#验证服务器}

开发公众号之前，要先验证服务器的有效性，官网有详细的说明：[公众开发平台文档](https://mp.weixin.qq.com/wiki/8/f9a0b8382e0b77d87b3bcc1ce6fbc104.html)

| 参数      | 描述                                                      |
|---------|---------------------------------------------------------|
| signature | 微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。 |
| timestamp | 时间戳                                                    |
| nonce     | 随机数                                                    |
| echostr   | 随机字符串                                                |

校验流程：
加密/校验流程如下：

1.  将token、timestamp、nonce三个参数进行字典序排序
2.  将三个参数字符串拼接成一个字符串进行sha1加密
3.  开发者获得加密后的字符串可与signature对比，标识该请求来源于微信

流程并不复杂，官网给出了代码示例，只不过是PHP的，换成python 也是很容易滴：

```python
@app.route('/',methods=['GET','POST'])
        def wechat():
        if request.method=='GET':
            token='your token'
            data=request.args
            signature=data.get('signature','')
            timestamp=data.get('timestamp','')
            nonce =data.get('nonce','')
            echostr=data.get('echostr','')
            s=[timestamp,nonce,token]
            s.sort()
            s=''.join(s)
        if(hashlib.sha1(s).hexdigest()==signature):
        return make_response(echostr)
        else:
            rec=request.stream.read()
            xml_rec=ET.fromstring(rec)
            tou = xml_rec.find('ToUserName').text
            fromu = xml_rec.find('FromUserName').text
            content = xml_rec.find('Content').text
            xml_rep = "
<xml>
<ToUserName><![CDATA[%s]]></ToUserName>
<FromUserName><![CDATA[%s]]></FromUserName>
<CreateTime>%s</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[%s]]></Content>
<FuncFlag>0</FuncFlag>
</xml>"
```

这样服务器就校验成功了，就可以编写相关的业务逻辑了


## <span class="section-num">2</span> 歌词查询 {#歌词查询}

笔者自己平时是打开音乐播放器，戴上耳机，就开始播放音乐；所以经常出现笔者听到某首歌曲觉得旋律非常熟悉，但是就无法想起歌名的情况，这种感觉实在不好，所以笔者觉得可以编写一个通过歌词查询歌曲，并返回所有歌词的功能。

思路大概是编写爬虫，通过歌词进行查询，然后对返回的html 页面进行检索和信息提取。剩下的事就是爬虫和解析页面了，笔者是使用[虾米](http://www.xiami.com/) 进行歌词查询的，使用 request 发送 http 请求，使用 lxml进行解析，其他就不一一细表了


## <span class="section-num">3</span> 单词查询 {#单词查询}

有时候，笔者在微信需要查询单词，但是又不想退出微信，所以就打算用公众号来查单词其实很简单，就是服务获取用户发给微信公众号的数据，再去请求有道之类词典的api,再把结果返回给服务器，服务器转发给用户


## <span class="section-num">4</span> 电影查询 {#电影查询}

有时无聊想去看电影，但是不知道看什么电影，因为选择太多，质量参差不齐的片太多了所以笔者会先去豆瓣看一下新上影的电影，看一下评分，然后再决定看什么电影。所以，笔者可以把这个功能搬到公众号来。如何实现呢？还是爬虫


## <span class="section-num">5</span> 总结 {#总结}

感觉这次开发公众号，笔者就是用 flask 编写 restful api, 然后做的其他事情就是编写爬虫。

[项目github地址](https://github.com/ramsayleung/SamrayJustForFun)
