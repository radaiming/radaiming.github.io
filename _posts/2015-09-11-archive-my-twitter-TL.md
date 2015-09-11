---
layout: post
title: "保存 twitter timeline"
categories: python
---

之前一直是利用 twip 来保存自己 twitter 的 timeline，twip 是个 twitter API proxy，比如说，在授权后访问

~~~~~~~~
https://example.com/blabla/1.1/statuses/home_timeline.json?count=20
https://api.twitter.com/1.1/statuses/home_timeline.json?count=20
~~~~~~~~

这两个 URL 所返回的 json 是一样的。之前一直用 twip，一是最开始的脚本跑了几年了懒得改了，二是授权方便，点两下鼠标就好了，不需要管 OAuth 细节。前段时间一不留神 revoke 掉了给 twip 的 API 权限，然后怎么试图重新授权都失败。挺久之前看到一个 python 写的 [twitter 库](https://github.com/sixohsix/twitter)，star 之后一直记着，于是上周就用上了。

用了之后发现这个库用起来相当简单，整个过程基本相当于无脑的状态，授权的话，开个 python console，调用它的 oauth\_dance 函数，自动开始 PIN-based OAuth，打开它给出的 URL，授权后把 PIN 输入，它就会把 OAuth token 和 OAuth secret 写入一个文件，以后可以继续用。这次我用的不是 home_timeline.json 这个 API，直接用了我很喜欢的 Stream API，用起来也是相当简单，主页上就有 demo。不过暂时不知道一些 API 的参数该怎么传给它，现在还没这需求，等我有需要了再翻它源码吧。

另外 Arrow 才是给人用的时间库，标准库里面那个简直是在虐人。

感觉自己还是很喜欢这种封装得相当好的库，用起来真爽。
